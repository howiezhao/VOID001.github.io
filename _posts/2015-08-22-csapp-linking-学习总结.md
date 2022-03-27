---
title: CSAPP Linking 学习总结
---
==================



#####  [August 22, 2015](https://web.archive.org/web/20201022003148/https://void-shana.moe/uncategorized/csapp-linking-%e5%ad%a6%e4%b9%a0%e6%80%bb%e7%bb%93.html "10:39 pm") 
[VOID001](https://web.archive.org/web/20201022003148/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201022003148/https://void-shana.moe/uncategorized/csapp-linking-%e5%ad%a6%e4%b9%a0%e6%80%bb%e7%bb%93.html#respond)





1.一个C语言程序是如何被编译成为可执行程序的


* 首先 通过C preprocessor 将C语言代码中包括的头文件都写入代码 生产 main.i文件
* 然后main.i的代码经过编译器 编译为汇编代码 main.s
* 然后main.s 的代码再经过汇编语言编译器 翻译为机器码的obj(Relocatable Object Files)文件main.o
* 这时候编译器的任务已经完成了 ,下一步是由链接器,将main.o,以及main.o依赖的所有obj文件 链接成为一个可执行文件, 生成了 a.out文件(静态链接)


2.什么是静态链接(由于很多词语涉及专业术语,专业术语用英语描述)


* 链接的过程分为两个子过程: Symbol Resoultion 和 Relocation. 下面概括一下两个过程:


2.1 Symbol Resolution  将每个定义好的(Relocatable Object File 中的)符号分配唯一的一个名称 , 举个例子说明一下, 如果有两个C语言文件 需要被链接成一个可执行的文件, 并且在a.c里有一个全局变量 声明 int counter, 在另一个文件b.c里也有同样的一个声明, 这时将两个a.o b.o文件链接为一个可执行文件的时候,就会给两个文件的counter分配不同的名字, 保证每个符号都有唯一的一个名字与之对应


2.2 Relocation 将每个Relocatable Object File 链接到的地址(如函数调用时的地址) 修正为正确的值. 同样举例来说明,在编译如下程序的时候



```
#include"vecAdd.h"

int main(void)
{
    vec a;
    vec b;
    a.x = 2; b.x = 3; a.y = 1; b.y = -1;
    vec c = Add\_vec(a,b);
    return 0;
}
```

由于编译器只是将每个C文件编译为相应的Relocatable Object File, 因而找不到在另一个文件,比如 vecAdd.c 里定义的函数 Add\_vec的详细实现, 这时候 编译器会给这个函数一个假的入口地址 ,留在以后交给连接器处理, 如果连接器在其他的Relocatable Object File里找到了 Add\_vec的定义的话 ,就会在Relocation这一步将这个函数的地址修改为正确的入口地址, 如果没有找到的话,就会报link error. 这就是Relocation的简单介绍.


3.Object Files 的种类:


ObjectFile 有三种 1. Relocatable Object File 2. Executable object file 3. Shared Object File , 其中第三个是与动态链接有关的一种Object File . 第二个就是我们俗知的可执行文件 , 另外根据文件格式不同 可以将ObjectFile分为 COFF, PE ,ELF, ELF是linux 上常见的Object File 格式.


4.Relocatable Object Files 详细说明


不论是文件 还是Object 还是程序 在计算机里存储都是以01形式存储的,都可以看作是文件 ,只不过由于规划的格式不同,解析的方式不同 ,才使得他们有的成为了文件有的成为了程序, Relocatable Object File 就可以看成有着特殊结构文件, 这个文件提供给了链接器在将Objects链接为可执行程序的需要的全部信息.


下面画出一个图来表示一下 Relocatable Object Files 的结构


2016.5.9 下文更新, 关于ELF文件结构可以参考这个文章


[ELF 文件格式初步探索](https://web.archive.org/web/20201022003148/http://120.27.97.96/?p=801)






---


[Uncategorized](https://web.archive.org/web/20201022003148/https://void-shana.moe/category/uncategorized) [C. Linux](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201022003148/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
gdb 下调试多线程](https://web.archive.org/web/20201022003148/https://void-shana.moe/linux/gdb-%e4%b8%8b%e8%b0%83%e8%af%95%e5%a4%9a%e7%ba%bf%e7%a8%8b.html)
[PREVIOUS 
Wine 调试bug总结(MSYS2篇)](https://web.archive.org/web/20201022003148/https://void-shana.moe/linux/wine-%e8%b0%83%e8%af%95bug%e6%80%bb%e7%bb%93msys2%e7%af%87.html)

            