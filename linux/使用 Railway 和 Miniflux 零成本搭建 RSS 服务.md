[![Featured image of post 使用Railway和Miniflux零成本搭建RSS服务](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway/obi-pixel6propix-UEQvUtRs224-unsplash_hu30910665284931ace4f57faa1e01d828_222582_800x0_resize_q75_box.jpg)](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway.html)

[技术](https://blog.cysi.me/categories/%E6%8A%80%E6%9C%AF.html)

### Miniflux 是一款优秀的 RSS 聚合软件，相较于 FreshRSS，TT-RSS 等同类，它更加简约迅速但主要功能却一个不少。更重要的是因为它的 v2 版本使用 Go 语言编写，因此可以直接运行在一些 PaaS 平台上，个人用户完全可以使用这些平台零成本搭建一个属于自己的 RSS 聚合阅读器。

RSS（聚合内容），一种 “古老” 的消息来源格式，古老到连著名的 RSS 聚合器兼阅读器 Google Reader 都快停止服务十年了。但近几年随着开源项目 [RSSHub](https://docs.rsshub.app/) 发布极大程度上丰富了可订阅内容，加之 RSS 本身无算法、纯用户主导（须用户主动订阅）的特性和对各互联网公司的算法推荐的唾弃，让 RSS 这一古老协议焕发了第二春。我也是从那时候开始**重新**依赖 RSS 获取信息。

想要使用 RSS 作为自己的信息获取工具，就一定需要一个 RSS 聚合工具，将你所感兴趣的信息源全部集中到一处，通常可以使用下面的几种方案：

-   本地应用程序（如 Reeder），但这种方案无法原生多设备同步，订阅源和阅读状况只在单台设备内存储，必须配合商业网络服务或自建服务来跨设备同步内容。
-   商业订阅服务（如 Feedly、Feedbin），但大多数需要每月支付一定的订阅费用。
-   自建服务（如 FreshRSS、tt-RSS、Miniflux），需要花一定的心思自行搭建，大多需要自行准备服务器，同样需要一定的物质（金钱）成本。

今天我们要讨论的是第三种方案，目前常用的自建 RSS 聚合器框架主要有：

-   [FreshRSS](https://freshrss.org/)，功能强大、易于使用，基于 PHP 和 SQL 数据库
-   [Tiny Tiny RSS](https://tt-rss.org/)（tt-RSS），功能非常强大、扩展性极强，基于 PHP 和 SQL 数据库
-   [Miniflux](https://miniflux.app/)，极度简约但应有的功能一个不缺，基于 Go 语言

可以看出，前两个框架虽然功能上更强大且搭建起来比较简单（可以使用 Docker，或者直接使用 PHP 虚拟主机搭建）但受限于 PHP+SQL 的组合，整个框架都显得比较 “重” 或者慢，且无法做到真正的 0 成本。因此我们将目光转向到了使用 Go 语言便携的简约框架 Miniflux。

事实上 Miniflux 官方就给出了[使用 Heroku 搭建的教程](https://miniflux.app/docs/howto.html#heroku)，但 Heroku 免费版会定时休眠且经过测试发现无法使用 ping 工具保持真唤醒状态（为什么说 “真唤醒”？因为此时在 Heroku 控制面板中该服务并没有显示在休眠，但实际上它已经处于待机状态了），严重影响首次访问速度，体验较差。

## Railway：不休眠的 PaaS 平台

虽然本文的目的是 0 成本搭建，但也并不意味着我们就要因此妥协使用体验。这时候我们可以把目光转向另一个 PaaS 平台：[Railway](https://railway.app/?referralCode=yukiakari)（该链接含 Referral Code)

![Railway.app网站主界面](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway/image-20220525114437152.png)

这是一个 2020 年成立的 PaaS 平台，无需信用卡的免费版可以提供 5 美元的额度，如果绑定信用卡则每个月可以使用 10 美元的额度，用户仅需支付超过该额度的使用费，对于普通个人用户而言，5 美元的额度早就足够。

—— 更重要的是，它支持数据库和 Docker，所以比起 Vercel 或 Netlify，它更像是 Heroku 的替代品，更优质的替代品。这就为我们达成 0 成本搭建 RSS 聚合器且不损失使用体验带来了可能。

## 动手搭建

说了这么多，现在让我们开始动手搭建基于 Miniflux+Railway 的专属于自己的 RSS 聚合器。

需要注意的是，为了让更多人能够自行搭建服务，因此本文中所有的操作都尽量使用有图形化的程序并使操作尽可能简单。

### 准备工作

正式开始之前，我们需要准备：

-   [GitHub](https://github.com/) 账号
-   [Railway](https://railway.app/?referralCode=yukiakari) 账号
-   [GitHub Desktop](https://desktop.github.com/) 客户端（可选，图形化管理更加简单，或者你可以直接使用 Git）
-   一台电脑

首先我们需要去 [Miniflux v2 的 GitHub 页面](https://github.com/miniflux/v2)并 Fork 该项目到自己的账户中。完成后打开并登录 GitHub Desktop 客户端， 点击顶部菜单栏的 “File - Clone a repository”，找到自己刚刚 Fork 的项目，选择好下载目录位置并点击 Clone 按钮，将该项目下载到本地。

### 创建稳定版分支

由于 Miniflux 的默认分支（master）本身其实是不稳定的开发版，直接部署使用开发版多多少少会遇到各种问题，为了保证服务的稳定性和可用性，故我们还需要单独创建一个稳定版分支。

![找到最新的稳定版版本号](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway/image-20220525123046584.png)

让我们回到 Miniflux 的 GitHub 页面，注意上图标黄的 “Release” 部分，可以看到当前稳定版版本号为 `2.0.36`，这也是该 Release 对应的 `tag`，记住这个版本号，在 GitHub Desktop 打开下载到本地的项目，点击左栏的 “History”，找到带有对应版本号的一栏，右键点击 “Create a branch from commit”，在弹出的窗口中输入新的分支名 `stable` 并 Create branch，完成后，顶部的第三个大按钮会变成 “Publish branch”，点击它使之同步到 GitHub 仓库中。至此，稳定版分支创建完成。

![创建新的稳定版分支](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway/image-20220525124538641.png) ![提交分支到云端](https://blog.cysi.me/2022/05/build-miniflux-rss-on-railway/image-20220525124707226.png)

### 更新稳定版分支

如果要更新你的稳定版分支，做法也很简单，首先确定 Miniflux 已经发布了新的稳定版，打开 GitHub Desktop，点击客户端窗口上的 “Current branch”，先从本地删掉 `stable` 分支，再重复上一节里面的创建稳定版分支的操作即可。

### 使用 Supabase 数据库

Miniflux 需要用到 PostgresSQL 作为数据库，虽然 Railway 本身可以直接创建数据库，但为了避免账单爆炸，还是建议使用 [Supabase](https://supabase.com/) 这类数据库服务。

注册完成后，创建一个 Project，数据库位置建议选择美东或者美西，设定并记住数据库密码，等待创建完成。

数据库创建好后，进入该 Project，在左侧菜单里面选择 Settings - Database，找到页面的 `Connection string`，获取你的数据库 URL，形如 `postgresql://postgres:[YOUR-PASSWORD]@db.xxxxxx.supabase.co:5432/postgres`，记得把 URL 中间的 `[YOUR-PASSWORD]` 改成自己先前设定的数据库密码，并复制备用。

## 部署到 Railway

现在我们可以将稳定版分支部署到 PaaS 平台，比如我们接下来会用到的 Railway。

进入 Railway 控制面板，新建项目（New Project）- Deploy from GitHub repo - 绑定 GitHub 账号并选择我们在上一节做好稳定版分支的项目，这时系统会自动开始部署，因为我们还没有设置数据库和环境变量，所以这次部署一定会失败，暂时不需要理会。

接着，回到刚刚连接的 GitHub Repo 模块，点击它，在右侧的弹出的窗口，导航到 “Settings” 选项卡，将 “Deployment Trigger” 下面的选项从 `master` 改为 `stable`，即切换为我们在上一节里面增加的稳定版分支。同时，你还可以在 “Service Domains” 里面自定义该服务的域名（免费提供 up.railway.app 子域名)，或绑定自己的域名。

完成后切换到 “Variables” 选项卡，点击右侧的 “RAW Editor”，将下面的变量按需修改并**去掉注释后**复制粘贴到文本框内，点击 Update 即可。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="fallback">DATABASE_URL=上一节复制留存的PostgreSQL链接
PROXY_IMAGES=all
RUN_MIGRATIONS=1
BASE_URL=https://域名，如果你无法自备域名则可以直接填写免费提供的子域名
ADMIN_PASSWORD=管理账户密码
ADMIN_USERNAME=管理账户用户名
CREATE_ADMIN=1
</code></pre></td></tr></tbody></table>

提交了环境变量后 Railway 会自动重新部署，可以在 “Deployments” 选项卡里面查看进度。完成后即可登录使用。Miniflux 原生支持 Fever 和 Google Reader API，可以在 Miniflux 的设置中启用，这样即可配合相应的 RSS 阅读客户端使用（iOS 推荐 Reeder，Android 推荐 FeedMe）。

**2022 年 7 月 27 日更新** 如果部署后无法使用，log 中提示 `app not found`，目前推测原因是 Railway 新使用的 Nixpacks Builder 的问题，在 Railway 里面，点击你的 Miniflux 模块的 Settings，找到 Builder，换成 Heroku，等待重新部署即可。

至此，一个简约但不简单的 0 成本 RSS 聚合器就搭建完成了！快去订阅自己喜欢的源享受干净可控的阅读环境吧！

有了自己的 RSS 聚合器后，剩余的工作就是寻找并订阅自己喜欢的内容（源）了，这里推荐使用 [RSSHub](https://rsshub.app/)，让各种原本无法使用 RSS 的平台支持 RSS，可以极大幅度丰富订阅内容。RSSHub 官方虽然有在免费提供服务，但由于访问量太大，较多网站会对之启用反爬措施，但好在零成本搭建一个 RSSHub 服务非常简单，可以阅读[这篇文章](https://qufy.me/post/%E6%89%93%E9%80%A0%E4%BD%A0%E7%8B%AC%E4%BA%AB%E7%9A%84-rss-%E9%98%85%E8%AF%BB%E7%8E%AF%E5%A2%83-rsshub-%E4%B8%8E-miniflux-%E8%87%AA%E5%BB%BA%E6%8C%87%E5%8D%97/#rsshub)了解如何使用 Vercel 免费搭建一个自己的 RSSHub。