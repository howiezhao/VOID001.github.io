---
title: Laravel 学习笔记 #1 Installation and Configuration
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [October 30, 2014](https://web.archive.org/web/20201022000756/https://void-shana.moe/webdev/laravel-%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0-1-installation-and-configuration.html "7:39 pm") 
[VOID001](https://web.archive.org/web/20201022000756/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201022000756/https://void-shana.moe/webdev/laravel-%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0-1-installation-and-configuration.html#respond)





听力猫猫巨巨的话，准备开始学PHP的框架 ，查了一下PHP框架的排名，准备学习Laravel框架 (* ^\_^ *)   本人的系统是Ubuntu14.10  PHP已经安装了 Apache也安装了 如果上不去那个Laravel的话，怎么解决你们懂的…


安装和配置
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


安装
--


1. 我们采用Composer+Laravel Installer 的方式进行安装  首先 安装 Composer，因为我的电脑里有curl了 ，所以用第一种方式下载Composer即可 如下

```
curl -sS https://getcomposer.org/installer | php
```

没有curl的话 就要用下面的代码下载



```
php -r "readfile('https://getcomposer.org/installer');" | php
```
2. 安装好Composer后 运行如下这些指令 一条条来

```
#下载Laravel Installer
./composer.phar global require "laravel/installer=~1.1"

#将Laravel Installer的运行路径写入环境变量 $PATH方便以后使用

export PATH=$PATH:~/.composer/vendor/bin 

#在你想要安装laravel的目录下 运行安装指令
laravel new YourProjectName
```
3. 这样就构建好了一个Laravel的PHP框架 在你当前的目录下的 YourProjectName文件夹下   **WARNING: 如果遇到这个错误：**  [GuzzleHttpExceptionRequestException]  

Error creating resource. [url] http://cabinet.laravel.com/latest.zip [  

type] 2 [message] fopen(http://cabinet.laravel.com/latest.zip): failed  

to open stream: php\_network\_getaddresses: getaddrinfo failed: Name or  

service not known [file] /home/void-admin/.composer/vendor/guzzlehttp  

/guzzle/src/Adapter/StreamAdapter.php [line] 367                                               **那是 因为 安装laravel时候需要去lookup的服务器找不到 这时就要翻墙了 不然 会没法新建这个框架**
4. 配置文件 在 app/config/config.php 下 有明确的文档说明
5. 关于 URL Rewrite (Semantic URL)   这里应用官方的文档  因为我自己还不了解什么是URL Rewrite



> 
> [Pretty URLs](https://web.archive.org/web/20201022000756/http://laravel.com/docs/4.2/installation#pretty-urls)
> --------------------------------------------------------------------------------------------------------------
> 
> 
> ### Apache
> 
> 
> The framework ships with a `public/.htaccess` file that is used to allow URLs without `index.php`. If you use Apache to serve your Laravel application, be sure to enable the `mod_rewrite` module.
> 
> 
> If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this one:
> 
> 
> 
> ```
> Options +FollowSymLinks
> RewriteEngine On
> 
> RewriteCond %{REQUEST\_FILENAME} !-d
> RewriteCond %{REQUEST\_FILENAME} !-f
> RewriteRule ^ index.php [L]
> ```
> 
>  
> 
> 


 






---


[PHP](https://web.archive.org/web/20201022000756/https://void-shana.moe/category/webdev/php), [Web Develop](https://web.archive.org/web/20201022000756/https://void-shana.moe/category/webdev) [C. Linux](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201022000756/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
HDU 1142 A Walk Through The Forest](https://web.archive.org/web/20201022000756/https://void-shana.moe/acmalgo/hdu-1142-a-walk-through-the-forest.html)
[PREVIOUS 
POJ 1300 Door Man 欧拉路&字符串处理TAT](https://web.archive.org/web/20201022000756/https://void-shana.moe/acmalgo/poj-1300-door-man-%e6%ac%a7%e6%8b%89%e8%b7%af%e5%ad%97%e7%ac%a6%e4%b8%b2%e5%a4%84%e7%90%86tat.html)

            