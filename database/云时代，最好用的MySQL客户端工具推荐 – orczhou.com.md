数据库图形客户端（GUI）工具，可以大大帮助开发者提升SQL编写与开发的效率。在云时代，企业越来越多的开始采用RDS，同时也还有部分本地IDC自建数据库，而在云端也会选择/尝试多个不同云厂商。“工欲善其事，必先利其器”，在这样的背景下，看看有哪些工具产品可供选择吧。

### **整体综述**

本文完整对比了12种MySQL图形客户端（GUI）工具，从产品体验、功能完整度、云适配、计费模式、OS兼容性等多个角度进行评估与分析，给出推荐。下面产品推荐与整体得分图，读者可根据自己的实际情况选择：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-11-1024x855.png)

根据从OS兼容性、收费模式、产品体验、云适配、功能完整度等角度，这里推荐的MySQL 图形化客户端工具（GUIT）如下：

-   Navicat
-   NineData
-   HeidiSQL
-   DBVisualizer
-   MySQL Workbench
-   Dbeaver
-   阿里云DMS
-   dbForge
-   BeeKeeper Studio
-   phpMyAdmin
-   SQLyog
-   Toad Edge

接下来，我们一起看看所有这些工具的特点吧。

### **NineData**

官网地址：[https://www.ninedata.cloud/NineData](https://www.ninedata.cloud/NineData)

是一款非常有特色的数据库SQL开发产品，对MySQL常用功能支持非常完整，包括智能的SQL补全、SQL执行历史、结果集编辑、数据对比、结构对比、数据迁移与复制等。它采用SaaS架构模式，用户不仅可以免费使用，而且无需下载安装，上手比较简单。NineData产品更新迭代比较敏捷，对于开发者的新需求响应比较迅速。另外，该产品在多云适配上是其重要的强项，支持多种连接和访问云数据库的方式，对阿里云、腾讯云、华为云、AWS等都有比较好的支持。另外，也适配国内比较流行的PolarDB、GaussDB、TDSQL等数据库。

对于新用户NineData还会赠送两个示例数据库，供用户使用。另外，NineData还提供了企业级SQL开发能力，支持多用户管理、数据库访问权限控制、变更流程、SQL规范、SQL与操作审计等内容，可以较好的解决企业内多人协作访问数据库的问题。

NineData综合评价如下：

-   产品体验：★★★★
-   云适配度：★★★★
-   功能完整：★★★★
-   是否收费：免费
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-12-1024x620.png)

### **Navicat**

官网地址：[https://navicat.com/en/](https://navicat.com/en/)

Navicat是一款来自香港的产品，约2000年左右发布，是一个老牌的商业化、闭源数据库管理软件，支持主流的Windows、Mac OS X以及Linux，最近两年开始支持订阅模式，个人使用价格约35美元/月，企业版约69美元/月（参考），国内购买为273元/月，有一定的价格门槛，但其使用体验也还不错，功能也比较完整，包括比较强大的SQL补全、导入导出、结果集编辑、E-R模型、数据对比、结构对比、数据迁移等，但有部分功能仅企业版才具备。Navicat的代码块功能做得比较强，可以非常方便自定义一些自己常用的SQL模板。

Navicat综合评价如下：

-   产品体验：★★★★★
-   云适配度：★★★
-   功能完整：★★★★★
-   是否收费：商业收费
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-14-1024x581.png)

### **MySQL Workbench**

官网地址：[https://www.mysql.com/products/workbench/Workbench](https://www.mysql.com/products/workbench/Workbench)

Workbench是最老牌的数据库管理工具了。最早由奥地利程序员Michael G. Zinner独立开发，之后Zinner于2003年加入了MySQL AB公司，并于2005年发布了最早的Workbench 5.0版本；2013年发布了，6.0版本；2018年，发布了8.0版本。整体上，该产品依旧随着MySQL的版本而持续更新，但是，更新节奏较慢，界面也非常“老”，并没有受到Oracle/MySQL的重视。

Workbench支持主流的Windows、Mac OS X以及Linux，并且开放源代码。但因为界面架构比较长时间没有更新，所以使用的交互体验一般。因为是MySQL官方工具，功能支持是比较完整的，包括SQL补全、SQL历史、导入导出、结果集编辑、E-R模型、数据对比、结构对比、数据迁移等功能都具备。另外，也提供商业化的企业版，支持部分MySQL企业版的功能。

MySQL Workbench综合评价如下：

-   产品体验：★★★
-   云适配度：★
-   功能完整：★★★★★
-   是否收费：免费
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-13-1024x765.png)

