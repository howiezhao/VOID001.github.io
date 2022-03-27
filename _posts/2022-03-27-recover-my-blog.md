---
title: Recover My Blog
render_with_liquid: false
---

# TL;DR

我的博客因为欠费 Gone 了，谢天谢地，通过 Archive 和 Google Analytics 我成功恢复了主要的 Content 和 Comments

# TODOs

* [x] Recover my major content from archive
* [x] Make Jekyll work again for my static blog
* [x] Add comments plugin
* [ ] Add RSS feed
* [ ] Add tag and category support

# Background

![pic](/assets/imgs/blog-is-gone.png)

几天前，我发现我的服务器 RSS 拉取长期失败, 心想：肯定是服务器欠费了 ,于是准备去续费, 结果我登陆了我服务器的 Console 错愕的发现，我的服务器已经被 Terminated 了，意味着数据全部 Lost, 我服务器上跑的 TODOBot TTSBot 等无法服务了不说，更重要的是我的 Blog 和我的网盘的数据都丢失了，虽然我的 Blog 有备份，但是备份设备目前不在身边，短期内因为疫情也没法拿到备份盘。顿时感觉事情不妙，好在小伙伴们发现，尽管我的博客服务器 Down 了，但是有部分 Content（访问比较频繁的）被 Archive 存下来了，这才让我稍稍松一口气。因为短期之内没有再次续一个服务器的打算，这次准备将博客就趁这个机会迁移到静态站点上。

同时，我想起来之前我有一个 Github Page repo (也就是这个 Page 所在 Repo), 我可以将它重新启用起来，并且把热门/重要文章加入到这个 Repo 中

于是就有了今天的 **从 Archive & Google Analytics 恢复我的博客**

# Prepare

![pic](/assets/imgs/ga-void-shana.jpg)
![pic](/assets/imgs/archive-void-shana.png)

通过 Archive 和 Google Analytics 我获得了博客的访问信息和历史数据, 同时 Google Analytics 虽然没有提供 API 可以获取到访问最多的 Pages, 但是可将统计数据导出为 CSV, 这样 我们就可以根据这个被访问的页面列表，来从 web archive 上尽可能地恢复相应的 Post 了

我写了一个简单的脚本，读取 CSV 里的 URL 和 Impressions 值，按照 Impressions 由高到低的顺序依次去 Web Archive 获取历史数据: 通过访问 `https://web.archive.org/{your_site_url_here}`, 即可获得这个 URL 对应的最新的一个 Archive 快照（如果有的话）

# Convert to Markdown with FrontMatters

现在可以通过 Web Archive 获取到网页快照了，可是静态站点生成器如 GithubPages 默认使用的 Jekyll, 用的是 Markdown 而不是 Rich Text 或者 HTML, 我们需要使用另一个工具对 Markdown 进行转换，在网上可以找到各种实现的 HTML 2 Markdown converter, 因为我们使用的 Python 编写获取 Archive page 脚本，自然也就选用了 Python 的一个 Library: [turndown](https://github.com/mixmark-io/turndown). 通过此插件将文章 HTML 转成 Markdown。

但是他转换出来的文章内容带有大量我不需要的 CSS 和 JS 等文本，同时因为我的 Wordpress Blog 页面上有着友情链接，分类文本，等多种区块，并不适应 Markdown 文本的展示，需要去掉他们，又简单的写了一个 purify_markdown 的 function, 从已有的 markdown 内容中提取出 Post 的时间，标题，和正文，以及评论

其中时间，标题等信息是 Jekyll 解析 Post 必不可少的内容，而正文和评论，是我最希望留下的两部分内容. 在运行这个脚本的时候，偶然发现 turndown 可能有一些 Bug: 对于一部分 Comment 内容，Convert 出来的 Markdown 对比原文缺少了若干行内容

原文内容:
![p](/assets/imgs/recover-my-blog-bug0.jpg)

转换后:
![p](/assets/imgs/recover-my-blog-bug1.jpg)

最后，这个脚本在每个 Markdown 的开头处增加 "FrontMatter" 也就是文章元数据，使得文章可以被 Jekyll Parse.

# Compatibility Issues

通过上一个步骤，我们已经获得了这些 Markdown 文件，下一步就是生成静态站点，在本地测试 `jekyll serve` 发现报错

![p](/assets/imgs/recover-jekyll-exception.jpg)

这个问题的原因是因为我的文本中有一些正文带有 `{{  }}` (对，比如本文现在也带有了 XD), 通过查阅资料发现可以通过指定一个 Flag(jekyll > 4.0 有效) `render_with_liquid: false` 来关闭某些文章对 Liquid 的渲染, 但是 Github Page 默认的 Jekyll 版本还是 < 4.0 不支持这个 Flag 的, 于是我只好抛弃了 Github 原生的 Github Page Generator, 采用 Github Action 自定义生成规则，并且为后续换用其他 Static Site Generator 提供条件.

# Miscs

作为一个博客，我希望他是能够让他人进行 comment 并且互动的，因而我选用了 (giscus)[https://giscus.app/] 作为静态站点博客的评论工具, 同时保留了历史评论在文章正文中。
