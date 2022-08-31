**Matrix 首页推荐** 

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。

文章代表作者个人观点，少数派仅对标题和排版略作修改。

___

## 博客探索

2005 年，我开始用博客记录感想，发布在 MSN space 平台和短命的 Google Sidewiki 上。六年后，MSN space 关闭 ，博客被转移到 WordPress 托管，我改用 Blogger，没多久就暂停了博客。

2018 年，我偶然接触到 [Jekyll](https://jekyllrb.com/)，被其简洁的界面和便捷性打动，重新恢复了博客记录。博客方向从感想记录转变到知识整理输出。Jekyll 方案需要首先在本地用 [[Markdown]] 编辑排版，然后同步到 GitHub 发布，最后以 [[Markdown]] 格式手动分发到各个渠道。当文章较少时，这套方案的体验感特别好。

到了 2021 年，**随着文章和发布渠道的增多，文章的修改和管理变得愈加困难**。慢慢地，我开始习惯本地 [[Markdown]] 只做初稿排版，更新则只在外部平台上进行。

我的文章都是工具教程类，要随着工具的更新而修改，有时甚至要对几年前发布的文章进行更新。因此，针对少量平台更新的策略，带来了文章版本混乱，让博客偏离了知识记录的初衷。为了保证文章版本统一，我把博客从 Jekyll 迁移到 WordPress，准备以 WordPress 作为统一版。

![](https://cdn.sspai.com/editor/u_/cafb7ptb34tbmlhk7110.png?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

然而，WordPress 方案很快就被放弃了。原因除了 WordPress 糟糕的编辑体验，更重要的是，我遇到了 Notion。其界面美观，编辑方便，支持对外展示，能导出为 [[Markdown]]、HTML 文件。

可是，没什么平台是完美的。Notion 不支持同步本地 [[markdown]] 内容，图床不能在站外使用，国内访问速度成谜。这令 **Notion 只适合个人分享，而非博客网站**。我用 Notion，纯属垂涎美色。

2022 年，由于疫情被封控在家两个月。时间多了，我继续折腾博客，希望找到一个界面美观，能自动发布且具备本地管理功能的博客方案。

## 博客方案

最初，我幻想着修改一篇文章同步到多个平台，但找了许久也没有合适的。网上所谓的一键分发软件，实际上是通过网页操作来完成发布，并不能自动修改更新。

剔除掉这类不现实的想法后，新的博客方案以 [[Markdown]] 版本为主，自动同步 WordPress，最后手动同步主要分发平台。

**最终方案**如下：

1.  初稿：Markdown 本地编辑文章，使用七牛云自建图床。
2.  发布：同步本地 Markdown 文本，自动发布，保持主要平台内容为最新。
3.  管理：本地更新修改 Markdown 文件，docsify 页面整合文本内容，博客后台管理文章版本。
4.  订阅：用户能通过 RSS、邮件或微信来订阅博客更新。

## 发布工具：WordPressXMLRPCTools

[WordPressXMLRPCTools](https://github.com/zhaoolee/WordPressXMLRPCTools) 能用 Markdown 生成博客，推送更新到 Github 后，通过 Github Actions 自动将文章更新到 WordPress，并将 WordPress 网站的文章索引更新到 Github 仓库的`README.md`，供搜索引擎收录。

基于 WordPressXMLRPCTools，我做了两点修改：

-   草稿箱：`_post`路径内新建`TEMP`文件夹，用于存放文章草稿。WordPress 推送程序会忽略`_post`子文件夹的内容，换言之，`TEMP`文件夹不会发布到 WordPress 网站。
-   文章聚合页：主目录新增`.nojekyll`，`index.html`，`_sidebar.md`文件，引入文档生成工具 docsify，将博客文章聚合在一个页面，方便快速定位和位置管理。

示例：[https://rockbenben.github.io/Blog\_WP/](https://rockbenben.github.io/Blog_WP/) 或 [https://docs.newzone.top/](https://docs.newzone.top/)

![](https://cdn.sspai.com/editor/u_/cafb7q5b34tbmn6f8g8g.png?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

### 使用流程

1.  进入[项目页面](https://github.com/rockbenben/Blog_WP)，点击「Use this template」，复制模板文件。
2.  回到你新建的 repo，删除 \_post 文件夹中的所有文件，参照主目录下`example_article.md`的格式编辑文章。
3.  按[WordPressXMLRPCTools 安装步骤](https://github.com/zhaoolee/WordPressXMLRPCTools#%E7%94%A8github-actions%E5%86%99markdown%E6%96%87%E7%AB%A0%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%E5%88%B0wordpress)执行，如遇报错，查看下方使用问题。
4.  修改主目录下的`index.html`和`_sidebar.md`文件，调整 docsify 网页设置。

-   `index.html`修改 docsify 网页标题、描述和关键词。
-   `_sidebar.md`修改 docsify 网页侧边栏，加入博客文章的标题和位置。

### 使用问题

#### **文章发布不成功**

`_post`文件夹添加了文档，但同步后，`README.md`和 WordPress 并没有添加文章。

检查以下两点：

-   文章后缀必须为「.md」，不支持「.markdown」或其他后缀格式。
-   进入 repo 页面中的`Actions`，检查最近一次的 update 是否正确。

#### Error: git denied to github-actions\[bot\]

遇到 GitHub Actions 报错：`git denied to github-actions[bot]`和`Process completed with exit code 128`。

依次点击该 repository 的`Setting - Code and automation - Actions - General`，然后在 Workflow permissions 中开启「Read and write permissions」。

#### Error: Process completed with exit code 1

遇到 GitHub Actions 报错：`Error: Process completed with exit code 1`，检查服务器是否开启了防火墙，含代码的文章容易被误认为木马。暂时关闭服务器防火墙，如 Nginx 防火墙、宝塔系统加固，可解决该问题。

#### 无法覆盖更新原文章

修改旧文章并同步后，WordPress 站的文章没同步修改，而是新增了一篇相同的文章。这是 WordPressXMLRPCTools 项目的 bug。项目作者 @zhaoolee 说，「**只要不改文件名，就可以通过更新 markdown，更新对应的文章内容。**」

但我和 @clairyitinggu 在未改文件名的情况下，都没能更新对应文章内容，而是重新发布了篇新文章。如果你也遇到相同的问题，建议手动将新文章内容覆盖旧文章，然后删除新文章。

这个 bug 可以当作是强提醒。当 WordPress 新增了旧文章，你就被提醒要在其他平台修改该文章，让文章版本保持统一。

#### WordPress 发布时间与实际不符

同步文章后，WordPress 显示的文章发布时间是 GitHub push 时间，而非文章真实的发布时间。

如果你将旧文章转移到 WordPress，文章的发布时间需在 WordPress 后台手动修改，无法在Markdown 文件中指定 WordPress 显示的发布时间。

## 本地管理 Markdown 文章

如果用 Windows 资源管理器管理 Markdown 文章，会存在 3 个问题：

-   资源管理器的视觉效果非常难看。
-   Markdown 文件名称不能展示关键信息，较难定位文档。文章越多，管理越困难。
-   无法对文章内容进行本地检索，只能通过文件名称猜测内容。

为解决这些问题，我借助飞书表格、RunAny 和 docsify 重构本地文章管理方案。

### 飞书文档管理

[飞书文档](https://www.feishu.cn/product/sheets) 功能与 Notion、Airtable 类似，可将文字、链接、图片聚合在同一页面，操作便捷。

打开飞书多维表格，填入本地 Markdown 文章的标题、本地位置、链接、标签和封面，即可聚合本地文章的关键信息。将表格视图切换为「画册视图」，文档管理界面更达到 90% 的 Notion 视觉效果。

![](https://cdn.sspai.com/editor/u_/cafb7qtb34tbmvch3s9g.png?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

![](https://cdn.sspai.com/editor/u_/cafb7r5b34tbmlhk711g.png?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

### RunAny 文档直达

在线文档中，无法像打开超链接一样直接打开本地文件。如果你想节省中间打开时间，可以使用 RunAny。

[RunAny](https://hui-zz.gitee.io/runany/#/) 是基于 AutoHotKey 的一键启动软件。按下方配置后，点击飞书表格中的「本地位置」，即可使用默认编辑器打开 md 文件。如果你的默认编辑器是 notepad++，则将下方命令中的`Code.exe` 替换为`notepad++.exe`。

```
;将 Runany 主目录下的 RunAny.ini 文件内的「编辑」模块替换为下方命令
-编辑(&Edit)
 --编程|cmd bat md ahk html js css json
 vscode|Code.exe
```

### docsify 全文检索

飞书表格可以搜索关键元素，但不能对检索全文。这时，我们需要使用 [docsify](https://docsify.js.org/#/)，一款能将 markdown 文档自动生成网站的工具，相当于轻量级的 GitBook。

docsify 使用简单，如果使用了前文我修改过的[发布工具](https://github.com/rockbenben/Blog_WP)，则无需配置。在发布工具文件夹内的空白区域，右键打开终端，执行命令 `docsify serve` 即可生成全文检索网页，默认管理链接为 `http://localhost:3000/#/`。

![](https://cdn.sspai.com/editor/u_/cafb7rdb34tbmqchrugg.png?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

如果你设置了 Github Pages，项目会默认启动在线 docsify 网页，页面链接取决于 Github 的设置。比如我的用户名是`rockbenben`，项目名是`Blog_WP`，所以 docsify 管理页面就是 [https://rockbenben.github.io/Blog\_WP/](https://rockbenben.github.io/Blog_WP/)。

40% 的网站基于 WordPress 架构，因此 WordPress 有超多的主题和插件，可以实现你想要的功能，比如 RSS、Newsletter。

如果你不是小白，又拥有较多的粉丝，可以使用 [Substack](https://substack.com/) 和 [竹白](https://zhubai.love/) 来分发博客。这两者都支持Newsletter 付费订阅。只针对国内用户的话，竹白可支持微信订阅。

## 后续

比起原来的 Jekyll，新方案的配置要复杂些，但使用并不难，推荐稿件多的人采用。

折腾新方案的过程中，我发现了篇 2021 年初写的文章。当时，稿子写到 90%，我就去忙其他事，忘了这篇文章。等到这次被发现，而它已经在草稿箱待了一年半。

用了新方案，稿件管理会变得很简单，稿件遗忘、找不到的情况也会减少许多。最近我出稿速度大增，也都跟这有关，都是从草稿箱捡回来的半成品。

写完这篇稿子，疫情封控也正好结束，终于可以出门了，希望永远别给我「免费假期」了。

#### 关联阅读

-   [WordPress 有力竞争者，高颜值全能博客平台：Ghost](https://sspai.com/post/65602)
-   [用 GitHub 搭建静态博客太繁琐？用这个小工具实现「傻瓜式」发布](https://sspai.com/post/58013)
-   [从部署到思考，我的 Ghost 博客搭建手记](https://sspai.com/post/68855)

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