**DBeaver**

官网地址：[https://dbeaver.io/](https://dbeaver.io/)

DBeaver 是一个基于 Java 开发数据库管理工具，提供开源免费的版本。因为是基于Java的，所以也能够支持Windows、Linux、macOS 等操作系统，其支持的数据库类型也比较多。同时也是因为基于Java，其在访问的不同的数据库版本时，有时候需要在线做一些驱动更新，需要访问GitHub的一些资源，而因为一些原因，这类更新经常失败，使其使用体验有一定打折。DBeaver也提供了基础的SQL补全、导入导出、结果集编辑等功能，但也有部分功能仅限于企业版（Pro版本）才提供，另外，软件似乎因为比较大的缘故，所以运行起来有点慢。

-   DBeaver综合评价如下：
-   产品体验：★★★
-   云适配度：★★
-   功能完整：★★★★
-   是否收费：开源免费（功能限制）+ 商业收费（完整版）

环境兼容：Windows / Linux / macOS

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-15-1024x540.png)

### **phpMyAdmin**

官网地址：[https://www.phpmyadmin.net/](https://www.phpmyadmin.net/)

这是另一个老牌的开源免费MySQL访问工具了，在云时代之前，开发者经常需要自己搭建自己完整的开发环境（例如“LAMP”）时，该软件还比较流行。从名字可以看出来，这是一个PHP的Web-Based的MySQL访问工具，所以需要使用并不是很方便，需要构建自己的Web服务器和PHP运行环境。一般来说，现在的开发者也并不会这么去做。

另外，phpMyAdmin一直没有商业化，主要靠捐赠和赞助的方式在运转（参考，有意思的是Navicat也在赞助列表，而且是唯一的白金赞助商），整体上，phpMyAdmin其迭代速度非常慢，功能支持也很有限，但是如果是简单、基础的使用，是没有问题的。但，如果是日常开发使用，并不是很推荐。

phpMyAdmin综合评价如下：

-   产品体验：★★
-   云适配度：★
-   功能完整：★★★
-   是否收费：开源免费
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-16-1024x757.png)

### **dbForge**

官网地址：[https://www.devart.com/dbforge/mysql/](https://www.devart.com/dbforge/mysql/)

dbForge是devart的核心产品，最早主要是支持SQL Server数据库，最近几年也发布了对MySQL数据库的支持，也是一个商业化收费软件，产品可以下载试用一段时间。根据使用经验来看，体验还是非常不错的，功能也非常完整。但是，仅支持Windows版本，标准版费用为199美元/年，起步价也并不便宜。

dbForge综合评价如下：

-   产品体验：★★★★★
-   云适配度：★★
-   功能完整：★★★★★
-   是否收费：商业收费
-   环境兼容：Windows
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-17-1024x615.png)

### **SQLYOG**

官网地址：[https://webyog.com/product/sqlyog/](https://webyog.com/product/sqlyog/%C2%A0) 

SQLyog更多的是专注于数据库的管理，包括性能、监控、优化等方面，也提供基础SQL编辑功能，所以在早期，其在DBA群体中比较受欢迎，但是在整体的开发者中，使用比率并不高。虽然，提供开源的社区版本，但是当前，公司主要在推广其商业版本。

另外，在云时代对于监控与实例管理方面的诉求在降低，在SQL开发与云适配上需求更强。从这个角度来看，并不是很推荐这个这个产品。此外，该软件仅支持Windows系统。最近几年这个产品发展比较缓慢，而且SQL开发功能也不再是主推的功能，所以也并不是特别推荐。

SQLYOG综合评价如下：

-   产品体验：★★★
-   云适配度：★
-   功能完整：★★★
-   是否收费：开源免费（功能限制）+ 商业收费（完整版）
-   环境兼容：Windows
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-18-1024x541.png)

**HeidiSQL**

