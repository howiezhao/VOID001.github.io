---
title: 使用  sync point 对多线程应用进行可控的并发测试
date: '2022-04-09 22:00:27'
---

> 在测试多线程应用程序的时候，我们往往会构造一些多线程并发执行的 test case 用来测试是否有 race condition 或者 unexpected behavior 出现。通常情况下，开发者可能会将多个 function 并发执行 N 次，以此来判断最终结果是否符合预期，这可能对于大多数情况是足够的。但是，当我们已经通过排查问题发现了一个因为 race 的原因导致的 Bug 的时候，我们很难通过上述通常的方法去重现它。为了能够构造出运行顺序完全可控的并发场景测试 case，rocksdb  实现了一套测试工具 SyncPoint，以此来解决上面的这个问题。本文介绍 rocksdb SyncPoint 的使用方法，以及其实现原理。

# SyncPoint API
首先介绍 SYNC POINT 的 API 接口, 接口定义在 `test_util/sync_point.h` 内：

```C++
void LoadDependency(const std::vector<SyncPointPair>& dependencies);
void SetCallBack(const std::string& point,
                const std::function<void(void*)>& callback);
void EnableProcessing();
void DisableProcessing();
void Process(const Slice& point, void* cb_arg = nullptr);
```

SyncPoint 被作为一个单例定义在整个 Project 内，并提供如下几个宏定义简化用户的使用：

* `TEST_SYNC_POINT(name)`: 在代码中定义一个 SyncPoint
* `TEST_SYNC_POINT_CALLBACK(name, cbarg)`: 在代码中定义一个可以进行回调的 SYNC POINT, 回调函数 `cb` 通过 `SetCallback` 注册给该 SyncPoint, 并且在这个宏被执行的时候以 `cb(cbarg)` 的形式调用
* `TEST_IDX_SYNC_POINT(name, idx)`: 只是将 `TEST_SYNC_POINT` 中的 “name" 换成了 "name" + "idx" 的字符串, 其他实现与 `TEST_SYNC_POINT` 完全相同

我们结合下面的例子介绍下 SyncPoint API 的使用方法

# Examples

## 1. 使用 SyncPoint 实现多线程代码按照固定顺序执行

为了简化实现以及突出 SyncPoint 的使用，我们选用一个 Hello World 例子来介绍 SyncPoint: 使用两个并发运行的 Thread 按顺序输出 1 - 4 的数字

```CPP
void Fun1() {
   printf("1\n");
   TEST_SYNC_POINT("SyncPointDemo::Fun1:1");
   TEST_SYNC_POINT("SyncPointDemo::Fun1:4");
   printf("4\n");
}

void Fun2() {
    TEST_SYNC_POINT("SyncPointDemo::Fun2:2");
    printf("2\n");
    printf("3\n");
    TEST_SYNC_POINT("SyncPointDemo::Fun2:3");
}
```

如上所示，是两个打印函数 Fun1, Fun2. 在上述两个函数中设置了总共 4 个 SyncPoint, 接下来我们需要编写测试来定义 SyncPoint 之间的依赖关系（先后顺序）：

```CPP
TEST(SyncPointDemoTest, SequentialPrintNumbers) {
    ROCKSDB_NAMESPACE::SyncPoint::GetInstance()->EnableProcessing(); 
    ROCKSDB_NAMESPACE::SyncPoint::GetInstance()->LoadDependency(
        {{"SyncPointDemo::Fun1:1", "SyncPointDemo::Fun2:2"}, 
         {"SyncPointDemo::Fun2:3", "SyncPointDemo::Fun1:4"}});
    std::thread t1(Fun1);
    std::thread t2(Fun2);
    t1.join();
    t2.join();
    SyncPoint::GetInstance()->DisableProcessing();
}
```

关注这里的 `LoadDependency` 函数，他接受一系列的 `SyncPointPair`, 每一个 `SyncPointPair` `{u, v}` 可以看作是一条有向边 ` [u] -> [v] ` 表明 `u` 发生在 `v` 之前。 在实际开发中，我们为了方便在测试里引用预先定义好的大量 SyncPoint ，我们会对每一个 SyncPoint 赋予有意义的名字。这里我们就通过 `LoadDependency` 构建了如下的依赖关系

```
[SyncPointDemo::Fun1:1] -> [SyncPointDemo::Fun2:2]
[SyncPointDemo::Fun2:3] -> [SyncPointDemo::Fun1:4]
```

而因为在同一个 Thread 内， `SyncPointDemo::Fun2:2` 一定会在 `SyncPointDemo::Fun2:3` 之前，所以四个 SyncPoint 的依赖关系图为一条链

```
[SyncPointDemo::Fun1:1] -> [SyncPointDemo::Fun2:2] -> [SyncPointDemo::Fun2:3] -> [SyncPointDemo::Fun1:4]
```

因而，程序的最终输出受到 SyncPoint 的控制，会始终为 "1\n2\n3\n4\n"。
`EnableProcessing` 函数会开启 SyncPoint 的功能，如果我们在测试入口处不调用这个函数，则 SyncPoint 不会生效，也就会看到乱序的输出结果。同理 `DisableProcessing` 停止 `SyncPoint` 的处理。


## 2. 使用 TEST_SYNC_POINT_CALLBACK 实现在测试用例中控制特定函数的返回值

在运行各种单元测试中，我们希望测试一些函数的异常返回情况是否能够被正确处理，而一个函数的异常返回情况可能有多种，因而我们也需要一种方便的编码手段+测试工具来注入不同的错误码，下面这段例子展示了如何通过 `TEST_SYNC_POINT_CALLBACK` 在特定测试中给某函数注入错误码:

