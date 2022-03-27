---
title: (ArchLinux) 使用systemd创建自定义的开机启动项
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [August 14, 2016](https://web.archive.org/web/20210418232828/https://void-shana.moe/linux/archlinux-%e4%bd%bf%e7%94%a8systemd%e5%88%9b%e5%bb%ba%e8%87%aa%e5%ae%9a%e4%b9%89%e7%9a%84%e5%bc%80%e6%9c%ba%e5%90%af%e5%8a%a8%e9%a1%b9.html "9:24 am") 
[VOID001](https://web.archive.org/web/20210418232828/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210418232828/https://void-shana.moe/linux/archlinux-%e4%bd%bf%e7%94%a8systemd%e5%88%9b%e5%bb%ba%e8%87%aa%e5%ae%9a%e4%b9%89%e7%9a%84%e5%bc%80%e6%9c%ba%e5%90%af%e5%8a%a8%e9%a1%b9.html#respond)





为了满足日常工作开发需要, 窝需要Cisco的VPN agent在开机额时候就启动, 而Arch不像Ubuntu用/etc/initrc下编写启动脚本, 下面简单记录一下编写vpnagentd的启动脚本以及启动的设置为开机启动的过程


首先在 /usr/lib/systemd/system下创建一个文件叫做vpnagentd.service然后写入如下内容


 



```
[Unit]
Description=Cisco VPN Service
Wants=NetworkManager.service
After=NetworkManager.service

[Service]
Type=forking
ExecStart=/opt/cisco/anyconnect/bin/vpnagentd
PIDFile=/var/run/vpnagentd.pid
ExecReload=/usr/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target

```

这里的ExecStart填写的是你要启动的vpnagentd所在的路径, 其他的参数都一目了然, WantedBy暂时先不解释(因为窝也没研究呢QAQ)


然后 执行  `systemctl enable vpnagentd.service`  然后就可以啦


 






---


[archlinux](https://web.archive.org/web/20210418232828/https://void-shana.moe/category/linux/archlinux), [Linux](https://web.archive.org/web/20210418232828/https://void-shana.moe/category/linux), [Linux\_Basis](https://web.archive.org/web/20210418232828/https://void-shana.moe/category/linux/linux_basis) [C. Linux](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210418232828/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
[看着不爽] NEUCrack 替代掌上东大的网页端查询工具](https://web.archive.org/web/20210418232828/https://void-shana.moe/webdev/%e7%9c%8b%e7%9d%80%e4%b8%8d%e7%88%bd-%e6%96%b0%e7%94%9f%e4%b8%8d%e9%9c%80%e8%a6%81%e6%8e%8c%e4%b8%8a%e4%b8%9c%e5%a4%a7%e4%b9%9f%e5%8f%af%e4%bb%a5%e6%9f%a5%e8%af%a2%e5%88%b0%e5%af%9d%e5%ae%a4.html)
[PREVIOUS 
[Linux 文件系统]一张图解释什么是硬链接什么是软链接](https://web.archive.org/web/20210418232828/https://void-shana.moe/linux/%e4%b8%80%e5%bc%a0%e5%9b%be%e8%a7%a3%e9%87%8a%e4%bb%80%e4%b9%88%e6%98%af%e7%a1%ac%e9%93%be%e6%8e%a5%e4%bb%80%e4%b9%88%e6%98%af%e8%bd%af%e9%93%be%e6%8e%a5.html)

            