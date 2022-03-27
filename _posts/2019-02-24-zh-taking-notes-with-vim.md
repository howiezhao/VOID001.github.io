---
title: 将 vim 作为日常笔记本使用
render_with_liquid: false
---
===============



#####  [February 24, 2019](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html "4:27 pm") 
[VOID001](https://web.archive.org/web/20210514125212/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [14 comments](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comments)





本文通过介绍如何使用 Vimwiki, perl graph-easy 以及 git 将 vim 配置为日常的笔记工具。注: 本文内容在于提供一种笔记解决方案，不在于比较各种方法的优劣，如果大家有心仪的推荐方案，欢迎提供探讨


0x00 Preface
------------


Note taking 是很多人日常会进行的活动，因而为自己营造一个良好的 note taking 体验十分重要。说道 plain-text note 想到的第一个当然是 org-mode, 可是虽然 org-mode 是十分好的笔记工具，但我并不是很愿意为了 org-mode 而去使用 emacs, 同时使用两个编辑器对大脑应该是一个伤害（大雾，有关 org-mode 的使用介绍可以看 [用 Org-mode 写编程文档](https://web.archive.org/web/20210514125212/https://blog.poi.cat/post/write-a-programming-doc-with-org-mode) ）。如果你也像我一样是一个 daily vim user，一定会希望能使用 vim 的方式来做笔记。但是当我们搜索相关 plugin 的时候，我们发现可以使用的笔记 plugin 十分有限，且没有较好的介绍这些 plugin 的文章，因而可能自己尝试许久后最终放弃了使用 vim 做笔记。（ 作为我个人而言，我曾经放弃使用 vim 做笔记的原因如下:


* vim 笔记格式不够直观，不能够获得很好的 preview 效果 (如用 markdown 格式做笔记)
* vim 笔记之间跳转 (navigation) 困难，无法灵活的引用内容
* 无法在vim 笔记内画图
* vim 笔记不能打 tag，也就不可能支持使用 tag 搜索相应笔记
* 笔记需要存储在一个 directory 里，没有一个直观的笔记 index 供我们管理笔记（如给标题添加注释，自由的添加删除笔记等）


后续受 @tonyluj 的推荐我尝试了 [vim-notes](https://web.archive.org/web/20210514125212/https://github.com/xolox/vim-notes)。该插件能够对 Markdown Notes 进行管理，Markdown 作为大家经常使用的标记语言，很直观的被我选做 plain text note 的基本 可是在使用的过程中我觉得有一些不易用的因素，如笔记文件名必须为笔记第一行内容，使得我的笔记文件夹下笔记名称出现空格，引号等转义后的字符，造成观感上的不适；另外 vim-notes 的跳转功能也不够强大，不支持创建带有 description 的 Link，因而当 vim-notes 里的 Link 较多的时候，笔记的可读性就降低了。vim-notes 支持搜索功能，可是对我而言，全文的搜索功能不如一个方便的索引功能更加适合，因而我使用了一段时间的 vim-notes 后就放弃了


而后在 @lilydjwg 百合仙子的推荐下，我尝试了 [vimwiki](https://web.archive.org/web/20210514125212/https://github.com/vimwiki/vimwiki) ,这个插件的名字听起来不像 note taking 插件 (对我来说是这样的)，而且同其他 vim note taking plugin 一样，也没有很多文章来介绍 vimwiki 的使用体验。因而这也是本篇文章的目的之一啦，希望本文能够为大家介绍一种舒适的 vim note taking 体验


0x01 Links
----------


vimwiki 第一个吸引我也是他的一个主要功能就是 **Link Navigation**. Vimwiki 初次使用的时候，会建立 note(实际上是 wiki, 因本文描述场景为 note taking 下面全部使用 note 而非 wiki) directory， 用于保存你的所有 note ，同其他插件不同的是，他在创建 note directory 的同时，还会创建一个叫做 **index note** 的文件，这个文件就是你的笔记的目录啦，同时，该文件和你用 vimwiki 创建的其他笔记文件没有任何区别，并且可以使用 Vim 内快捷组合 <Leader>ww 打开该文件（也可使用 `:VimwikiIndex`)， index note 支持所有 vimwiki note syntax. 这是我的 note index 的截图


![](https://web.archive.org/web/20210514125212im_/https://void-shana.moe/wp-content/uploads/2019/02/shot-1024x616.png)index page  
  

可以看到，文档里有很多 Link, 这些 Link 有的 Link 到一个笔记，有的 Link 到一个外部网站，我们还可以对这些笔记进行分类，把不同的笔记放在不同的分类下。  




> link syntax
> 
> [[link|description]] 其中 description 可以省略，省略默认显示 link 的内容, 添加 description 后则只显示 description. link 可以为 笔记文件名，raw URL，file URI (external files), anchor 十分灵活


如上文的 index page 其中的 Link 的 raw format 是这样的: `[[lc-longest-palin-str|Longest Palindromic String]]`，跳转到该 link 的操作也十分简单，**只需要在 Link 上点击回车，就会跳转到相应的内容，使用 backspace 即可返回到上一层**。创建 link 的方式除了手动输入 [[]] 之外，还可以将光标移动到一个单词上点击回车， vimwiki 会自动将其转换为一个 link，如果该 link 指向的 note 不存在，跳转的时候 vimwiki 会自动创建该 note 文件供编辑。


Link 可以为多种格式，它可以是一个 URL，在你使用 Enter 跳转的时候将该 URL 在你的默认浏览器打开，也可以是外部文件，比如一个图片文件，在跳转的时候会使用合适的软件打开该文件 (vimwiki 使用 xdg-open) 同时 Link 支持 subdirectory, 如上面 index page 里的`[ ] [[os/linux-kernel-rcu-000|Read Copy Update Mechanism]]` 的笔记文件链接到 $NOTE\_DIR/os/linux-kernel-rcu-000.mw 这个 note。Link 也支持 anchor ,可以在同一个笔记里跳转到相应的 anchor ，我们可以使用该手段在 note 实现如 footnote 等实用功能。


0x01 Tags
---------


vimwiki 的另一个十分实用的功能是 note tagging



> tag syntax  
> 
> 
> :tag1:tag2:~:tagn: 或者 :tag1: :tag2: ~ :tagn: 支持松散或紧密两种格式


我们在写博客和使用现代笔记软件的时候，往往都希望能够灵活的给笔记打 tag，因为笔记是零散的想法的集合，一篇笔记内容可能交叉属于多个 category ，通过 tagging 我们可以将笔记分类管理起来 。而 vimwiki 对 tagging 的支持也是十分好的，它不仅仅支持给文章打 tag ，还支持给文章的每一个标题打 tag ，并且提供了十分方便的 tag indexing and searching 功能，我们下面就来看一下


使用 `:VimwikiGenerateTags` 可以对现有的全部 tag 生成 index 示例效果如下


![](https://web.archive.org/web/20210514125212im_/https://void-shana.moe/wp-content/uploads/2019/02/image-843x1024.png)使用 Generate Tag 生成的 Tag Index
使用 `:VimwikiSearchTags` 可以搜索含有某个 tag 的 note，也可以使用 `:VimwikiSearch`搜索 note 的全文


0x02 Tables and Graph
---------------------


vimwiki 支持 markdown styled table syntax 并实现了自动对齐，Tab 切换 cell 等功能，在 vimwiki 里创建表格的时候，使用 `:VimwikiTable <row> <col>`创建一个空白的表格，然后进入　insert mode, 编辑单个 cell 的内容后点击 Tab ，表格会自动对齐，如果编辑的 cell 为表格中最后一个 cell ，点击 Tab　后会创建新一行供继续编辑。效果如下 (如果看不到下面的视频说明*网络*不好（x



做笔记的时候我们还需要绘制一些 digram 如 UML 图，流程图，关系图等。vimwiki 并没有支持这样的功能，因而我使用 [graph-easy(AUR)](https://web.archive.org/web/20210514125212/https://aur.archlinux.org/packages/perl-graph-easy/) 实现了该功能，graph-easy 支持直接将 [DOT Language](https://web.archive.org/web/20210514125212/https://www.wikiwand.com/en/DOT_(graph_description_language)) 转换为 ascii digram，我编写了一个简单的 vim plugin 将其集成到了 vim 内，效果如下:



该插件可以通过 Vim 的 Plugin Manager 进行安装 插件地址: [https://github.com/VOID001/graph-easy-vim](https://web.archive.org/web/20210514125212/https://github.com/VOID001/graph-easy-vim)


0x03 Generate HTML
------------------


vimwiki 可以很方便的将内容导出为 HTML ，命令为 `:Vimwiki2HTML :VimwikiAll2HTML` `:Vimwiki2HTMLBrowse`，分别为生成单页 HTML, 将 note directory 下全部文件生成 HTML ，生成单页 HTML 并打开浏览器浏览。vimwiki 会将 HTML 生成在 $NOTEDIR\_html 文件夹下。支持自定义 css file 生成自定义样式的笔记，默认的 HTML 风格是这样的


![](https://web.archive.org/web/20210514125212im_/https://void-shana.moe/wp-content/uploads/2019/02/image-1-1024x665.png):Vimwiki2HTML 生成的 index page 默认样式
0x04 Configuration
------------------


以上就是对 vimwiki 主要功能的一个简单介绍了，本人刚刚开始使用，诸如 Diary Calendar 之类的功能还没有开始使用，而且这两个功能对我的用处较小，因而不对这些功能进行介绍，下面介绍下使用 vimwiki 的基本 configuration


默认 vimwiki 会将 note directory 在 ~/vimwiki 下，我们可以通过配置来更改它



> g:vimwiki\_list option
> 
> vimwiki 支持多个 note , 每一个 note 的配置项是一个 Object， 多个 Object 构成该 g:vimwiki\_list option 的 value.   
> 


我的配置如下:



```
let g:vimwiki\_list = [{
            \ 'path': '~/Documents/Notes/', 'index': 'index', 'ext': '.mw',
            \ 'auto\_tags': 1,
            \ 'nested\_syntaxes': {'py': 'python', 'cpp': 'cpp', 'c':'c'}
            \ }]
nmap <Leader>tt <Plug>VimwikiToggleListItem

```

有关详细的配置，参考 :help vimwiki 的 vimwiki-local-options section，这里只介绍少量的基本设置参数


* path: 该 note 的 directory
* ext: 识别为 note 的文件后缀，为了兼容 git gogs 等的 syntax highlighting 使用了 mediawiki 的后缀 (*.mw *.mediawiki)
* auto\_tags: 是否自动生成 tag，设置为１表示自动生成，即 note 中如果有 tag 就会加入到 vimwiki 的 tag file 中
* nested\_syntaxes: 设置支持的高亮代码块类型，是 key-value pairs


0x05 Syntax
-----------


如果你看到了这里，说明你有可能会尝试下 vimwiki 因而我们在这里介绍下其基本语法。


vimwiki 支持多种语法，包括自己的语法 vimwiki, 并且对 markdown 和 mediawiki 提供了支持 ，我个人推荐使用 vimwiki 语法，该语法和 vimwiki 的各种功能兼容最好，并且只有该格式支持 `:Vimwiki2HTML` 


以下是基本文字效果语法:



```
  *bold text*
  \_italic text\_
  ~~strikeout text~~
  `code (no syntax) text`
  super^script^
  sub,,script,,
```

标题语法为 `= TITLE =` ，该标题为一级标题，二级标题则变为两个 “=”: `== SUB TITLE ==` 以此类推


列表语法基本同 Markdown 一致 注意需要在每一个 Item 和 Mark 之间添加 Space



```
* This is a list item
*This is NOT a list item
1. order list item 1
2. ...

...  支持多种 mark 这里不全部列出
```

pre-formatted text 一般用作在笔记内插入代码，保留其原本的格式，并且使用相应语法的高亮，他的语法如下:



```
{{{lang
    // Code goes here
}}}
```

上面生成 ascii graph 的时候大家已经见过这个语法了，为了保留 ascii graph 原本的 format ，我将其作为一个 pre-formatted code block 处理。


关于其他没有提到的 syntax ，可以参考 `:help vimwiki` 的 vimwiki-syntax section


0x06 Ending
-----------


使用 vimwiki 使我获得了如使用 Modern Note Taking App 一样的体验，并且保留了我的 vim 使用习惯，因为 vim 的灵活性，我们可以进一步提升 note taking 使用体验实现诸如 Cloud Sync, fuzzy tag finder 之类的功能，不知道大家在看完本文后会不会去尝试下 vimwiki 呢，希望能够听到大家的使用心得，同时欢迎大家在评论区介绍自己是如何记录笔记的 🙂  







---


[Linux](https://web.archive.org/web/20210514125212/https://void-shana.moe/category/linux), [vim](https://web.archive.org/web/20210514125212/https://void-shana.moe/category/linux/vim) [archlinux](https://web.archive.org/web/20210514125212/https://void-shana.moe/tag/archlinux), [linux](https://web.archive.org/web/20210514125212/https://void-shana.moe/tag/linux), [vim](https://web.archive.org/web/20210514125212/https://void-shana.moe/tag/vim) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/b7fd1168153a5d7fdd0f395a02a2a11d?s=50&d=identicon&r=g) **[SilverRainZ](https://web.archive.org/web/20210514125212/http://silverrainz.me/)** says: 
[February 24, 2019 at 11:36 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-530)
/me 在使用一个还未存在的自建笔记系统，每次想写点东西都因为「无法解决依赖」而失败。


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[February 24, 2019 at 11:39 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-532)
	可以来试试看 vimwiki 哦，也许真的能成为 LA 日常使用的笔记工具 XD


2. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/89874964e3e524c2cbc10e0087061f77?s=50&d=identicon&r=g) **[Junix](https://web.archive.org/web/20210514125212/https://junyixu.github.io/)** says: 
[February 26, 2019 at 6:20 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-536)
手机上想查看可以用什么方法呢


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[February 27, 2019 at 8:42 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-540)
	可以用一个 github 客户端，在 vim 设置成每次保存的时候 push 到 github 上，然后使用 Android Github Client (e.g: Fasthub) 来查看


3. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[February 28, 2019 at 3:32 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-541)
窝是觉得我自己的笔记本工具最好不要是那种 web editor，所以没选择那些方案


4. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[March 5, 2019 at 10:10 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-545)
根据百合的指导，窝给 vimwiki 记事本添加了自动同步(伪)功能  
然后把里面的路径都替换成你自己的路径就好啦


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/89874964e3e524c2cbc10e0087061f77?s=50&d=identicon&r=g) **[Junix](https://web.archive.org/web/20210514125212/https://junyixu.github.io/)** says: 
	[August 5, 2019 at 5:43 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-778)
	因为懒得再学一种语法，就直接用markdown 了，因为预览直接用 vim 的 markdown 预览插件，md 转换成 html 可以用 pandoc.  


5. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/74242ff548975ef430c690678bd49615?s=50&d=identicon&r=g) **[奶爸笔记](https://web.archive.org/web/20210514125212/https://blog.naibabiji.com/)** says: 
[April 29, 2019 at 12:53 am](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-582)
你这文章好长，引起我Brave提示无响应好几次。我只在vps上用过vim，都不知道你说的是不是和我说的一样东西。


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[April 29, 2019 at 8:56 am](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-583)
	Chrome & Firefox 看都没有问题  


6. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/d219af79b45e5891507fda4c4c2139a0?s=50&d=identicon&r=g) **[repostone](https://web.archive.org/web/20210514125212/https://repostone.home.blog/)** says: 
[May 8, 2019 at 4:48 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-594)
看博主什么时候回来。


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[May 8, 2019 at 5:03 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-595)
	一直在哦，只不过最近没有什么有趣的东西来写 -A-


7. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/d3f616233073c2bf156aa1da5c7ec117?s=50&d=identicon&r=g) **chenbxxx** says: 
[August 23, 2019 at 5:14 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-843)
有没有Vim处理中英文切换的路子 -\_<


8. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/8824e74d66ee336c6352fcfe2d2dd380?s=50&d=identicon&r=g) **muou333000** says: 
[May 8, 2020 at 3:18 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-1122)
你好，问下：能把org里面的文件转到vimwiki里面来么？


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[June 21, 2020 at 3:57 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-1217)
	你好， 我没有尝试过， 不过我认为 org-mode 表达的信息是 vimwiki 的超集， convert 到 vimwiki 会丢失一定的语义，不过 convert 应是可行的


	1. ![](https://web.archive.org/web/20210514125212im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[June 21, 2020 at 3:57 pm](https://web.archive.org/web/20210514125212/https://void-shana.moe/linux/zh-taking-notes-with-vim.html#comment-1217)
	你好， 我没有尝试过， 不过我认为 org-mode 表达的信息是 vimwiki 的超集， convert 到 vimwiki 会丢失一定的语义，不过 convert 应是可行的

            
