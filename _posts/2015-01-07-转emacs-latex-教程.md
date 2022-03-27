---
title: (转)Emacs+ Latex 教程
render_with_liquid: false
---
==================



#####  [January 7, 2015](https://web.archive.org/web/20190506155039/https://void-shana.moe/linux/%e8%bd%acemacs-latex-%e6%95%99%e7%a8%8b.html "2:19 pm") 
[VOID001](https://web.archive.org/web/20190506155039/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20190506155039/https://void-shana.moe/linux/%e8%bd%acemacs-latex-%e6%95%99%e7%a8%8b.html#respond)






Emacs + LaTeX 快速上手(原网址 http://cs2.swfc.edu.cn/~wx672/lecture\_notes/linux/latex/latex\_tutorial.html)
=====================================================================================================


* 本教程完全针对本校D215机房Ubuntu系统中的Emacs和LaTeX配置。关于如何配置，请看[这里](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/install.html)。
* 本教程中涉及的LaTeX源文件和图片都可以在[这里](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/)找到。



目录
--




1 Emacs + AucTeX，60分钟快速入门
-------------------------




为什么非要推荐LaTeX？
这完全是出于个人喜好。从1996年开始接触计算机到现在，Windows、UNIX、MS-Word、LaTeX 我都用过了。我觉得我该把我认为优秀的东西推荐给你。即使你不感兴趣，但做为计科专业的学生你应该知道它的存在。
为什么非要推荐Emacs？
这不仅是出于个人喜好，也不仅是因为Emacs是最优秀的编辑器，我最基本的目的是，希望每一个计科专业的学生能熟练使用键盘。



### 1.1 放松心情



LaTeX很强大，但对于初学者来说，你不必关心它有多强大，因为最为常用的命令和环境不过就是那么几个。而且你也不必手工输入这些命令，只要你用Emacs+AucTeX，几个简单的快捷键就足以满足你的基本需求了。对于格式复杂的需求，通常你只要套用模版就可以解决问题了。所以，大家只要把Emacs用熟，一切迎刃而解。





### 1.2 用LaTeX写文章就是在编程



我们先回忆一下用Emacs写一个 `hello.c` 的过程:


1. 打开Emacs；
2. 开始编辑一个新文件，名字叫 `hello.c`:

```
C-x C-f
```

在Emacs窗口的最下面（也就是 `mini buffer` 里）写上新文件的名字 `hello.c`:



```
hello.c
C-j
```
3. 向文件里写东西:

```
#include <stdio.h>
int main(int argc, char *argv[])
{
  printf ("Hello, world!n");
  return 0;
}
```

保存：



```
C-x C-s
```

编译：



```
gcc hello.c
```

运行：



```
./a.out
```


再来看一下用Emacs写一个 `hello.tex` 的过程：


1. 打开Emacs；
2. 开始编辑一个新文件，名字叫 `hello.tex`:

```
C-x C-f
```

在Emacs窗口的最下面（也就是 `mini buffer` 里）写上新文件的名字 `hello.tex`:



```
hello.tex
C-j
C-j
```
3. 向文件里写东西:

```
documentclass{article}
begin{document}
Hello, world!
end{document}
```

保存：



```
C-x C-s
```

编译：



```
xelatex hello.tex
```

看结果：



```
evince hello.pdf
```


怎么样？ `hello.c` 和 `hello.tex` 的编辑过程没什么分别吧。只要把Emacs用熟练，不管写什么程序，都是这么个过程。你


1. 不必学习VC去写C/C++，
2. 不必学习eclipse去写Java，
3. 不必学习MS-Word去写报告、幻灯片，
4. 不必学习……


一句话，“Everything Emacs”，可以省下大量不必要的学习时间。人生苦短，何必让你的生活被 `VC/eclipse/MS-Word` 搞得头昏脑胀呢？ **简单而强大，本就是计科专业学生和非专业学生应有的不同** 。


如果你对Emacs操作还很陌生，那么现在就打开Emacs



```
C-h t
```

重温一下那些基本操作吧。




#### 1.2.1 什么是 `C-x C-f` ？



这么说，


1. 把你的双手在标准键盘上放好，
2. 左手小指稍向左移，按在 `caps lock` 键上。按住别动。（D215机房的 `caps lock` 键被我们改成 `Control` 键了）
3. 小指按在 `caps lock` 上别放开，左手无名指稍向下移，在 `x` 键上按一下就放开，这就是 `C-x` 。
4. 小指按在 `caps lock` 上别放开，左手食指在 `f` 键上按一下，这就是 `C-f` 。现在左手各指都可以放开了。


这就是 `C-x C-f` ，作用是要求打开一个文件， `f` 代表 `file` 。那么，告诉我


* 什么是 `C-x C-s` ？
* 什么是 `C-x 2` ？什么是 `C-x 3` ？什么是 `C-x 0` ？什么是 `C-x 1` ？什么是 `C-x o` ？
* 什么是 `C-x h` ？什么是 `C-w` ？
* 什么是 `C-g` ？
* 什么是 `C-j` ？什么是 `C-i` ？
* 什么是 `C-/` ？
* 什么是 `C-k` ？什么是 `C-y` ？
* 什么是 `C-d` ？什么是 `M-d` ？(`M` 代表 `Meta` 键, 也就是 `Alt` 键)
* 什么是 `C-a` ？什么是 `C-e` ？什么是 `C-f` ？什么是 `C-b` ？什么是 `C-n` ？什么是 `C-p` ？


【记住】使用Emacs的时候，一定要忘掉鼠标，两只手静静地放在 `标准键盘` 上!






### 1.3 生活可以更轻松



AucTeX是Emacs的一个功能模块，为LaTeX编程提供了巨大的便利。有了AucTeX，你的LaTeX生活可以象 `Hello, world!` 一样简单。现在就跟着我，手把手地领略一下简单的乐趣吧。


一切当然是从打开Emacs开始：



```
emacs &
```

现在开始编辑一个新文件 `simple.tex`:



```
C-x C-f
```

在Emacs窗口的最下方，也就是 `mini buffer` 里，会有提示，让你输入文件名。输入 `simple.tex` ，然后按 `C-j` 。


如果这时 `mini buffer` 里有如下提示：



```
Master file: (default this file) ...
```

直接按 `C-j` 就可以了。知道了吧, `C-j` 就是我们的“回车键”。如果你的手正放在【标准键盘】上，那么，左手小指向左一偏，按到的正是【Control】键（记得？CapsLock被我们改成Control了）。右手食指下不正是【j】键吗？怎么样，比回车键更方便吧。


现在，开始向 `simple.tex` 文件里写东西了，用AucTeX的方式：



```
C-c C-e
```

`e` 代表 `environment` 。“环境”到底是什么呢？意会吧，用用就明白了。 在 `mini buffer` 里会有提示，



```
Environment type: (default document)
```

这是在问你是不是要写一篇文章啊？你当然该用 `C-j` 来告诉它“是”。


这时， `mini buffer` 又会提示，



```
Document class: (default article)
```

这是在问你是不是要写一篇 `article` 类型的文件啊？还是 `C-j` 。


这时， `mini buffer` 又会提示，



```
Options:
```

这是在问你是否有什么特殊选项啊？用 `C-j` 来告诉它说“不需要”。


现在，你的 `simple.tex` 文件里应该有这么几行东西了：



```
1:  documentclass{article}  % Class有 article, book, report, letter...可供选择
2:  
3:  begin{document} %文章的开始
4:  |
5:  end{document}   %文章的结束
```

在上一节里，你已经会写 `Hello, world!` 了。现在，我们要写点像模像样的东西，当然还是用简单的方式。为了更简单，我们直接套用Andrew Roberts写的 [`simple.tex`](https://web.archive.org/web/20190506155039/http://en.wikibooks.org/wiki/LaTeX/simple.tex) 。我们把注意力放在用Emacs写文章的过程上。


确保你的光标在 `begin{document}` 和 `end{document}` 之间，也就是文章的第 [4](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-simple) 行。然后按



```
C-c C-m
```

这时 `mini buffer` 里会有如下提示：



```
Macro (default ref): 
```

这是系统在等待你输入一个 `Macro` ，说白了就是“命令”。输入：



```
title
C-j
```

这时你的文章多了一行 `title{}` (第[4](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-title)行)变成了这样：



```
1:  documentclass{article}
2:  
3:  begin{document}
4:  title{}
5:  end{document}
```

光标停在 `title{}` 的花括号里。不用说你也知道，该输入文章的标题了。那么就给它一个标题：



```
1:  documentclass{article}
2:  
3:  begin{document}
4:  title{How to Structure a LaTeX{} Document}
5:  end{document}
```

发现了吗？凡是以反斜杠开头的都是命令， `LaTeX{}` 也是。它的唯一作用就是把 LaTeX 这五个字母输出成一副怪样子。


好了，在 `title` 下新起一行。再



```
C-c C-m
```

你肯定知道 `C-c C-m` 是干什么用的了吧？就是要输入一个 `Macro`, 也就是 `LaTeX` 命令。 `mini buffer` 里又会有提示：



```
Macro (default title): 
```

Emacs会把我们上次输入的Macro，也就是title，做为默认值提示出来。不用管它，输入：



```
author
C-j
```

不用我说了吧？在 `author{}` 的花括号里输入作者的名字。当然，也可以把自己的通信地址、email写在里面。就像下面这样：



```
 1:  documentclass{article}
 2:  
 3:  begin{document}
 4:  title{How to Structure a LaTeX{} Document}
 5:  author{Andrew Roberts\
 6:          School of Computing,\
 7:          University of Leeds,\
 8:          Leeds,\
 9:          United Kingdom,\
10:          LS2 1HE\
11:          emph{[[email protected]](/web/20190506155039/https://void-shana.moe/cdn-cgi/l/email-protection)}}
12:  end{document}
```

注意， `\` 代表“强制换行”。


再新起一行，加上日期：



```
C-c C-m
date
C-j
C-c C-m
today
C-j
```

其实，没有 `date{today}` 这一句系统会自动把今天的日期添加上的。而且 `date{}` 里面也不一定非要是当天的日期。


title, author, date 一般被叫做文章的 `top matter` 。


再新起一行，写



```
maketitle
C-j
```

`maketitle` 自然是要排版 `top matter` 了。换句话说，不要标题的话可以省略掉这个命令。现在文章变成了这样：



```
 1:  documentclass{article}
 2:  
 3:  begin{document}
 4:  title{How to Structure a LaTeX{} Document}
 5:  author{Andrew Roberts\
 6:    School of Computing,\
 7:    University of Leeds,\
 8:    Leeds,\
 9:    United Kingdom,\
10:    LS2 1HE\
11:    emph{[[email protected]](/web/20190506155039/https://void-shana.moe/cdn-cgi/l/email-protection)}}
12:  date{today}
13:  maketitle
14:  
15:  end{document}
```

好奇的话，现在可以编译一下，看看PDF文件的效果：



```
C-c C-c
C-j
C-c C-v
```

如果你的 `xpdf` 没出毛病的话，一个PDF文件应该显示在屏幕上了。如果 `xpdf` 不正常，那么就打开命令行，敲：



```
evince simple.pdf &
```

效果还满意吧？保持你的好奇心。在下面的操作中，你随时可以编译一下看看效果。


好了，回到Emacs。现在你的光标应该停在 `maketitle` 的下面一行。我们开始写“摘要”部分。敲键盘



```
C-c C-e
```

前面我们已经见到过 `C-c C-e` 了，就是要开始一个“环境”。接着来， `mini buffer` 里现在又有提示了：



```
Environment type: (default itemize)
```

是在问你要开始那种环境啊？默认是开始 `itemize` 环境，因为它是最常用的环境。但我们现在要写的是“摘要”，告诉它：



```
abstract
C-j
```

`abstract` 就是“摘要”的意思。科技论文都是要有摘要的嘛。于是，你的文章变成了这样:



```
1:  % 此处略去十数行
2:  
3:  maketitle
4:  
5:  begin{abstract} 
6:    |
7:  end{abstract}
8:  
9:  end{document}
```

光标停在 `begin{abstract}` 和 `end{abstract}` 之间(第[6](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-abstract)行)。好，现在往摘要部分里填点东西:



```
 1:  % 此处略去十数行
 2:  
 3:  maketitle
 4:  
 5:  begin{abstract}
 6:  In this article, I shall discuss some of the fundamental topics in
 7:  producing a structured document.  This document itself does not go into
 8:  much depth, but is instead the output of an example of how to implement
 9:  structure. Its LaTeX{} source, when in used with my tutorial
10:  provides all the relevant information.
11:  end{abstract}
12:  
13:  |
```

现在，我们要接着上面的例子，写更多更长的东西了。为了编号方便，文章末尾的 `end{document}` 我也不再写出来了。


好， 按 `C-n` 把光标移到 `end{abstract}` 的[下一行](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-abs-end)。让我们开始文章的第一节：



```
C-c C-s
```

`s` 代表 `section`, “节”的意思。 `mini buffer` 提示：



```
Level: (default section)
```

也就是问你，是不是要起一个新 `section` 啊？没错，我就是要起一个新的章节，于是直接 `C-j` 。


`mini buffer` 又提示：



```
Title:
```

也就是问你，章节标题是……？那就给它个标题吧，就叫“ `Introduction` ”。 `C-j` 之后， `mini buffer` 继续提示：



```
Label: sec:introduction
```

这是在问你，要不要给这个新章节打个标签，比如 `sec:introduction`, 以后也许要索引到它呢？这个暂时无关紧要， `C-j` 就是了。(第[18](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-intro)行)



```
14:  % 此处略去十数行
15:  
16:  end{abstract}
17:  
18:  section{Introduction}
19:  label{sec:introduction}
20:
```

给这一节添加内容：



```
 1:  % 此处略去十数行
 2:  
 3:  section{Introduction}
 4:  label{sec:introduction}
 5:  
 6:  This small document is designed to illustrate how easy it is to create a well structured
 7:  document within LaTeXcitee{lamport94}.  You should quickly be able to see how the article
 8:  looks very professional, despite the content being far from academic.  Titles, section
 9:  headings, justified text, text formatting etc., is all there, and you would be surprised
10:  when you see just how little markup was required to get this output.
11:
```

注意到了吗？在这一节里有一个新命令 `cite{}` (我‘笔误’成了 `citee{}`, 以避开 `org->html` 转换的一个小bug), 这是在引用一个参考文献。先不管它，后面再说。


如法炮制，再添加几个章节：



```
12:  section{Structure}
13:  label{sec:structure}
14:  
15:  One of the great advantages of LaTeX{} is that all it needs to know is
16:  the structure of a document, and then it will take care of the layout
17:  and presentation itself.  So, here we shall begin looking at how exactly
18:  you tell LaTeX{} what it needs to know about your document.
19:  
20:  subsection{Top Matter}
21:  label{sec:top-matter}
22:  
23:  The first thing you normally have is a title of the document, as well as
24:  information about the author and date of publication.  In LaTeX{} terms,
25:  this is all generally referred to as emph{top matter}.
26:  
27:  |
```

注意到 [`emph{}`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-emph) 了吗？它代表 `emphasize` ，“强调”。英文习惯用斜体字来表示强调的东西，那么自然， `emph{top matter}` 就是把 `top matter` 排版成 *top matter* 了。


注意到 [`subsection{}`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-subsec) 了吗？一会儿，我们还会看到 `subsubsection` 。不用解释吧，文章的章节次序是这样：


1. chapter
2. section
3. subsection
4. subsubsection
5. paragraph
6. subparagraph


其中， `chapter` 是给 `book` 和 `report` 用的，而 `article` 是从 `section` 开始。


现在我们就来一个 `subsubsection` 。不出所料的话，光标应该在第[27](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-subsub)行。那么就：



```
C-c C-s
```

`mini buffer` 提示：



```
Level: (default subsection)
```

当然输入：



```
subsubsection
C-j
```

`mini buffer` 提示：



```
Title:
```

输入：



```
Article Information
C-j
```

`mini buffer` 提示：



```
Label: sec:article-information
```

似曾相识吧？敲 `C-j`, 那么，你看到的是：



```
28:  subsubsection{Article Information}
29:  label{sec:article-information}
30:
```

也就是说，我们有了一个 `subsubsection` (第[28](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-subsub2)行) 。现在，我们再添加一个 `environment`:



```
C-c C-e
```

`mini buffer` 提示：



```
Environment type: (default abstract)
```

我们当然不再需要 `abstract` 了，现在我们要的是 `itemize` ，也就是“不带序号的列表”。那么当然输入：



```
itemize
C-j
```

于是看到：



```
31:  begin{itemize}
32:  item
33:  end{itemize}
34:
```

光标停在 `item` 的后面(第[32](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-item)行)。非常好，这正是我想要的。直接输入如下文字：



```
verb|title{}| --- The title of the article.
```

然后，



```
M-return
```

也就是，左手拇指按住 `Alt` 键，同时右手小指去敲回车键。你会看到这样的效果：



```
1:  begin{itemize}
2:  item verb|title{}| --- The title of the article.
3:  item
4:  end{itemize}
5:
```

也就是说，不仅换了行，而且自动有了 `item` 等待你输入新的东西(第[3](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-item2)行)。


你一定注意到了 `verb||` 这个新命令。它的作用和bash命令行的单引号 (`’`) 是一样的。还记得吧，在命令行，单引号里的东西是原样输出的。 `verb||` 里的东西也一样。 `verb` 是 `verbatim` 一词的缩写，就是“原样引用”的意思。好奇的话，就 `C-c C-c` 一下，看看效果。


好，继续输入：



```
verb|date| --- The date. Use:
```

得到：



```
1:  begin{itemize}
2:  item verb|title{}| --- The title of the article.
3:  item verb|date| --- The date. Use: |
4:  end{itemize}
5:
```

没什么好说的。现在我们要在 `itemize` 环境里面再来一个 `itemize` 。光标现在应该在第[3](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-use)行的最后。敲：



```
C-c C-e
C-j
```

于是得到：



```
1:  begin{itemize}
2:  item verb|title{}| --- The title of the article.
3:  item verb|date| --- The date. Use:
4:    begin{itemize}
5:    item 
6:    end{itemize}
7:  
8:  end{itemize}
9:
```

简单吧？不用说了，你肯定知道下面这些是怎么来的了吧。



```
1:  begin{itemize}
2:  item verb|title{}| --- The title of the article.
3:  item verb|date| --- The date. Use:
4:    begin{itemize}
5:    item verb|date{today}| --- to get the date that the document is typeset.
6:    item verb|date{}| --- for no date.
7:    end{itemize}
8:  end{itemize}
9:
```

编译一下，看看效果吧。


好了，请你现在照猫画虎，再来一个 `subsubsection` ，标题叫 `Author Information` 。模仿上面的东西，来得到下面的东西：



```
10:  subsubsection{Author Information}
11:  label{sec:author-information}
12:  
13:  The basic article class only provides the one command:
14:  begin{itemize}
15:  item verb|author{}| --- The author of the document.
16:  end{itemize}
17:  
18:  It is common to not only include the author name, but to insert new lines (verb|\|)
19:  after and add things such as address and email details.  For a slightly more logical
20:  approach, use the AMS article class (emph{amsart}) and you have the following extra
21:  commands:
22:  
23:  begin{itemize}
24:  item verb|address| --- The author's address.  Use the new line command (verb|\|) for line
25:    breaks.
26:  item verb|thanks| --- Where you put any acknowledgments.
27:  item verb|email| --- The author's email address.
28:  item verb|urladdr| --- The URL for the author's web page.
29:  end{itemize}
30:
```

怎么样，不太困难吧？ 目前为止，我们用到的无非是下面这几个快捷键操作而已：









| `C-j` | `C-c C-m` | `C-c C-s` | `C-c C-e` | `M-return` |


下面让我们再起一个小节：



```
C-c C-s
subsection
C-j
Sectioning Commands
C-j
C-j
```

得到：



```
31:  % 此处略去数十行
32:  
33:  subsection{Sectioning Commands}
34:  label{sec:sectioning-commands}
35:
```

添加一些文字：



```
 1:  % 此处略去数十行
 2:  
 3:  subsection{Sectioning Commands}
 4:  label{sec:sectioning-commands}
 5:  
 6:  The commands for inserting sections are fairly intuitive.  Of course,
 7:  certain commands are appropriate to different document classes.
 8:  For example, a book has chapters but a article doesn't.
 9:  
10:  %A simple table.  The center environment is first set up, otherwise the
11:  %table is left aligned.  The tabular environment is what tells Latex
12:  %that the data within is data for the table.
13:
```

你应该能猜到，凡是跟在 `%` 后面的都是注释。


在这一小节，我们来尝试一下表格的输入。


先起一个新“环境”，叫 `center` ：



```
C-c C-e
center
C-j
```

得到：



```
 1:  % 此处略去数十行
 2:  
 3:  subsection{Sectioning Commands}
 4:  label{sec:sectioning-commands}
 5:  The commands for inserting sections are fairly intuitive.  Of course,
 6:  certain commands are appropriate to different document classes.
 7:  For example, a book has chapters but a article doesn't.
 8:  
 9:  %A simple table.  The center environment is first set up, otherwise the
10:  %table is left aligned.  The tabular environment is what tells Latex
11:  %that the data within is data for the table.
12:  
13:  begin{center}
14:  
15:  end{center}
16:
```

`center` 当然是“居中”的意思了。在 `center` 环境里面，我们添加一个 `tabular` 环境：



```
C-c C-e
tabular
C-j
```

这时你会看到这样的提示：



```
(Optional) Position:
```

直接 `C-j` ，又看到提示了：



```
Format:
```

这是问你，表格的格式……有几列？每列之间要不要有竖线分割？……我的答案是这样：



```
|l|l|
```

也就是：竖线（|）小写L（l）竖线（|）小写L（l）竖线（|）。小写L代表 `left` ，也就是“左对齐”的意思。那么，你应该恍然大悟了，不就是……竖线-左对齐-竖线-左对齐-竖线嘛。那么，举一反三，除了小写L，我们还会见到r（右对齐）和c（居中）。现在，



```
C-j
```

得到如下结果：



```
 1:  % 此处略去数十行
 2:  
 3:  subsection{Sectioning Commands}
 4:  label{sec:sectioning-commands}
 5:  The commands for inserting sections are fairly intuitive.  Of course,
 6:  certain commands are appropriate to different document classes.
 7:  For example, a book has chapters but a article doesn't.
 8:  
 9:  %A simple table.  The center environment is first set up, otherwise the
10:  %table is left aligned.  The tabular environment is what tells Latex
11:  %that the data within is data for the table.
12:  
13:  begin{center}
14:    begin{tabular}{|l|l|}
15:  
16:    end{tabular}
17:  end{center}
18:
```

表格环境里也是可以有注释的：



```
 1:  % 此处略去数十行
 2:  
 3:  subsection{Sectioning Commands}
 4:  label{sec:sectioning-commands}
 5:  The commands for inserting sections are fairly intuitive.  Of course,
 6:  certain commands are appropriate to different document classes.
 7:  For example, a book has chapters but a article doesn't.
 8:  
 9:  %A simple table.  The center environment is first set up, otherwise the
10:  %table is left aligned.  The tabular environment is what tells Latex
11:  %that the data within is data for the table.
12:  
13:  begin{center}
14:    begin{tabular}{|l|l|}
15:      % The tabular environment is what tells Latex that the data within is
16:      % data for the table.  The arguments say that there will be two
17:      % columns, both left justified (indicated by the 'l', you could also
18:      % have 'c' or 'r'.  The bars '|' indicate vertical lines throughout
19:      % the table.
20:      | (hline)
21:    end{tabular}
22:  end{center}
23:
```

现在我们开始画表格，边画边体会一下上面几行注释的意思吧。光标现在应该在第[(hline)](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-hline)行，先画一条横线：



```
hline
C-j
```

所谓 `hline` ，顾名思义，就是 `horizontal line` 。


开始第一行：



```
Command & Level \ hline
C-j
```

那个 `&` 就是两列之间的分隔符， `\` 我们见过，表示换行。


照猫画虎，把所有的行都加上，得到如下结果：



```
 1:  % 此处略去数十行
 2:  
 3:  subsection{Sectioning Commands}
 4:  label{sec:sectioning-commands}
 5:  The commands for inserting sections are fairly intuitive.  Of course,
 6:  certain commands are appropriate to different document classes.
 7:  For example, a book has chapters but a article doesn't.
 8:  
 9:  %A simple table.  The center environment is first set up, otherwise the
10:  %table is left aligned.  The tabular environment is what tells Latex
11:  %that the data within is data for the table.
12:  
13:  begin{center}
14:    begin{tabular}{|l|l|}
15:      % The tabular environment is what tells Latex that the data within is
16:      % data for the table.  The arguments say that there will be two
17:      % columns, both left justified (indicated by the 'l', you could also
18:      % have 'c' or 'r'.  The bars '|' indicate vertical lines throughout
19:      % the table.
20:      hline 
21:      Command & Level \ hline
22:      verb|part{}| & -1 \
23:      verb|chapter{}| & 0 \
24:      verb|section{}| & 1 \
25:      verb|subsection{}| & 2 \
26:      verb|subsubsection{}| & 3 \
27:      verb|paragraph{}| & 4 \
28:      verb|subparagraph{}| & 5 \
29:      hline
30:    end{tabular}
31:  end{center}
32:
```

这张表格的效果应该类似于下面这样:






| Command | Level |
| --- | --- |
| `part{}` | -1 |
| `chapter{}` | 0 |
| `section{}` | 1 |
| `subsection{}` | 2 |
| `subsubsection{}` | 3 |
| `paragraph{}` | 4 |
| `subparagraph{}` | 5 |


现在编译一下，看看效果，还过得去吧？


好了，表格画完了。再添加点文字：



```
 1:  % 此处删去数十行……
 2:  
 3:  begin{center}
 4:    begin{tabular}{|l|l|}
 5:      % The tabular environment is what tells Latex that the data within is
 6:      % data for the table.  The arguments say that there will be two
 7:      % columns, both left justified (indicated by the 'l', you could also
 8:      % have 'c' or 'r'.  The bars '|' indicate vertical lines throughout
 9:      % the table.
10:      hline
11:      Command & Level \ hline
12:      verb|part{}| & -1 \
13:      verb|chapter{}| & 0 \
14:      verb|section{}| & 1 \
15:      verb|subsection{}| & 2 \
16:      verb|subsubsection{}| & 3 \
17:      verb|paragraph{}| & 4 \
18:      verb|subparagraph{}| & 5 \
19:      hline
20:    end{tabular}
21:  end{center}
22:  
23:  Numbering of the sections is performed automatically by LaTeX{}, so don't bother adding
24:  them explicitly, just insert the heading you want between the curly braces.  If you don't
25:  want sections number, then add an asterisk (*) after the section command, but before the
26:  first curly brace, e.g., verb|section*{A Title Without Numbers}|.
27:
```

现在我们讲讲“参考文献”。其实还是个 `environment` ，叫 `thebibliography` 。试试吧，



```
C-c C-e
thebibliography
C-j
```

`mini buffer` 提示：



```
Label for BibItem: 99
```

这是问你，在引用参考文献时，采用两个字符宽的标签是否合适？你知道, 参考文献在文章中被引用到的时候，不都是以 `[1],[2]...[99]` 这样的形式出现吗？所谓“两个字符的宽度”，也就是说方括号里的数字不超过两个，也就是说你的文章中最多可以有 `[1]...[99]`99个文献索引。这对于一篇普通的文章来说肯定是足够多了。


如果你还是不知道这合不合适，那么就假装合适, 按 `C-j`, `mini buffer` 提示：



```
(Optional) Bibitem label:
```

这是问你，要不要给每个参考文献条目加个标签？不理它, 按 `C-j`, `mini buffer` 提示：



```
Add key (default none):
```

这是必须要理的。所谓 `key` ，实际上就是一条参考文献的“标识号”，它是和前面我们见到的 `cite{}` 命令配合使用的。在引用一条参考文献时，就必然要通过它的标识号来唯一地找到它。比如 `citee{lamport94}` 就是要从参考文献列表中找到 `lamport94` 所对应的那一条。没明白？那么我们先给它一个 `key` ，等会儿编译一下，看看效果就明白了。输入：



```
lamport94
C-j
```

得到如下效果：



```
28:  begin{thebibliography}{99}
29:  bibitem{lamport94}
30:  end{thebibliography}
31:
```

现在把参考文献写进去就行了：



```
 1:  % 此处略去数十行……
 2:  
 3:  begin{thebibliography}{99}
 4:  bibitem{lamport94}
 5:    Leslie Lamport,
 6:    emph{LaTeX: A Document Preparation System}.
 7:    Addison Wesley, Massachusetts,
 8:    2nd Edition,
 9:    1994.
10:  end{thebibliography}
11:
```

再加一条：



```
M-return
C-j
wikibooks
C-j
http://en.wikibooks.org/wiki/LaTeX/simple.tex
```

得到：



```
 1:  % 此处略去数十行……
 2:  
 3:  begin{thebibliography}{99}
 4:  bibitem{lamport94}
 5:    Leslie Lamport,
 6:    emph{LaTeX: A Document Preparation System}.
 7:    Addison Wesley, Massachusetts,
 8:    2nd Edition,
 9:    1994.
10:  bibitem{wikibooks}
11:    http://en.wikibooks.org/wiki/LaTeX/simple.tex
12:  end{thebibliography}
13:
```

一篇像模像样的科技论文到此就算是大功告成了。



```
documentclass{article}
usepackage{times}

begin{document}

% Article top matter
title{How to Structure a LaTeX{} Document} %LaTeX is a macro for printing the Latex logo
author{Andrew Roberts\
  School of Computing,\
  University of Leeds,\
  Leeds,\
  United Kingdom,\
  LS2 1HE\
  emph{[[email protected]](/web/20190506155039/https://void-shana.moe/cdn-cgi/l/email-protection)}}  %emph formats the text to a typewriter style font
date{today}  %today is replaced with the current date
maketitle

begin{abstract}
  In this article, I shall discuss some of the fundamental topics in
  producing a structured document.  This document itself does not go into
  much depth, but is instead the output of an example of how to implement
  structure. Its LaTeX{} source, when in used with my tutorial
  provides all the relevant information.  end{abstract}

section{Introduction}
This small document is designed to illustrate how easy it is to create a
well structured document within LaTeXcite{lamport94}.  You should quickly be able to
see how the article looks very professional, despite the content being
far from academic.  Titles, section headings, justified text, text
formatting etc., is all there, and you would be surprised when you see
just how little markup was required to get this output.

section{Structure}
One of the great advantages of LaTeX{} is that all it needs to know is
the structure of a document, and then it will take care of the layout
and presentation itself.  So, here we shall begin looking at how exactly
you tell LaTeX{} what it needs to know about your document.

subsection{Top Matter}
The first thing you normally have is a title of the document, as well as
information about the author and date of publication.  In LaTeX{} terms,
this is all generally referred to as emph{top matter}.

subsubsection{Article Information}
% Set up an 'itemize' environment to start a bulleted list.  Each
% individual item begins with the item command.  Also note in this list
% that it has two levels, with a list embedded in one of the list items.
begin{itemize}
item verb|title{}| --- The title of the article.
item verb|date{}| --- The date. Use:
  begin{itemize}
  item verb|date{today}| --- to get the date that the document is typeset.
  item verb|date{}| --- for no date.
  end{itemize}
end{itemize}

subsubsection{Author Information}
The basic article class only provides the one command:
begin{itemize}
item verb|author{}| - The author of the document.
end{itemize}

It is common to not only include the author name, but to insert new
lines (verb|\|) after and add things such
as address and email details.  For a slightly more logical approach, use
the AMS article class (emph{amsart}) and you have the following extra
commands:

begin{itemize}
item verb|address| - The author's address.  Use the new line command (verb|\|) for line breaks.
item verb|thanks| - Where you put any acknowledgments.
item verb|email| - The author's email address.
item verb|urladdr| - The URL for the author's web page.
end{itemize}

subsection{Sectioning Commands}
The commands for inserting sections are fairly intuitive.  Of course,
certain commands are appropriate to different document classes.
For example, a book has chapters but a article doesn't.

% A simple table.  The center environment is first set up, otherwise the
% table is left aligned.  The tabular environment is what tells Latex
% that the data within is data for the table.
begin{center}
  begin{tabular}{|l|l|} 
    % The tabular environment is what tells Latex that the data within is
    % data for the table.  The arguments say that there will be two
    % columns, both left justified (indicated by the 'l', you could also
    % have 'c' or 'r'.  The bars '|' indicate vertical lines throughout
    % the table.

    hline  % Print horizontal line
    Command & Level \ hline  % Columns are delimited by '&'.  And
    % rows are delimited by '\'
    verb|part{}| & -1 \
    verb|chapter{}| & 0 \
    verb|section{}| & 1  \
    verb|subsection{}| & 2 \
    verb|subsubsection{}| & 3 \
    verb|paragraph{}| & 4 \
    verb|subparagraph{}| & 5 \
    hline
  end{tabular}
end{center}

Numbering of the sections is performed automatically by LaTeX{}, so don't
bother adding them explicitly, just insert the heading you want between
the curly braces.  If you don't want sections number, then add an asterisk (*) after the
section command, but before the first curly brace, e.g., verb|section*{A Title Without Numbers}|.

% Create the environment for the bibliography.  Since there is only one
% reference, set the label width to be one character (I shall follow
% convention as use the number '9'.  This is because it helps to remind
% that it is the maximum number of refs that is now permitted by that
% width).
begin{thebibliography}{99}
  % The bibitem is to start a new reference.  Ensure that the cite\_key is
  % unique.  You don't need to put each element on a new line, but I did
  % simply for readability.
bibitem{lamport94}
  Leslie Lamport,
  emph{LaTeX: A Document Preparation System}.
  Addison Wesley, Massachusetts,
  2nd Edition,
  1994.
bibitem{wikibooks}
  http://en.wikibooks.org/wiki/LaTeX/simple.tex
end{thebibliography} %Must end the environment

end{document}
```

编译一下，看看[结果](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/simple.pdf)吧。


连看带写，把前面这些东西走一遍，应该不超过一个小时吧。现在我们总结一下你用到的东西：


* LaTeX的基本命令和环境  





| `title{}` | `author{}` | `date{}` |
| `section{}` | `subsection{}` | `subsubsection{}` |
| `itemize` | `tabular` | `center` |
* 最基本的Emacs快捷键（以 `C-x` 开头的比较多）  








| 文件操作 | `C-x C-f` | `C-x C-s` | `C-x C-c` |  |  |
| 编辑操作 | `C-g` | `C-j` | `C-i` | `C-/` |  |
|  | `[[email protected]](/web/20190506155039/https://void-shana.moe/cdn-cgi/l/email-protection)` | `C-SPACE` | `C-x h` | `C-w` | `C-INSERT` |
|  | `C-k` | `C-y` |  |  |  |
|  | `C-d` | `M-d` |  |  |  |
| 移动光标 | `C-a` | `C-e` | `C-f` | `C-b` |  |
|  | `C-n` | `C-p` | `C-v` |  |  |
| 窗口操作 | `C-x 2` | `C-x 3` | `C-x 0` | `C-x 1` | `C-x o` |
| 帮助 | `C-h i` | `C-h t` | `C-h k` | `C-h f` | `C-h v` |
* 最基本的AucTeX快捷键（以 `C-c` 开头的比较多）  






| C-c C-e | C-c C-s | C-c C-m | M-return |
| C-c C-c | C-c C-v |  |  |






2 入门以后……
--------



有了第一部分的基础，你可以自己看点参考资料了。


1. AucTeX手册就在Emacs里面：

```
C-h i
m
auctex
C-j
```
2. 花两个小时看看[lshort](https://web.archive.org/web/20190506155039/http://tobi.oetiker.ch/lshort/lshort.pdf) 。打开命令行窗口，敲：

```
sudo apt-get install texlive-doc-en
texdoc lshort
```


后面，我们还会简要介绍


1. 如何插入图片
2. 如何写数学公式
3. 如何插入程序代码
4. 如何写中文
5. 如何用毕业论文模版
6. 如何做幻灯片
7. emacs org-mode




### 2.1 如何插入图片




```
1:  documentclass{article}
2:  
3:  usepackage{graphicx}
4:  graphicspath{{./figs/}{./}}
5:  
6:  begin{document}
7:  includegraphics[width=5cm]{tux}
8:  end{document}
```

怎么样，能看明白吗？插入图片用到了三个新命令：


1. `usepackage{graphicx}`, 这是在说“我要用到一个名字叫 `graphicx` 的package(宏包)”。似曾相识吧？它不是很类似`#include<stdio.h>` 吗? [`includegraphics{}`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-ig) 就是这个宏包提供的命令之一。想详细了解 `graphicx`, 敲：

```
texdoc graphicx
```
2. `graphicspath{{./figs/}{./}}`, 显然这是在指明graphics(图片)所在的path(路径，位置)。
3. `includegraphics[width=5cm]{tux}`, 这当然就是插入图片了。
	1. 图片的名字叫 `tux.pdf`, 后缀（.pdf）可以被省略掉。显然 `tux.pdf` 应该被存放在 `./` 或者 `./figs/` 中，才能被找到。我喜欢PDF图片，因为它可以自由缩放。你当然可以插入jpeg、png图片。
	2. 宽度是5cm，也可以是相对宽度，比如 `[width = .5textwidth]` 就表示宽度等于0.5倍的行宽。


如果你希望图片“居中”摆放，那自然是要用到 `center` 了：



```
 1:  documentclass{article}
 2:  
 3:  usepackage{graphicx}
 4:  graphicspath{{./figs/}{./}}
 5:  
 6:  begin{document}
 7:  begin{center}
 8:    includegraphics[width=5cm]{tux}
 9:  end{center}
10:  end{document}
```

“哎，为什么没有图片说明？比如，【图1: Linux图标】？” 当然可以有，我们要用到一个新environment了，就叫 `figure` ：



```
C-c C-e
figure
C-j
```

`mini buffer` 提示：



```
(Optional) Float position:
```

这是在问你图片放在那里比较好啊？是靠上？还是靠下？还是……懒得操心？那还是让LaTeX来决定吧, `C-j`, `mini buffer` 提示：



```
Caption:
```

这是在提示你输入图片的说明文字。那么输入：



```
Linux logo
```

`mini buffer` 提示：



```
Center? (y or n):
```

当然是：



```
y
```

`mini buffer` 提示：



```
Label: fig:
```

这是要你给图片打个标签，以后方便索引到它。那么就给个标签：



```
linux-logo
```

于是得到：



```
 1:  documentclass{article}
 2:  
 3:  usepackage{graphicx}
 4:  graphicspath{{./figs/}{./}}
 5:  
 6:  begin{document}
 7:  
 8:  begin{figure}
 9:    centering
10:    | (fig)
11:    caption{Linux logo}
12:    label{fig:linux-logo}
13:  end{figure}
14:  
15:  end{document}
```

现在你建立了一个完美的图片环境，别忘了把图片放进去。当然放在第[(fig)](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-fig)行：



```
 1:  documentclass{article}
 2:  
 3:  usepackage{graphicx}
 4:  graphicspath{{./figs/}{./}}
 5:  
 6:  begin{document}
 7:  
 8:  begin{figure}
 9:    centering
10:    includegraphics[width=5cm]{tux}
11:    caption{Linux logo}
12:    label{fig:linux-logo}
13:  end{figure}
14:  
15:  end{document}
```

来试试 `label{}` 的作用：



```
 1:  documentclass{article}
 2:  
 3:  usepackage{graphicx}
 4:  graphicspath{{./figs/}{./}}
 5:  
 6:  begin{document}
 7:  
 8:  begin{figure}
 9:    centering
10:    includegraphics[width=5cm]{tux}
11:    caption{Linux logo}
12:    label{fig:linux-logo} (label)
13:  end{figure}
14:  
15:  Figure~ref{fig:linux-logo} is the famous Linux Tux! (ref)
16:  end{document}
```

编译一下，看看效果吧。 [`label{}`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-label) 和 [`ref{}`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-ref) 总是配合使用的，一个用来打标签，另一个用来找到它。





### 2.2 如何插入数学公式



举个简单的例子吧：



```
1:  documentclass{article}
2:  
3:  begin{document}
4:  
5:  This is a simple math example: $c^2=a^2+b^2$
6:  
7:  end{document}
```

结果是这样：This is a simple math example: c2=a2+b2


* 美元符号($)在LaTeX里面是特殊字符。夹在两个美元符号之间的东西，会被当做数学公式来排版。
* 如果想让数学公式独占一行的话，就这样：

```
1:  documentclass{article}
2:  
3:  begin{document}
4:  
5:  This is a simple math example: $$c^2=a^2+b^2$$
6:  
7:  end{document}
```

也就是每边用两个美元符号。如果想给数学公式排序号的话，就把它放进 equation 环境里：



```
C-c C-e
equation
C-j
c^2 = a^2 + b^2
```

结果就是这样：



```
 1:  documentclass{article}
 2:  
 3:  begin{document}
 4:  
 5:  This is a simple math example: $$c^2=a^2+b^2$$
 6:  begin{equation}
 7:    label{eq:1}
 8:    c^2 = a^2 + b^2
 9:  end{equation}
10:  end{document}
```

编译一下看看，公式右边是不是有序号了？简单吧。


数学公式排版是LaTeX的强项，各式各样的数学符号、怪异字符无所不及。当然用不着都记住。你只要记住上面这点东西，剩下的，知道查手册就行了：


	1. [lshort](https://web.archive.org/web/20190506155039/http://tobi.oetiker.ch/lshort/lshort.pdf), chapter 3
	
	```
	texdoc lshort
	```
	2. The LaTeX Companion, [chapter 8](https://web.archive.org/web/20190506155039/ftp://ftp.tex.ac.uk/purged/info/companion-rev/ch8.pdf)





### 2.3 如何插入程序代码



看个例子就了然了：



```
 1:  documentclass{article}
 2:  
 3:  usepackage{minted}
 4:  
 5:  begin{document}
 6:  
 7:  begin{minted}{c}
 8:    #include <stdio.h>
 9:    int main(int argc, char *argv[])
10:    {
11:      printf ("Hello, world!n");
12:      return 0;
13:    }
14:  end{minted}
15:  
16:  end{document}
17:  
18:  %%% Local Variables: 
19:  %%% mode: latex
20:  %%% TeX-master: t
21:  %%% End:
```

1. 首先要 `usepackage{minted}` 。在LaTeX文件中插入代码的工具包有不少，比较传统的是 `listings`, 比较新潮的是 `minted`。我们当然选用新潮的。想详细了解 `minted` 的用法，就

```
texdoc minted
```
2. 开始一个新环境 `minted` ，把你的程序拷贝进来。怎么？程序太长，拷贝起来太麻烦？那么可以这样：

```
 1:  documentclass{article}
 2:  
 3:  usepackage{minted}
 4:  
 5:  begin{document}
 6:  
 7:  inputminted{c}{hello.c}
 8:  
 9:  end{document}
10:  
11:  %%% Local Variables: 
12:  %%% mode: latex
13:  %%% TeX-master: t
14:  %%% End:
```

简单吧？ [`hello.c`](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html#coderef-hello) 当然要和你的 `TeX` 文件在同一个目录下，否则你要指明详细路径。


`minted` 宏包提供了丰富的命令，可以支持数十种编程语言，后台调用强大的 `pygments` 来打扮你的程序代码，可以把程序以各种你能想到的方式排版出来。


当然，使用 `minted` 的前提条件是，系统里已经安装好了 `python` 和 `pygments` 。这一点在 `minted` 的手册里已经说得很清楚了。



```
texdoc minted
```

关于强大的 `pygments`, 你该去它的网站看看：[http://pygments.org/](https://web.archive.org/web/20190506155039/http://pygments.org/)





### 2.4 如何输入中文



如果你的Ubuntu配置和D215机房是一致的，那么输入中文就是“piece of cake”：



```
C-x C-f
chinese.tex
C-j
M-x
article [TAB]
```

一个现成的LaTeX模版立时展现在你面前：



```
documentclass[12pt,a4paper]{article}
usepackage[top=3cm,bottom=3cm,left=3cm,right=3cm]{geometry}

usepackage{graphicx}
graphicspath{{./figs/}{../figs/}{./}{../}} %note that the trailing “/” is required

usepackage{latexsym,pifont,units,amsmath,amsfonts,amssymb,marvosym}

usepackage{indentfirst}
usepackage[indentafter,pagestyles]{titlesec}

usepackage{xcolor}

usepackage{multicol,rotating,soul}
setul{1.5pt}{.4pt}% 1.5pt below contents

usepackage{xltxtra} %fontspec,xunicode are loaded here.
defaultfontfeatures{Mapping=tex-text}
setsansfont{DejaVu Sans}
setmainfont{DejaVu Serif}
setmonofont{DejaVu Sans Mono}
usepackage{xeCJK}
setCJKmainfont[BoldFont={WenQuanYi Zen Hei}, ItalicFont={WenQuanYi Zen Hei}]{SimSun}
setCJKfamilyfont{hei}{WenQuanYi Zen Hei}
setCJKfamilyfont{song}{SimSun}
newcommand{ziju}[1]{renewcommand{CJKglue}{hskip #1}}

usepackage{hyperref}

usepackage{fancyhdr}
pagestyle{fancy}

% use snippet 'im', 'minted' to add code blocks
usepackage{minted}

renewcommand{refname}{参考文献} % article.cls
renewcommand{contentsname}{目录}
renewcommand{listfigurename}{插图目录}
renewcommand{listtablename}{表格目录}
renewcommand{abstractname}{摘要}
renewcommand{appendixname}{附录}
renewcommand{indexname}{索引}
renewcommand{figurename}{图}
renewcommand{tablename}{表}

newcommand{code}[1]{{fontspec{DejaVu Sans Mono}{textcolor{voilet}{#1}}}}
newcommand{cfbox}[2]{%
  colorlet{currentcolor}{.}%
  {color{#1}fbox{color{currentcolor}#2}}%
}

title{}
author{Wang Xiaolin}

begin{document}

maketitle{}

% renewcommand{abstractname}{Abstract}
begin{abstract}

end{abstract}

clearpage

pagenumbering{roman}
tableofcontents
listoffigures{}
% listoftables{}

clearpage

pagenumbering{arabic}

end{document}
% (setq-default TeX-master nil)
```

这个模版里有你写一篇漂亮中文文章所需要的一切。光标就停在 `title{}` 的花括号里，那么就开始用中文填空吧。Happy TeXing!





### 2.5 完美的毕业论文



* 写普通文章要用 `documentclass{article}` ；
* 写报告要用 `documentclass{report}` ；
* 写书要用 `documentclass{book}` ；
* 写信要用 `documentclass{letter}` ；
* 那么写毕业论文自然要用毕业论文模版了：

```
documentclass{swfcthesis}
```

写毕业论文是个庄严的大事情，当然要为它专门建立一个目录吧。目录建好了，把 [swfcthesis.cls](https://web.archive.org/web/20190506155039/http://cs3.swfc.edu.cn/~wx672/swfcthesis/swfcthesis/swfcthesis.cls) 文件拷贝到这个目录里。然后就可以用它来写你的论文了。


简而言之，swfcthesis.cls 就是我们的论文模版，它提供了如下一些基本命令：


1. `Title{论文标题}`
2. `Author{作者}`
3. `Advisor{指导教师姓名}`
4. `AdvisorTitle{指导教师职称}`
5. `AdvisorInfo{指导教师简介}`
6. `Month{这里填月份}`
7. `Year{这里填年份}`
8. `Univ{这里填校名}`
9. `School{这里填院名}`
10. `Subject{这里填专业}`
11. `Docname{本科毕业（设计）论文}` ：目前我们只关心本科毕业论文
12. `Abstract{这里填上中文摘要}`
13. `Keywords{这里填上关键字}`
14. `Acknowledgments{这里填上鸣谢某某某}`
15. `enTitle{这里填上英文标题}`
16. `enAuthor{这里填上你的洋名字}`
17. `enUniv{这里填上英文校名}`
18. `enSchool{这里填上英文院名}`
19. `enAbstract{这里填上英文摘要}`
20. `enKeywords{这里填上英文关键字}`
21. `makepreliminarypages{}` ：生成封面、摘要、英文摘要……
22. `Appendix{}` ： 附录部分由此开始
23. `advisorinfopage{}` ： 生成【教师简介】
24. `acknowledgmentspage{}` ： 生成【鸣谢】


具体如何来用这些命令呢？简而言之，



```
C-x C-f
sample.tex
C-j C-j
thesis [TAB]
```

现在一个论文模版就呈现在你的眼前了。



```
documentclass{swfcthesis}

begin{document}

Title{论文标题}        % 论文标题
Author{作者姓名}       % 作者姓名 
Advisor{指导教师姓名}      %指导教师姓名
AdvisorTitle{指导教师职称} %指导教师职称
AdvisorInfo{指导教师简介（约百余字）}  %指导教师简介（约百余字）
Month{六}                  %月份（比如 六）
Year{二〇一二}             %年份（比如 二〇一二）
Univ{西南林业大学}             %校名
School{计算机与信息科学学院}           %院系名称
Subject{计算机科学与技术专业}  %专业名称（比如 电子信息工程专业）
Docname{本科毕业（设计）论文}  %本科？研究生？
Abstract{这里写论文摘要（约两百字）}   %论文摘要（约两百字）
Keywords{这里写关键字，比如 电阻, 电容}        %关键字（比如 电阻, 电容, 电感, 电灯泡）
Acknowledgments{这里写鸣谢（约百余字）}                %鸣谢 （感谢人民感谢党，约百字）
enTitle{英文标题}                      %英文标题
enAuthor{英文姓名}                     %作者英文姓名
enUniv{Southwest Forestry University}  %英文校名
enSchool{School of Computer and Information Science}   %英文院系名称
enAbstract{英文摘要}                    %论文英文摘要
enKeywords{英文关键字}          %英文关键字

%%% 下面六行不要动！
makepreliminarypages
frontmatter
tableofcontents
listoffigures
listoftables
mainmatter
%%% 上面六行不要动！

chapter{第一章标题}    %第一章标题
label{cha:one}

正文部分从此开始。可以参考模版目录中的各章节tex文件来写。

正文部分从此开始。可以参考模版目录中的各章节tex文件来写。

正文部分从此开始。可以参考模版目录中的各章节tex文件来写。

正文部分从此开始。可以参考模版目录中的各章节tex文件来写。

chapter{步入正题}
label{cha:two}

可以参考模版目录中的各章节tex文件来写。

可以参考模版目录中的各章节tex文件来写。

可以参考模版目录中的各章节tex文件来写。

可以参考模版目录中的各章节tex文件来写。

可以参考模版目录中的各章节tex文件来写。

可以参考模版目录中的各章节tex文件来写。

%%% 正文部分到此结束。下面是『参考文献』、『指导教师简介』、『鸣谢』、『附录』
Appendix{} % 不要动这一行！
% 下面是参考文献部分
begin{thebibliography}{99}
bibitem{标签}作者，书名，出版社，出版日期
bibitem{标签}作者，书名，出版社，出版日期
bibitem{标签}作者，书名，出版社，出版日期
bibitem{标签}作者，书名，出版社，出版日期
bibitem{标签}作者，书名，出版社，出版日期
end{thebibliography}

advisorinfopage{} % 不要动这一行！
acknowledgmentspage{} % 不要动这一行！

%%% 下面是附录部分。

chapter{我也不知道为什么要写附录}              %附录一
label{cha:app}

可以参考模版目录中的 appendix.tex 文件来写。

可以参考模版目录中的 appendix.tex 文件来写。

可以参考模版目录中的 appendix.tex 文件来写。

chapter{我居然编程了！}                        %附录二
label{cha:app}

可以参考模版目录中的 src.tex 文件来插入代码。比如：

inputminted[fontsize=small]{c}{hello.c}

或者，

begin[fontsize=small]{minted}{c}
  int main()
  {
    printf("Hello, world!n");
    return 0;
  }
end{minted}

或者，

begin{listing}[H]
  inputminted{c}{hello.c}
  caption{我居然编程了！}
  label{lst:hello}
end{listing}  

或者，

begin{listing}[H]
  begin{minted}{c}
    int main()
    {
      printf("Hello, world!n");
      return 0;
    }
  end{minted}
  caption{我居然编程了！}
  label{lst:helloagain}
end{listing}  

end{document} % 此行后面不要有任何文字。

%%% Local Variables:
%%% mode: latex
%%% TeX-master: t
%%% End:
```

你要做的就是用你前面学到的那些LaTeX命令来填空。别忘了，你随时可以编译一下，看看效果。


还不明白？再看个例子来找找感觉吧：


* thesis2.tex



```
documentclass{swfcthesis}

begin{document}

Title{金庸笔下武功详解}
Author{周伯通}
Advisor{王重阳}
AdvisorTitle{祖师}
AdvisorInfo{王重陽footnote{参见维基百科-href{http://zh.wikipedia.org/wiki/%E7%8E%8B%E9%87%8D%E9%99%BD}{王重阳}}（1113年1月13日－1170年），原名中孚，字允卿，又名世雄，字德威，入道后改名喆，字知明，道号重阳子，故称王重阳。北宋末京兆咸阳（今陕西咸阳）大魏村人。中國道教分支全真道的始創人，后被尊为道教的北五祖之一。他有七位出名的弟子，在道教历史上称为北七真。}
Month{六}
Year{一二三七}
Univ{中华武林大学}
Docname{本科毕业（设计）论文}
School{全真玄门正宗学院}
Subject{打架斗殴专业}
Abstract{中國武術footnote{参见维基百科 -
    href{http://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E6%AD%A6%E6%9C%AF}{中国武术}}
  是中國传统文化的重要一環。兩廣人稱為功夫，民國初期簡稱為國術（後為中央國術館正式採用之名稱）；被視
  為中國文化之精粹，故又稱國粹。由於歷史發展和地域分佈關係，衍生出不同門派。中國武術主要內容包括搏擊
  技巧、格鬥手法、攻防策略和武器使用等技術。當中又分為理論和實踐兩個範疇。從實踐中帶來了有關體育、健
  身、和中國武術獨有之氣功、及養生等重要功能。理論中帶來了不少前人之經驗和拳譜記錄。因此，它体现中华
  民族对攻防技击及策略上的理解。加上經驗上積累，以自立、自強、健體養生為目標的自我運作，練習套路时顯
  示出身體動作之優美姿態。中國武術往往帶有思想冶鍊的文化特徵及人文哲學的特色、意義，對現今中國的大眾
  文化有著深遠影響Cite{wushucn}。}
Keywords{金庸，武术，一陽指，双手互搏，空明拳，七傷拳，吸星大法，葵花宝典，九陰真經，九陽真
  經，天山六陽掌，天羅地網勢，蛤蟆功，倚天屠龍功，弹指神通，先天功，打狗棒法，全真剑法，摧心掌，降龍
  十八掌，六脈神劍，火焰刀，黯然銷魂掌，龍爪擒拿手，兰花拂穴手，龍象般若掌，劈空掌，玉女素心剑法，北
  冥神功，碧海潮生曲}
Acknowledgments{感谢师兄王重阳传我全真玄门正宗功夫，感谢段皇爷不杀之恩，感谢刘贵妃不怨旧恶，感谢桃
  花岛主黄老邪助我练就空明拳和双手互搏之术，感谢郭靖兄弟让我看《九阴真经》，感谢小龙女教我驭蜂之术。}
enTitle{Jin Yong's Chinese Martial Arts Illustrated}
enAuthor{Zhou Botong}
enUniv{Chinese Kungfu University}
enSchool{School of Taoism}
enAbstract{Chinese martial
  artsfootnote{Wikipedia - href{http://en.wikipedia.org/wiki/Chinese\_martial\_arts}{Chinese martial arts}}, also referred to by the Mandarin Chinese term wushu and popularly as kung fu, are a number of fighting styles that have developed over the centuries in China. These fighting styles are often classified according to common traits, identified as "families", "sects" or "schools" of martial arts. Examples of such traits include physical exercises involving animal mimicry, or training methods inspired by Chinese philosophies, religions and legends. Styles which focus on qi manipulation are labeled as internal, while others concentrate on improving muscle and cardiovascular fitness and are labeled external. Geographical association, as in northern and southern, is another popular method of categorizationCite{wushu}.}
enKeywords{Jin Yong, Chinese martial arts, Kungfu}

makepreliminarypages %生成封面、摘要、英文摘要……
frontmatter          %目录部分由此开始
tableofcontents      %生成目录
listoffigures        %图片目录
listoftables         %表格目录
mainmatter           %正文部分由此开始

写啊写啊写啊写啊写啊写啊写啊写啊写啊写啊写啊

                      %正文部分到此结束
Appendix{}           % 附录部分由此开始

include{bibliography} % 把参考文献（bibliography.tex）包含进来
advisorinfopage{}     % 生成【教师简介】
acknowledgmentspage{} % 生成【鸣谢】
include{appendix}     % 如果有附录（appendix.tex）的话，把它包含进来
include{src}          % 把程序代码（source.tex）附在这里

end{document}
%%% Local Variables: 
%%% mode: latex
%%% TeX-master: t
%%% End:
```

上面的例子中，用到了三个 `include{}` 命令，分别把 `bibliography.tex`, `appendix.tex`, 和 `source.tex` 三个文件的内容包含了进来。也就是说，你事先写好了三个文件：


1. `bibliography.tex`: 用来存放参考文献
2. `appendix.tex`: 用来存放附录内容
3. `source.tex`: 用来存放程序代码


下面是这三个文件的例子：


* bibliography.tex



```
cleardoublepage
phantomsection
addcontentsline{toc}{chapter}{参考文献}

begin{thebibliography}{99}
bibitem{shediao} 金庸, 《射雕英雄传》, 广州出版社, 2003. 
bibitem{shendiao} 金庸, 《神雕侠侣》, 陕西人民出版社, 1992. 
bibitem{yitian} 金庸, 《倚天屠龙记》, 广州出版社, 2002. 
end{thebibliography}

%%% Local Variables: 
%%% mode: latex
%%% TeX-master: "./thesis2"
%%% End:
```

* appendix.tex



```
chapter{关于金庸}
label{sec:appendixname}

section{金庸简介}

金庸footnote{参见维基百科-href{http://zh.wikipedia.org/wiki/%E9%87%91%E5%BA%B8}{金
      庸}}，大紫荊勳賢，OBE，原名查良鏞，（Louis Cha Leung Yung，1924年2月6日－）浙江海寧人，東吳大
  学法學士、英國劍橋大學歷史碩士、博士。查良鏞在1948年移居香港，以筆名金庸著作多部膾炙人口的武俠小說，
  是华人界最知名的武俠小說作家之一。金庸亦是香港新闻、文艺界的杰出创业者及评论家，以及著名的社会活动
  家。金庸、古龍與梁羽生普遍被認為是新派武侠小说的代表作家，被誉为武侠小说作家的「泰斗」，更有「金迷」
  们尊称其为「金大侠」或「查大俠」，亦被喻為「香港四大才子」之一Cite{jinyongcn}。
begin{itemize}
item 傳說金庸對古典樂十分喜好，判別能力超強。可以隨便聽過一段樂句，就告訴你這是哪位作曲家的哪首作品。
item 金庸愛車，跑車为尤。
item 接受訪談時，金庸認為自己像張無忌（《倚天屠龍記》），妻子像夏青青（《碧血劍》）。
item 金庸在《鹿鼎記》后記中，表示自己最喜歡的作品是《倚天屠龍記》，《笑傲江湖》，《神鵰俠侶》和《飛狐外
  傳》。
item 金庸在《倚天屠龍記》后記中，表示自己最愛的女性人物是小昭（《倚天屠龍記》）。
item 曾开除金庸的國立政治大學（其前身即中央政治學校）于2007年5月19日，授予金庸榮譽博士學位。
item 金庸於香港大學設立“查良鏞學術基金”，並擔任主席，主力邀講各國學者定期舉行學術講座和研討會。
item 香港大學於2009年2月27日成立國際金庸研究會。研究會並非註冊社團，由金庸及前港大中文系主任單周堯教授擔任顧問，創會會長為李思齊教授及伍懷璞教授，現任會長為港大文學院黎活仁副教授Cite{jinyongcn}。
end{itemize}

%%% Local Variables: 
%%% mode: latex
%%% TeX-master: "./thesis2"
%%% End:
```

* source.tex



```
chapter{程序代码}

begin{listing}[H]
  inputminted[fontsize=footnotesize,frame=single]{c}{hello.c}
  caption{一个著名的C程序}
  label{lst:hello}
end{listing}

%%% Local Variables: 
%%% mode: latex
%%% TeX-master: "./thesis2"
%%% End:
```

关于论文模版的实现细节、应用实例等信息都可以在[这里](https://web.archive.org/web/20190506155039/http://cs3.swfc.edu.cn/~wx672/swfcthesis/swfcthesis/)找到。





### 2.6 LaTeX Beamer — 完美的幻灯片




#### 2.6.1 Hello, world!



让我们还是先拿英文来举例子，最后再说怎么写中文。



```
documentclass{beamer}

begin{document}

title{A Simple Presentation}
author{A LaTeX{}er}

begin{frame}
  titlepage
end{frame}

begin{frame}{Hello, world!}
  This is a "Hello, world!" example of writing presentation with LaTeX{} beamer.
end{frame}

end{document}

%%% Local Variables: 
%%% mode: latex
%%% TeX-master: t
%%% End:
```

一套幻灯片是由若干幅画面（frame）组成的，每个 `frame` 可以有一个标题（frametitle）。Easy? 编译一下，看看效果，还不坏吧？


`frame` 里面可以放任何东西，比如


* `itemize`, `enumerate`

```
 1:  begin{frame}frametitle{Examples}
 2:    Itemize example:
 3:    begin{itemize}
 4:    item Hello, world! is simple; 
 5:    item LaTeX{} beamer is easy.
 6:    end{itemize}
 7:    Enumerate example:
 8:    begin{enumerate}
 9:    item LaTeX{} beamer is powerful, and
10:    item beautiful.
11:    end{enumerate}
12:  end{frame}
```
* `example`, `exampleblock`

```
 1:  begin{frame}frametitle{Examples}
 2:    begin{example}
 3:      Itemize example:
 4:      begin{itemize}
 5:      item Hello, world! is simple;
 6:      item LaTeX{} beamer is easy.
 7:      end{itemize}
 8:    end{example}
 9:    begin{exampleblock}{Enumerate example:}
10:      begin{enumerate}
11:      item LaTeX{} beamer is powerful, and
12:      item beautiful.
13:      end{enumerate}
14:    end{exampleblock}
15:  end{frame}
```
* `columns`

```
 1:  begin{frame}frametitle{Examples}
 2:    begin{columns}
 3:      begin{column}{.5textwidth}
 4:        begin{example}
 5:          Itemize example:
 6:          begin{itemize}
 7:          item Hello, world! is simple;
 8:          item LaTeX{} beamer is easy.
 9:          end{itemize}
10:        end{example}
11:      end{column}
12:      begin{column}{.5textwidth}
13:        begin{exampleblock}{Enumerate example:}
14:          begin{enumerate}
15:          item LaTeX{} beamer is powerful, and
16:          item beautiful.
17:          end{enumerate}
18:        end{exampleblock}
19:      end{column}
20:    end{columns}
21:  end{frame}
```
* 逐行显示：

```
 1:  begin{frame}frametitle{Examples}
 2:    Itemize example: pause
 3:    begin{itemize}
 4:    item Hello, world! is simple;  pause
 5:    item LaTeX{} beamer is easy. pause
 6:    end{itemize}
 7:    Enumerate example: pause
 8:    begin{enumerate}
 9:    item LaTeX{} beamer is powerful, and pause
10:    item beautiful. pause
11:    end{enumerate}
12:  end{frame}
```


白加黑太单调了？那么，尝试一下七色阳光：



```
documentclass{beamer}

usetheme{Warsaw} % 换个漂亮的式样

begin{document}

title{A Simple Presentation}
author{A LaTeX{}er}

begin{frame}
  titlepage
end{frame}

begin{frame}{Hello, world!}
  This is a "Hello, world!" example of writing presentation in LaTeX beamer.
end{frame}

end{document}

%%% Local Variables: 
%%% mode: latex
%%% TeX-master: t
%%% End:
```

LaTeX beamer提供了大量现成的花哨式样，你只要



```
usetheme{你喜欢的式样}
```

就行了。去哪里找？



```
texdoc beamer
```




#### 2.6.2 中文幻灯片



如果你的Ubuntu配置和D215机房是一致的，那么输入中文就是“piece of cake”：



```
C-x C-f
beamer-cn.tex
C-j
M-x
beamer [TAB]
```

一个现成的LaTeX beamer模版立时展现在你面前：



```
documentclass[ignorenonframetext,xcolor=svgnames,hyperref={xetex,colorlinks,linkcolor=blue},compress]{beamer}

usepackage{pgfpages}

usepackage{latexsym,pifont,units,amsmath,amsfonts,amssymb,marvosym}

usepackage{xltxtra} %fontspec,xunicode are loaded here.
defaultfontfeatures{Mapping=tex-text}
setsansfont{DejaVu Sans}
setmainfont{DejaVu Serif}
setmonofont{DejaVu Sans Mono}
usepackage{xeCJK}
setCJKmainfont[BoldFont={WenQuanYi Zen Hei}, ItalicFont={WenQuanYi Zen Hei}]{SimSun}
setCJKfamilyfont{hei}{WenQuanYi Zen Hei}
setCJKfamilyfont{song}{SimSun}
newcommand{ziju}[1]{renewcommand{CJKglue}{hskip #1}}

% usepackage{graphicx} % beamer loads graphicx already.
graphicspath{{./figs/}{../figs/}{./}{../}} %note that the trailing “/” is required

usepackage{tikz}
usetikzlibrary{arrows,decorations.pathmorphing,backgrounds,positioning,fit}

usepackage{multicol,rotating,soul,varwidth}
setul{1.5pt}{.4pt}% 1.5pt below contents

% use snippet 'im', 'minted' to add code blocks
usepackage{minted}
% usemintedstyle{trac}

newcommand{cfbox}[2]{%
  colorlet{currentcolor}{.}%
  {color{#1}fbox{color{currentcolor}#2}}%
}
newcommand{code}[1]{{fontspec{DejaVu Sans Mono}{textcolor{violet}{#1}}}}

usetheme{default}
usecolortheme{sidebartab}
usefonttheme{serif}
setbeamertemplate{footline}[frame number]
setbeamertemplate{navigation symbols}{}
setbeamertemplate{blocks}[rounded][shadow=true]
setbeamercolor{block title}{fg=Green}
setbeamercolor{structure}{fg=Green}

begin{document}

begin{frame}
  title{}
  author{Wang Xiaolin}
  titlepage
  vfill
  tiny{
    ding{41} [[email protected]](/web/20190506155039/https://void-shana.moe/cdn-cgi/l/email-protection)
    % ding{37} 
  }
end{frame}

end{document}
% (setq-default TeX-master nil)
```

这个模版里有了你做漂亮幻灯片所需要的一切。光标就停在 `title{}` 的花括号里，那么就开始用中文填空吧。我所有的课程讲义都是从这个模版产生的。


如果你嫌它不好看，那么尽管自由发挥，参考《Beamer用户手册》就可以了：



```
texdoc beamer
```

Happy TeXing!






### 2.7 Emacs Org-mode — 自动生成LaTeX和PDF



Org-mode是Emacs的一个功能模块，它是用来写org文件的。


* 那什么是org文件？就是以 `.org` 结尾的文件。
* 那org文件长什么样？用Emacs打开[这个文件](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.org)，看看就明白了。


换言之，你正在看的这篇小教程就是用org文件生成的。写好的[org文件](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.html)不仅可以生成漂亮的html文件，还可以生成不错的[LaTeX文件](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.tex)、[PDF文件](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.pdf)、[纯文本文件](https://web.archive.org/web/20190506155039/http://cs2.swfc.edu.cn/~wx672/lecture_notes/linux/latex/latex_tutorial.txt)……


* 如你所看到的，org文件的语法极其简单，基本上不用学就能用了。
* Org-mode也有一套方便的快捷键，自己试试吧。
	+ 针对标题的操作：  
	
	
	
	
	
	
	| `C-return` | `C-c C-n` | `C-c C-p` |  |
	| `M-up` | `M-down` | `M-left` | `M-right` |
	| `tab` | `shift-tab` | `C-tab` |  |
	+ 针对列表的操作：  
	
	
	
	
	
	
	| `M-return` | `C-j` |  |  |
	| `M-up` | `M-down` | `M-left` | `M-right` |
	| `shift-left` | `shift-right` |  |  |
	| `alt-shift-left` | `alt-shift-right` |  |  |
	| `tab` |  |  |  |
	+ 针对文章输出的操作：
		- `C-c C-e t` : 修改默认的输出参数
		- `C-c C-e h` : 输出HTML
		- `C-c C-e l` : 输出LaTeX
		- `C-c C-e p` : 输出PDF
		- `C-c C-e a` : 输出ascii（纯文本）
* 除了写点简单的文章，Org-mode最强大的地方是它的“管理”功能，org本就代表 `organization`
	+ 项目管理，任务管理，日程管理，账务管理……
* Org-mode手册就在Emacs里面：

```
C-h i
m
org
C-j
```
* 更多参考资料：
	+ [orgmode.org](https://web.archive.org/web/20190506155039/http://orgmode.org/)
	+ [Tutorials](https://web.archive.org/web/20190506155039/http://orgmode.org/worg/org-tutorials/index.html)
	+ [FAQ](https://web.archive.org/web/20190506155039/http://orgmode.org/worg/org-faq.html)











---


[emacs](https://web.archive.org/web/20190506155039/https://void-shana.moe/category/linux/emacs), [Linux](https://web.archive.org/web/20190506155039/https://void-shana.moe/category/linux), [文章转载](https://web.archive.org/web/20190506155039/https://void-shana.moe/category/%e6%96%87%e7%ab%a0%e8%bd%ac%e8%bd%bd) [C. Linux](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20190506155039/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
justQuiz!开发计划-1](https://web.archive.org/web/20190506155039/https://void-shana.moe/projects/justquiz%e5%bc%80%e5%8f%91%e8%ae%a1%e5%88%92-1.html)
[PREVIOUS 
总结 The 9th Zhejiang Provincial Collegiate Programming Contest](https://web.archive.org/web/20190506155039/https://void-shana.moe/acmalgo/%e6%80%bb%e7%bb%93-the-9th-zhejiang-provincial-collegiate-programming-contest.html)

            
