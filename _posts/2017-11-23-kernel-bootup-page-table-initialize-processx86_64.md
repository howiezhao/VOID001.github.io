---
title: Kernel Bootup Page Table Initialize Process(x86\_64)
---
====================================================



#####  [November 23, 2017](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html "2:21 am") 
[VOID001](https://web.archive.org/web/20210514134902/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [3 comments](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html#comments)





This article will provide detailed information about the kernel bootup page table setup.


In a brief view, the kernel setup page table in three steps:


1. Setup the 4GB identity mapping
2. Setup 64bit mode page table early\_top\_pgt
3. Setup 64bit mode page table init\_top\_pgt


The last two steps are both higher mapping: Map the 512MB physical address to virtual address 0xffff80000000 – 0xffff80000000 + 512MB.


Next, we will talk about the details. We will use the 4.14 version code to explain the process.


*You need to know the IA32e paging mechanism and relocation to read the article. The Intel manual has a good explaination of IA32e paging*


[https://github.com/torvalds/linux/blob/v4.14/arch/x86/boot/compressed/head\_64.S](https://web.archive.org/web/20210514134902/https://github.com/torvalds/linux/blob/v4.14/arch/x86/boot/compressed/head_64.S)


Before decompression
--------------------


When the kernel is being loaded, it is either decompressed by a third-party bootloader like GRUB2 or by the kernel itself. Now we will talk about the second condition. The code started from arch/x86/boot/header.S . It is in 16bit real mode at the time. Then in code arch/x86/boot/compressed/head\_64.S  We setup the first page table in 32bit mode. We need this page table to take us to do take us to 64bit mode.


The following code is the set-up process



```
/*
 * Prepare for entering 64 bit mode
 */

	/* Load new GDT with the 64bit segments using 32bit descriptor */
	addl	%ebp, gdt+2(%ebp)
	lgdt	gdt(%ebp)

	/* Enable PAE mode */
	movl	%cr4, %eax
	orl	$X86\_CR4\_PAE, %eax
	movl	%eax, %cr4

 /*
  * Build early 4G boot pagetable
  */
	/*
	 * If SEV is active then set the encryption mask in the page tables.
	 * This will insure that when the kernel is copied and decompressed
	 * it will be done so encrypted.
	 */
	call	get\_sev\_encryption\_bit
	xorl	%edx, %edx
	testl	%eax, %eax
	jz	1f
	subl	$32, %eax	/* Encryption bit is always above bit 31 */
	bts	%eax, %edx	/* Set encryption mask for page tables */
1:

	/* Initialize Page tables to 0 */
	leal	pgtable(%ebx), %edi
	xorl	%eax, %eax
	movl	$(BOOT\_INIT\_PGT\_SIZE/4), %ecx
	rep	stosl

	/* Build Level 4 */
	leal	pgtable + 0(%ebx), %edi
	leal	0x1007 (%edi), %eax
	movl	%eax, 0(%edi)
	addl	%edx, 4(%edi)

	/* Build Level 3 */
	leal	pgtable + 0x1000(%ebx), %edi
	leal	0x1007(%edi), %eax
	movl	$4, %ecx
1:	movl	%eax, 0x00(%edi)
	addl	%edx, 0x04(%edi)
	addl	$0x00001000, %eax
	addl	$8, %edi
	decl	%ecx
	jnz	1b

	/* Build Level 2 */
	leal	pgtable + 0x2000(%ebx), %edi
	movl	$0x00000183, %eax
	movl	$2048, %ecx
1:	movl	%eax, 0(%edi)
	addl	%edx, 4(%edi)
	addl	$0x00200000, %eax
	addl	$8, %edi
	decl	%ecx
	jnz	1b

	/* Enable the boot page tables */
	leal	pgtable(%ebx), %eax
	movl	%eax, %cr3
```

Notice that from the comment above. %ebx contain the address where we move kernel to make a safe decompression. Which means we should treat %ebx as an offset to the compiled binary. The compiled binary start at 0. So we fix-up the difference to reach the real physical address.



```
	/* Build Level 4 */
	leal	pgtable + 0(%ebx), %edi
	leal	0x1007 (%edi), %eax
	movl	%eax, 0(%edi)
	addl	%edx, 4(%edi)
```

The above code setup Top level page directory. This only set the lowest page directory entry to (1007 + pgtable). This is a pointer to the next level page table. And next level page table start at 0x1000 + pgtable. The last line adds %edx to 4+%edi will set encryption masks if SEV is active. Currently, we can omit this line.


Then we look at the next level.



```
	/* Build Level 3 */
	leal	pgtable + 0x1000(%ebx), %edi
	leal	0x1007(%edi), %eax
	movl	$4, %ecx
1:	movl	%eax, 0x00(%edi)
	addl	%edx, 0x04(%edi)
	addl	$0x00001000, %eax
	addl	$8, %edi
	decl	%ecx
	jnz	1b
```

Here, we can see we set up four entries. and each entry point to another page directory.



```
	/* Build Level 2 */
	leal	pgtable + 0x2000(%ebx), %edi
	movl	$0x00000183, %eax
	movl	$2048, %ecx
1:	movl	%eax, 0(%edi)
	addl	%edx, 4(%edi)
	addl	$0x00200000, %eax
	addl	$8, %edi
	decl	%ecx
	jnz	1b
```

This is the last level of page directory, these entry will point to a physical page frame directly. Now let’s take a look at the code. It sets up 2048 entries. Each entry with a Page Flag R/W = 1 U/S = 0 PS = 1. This means the page is read / write by kernel only and its size is 2MB. Each PTE(Page Table Entry) is a 8 Byte block data. So one page can contain at most 512 entries. Here kernel setup 4 pages of Level 2 Page Directory. The following image show the current page table structure.


In total we have 2048 * 2MB = 4GB physical address, identity mapped to 0 – 4GB linear address.


![](https://web.archive.org/web/20210514134902im_/https://void-shana.moe/wp-content/uploads/2017/11/aaa-e1511368338472-1024x768.jpg)


 


Then we use a long return to switch to 64bit mode.


![](https://web.archive.org/web/20210514134902im_/https://void-shana.moe/wp-content/uploads/2017/11/Screenshot_20171123_003421-1024x503.png)


Kernel push the startup\_64 and CS register to stack, then perform a long return to enter 64bit mode.  And then after copy the compressed kernel, we jump to symbol relocated



```
/*
 * Jump to the relocated address.
 */
	leaq	relocated(%rbx), %rax
	jmp	*%rax
```

In the relocated code, we do the kernel decompression.



```
/*
 * Do the extraction, and jump to the new kernel..
 */
	pushq	%rsi			/* Save the real mode argument */
	movq	%rsi, %rdi		/* real mode address */
	leaq	boot\_heap(%rip), %rsi	/* malloc area for uncompression */
	leaq	input\_data(%rip), %rdx  /* input\_data */
	movl	$z\_input\_len, %ecx	/* input\_len */
	movq	%rbp, %r8		/* output target address */
	movq	$z\_output\_len, %r9	/* decompressed length, end of relocs */
	call	extract\_kernel		/* returns kernel location in %rax */
	popq	%rsi
```

The decompressed kernel is compiled at high address(we take ffffffff81000000 for example). But now we don’t have the correct page table to do the mapping. Fortunately, the **extract\_kernel** function returns the physical address of the decompressed kernel. (Which is %ebp, equals to %ebx). After decompression, %rax contains the kernel physical start address. We jump there to perform the further setup.


Start execution in vmlinux
--------------------------


We now arrived at arch/x86/kernel/head\_64.S. Before we continue, we must notice two things first.


* After decompression, the kernel is placed at physical address %rbp (If we do not set CONFIG\_RELOCATABLE it’s equal to 0x1000000  

).
* After decompression, we now in the kernel code compiled with the virtual address ffffffff81000000(as we mentioned above).


So here is a big pitfall. We cannot access ANY of the symbols in vmlinux currently. Because we only have a basic identity mapping now. But we need to visit the variables. How can we make it? The kernel uses a trick here, I will show it below



```
static void \_\_head *fixup\_pointer(void *ptr, unsigned long physaddr)
{
	return ptr - (void *)\_text + (void *)physaddr;
}

```

This function fixup the symbol virtual address to the real physical address.


**“Current Valid Addr” = “Virtual Hi Addr” – “Kernel Virtual Address Base Addr” + “%rax Extracted kernel physical address”.**


Now we continue reading the arch/x86/kernel/head\_64.S  assembly code, this is where we landed from arch/x86/compressed/head\_64.S


The enrty is startup\_64:



```
startup\_64:
	/*
	 * At this point the CPU runs in 64bit mode CS.L = 1 CS.D = 0,
	 * and someone has loaded an identity mapped page table
	 * for us.  These identity mapped page tables map all of the
	 * kernel pages and possibly all of memory.
	 *
	 * %rsi holds a physical pointer to real\_mode\_data.
	 *
	 * We come here either directly from a 64bit bootloader, or from
	 * arch/x86/boot/compressed/head\_64.S.
	 *
	 * We only come here initially at boot nothing else comes here.
	 *
	 * Since we may be loaded at an address different from what we were
	 * compiled to run at we first fixup the physical addresses in our page
	 * tables and then reload them.
	 */

	/* Set up the stack for verify\_cpu(), similar to initial\_stack below */
	leaq	(\_\_end\_init\_task - SIZEOF\_PTREGS)(%rip), %rsp

	/* Sanitize CPU configuration */
	call verify\_cpu

	/*
	 * Perform pagetable fixups. Additionally, if SME is active, encrypt
	 * the kernel and retrieve the modifier (SME encryption mask if SME
	 * is active) to be added to the initial pgdir entry that will be
	 * programmed into CR3.
	 */
	leaq	\_text(%rip), %rdi
	pushq	%rsi
	call	\_\_startup\_64
	popq	%rsi

	/* Form the CR3 value being sure to include the CR3 modifier */
	addq	$(early\_top\_pgt - \_\_START\_KERNEL\_map), %rax
	jmp 1f
```

In this article, we talk about self loading, instead of using a third party 64bit bootloader like GRUB. So as the comment said, we come here from arch/x86/boot/compressed/head\_64.S. If we config the kernel with CONFIG\_RELOCATABLE, the kernel won’t run at the place we compiled, page table fixup need to be performed. The page table is fixed in **\_\_startup\_64**



```
unsigned long \_\_head \_\_startup\_64(unsigned long physaddr,
				  struct boot\_params *bp)
{
	unsigned long load\_delta, *p;
	unsigned long pgtable\_flags;
	pgdval\_t *pgd;
	p4dval\_t *p4d;
	pudval\_t *pud;
	pmdval\_t *pmd, pmd\_entry;
	int i;
	unsigned int *next\_pgt\_ptr;

	/* Is the address too large? */
	if (physaddr >> MAX\_PHYSMEM\_BITS)
		for (;;);

	/*
	 * Compute the delta between the address I am compiled to run at
	 * and the address I am actually running at.
	 */
	load\_delta = physaddr - (unsigned long)(\_text - \_\_START\_KERNEL\_map);

	/* Is the address not 2M aligned? */
	if (load\_delta & ~PMD\_PAGE\_MASK)
		for (;;);

	/* Activate Secure Memory Encryption (SME) if supported and enabled */
	sme\_enable(bp);

	/* Include the SME encryption mask in the fixup value */
	load\_delta += sme\_get\_me\_mask();

	/* Fixup the physical addresses in the page table */

	pgd = fixup\_pointer(&early\_top\_pgt, physaddr);
	pgd[pgd\_index(\_\_START\_KERNEL\_map)] += load\_delta;

	if (IS\_ENABLED(CONFIG\_X86\_5LEVEL)) {
		p4d = fixup\_pointer(&level4\_kernel\_pgt, physaddr);
		p4d[511] += load\_delta;
	}

        /* Omit some fixup code for simplicity */

	return sme\_get\_me\_mask();
}

```

We compute the load\_delta, and fixup the early\_top\_pgt. Now we just assume we don’t configure the kernel with CONFIG\_RELOCATABLE. Then we can look at the page table built at compile time. First we look at the top level **early\_top\_pgt.**It set only the last entry point to level3 page table. which means only virtual address start with 0xff8000000000 will be valid.



```
NEXT\_PAGE(early\_top\_pgt)
	.fill	511,8,0
#ifdef CONFIG\_X86\_5LEVEL
	.quad	level4\_kernel\_pgt - \_\_START\_KERNEL\_map + \_PAGE\_TABLE\_NOENC
#else
	.quad	level3\_kernel\_pgt - \_\_START\_KERNEL\_map + \_PAGE\_TABLE\_NOENC
#endif
```

Now we look at the next level (We do not use 5 Level Paging).



```
NEXT\_PAGE(level3\_kernel\_pgt)
	.fill	L3\_START\_KERNEL,8,0
	/* (2^48-(2*1024*1024*1024)-((2^39)*511))/(2^30) = 510 */
	.quad	level2\_kernel\_pgt - \_\_START\_KERNEL\_map + \_KERNPG\_TABLE\_NOENC
	.quad	level2\_fixmap\_pgt - \_\_START\_KERNEL\_map + \_PAGE\_TABLE\_NOENC
```

This level we have two entries, one for kernel address space. One for fixmap address space, fixmap address space is used for IO mapping, DMA, etc. Now we just look at the fixmap address space. It’s at index 510. in binary mode 0b111111110. Combine with the top level we get a smaller linear address space. Only address start from 0xffff80000000 is valid.


Then it’s the last level page directory. **level2\_kernel\_pgt**



```
NEXT\_PAGE(level2\_kernel\_pgt)
	/*
	 * 512 MB kernel mapping. We spend a full page on this pagetable
	 * anyway.
	 *
	 * The kernel code+data+bss must not be bigger than that.
	 *
	 * (NOTE: at +512MB starts the module area, see MODULES\_VADDR.
	 *  If you want to increase this then increase MODULES\_VADDR
	 *  too.)
	 */
	PMDS(0, \_\_PAGE\_KERNEL\_LARGE\_EXEC,
		KERNEL\_IMAGE\_SIZE/PMD\_SIZE)
```

This level is a mapping to physical address 0 – 512MB (it maps more than that, but we only need 512MB) So we get the current mapping then.


**Linear: 0xffff80000000 – 0xffff80000000 + 512MB =====> Physical: 0 – 512MB**


You can use a gdb to print the page table and debug it in your own. Here is a simple “it works!” script for parsing the page directory entry



```
#!/usr/bin/python

import argparse

def main():
    parser = argparse.ArgumentParser(description='Page Table Entry Decoder\n Convert into human-friendly mode')
    parser.add\_argument('value', type=str)
    args = parser.parse\_args()
    value = (int(args.value, 16))
    P = value & 0x0000000000000001
    if not P:
        print("PE = 0, page not present")
        return
    else:
        print("PE = 1")

    RW = value & 0x0000000000000002
    if not RW:
        print("R/W = 0, Only Read access")
    else:
        print("R/W = 1, Read/Write access")

    US = value & 0x0000000000000004
    if not US:
        print("U/S = 0, only for kernel access")
    else:
        print("U/S = 1, user/kernel access")
    
    PS = (value & 0x0000000000000080) >> 7
    PHY = ((value >> 12) & 0x000ffffffffff) << 12

    print("PS = {:d}".format(PS))
    print("PHYADDR = {:x}".format(PHY))


if \_\_name\_\_ == '\_\_main\_\_':
    main()

```

Kernel load the early\_top\_pgt into cr3 using the following code



```
    addq	$(early\_top\_pgt - \_\_START\_KERNEL\_map), %rax
    ...
    addq	phys\_base(%rip), %rax
    movq	%rax, %cr3
```

The current page table structure is shown below:


![](https://web.archive.org/web/20210514134902im_/https://void-shana.moe/wp-content/uploads/2017/11/pasted-image-0-e1511372882463.png)


Now we are free to visit any kernel symbol without to force convert the address using **fixup\_address**or something else. We can go further to the init/main.c code.


We use a long return to get to get to x86\_64\_start\_kernel



```
	pushq	$.Lafter\_lret	# put return address on stack for unwinder
	xorq	%rbp, %rbp	# clear frame pointer
	movq	initial\_code(%rip), %rax
	pushq	$\_\_KERNEL\_CS	# set correct cs
	pushq	%rax
```

**initial\_code** here is defined as **x86\_64\_start\_kernel**.


 


Moving to init/main.c
---------------------


We are now at arch/x86/kernel/head64.c and in function **x86\_64\_start\_kernel**



```
asmlinkage \_\_visible void \_\_init x86\_64\_start\_kernel(char * real\_mode\_data)
{
	/*
	 * Build-time sanity checks on the kernel image and module
	 * area mappings. (these are purely build-time and produce no code)
	 */
	BUILD\_BUG\_ON(MODULES\_VADDR < \_\_START\_KERNEL\_map);

        /* Omit some initialization code for simplicity */

	/* set init\_top\_pgt kernel high mapping*/
	init\_top\_pgt[511] = early\_top\_pgt[511];

	x86\_64\_start\_reservations(real\_mode\_data);
}
```

We set up **init\_top\_pgt[511]** same as **early\_top\_pgt[511]**  **. init\_top\_pgt** is the final kernel page table. From **x86\_64\_start\_reservations**we get to **start\_kernel**This is a function located at init/main.c



```
asmlinkage \_\_visible void \_\_init start\_kernel(void)
{
        /* Omit some code for simplicity */

	boot\_cpu\_init();
	page\_address\_init();
	pr\_notice("%s", linux\_banner);
	setup\_arch(&command\_line);

        /* Omit some code for simplicity */

	rest\_init();
}
```

After calling **setup\_arch**, CR3 is loaded with init\_top\_pgt. Then the kernel page table will not change. I wonder if there is a change to switch kernel page table from 2MB size physical page to 4KB physical page, but it seems that the CR3 remained unchanged, and I examined the page entries, they remain unchanged, too. Even the code has executed into **rest\_init** then do\_idle


The following function is a simple debug function to output the current CR3 register since GDB cannot get the CR3 register value, I just print it out to see when it changed.



```
asmlinkage \_\_visible unsigned long shana\_debug\_cr3(void) {
    unsigned long cr3\_value = 0xffffffff;
    asm volatile("mov %%cr3, %0"
            : "=r"(cr3\_value));
    printk("shana\_debug\_cr3: %x", cr3\_value);
    return cr3\_value;
}
```

 






---


[Kernel](https://web.archive.org/web/20210514134902/https://void-shana.moe/category/kernel), [Linux](https://web.archive.org/web/20210514134902/https://void-shana.moe/category/linux) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514134902im_/https://secure.gravatar.com/avatar/5aa864692767e0d284fc73adc5c9f83d?s=50&d=identicon&r=g) **068089dy** says: 
[November 25, 2017 at 10:48 am](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html#comment-346)
不明觉厉


	1. ![](https://web.archive.org/web/20210514134902im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[November 25, 2017 at 10:57 am](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html#comment-347)
	就是 linux 内核的页表设置过程啦～ x86\_64 的


2. ![](https://web.archive.org/web/20210514134902im_/https://secure.gravatar.com/avatar/2567491c82a9e1fd9bb32a00682d83a3?s=50&d=identicon&r=g) **Theo** says: 
[May 26, 2019 at 10:11 pm](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html#comment-616)
请问说init\_top\_pgt就是最终页表，但是我在Systems.map里面看到它的地址是ffffffff83a00000，可直接手动走cr3页表里面翻译得到的地址却都是形如0xffff880003c01067这样的地址，并不是Systems.map这些地址，请问你当时的0xffff80000000 – 0xffff80000000 + 512MB这个范围是怎么得到的呢？


2. ![](https://web.archive.org/web/20210514134902im_/https://secure.gravatar.com/avatar/2567491c82a9e1fd9bb32a00682d83a3?s=50&d=identicon&r=g) **Theo** says: 
[May 26, 2019 at 10:11 pm](https://web.archive.org/web/20210514134902/https://void-shana.moe/linux/kernel-bootup-page-table-initialize-processx86_64.html#comment-616)
请问说init\_top\_pgt就是最终页表，但是我在Systems.map里面看到它的地址是ffffffff83a00000，可直接手动走cr3页表里面翻译得到的地址却都是形如0xffff880003c01067这样的地址，并不是Systems.map这些地址，请问你当时的0xffff80000000 – 0xffff80000000 + 512MB这个范围是怎么得到的呢？

            