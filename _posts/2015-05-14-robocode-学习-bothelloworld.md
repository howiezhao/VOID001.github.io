---
title: (Robocode 学习) botHelloWorld
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [May 14, 2015](https://web.archive.org/web/20210418232305/https://void-shana.moe/robocode/robocode-%e5%ad%a6%e4%b9%a0-bothelloworld.html "10:25 pm") 
[VOID001](https://web.archive.org/web/20210418232305/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210418232305/https://void-shana.moe/robocode/robocode-%e5%ad%a6%e4%b9%a0-bothelloworld.html#respond)





这是第一篇学习Robocode的文章 本篇将会对Robocode的开发环境进行搭建 ,以及 Robot类的基本API进行解释,最后会有botHelloWorld  , 我的第一个Robot ~~ QWQ



1.Robocode 环境的安装和Eclipse的配置
---------------------------


Robocode是一个跨平台的游戏, 在robocode官网下载 Robocode的安装包,只要电脑内有Java的运行环境就可以使用Robocode了~


官方网站: http://robocode.sourceforge.net/


Linux下 安装方式:


下载好最新的jar安装包后 在Shell运行如下命令:java -jar /path/to/your/robocode\_install/  然后,切换到你安装的目录后,执行如下命令~ ./robocode.sh


### 1.Eclipse环境的配置:


安装好Eclipse和java环境,然后新建一个工程 JavaProject 如图


:[![PPST](https://web.archive.org/web/20210418232305im_/http://localhost/My_Blog/wp-content/uploads/2015/05/PPST-300x169.png)](https://web.archive.org/web/20210418232305/http://localhost/My_Blog/wp-content/uploads/2015/05/PPST.png)


 


### 2.设置JavaProject


建立好JavaProject后 为其添加一个package 命名为一个独一无二的名字,这个名字就是你在Robocode里看到的你的所有机器人所属的包的名字,这样就建立好了一个包.接下来是设置Robocode的lib能被我们的JavaProject使用,右键点击左侧的src文件夹,选择Preference,然后在弹出的设置框内选择Native Library[![asdsaf](https://web.archive.org/web/20210418232305im_/http://localhost/My_Blog/wp-content/uploads/2015/05/asdsaf-300x210.png)](https://web.archive.org/web/20210418232305/http://localhost/My_Blog/wp-content/uploads/2015/05/asdsaf.png)


然后 选择Externel ,将Robocode下的lib文件夹添加到目录里即可


至此,对Eclipse的所有环境设置完毕,注意Eclipse会自动编译你写好的java文件,你可以在Build菜单下取消自动编译改为手动


### 3.Robocode下的一些必要设置


运行robocode 然后选择 Options->Preference->DevelopmentOptions将你刚刚创建的WorkSpace的bin文件夹添加进去~ 然后稍等片刻Robocode更新完数据库时候,就能在Robot的列表里看到你在Eclipse下开发的Robot了~


 


我的第一个Bot botHelloWorld
----------------------


很简单的一个机器人 直接贴代码了QWQ



```
package voidbot.voidword;
import robocode.*;
import java.math.*;
import java.awt.Color;

public class botHelloWorld extends Robot
{
    double hp,myX,myY;
    double gunHeat;
    boolean hitOnBot;
    public void run()
    {
        hitOnBot = false;
        while(true)
        {
            //doNothing();
            //System.out.println("Hello");
            for(int i=0;i<4 && !hitOnBot ;i++)
            {
                turnRight(90);
                ahead(300);
            }
        }
    }
    /*
     * (non-Javadoc)
     * @see robocode.Robot#onScannedRobot(robocode.ScannedRobotEvent)
     */
    public void onScannedRobot(ScannedRobotEvent event)
    {
        if(event.getDistance()<30)
        {
            fire(5);
        }
        else
        {
            fire(3);
        }
        scan();
    }
    public void onHitByBullet(HitByBulletEvent event)
    {
        turnGunRight(360);
        ahead(100);
    }
    public void onHitRobot(HitRobotEvent event)
    {
        hitOnBot = true;
        turnGunRight(360);
        hitOnBot = false;
    }
}

```

 






---


[Robocode~](https://web.archive.org/web/20210418232305/https://void-shana.moe/category/robocode) [C. Linux](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210418232305/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
动态规划建模方法](https://web.archive.org/web/20210418232305/https://void-shana.moe/acmalgo/%e5%8a%a8%e6%80%81%e8%a7%84%e5%88%92%e5%bb%ba%e6%a8%a1%e6%96%b9%e6%b3%95.html)
[PREVIOUS 
[转载]有向图强连通分量的Tarjan算法](https://web.archive.org/web/20210418232305/https://void-shana.moe/acmalgo/%e8%bd%ac%e8%bd%bd%e6%9c%89%e5%90%91%e5%9b%be%e5%bc%ba%e8%bf%9e%e9%80%9a%e5%88%86%e9%87%8f%e7%9a%84tarjan%e7%ae%97%e6%b3%95.html)

            