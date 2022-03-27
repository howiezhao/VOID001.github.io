---
title: setcap wine 遇到的问题解决
---
===================



#####  [November 11, 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/uncategorized/setcap-wine-%e9%81%87%e5%88%b0%e7%9a%84%e9%97%ae%e9%a2%98%e8%a7%a3%e5%86%b3.html "1:02 am") 
[VOID001](https://web.archive.org/web/20201020200037/https://void-shana.moe/author/void001 "View all posts by VOID001")





之后为了让普通用户也能通过wine访问到网络接口,需要给wine的相关程序(wine-preloader 和 wineserver wine) 一些特权


 


参考  man capabilities 和 setcap


* 首先需要给CAP\_NET\_RAW 权限 这个要给到 wine-preloader 上,不过在设置完这个权限之后,执行 wine XXX.exe 会报错


wine: error while loading shared libraries: libwine.so.1: cannot open shared object file: No such file or directory


这是因为特权模式的程序运行的时候是不会检测到相对路径的lib库的, 具体参考文章如下:http://stackoverflow.com/q/6493846/296473  解决方法在文中已有了 就不赘述了


* 上述问题解决后,运行wine 会提示  Internal errors- Invalid parameters  , 而且winedbg不可以attach到进程, 这个问题的解决方法是给wineserver(不是wine-preloader)CAP\_SYS\_PTRACE权限


 


 






---


[Linux](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux), [Threads and Process](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/threads-and-process), [Uncategorized](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/uncategorized), [wine](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/wine), [计算机系统原理](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%b3%bb%e7%bb%9f%e5%8e%9f%e7%90%86) [C. Linux](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
lightdm 配置](https://web.archive.org/web/20201020200037/https://void-shana.moe/uncategorized/lightdm-%e9%85%8d%e7%bd%ae.html)
[PREVIOUS 
[被迫参加的hackathon] 在线编译系统开发小记](https://web.archive.org/web/20201020200037/https://void-shana.moe/webdev/%e8%a2%ab%e8%bf%ab%e5%8f%82%e5%8a%a0%e7%9a%84%e9%bb%91%e9%a9%ac-%e5%9c%a8%e7%ba%bf%e7%bc%96%e8%af%91%e7%b3%bb%e7%bb%9f%e5%bc%80%e5%8f%91%e5%b0%8f%e8%ae%b0.html)
Comments are closed. 
#### 统计信息nanodesu~~~
 当前在线人数 (｡･ω･)ﾉﾞ **0**  
修罗酱真诚欢迎更多的朋友来我的WOWO做客哦~
Search for:
Search
  #### Recent Posts
 * [Customize Your Zsh Prompt](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/customize-your-zsh-prompt.html)
* [将 vim 作为日常笔记本使用](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/zh-taking-notes-with-vim.html)
* [Running Arch Linux with customized kernel in QEMU](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html)
* [proxychains-ng 原理解析](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/proxychains-ng.html)
* [闭关三月 博客之后更新](https://web.archive.org/web/20201020200037/https://void-shana.moe/uncategorized/%e9%97%ad%e5%85%b3%e4%b8%89%e6%9c%88-%e5%8d%9a%e5%ae%a2%e4%b9%8b%e5%90%8e%e6%9b%b4%e6%96%b0.html)
#### Recent Comments
* VOID001 on [将 vim 作为日常笔记本使用](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-1217)
* VOID001 on [留言板](https://web.archive.org/web/20201020200037/https://void-shana.moe/others/%e7%95%99%e8%a8%80%e6%9d%bf#comment-1216)
* Jim Yu on [留言板](https://web.archive.org/web/20201020200037/https://void-shana.moe/others/%e7%95%99%e8%a8%80%e6%9d%bf#comment-1213)
* VOID001 on [VOID](https://web.archive.org/web/20201020200037/https://void-shana.moe/void#comment-1123)
* muou333000 on [将 vim 作为日常笔记本使用](https://web.archive.org/web/20201020200037/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-1122)
#### 友情链接
 [YongHao Hu's Blog](https://web.archive.org/web/20201020200037/https://yonghaowu.github.io/)  
[亿臻的知乎主页(非个人博客站点)](https://web.archive.org/web/20201020200037/https://www.zhihu.com/people/qinlibo_nlp)  
[yungkcx's Blog](https://web.archive.org/web/20201020200037/https://yungkcx.github.io/)  
[Asm Def](https://web.archive.org/web/20201020200037/https://cnblogs.com/Asm-Definer)  
[Five Yellow Mice's Blog(一个ruby&php dalao)](https://web.archive.org/web/20201020200037/https://fiveyellowmice.com/)  
[老K(csslayer)的另一个博客](https://web.archive.org/web/20201020200037/https://marisa-kirisa.me/anchor/)  
[Solomon(萌萌的Poi的自制博客~)](https://web.archive.org/web/20201020200037/https://blog.poi.cat/)  
[Farseerfc的小窩(ArchLinux TU ~ 可爱的爱呼吸酱)](https://web.archive.org/web/20201020200037/https://farseerfc.me/)  
[Sherlock-Holo的博客～（夏狼好可爱！)](https://web.archive.org/web/20201020200037/https://sherlock-holo.github.io/)  
[KK的博客~(前辈好OwO)](https://web.archive.org/web/20201020200037/https://ikk.me/)  
[智乃酱(Frantic1048)的博客~(萌萌的智乃> <)](https://web.archive.org/web/20201020200037/http://frantic1048.logdown.com/)  
[No Silver Bullet | LA  
的博客=w=](https://web.archive.org/web/20201020200037/https://tech.silverrainz.me/ )  
某兔子の御用花园 | 喵兔的博客\w/  
[千千blog (萌千千=w](https://web.archive.org/web/20201020200037/https://wwyqianqian.github.io/)   
[Typeblog | 彼得彼得的博客w](https://web.archive.org/web/20201020200037/https://typeblog.net/)   
[静静's Blog | 好耶w](https://web.archive.org/web/20201020200037/https://kernel.moe/)  
[南浦月 | 乔~~姐姐~~老师的博客](https://web.archive.org/web/20201020200037/https://blog.nanpuyue.com/)  
[初等記憶體 | 艾雨寒的博客](https://web.archive.org/web/20201020200037/https://axionl.github.io/)  
[It's Kiri~!! | Kiriririri 的博客](https://web.archive.org/web/20201020200037/https://kirikira.com/)  
[失う | Equim 酱的博客~ 是 Gopher 呢](https://web.archive.org/web/20201020200037/https://ekyu.moe/)  
[Makito's Notebook | 是可爱的 Android Dev Makito 呢~~ (吸吸)](https://web.archive.org/web/20201020200037/https://blog.keep.moe/)  
[灰灰，可爱的男孩纸](https://web.archive.org/web/20201020200037/https://huihui.moe/)  
[DuckSoft's Blog | Ex nihilo ad astra](https://web.archive.org/web/20201020200037/https://www.ducksoft.site/)  
[Leo's Field, 一位 UW 的数学系大大!](https://web.archive.org/web/20201020200037/https://szclsya.me/)
#### Archives
 * [June 2019](https://web.archive.org/web/20201020200037/https://void-shana.moe/2019/06)
* [February 2019](https://web.archive.org/web/20201020200037/https://void-shana.moe/2019/02)
* [January 2019](https://web.archive.org/web/20201020200037/https://void-shana.moe/2019/01)
* [August 2018](https://web.archive.org/web/20201020200037/https://void-shana.moe/2018/08)
* [January 2018](https://web.archive.org/web/20201020200037/https://void-shana.moe/2018/01)
* [November 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/11)
* [September 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/09)
* [July 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/07)
* [June 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/06)
* [May 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/05)
* [April 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/04)
* [March 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/03)
* [February 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/02)
* [January 2017](https://web.archive.org/web/20201020200037/https://void-shana.moe/2017/01)
* [December 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/12)
* [November 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/11)
* [October 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/10)
* [September 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/09)
* [August 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/08)
* [July 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/07)
* [June 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/06)
* [May 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/05)
* [April 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/04)
* [March 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/03)
* [February 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/02)
* [January 2016](https://web.archive.org/web/20201020200037/https://void-shana.moe/2016/01)
* [November 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/11)
* [October 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/10)
* [September 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/09)
* [August 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/08)
* [May 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/05)
* [April 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/04)
* [March 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/03)
* [February 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/02)
* [January 2015](https://web.archive.org/web/20201020200037/https://void-shana.moe/2015/01)
* [December 2014](https://web.archive.org/web/20201020200037/https://void-shana.moe/2014/12)
* [November 2014](https://web.archive.org/web/20201020200037/https://void-shana.moe/2014/11)
* [October 2014](https://web.archive.org/web/20201020200037/https://void-shana.moe/2014/10)
* [September 2014](https://web.archive.org/web/20201020200037/https://void-shana.moe/2014/09)
* [August 2014](https://web.archive.org/web/20201020200037/https://void-shana.moe/2014/08)
#### Categories
 * [ACM Algorithm](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo "ACM&算法")
* [AI](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/ai)
* [Android](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/android)
* [archlinux](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/archlinux)
* [C](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/c)
* [DynamicProgramming](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/dynamicprogramming)
* [emacs](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/emacs)
* [gcc](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/gcc)
* [golang](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/golang)
* [GraphTheory](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/graphtheory)
* [hash](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/hash)
* [Kernel](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/kernel)
* [Kernel](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/kernel-linux)
* [Linux](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux)
* [Linux\_Basis](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/linux_basis)
* [LuaSource](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/c/luasource)
* [Minecraft](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/mc "Minecraft 游戏相关心得，记录")
* [Number Theory](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/number-theory)
* [PHP](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/webdev/php "PHP")
* [Projects](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/projects)
* [Robocode~](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/robocode)
* [Ruby](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/ruby)
* [STL](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/stl)
* [Threads and Process](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/threads-and-process)
* [Uncategorized](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/uncategorized)
* [vim](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/linux/vim)
* [Vocaloid](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/vocaloid)
* [Water](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/acmalgo/water)
* [Web Develop](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/webdev "Web开发")
* [wine](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/wine)
* [基础网络知识](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e5%9f%ba%e7%a1%80%e7%bd%91%e7%bb%9c%e7%9f%a5%e8%af%86)
* [文章转载](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e6%96%87%e7%ab%a0%e8%bd%ac%e8%bd%bd)
* [杂七杂八](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e6%9d%82%e4%b8%83%e6%9d%82%e5%85%ab)
* [计算机系统原理](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%b3%bb%e7%bb%9f%e5%8e%9f%e7%90%86)
* [计算机网络](https://web.archive.org/web/20201020200037/https://void-shana.moe/category/%e8%ae%a1%e7%ae%97%e6%9c%ba%e7%bd%91%e7%bb%9c)
#### Tags
[archlinux](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/archlinux)
[C](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/c)
[C. Linux](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/c-linux)
[kernel](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/kernel)
[Laravel](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/laravel)
[linux](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/linux)
[os](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/os)
[PHP](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/php)
[prompt](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/prompt)
[Python](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/python)
[qemu](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/qemu)
[rtmp](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/rtmp)
[Shell](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/shell)
[video](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/video)
[vim](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/vim)
[Web](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/web)
[wine](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/wine)
[zsh](https://web.archive.org/web/20201020200037/https://void-shana.moe/tag/zsh)
© 2020  | 
Proudly Powered by [WordPress]( https://wordpress.org/)
 | 
Theme: [Nisarg](https://web.archive.org/web/20201020200037/https://wordpress.org/themes/nisarg/) 
/* <![CDATA[ */
var screenReaderText = {"expand":"expand child menu","collapse":"collapse child menu"};
/* ]]> */
/* <![CDATA[ */EnlighterJS\_Config = {"selector":{"block":"pre.EnlighterJSRAW","inline":"code.EnlighterJSRAW"},"language":"generic","theme":"git","indent":2,"hover":"hoverEnabled","showLinenumbers":true,"rawButton":true,"infoButton":true,"windowButton":true,"rawcodeDoubleclick":true,"grouping":true,"cryptex":{"enabled":false,"email":"mail@example.tld"}};!function(){var a=function(a){var b="Enlighter Error: ";console.error?console.error(b+a):console.log&&console.log(b+a)};return window.addEvent?"undefined"==typeof EnlighterJS?void a("Javascript Resources not loaded yet!"):"undefined"==typeof EnlighterJS\_Config?void a("Configuration not loaded yet!"):void window.addEvent("domready",function(){EnlighterJS.Util.Init(EnlighterJS\_Config.selector.block,EnlighterJS\_Config.selector.inline,EnlighterJS\_Config)}):void a("MooTools Framework not loaded yet!")}();;/* ]]> */

            