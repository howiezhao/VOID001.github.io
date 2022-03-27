---
title: Gmail + Mutt 报错 no authenticators found 解决办法
---
============================================



#####  [February 1, 2017](https://web.archive.org/web/20210614030453/https://void-shana.moe/linux/gmail-mutt-%e6%8a%a5%e9%94%99-no-authenticators-found-%e8%a7%a3%e5%86%b3%e5%8a%9e%e6%b3%95.html "1:55 pm") 
[VOID001](https://web.archive.org/web/20210614030453/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210614030453/https://void-shana.moe/linux/gmail-mutt-%e6%8a%a5%e9%94%99-no-authenticators-found-%e8%a7%a3%e5%86%b3%e5%8a%9e%e6%b3%95.html#respond)





初步测试是配置文件错误导致的，这里要注意几个配置不要填错


下面是一个可用的配置



```
set imap\_user = '[[email protected]](/web/20210614030453/https://void-shana.moe/cdn-cgi/l/email-protection)'
set imap\_pass = '************'
set folder = 'imaps://imap.gmail.com/'
set spoolfile = +INBOX
set record = "+[Gmail]/Sent Mail"
set postponed = "+[Gmail]/Drafts"
set smtp\_url = 'smtps://[[email protected]](/web/20210614030453/https://void-shana.moe/cdn-cgi/l/email-protection)'
set smtp\_pass = '***************'
```

### 注意: smtp\_url 要填写 smtps://[[email protected]](/web/20210614030453/https://void-shana.moe/cdn-cgi/l/email-protection) 而不是下面的其中一种


* smtps://[[email protected]](/web/20210614030453/https://void-shana.moe/cdn-cgi/l/email-protection)@smtp.gmail.com:465
* smtps://[[email protected]](/web/20210614030453/https://void-shana.moe/cdn-cgi/l/email-protection):587
* 。。。


填写正确之后， 可以使用mutt进行邮件的发送


 






---


[Linux](https://web.archive.org/web/20210614030453/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210614030453/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Eudyptula Challenge 1 — 8 总结](https://web.archive.org/web/20210614030453/https://void-shana.moe/linux/eudyptula-challenge-1-8-%e6%80%bb%e7%bb%93.html)
[PREVIOUS 
IO 多路复用 — select 和 poll](https://web.archive.org/web/20210614030453/https://void-shana.moe/linux/io-%e5%a4%9a%e8%b7%af%e5%a4%8d%e7%94%a8-select-%e5%92%8c-poll.html)

            