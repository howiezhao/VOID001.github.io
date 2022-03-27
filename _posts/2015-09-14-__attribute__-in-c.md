---
title: \_\_attribute\_\_ in C
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [September 14, 2015](https://web.archive.org/web/20210120184958/https://void-shana.moe/wine/__attribute__-in-c.html "8:49 pm") 
[VOID001](https://web.archive.org/web/20210120184958/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210120184958/https://void-shana.moe/wine/__attribute__-in-c.html#respond)





`__atrribute__` in C
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


Take a look at this code for example (in Cygwin master branch /winsup/cygwin/winsup.h)



```
#include<stdio.h> 


\_\_attribute\_\_((constructor)) void fun1()
{ 
    printf("Hello "); 
} 
\_\_attribute\_\_((destructor)) void fun2() 
{ 
    printf("Worldn"); 
} 
int main(void) 
{
    return 0; 
}
```

Then gcc xxx.c and then run the code it will print out “Hello World” ~


So why it works?


The code with constructor attribute run before main function and code with destructor attr run after main function


The `__attribute__` can do more than above


Another usage of `__attribute__` is to customize the section of a certain code , take a look at this example:



```
 \_\_attributes\_\_((section(".my\_custom\_section"))) int main(void) { return 0; }
```

Then compile the code and run **readelf** tool to check the section in the code:  

`$ readelf -S a.out`


You can see a section called my\_custom\_section and with AX acess(alloc + executable)


This proves that we can use `__attribute__` to customize the section






---


[wine](https://web.archive.org/web/20210120184958/https://void-shana.moe/category/wine), [计算机系统原理](https://web.archive.org/web/20210120184958/https://void-shana.moe/category/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%b3%bb%e7%bb%9f%e5%8e%9f%e7%90%86) [C. Linux](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210120184958/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
计算机网络的体系结构(Computer Network Architecture)](https://web.archive.org/web/20210120184958/https://void-shana.moe/%e5%9f%ba%e7%a1%80%e7%bd%91%e7%bb%9c%e7%9f%a5%e8%af%86/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%bd%91%e7%bb%9c%e7%9a%84%e4%bd%93%e7%b3%bb%e7%bb%93%e6%9e%84computer-network-architecture.html)
[PREVIOUS 
Linking](https://web.archive.org/web/20210120184958/https://void-shana.moe/linux/linking.html)

            