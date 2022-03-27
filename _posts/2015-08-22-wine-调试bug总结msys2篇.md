---
title: Wine 调试bug总结(MSYS2篇)
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [August 22, 2015](https://web.archive.org/web/20210120184254/https://void-shana.moe/linux/wine-%e8%b0%83%e8%af%95bug%e6%80%bb%e7%bb%93msys2%e7%af%87.html "10:22 am") 
[VOID001](https://web.archive.org/web/20210120184254/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210120184254/https://void-shana.moe/linux/wine-%e8%b0%83%e8%af%95bug%e6%80%bb%e7%bb%93msys2%e7%af%87.html#respond)





我将如何调试MSYS2中的某个开源程序的bug的过程总结下来,提供给需要的人  

1.配置好环境  

– 获得wine-staging的源码  

* clone最新的wine代码  

* clone wine-staging的代码  

* 在wine-staging下使用 patchinstall.sh脚本 将wine-staging的改动应用到wine的代码树上  

– 编译并安装wine  

* 安装编译wine需要的库文件(参见Winehq的官方资料)  

* configure并且编译安装代码  

2.利用自己配置好的环境重现bug 注意一定要下载正确版本的代码进行编译3.下载要debug的软件安装的源码并且编译成有符号表的程序 在 gdb下调试 利用step in 和 next 等命令 ,找出bug发生的地方(就是出现fault 的那个点)  

-在MSYS2下git clone不可以用 因此需在Linux下进行某些代码的clone 然后再makepkg的时候i记得加上 –noextract 参数  

– 学习PKGBUILD的语法和写法  

– 学习makepkg 等指令 ( 如果不能通过checksum checkpgp 就用 –skippgpcheck 和 –skipchecksums 跳过)  

4.阅读该程序的源代码 , 从bug发生的点向上找, 找出问题所在 , 定位到函数内, ( 如果bug是因为SIGSEGV , 那么找出哪个变量的访问发生了SIGSEGV, 并且确定出何时这个变量不能被access了)  

5.通过搜集到的信息写出POSIX C 的测试样例 重现此bug (此时应该结合Google man page 以及各种资料 找出你要使用的函数的用法,以及作用)  

7.进一步研究自己写好的测试样例, 用 RELAY日志把程序运行时的系统日志收集到  

–  根据要记录的日志的模块的不同 WINEDEBUG后加不同的参数 , 不过所有的都要 WINEDEBUG=+relay,+tid,+server , 而且要注意 server加上后 ,先执行  

wineserver -k 然后再启动你要调试的程序 ,server的日志只有在第一个wine的进程启动时,启动server才会记录 , 关于这些参数的意义 +relay表示输出relay日志 , +tid表示打印日志的时候带有线程号 , +server表示i记录下server的详细日志  

– 得到日志之后 需要对日志进行处理,才能获得有用的信息 ,处理方法如下  

* 先grep “Starting process” (大小写自己确认一下) 然后找到你运行的程序的名字 ,然后找到进程号  

* grep那个进程号,将结果存下来  

*  根据你修复的bug不同 ,可能有不同的特点 ,比如特殊的参数 或者特殊的字符串(如打开的文件的名字) 找到与这个bug关系密切的系统函数 , 大致理解这个bug是由什么样的系统调用产生的.  

* 关于如何读relay的日志 , 参考Wine -Developer manual  

* 将关键的行提取出来,形成一个新的日志 ,并且整理出调用关系 .  

6.进一步将调用的细节确定下来  

– 编译安装MSYS2-runtime的代码,要有 -g 参数 (如果编译完成不了的话,甚至源码无法get到的话 ,请看第三步)  

* 编译安装的几个细节 ,首先PKGBUILD里的options一定要有debug, runtime的代码库要自己先clone好 同时要注意使用noextract , 编译好pkg文件之后 将两个.pkg.tar.xz文件一起安装 用 pacman -U .pkg.tar.xz 即可,也可以到PKG文件（名称记不清楚）里修改，把git的地址修改成自己本地的源码


* 安装好之后需要重启一下 MSYS2 ,然后 对程序进行单步调试, step in 到MSYS2-runtime的代码 , 找出和这个bug有关的系统调用,然后 整理出来整个系统调用的详细脉络,包括每次调用的参数  

(这里要注意一点: 如果某个参数是一个指针 ,要记录下来这个指针指向的内容的各个参数 , 而不是仅仅记录这个地址)  

7.写出WinAPI版本的测试用例
* 根据6收集到的足够的信息 , 然后可以开始写样例了~
* 根据收集到的信息写好样例( 最好将所有16进制的数都写成英文宏定义的形式)
* 在Windows上和Linux上进行样例的测试 . 检测一致性






---


[Linux](https://web.archive.org/web/20210120184254/https://void-shana.moe/category/linux), [wine](https://web.archive.org/web/20210120184254/https://void-shana.moe/category/wine) [C. Linux](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210120184254/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
CSAPP Linking 学习总结](https://web.archive.org/web/20210120184254/https://void-shana.moe/uncategorized/csapp-linking-%e5%ad%a6%e4%b9%a0%e6%80%bb%e7%bb%93.html)
[PREVIOUS 
CF 543A Writing Code 完全背包问题](https://web.archive.org/web/20210120184254/https://void-shana.moe/acmalgo/cf-543a-writing-code-%e5%ae%8c%e5%85%a8%e8%83%8c%e5%8c%85%e9%97%ae%e9%a2%98.html)

            