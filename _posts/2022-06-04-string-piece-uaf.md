---
title: 一个由错误的使用 String Piece 导致的 Use After Free 问题排查
---

# TL;DR

- 构造 `StringPiece` 的时候要保证他底层的 string 生命周期比 `StringPiece` 长
- Address Sanitizer is your friend
- From my friend: 人生苦短，请用 Clang

# Background

StringPiece (or `absl::string_view` `std::string_view`) 他们都是为了让开发者可以访问而不持有(own) 一个字符串而准备的便利 utility，但是错误的使用他们可能会导致很危险的内存问题，我们来看下面这段代码

（以 absl::string_view 在 C++14 上的实现为准）

```cpp
#include <cstdio>
#include <functional>
#include <iostream>
#include <string>
#include <utility>
#include <vector>
#include "absl/strings/string_view.h"

typedef std::pair<absl::string_view, absl::string_view> StringViewPair;

int main(void) {
  std::string s1;
  std::string s2;
  s1 = "Good Morning Evil \n";
  s2 = "Good Evening Evil \n";
  StringViewPair pair = std::make_pair(s1, s2);
	printf("%p %p %p %p\n", s1.data(), s2.data(), pair.first.data(), pair.second.data());
	printf("%s\n%s\n", pair.first.data(), pair.second.data());

  return 0;
}
```

这段代码看起来稀松平常，首先创建了两个 string s1, s2, 在堆上分配了两个字符串的空间，然后使用这两个 string 作为参数传递给 `std::make_pair` 构造一个 `absl::string_view` 的 pair,

运行这段代码，会发现输出的内容出现了乱码/不完整等问题，

```bash

(っ・ω・)っ ./demo                                                                                                                                                                                                                                                                                                 remote ⚛
Xі
```

这里的输出和我们的字符串完全对不上，敏锐的读者可能看到这里已经想到了是 pair.first / pair.second 的 data 指向的内存出现了问题，那我们先把各个 str 的 data 指针都打出来看看，按照我们的想法以及 `absl::string_view` 的实现，这个 `StringViewPair` 应该存了 s1, s2 的底层指针，但是打印出来的字符串却是 dirty 的

```cpp
class string_view {
// ...

template <typename Allocator>
  string_view(  // NOLINT(runtime/explicit)
      const std::basic_string<char, std::char_traits<char>, Allocator>& str
          ABSL_ATTRIBUTE_LIFETIME_BOUND) noexcept
      // This is implemented in terms of `string_view(p, n)` so `str.size()`
      // doesn't need to be reevaluated after `ptr_` is set.
      // The length check is also skipped since it is unnecessary and causes
      // code bloat.
      : string_view(str.data(), str.size(), SkipCheckLengthTag{}) {}
// ...
}
```

# Debugging

首先，我们在上面的 sample 代码中添加一行代码 (上面的 sample 中已添加)

```cpp
printf("%p %p %p %p\n", s1.data(), s2.data(), pair.first.data(), pair.second.data());
```

在我本地的输出如下

```
0x60300000efe0 0x60300000efb0 0x60300000ef80 0x60300000ef50
```

