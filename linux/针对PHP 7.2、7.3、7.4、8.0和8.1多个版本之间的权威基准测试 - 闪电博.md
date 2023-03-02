![针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/php-benchmarks-2022-1024x512-1.jpg "针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试")

2021年对于PHP来说是多事之秋。PHP 8.0已经一岁了，万众瞩目的PHP 8.1于2021年11月25日发布，带来了许多令人兴奋的功能。您可以在我们的深入文章中了解所有最新的[PHP 8.1功能](https://www.wbolt.com/php-8-1.html)。

我们针对各种PHP平台发布深入的性能基准测试，以了解不同的PHP版本如何相互叠加。今年，我们在**14个独特的PHP平台/配置中****对5个不同的PHP版本进行了基准测试**，包括WordPress、Drupal、Joomla、Laravel、Symfony等等。我们还测试了其他流行的PHP平台，例如[WooCommerce](https://www.wbolt.com/woocommerce-tutorial.html)、[Easy Digital Downloads](https://www.wbolt.com/easy-digital-downloads.html)、October CMS和Grav。

我们始终鼓励使用受支持的最新PHP版本。它们不仅是最安全的，而且还提供了许多性能改进。今天，我们将向您展示PHP 8.0和8.1如何在几乎所有我们与之抗衡的事物中脱颖而出。

1.  [PHP的现状](https://www.wbolt.com/php-benchmarks.html#the-state-of-php)
2.  [PHP基准测试](https://www.wbolt.com/php-benchmarks.html#php-benchmarks-2022) 
3.  [WordPress 5.9-RC2](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2)
4.  [WordPress 5.9-RC2 + WooCommerce 6.1.1](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2--woocommerce-611)
5.  [WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2--easy-digital-downloads-21141)
6.  [Drupal 9.3.3](https://www.wbolt.com/php-benchmarks.html#drupal-933)
7.  [Joomla！4.0.6](https://www.wbolt.com/php-benchmarks.html#joomla-406)
8.  [Grav 1.7.29](https://www.wbolt.com/php-benchmarks.html#grav-1729)
9.  [OctoberCMS 1.3.1](https://www.wbolt.com/php-benchmarks.html#octobercms-131)
10.  [Laravel 8.80.0](https://www.wbolt.com/php-benchmarks.html#laravel-8800)
11.  [Symfony 5.4.2](https://www.wbolt.com/php-benchmarks.html#symfony-542)
12.  [CodeIgniter 4.1.8](https://www.wbolt.com/php-benchmarks.html#codeigniter-418)
13.  [CakePHP 4.3.4](https://www.wbolt.com/php-benchmarks.html#cakephp-434)
14.  [Craft CMS 3.7.30.1](https://www.wbolt.com/php-benchmarks.html#craft-cms-37301)
15.  [Kirby 3.6.1.1](https://www.wbolt.com/php-benchmarks.html#kirby-3611)
16.  [Flarum 1.2.0](https://www.wbolt.com/php-benchmarks.html#flarum-120)
17.  [更新到PHP 8.1](https://www.wbolt.com/php-benchmarks.html#update-to-php-81)

### PHP的现状

PHP（PHP的递归首字母缩写词：Hypertext Preprocessor）是使用最广泛的服务器端脚本和编程语言之一。它是开源的，主要用于Web开发。由于PHP为大部分核心WordPress软件提供支持，因此它是WordPress社区非常重要的语言。

![PHP Logo](https://static.wbolt.com/wp-content/uploads/2022/02/php-logo-official-1100px-1024x570.png "PHP Logo")

PHP Logo

虽然有些人可能会认为PHP已死，但事实并非如此。[根据W3Techs的数据](https://www.wbolt.com/go?_=8123d73b83aHR0cHM6Ly93M3RlY2hzLmNvbS90ZWNobm9sb2dpZXMvZGV0YWlscy9wbC1waHA%3D)， **78.1%**的网站都使用PHP，他们知道其服务器端编程语言。这几乎是**5个网站中的有4个在用PHP**！

PHP比以往任何时候都更有活力、更快、更好。

![PHP位于服务器端编程语言的最顶端](https://static.wbolt.com/wp-content/uploads/2022/02/Server-side-Languages-PHP-W3Techs-1024x736.png "PHP位于服务器端编程语言的最顶端")

PHP位于服务器端编程语言的最顶端

如果你觉得它已经死了，我们想知道什么是活着的！即使[与JavaScript](https://www.wbolt.com/php-vs-javascript.html)及其新的服务器端实现相比，PHP在它旁边也显得高大而自豪。

但是，PHP社区存在一个大问题。许多网站仍在使用过时的版本和不受支持的PHP安装。[根据W3Techs的数据](https://www.wbolt.com/go?_=8123d73b83aHR0cHM6Ly93M3RlY2hzLmNvbS90ZWNobm9sb2dpZXMvZGV0YWlscy9wbC1waHA%3D)，**29.9%的网站仍使用PHP 5.6或更低版本**。

![WordPress PHP版本](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress-PHP-Versions-2022Feb01-1024x819.png "WordPress PHP版本")

WordPress PHP版本（截至2022年2月1日）

在[WordPress统计数据](https://www.wbolt.com/go?_=5282831f7eaHR0cHM6Ly93b3JkcHJlc3Mub3JnL2Fib3V0L3N0YXRzLw%3D%3D)方面，只有**50.6%**的网站运行在[受支持的PHP版本](https://www.wbolt.com/go?_=d7d2e0a671aHR0cHM6Ly93d3cucGhwLm5ldC9zdXBwb3J0ZWQtdmVyc2lvbnMucGhw)（7.4 或更高版本）上。更糟糕的是，所有WordPress网站中有**10.2%**运行在PHP 5.6或更低版本上。它比整个PHP社区要好，但许多网站的后门都敞开着。

我们认为这个难题有很多原因：

-   WordPress社区缺乏关于PHP及其在WordPress中的关键作用的教育。
-   在较新的PHP版本（尤其是PHP 8.0及更高版本）上运行的插件和主题的兼容性问题。
-   WordPress托管服务提供商因为担心给客户带来问题而不愿推出新的PHP版本。

我们都应遵循与PHP相同的[寿命终止 (EOL) 时间表](https://www.wbolt.com/go?_=d7d2e0a671aHR0cHM6Ly93d3cucGhwLm5ldC9zdXBwb3J0ZWQtdmVyc2lvbnMucGhw)来解决这个令人不安的问题。它有助于让我们托管的所有WordPress网站尽可能快速和安全。

**注意：** PHP 8.0带来了许多重大变化，所以很多用户还没有转向它。但是，我们预计很快会有更多网站转向它。

### PHP基准测试 (2022)

尽管没有积极支持PHP 7.2、7.3和7.4 ，但许多网站仍在它们上运行。因此，我们决定测试五个不同的PHP版本，以便您了解较新的PHP版本在性能方面的表现。

今年的热门选择当然是新发布的PHP 8.1。这是PHP世界中最新、最激动人心的发展，这是有充分理由的。并非所有基于PHP的框架和CMS都完全支持它，但我们已经尽可能多地对其进行了测试。

我们为每个测试使用了每个平台的最新版本，并针对**1,000个请求对其中一个具有****15个并发用户**的URL进行了基准测试。我们多次进行基准测试以确保结果一致。此外，我们只考虑了前3个结果的平均值。

您可以在下面找到我们测试环境的详细信息：

-   **机器：** Intel Xeon（30核CPU）、120GB RAM、1TB HDD。它是由Google Cloud Platform提供支持并在隔离容器中运行的计算优化 (C2)虚拟机。
-   **操作系统：** Ubuntu 20.04.1 LTS（Focal Fossa）
-   **网络服务器：** Nginx 1.18.0 (nginx/1.18.0)
-   **数据库：** MariDB 10.5.8 (MariaDB-1:10.5.8+maria~focal)
-   **PHP版本：** 7.2、7.3、7.4、8.0、8.1
-   **页面缓存：**在所有平台和配置上禁用。
-   **OPcache：**使用[推荐的php.ini设置](https://www.wbolt.com/go?_=74a858001baHR0cHM6Ly93d3cucGhwLm5ldC9tYW51YWwvZW4vb3BjYWNoZS5pbnN0YWxsYXRpb24ucGhw)在所有平台和配置上启用OPcache ，除了`opcache.max_accelerated_files`我们从**4000**提高到**50000**的值。使用的OPcache设置是：

```
opcache.memory_consumption=128 
opcache.interned_strings_buffer=8 
opcache.max_accelerated_files=50000 
opcache.revalidate_freq=2 
opcache.fast_shutdown=1 opcache.enable_cli=1
```

由于[OPcache](https://www.wbolt.com/go?_=21a9854b2daHR0cHM6Ly93d3cucGhwLm5ldC9tYW51YWwvZW4vaW50cm8ub3BjYWNoZS5waHA%3D)通过将预编译的脚本字节码存储在服务器的共享内存中来提高PHP性能，因此它消除了PHP为每个请求加载和解析脚本的需要。

#### 测试的PHP平台和配置

我们的基准测试包括以下14个PHP平台/配置。单击下面的任何一个以直接跳至其测试结果和注释。我们以**每秒请求数**来衡量数据。请求越多越好。

1.  [WordPress 5.9-RC2](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2)
2.  [WordPress 5.9-RC2 + WooCommerce 6.1.1](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2--woocommerce-611)
3.  [WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1](https://www.wbolt.com/php-benchmarks.html#wordpress-59rc2--easy-digital-downloads-21141)
4.  [Drupal 9.3.3](https://www.wbolt.com/php-benchmarks.html#drupal-933)
5.  [Joomla！4.0.6](https://www.wbolt.com/php-benchmarks.html#joomla-406)
6.  [Grav 1.7.29](https://www.wbolt.com/php-benchmarks.html#grav-1729)
7.  [OctoberCMS 1.3.1](https://www.wbolt.com/php-benchmarks.html#octobercms-131)
8.  [Laravel 8.80.0](https://www.wbolt.com/php-benchmarks.html#laravel-8800)
9.  [Symfony 5.4.2](https://www.wbolt.com/php-benchmarks.html#symfony-542)
10.  [CodeIgniter 4.1.8](https://www.wbolt.com/php-benchmarks.html#codeigniter-418)
11.  [CakePHP 4.3.4](https://www.wbolt.com/php-benchmarks.html#cakephp-434)
12.  [Craft CMS 3.7.30.1](https://www.wbolt.com/php-benchmarks.html#craft-cms-37301)
13.  [Kirby 3.6.1.1](https://www.wbolt.com/php-benchmarks.html#kirby-3611)
14.  [Flarum 1.2.0](https://www.wbolt.com/php-benchmarks.html#flarum-120)
15.  [更新到PHP 8.1](https://www.wbolt.com/php-benchmarks.html#update-to-php-81)

由于每个平台上的演示内容可能会有很大差异，我们测试了他们的准系统安装的原始性能。这里的目标是对各种PHP版本进行基准测试——CMS和框架仅用作工具。您不应该使用这些基准测试结果来衡量一个平台与另一个平台，而是它如何在不同的PHP版本上与自己竞争。

我们还包括了它们的大小和屏幕截图，让您更好地了解测试的页面。有些很小，有些则很大。

### WordPress 5.9-RC2

[WordPress](https://www.wbolt.com/go?_=8e7daf5bbaaHR0cHM6Ly93b3JkcHJlc3Mub3JnLw%3D%3D)是我们测试的第一个平台。毕竟，它为您正在阅读的这个博客和互联网上所有网站的43.0%提供了支持。它是一款免费的开源软件，可用于创建精美的网站、博客和应用程序。

![WordPress](https://static.wbolt.com/wp-content/uploads/2022/02/wordpress-logo-1-1024x209-1.png "WordPress")

我们从WordPress 5.9-RC2（Release Candidate 2）开始，这是本文基准测试时的最新版本。它安装了新的二〇二二主题。我们针对**15个并发用户的****1000个请求**对URL进行了基准测试。相同的方法用于所有其他测试。

![测试的WordPress页面](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress-5.9-RC2-PHP-Benchmarks.jpg "测试的WordPress页面")

测试的WordPress页面

-   **网址测试：** `/hello-world/`
-   **主题：**二〇二二
-   **注意：**博客页面包括带有文本徽标的标题、导航菜单、文章正文、一条评论和页脚小工具，例如搜索、最近的文章和最近的评论。
-   **图片来源：** [WordPress.org](https://www.wbolt.com/go?_=3e7580b741aHR0cHM6Ly93b3JkcHJlc3Mub3JnL2Rvd25sb2FkLw%3D%3D)

> **注：**基准数据以每秒请求数为单位。请求越多越好。

![WordPress 5.9-RC2 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/PHP-Benchmarks_graphs_WP-5.9-RC2.png "WordPress 5.9-RC2 PHP基准测试")

WordPress 5.9-RC2 PHP基准测试

#### 基准测试结果

-   WordPress 5.9-RC2 PHP 7.2基准测试结果：106.56 req/sec
-   WordPress 5.9-RC2 PHP 7.3基准测试结果：108.45 req/sec
-   WordPress 5.9-RC2 PHP 7.4基准测试结果：110.24 req/sec
-   WordPress 5.9-RC2 PHP 8.0基准测试结果：111.10 req/sec
-   **WordPress 5.9-RC2 PHP 8.1基准测试结果：163.43 req/sec 🏆**

PHP 8.1是明显的赢家，比PHP 8.0快**47.10% 。**考虑到所有其他结果的接近程度，这在这里令人惊讶。如果将它与PHP 7.2进行比较，它每秒可以处理**超过50%的请求（或事务）**。

> **提醒：**更广泛的WordPress生态系统（插件、主题、开发工具等）中的PHP 8.1支持状态几乎无法得知。如果您计划将生产或任务关键型站点的环境升级到PHP 8.1，请事先进行彻底测试以确保它不会中断。

### WordPress 5.9-RC2 + WooCommerce 6.1.1

[WooCommerce](https://www.wbolt.com/woocommerce-tutorial.html)是WordPress的开源电子商务解决方案。与其他流行的电子商务平台不同，它是完全可定制和可扩展的。WooCommerce也是WordPress社区中最受欢迎的电子商务插件之一，为互联网上14%的电子商务网站提供支持。

![WooCommerce](https://static.wbolt.com/wp-content/uploads/2022/02/woocommerce-logo-1024x461-1.png "WooCommerce")

对于我们的下一个测试，我们在WordPress之上安装了WooCommerce。我们使用免费的[Storefront主题](https://www.wbolt.com/go?_=6d841cf242aHR0cHM6Ly93b29jb21tZXJjZS5jb20vc3RvcmVmcm9udC8%3D)和WooCommerce的默认数据来设置测试站点。测试的URL是单个产品页面。

![测试的WooCommerce页面](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress-5.9-RC2-WooCommerce-PHP-Benchmarks.jpg "测试的WooCommerce页面")

测试的WooCommerce页面

-   **网址测试：** `/product/hoodie/`
-   **主题：**Storefront 3.9.1
-   **注意：**单个产品页面包括带有Logo、标语、导航菜单、搜索小部件和购物车的标题。Body有一个产品，其中包含图像、描述、添加到购物车按钮、评论和多个侧边栏小工具。底部是包含三个产品的相关产品小部件。它还包括一个侧面拉出小部件，用于展示更多产品。
-   **图片来源：** [WordPress插件库](https://www.wbolt.com/go?_=90ec3387e7aHR0cHM6Ly93b3JkcHJlc3Mub3JnL3BsdWdpbnMvd29vY29tbWVyY2Uv)

![WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress5-9-RC2-WooCommerce6-1-1-PHP-Benchmarks_graphs.png "WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP基准测试")

WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP基准测试

#### 基准测试结果

-   WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP 7.2基准测试结果：130.73 req/sec
-   WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP 7.3基准测试结果：137.52 req/sec
-   WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP 7.4基准测试结果：141.48 req/sec
-   WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP 8.0基准测试结果：141.71 req/sec
-   **WordPress 5.9-RC2 + WooCommerce 6.1.1 PHP 8.1基准测试结果：147.67 req/sec 🏆**

PHP 8.1也是 WooCommerce的明显赢家。它以微弱优势击败了PHP 8.0。

### WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1

[Easy Digital Downloads](https://www.wbolt.com/easy-digital-downloads.html)是一个免费的WordPress电子商务插件。它由[Pippin](https://www.wbolt.com/go?_=e9b188b046aHR0cHM6Ly9waXBwaW5zcGx1Z2lucy5jb20v)（现在由Awesome Motive所有）创建，完全专注于帮助您销售数字产品（例如电子书、软件、视频游戏）。

![Easy Digital Downloads](https://static.wbolt.com/wp-content/uploads/2022/02/easy-digital-downloads-1024x166-1.png "Easy Digital Downloads")

对于Easy Digital Downloads，我们使用其免费的[Themed主题](https://www.wbolt.com/go?_=9fae3bc560aHR0cHM6Ly9lYXN5ZGlnaXRhbGRvd25sb2Fkcy5jb20vZG93bmxvYWRzL3RoZW1lZGQv)及其默认内容来设置测试站点。测试的页面是单个产品页面。

![测试的Easy Digital Downloads页面](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress-5-9-RC2-Easy-Digital-Downloads-2-11-4-1-PHP-Benchmarks.jpg "测试的Easy Digital Downloads页面")

测试的Easy Digital Downloads页面

-   **网址测试：** `/downloads/money-buys-happiness/`
-   **主题：**主题
-   **备注：** EDD的单品页面比较轻量，包含图片、描述、购买按钮，以及一些分类链接。页眉有徽标、标语和购物车，而页脚有基本的版权文本。
-   **图片来源：** [Easy Digital Downloads官网](https://www.wbolt.com/go?_=e38a5a2bbeaHR0cHM6Ly9lYXN5ZGlnaXRhbGRvd25sb2Fkcy5jb20vZ2V0dGluZy1zdGFydGVkLw%3D%3D)

![WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/WordPress5-9-RC2-Easy-Digital-Downloads-PHP-Benchmarks-Graphs.png "WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP基准测试")

WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP基准测试

#### 基准测试结果

-   WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP 7.2基准测试结果：352.87 req/sec
-   WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP 7.3基准测试结果：382.17 req/sec
-   WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP 7.4基准测试结果：392.07 req/sec
-   **WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP 8.0基准测试结果：407.59 req/sec** 🏆
-   WordPress 5.9-RC2 + Easy Digital Downloads 2.11.4.1 PHP 8.1基准测试结果：不支持🚫

在进行基准测试时，最新的EDD版本还不支持PHP 8.1。与前一年的基准测试一样，PHP 8.0在WordPress和Easy Digital Downloads方面胜过所有其他PHP版本。

> **注：**事实证明，PHP 8.0和8.1在 WordPress、WooCommerce和Easy Digital Downloads方面的速度更快。如果您使用WordPress运行您的任何网站，您应该计划尽快转换到PHP 8.0及更高版本。

### Drupal 9.3.3

Drupal是一个免费和开源的内容管理软件。它因其灵活和模块化的特性而广受欢迎。[据W3Techs称](https://www.wbolt.com/go?_=44c98d8c84aHR0cHM6Ly93M3RlY2hzLmNvbS90ZWNobm9sb2dpZXMvb3ZlcnZpZXcvY29udGVudF9tYW5hZ2VtZW50)，1.3%的网站使用Drupal，其中2.0%的网站使用内容管理系统。

![Drupal](https://static.wbolt.com/wp-content/uploads/2022/02/drupal-logo.png "Drupal")

我们使用其[Umami安装配置文件](https://www.wbolt.com/go?_=b5f96026bdaHR0cHM6Ly93d3cuZHJ1cGFsLm9yZy9kb2NzL3VtYW1pLWRydXBhbC1kZW1vbnN0cmF0aW9uLWluc3RhbGxhdGlvbi1wcm9maWxl)安装了Drupal ，这是一个演示Drupal核心功能的演示食品杂志网站。

![测试的Drupal页面](https://static.wbolt.com/wp-content/uploads/2022/02/Drupal-9-3-3-PHP-Benchmarks.jpg "测试的Drupal页面")

测试的Drupal页面

-   **网址测试：** `/en/articles/dairy-free-and-delicious-milk-chocolate/`
-   **主题：**鲜味食品杂志
-   **注意：**测试页面是一篇文章，包括许多功能，例如搜索小部件、语言转换器小部件、登录模块、面包屑、带有精选文章小部件的侧边栏小部件、食谱集合小部件、注册表单。
-   **图片来源：** [Drupal.org](https://www.wbolt.com/go?_=88440f1294aHR0cHM6Ly93d3cuZHJ1cGFsLm9yZy9wcm9qZWN0L2RydXBhbA%3D%3D)

![Drupal 9.3.3 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Drupal-9-3-3-PHP-Benchmarks_graphs.png "Drupal 9.3.3 PHP基准测试")

Drupal 9.3.3 PHP基准测试

#### 基准测试结果

-   Drupal 9.3.3 PHP 7.2基准测试结果：不支持🚫
-   Drupal 9.3.3 PHP 7.3基准测试结果：267.62 req/sec
-   Drupal 9.3.3 PHP 7.4基准测试结果：268.84 req/sec
-   Drupal 9.3.3 PHP 8.0基准测试结果：289.04 req/sec
-   **Drupal 9.3.3 PHP 8.1基准测试结果：302.27 req/sec 🏆**

自从我们上次对其进行基准测试以来，Drupal 9.xx已经取得了长足的进步。它不仅与较新的PHP版本兼容，而且性能也非常出色。我们很高兴看到它如何发展！

### Joomla！4.0.6

Joomla！是另一个免费和开源的内容管理系统。它于2005年首次发布，是当今使用的第二受欢迎的开源CMS。据W3Techs称，Joomla！被他们跟踪的所有网站中的[1.7%在使用](https://www.wbolt.com/go?_=be50d9f985aHR0cHM6Ly93M3RlY2hzLmNvbS90ZWNobm9sb2dpZXMvZGV0YWlscy9jbS1qb29tbGE%3D)。

![Joomla！](https://static.wbolt.com/wp-content/uploads/2022/02/Joomla-logo-light-background.png "Joomla！")

对于Joomla！基准测试，我们使用了所有Joomla附带的免费Cassiopeia模板！4.x发行版。

![测试的Joomla页面](https://static.wbolt.com/wp-content/uploads/2022/02/Joomla-4-0-6-PHP-Benchmarks.jpg "测试的Joomla页面")

测试的Joomla页面

-   **网址测试：**主页
-   **主题：**Cassiopeia
-   **注意：**   Joomla！安装了“默认英语 (GB) 示例数据”，它为网站添加了基本内容。主页包含几段内容、一个搜索小部件以及侧边栏上的其他基本小部件，例如登录表单、热门标签和最新文章。
-   **图片来源：** [Joomla.org](https://www.wbolt.com/go?_=9e6a64d472aHR0cHM6Ly93d3cuam9vbWxhLm9yZy8%3D)

![Joomla！4.0.6 PHP基准](https://static.wbolt.com/wp-content/uploads/2022/02/Joomla-4-0-6-PHP-Benchmarks_graphs.png "Joomla！4.0.6 PHP基准")

Joomla！4.0.6 PHP基准

#### 基准测试结果

-   Joomla！4.0.6 PHP 7.2基准测试结果：38.18 req/sec
-   Joomla！4.0.6 PHP 7.3基准测试结果：39.41 req/sec
-   Joomla！4.0.6 PHP 7.4基准测试结果：39.57 req/sec
-   Joomla！4.0.6 PHP 8.0基准测试结果：39.84 req/sec
-   **Joomla！4.0.6 PHP 8.1基准测试结果：41.97 req/sec** 🏆

经过[一些打嗝](https://www.wbolt.com/go?_=3eb4f4be0aaHR0cHM6Ly9mb3J1bS5qb29tbGEub3JnL3ZpZXd0b3BpYy5waHA%2FZj04MDMmYW1wO3Q9OTc3MTI4)，Joomla！重回正轨。结果在这里遵循预期的模式——PHP 8.1是无可争议的冠军，紧随其后的是PHP 8.0，然后是其余的。

### Grav 1.7.29

[Grav](https://www.wbolt.com/go?_=f2efddd801aHR0cHM6Ly9nZXRncmF2Lm9yZy8%3D)是一个开源的平面文件CMS。它不需要数据库即可操作，但功能丰富。Grav从文本文件中查询内容。这使得它轻巧且易于安装在几乎任何服务器上。

![Grav](https://static.wbolt.com/wp-content/uploads/2022/02/grav-cms.png "Grav")

执行此测试时，Grav需要PHP 7.3及更高版本才能工作。我们使用了为测试提供默认登录页面的Base Grav包。

![测试的Grav页面](https://static.wbolt.com/wp-content/uploads/2022/02/Grav-1-7-29-PHP-Benchmarks.jpg "测试的Grav页面")

测试的Grav页面

-   **网址测试：**主页
-   **主题：**Quark
-   **注意：**测试页面是一个简单的页面，包含很多内容，包括Header、Logo、Navigation Menu 和 Footer。[Grav Core Caching](https://www.wbolt.com/go?_=afcfd3007baHR0cHM6Ly9sZWFybi5nZXRncmF2Lm9yZy8xNy9hZHZhbmNlZC9wZXJmb3JtYW5jZS1hbmQtY2FjaGluZw%3D%3D)已被禁用以测试PHP的原始性能。
-   **图片来源：** [Grav官网](https://www.wbolt.com/go?_=f2efddd801aHR0cHM6Ly9nZXRncmF2Lm9yZy8%3D)

![针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试插图21](https://static.wbolt.com/wp-content/uploads/2022/02/Grav-1-7-29-PHP-Benchmarks_graphs.png "针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试插图21")

Grav 1.7.29 PHP基准测试

#### 基准测试结果

-   Grav 1.7.29 PHP 7.2基准测试结果：不支持🚫
-   Grav 1.7.29 PHP 7.3基准测试结果：1800.07 req/sec
-   Grav 1.7.29 PHP 7.4基准测试结果：1848.02 req/sec
-   Grav 1.7.29 PHP 8.0基准测试结果：1931.72 req/sec
-   **Grav 1.7.29 PHP 8.1基准测试结果：2137.43 req/sec 🏆**

PHP 8.1是Grav无可争议的赢家，紧随其后的是PHP 8.0和其他版本。

作为一个相对较新的CMS，它的市场份额比WordPress小。因此，它可以很快放弃对旧PHP版本的支持。这是现代CMS最显着的优势之一。

### OctoberCMS 1.3.1

[OctoberCMS](https://www.wbolt.com/go?_=830e25b69aaHR0cHM6Ly9vY3RvYmVyY21zLmNvbS8%3D)是一个基于Laravel PHP框架的CMS。最初是免费和开源的，OctoberCMS在2021年[改变其许可模式](https://www.wbolt.com/go?_=a21f27c1b3aHR0cHM6Ly9vY3RvYmVyY21zLmNvbS9ibG9nL3Bvc3Qvb2N0b2Jlci1jbXMtbW92ZXMtYmVjb21lLXBhaWQtcGxhdGZvcm0%3D)后现在是一个付费平台。使用Laravel的力量制作动态网站在开发人员中很受欢迎。[根据W3Techs](https://www.wbolt.com/go?_=e48ddf9da0aHR0cHM6Ly93M3RlY2hzLmNvbS90ZWNobm9sb2dpZXMvZGV0YWlscy9jbS1vY3RvYmVyY21z)，OctoberCMS仅支持**0.1%**的网站。

![OctoberCMS](https://static.wbolt.com/wp-content/uploads/2022/02/october-cms.png "OctoberCMS")

我们为测试站点使用了OctoberCMS的默认演示主题。这是一个布局明确的响应式主题。

![测试的OctoberCMS页面](https://static.wbolt.com/wp-content/uploads/2022/02/OctoberCMS-1-3-1-PHP-Benchmarks.jpg "测试的OctoberCMS页面")

测试的OctoberCMS页面

-   **网址测试：**主页
-   **主题：**演示主题
-   **注意：**测试页面包含许多元素，包括徽标、导航菜单、文本部分、代码嵌入等。我们遵循其有关[提高性能](https://www.wbolt.com/go?_=4ad0517066aHR0cHM6Ly9kb2NzLm9jdG9iZXJjbXMuY29tLzIueC9zZXR1cC9kZXBsb3ltZW50Lmh0bWwjaW1wcm92aW5nLXBlcmZvcm1hbmNl)的文档，以确保其设置为尽可能高效地运行。在撰写本文时，OctoberCMS[需要PHP 7.2+](https://www.wbolt.com/go?_=30ce9fb5b2aHR0cHM6Ly9vY3RvYmVyY21zLmNvbS9kb2NzL3NldHVwL2luc3RhbGxhdGlvbg%3D%3D)才能运行，并且还不支持PHP 8.1。
-   **图片来源：** [OctoberCMS官网](https://www.wbolt.com/go?_=7a5843da2baHR0cDovL29jdG9iZXJjbXMuY29tL2Rvd25sb2Fk)

![OctoberCMS 1.3.1 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/OctoberCMS-1-3-1-PHP-Benchmarks_graphs.png "OctoberCMS 1.3.1 PHP基准测试")

OctoberCMS 1.3.1 PHP基准测试

#### 基准测试结果

-   OctoberCMS 1.3.1 PHP 7.2基准测试结果：417.13 req/sec
-   OctoberCMS 1.3.1 PHP 7.3基准测试结果：458.63 req/sec
-   OctoberCMS 1.3.1 PHP 7.4基准测试结果：532.65 req/sec
-   **OctoberCMS 1.3.1 PHP 8.0基准测试结果：640.08 req/sec 🏆**
-   OctoberCMS 1.3.1 PHP 8.1基准测试结果：不支持🚫

PHP 8.0是明显的赢家。OctoberCMS在PHP 8.0上每秒处理的请求比在PHP 7.4上多**20.16% 。**我们渴望看到它的下一个主要更新在PHP 8.1上的表现如何。

### Laravel 8.80.0

[Laravel](https://www.wbolt.com/go?_=bcff8db389aHR0cHM6Ly9sYXJhdmVsLmNvbS8%3D)是目前最流行的[PHP框架](https://www.wbolt.com/something-about-php-frameworks.html)。它由Taylor Otwell创建，于2011年6月发布。您可以使用Laravel开发几乎任何Web应用程序，包括CMS、电子商务网站、应用程序等等。

![我们使用默认的Laravel登陆页面来对Laravel进行基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/laravel-logo.png "我们使用默认的Laravel登陆页面来对Laravel进行基准测试")

我们使用默认的Laravel登陆页面来对Laravel进行基准测试

正如Laravel创始人[Taylor Otwell之前指出的](https://www.wbolt.com/go?_=d2c2cdef67aHR0cHM6Ly9tZWRpdW0uY29tL0B0YXlsb3JvdHdlbGwvYmVuY2htYXJraW5nLWxhcmF2ZWwtc3ltZm9ueS16ZW5kLTJjMDFjMmIyNzBmOA%3D%3D)，你不应该使用这些基准测试结果来比较Laravel和其他PHP框架。这里的目标是查看Laravel在一切都保持不变的情况下在不同PHP版本上的表现。

![测试的Laravel页面](https://static.wbolt.com/wp-content/uploads/2022/02/Laravel-8-80-0-PHP-Benchmarks-1.jpg "测试的Laravel页面")

测试的Laravel页面

-   **网址测试：**主页
-   **主题：**纯HTML
-   **注意：**测试页面有许多基本的HTML元素。虽然它不是一个成熟的Web应用程序，但其目标是对PHP而非Laravel进行基准测试。
-   **图片来源：** [Laravel官方仓库](https://www.wbolt.com/go?_=bcff8db389aHR0cHM6Ly9sYXJhdmVsLmNvbS8%3D)

![Laravel 8.80.0 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Laravel-8-80-0-PHP-Benchmarks_graphs.png "Laravel 8.80.0 PHP基准测试")

Laravel 8.80.0 PHP基准测试

#### 基准测试结果

-   Laravel 8.80.0 PHP 7.2基准测试结果：不支持🚫
-   Laravel 8.80.0 PHP 7.3基准测试结果：2278.86 req/sec
-   Laravel 8.80.0 PHP 7.4基准测试结果：2303.23 req/sec
-   **Laravel 8.80.0 PHP 8.0基准测试结果：2376.40 req/sec 🏆**
-   Laravel 8.80.0 PHP 8.1基准测试结果：2002.94 req/sec

很高兴看到Laravel支持最新的PHP版本。PHP 8.0是Laravel无可争议的冠军，而PHP 8.1排在最后。这里有一些改进的空间。也许刚刚发布的[Laravel 9](https://www.wbolt.com/laravel-9.html)可能会产生有趣的结果，但这是我们下一个基准测试的结果。

### Symfony 5.4.2

[Symfony](https://www.wbolt.com/go?_=52adc87558aHR0cHM6Ly9zeW1mb255LmNvbS8%3D)是一组可重用的PHP组件和PHP框架，用于构建Web应用程序、API、微服务和Web服务。它是一个免费的开源软件，于2005年10月22日发布。

![Symfony](https://static.wbolt.com/wp-content/uploads/2022/02/symfony.png "Symfony")

虽然Symfony已经发布了6.x版本，但它只支持PHP 8.0及更高版本。因此，我们决定更倾向于使用其最新的5.4.2版本来对PHP进行基准测试。

[您可以使用演示应用程序](https://www.wbolt.com/go?_=07690e4debaHR0cHM6Ly9naXRodWIuY29tL3N5bWZvbnkvZGVtbw%3D%3D)安装Symfony 。这是一个参考CMS应用程序，演示了如何最好地使用Symfony及其各种功能。我们使用这个演示应用程序的主页来对Symfony进行基准测试。

![测试的Symfony页面](https://static.wbolt.com/wp-content/uploads/2022/02/Symfony-5-4-2-PHP-Benchmarks.jpg "测试的Symfony页面")

测试的Symfony页面

-   **网址测试：**主页
-   **主题：** Symfony演示
-   **注意：**测试页面包含带有徽标的标题、主页链接、搜索小部件、语言转换器小部件和带有许多帖子的博客。还有一个侧边栏，带有小文本框、“显示代码”和“博客帖子RSS”等小部件。
-   **图片来源：** [Symfony官方仓库](https://www.wbolt.com/go?_=07690e4debaHR0cHM6Ly9naXRodWIuY29tL3N5bWZvbnkvZGVtbw%3D%3D)

![Symfony 5.4.2 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Symfony-5-4-2-PHP-Benchmarks_graphs.png "Symfony 5.4.2 PHP基准测试")

Symfony 5.4.2 PHP基准测试

#### 基准测试结果

-   Symfony 5.4.2 PHP 7.2基准测试结果：不支持🚫
-   Symfony 5.4.2 PHP 7.3基准测试结果：416.18 req/sec
-   Symfony 5.4.2 PHP 7.4基准测试结果：434.95 req/sec
-   Symfony 5.4.2 PHP 8.0基准测试结果：443.79 req.sec
-   **Symfony 5.4.2 PHP 8.1基准测试结果：524.78 req/sec 🏆**

使用Symfony，PHP 8.1与其他版本之间存在巨大差异。例如，Symfony在PHP 8.1上的运行速度比PHP 7.4快**20.65% 。**

### CodeIgniter 4.1.8

[CodeIgniter](https://www.wbolt.com/go?_=586b028b7caHR0cHM6Ly93d3cuY29kZWlnbml0ZXIuY29tLw%3D%3D)是一个占用空间很小的PHP框架。例如，它的最新版本是1.2 MB下载。它由[EllisLab](https://www.wbolt.com/go?_=173bdf60d3aHR0cHM6Ly93d3cuZWxsaXNsYWIuY29tLw%3D%3D)创建并由[不列颠哥伦比亚理工学院](https://www.wbolt.com/go?_=4d61f6b1c2aHR0cHM6Ly93d3cuYmNpdC5jYS8%3D)培育。尽管CodeIgniter很大，您仍然可以使用它来开发功能齐全的Web应用程序。

![CodeIgniter](https://static.wbolt.com/wp-content/uploads/2022/02/codeigniter-logo.png "CodeIgniter")

为了对CodeIgniter进行基准测试，我们使用他们的[官方教程](https://www.wbolt.com/go?_=014a883b50aHR0cHM6Ly9jb2RlaWduaXRlcjQuZ2l0aHViLmlvL3VzZXJndWlkZS90dXRvcmlhbC9pbmRleC5odG1s)设置了一个演示应用程序。它使用基本的HTML主题并输出许多“新闻”项目。

![测试的CodeIgniter页面](https://static.wbolt.com/wp-content/uploads/2022/02/CodeIgniter-4-1-8-PHP-Benchmarks.jpg "测试的CodeIgniter页面")

测试的CodeIgniter页面

-   **网址测试：** `/news/`
-   **主题：**纯HTML
-   **注释：**测试页面包含一个包含标题、内容和指向主要内容的链接的新闻项目列表。该数据库包括1个表“news”，其中包含1000行新闻项目，列 -> id、title、slug、body。该页面连接到数据库并显示表中的所有帖子。CodeIgniter应用程序包含1个路由和1个控制器来显示此内容。
-   **图片来源：** [CodeIgniter.com官网](https://www.wbolt.com/go?_=586b028b7caHR0cHM6Ly93d3cuY29kZWlnbml0ZXIuY29tLw%3D%3D)

![针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试插图33](https://static.wbolt.com/wp-content/uploads/2022/02/CodeIgniter-4-1-8-PHP-Benchmarks_graphs.png "针对PHP 7.2、7.3、7.4、8.0和8.1多个版本之间的权威基准测试插图33")

CodeIgniter 4.1.8 PHP基准测试

#### 基准测试结果

-   CodeIgniter 4.0.4 PHP 7.2基准测试结果：不支持🚫
-   CodeIgniter 4.0.4 PHP 7.3基准测试结果：不支持🚫
-   CodeIgniter 4.0.4 PHP 7.4基准测试结果：1907.33 req/sec
-   CodeIgniter 4.0.4 PHP 8.0基准测试结果：1770.33 req/sec
-   **CodeIgniter 4.0.4 PHP 8.1基准测试结果：1920.51 req/sec 🏆**

PHP 8.1是CodeIgniter中最快的，每秒执行的请求比PHP 8.0多**8.48% 。**然而，令人惊讶的是，PHP 7.4的性能比PHP 8.0好得多——它几乎与PHP 8.1相当。

### CakePHP 4.3.4

[CakePHP](https://www.wbolt.com/go?_=c1a069e48daHR0cHM6Ly9jYWtlcGhwLm9yZy8%3D)是一个用于开发PHP应用程序的开源Web框架。它承诺让构建Web应用程序更简单、更快、代码更少。

![CakePHP](https://static.wbolt.com/wp-content/uploads/2022/02/CakePHP-logo.jpg "CakePHP")

为了对CakePHP进行基准测试，我们使用了它的默认登录页面。在进行基准测试之前，我们将其连接到数据库。

![测试的CakePHP页面](https://static.wbolt.com/wp-content/uploads/2022/02/CakePHP-4-3-4-PHP-Benchmarks.jpg "测试的CakePHP页面")

测试的CakePHP页面

-   **网址测试：**主页
-   **主题：**纯HTML
-   **注意：**测试页面是一个简单的HTML登陆页面，带有一些样式。它提供了有关当前CakePHP安装的简要信息。
-   **图片来源：** [CakePHP官方仓库](https://www.wbolt.com/go?_=545dc0f2ceaHR0cHM6Ly9naXRodWIuY29tL2Nha2VwaHAvY2FrZXBocC9yZWxlYXNlcw%3D%3D)

![CakePHP 4.3.4 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/CakePHP-4-3-4-PHP-Benchmarks_graphs.png "CakePHP 4.3.4 PHP基准测试")

CakePHP 4.3.4 PHP基准测试

#### 基准测试结果

-   CakePHP 4.2.2 PHP 7.2基准测试结果：743.46 req/sec
-   CakePHP 4.2.2 PHP 7.3基准测试结果：874.69.28 req/sec
-   CakePHP 4.2.2 PHP 7.4基准测试结果：954.30 req/sec
-   **CakePHP 4.2.2 PHP 8.0基准测试结果：973.02 req/sec 🏆**
-   CakePHP 4.2.2 PHP 8.1基准测试结果：918.21 req/sec

令人惊讶的是，PHP 8.0与CakePHP取得了成功。然而，所有的基准测试结果都太接近了，不能称之为绝对的赢家。PHP 8.1仅比PHP 8.0慢**5.6% 。**CakePHP 4.3.x的未来更新可能会解决这个差异。

### Craft CMS 3.7.30.1

[Craft CMS](https://www.wbolt.com/go?_=fec4a8a4bdaHR0cHM6Ly9jcmFmdGNtcy5jb20v)是一个专注于用户友好性的开源内容管理系统。它的后端是完全可定制的。Craft CMS使用内置工具为不同的内容类型设计自定义字段布局，还使使用[自定义内容类型](https://www.wbolt.com/wordpress-custom-post-types.html)变得超级简单。

如果您打算创建自定义电子商务商店，请查看[Craft Commerce](https://www.wbolt.com/go?_=1478ee5a79aHR0cHM6Ly9jcmFmdGNtcy5jb20vY29tbWVyY2U%3D)。对于Craft CMS的本地开发环境，还有[Craft Nitro](https://www.wbolt.com/go?_=747a30a884aHR0cHM6Ly9jcmFmdGNtcy5jb20vYmxvZy9jcmFmdC1uaXRybw%3D%3D)。

![Craft CMS](https://static.wbolt.com/wp-content/uploads/2022/02/craft-cms-logo.png "Craft CMS")

对于Craft CMS基准测试，我们使用了其默认的管理员登录页面。这是一个简单的登录页面，其中包含用于访问站点后端的登录表单。

![测试的Craft CMS页面](https://static.wbolt.com/wp-content/uploads/2022/02/Craft-3-7-30-1-PHP-Benchmarks.jpg "测试的Craft CMS页面")

测试的Craft CMS页面

-   **网址测试：** `/admin/login/`
-   **主题：**默认
-   **注意：**测试页面是一个简单的登录页面，带有一个表单。
-   **图片来源：** [Craft CMS官方仓库](https://www.wbolt.com/go?_=4b6ea07e88aHR0cHM6Ly9naXRodWIuY29tL2NyYWZ0Y21zL2RlbW8%3D)

![Craft CMS 3.7.30.1 PHP基准](https://static.wbolt.com/wp-content/uploads/2022/02/CraftCMS-3-7-30-1-PHP-Benchmarks_graphs.png "Craft CMS 3.7.30.1 PHP基准")

Craft CMS 3.7.30.1 PHP基准

#### 基准测试结果

-   Craft CMS 3.5.17.1 PHP 7.2基准测试结果：75.32 req/sec
-   Craft CMS 3.5.17.1 PHP 7.3基准测试结果：74.69 req/sec
-   Craft CMS 3.5.17.1 PHP 7.4基准测试结果：81.68 req/sec
-   Craft CMS 3.5.17.1 PHP 8.0基准测试结果：417.21 req/sec
-   **Craft CMS 3.5.17.1 PHP 8.1基准测试结果：443.18 req/sec 🏆**

PHP 8.1凭借Craft CMS位居榜首。与我们之前的基准测试不同，Craft CMS现在支持PHP 8.0和PHP 8.1 — 太棒了！

### Kirby 3.6.1.1

[Kirby](https://www.wbolt.com/go?_=8a555c9589aHR0cHM6Ly9nZXRraXJieS5jb20v)是一个专注于内容创建和发布的平面文件CMS。虽然它的源代码是公开的，但它不能在公共服务器上免费使用。您可以使用Kirby自定义包含表单、文章、画廊、电子表格等的编辑界面。

![Kirby](https://static.wbolt.com/wp-content/uploads/2022/02/kirby-logo.png "Kirby")

您可以使用Starterkit安装Kirby，它设置了一个功能齐全的演示站点。我们使用它的About Us页面进行此基准测试。

![测试的Kirby页面](https://static.wbolt.com/wp-content/uploads/2022/02/Kirby-3-6-1-1-PHP-Benchmarks.jpg "测试的Kirby页面")

测试的Kirby页面

-   **网址测试：** `/about/`
-   **主题：**初学者工具包
-   **注意：**测试页面是一个关于我们的页面，其中包含特色图片、文本、小部件、页眉、导航菜单、社交媒体图标和页脚。
-   **图片来源：** [Kirby官网](https://www.wbolt.com/go?_=574d563041aHR0cHM6Ly9nZXRraXJieS5jb20vdHJ5)

![Kirby 3.6.1.1 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Kirby-3-6-1-1-PHP-Benchmarks_graphs.png "Kirby 3.6.1.1 PHP基准测试")

Kirby 3.6.1.1 PHP基准测试

#### 基准测试结果

-   Kirby 3.6.1.1 PHP 7.2基准测试结果：不支持🚫
-   Kirby 3.6.1.1 PHP 7.3基准测试结果：不支持🚫
-   Kirby 3.6.1.1 PHP 7.4基准测试结果：3326.72 req/sec
-   Kirby 3.6.1.1 PHP 8.0基准测试结果：3514.96 req/sec 🏆
-   **Kirby 3.6.1.1 PHP 8.1基准测试结果：3922.77 req/sec 🏆**

PHP 8.1在Kirby的基准测试中脱颖而出。还值得一提的是，Kirby在我们测试的所有PHP平台上每秒处理的请求最多。即使这是一个苹果和橘子的比较，这仍然是值得辜负的。它的主要缺点是它不是免费使用的。

### Flarum 1.2.0

[Flarum](https://www.wbolt.com/go?_=b023972849aHR0cHM6Ly9mbGFydW0ub3JnLw%3D%3D)是一个免费的开源论坛软件，用于在线讨论。

![Flarum](https://static.wbolt.com/wp-content/uploads/2022/02/Flarum-Logo.jpg "Flarum")

您可以使用演示站点安装Flarum。我们还添加了三个带有几段文字的线程。

![测试的Flarum页面](https://static.wbolt.com/wp-content/uploads/2022/02/Flarum-1-2-0-PHP-Benchmarks-.jpg "测试的Flarum页面")

测试的Flarum页面

-   **网址测试：**主页
-   **主题：**默认主题
-   **注意：**测试页面是论坛主页，带有标题、徽标、搜索小部件、特色文本块、导航菜单、通知图标、侧边菜单、讨论线程列表、其他小部件和页脚。最新的 Flarum 版本还不支持 PHP 8.1，所以我们无法对其进行基准测试。
-   **图片来源：** [Flarum官网](https://www.wbolt.com/go?_=b023972849aHR0cHM6Ly9mbGFydW0ub3JnLw%3D%3D)

![Flarum 1.2.0 PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Flarum-1-2-0-PHP-Benchmarks_graphs.png "Flarum 1.2.0 PHP基准测试")

Flarum 1.2.0 PHP基准测试

#### 基准测试结果

-   Flarum 1.2.0 PHP 7.2基准测试结果：不支持🚫
-   Flarum 1.2.0 PHP 7.3基准测试结果：120.21 req/sec
-   **Flarum 1.2.0 PHP 7.4基准测试结果：122.06 req/sec 🏆**
-   Flarum 1.2.0 PHP 8.0基准测试结果：119.67 req/sec
-   Flarum 1.2.0 PHP 8.1基准测试结果：不支持🚫

Flarum是我们PHP基准测试的新成员。由于它是一种流行的PHP论坛软件，我们很高兴能对其进行测试，看看它的性能如何。虽然PHP 7.4在Flarum上表现最好，但在我们基准测试的所有其他PHP版本上几乎相同。

### 更新到PHP 8.1

PHP 8.1引入了许多令人兴奋的特性。其中一些是激进的、破坏性的更改，与以前的PHP版本（主要是<PHP 8.0）不兼容。

如果您网站的所有功能都可以正常运行，那么您没有理由不更新到PHP 8.1。如果上述结果还不能让您信服，我们不确定还有什么可以说服您！

### PHP基准测试结果的要点

![编译的PHP基准测试](https://static.wbolt.com/wp-content/uploads/2022/02/Kinsta-PHP-Benchmarks-2022.png "编译的PHP基准测试")

编译的PHP基准测试

从上面的基准测试结果中，您可以看到PHP 8.1在大多数PHP平台和配置中处于领先地位，紧随其后的是PHP 8.0。

以下是我们从2022年PHP基准测试结果中得出的扩展结论：

-   **对于WordPress，PHP 8.1是所有基准测试中最快的**（Stock WordPress 5.6和WooCommerce）。Easy Digital Downloads尚不支持PHP 8.1，但我们可以期待类似的性能改进。
-   如果您使用的是WordPress，并且您的所有主题和插件都与PHP 8.1兼容，那么您没有理由不将您的PHP版本更新到PHP 8.1。您会欣赏它带来的性能优势。
-   PHP 8.0是Laravel框架中最快的，这是用于构建Web应用程序的最流行的PHP框架。Laravel 9在基准测试时尚未发布。我们将在以下基准测试中使用它。
-   如果您使用的任何插件或主题尚不兼容PHP 8.0，更不用说PHP 8.1，我们建议您与他们的开发人员联系并告知他们。
-   随着对PHP 7.4的支持即将于2022年底结束，您应该计划尽快将您的网站迁移到PHP 8.0及更高版本。
-   PHP 8.0预示着PHP的新曙光，就像PHP 7.0在PHP 5.6盛行时一样。PHP 8.1将火炬向前推进了一大步。我们预计以后的PHP 8.x版本会进一步优化性能和安全性。
-   我们没有在启用JIT的情况下测试PHP 8.x。虽然PHP的新JIT编译器不会为WordPress等实际应用程序带来任何显着的性能优势，但看看它在实际使用中的表现会很有趣。
-   如果他们不跟上较新的PHP版本，请重新考虑您的托管服务提供商。
-   如前所述，在将您的网络服务器环境更新到PHP 8.0和PHP 8.1之前，请彻底测试您的站点。
-   除了升级到最新的PHP版本外，WordPress用户还可以使用其他Web性能增强技术进一步加速他们的网站。我们已将它们全部编译在我们关于[如何加速您的WordPress网站](https://www.wbolt.com/speed-up-wordpress.html)的终极指南中。

这是对所有各种PHP平台的基准测试。我们对PHP 8.1感到非常兴奋。我们希望你也是！