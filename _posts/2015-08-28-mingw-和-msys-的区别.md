---
title: mingw 和 MSYS 的区别
---
================



#####  [August 28, 2015](https://web.archive.org/web/20210120194017/https://void-shana.moe/linux/mingw-%e5%92%8c-msys-%e7%9a%84%e5%8c%ba%e5%88%ab.html "9:01 am") 
[VOID001](https://web.archive.org/web/20210120194017/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210120194017/https://void-shana.moe/linux/mingw-%e5%92%8c-msys-%e7%9a%84%e5%8c%ba%e5%88%ab.html#respond)





最近在wine下使用msys和mingw进行一些交叉编译和测试0.0 对于什么时候用msys的shell 什么时候用mingw 有些困惑 ,现在大致明白了二者的区别 ,在此总结一下.


mingw实际上是minimalist GNU for windows的缩写 ,也就是mingw提供了一个简化的GNU开发组件(如automake gcc 等) 是gcc 和 gnu binutils 在Windows上的移植,这些程序可以直接在Windows的环境下运行 他们依赖的是msvcrt.dll这些Windows下系统自带的库,不借助外部库就可以使用 .


msys是Minimal SYStem 的缩写 ,他提供了调用Linux下的系统函数 (如 open flock等)的一个集合 , 因此msys可以用来编译原生的Linux上的代码 , 但是mingw不可以,因为mingw内没有提供调用Linux系统函数的功能,不过相应的,运行在msys下编译好的具有POSIX C代码的可执行程序的时候 ,需要加载msys自己的dll , 因此这个程序不能独立的在windows系统下运行 而需要依赖msys的dll


如果想要验证我的说法的话 , 可以在自己机器上先配置好msys32 安装好 ,然后 首先在MSYS下编译一个有POSIX C代码的程序 编译为 exe格式 ,然后 在Windows的资源管理器中找到这个程序直接执行,  你会发现这个程序执行不了 错误是需要MSYS的dll  , 这样就验证了我的第二条总结 . 同样的,将这段代码带到mingw下命令行下, 然后用mingw的编译器进行编译 , 你会发现编译出问题 编译器找不到POSIX C的函数的头 ,如果你将header文件人为安装好,那么链接器会出错 , 找不到对POSIX C函数的定义 . 因此验证了我总结的第一条


 


关于如何安装mingw msys 在Windows上 ,很多blog都已经说明了这里不再赘述,查看官方文档或者查看blog都可以解决这个问题






---


[Linux](https://web.archive.org/web/20210120194017/https://void-shana.moe/category/linux), [wine](https://web.archive.org/web/20210120194017/https://void-shana.moe/category/wine) [C. Linux](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210120194017/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Windows 下同步(多线程协调)的实现 的简单解释](https://web.archive.org/web/20210120194017/https://void-shana.moe/linux/windows-%e4%b8%8b%e5%90%8c%e6%ad%a5%e5%a4%9a%e7%ba%bf%e7%a8%8b%e5%8d%8f%e8%b0%83%e7%9a%84%e5%ae%9e%e7%8e%b0-%e7%9a%84%e7%ae%80%e5%8d%95%e8%a7%a3%e9%87%8a.html)
[PREVIOUS 
gdb 下调试多线程](https://web.archive.org/web/20210120194017/https://void-shana.moe/linux/gdb-%e4%b8%8b%e8%b0%83%e8%af%95%e5%a4%9a%e7%ba%bf%e7%a8%8b.html)

            