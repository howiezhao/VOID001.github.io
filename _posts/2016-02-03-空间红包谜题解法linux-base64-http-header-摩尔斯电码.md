---
title: 空间红包谜题解法(Linux + base64 + HTTP Header + 摩尔斯电码)
---
==============================================



#####  [February 3, 2016](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html "11:51 am") 
[VOID001](https://web.archive.org/web/20201020200834/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [3 comments](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html#comments)





题目
--


解密红包, 支付宝中文口令红包 [![](https://web.archive.org/web/20201020200834im_/http://120.27.97.96/wp-content/uploads/2016/02/FveHO1l4MSPT3TYG5TLs89U4sdbQ.png "FveHO1l4MSPT3TYG5TLs89U4sdbQ")](https://web.archive.org/web/20201020200834/http://120.27.97.96/wp-content/uploads/2016/02/FveHO1l4MSPT3TYG5TLs89U4sdbQ.png)


(后来被人说二维码扫完还得用电脑重输一遍网址好麻烦 = = 于是我就把二维码对应的网址给出来了 : [http://1.justquiz.sinaapp.com/voidmengmengda](https://web.archive.org/web/20201020200834/http://1.justquiz.sinaapp.com/voidmengmengda)) —- 这就是完整的题干了, 谜题总共有三关, 个人认为难度是递增的, 下面放出每一关的解法


解法
--


* 第一关
=====


首先看第一关的内容, 网页本身查看源码并没有什么用处, 不过注意到标题, “请用桌面浏览器打开哦~” 桌面浏览器(即Chrome Firefox) 和一般的移动端浏览器的区别就是, 桌面浏览器支持开发者视图, 可以看到更多详细信息, 除了看到源码, 我们还可以看到cookie session storage等东西, 也包括请求的header, 而这一关的线索就藏在了header里, 在服务器Response你的Request的时候, 会在header里带上下一关的线索,  之后我在空间里也对第一关进行了提示, 有十多人在提示之后到达了第二关QAQ 比我预期的少了很多 附上第一关解决的截图(点击可查看大图) 图中的 HintURL 就是第二关的线索 [![](https://web.archive.org/web/20201020200834im_/http://120.27.97.96/wp-content/uploads/2016/02/Screenshot-from-2016-02-03-10-44-41.png "Level1 Solved")](https://web.archive.org/web/20201020200834/http://120.27.97.96/wp-content/uploads/2016/02/Screenshot-from-2016-02-03-10-44-41.png)


 


* 第二关
=====


( 列出服务器网站目录) 进入到第二关, 你会发现看到一个在线编译器, 以及这样一段提示想信息:



> 
> Happy New Year~! Congrats! You have solve the first Challenge! See what you can find here
> Remember : This is a BUGGY online Compiler
> 


This is a BUGGY online Compiler 表示这个在线编译器是有bug的, 具体是什么bug就需要大家自己去找了, 这一关我没有再给提示, 大家稍微尝试之后就会发现, 这个online Compiler没有禁止系统指令的执行, (虽然用户权限是正确的, 无法删除root 的文件, 但是还是有很多事情可以做的)  因而编写下面代码到在线编译器里 , (我选用C语言做示例)


```
#include<stdio.h>

int main(void)
{
    system("ls -al");
    return 0;
}
```


执行之后 ,会显示如下信息 :



```
total 348
drwxr-xr-x 10 root root   4096 Feb  2 10:01 .
drwxr-xr-x  3 root root   4096 Nov  9 07:37 ..
-rw-r--r--  1 root root   4254 Feb  2 06:34 actionclass.php
drwxr-xr-x  8 root root   4096 Nov  8 04:13 amoeba
-rw-r--r--  1 root root   2196 Nov  9 06:48 backend.php
drwxr-xr-x  2 root root   4096 Nov  8 04:13 contact
drwxr-xr-x  2 root root   4096 Nov  8 04:13 css
-rw-r--r--  1 root root   4096 Nov  8 04:11 .\_css
-rw-r--r--  1 root root 216524 Nov  9 06:48 DMKSB.jpg
drwxr-xr-x  3 root root   4096 Nov  8 04:13 fonts
-rw-r--r--  1 root root   4096 Nov  8 04:11 .\_fonts
-rw-r--r--  1 root root   2434 Nov  9 06:48 function.php
-rw-r--r--  1 root root     27 Nov  9 06:48 .gitignore
drwxr-xr-x  5 root root   4096 Nov  8 04:13 img
-rw-r--r--  1 root root   5767 Nov  8 04:19 index.html.old
-rw-r--r--  1 root root    168 Feb  2 02:57 index.php
drwxr-xr-x  3 root root   4096 Nov  8 04:13 js
-rw-r--r--  1 root root   4096 Nov  8 04:11 .\_js
-rw-r--r--  1 root root   5638 Nov  9 06:48 login.php
-rw-r--r--  1 root root    580 Nov  9 06:48 queue.php
-rw-r--r--  1 root root     98 Nov  8 14:10 readme.txt
drwxrwxrwx  2 root root  20480 Feb  3 02:51 sandbox
drwxr-xr-x  2 root root   4096 Nov  8 04:13 skin
-rw-r--r--  1 root root  10007 Feb  2 10:01 submit.php
-rw-r--r--  1 root root    166 Feb  2 07:12 thisIStheHintYouNeed.html
```

稍微留心一下, 就会发现, 你列出的目录, 就是你当前访问的网站所在的目录, 同一级 ,再仔细看一眼列表 ,找到提示文件 thisIStheHintYouNeed.html 通过浏览器访问这个文件 , 得到下一关的提示信息, 本关解决. 附上本关解决之后的截图(点击查看大图) [![](https://web.archive.org/web/20201020200834im_/http://120.27.97.96/wp-content/uploads/2016/02/Screenshot-from-2016-02-03-10-54-54.png "Screenshot from 2016-02-03 10-54-54")](https://web.archive.org/web/20201020200834/http://120.27.97.96/wp-content/uploads/2016/02/Screenshot-from-2016-02-03-10-54-54.png)


 


* 第三关
=====


这是本谜题的最后一关了, 也是我制作的时候花时间最多的一关(毕竟第一次用CoolEdit 以前从来没玩过QAQ) 通过第二关的最后提示, 得到一个网址 : 在浏览器里访问改网址 得到类似下面的东西(仅截取部分做展示)



```
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
```

看到这种形式的文件, 第一反应就是base64编码, 尝试用在线解码器进行解码 , 结果发现解出乱码了 , 推测这可能是一个二进制文件, 于是用Linux下自带的解码工具 (Windows下也许也有,不过我不清楚00.00) 将文件解码之后, 输出到另一个文件, 注意不要带后缀名, 带后缀名会影响linux对文件类型的推测, 一会儿要进行文件类型的推测 执行下面指令得到解码后的文件 :



```
wget http://1.voidword.sinaapp.com/hongbao/drowssap.txt
base64 -d drowssap.txt > hint
```

然后推测文件类型 , 通过hexdump xxd 等工具并没有看出文件是什么类型的 = = (也许是我自己能力不够) 因此使用linux下的file指令推测文件类型



```
╭─[[email protected]](/web/20201020200834/https://void-shana.moe/cdn-cgi/l/email-protection)\_PC /tmp  
╰─➤  file hint 
hint: MPEG ADTS, layer III, v1, 128 kbps, 44.1 kHz, JntStereo
```

通过这个可以看出, 这应该是一个媒体文件 , 因此用播放器对其进行播放 , (linux 下有工具 playsound 或者修改后缀名进行播放) 听到电报音, 长短结合,  想到可能是摩尔斯电码, 然后根据摩尔斯电码表对应下来 密码为 100081  转成中文 一零零八一, 输入进口令, 然而并不对 , 这是这一关的最后一个陷阱, 发现不对之后, 回去看一下第二关的提示URL :  最后面 drowssap.txt 看起来是不是很眼熟,  他就是单词password的逆转, 这是重要的提示信息,因此就说明要对摩尔斯电码进行逆转, 那么到底是把所有的数字颠倒顺序, 还是指把点(.) 和划(-) 的表示颠倒? 尝试之后发现 , 逆转得到的密码为 180001  而将点划表示颠倒之后得到的密码为 655536  作为一个程序员, 这两个选择哪个就不难决定了吧~  到此整个谜题全部解开了, 在支付宝页面输入正确的密码对应的中文, 就会拿到红包啦~    


 


 


后记 : 最后也没有人拿到那个红包QAQ,  不过还是有很多人很接近答案了~ 解密红包的乐趣在于解密而不是红包~ 我是玩的很愉快, 大家也玩的很愉快就好~


 


附上解密后的音频文件:


[hint](https://web.archive.org/web/20201020200834/http://120.27.97.96/wp-content/uploads/2016/02/hint.mp3)


以及摩尔斯电码对照表


数字
--




| **字符** | **电码符号** | **字符** | **电码符号** | **字符** | **电码符号** | **字符** | **电码符号** |
| 0 | ━ ━ ━ ━ ━ | 1 | ．━ ━ ━ ━ | 2 | ．．━ ━ ━ | 3 | ．．．━ ━ |
| 4 | ．．．．━ | 5 | ．．．．． | 6 | ━ ．．．． | 7 | ━ ━ ．．． |
| 8 | ━ ━ ━ ．． | 9 | ━ ━ ━ ━ ． |  |  |  |


### 


 


### 






---


[archlinux](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/linux/archlinux), [Linux](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/linux), [Linux\_Basis](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/linux/linux_basis), [PHP](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/webdev/php), [Web Develop](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/webdev), [基础网络知识](https://web.archive.org/web/20201020200834/https://void-shana.moe/category/%e5%9f%ba%e7%a1%80%e7%bd%91%e7%bb%9c%e7%9f%a5%e8%af%86) [C. Linux](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201020200834/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20201020200834im_/https://secure.gravatar.com/avatar/3ce01e70cf8d4c14a0d55902d0499aaf?s=50&d=identicon&r=g) **Faustmeow** says: 
[February 3, 2016 at 4:50 pm](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html#comment-25)
好想再玩一个，最喜欢解谜了 \_(:Ⅰ」)\_


2. ![](https://web.archive.org/web/20201020200834im_/https://secure.gravatar.com/avatar/e8dae3cdb92efc1b30b330109517d7d4?s=50&d=identicon&r=g) **[woshishabi](https://web.archive.org/web/20201020200834/http://wssb.com/)** says: 
[February 3, 2016 at 5:31 pm](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html#comment-26)
我擦太特么高端了


3. ![](https://web.archive.org/web/20201020200834im_/https://secure.gravatar.com/avatar/892cd873c037dc12b9aaddad43d45e98?s=50&d=identicon&r=g) **萌之领域** says: 
[February 4, 2016 at 12:32 pm](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html#comment-27)
QAQ 玩到最后猜到100081结果没有转成655536 忧伤 T T


3. ![](https://web.archive.org/web/20201020200834im_/https://secure.gravatar.com/avatar/892cd873c037dc12b9aaddad43d45e98?s=50&d=identicon&r=g) **萌之领域** says: 
[February 4, 2016 at 12:32 pm](https://web.archive.org/web/20201020200834/https://void-shana.moe/linux/%e7%a9%ba%e9%97%b4%e7%ba%a2%e5%8c%85%e8%b0%9c%e9%a2%98%e8%a7%a3%e6%b3%95linux-base64-http-header-%e6%91%a9%e5%b0%94%e6%96%af%e7%94%b5%e7%a0%81.html#comment-27)
QAQ 玩到最后猜到100081结果没有转成655536 忧伤 T T

            