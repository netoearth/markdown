多年来，AWS 凭借其快速部署、快速调整、多区域部署、灵活、稳定等特性在市场上备受好评，根据 Synergy Research Group 的 2022 年第二季度统计数据，全球云服务器市场本季度营收 547 亿美金，过去 12 个月营收达到 2050 亿美金。各大云服务供应商中，亚马逊云以接近 34%的市占率排名第一，比第二、第三名加起来都高。

但优势明显的[AWS](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651124439&idx=4&sn=62a689cb15df69b8fb7c0471e8b7c791&chksm=bdb90a848ace83927afe475b487935cd6c69b611088cded812504b180989a149b876e894c3a7&scene=27#wechat_redirect "xxx")却并非适合所有企业，对于很多全公司员工甚至不足 10 人的小型初创公司来讲，AWS 的使用成本还是超出了他们的预期，无奈之下，这些企业不得不另辟蹊径，选择一些更加经济实惠的代替方案。

相比于 AWS，Fly.io 带来了卓越的开发者体验，且上手难度很低。当然，[Fly.io](https://fly.io/ "xxx")也有一些仍显粗糙的部分。如果大家愿意自己打理基础设施，并能够接受缺乏一流技术支持的现实，那 Fly.io 的确值得关注。

总的来说，主流云服务商的产品更“花里胡哨”，Fly.io 则更加简单务实。

### Terrateam 公司简介

[Terrateam](https://terrateam.io/blog/flying-away-from-aws "xxx") 是一个 CI/CD 平台，该公司由实践经验丰富的软件工程师创立，对于 Terrateam 来说，保持简单而有效很重要。

跟其他初创公司一样，Terrateam 刚刚起步的时候，大家都想快速行动、看看业务方向有没有搞头。AWS 当然是最直观的选择，能帮 Terrateam 在短时间内站稳脚跟。只用一点免费积分，整整一年都不用担心基础设施了，简直堪称完美。

但时间过得很快，免费积分用完了。看着 AWS 的账单，Terrateam 团队发现这样的开销对一家全靠自己的初创公司来说实在太高了。于是 Terrateam 团队开始四下寻找更便宜的替代品，同时又不能影响稳定性、安全性和可扩展性。

### 为什么要“逃离”AWS？

迁移的主要动机当然是成本。[AWS可不便宜](https://www.infoq.cn/article/tTWtyKOqhBtGmJWzTZk5 "xxx")，Terrateam 团队表示：“其中很多功能我们也用不上。也许未来用得上吧，但目前没必要”。

在设计上，Terrateam 需要的是一个体量较小的简单基础设施堆栈，具体包括：

-   一套 Postgres 数据库；
    
-   Web 服务：一个简单的 Docker 容器，负责监听一个端口，需要能横向扩展且保持轻量化。这样一套简单堆栈的好处，就是能为他们提供很多供应商选项。Terrateam 知道自己的需求：便宜的 PaaS，同时剔除一切非必要元素。Terrateam 需要的其实是一种类似于现代版 Heroku 的产品。
    

经过快速实验和概念验证之后，Terrateam 把目光投向了 Fly.io，并决定建立一套登台环境来充实更多细节。

### 准备工作

在建立登台环境时，Terrateam 想要一一映射现有 AWS 基础设施中的所有部分，把它们跟 Fly.io 组件匹配起来。

-   AWS ALB → Fly.io Load Balancer
    
-   AWS ECS → Fly.io Nomad
    
-   AWS RDS → Fly.io Postgres (大差不差)下一步，则是创建 Fly.io 应用程序和配置文件。
    

Fly.io 拥有强大的 CLI，允许用户为每个应用程序指定相应的 fly.toml 配置文件。使用此配置，Terrateam 团队创建了应用程序并轻松完成必要配置。整体体验相当不错。

来看看 Terrateam 的登台 fly.toml:

```
app = "terrateam-app-staging"
```

复制代码

### 测试

在着手构建 Terrateam 登台环境时，技术团队很快遇到了以下障碍。好在这些问题都有简单的修复办法。

#### IPv6

Fly.io 应用端点会解析为 IPv6 地址。

josh@elmer:~ $ fly ssh console --app terrateam-app-stagingConnecting to fdaa:0:c037:a7b:c6ef:47dd:247:2... complete/ # dig terrateam-db-staging.internal A terrateam-db-staging.internal AAAA +shortfdaa:0:c037:a7b:c207:e395:9a80:2/ #

但 Terrateam 的应用程序并不支持 IPv6。这是 Terrateam 团队自身的问题，但用上 Happy Eyeballs 算法后，麻烦立马解决。

#### Postgres 与 SSL

[数据库](https://www.infoq.cn/article/K30UQfJt7TqDVLDgjIK2 "xxx")连接不正常。虽然 Terrateam 团队可以通过应用程序解析并抵达数据库端点，但却始终无法连接。技术团队使用的数据库强制要求用 SSL 建立安全连接。当然可以直接覆盖掉这项要求，但这里用 SSL 配置 Postgres 明显更安全。

搜索 Fly.io 文档，似乎没找到用 fly.toml 文件来配置 SSL 的简便方法。经过进一步调查，Terrateam 团队意识到 Fly.io Postgres 跟 AWS RDS 还是有一定区别。官方文档也明确提到，“这不是托管 Postgres。”

Fly.io CLI 提供对创建新数据库的特别支持，但也就只限于创建环节。后续的管理、扩展、升级、故障转移和配置都得由用户亲自动手。这并不是在抱怨，毕竟 Fly.io 的态度非常诚恳坦率，会明确说他们能实现什么、不能实现什么。

要用 SSL 配置 Postgres，首先需要创建证书，之后是在 postgresql.conf 中部署正确配置。到这里，第二个问题顺利解决。

### 正式迁移

在解决了登录环境中的问题之后，是时候创建 Terrateam Fly.io 生产应用程序并正式启动迁移了。

Terrateam 团队讨论了两种迁移方法：

-   零停机实时迁移
    
-   尽可能缩短停机时间的快速迁移
    

#### 实时迁移

根据个人经验，每次谈起迁移时，大家都会先讨论零应用停机完成迁移的实施难度。Terrateam 团队后来整理了一份涉及复杂 Nginx 配置的方案，但最终没有采用。

与实时迁移所对应的努力和复杂程度相比，Terrateam 团队觉得还不如选择短暂停一会儿机的快速迁移。有些事情确实是越简单、越“傻瓜”越好。再加上能够重新发送期间错过的 GitHub webhook，进一步坚定了技术团队用短时间停机换简单迁移的决心。

#### 快速迁移

快速[迁移](https://www.infoq.cn/article/4u0MQ4321CGS7uEAygpY "xxx")就很直观了：

1.  用低 DNS TTL 更新 app.terrateam.io
    
2.  阻断 AWS ALB 上的传入连接
    
3.  将 AWS RDS 数据库迁移至新的 Fly.io 数据库
    
4.  更新 app.terrateam.io 以指向新的 Fly.io 应用端点
    
5.  重新发送遗漏的 GitHub webhook（事实证明并不存在遗漏）优势
    

在用 Fly.io 托管 Terrateam 之后，技术团队表示体会到了不少优势，而且都跟 Fly.io 组织有关。事实证明，Fly.io 非常了解典型工程团队的基础设施构建需求。

#### 可观察性

Fly.io 免费提供良好的可观察性。创建新应用程序时，系统会自动提供 Grafana 仪表板，其中包含各种常用图表。此外，还能轻松在现有图表和仪表板之上，创建更多新型图表和仪表板，大大降低应用程序的观察难度。与 AWS CloudWatch，这简直就是一股清新的空气。

另外，如果将应用程序配置为公开 Prometheus 端点，这些指标也能自动显示在 Grafana 仪表板上。

![](https://static001.geekbang.org/infoq/78/78fa9710242a92f9643f28d62d52f3ee.png)

#### 远程访问

跟 AWS 相比，这又是 Fly.io 另一个大放异彩的特性。Terrateam 经常需要远程访问运行中的容器，特别是在首次构建基础设施或解决持续存在的问题时。有时候，只需要一个 shell。

Fly.io CLI 提供一种非常简单的访问权限获取方式：

josh@elmer:~ $ fly ssh console --app terrateam-app-stagingConnecting to fdaa:0:c037:a7b:c6ef:47dd:247:2... complete/ #

这样做真是太棒了，终于不用受 VPN、SSH 密钥、堡垒主机的折磨，fly ssh console 命令才是真正的便利！

#### IPv6 专用网络

每位 Fly.io 客户都会收到一个带有 IPv6 端点的自动安全专用网络。对于像 Terrateam 这样的简单应用，这可太方便了。用不着自己建立单独的专用网络，也不必担心 CIDR、子网、路由及网络复杂性带来的其他问题。一切都正常运行，一切都刚刚好。

#### 多区域可扩展性

Fly.io 的多区域可扩展性表现极佳。通过在 fly.toml 中指定多个区域，Terrateam 的应用程序就能神奇地存在于多个区域中。这些区域可以随时变更，无需停机。对其他云服务商来说，同样的效果可没这么容易实现。

#### 简洁的 UI

Fly.io 的仪表板非常简洁、条理清晰且易于导航。它不像其他云厂商的仪表板那样杂乱无章。事技术团队总能在其中轻松找到自己需要的条目，请务必保持下去。

![](https://static001.geekbang.org/infoq/f4/f47a665383c773fccb0d667f1880042e.png)

### 短板

#### Terraform 提供程序

当然，免费的软件也有其自身短板。技术团队原本想用 Terraform 创建所有内容，但很快意识到根本不行。Fly.io Terraform 的提供程序不够强大，无法创建完整环境。虽然令人失望，但技术团队还是决定继续前进。请注意，这对 Terrateam 是个大问题，因为 Terrateam 是一家强调易用性的公司。但技术团队已经定下计划，后续会逐渐为 Fly.io Terraform 提供程序做贡献。

#### Postgres 的可用性问题

Fly.io 使用[Stolon](https://github.com/sorintlab/stolon "xxx")集群管理器来实现 Postgres 的故障转移。技术团队表示：“Stolon 的故障转移可靠性并不好。这是个问题，毕竟它就是专为这事而存在的。”

技术团队成员表示，自己曾经历过一次事故，被迫以手动方式创建新数据库并从备份中恢复。Fly.io 已经收到反馈，并努力用更强大的方案取代 Stolon。

#### 日志记录

Fly.io 提供的容器日志方案确实有点简陋了。虽然用 Fly.io CLI 和 Fly.io 仪表板都能轻松查看日志，但其中只保留少部分内容。所以，技术团队只能在应用之内或将日志发送至外部服务，借此为各 Fly.io 应用程序建立单独的远程日志记录方案。这会带来额外的运营开销，希望以后会有改善。

![](https://static001.geekbang.org/infoq/0e/0eaa79faadf8f6884f0ecc5bd5c00cda.png)

#### 技术支持

Fly.io 并不是那种服务很周到的企业，如果大家特别需要随时获取技术支持，但 Fly.io 恐怕不太合适。

在通过电子邮件发送支持时（目前只提供邮件联络选项），往往要几个小时甚至几天后才能得到回复。有时候 Fly.io 那边干脆没有跟进，感觉他们的支持水平不如其他云服务商。

但毕竟 Fly.io 强调的就是客户自主运营基础设施，他们只负责提供构建块。有良好的服务支持当然好，但他们认为环境的管理工作最终还是客户的事。

Terrateam 团队表示，对于 Fly.io 还是充满了期待的，还是希望未来能有更好的支持体验。即使没法直接从工作人员那得到具体答案，至少也应该得到一点指引。

### 写在最后

没有哪套平台真正完美无缺，而且在 Terrateam 看来，Fly.io 甚至已经在可能的范围内做得足够好。Terrateam 团队在使用后表示对它没有太多抱怨，特别是考虑到他们明确解释了自己的产品能做什么、不能做什么。

Terrateam 的堆栈设计非常简单，也许这才是 Fly.io 特别契合需求的原因。Terrateam 不需要太多活动部件或基础设施，最核心的元素就是数据库加容器。

但各位读者朋友的需求可能并非如此，如果大家需要消息中间件、S3 存储桶、IAM 等更完备的服务组件，那 Fly.io 可能不适合你。

参考链接：

[https://terrateam.io/blog/flying-away-from-aws](https://terrateam.io/blog/flying-away-from-aws)