我们发现 `s1.data()` 和 `pair.first.data()` 他们两个的内存地址并不相同，这说明，在 make pair 的时候，可能发生了内存拷贝，通过查阅 [cppreference](https://en.cppreference.com/w/cpp/utility/pair/make_pair) 我们得到了如下的解释

> The deduced types `V1` and `V2` are [std::decay](http://en.cppreference.com/w/cpp/types/decay)<T1>::type and [std::decay](http://en.cppreference.com/w/cpp/types/decay)<T2>::type (the usual type transformations applied to arguments of functions passed by value) unless application of [std::decay](https://en.cppreference.com/w/cpp/types/decay) results in [std::reference_wrapper](http://en.cppreference.com/w/cpp/utility/functional/reference_wrapper)<X> for some type `X`, in which case the deduced type is `X&`.
> 

看起来，make_pair 在进行类型推断的时候，会将参数类型推断后，按照值的方式传递，而非按照引用的方式传递，因而在这个过程中，内存发生了拷贝

分析 C++ 内存相关问题的利器就是 **AddressSanitizer** (ASAN), 我们挂上 ASAN 跑这个程序来看结果如何，在构建参数中添加 `-fsanitize=address` 后再次运行我们的代码，得到了如下的输出

```
0x60300000efe0 0x60300000efb0 0x60300000ef80 0x60300000ef50
=================================================================
==2649769==ERROR: AddressSanitizer: heap-use-after-free on address 0x60300000ef80 at pc 0x7f951d7295ce bp 0x7ffde4a9d470 sp 0x7ffde4a9cc20
READ of size 2 at 0x60300000ef80 thread T0
    #0 0x7f951d7295cd  (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8a5cd)
    #1 0x7f951d729d8a in vprintf (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8ad8a)
    #2 0x7f951d729e47 in __interceptor_printf (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8ae47)
    #3 0x55c93507994e in main /data04/playground/cpp/string-piece-uaf/main.cc:19
    #4 0x7f951b9472e0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x202e0)
    #5 0x55c935079669 in _start (/data04/playground/cpp/string-piece-uaf/build/demo+0x2669)

0x60300000ef80 is located 0 bytes inside of 20-byte region [0x60300000ef80,0x60300000ef94)
freed by thread T0 here:
    #0 0x7f951d7621f0 in operator delete(void*) (/usr/lib/x86_64-linux-gnu/libasan.so.3+0xc31f0)
    #1 0x55c935079e54 in std::pair<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::~pair() /usr/include/c++/6/bits/stl_pair.h:194
    #2 0x55c9350798b3 in main /data04/playground/cpp/string-piece-uaf/main.cc:16
    #3 0x7f951b9472e0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x202e0)

previously allocated by thread T0 here:
    #0 0x7f951d761bf0 in operator new(unsigned long) (/usr/lib/x86_64-linux-gnu/libasan.so.3+0xc2bf0)
    #1 0x7f951d018116 in void std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >::_M_construct<char*>(char*, char*, std::forward_iterator_tag) (/usr/lib/x86_64-linux-gnu/libstdc++.so.6+0x120116)
    #2 0x7ffde4a9d5cf  (<unknown module>)
    #3 0x5  (<unknown module>)

SUMMARY: AddressSanitizer: heap-use-after-free (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8a5cd)
Shadow bytes around the buggy address:
  0x0c067fff9da0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9db0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9dc0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9dd0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9de0: fa fa fa fa fa fa fa fa fa fa fd fd fd fa fa fa
=>0x0c067fff9df0:[fd]fd fd fa fa fa 00 00 00 07 fa fa 00 00 00 07
  0x0c067fff9e00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9e10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9e20: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9e30: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff9e40: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==2649769==ABORTING
```

对于不熟悉 ASAN 的读者，上面的输出最初看可能有点难以理解，我们来关注一下重点部分

```
==2649769==ERROR: AddressSanitizer: heap-use-after-free on address 0x60300000ef80 at pc 0x7f951d7295ce bp 0x7ffde4a9d470 sp 0x7ffde4a9cc20
```

首先最前面列出了进程ID，然后是错误的类型 `heap-use-after-free` 是一个堆上内存在释放后又被使用的错误，具体的地址我们也可以看到，就是 `pair.first.data()` ，继续查看 ASAN 的输出

```
READ of size 2 at 0x60300000ef80 thread T0
    #0 0x7f951d7295cd  (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8a5cd)
    #1 0x7f951d729d8a in vprintf (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8ad8a)
    #2 0x7f951d729e47 in __interceptor_printf (/usr/lib/x86_64-linux-gnu/libasan.so.3+0x8ae47)
    #3 0x55c93507994e in main /data04/playground/cpp/string-piece-uaf/main.cc:19
    #4 0x7f951b9472e0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x202e0)
    #5 0x55c935079669 in _start (/data04/playground/cpp/string-piece-uaf/build/demo+0x2669)

```

这里我们首先忽略 Stack Trace 信息，来看两个关键点

```
READ of size 2 at 0x60300000ef80 thread T0
```

说明了，这一个 `heap-use-after-free` 问题的触发是由读取这个内存而导致的，那么我们看一下这个内存在哪里被释放了

```
0x60300000ef80 is located 0 bytes inside of 20-byte region [0x60300000ef80,0x60300000ef94)
freed by thread T0 here:
    #0 0x7f951d7621f0 in operator delete(void*) (/usr/lib/x86_64-linux-gnu/libasan.so.3+0xc31f0)
    #1 0x55c935079e54 in std::pair<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::~pair() /usr/include/c++/6/bits/stl_pair.h:194
    #2 0x55c9350798b3 in main /data04/playground/cpp/string-piece-uaf/main.cc:16
    #3 0x7f951b9472e0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x202e0)
```

根据这个 Call Trace，我们能够对应到，在代码 `main.cc`  L16 的位置，这个 basic string （也就是 `std::string` 就被释放掉了。

# Analyze

我们来回顾一下最初的代码片段，现在我们知道了, `std::make_pair` 会 copy 两个 string `s1` `s2`并且产生一个 `std::pair<std::string, std::string>` 将他们作为参数传递给等号左边的 `StringViewPair` ，而等号左边的 StringViewPair 不 own 这两块内存，只是保存一个到他们的 Reference，因而，当这个构造结束后，等号右边的 `std::pair<std::string, std::string>` 就被释放掉，因而这两个拷贝出来的 `std::string` 的内存也随之被释放掉（如 ASAN 的输出）

因此就导致了 heap use after free 问题的发生

# Solution

## #1

因为 C++ 仅能根据右值进行类型推断，不像 Rust 那样聪明，可以根据右值以及该变量的使用方式进行类型推断，所以我们的解法之一就是，在 make pair 的时候给出类型

```cpp
StringViewPair pair = std::make_pair<absl::string_view, absl::string_view>(s1, s2);
```

这样，在 make pair 的时候就会得知我们需要的是 `std::pair<absl::string_view, absl::string_view>` 而非 `std::pair<std::string, std::string>` , 从而直接根据所给的参数调用 `absl::string_view` 构造函数，将 s1, s2 转为 `const std::string&` 传递给构造函数，避免了将 s1 s2 拷贝出一个临时的字符串

## #2

但是上面这种改法会让你的 Linter 抱怨起来

```
build/explicit_make_pair: For C++11-compatibility, omit template arguments from make_pair OR use pair directly OR if appropriate, construct a pair directly
```

因而这里还有一种更为美观的改法，参考 cppreference 上面对于 `make_pair` 的描述，我们使用 `std::ref` 将两个对象 `s1` `s2`转换为 `std::string&` （实际上是转换为 `reference_wrapper<std::string>` ），从而实现相同的效果并且避免了 Linter 的警告

# Misc

本文中展现的这个问题，是 StringPiece / string_view 的一个使用陷阱，并且目前没有很好的编译期检查出这种问题的方法

[https://github.com/isocpp/CppCoreGuidelines/issues/1038](https://github.com/isocpp/CppCoreGuidelines/issues/1038)

Github 上对于此问题的讨论，最终止于

> the use-after-free potential with `string_view`
 is accounted for in the Lifetime profile, even if there may be bugs or limitations in preliminary implementations that mean it is not diagnosed.
> 

因而，在 C++ 开发中，**将 address sanitizer 时刻保持开启，是很重要的**，能够帮助我们发现很多编译期无法检查出的问题

那么到这里了，本文还有一个结论没有解释

> From my friend: 人生苦短，请用 Clang
> 

对一些奇怪的 GCC Clang 表现感兴趣的朋友可以继续阅读，单纯来了解 StringPiece pitfall 的朋友可以在这里停下了，下面要介绍的内容可能比较黑暗

## Darkness Moment

为了继续我们的旅程，我们需要大家有多种编译器，这时候就应该拿出 godbolt 在线编译器平台了

因为我没能够在 godbolt 里成功引用 absl 并构建出二进制，在 godbolt 里将 absl::string_view 替换为了 leveldb 的 `Slice` ，因为各类 `string_view` 的实现，尤其是构造函数的实现都很类似，因而我们的问题在 `Slice` 上也可以得到复现

[https://godbolt.org/z/rcvs55zfa](https://godbolt.org/z/rcvs55zfa)

上面这个链接里放了类似于我们本文中描述的 use after free 问题的代码，但是我们发现在 GCC 的某些特定版本下，ASAN 竟然没有报错

![](/assets/imgs/string-piece-uaf1.png)

而当我们稍加修改 L117, 去掉 “\n” 之后， asan 又输出了

![](/assets/imgs/string-piece-uaf2.png)

经过一番排查，发现，是由两个原因导致的

1. 某些版本的 GCC（如 6.3）会将 `printf("%s\n");` 在编译时转变为对 `puts` 的调用(可以查看 godbolt 的汇编代码进行确认)
2. 而 6.3 的 GCC 的 libasan.so 恰好没有对 `puts` 进行 hook

因而导致了上述情况的发生，也就说明了 ASAN 并不是万能的，想要避免一切内存问题，还是要靠程序员自身的本事（🤦‍♀️）

同时我们还注意到在 L114 行有一个注释，当使用 GCC 版本低于 5.0 时（如 4.9.3）这个字符串根本不会被 Copy，结果就导致了 `s1` 和 `pair.first` 指向了相同的字符串地址空间。这是因为在 GCC 5.0 以下，对于字符串拷贝的优化，GCC 使用的是 COW，也就是说，因为我们的代码在创建了 `pair.first` 后没有对字符串有任何写操作，因而他直接和 `s1` 共用了内存，也就没有 heap-use-after-free 这个问题了。

# Ending

本文的产出离不开几个朋友的帮助，感谢 Wei，可怜等朋友帮忙一起 Debug 奇怪的 ASAN 不能 work 的问题（真的 Debug 了两天才找出问题。。，本来这个博客只需要半天时间就写出来了，结果搞了两天）
