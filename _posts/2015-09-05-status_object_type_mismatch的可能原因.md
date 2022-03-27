---
title: STATUS\_OBJECT\_TYPE\_MISMATCH的可能原因
---
===================================



#####  [September 5, 2015](https://web.archive.org/web/20210514134552/https://void-shana.moe/wine/status_object_type_mismatch%e7%9a%84%e5%8f%af%e8%83%bd%e5%8e%9f%e5%9b%a0.html "12:07 pm") 
[VOID001](https://web.archive.org/web/20210514134552/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210514134552/https://void-shana.moe/wine/status_object_type_mismatch%e7%9a%84%e5%8f%af%e8%83%bd%e5%8e%9f%e5%9b%a0.html#respond)





可能你创建的是一个内核Object而不是文件系统的Object , 但是你给内核Object赋值的Attributes却是文件系统的ObjectAttr ,因而创建这个Object的时候就会失败






---


[wine](https://web.archive.org/web/20210514134552/https://void-shana.moe/category/wine) [C. Linux](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210514134552/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Linking](https://web.archive.org/web/20210514134552/https://void-shana.moe/linux/linking.html)
[PREVIOUS 
Windows 下同步(多线程协调)的实现 的简单解释](https://web.archive.org/web/20210514134552/https://void-shana.moe/linux/windows-%e4%b8%8b%e5%90%8c%e6%ad%a5%e5%a4%9a%e7%ba%bf%e7%a8%8b%e5%8d%8f%e8%b0%83%e7%9a%84%e5%ae%9e%e7%8e%b0-%e7%9a%84%e7%ae%80%e5%8d%95%e8%a7%a3%e9%87%8a.html)

            