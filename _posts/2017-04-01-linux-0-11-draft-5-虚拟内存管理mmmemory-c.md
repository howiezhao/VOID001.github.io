---
title: (Linux 0.11) Draft 5 虚拟内存管理(mm/memory.c)
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [April 1, 2017](https://web.archive.org/web/20201020195949/https://void-shana.moe/linux/linux-0-11-draft-5-%e8%99%9a%e6%8b%9f%e5%86%85%e5%ad%98%e7%ae%a1%e7%90%86mmmemory-c.html "4:58 pm") 
[VOID001](https://web.archive.org/web/20201020195949/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201020195949/https://void-shana.moe/linux/linux-0-11-draft-5-%e8%99%9a%e6%8b%9f%e5%86%85%e5%ad%98%e7%ae%a1%e7%90%86mmmemory-c.html#respond)





Overview
--------


本文介绍 32 位保护模式下的内存分页， 以及 linux0.11 对物理内存的管理。


### 意义


既然要使用内存分页，那么就一定有他的优势所在。我们首先来探讨一下使用内存分页的意义， 分页在于为进程提供了虚拟地址空间，通过提供的这个虚拟地址空间，我们可以做到下面的这些:


1. 精细的权限控制粒度: 内存页大小以 4KB 为一页单位（也可以是4MB, 这里不讨论这种情况)， 可以针对每一个内存页设置该4K空间的访问权限。相比分段的粗粒度精细了很多，目前的分段功能只是一个兼容性考虑，分段都是将整个0 – 最大内存空间直接映射为一个段;
2. 可以使得进程独享地址空间: 通过切换CR3寄存器的内容，可以实现多个进程占用同一个虚拟地址空间。
3. 有效的解决了进程对大块连续内存的请求: 这里用一个例子来说明一下。


例子是这样的： 如果我们有16MB的物理内存，现在有进程A B C分别占用5MB, 2MB, 3MB内存， 且不共享内存。然后现在进程B退出了，我们还剩余8MB内存，可是如果不使用分页的话，内存连续分配， 我们现在没有办法分配出一个连续的8MB的空间了，只能分配最多最多6MB连续的内存空间， 如下图所示。


![](https://web.archive.org/web/20201020195949im_/https://void-shana.moe/wp-content/uploads/2017/04/Document_2017-04-01_16-52-34-e1491037088794.jpeg)


而现在存在分页，虚拟地址，我们完全可以分配连续的8MB的虚拟地址空间，其中2MB映射到物理内存的中间那个2MB空隙，其余6MB映射到最后剩余的6MB。


由此可见，分页是十分必要的，那么下面就来介绍一下如何实现分页&虚拟内存管理


### 地址转换过程


我们先来看一下虚拟地址是如何转换为物理地址的 完整的转换过程为


`虚拟地址 --> (GDT) --> 线性地址 --> (Page Table) --> 物理地址`


关于VirtualAddr -> LinearAddr 的过程我在[之前的文章中](https://web.archive.org/web/20201020195949/https://void-shana.moe/linux/%e4%b8%8d%e6%98%af%e7%a7%91%e6%99%ae%e5%90%91-re-%e4%bb%8e%e9%9b%b6%e5%bc%80%e5%a7%8b%e7%9a%84%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%bc%80%e5%8f%91-%e7%ac%ac%e4%ba%8c%e9%9b%86.html)有过介绍，这里就不做说明，想要跳过那个文章的读者可以简单的理解为目前的虚拟地址和线性地址值是相同的，因而我们直接从线性地址出发。首先来看下图


![](https://web.archive.org/web/20201020195949im_/https://void-shana.moe/wp-content/uploads/2017/04/Spectacle.w14448.png)


 


（图片来源：Intel® 64 and IA-32 Architectures Software Developer’s Manual）


线性地址为32位的地址， 其中高十位为 Page Directory，中间十位为 Page Table，最后十二位是 Offset。过程如下：


1. 根据 CR3 寄存器（[忘记了寄存器的功能的话戳这里](https://web.archive.org/web/20201020195949/https://void-shana.moe/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%b3%bb%e7%bb%9f%e5%8e%9f%e7%90%86/linux-0-11-draft-2-80386-system-architecture.html)）找到了页目录表的地址。
2. 根据 Linear Address 的高十位与页表目录地址相加，找到了对应的页目录项(PDE)。
3. 页目录项内存有该页目录的起始地址，将此地址与 Linear Address 21 – 12 位（即中间十位）相加，找到对应的页表项(PTE)
4. 根据页表项中记载的物理页地址，找到对应的物理页起始地址，再与 Offset 相加，对应到实际的物理地址。


通过上面的过程就实现了线性地址到物理地址的转换，可以看出，页表和页目录在这个过程中扮演了十分重要的角色，下面就来介绍一下他们的结构。


### 结构


一个页目录项（PTE）和页表项（PDE）均占用 4Bytes 空间


![](https://web.archive.org/web/20201020195949im_/https://void-shana.moe/wp-content/uploads/2017/04/Spectacle.b14448-1024x78.png)


（图片来源：Intel® 64 and IA-32 Architectures Software Developer’s Manual）


如上图所示，这是一个 Page Table Entry (页面大小为4K)


首先我们看 Bit 0 ,这是 Present bit，只有当这个 Bit 为 1 的时候此项才表示一个 PTE(或者PDE)，否则就是一个无效的表项。


Bit 1 表示这个页面的读写权限，当这个位被置 1 的时候 表示可读可写， 为 0 则表示只读 ，**不过这里要注意一点，对于 Ring0 且 CR0 的 WP 位为 0 情况来说，Ring0 可以对只读页面进行写入，如果想要让 Ring 0 也不可以写入只读页面，需要将 CR0 的 WP 位置1**。


Bit 2 表示这个页面是系统页面还是用户页面，如果是系统页面则用户态无法使用此页面。


后续的标志位暂时不介绍。我们来看高20位， 这里表示了物理页的起始地址，如果这是一个页目录项而不是页表项，则这里表示的是页目录的起始物理地址


### 转换缓存


通过上述的过程我们也可以看出， Page Transform 的过程需要多次读取内存，会映像执行效率，为了解决这个问题，CPU对转换信息进行了缓存，如  TLB, PCID  等技术，这些不在本文进行介绍，这里要说明的是，因为缓存的存在，导致如果更新了已被缓存的页表项的信息（如修改FLAG， 或者物理地址），我们需要使缓存失效，具体的做法就是重新装载一次 CR3 寄存器。


### 示例


说了这么多理论知识，我们来进行几个实际的操作看看。我们基于 linux0.11 的模型来进行下面四个实验(毕设结束后会放出git repo，目前暂无）


* 使得某个特定的线性地址无效 （下文有介绍）
* 将线性地址映射到给定的物理地址 （提示，查看 mm.c put\_page 函数）
* 修改该页的权限为只读并验证  （修改 FLAG 的另一个实验）
* 给出当前线性地址所在页的详细信息 （打印出必要的FLAG信息，以及该线性地址对应的物理页地址）


#### disable\_linear 实现


我们设计了这样的函数原型:


`void disable_linear(unsigned long addr)`


这个函数内我们需要对线性地址进行禁用， 根据上文中所介绍的，使得线性地址无效，实际上就是使得那个对应的页表项无效，因而我们需要通过线性地址对应到其页表项，并对该页表项的 Bit 0 清零。如何 Disable 与 Reload TLB 留给大家自行实现



```
void disable\_linear(unsigned long addr) {
    unsigned long *pte = linear\_to\_pte(addr);
    // Disable it
    //code

    // Then RELOAD the TLB
    // code

    return ;
}
```

linear\_to\_pte这个函数的实现过程实际上就是地址转换过程的前三步， 实现的时候要注意移位操作不要出问题，其他的没有什么难点


这里给出一个实现



```
// Helper function to convert linear address to PTE
// return physical address on success
// return NULL(0) on failed
unsigned long *linear\_to\_pte(unsigned long addr) {
    // get the Page Directory Entry first
    // This variable will be used to refer to pte later
    // (for saving memory XD)
    unsigned long *pde = (unsigned long *)((addr >> 20) & 0xffc);
    // Page dir not exist
    // Or the address is not inside the page table address range(<=4KB)
    if(!(*pde & 1) || pde > 0x1000) {
        return 0;
    }
    // Now it is page table address :P
    pde = (unsigned long *)(*pde & 0xfffff000);
    // Page table address + page\_table index = PTE
    //
    // Remember: x >> 12 & 0x3ff != x >> 10 & 0xffc (后者比前者二进制末尾多了两个0)
    return pde + ((addr >> 12) & 0x3ff);
}
```

上面的代码对页目录项是否存在，以及是否是页目录项进行了判断，失败将返回 0 。


下面我们需要对函数的正确性进行验证，由于此代码版本还没有补充缺页异常的处理代码，因而当访问失效的虚拟内存地址时，会引发Page Fault -> Double Fault -> Triple Fault最后导致我们的OS不断重启，我们来验证一下，验证用代码如下：



```
int mmtest\_main(void) {
    int i = 0;
    printk("Running Memory function tests\n");
    printk("1. Make Linear Address 0xdad233 unavailable\n");

    disable\_linear(0xdad233);
   
    unsigned long *x = 0xdad233;
    *x = 0x23333333;

    // Crash!!!!!!
}

```

运行之后， 看到 qemu / bochs 的确在不停的 reboot , 并且可以通过 bochs 的 GUI 界面看到我们的页表由原有的16MB一致映射变为了中间缺少 0xdad000 页。


下面以实验结果截图结束本文


![](https://web.archive.org/web/20201020195949im_/https://void-shana.moe/wp-content/uploads/2017/04/Spectacle.G14448.png)






---


[C](https://web.archive.org/web/20201020195949/https://void-shana.moe/category/linux/c), [Linux](https://web.archive.org/web/20201020195949/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20201020195949/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020195949/https://void-shana.moe/tag/kernel) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
[Linux 0.11] Draft 6 IA32架构下多任务的硬件支持](https://web.archive.org/web/20201020195949/https://void-shana.moe/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%b3%bb%e7%bb%9f%e5%8e%9f%e7%90%86/linux-0-11-draft-6-ia32%e6%9e%b6%e6%9e%84%e4%b8%8b%e5%a4%9a%e4%bb%bb%e5%8a%a1%e7%9a%84%e7%a1%ac%e4%bb%b6%e6%94%af%e6%8c%81.html)
[PREVIOUS 
[Linux 0.11] Draft 4 GCC Assembly](https://web.archive.org/web/20201020195949/https://void-shana.moe/linux/draft-4-gcc-assembly.html)

            