---
title: proxychains-ng 原理解析
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [August 14, 2018](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html "5:04 pm") 
[VOID001](https://web.archive.org/web/20210514123224/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [12 comments](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comments)





Preface
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


提起 proxychains 相信大家都并不陌生，这个程序可以方便的让你在终端使用 SOCKS5, SOCKS4, HTTP 等协议代理网络访问，而不需要为了转换 SOCKS5 协议再搭建一个 HTTP 的代理来使用 http\_proxy, https\_proxy 这些 Shell 内置的环境变量来访问网络了。不过 proxychains 并不对所有的应用程序有效，一个典型的情况是 Golang 编写的 程序是无法使用 proxychains 进行代理的。在使用 proxychains 的时候会报这样的错误:




```
dial tcp 224.0.0.1:80: connect: network is unreachable
```

下面就通过对 proxychains-ng 的原理的解析，来解答这个问题，并且为 golang 编写的程序提供一个解决方案。


Shared Libraries
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


Linux 下的很多程序都依赖着多种多样的**动态链接库(shared library)**，使用动态链接库既可以节省磁盘的空间大小（你编译出来的程序不会特别大），同时也会节省程序的运行内存，多个共享动态链接库的进程只需要一份库在内存中。若是静态链接的话，则每一个进程都要带一份库。通过 ls -l /usr/lib (根据发行版不同路径可能会有不同)即可看到很多动态链接库。


首先来介绍几个动态链接库的基本知识，大家会发现这个文件夹下面有很多链接，比如




```
lrwxrwxrwx   1 root root        19 Aug  7 00:22 libzmf-0.0.so -> libzmf-0.0.so.0.0.2                                                                                                                         
lrwxrwxrwx   1 root root        19 Aug  7 00:22 libzmf-0.0.so.0 -> libzmf-0.0.so.0.0.2
```

有两个指向 libzmf-0.0.so.0.0.2 的软连接这些文件的名字很相似，那么具体都代表什么呢，下面就来进行说明。


对于一个动态链接库来说，有三个名字，分别是 soname, linkername 和 realname


* linkername: libxxx.so (没有任何版本号) 在安装 library 的时候建立，是一个链接到 realname 的软链接
* soname: libxxx.so.(VER) (带有版本号) 在安装 library 的时候建立，是一个链接到 realname 的软链接
* realname: libxxx.so.(VER).(MINOR).[RELEASE] (必须带有版本号和 minor number, 可选的为带有 release number) 是该 library 本身


对于上面这个例子来说 libzmf-0.0 的 soname 就是 libzmf-0.0.so.0， linkername 是 libzmf-0.0.so，realname 是 libzmf-0.0.so.0.0.2


当一个程序指定要链接的动态链接库的时候，他们指定的是这个链接库的 soname, 而不是 realname 这样的考量是在链接库更新 minor number 的时候，不需要对这个程序进行重新链接，至于为什么没有用 linkername 是为了 ABI 兼容性考虑，当一个库升级后 ABI 发生了变化时，依赖这个库的程序必须要重新编译才能使用，否则就会因为 ABI 不兼容导致段错误等问题发生。因而当一个库的 MAJOR VER NUMBER 更新时，说明它有 ABI Breaking Change. 而当一个库只是更新了 MINOR/RELEASE NUMBER 的时候 这时我们不需要进行重新编译。


Dynamic Loading Progress
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


本文重点在于讲解 proxychains 的原理，因而对 loader 部分只提及相关部分，下述过程并不是完整的程序加载过程


在 Linux 上所有动态链接的程序都会链接一个 ld-linux-xxxx.so(下面简称 ld-linux.so) 的动态链接库，这个动态链接库很特殊，它会解析该程序所需的 shared libraires ，并且加载他们以及他们必要的依赖 我们可以通过查看每一个动态链接的程序的 Dynamic Section 了解到其依赖的链接库都是什么。比如这是 curl **直接依赖**的动态链接库:




```
╰─(´・ω・)つ  readelf -a /usr/bin/curl | grep NEEDED
 0x0000000000000001 (NEEDED)             Shared library: [libcurl.so.4]
 0x0000000000000001 (NEEDED)             Shared library: [libpthread.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```

注意这些只是 “直接依赖”, ld-linux.so 还会去解析这些依赖的 library 的依赖是什么，最后得到我们通过 ldd 看到的输出结果




```
╰─(´・ω・)つ  ldd /usr/bin/curl                                                                                                                                                                          1 ↵
        linux-vdso.so.1 (0x00007fffb9f6a000)
        libcurl.so.4 => /usr/lib/libcurl.so.4 (0x00007fd1cbbd6000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007fd1cbbb5000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007fd1cb9f1000)
        libnghttp2.so.14 => /usr/lib/libnghttp2.so.14 (0x00007fd1cb7cc000)
        libidn2.so.0 => /usr/lib/libidn2.so.0 (0x00007fd1cb5af000)
        libpsl.so.5 => /usr/lib/libpsl.so.5 (0x00007fd1cb39f000)
        libssl.so.1.1 => /usr/lib/libssl.so.1.1 (0x00007fd1cb133000)
        libcrypto.so.1.1 => /usr/lib/libcrypto.so.1.1 (0x00007fd1cacb6000)
        libgssapi\_krb5.so.2 => /usr/lib/libgssapi\_krb5.so.2 (0x00007fd1caa68000)
        libkrb5.so.3 => /usr/lib/libkrb5.so.3 (0x00007fd1ca77f000)
        libk5crypto.so.3 => /usr/lib/libk5crypto.so.3 (0x00007fd1ca54c000)
        libcom\_err.so.2 => /usr/lib/libcom\_err.so.2 (0x00007fd1ca348000)
        libz.so.1 => /usr/lib/libz.so.1 (0x00007fd1ca12f000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fd1cc0e0000)
        libunistring.so.2 => /usr/lib/libunistring.so.2 (0x00007fd1c9daf000)
        libdl.so.2 => /usr/lib/libdl.so.2 (0x00007fd1c9daa000)
        libkrb5support.so.0 => /usr/lib/libkrb5support.so.0 (0x00007fd1c9b9d000)
        libkeyutils.so.1 => /usr/lib/libkeyutils.so.1 (0x00007fd1c9999000)
        libresolv.so.2 => /usr/lib/libresolv.so.2 (0x00007fd1c997e000)

```

对于每一个 library 是如何解析到其路径的具体过程，可以通过查看 man 8 ld-linux  了解具体过程


Special Environment Variable: LD\_PRELOAD
-----------------------------------------


在 ld-linux(8) 的 Man Page 里 我们可以看到这样一个环境变量的说明: LD\_PRELOAD



> A list of additional, user-specified, ELF shared objects to be loaded before all others.
> 
> 


当 secure-execution 模式没有开启的时候 指定 在 LD\_PRELOAD 里的 shared library 会比其他任何 shared libray 都先加载. 这就给我们去伪造, hook 调用函数提供了途径。


背景知识铺垫到这里就结束了，接下来我们将结合 proxychains-ng 的代码介绍其原理


Proxychains-ng 的原理
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


简单来说, proxychains-ng 就是 hook 了 libc 里提供的基本网络通讯函数




```
//╰─(´・ω・)つ  cat libproxychains.c  | grep SETUP
#define SETUP\_SYM(X) do { if (! true\_ ## X ) true\_ ## X = load\_sym( # X, X ); } while(0)
        SETUP\_SYM(connect);
        SETUP\_SYM(sendto);
        SETUP\_SYM(gethostbyname);
        SETUP\_SYM(getaddrinfo);
        SETUP\_SYM(freeaddrinfo);
        SETUP\_SYM(gethostbyaddr);
        SETUP\_SYM(getnameinfo);
        SETUP\_SYM(close);
        SETUP\_SYM(\_\_xnet\_connect);

```

这些包在 SETUP\_SYM 里的函数(在 SOLARIS 中 connect 是 \_\_xnet\_connect)会被 proxychains 进行 hook， 然后通过内置的 hook 函数进行后续代理操作。


我们查看 [src/libproxychains.c](https://web.archive.org/web/20210514123224/https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/libproxychains.c#L442) 可以发现，这个 libproxychains.c 含有 connect, sendto, … 这些函数, 而且函数的签名和 connect(3) sendto(3)… 的都一样


这就是 proxychains 的原理所在，proxychains 将这些函数重写一份，并且 export libproxychains 为 shared library. 当该 Library 被 preload (设置在 LD\_PRELOAD) 里的时候，则在程序调用 connect , close 等网络相关的 libc 函数的时候，就会被 proxychains 接管。


我们在代码里还能看到很多的 true\_xxx 函数, 他们只有函数调用没有定义, 在 src/core.h 中定义这些符号从外部引用




```
// src/core.h
extern connect\_t true\_connect;

```

为了进一步理解 proxychains 我们需要弄清楚 这个 true\_xxx 从何而来, 因为这些函数被钩子函数们屡次调用。我们现在就回到 SETUP\_SYM 这个宏的定义上来


SETUP\_SYM 这个宏就是 true\_xxx 系列函数的解析的关键




```
#define SETUP\_SYM(X) do { if (! true\_ ## X ) true\_ ## X = load\_sym( # X, X ); } while(0)

```

我们以 connect 为例，展开一下这个宏: SETUP\_SYM(connect) 被展开为




```
 do { if (! true\_connect ) true\_connect = load\_sym( "connect", connect ); } while(0);

```

这里 宏 invoke 了 load\_sym 函数, 该函数[如下](https://web.archive.org/web/20210514123224/https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/libproxychains.c#L83) :




```
static void* load\_sym(char* symname, void* proxyfunc) {

	void *funcptr = dlsym(RTLD\_NEXT, symname);

	if(!funcptr) {
		fprintf(stderr, "Cannot load symbol '%s' %s\n", symname, dlerror());
		exit(1);
	} else {
		PDEBUG("loaded symbol '%s'" " real addr %p  wrapped addr %p\n", symname, funcptr, proxyfunc);
	}
	if(funcptr The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog) proxyfunc) {
		PDEBUG("circular reference detected, aborting!\n");
		abort();
	}
	return funcptr;
}
```

load\_sym 调用了 dlsym 并且将 dlsym 返回值返回，然后通过上面的宏我们就知道 true\_xxx 就会得到这个返回的地址 也就是函数地址。另一问题就是，这个返回的地址意味什么？相信很多人已经猜到了，true\_xxx 这些函数就应该是指向那些没有被 hook 的原始网络函数的。我们现在查看 dlsym 的具体调用的含义。通过 dlsym(3) 我们知道了， dlsym 的两个参数分别为 dlopen 打开的 handle, 以及要解析的 symbol name。而  RTLD\_NEXT 和 RTLD\_DEFAULT 是两个 pseudo-handle。我们这里贴一下 RTLD\_NEXT 的解释的全文



> RTLD\_NEXT  
> 
> Find the next occurrence of the desired symbol in the search order after the current object. This allows one to provide a wrapper around a function in another shared object, so that,  
> 
> for example, the definition of a function in a preloaded shared object (see LD\_PRELOAD in ld.so(8)) can find and invoke the “real” function provided in another shared object (or for  
> 
> that matter, the “next” definition of the function in cases where there are multiple layers of preloading).
> 
> 


可以知道，这个 pseudo-handle 会通过解析当前的 library search path 找到 **第二个**symbol name 等于 symname 的函数，manpage 里还贴心的给出了一个应用场景，就是在这种 LD\_PRELOAD 的情况下想要加载 “real” 函数的时候，这样可以方便的进行加载。


我们再来查看一下 hooked connect 函数的[具体逻辑](https://web.archive.org/web/20210514123224/https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/libproxychains.c#L459)




```
	if(!((fam  The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog) AF\_INET || fam The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog) AF\_INET6) && socktype The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog) SOCK\_STREAM))
		return true\_connect(sock, addr, len);
```

通过这个判断可以看到，当该链接不满足 TCP 链接的条件的时候，是会去调用 libc 的 connect 函数继续下去


[这里](https://web.archive.org/web/20210514123224/https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/libproxychains.c#L499)




```
	ret = connect\_proxy\_chain(sock,
				  dest\_ip,
				  htons(port),
				  proxychains\_pd, proxychains\_proxy\_count, proxychains\_ct, proxychains\_max\_chain);
```

就是 proxychains 将链接转到了自己的 SOCKS 链接逻辑里的调用。这之后的一切就随 proxychains 操作了。


看到这里相信大家对 proxychains 如何做到让其他程序能够代理链接有一定认识了。那么还有一个小问题没有解答，在 ArchLinux 和其他一些发行版上使用 proxychains 的时候我们也没有手动设置 LD\_PRELOAD 这个环境变量，他是如何被设置的呢? 这里我们只需要去看 [https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/main.c#L139](https://web.archive.org/web/20210514123224/https://github.com/rofl0r/proxychains-ng/blob/1c8f8e4e7e31e64131f5f5e031f216b557f7b5ed/src/main.c#L139)




```
#define LD\_PRELOAD\_ENV "LD\_PRELOAD"
/* all historic implementations of BSD and linux dynlinkers seem to support
   space as LD\_PRELOAD separator, with colon added only recently.
   we use the old syntax for maximum compat */
#define LD\_PRELOAD\_SEP " "
#endif
	char *old\_val = getenv(LD\_PRELOAD\_ENV);
	snprintf(buf, sizeof(buf), LD\_PRELOAD\_ENV "=%s/%s%s%s",
	         prefix, dll\_name,
	         /* append previous LD\_PRELOAD content, if existent */
	         old\_val ? LD\_PRELOAD\_SEP : "",
	         old\_val ? old\_val : "");
	putenv(buf);
	execvp(argv[start\_argv], &argv[start\_argv]);
	perror("proxychains can't load process....");
```

这里通过 putenv 设置了 LD\_PRELOAD 的环境变量，然后执行了 execvp 调用命令行后面指定的程序。


通过上述 code reading 我们可以得出结论: **proxychains 是通过 LD\_PRELOAD 让自己在其他所有 shared library 之前被解析, 并导出 libc 的网络功能函数 connect, close, sendto, … 等函数, 通过此方法 hook libc API , 来达到让其他的程序能够通过其进行 SOCKS5 proxy 访问的效果**


那么我们来尝试一下吧~ 我们来 hook 一下 open 函数看看会出现什么事情




```
*
 * Try to hook open function and disguise the system
 *
 */

#include <stdio.h>
#include <errno.h>
#include <sys/socket.h>

int open(const char *path, int oflag, ...) {
    printf("Hooked open!\n");
    return 1;
}

```

我们使用以下参数进行编译




```
gcc -fPIE -c open.c && gcc -shared -o libopen.so open.o

```

我们让 open 恒定返回 fd = 0 即为进程默认打开的标准输入 (/dev/pts) 我们执行 LD\_PRELOAD=./libopen.so cat /usr/bin/vim 程序先是输出了一行 “Hooked open” 然后就 block 在了那里，好像在等待读入输入一样，而这个操作的原始行为应该是 cat 出来 /usr/bin/vim 这个 binary file 然后导致 terminal 乱码(逃 因而我们现在可以说，我们成功的 hook 了 libc 的函数 \w/ 感兴趣的朋友可以试试把上述代码的返回值修改为大于2的值，然后看看会发生什么。


Why Some Programs (e.g. Golang) Cannot Use It?
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


通过上文我们知道了，很多的 golang 程序都是静态链接的程序，当然不涉及到任何 shared library preload, 对于这些程序来说我们没有办法让他们使用 proxychains.


但是最初的这个奇怪的报错是什么?




```
dial tcp 224.0.0.1:80: connect: network is unreachable
```

我们 grep 224 发现这个的确出现在了 proxychains-ng 的代码中，而且还是一个 DNS 相关的变量, 我们可以猜测 对于涉及网络请求的golang程序，可能有一部分函数被 proxychains hook 了(Thanks to [@Equim](https://web.archive.org/web/20210514123224/https://ekyu.moe/))。为了验证我们的猜想，我们给 proxychains 的每一个 Hook 的函数加上调试输出，下面放出一个 demo 程序，分别使用 golang 的 http.Get 和 net.Dial 两个方式向 myip.ipip.net 请求自己的 IP 地址




```
package main

import (
	"bufio"
	"fmt"
	"io/ioutil"
	"net"
	"net/http"
)

func \_dial() {
	conn, err := net.Dial("tcp", "myip.ipip.net:80")
	if err != nil {
		fmt.Printf("dial error: %s\n", err)
		return
	}

	reqstr := "GET / HTTP/1.1\r\nHost: myip.ipip.net\r\nUser-Agent: curl/7.61.0\r\nAccept: */*\r\n\r\n"
	\_, err = fmt.Fprintf(conn, reqstr)
	if err != nil {
		fmt.Printf("read error: %s\n", err)
		return
	}
	b := make([]byte, 100000)
	\_, err = bufio.NewReader(conn).Read(b)
	if err != nil {
		fmt.Printf("read error: %s\n", err)
		return
	}

	fmt.Printf("%s\n", b)
}

func \_http() {
	resp, err := http.Get("http://myip.ipip.net")
	if err != nil {
		fmt.Printf("http.get: %s", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("readall: %s", err)
		return
	}
	fmt.Printf("%s", body)
}

func main() {
	println("DIAL")
	\_dial()
	println("HTTP")
	\_http()
}

```

我们使用 go build 上面的代码 -> demo 然后 通过增加了调试信息的 Proxychains 执行 demo 得到如下输出:




```
╰─(´・ω・)つ  ./proxychains4 -q ~/playground/go/go\_connect/go\_connect\_gc
DIAL
DBG: getaddrinfo
...
DBG: freeaddrinfo
dial error: dial tcp 224.0.0.1:80: connect: network is unreachable
HTTP
DBG: getaddrinfo
...
DBG: freeaddrinfo
http.get: Get http://myip.ipip.net: dial tcp 224.0.0.1:80: connect: network is unreachable%
```

我们发现，这个 demo 程序的 getaddrinfo 和 freeaddrinfo 被 hook 到了 proxychains 其他函数没有。因而我们的猜想得到了验证，具体可以去参考 golang source code (这里暂时不进行讨论)


A Way For Golang Programs to Use Proxychains
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


这个问题的答案就是使用 [gccgo](https://web.archive.org/web/20210514123224/https://golang.org/doc/install/gccgo)


我们先来看效果


不使用 proxychains 直接运行 编译得到的 demo




```
DIAL
HTTP/1.1 200 OK
Date: Tue, 14 Aug 2018 13:09:06 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 67
Connection: keep-alive
X-Via-JSL: dca9b80,-
Set-Cookie: \_\_jsluid=xxxxxxxxxxxxxxxxxxxxxxxxxxx; max-age=31536000; path=/; HttpOnly
X-Cache: bypass

当前 IP：*.*.*.*  来自于：中国 XXXXXXXXXXXXXXXXXXX

HTTP
当前 IP：*.*.*.*  来自于：中国 XXXXXXXXXXXXXXXXXXX

```

使用 proxychains 代理后




```
DIAL
HTTP/1.1 200 OK
Date: Tue, 14 Aug 2018 13:10:09 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 73
Connection: keep-alive
Set-Cookie: \_\_cfduid=xxxxxxxxxxxxxxxxxxxxxxx; expires=Wed, 14-Aug-19 13:10:09 GMT; path=/; domain=.ipip.net; HttpOnly
Server: cloudflare
CF-RAY: 4xxxxxxxxxxxxxxx-NRT

当前 IP：*.*.*.*  来自于：日本  XXXXXXXXXXXXXXXXXX

HTTP 
当前 IP：*.*.*.*  来自于：日本 XXXXXXXXXXXXXXXXXX
```

可以看出，这次 proxychains 生效了! Hooray


那么为什么生效了呢?


我们对比两个不同 compiler 编译出来的 go binary 的 shared library 可以发现




```
╰─(´・ω・)つ  ldd go\_connect\_shared go\_connect\_gc 
go\_connect\_shared:
        linux-vdso.so.1 (0x00007ffdcdf00000)
        libgo.so.13 => /usr/lib/libgo.so.13 (0x00007f368872e000)
        libm.so.6 => /usr/lib/libm.so.6 (0x00007f36885a9000)
        libgcc\_s.so.1 => /usr/lib/libgcc\_s.so.1 (0x00007f368858f000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007f36883cb000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007f3689fe6000)
        libz.so.1 => /usr/lib/libz.so.1 (0x00007f36881b4000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007f3688193000)
go\_connect\_gc:
        linux-vdso.so.1 (0x00007ffe4bf5f000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007fb99f7d6000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007fb99f612000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fb99f858000)

```

他们两个都 link 了 libc(内含有 connect 函数) 为什么一个会接受 proxy 一个不能呢? 猜测可以是, connect 函数在 gc compiler 版的 go 中调用的不是 libc 的 connect, 而在 gccgo 里则是调用了这个 我们需要通过阅读源码来弄清楚 connect 函数是来自哪里的。


GC(Go Compiler)下的调用
-------------------


我们先来看 connect 在 gc (我们熟知的默认 go compiler) 下 connect 的调用链路:


在 <go\_src>/src/syscall 下有一系列的 syscall 文件, 对于 Linux 64 bit 我们仅需要看 src/syscall/syscall\_linux\_amd64.go 这个文件，这里我们发现了




```
//sys	Statfs(path string, buf *Statfs\_t) (err error)
//sys	SyncFileRange(fd int, off int64, n int64, flags int) (err error)
//sys	Truncate(path string, length int64) (err error)
//sys	accept(s int, rsa *RawSockaddrAny, addrlen *\_Socklen) (fd int, err error)
//sys	accept4(s int, rsa *RawSockaddrAny, addrlen *\_Socklen, flags int) (fd int, err error)
//sys	bind(s int, addr unsafe.Pointer, addrlen \_Socklen) (err error)
//sys	connect(s int, addr unsafe.Pointer, addrlen \_Socklen) (err error)
//sys	fstatat(fd int, path string, stat *Stat\_t, flags int) (err error) = SYS\_NEWFSTATAT
//sysnb	getgroups(n int, list *\_Gid\_t) (nn int, err error)
//sysnb	setgroups(n int, list *\_Gid\_t) (err error)
//sys	getsockopt(s int, level int, name int, val unsafe.Pointer, vallen *\_Socklen) (err error)
//sys	setsockopt(s int, level int, name int, val unsafe.Pointer, vallen uintptr) (err error)
//sysnb	socket(domain int, typ int, proto int) (fd int, err error)
//sysnb	socketpair(domain int, typ int, proto int, fd *[2]int32) (err error)
//sysnb	getpeername(fd int, rsa *RawSockaddrAny, addrlen *\_Socklen) (err error)
//sysnb	getsockname(fd int, rsa *RawSockaddrAny, addrlen *\_Socklen) (err error)
//sys	recvfrom(fd int, p []byte, flags int, from *RawSockaddrAny, fromlen *\_Socklen) (n int, err error)
```

这样一段含有 connect 的签名的注释，每一行的 //sys //sysnb 会被 perl 脚本 src/syscall/mksyscall.pl 给展开， connect 展开后是这样的




```
func connect(s int, addr unsafe.Pointer, addrlen \_Socklen) (err error) {
        \_, \_, e1 := Syscall(SYS\_CONNECT, uintptr(s), uintptr(addr), uintptr(addrlen))
        if e1 != 0 {
                err = errnoErr(e1)
        }
        return
}
```

查看 Syscall 的实现 我们跟踪到了 src/syscall/asm\_linux\_amd64.s 内的代码




```
TEXT ·Syscall(SB),NOSPLIT,$0-56
	CALL	runtime·entersyscall(SB)
	MOVQ	a1+8(FP), DI
	MOVQ	a2+16(FP), SI
	MOVQ	a3+24(FP), DX
	MOVQ	$0, R10
	MOVQ	$0, R8
	MOVQ	$0, R9
	MOVQ	trap+0(FP), AX	// syscall entry
	SYSCALL
	CMPQ	AX, $0xfffffffffffff001
	JLS	ok
	MOVQ	$-1, r1+32(FP)
	MOVQ	$0, r2+40(FP)
	NEGQ	AX
	MOVQ	AX, err+48(FP)
	CALL	runtime·exitsyscall(SB)
	RET
ok:
	MOVQ	AX, r1+32(FP)
	MOVQ	DX, r2+40(FP)
	MOVQ	$0, err+48(FP)
	CALL	runtime·exitsyscall(SB)
	RET
```

可以看到这里是通过直接调用 syscall 进行了系统调用，而非使用了 libc 提供的 connect 函数。因而我们在这种情况下是无法让 connect 被 proxychains 给 hook 的


GCCGO 下的调用
----------


我们再来看一下 gccgo 对 connect 的调用链路:


在 gccgo/libgo/go/syscall 下我们查看文件 socket\_posix.go 可以看到




```
//sys   bind(fd int, sa *RawSockaddrAny, len Socklen\_t) (err error)
//bind(fd \_C\_int, sa *RawSockaddrAny, len Socklen\_t) \_C\_int

//sys   connect(s int, addr *RawSockaddrAny, addrlen Socklen\_t) (err error)
//connect(s \_C\_int, addr *RawSockaddrAny, addrlen Socklen\_t) \_C\_int

```

同样，这段代码也会被一个 mksyscall.awk 的宏展开为:




```
// Automatically generated wrapper for connect/connect
//extern connect
func c\_connect(s \_C\_int, addr *RawSockaddrAny, addrlen Socklen\_t) \_C\_int
func connect(s int, addr *RawSockaddrAny, addrlen Socklen\_t) (err error) {
        Entersyscall()
        \_r := c\_connect(\_C\_int(s), addr, Socklen\_t(addrlen))
        var errno Errno
        setErrno := false
        if \_r < 0 {
                errno = GetErrno()
                setErrno = true
        }
        Exitsyscall()
        if setErrno {
                err = errno
        }
        return
}

```

我们可以看到, 这里使用了 extern directive 将函数 c\_connect 引用指向了外部的 connect 符号,  通过查看 libgo 的依赖关系(ldd) 我们发现 libgo 依赖 libc, libc 提供了 connect 因而 gccgo 编译出来的程序的 connect 是通过 libc 调用，而不是内部自行解决了，所以我们可以通过 proxychains 来进行 hook。


验证
--


最后再来验证一下我们的这个结论。  对于查看 shared lib 的相关内部过程，可以用一个神奇的环境变量 LD\_DEBUG, 我们使用 LD\_DEBUG=bindings 来展示出每一个符号的 bind 过程，查看两个不同的 go compiler 编译出的程序在 symbol resolution 时有什么不同。（都已经使用 LD\_PRELOAD preload 了 libproxychains4.so )


![](https://web.archive.org/web/20210514123224im_/https://void-shana.moe/wp-content/uploads/2018/08/gcc-1-1024x292.png)For GCCGO
![](https://web.archive.org/web/20210514123224im_/https://void-shana.moe/wp-content/uploads/2018/08/gc-1-1024x374.png)For GC
我们可以看出，在 gccgo 编译的版本中， libgo 需要的外部 symbol connect 被 bind 到了 GLIBC connect 而在 gc 编译的版本中则不存在这样的 binding. 因而我们得出结论 gccgo 编译的代码可以被 proxychain hook


Reference
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


* ld-linux man page
* dlsym(3) man page
* http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html
* https://github.com/rofl0r/proxychains-ng


Misc
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)


遗留问题: 在查看 LD\_DEBUG=bindings 的时候，我们可以看到这样一段奇怪的 bind: binding file /usr/lib/libproxychains4.so [0] to /usr/lib/libpthread.so.0 [0]: normal symbol `connect’ proxychains 的 connect 竟然 bind 到了 libpthread 上，这让我很费解，尝试了 LD\_DEBUG 看 curl 的 binding 也是最后到了 libpthread 上，这里就让我产生了一个悬而未解的问题: 莫非 proxychains 不仅仅 hook 了 libc 还 hook 了 libpthread? 我查看了 LD\_DEBUG=1 curl xxx.cn 的输出，发现所有的 connect symbol 都是解析到了 libpthread.so 上，也许，所有的 connect 都没有走 libc 而是走了 libpthread?z 这就是另一个问题了。


后记: 因为最近在准备出国留学，备考申请事情多得很忙，几乎没有多少时间来研究技术，因而博客搁置的时间远比 3 个月长，甚至有的朋友留言说博主已经凉了，在此对大家的关注表示感谢，因为很多事情导致了博客没有更新。在备考结束后博客还会保持以前的进度持续更新的。同时感谢 [@Equim](https://web.archive.org/web/20210514123224/https://ekyu.moe/) 在撰写本文时提供的种种帮助，关于她说的另一个问题我在这里没有讲到，也是和 proxychains 有关的一个问题，链接在这里: [https://hackerone.com/reports/361269](https://web.archive.org/web/20210514123224/https://hackerone.com/reports/361269) 感兴趣的各位可以去看看


然后我就要继续去准备 GRE 了(x






---


[C](https://web.archive.org/web/20210514123224/https://void-shana.moe/category/linux/c), [Kernel](https://web.archive.org/web/20210514123224/https://void-shana.moe/category/linux/kernel-linux), [Linux](https://web.archive.org/web/20210514123224/https://void-shana.moe/category/linux) [C](https://web.archive.org/web/20210514123224/https://void-shana.moe/tag/c), [kernel](https://web.archive.org/web/20210514123224/https://void-shana.moe/tag/kernel), [linux](https://web.archive.org/web/20210514123224/https://void-shana.moe/tag/linux) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/13b998737e96dd10135326cacc26985c?s=50&d=identicon&r=g) **ZackXu** says: 
[August 14, 2018 at 5:07 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-443)
娜娜更新了！


	1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[August 15, 2018 at 10:20 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-446)
	终于更新了(x


2. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/1896080d53de450a79795654ceac2dbb?s=50&d=identicon&r=g) **[Neo\_Chen](https://web.archive.org/web/20210514123224/https://thunix.org/~neo_chen/)** says: 
[August 14, 2018 at 8:53 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-444)
終於更新了，耶！


	1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[August 15, 2018 at 10:21 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-447)
	最近好忙QvQ


3. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/e466883dea9a8b501f9788066cd021de?s=50&d=identicon&r=g) **[kookxiang](https://web.archive.org/web/20210514123224/https://ikk.me/)** says: 
[August 15, 2018 at 9:59 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-445)
夏娜大佬！跪了


	1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[August 15, 2018 at 10:22 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-448)
	kk 才是大佬! 我是萌新!


4. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[August 15, 2018 at 10:50 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-450)
现在不是草稿了呀，是因为当时手残点错了(


5. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/64718fe14af5b212ad2428ce59affcb2?s=50&d=identicon&r=g) **[灰灰](https://web.archive.org/web/20210514123224/https://huihui.moe/)** says: 
[August 20, 2018 at 12:53 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-461)
好耶！博主热了！


	1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[August 20, 2018 at 12:57 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-462)
	好耶是灰灰! 扑(


6. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/43fdc497850cead58be4e6219c7d96e8?s=50&d=identicon&r=g) **[a-wing](https://web.archive.org/web/20210514123224/https://a-wing.top/)** says: 
[September 27, 2018 at 9:11 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-489)
夏娜姐姐好棒～


	1. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[September 27, 2018 at 9:20 am](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-490)
	好耶，是夏娜妹妹/


7. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/3e3c1cb94b9ad3551cd9d43f2e52e105?s=50&d=identicon&r=g) **[依云](https://web.archive.org/web/20210514123224/https://blog.lilydjwg.me/)** says: 
[January 31, 2019 at 10:37 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-517)
libpthread.so 确实包括了一份 connect，以及其他二三十个常见函数。在 glibc 的 glibc/nptl/Makefile 文件中可以看到，这些函数是为了兼容性才存在于 libpthread.so 里的。


7. ![](https://web.archive.org/web/20210514123224im_/https://secure.gravatar.com/avatar/3e3c1cb94b9ad3551cd9d43f2e52e105?s=50&d=identicon&r=g) **[依云](https://web.archive.org/web/20210514123224/https://blog.lilydjwg.me/)** says: 
[January 31, 2019 at 10:37 pm](https://web.archive.org/web/20210514123224/https://void-shana.moe/linux/proxychains-ng.html#comment-517)
libpthread.so 确实包括了一份 connect，以及其他二三十个常见函数。在 glibc 的 glibc/nptl/Makefile 文件中可以看到，这些函数是为了兼容性才存在于 libpthread.so 里的。

            