---
title: ULK Chapter2 总结
---
===============



#####  [January 9, 2017](https://web.archive.org/web/20210614023315/https://void-shana.moe/linux/ulk-chapter2-%e6%80%bb%e7%bb%93.html "2:09 pm") 
[VOID001](https://web.archive.org/web/20210614023315/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210614023315/https://void-shana.moe/linux/ulk-chapter2-%e6%80%bb%e7%bb%93.html#respond)





本章的知识结构如下


![](https://web.archive.org/web/20210614023315im_/https://voidisprogramer.com/wp-content/uploads/2017/01/ch3-1024x496.png)


地址转换过程
------



```
--Logical Address--> [Segment Unit] --Linear Address--> [Paging Unit] --Physical Address-->



```

谈到内存地址，具体来讲有三种不同类型的地址，逻辑地址，线性地址，物理地址。


* 逻辑地址对应内存的分段
* 线性地址对应分页
* 物理地址对应到硬件芯片上的内存单元的地址


MMU通过Segment Unit与Paging Unit两个硬件电路将一个逻辑地址转为物理地址， 具体转换过程如上代码段所示


下面对整个转换过程进行概述


### 段选择符



```
 15                            0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
+                         +T+ R +
+          index          +I+ P +
+                         + + L +
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

[0-1]RPL: 请求者特权级


[2]TI: Table Indicator, 指明是从GDT还是LDT中取出段描述符(Segment Descriptor)


[3-15]index:用来指定从GDT中第index项取出Segment Descriptor


### 段描述符


![](https://web.archive.org/web/20210614023315im_/https://voidisprogramer.com/wp-content/uploads/2017/01/a_segment_descriptor.png)


对段描述符的解释参考 [[不是科普向?] RE: 从零开始的操作系统开发 第二集](https://web.archive.org/web/20210614023315/https://voidisprogramer.com/linux/%e4%b8%8d%e6%98%af%e7%a7%91%e6%99%ae%e5%90%91-re-%e4%bb%8e%e9%9b%b6%e5%bc%80%e5%a7%8b%e7%9a%84%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%bc%80%e5%8f%91-%e7%ac%ac%e4%ba%8c%e9%9b%86.html) 中相应内容即可


### Logical Addr ===> Linear Addr


逻辑地址的高16位为段选择符(Segment Selector)其余32位(或者64位)为偏移量(Offset)


流程如下


1. 检查TI确定是从GDT(TI=0)还是从LDT(TI =1)中选择段描述符
2. 从Segment Selector的index字段计算出段描述符的地址，计算方法 index * 8 (一个Segment Descriptor大小为8) 并与gdtr/ldtr中的内容相加得到Segment Descriptor
3. 把逻辑地址的offset字段与Segment Descriptor中的Base字段相加，得到Linear Address


 


而Linux中的分段只是一个兼容性考虑，分段和分页的作用是重复的，分页能以更精细的粒度对内存进行管理，因而在Linux中分页是主要的手段，Linux分段中的用户代码段，数据段，内核代码段，数据段，都是以Base = 0 ，因而Linux下的逻辑地址和线性地址是一致的，（因为Base = 0 所以 Linear Address = Base + Offset = Offset) 即逻辑地址的偏移量和线性地址的值是相等的


 


### Linear Addr ==> Physical Addr


至此我们已经得到了线性地址，下面需要将线性地址通过分页单元转为物理地址，下面以基本的分页模型对分页进行解释


80×86的常规分页结构为


页目录 页表 页框


如图


![](https://web.archive.org/web/20210614023315im_/https://voidisprogramer.com/wp-content/uploads/2017/01/PTE-1-1024x697.png)


如图，Page Directory和PageTable的结构类似，都是由一些flag bit加上Field字段，Field字段表示页框的物理地址，如果是一个PageTable的话，那么这个页框就含有一页数据，Page Directory的话，页框内含有的是一个页的PageTable(页表的大小就是一个PageTable)


 


寻址的方式如下，首先根据CR3寄存器的内容，找到PageDirectory入口的物理地址，然后加上Linear Addr中的DIRECTORY,找到对应的PDE项，并根据此内容找到对应的Page Table的物理地址，并根据Linear Addr中的TABLE找到相应的Page即页框的物理地址，最后将此物理地址加上Linear Addr的OFFSET字段,最后得到物理地址


 


而在64位系统上，如果依旧采用这种最基本的分页结构的话，那么假设页框大小为4K，所以OFFSET位依旧为12bit,然后其余的52bit可以分给PT(Page Table的简称，下同)和PD(Page Directory的简称，下同)，这样就会导致我们每个进程的页目录和页表变得非常非常多，超过256000项


因为这个原因，所以对于64位处理器的分页系统，都使用了更多级别的分页级别，如x86\_64使用48位寻址，分页为4级线性地址分级为 9 + 9 + 9 + 9 + 12 如下图所示


![](https://web.archive.org/web/20210614023315im_/https://voidisprogramer.com/wp-content/uploads/2017/01/LinuxPg-1024x654.png)


原理都是一样的，只不过分页级别变多了


以上就是Logical Address ===> Physical Address的转换的大致流程


 


缓存技术
----


因为CPU Register的读取和内存的读取速度相差甚大，为了缩小这个差距，避免CPU等待过长的时间，使用缓存技术来将内存中的部分数据缓存在高速静态RAM(SRAM)里，即为高速缓存技术(Hardware Cache)


此外，将Logical Address转换为Physical Address也需要多次进行内存的读取（查找各级页表），为了加速此过程，引入了转换后援缓冲器Translation Lookaside Buffer(TLB)技术


### Hardware Cache


对高速缓存的原理描述超出本文的范围，这里仅对高速缓存的读写方式进行一定说明:


在更新内存数据的时候，是同时更新高速缓存和内存的数据，还是仅仅更新缓存的数据，根据这个有两种不同的写策略


* 通写(write-through): 既写RAM也写缓存
* 回写(write-back): 只写缓存，只有当需要FLUSH的时候才更新RAM


操作系统可以针对不同的页框配置不同的告诉缓存管理策略，通过PCD位和PWT位


* PCD表示是否对此页框启用告诉缓存
* PWT表示是否采用write-through的策略


在Linux下，对全部的页框都启用告诉缓存，并且都采用write-back策略


### TLB（快表）


TLB的作用是将Logical Address对应的Linear Address缓存起来，以加速对内存的访问


但是这个缓存是有失效期的，最明显的一个失效就是当PD都被替换了的时候，也就是cr3的寄存器内容更新了，硬件上，当cr3更新的时候，TLB的内容会自动失效


如何能够更好的利用TLB来加速访问是对Linux性能影响十分重要的一部分，为了避免多处理器系统上无用的TLB刷新，内核使用一种叫做Lazy TLB的技术，**关于Lazy TLB技术的具体实现之后补充**


不同的分页方式
-------


上述已经对常规分页进行举例了，下面对不同的几种分页方式进行简单整理


### 扩展分页


80×86模型的另一种分页模式，页框大小为4MB而不是4KB，用于把大段连续的线性地址转换成相应的物理地址，因为没有了PT只有PD，节省了内存，同时PT—>Phy而不是PT–>PD–>Phy，少了一级转换效率也相应提高,


### 物理地址扩展(PAE)分页机制


早些时候，内存容量都很少，而当需要大内存(>4G)的服务器&程序出现的时候，上述的寻址方式就不能使用了，因而Intel将内存地址位数从32 –>36位（即将引脚从32个扩展到36个) 而之前的所有的地址转换都是从32位Linear Address转换到32位Physical Address的，需要有一种新的转换机制，将32bit linear addr->36bit phy addr，这种机制即为PAE机制，详情可以参考Intel手册 Vol3A相应的内容


Linux中的实现
---------


对于Linux的具体处理将在之后在整理


 


 


 






---


[C](https://web.archive.org/web/20210614023315/https://void-shana.moe/category/linux/c), [Kernel](https://web.archive.org/web/20210614023315/https://void-shana.moe/category/kernel), [Kernel](https://web.archive.org/web/20210614023315/https://void-shana.moe/category/linux/kernel-linux), [Linux](https://web.archive.org/web/20210614023315/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210614023315/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Linux Kernel Development Resources](https://web.archive.org/web/20210614023315/https://void-shana.moe/linux/linux-kernel-development-resources.html)
[PREVIOUS 
将内核编译到自定义的目录下](https://web.archive.org/web/20210614023315/https://void-shana.moe/linux/build-kernel-to-a-customized-directory.html)

            