```CPP
int do_somthing() {
    int retval = 0;

    // complex logic here
    ...

    // When executing to this point, the callback will be invoked with argument `retval`
    TEST_SYNC_POINT_CALLBACK("SyncPointDemo::retval_inject", &retval);

    return retval;
}
```

假定我们有一个函数 `do_something`, 他会完成若干工作后返回 `int` 类型的错误码给调用者，而调用者会根据不同的错误码进行不同的处理，我们现在希望在测试中覆盖掉错误码，让它在本测试中永远返回 `-1`, 则编写如下的测试代码：

```
TEST(SyncPointDemoTest, CallbackSetErrorCode) {
    SyncPoint::GetInstance()->EnableProcessing();
    std::function<void(void*)> cb = [] (void *arg) -> void {
      int *ptr = static_cast<int *>(arg);
      *ptr = -1;
    };
    SyncPoint::GetInstance()->SetCallBack("SyncPointDemo::retval_inject", cb);

    auto p = do_something();
    ASSERT_EQ(p, -1);
}
```

这个测试程序会通过测试，`p` 的值被改写为 `-1`. 我们来看 `SetCallback` 这个函数，它会对一个给定的 SyncPoint 设置 Callback 调用，Callback 函数的签名为 `void(void *)`, 因而我们编写了一个 lambda 函数 `cb` 对错误码进行改写，当上述函数 `do_something` 执行到 `TEST_SYNC_POINT_CALLBACK` 的时候，会调用 `cb` 并且将 `&retval` 内存处的值修改为 `-1`. 以此方法实现了错误码的注入。

SyncPoint 的灵活性使得它的功能远不止我所 Demo 的这些，rocksdb 在构建一些特定的 Transaction Sequence 的测试用例中也大量的使用了 Sync Point，同时 `TEST_SYNC_POINT_CALLBACK` 定义的 SyncPoint 的顺序也是可以 `LoadDependency` 控制的

# SyncPoint Internals

SyncPoint 的功能很强大，接下来看下他的实现, 他的基本原理是通过拓朴排序的方式, 从入度为 0 的 SyncPoint 执行并且随着执行的过程移除已经执行完毕的点，重复此过程直到所有的 SyncPoint 都被执行完毕, 当一个点不满足执行条件（入度为0）的时候就通过 conditional variable 将此线程置为 sleep 状态。

核心函数为 `LoadDependency` 以及 `Process`, 我们上面说的所有 `TEST_SYNC_POINT_*` 宏都是最终去调用 `Process` 函数。

## LoadDependency

```CPP
void SyncPoint::Data::LoadDependency(const std::vector<SyncPointPair>& dependencies) {
  std::lock_guard<std::mutex> lock(mutex_);
  successors_.clear();
  predecessors_.clear();
  cleared_points_.clear();
  for (const auto& dependency : dependencies) {
    successors_[dependency.predecessor].push_back(dependency.successor);
    predecessors_[dependency.successor].push_back(dependency.predecessor);
    point_filter_.Add(dependency.successor);
    point_filter_.Add(dependency.predecessor);
  }
  cv_.notify_all();
}
```

`successor_` 与 `predecessor_` 都是 unordered_map, 通过这两个 hash map 存储了点之间的关系, `successor_[x] = y` 意味着 `x -> y` 这样一条边，而 `predecessor_[x] = y` 意味着 `y -> x` 这样一条边. `point_filter` 我们放到后面来讲.


## Process
接下来我们看 Process 函数, 因为函数较长，就不贴过来了, 这里放一个源码链接

[Source Code](https://github.com/facebook/rocksdb/blob/678ba5e41cc7ae62a7d99a8307c2477f4de6d5d2/test_util/sync_point_impl.cc#L104)

首先我们来看 `point_filter_`, 它是一个 `BloomFilter` ，用来过滤掉一些完全不存在的 SyncPoint, 提前结束 `Process` 流程以免不必要的 mutex lock。
跳过 Marker 的逻辑，我们可以看到一个 `PredecessorsAllCleared` 函数，它会去检查是否该节点的所有前置节点都已经执行完毕（执行完毕的节点会被插入到 `cleared_points_` 这个集合中），如果没有执行完毕，则该 SyncPoint 的执行就会阻塞在条件变量 `cv` 上, 否则就继续执行后续逻辑。
接下来会根据注册的 Callback 对此 SyncPoint 调用相应的 Callback。也就是上述例子 2 中解释的流程。
最后，因为这个点能够被执行已经表明它没有任何前置节点没有被执行，它自己也已经执行完毕，把它自己加入到 `cleared_points_` 里，以此简化了拓扑排序中需要计算每个节点的入度这一过程，很是巧妙

## Misc

另外，rocksdb 仅会在 DEBUG 模式下生成 SyncPoint 相关的代码. 保证了代码在 Release 模式下的性能完全不受到影响。


# Final

本文介绍了如何构造执行顺序确定的并发测试用例，介绍了 rocksdb 的一种确定性调试工具 SyncPoint 以及其用法和原理。在 Rocksdb 内，除了 SyncPoint 这种测试工具，还提供了 FailPoint 以及 KillPoint 等用来模拟在代码执行到特定情况下产生特定错误，接收到特定信号等测试工具。对于存储系统的代码来说，拥有足量的可回归的测试用例是非常重要的，同时一个复杂的存储系统涉及到事务，前后台线程交互等复杂逻辑，而 RocksDB 通过提供上述工具为开发者降低了测试代码的编写负担，也保证了系统的可靠性。
同时感谢我的同事在公司分享了 SyncPoint 这样的工具，因为有了同事的介绍才会有本篇文章，希望这个简单易用的工具能够帮助更多开发者写出更加完备的测试。


