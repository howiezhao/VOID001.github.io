---
title: (Lua 源码解读 0) lstrlib.c 模式匹配系列函数解析
---
=================================



#####  [September 3, 2016](https://web.archive.org/web/20201020195343/https://void-shana.moe/linux/c/lua-%e6%ba%90%e7%a0%81%e8%a7%a3%e8%af%bb-0-lstrlib-c-%e6%a8%a1%e5%bc%8f%e5%8c%b9%e9%85%8d%e7%b3%bb%e5%88%97%e5%87%bd%e6%95%b0%e8%a7%a3%e6%9e%90.html "11:33 am") 
[VOID001](https://web.archive.org/web/20201020195343/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201020195343/https://void-shana.moe/linux/c/lua-%e6%ba%90%e7%a0%81%e8%a7%a3%e8%af%bb-0-lstrlib-c-%e6%a8%a1%e5%bc%8f%e5%8c%b9%e9%85%8d%e7%b3%bb%e5%88%97%e5%87%bd%e6%95%b0%e8%a7%a3%e6%9e%90.html#respond)





前言
--


当当当当~~~~ lua源码阅读的记录文开坑了， 窝准备把lua的源码都通读一遍，并且给出自己的记录&体会，最后，根据阅读源码学到的东西，考虑实现一个相关的小project(这是一个大坑，flag已立)


提供一些我找到的对阅读源码有帮助的资源


http://del.icio.us/adnamin/lua+source


窝按照这里提供的顺序阅读lua源码， 至少起步是按照这个顺序来的，等到自己对这个代码比较熟练的时候，可能会考虑按照自己的喜好顺序来


https://www.reddit.com/comments/63hth/ask\_reddit\_which\_oss\_codebases\_out\_there\_are\_so/c02pxbp


lmathlib.c中的很多函数都只是一个wrapper的作用，在这里就先不过多介绍了，我们先来介绍lstrlib.c的函数&功能


lstrlib.c提供了lua的字符串相关的库函数&helper函数，一个重点功能是提供了pattern matching的功能，注意它提供的pattern和正则表达式是有区别的，应该说功能没有正则表达式强大，但是日常匹配使用也是够了，lua官方对这个pattern有一个很系统的解释，这里总结一下，并且理解这个对阅读相关的代码也有很大的帮助，先放一个Lua5.1原生文档对Pattern的说明的链接


https://www.lua.org/manual/5.1/manual.html#5.4.1


下面就来解释一下Lua里面的模式匹配及其定义的体系


Lua 的 Pattern 体系
----------------


* character class 是组成一个pattern串的基本单位， 每一个character class代表一类字符，他们可以是一个set里的，可以是set的补集，可以是转义字符定义的类型如%d数字%l小写字母等
* pattern item是用来匹配的基本单位，一个pattern item由一个character class加上*,+,…等特定的符号组成，表示重复多次、一次等含义,几个特殊的pattern,一个是 %n, n可以是0-9的数字， 表示匹配的内容和capture的内容一致，%bxy表示balance match从x开始到y的一个字符串, 关于什么是balance match详见文档
* pattern是匹配的基本单位,一个pattern由一个或者多个pattern item组成，可以在首部含有^尾部含有$，语意和正则表达式内的这两个符号的语意一致
* capture 是将一个pattern用小括号括起来, 然后如果源子串match了这个括号内的pattern,会被“捕捉”(存储)起来供之后的匹配使用


 


### 源码解析


首先我们来看一下 最下方export的函数,这里export了一系列的函数如下



```
static const luaL\_Reg strlib[] = {
  {"byte", str\_byte},
  {"char", str\_char},
  {"dump", str\_dump},
  {"find", str\_find},
  {"format", str\_format},
  {"gfind", gfind\_nodef},
  {"gmatch", gmatch},
  {"gsub", str\_gsub},
  {"len", str\_len},
  {"lower", str\_lower},
  {"match", str\_match},
  {"rep", str\_rep},
  {"reverse", str\_reverse},
  {"sub", str\_sub},
  {"upper", str\_upper},
  {NULL, NULL}
};

```

我们通过查看文档，对比名称发现，核心的函数string.find string.gmatch是用来处理整个Pattern Matching逻辑的函数，在源文件里这两个函数分别叫做 str\_find, gmatch, 我们先来查看一下str\_find函数，源码就不发上来了，大家可以很容易的在lua官网get到source code,这个函数就是一个wrapper,调用了str\_find\_aux（可以认为这是一个auxillary）我们下面来解析一下str\_find\_aux这个函数


我们首先来查看一下string.find的用法


### `string.find (s, pattern [, init [, plain]])`


用C去取lua的参数的时候,是通过lua的Stack去取的参数参数压栈顺序是从左到右依次压栈



```
  size\_t l1, l2;
  const char *s = luaL\_checklstring(L, 1, &l1);
  const char *p = luaL\_checklstring(L, 2, &l2);
  ptrdiff\_t init = posrelat(luaL\_optinteger(L, 3, 1), l1) - 1;
  if (init < 0) init = 0;
  else if ((size\_t)(init) > l1) init = (ptrdiff\_t)l1;
```

上面这一段代码是从lua的Stack中获取参数到C, *s是s *p是pattern init最后为字符串的起始位置，luaL系列的这几个函数的第二个参数均是用于标示这个元素在栈中的位置,正数表示从Stack Bottom开始算起，负数表示栈顶算起, -1表示栈顶，想了解更多相关知识参考我之前的文章[用C语言操作lua](https://web.archive.org/web/20201020195343/https://voidisprogramer.com/linux/c/%e7%94%a8c%e8%af%ad%e8%a8%80%e6%93%8d%e4%bd%9clua.html),通过这些代码就获取到了用户传递给Lua的三个参数


下面的一段代码



```
if (find && (lua\_toboolean(L, 4) ||  /* explicit request? */
      strpbrk(p, SPECIALS) == NULL)) {  /* or no special characters? */
    /* do a plain search */
    const char *s2 = lmemfind(s+init, l1-init, p, l2);
    if (s2) {
      lua\_pushinteger(L, s2-s+1);
      lua\_pushinteger(L, s2-s+l2);
      return 2;
    }
  }
```

是处理plain search, 所谓plain search就是不带任何转义 特殊模式串,完全就是pattern是什么样的就去找什么(这时候  [, %,等在匹配的时候有特殊含义的符号,均不表达特殊含义而只是一个字符),plain search的条件也可以很清晰的看出来,如果这个字符串不含有特殊字符，或者传递的第四个参数plain == true的话，那么就做plain search，并返回结果，将结果压栈, return 2表示返回两个值给caller


之后的这一段代码,是Pattern Matching的主体,这里面涉及到了字符串的模式匹配等内容,我们先来看一下



```
else {
    MatchState ms;
    int anchor = (*p == '^') ? (p++, 1) : 0;
    const char *s1=s+init;
    ms.L = L;
    ms.src\_init = s;
    ms.src\_end = s+l1;
    do {
      const char *res;
      ms.level = 0;
      if ((res=match(&ms, s1, p)) != NULL) {
        if (find) {
          lua\_pushinteger(L, s1-s+1);  /* start */
          lua\_pushinteger(L, res-s);   /* end */
          return push\_captures(&ms, NULL, 0) + 2;
        }
        else
          return push\_captures(&ms, s1, res);
      }
    } while (s1++ < ms.src\_end && !anchor);
  }
```

首先去掉匹配pattern的锚点^,配置好MatchState(Lua使用matchState进行Pattern Matching的信息存储以及处理), 一个do循环从字符串的开头到结尾去match匹配的pattern， 有关match的解析，我会在之后的文中进行介绍，这里因为是第一篇lkua分析文，目前不会这么深入.


我们再来看一下gmatch这个函数


这里面有一个函数之前没有遇到过,就是这个**lua\_pushcclosure**,因为目前我的知识有限，暂时理解为这个函数是用来传递一个函数到luaStack并且去调用它，对于这个函数的具体解读也会在之后进行


对于lstrlib.c的解读就到这里，主要是解读了Pattern Matching的相关内容,本人水平有限，刚接触lua源码，可能有的地方解答不对，欢迎大家批评指正，同时欢迎大家在下方评论留言探讨交流=w=






---


[C](https://web.archive.org/web/20201020195343/https://void-shana.moe/category/linux/c), [LuaSource](https://web.archive.org/web/20201020195343/https://void-shana.moe/category/linux/c/luasource) [C. Linux](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201020195343/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
NEUOJ Developer Wanted!!!](https://web.archive.org/web/20201020195343/https://void-shana.moe/uncategorized/neuoj-developer-wanted.html)
[PREVIOUS 
Protected: 调味料BD](https://web.archive.org/web/20201020195343/https://void-shana.moe/uncategorized/%e8%b0%83%e5%91%b3%e6%96%99bd.html)

            