官网地址：[https://www.heidisql.com/](https://www.heidisql.com/) 

HeidiSQL也是一个发展了很长时间的MySQL客户端，使用Delphi构建，所以整体上，有非常好的Windows使用体验。但是不能支持macOS或者Linux。因为发展时间比较长，功能也比较完整。新增了部分对于云产品的适配，例如，如果类型选择的是AWS RDS，那么在kill连接的时候会使用特定的存储过程进行kill。

HeidiSQL综合评价如下：

-   产品体验：★★★★★
-   云适配度：★★★
-   功能完整：★★★★
-   是否收费：开源免费
-   环境兼容：Windows
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-19-1024x544.png)

### **阿里云DMS**

官网地址：[https://www.aliyun.com/product/dms](https://www.aliyun.com/product/dms)

因为阿里云在国内市占率非常高，所以，阿里云DMS也是一个使用比较广，但是也因为其为阿里云的产品，所以其作为MySQL管理工具并不是非常有名。DMS比较完整的支持MySQL日常SQL开发相关的工作，其功能矩阵也比较完整，可以完成日常的开发工作。

DMS对于阿里云数据库的适配自然是非常好，使用也比较便利。但，其对于其他云数据库（诸如腾讯、华为、AWS）的支持就比较有限，而且似乎也并不会在这方面做任何的投入。另外，DMS最近一年的产品大方向主要是在于”一站式的数据管理”，所以新增了数据资产、数据开发任务编排等功能。不再是一个SQL开发工具。

阿里云DMS综合评价如下：

-   产品体验：★★★★
-   云适配度：★★
-   功能完整：★★★★★
-   是否收费：免费
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-20-1024x658.png)

### **BeeKeeper Studio**

  
官网或下载地址：[https://www.beekeeperstudio.io/](https://www.beekeeperstudio.io/)

Beekeeper目前是由一个由个人开发的MySQL GUI软件。界面简洁现代，支持比较基础的SQL开发功能，包括了SQL窗口、创建表等能力，同时有非常好的平台兼容性。向用户提供免费的功能有限的社区版，完整版是收费的，最低价格为19美元。

BeeKeeper Studio综合评价如下：

-   产品体验：★★★★
-   云适配度：★
-   功能完整：★★★
-   是否收费：开源免费（功能限制）+ 商业收费（完整版）
-   环境兼容：Windows / Linux / macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-21-1024x602.png)

**Toad Edge**

官网地址：[https://www.quest.com/products/toad-edge/](https://www.quest.com/products/toad-edge/)

Toad Edge是Quest公司的产品之一。主要支持MySQL和PostgreSQL，当然Toad系列也有支持Oracle/Db2/SQL Server等商业数据库，但都需要下载独立的软件。另外，该软件一般是通过销售渠道去销售的，所以网上也看不到其价格。当前，支持Windows和macOS版本。其功能支持也比较完整，另外，在SQL提示功能上， 比较有特色，支持比较常见的SQL代码提示。

Toad Edge产品综合评价：

产品体验：★★★

云适配度：★

功能完整：★★★★

是否收费：商业收费

环境兼容：Windows / macOS

产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-22-1024x695.png)

**DbVisualizer**

官网地址：[https://www.dbvis.com/](https://www.dbvis.com/)

DbVisualizer发展时间也比较长了，支持的数据库种类也非常多，底层是基于Java构建的，有不错的平台兼容性，支持Windows / Linux / macOS，在市场也获得不错认可。不过，该软件仅支持英语，并没有对应的中文支持。

DbVisualizer综合评价如下：

-   产品体验：★★★★
-   云适配度：★★
-   功能完整：★★★★★
-   是否收费：免费版（功能受限）+ 商业收费
-   环境兼容：Windows / Linux /macOS
-   产品截图：

![](https://www.orczhou.com/wp-content/uploads/2023/01/image-23.png)

### **最后**

通过Wine等方式支持的OS平台，这里并没有考虑，因为根据经验来看，大多数情况下，稳定性都不太好。另外，市面上也还有一些产品超过两年未更新，这里就不再介绍了，例如MyDB Studio；也有部分软件平台属性太强，例如Sequel Pro仅支持Mac，这里也没有介绍。总体上，打分有较强的主观性，所以仅供参考。