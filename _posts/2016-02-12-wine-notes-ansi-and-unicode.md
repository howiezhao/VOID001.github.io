---
title: Wine Notes AnsiString and Unicode String
update: 2016-2-12
---

Windows下有两种字符的格式 __Unicode__ 和 __ANSI__, Windows下的很多API(Rtl系列, 之类的) 是需要认清二者的区别的

<div class="divider"></div>

### 二者的对比

{% highlight C %}
#ifndef __STRING_DEFINED__
#define __STRING_DEFINED__
typedef struct _STRING {
  USHORT Length;
  USHORT MaximumLength;
  PCHAR Buffer;
} STRING, *PSTRING;
#endif

typedef STRING ANSI_STRING;
{% endhighlight %}

而`PCHAR` 是　`char*` 的一个定义 , 因而ANSI_STRING 每一个字符占一个字节(sizeof(char))

{% highlight C %}
#ifndef __UNICODE_STRING_DEFINED__
#define __UNICODE_STRING_DEFINED__
typedef struct _UNICODE_STRING {
  USHORT Length;        /* bytes */
  USHORT MaximumLength; /* bytes */
  PWSTR  Buffer;
} UNICODE_STRING, *PUNICODE_STRING;
#endif

typedef const UNICODE_STRING *PCUNICODE_STRING;
{% endhighlight %}

这里的 PWSTR 的定义是 WCHAR*, 即宽字符指针, 查看WCHAR的定义后知, WCHAR为 (unsigned short) 型(16bit char)

<div class="divider"></div>

### 初始化WCHAR 的时候要用下面这种形式进行初始化:

{% highlight C %}
WCHAR sample_1[] = L"Example";
WCHAR sample_2[] = {'E', 'x', 'a', 'm', 'p', 'l', 'e', 0};
{% endhighlight %}

一定要注意第二种形式最后的 `0`

<div class="divider"></div>

### WCHAR, char 之间的转换

二者之间的转换可以通过两个函数来完成, 先将char的字符串转为 `ANSI_STRING`, 然后将`ANSI_STRING`,转为`UNICODE_STRING`, 从而`UNICODE_STRING`中的`Buffer`即为转换后的结果

参考Sebastian Lackner 的[Sample](http://pastebin.com/raw/m43tkLdT)

通过 `RtlInitAnsiString` 将 `filename` 的内容写入一个`ANSI_STRING` `dospath` 内, 然后调用 `RtlAnsiStringToUnicodeString` 并且设置 `AllocateDestinationString` 为 TRUE, 让这个函数自行分配内存给dospathW, 要注意这种方式分配的内存要用 `RtlFreeUnicodeString` 手动回收

<div class="divider"></div>

### Wine 下打印WCHAR

使用函数 wine_dbgstr_* 系列函数, 这里使用 wine_dbgstr_w, 打印宽字符

### *防止内存泄露*

* 使用RtlInitAnsiString创建的字符串不需要手动Free
* 使用RtlAnsiStringToUnicodeString, 并且 AllocateDestinationString 为 TRUE 的话需要手动释放内存
* 使用wine_nt_to_unix_filename 得到的ANSI_STRING 需要手动释放内存


