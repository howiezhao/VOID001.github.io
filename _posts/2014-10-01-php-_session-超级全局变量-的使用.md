---
title: PHP $\_SESSION 超级全局变量 的使用
---
=========================



#####  [October 1, 2014](https://web.archive.org/web/20210418233646/https://void-shana.moe/webdev/php-_session-%e8%b6%85%e7%ba%a7%e5%85%a8%e5%b1%80%e5%8f%98%e9%87%8f-%e7%9a%84%e4%bd%bf%e7%94%a8.html "2:14 pm") 
[VOID001](https://web.archive.org/web/20210418233646/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210418233646/https://void-shana.moe/webdev/php-_session-%e8%b6%85%e7%ba%a7%e5%85%a8%e5%b1%80%e5%8f%98%e9%87%8f-%e7%9a%84%e4%bd%bf%e7%94%a8.html#respond)





SESSION 变量，是给每个不同的用户分配一个不同的UID然后把相应的变量的值存储在服务器端，画个图就是这样的：


[![](https://web.archive.org/web/20210418233646im_/http://120.27.97.96/wp-content/uploads/2014/10/whatisPHPSessionVar.png "whatisPHPSessionVar")](https://web.archive.org/web/20210418233646/http://120.27.97.96/wp-content/uploads/2014/10/whatisPHPSessionVar.png)


如这个图片所示，有两个用户接入了某个服务器 一个是User1 UID=001 一个是User2 UID=002 那么，他们同时向服务器发出请求的时候，服务器端要对身份进行认证，只有username=admin的用户才有权限访问，这时候，在服务器端就要用到下面的代码 ：



```
<?php
session\_start();
check\_login(); ?>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd";>

<html>

 <head>

  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">

<?php
      //OTHER CODES ..........

	function check\_login()
	{
		if(!isset($\_SESSION['username']))
		{
			header("Location:index.php");
			exit();
		}
		return ;
	}
?>

	</head>

 </html>
```

这段代码的作用是：当用户访问了该页面的时候，先检测用户的username是否为admin 如果是的话，执行其它代码 否则返回登录页面 。具体的识别机制就是上面的那样，**首先，要使用 session\_start()函数，而且，这个函数要用在所有输出操作未进行之前，不然就没法建立会话了**，建立会话之后，服务器端在login页面会给用户分配一个UID如上图，当然，分配的是加密的ID 然后并且在服务器端给这个UID建立相应的PHPSESSION 数组，具体实现是给用户发送了一个SESSION Cookie 在关闭浏览器的时候，就会把Cookie销毁，然后，在用户向服务器发送访问页面请求的时候，服务器端会根据这个用户的UID找到这个用户的SESSION 变量，并且该用户的SESSION变量不会被别人访问到。然后 当用户存在 $\_SESSION[‘username’]这个变量时，就登录成功，否则失败。






---


[PHP](https://web.archive.org/web/20210418233646/https://void-shana.moe/category/webdev/php), [Web Develop](https://web.archive.org/web/20210418233646/https://void-shana.moe/category/webdev) [C. Linux](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210418233646/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Linux XAMPP 安装](https://web.archive.org/web/20210418233646/https://void-shana.moe/uncategorized/linux-xampp-%e5%ae%89%e8%a3%85.html)
[PREVIOUS 
“复杂” 读书笔记 (美 梅拉尼 米歇尔)](https://web.archive.org/web/20210418233646/https://void-shana.moe/ai/%e5%a4%8d%e6%9d%82-%e8%af%bb%e4%b9%a6%e7%ac%94%e8%ae%b0-%e7%be%8e-%e6%a2%85%e6%8b%89%e5%b0%bc-%e7%b1%b3%e6%ad%87%e5%b0%94.html)

            