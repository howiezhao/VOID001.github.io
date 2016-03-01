---
title: Wine Notes -- Coding Hints
---

## About Coding Style(Habit)

* 代码风格模仿上下文的代码风格, 比如 wpcap.c 内所有的if语句风格都是if<space>(exp), 那么新写的代码也要是这个风格
* 一定要使用恰当的变量类型, 例如`int status`要改为`NTSTATUS status`之类的
* 在`TRACE` 的前后加入空行,增加代码的可读性
* 在修改某一个bug的时候, 要保证将引起这个bug的所有代码, 以及和这个bug相关的所有代码修改掉, 多用grep
* ok 宏测试到的参数就不需要trace输出了
* 函数定义一定要含有调用惯例(Coding Conventions) CDECL STDCALL..
* 函数参数类型不要用 LP* P* 同时变量名也不要以p开头, 变量名常用全小写字母
* spec 文件里的 ordinal number 忽略掉 用 @ 代替, 现在已经不关心 ordinal number 了
* TRACE 信息要尽量少(?)
* if while for 后面加 空格 再加 "()"
* if内只有一个语句的话不要加大括号
* 给代码块和代码块之间加上一定的空行使得代码可读性更好

## About Error Handling

* 在自己编写的函数中, 调用其他的函数出现错误的时候, 要注意处理错误, 需要在Win下写testcase进行对比
* 错误处理的时候, 注意设置好返回值, 以及Errorcode, SetLastError用于设定ErrorCode
* 不要用十六进制数字表示错误, 用定义好的宏来表示 (e.g. 不要用 `0x3` 用 `ERROR_PATH_NOTFOUND`)
* 不需要关注代码成功运行的时候错误代码是什么, 在代码运行正确的时候不会去考虑错误代码的问题
* 不要自己做NTSTATUS -> ERROR Code 的映射, 直接使用函数 `RtlNtStatusToDosError` 即可
* 注意Calling Convention 要对应上

## About Memory

* 当程序SEGFAULT的时候, 注意检查是否出现了 Double-free 的情况
* 自己写的代码要注意避免memory leak, 所有手动分配的内存都要记得收回, 同时要注意WinAPI的说明, 有的函数也需要手动收回内存
* 使用HeapAlloc而不是malloc分配内存, 使用HeapFreee而不是free释放内存
* HeapAlloc 分配内存之后检查内存分配是否成功
* 每一个return PATH 之前都要将内存Free掉
* 不需要分配内存就能解决的问题就不要进行内存分配

## Handle Build Warning

* 如果是下面这个错错误:
{% highlight shell %}
In file included from wpcap.c:26:0:
../../include/ntstatus.h:331:0: warning: "STATUS_FLOAT_UNDERFLOW" redefined
#define STATUS_FLOAT_UNDERFLOW           ((NTSTATUS) 0xC0000093)
^
In file included from ../../include/windef.h:266:0,
   from ../../include/windows.h:37,
   from ../../include/winsock.h:110,
   from ../../include/winsock2.h:47,
   from wpcap.c:22:
   ../../include/winnt.h:607:0: note: this is the location of the previous definition
#define STATUS_FLOAT_UNDERFLOW           ((DWORD) 0xC0000093)
{% endhighlight %}

那么用下面的方法来解决:
将`#include "ntstatus.h"`放在include最上面, 然后紧跟着 `#define WIN32_NO_STATUS`
