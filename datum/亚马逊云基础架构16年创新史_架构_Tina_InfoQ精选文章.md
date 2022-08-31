在亚马逊的每一份年报中，Jeff Bezos 都会附上一份 1997 年[致股东信](https://s2.q4cdn.com/299287126/files/doc_financials/annual/Shareholderletter97.pdf "xxx")的原件副本。在信中，Bezos 概述了亚马逊是否成功的基本衡量标准：坚持不懈地关注客户、创造长期价值而不是关注企业短期利润，以及持续进行大胆的创新。Bezos 写道，“如果我们执行得很好，那么每天都是‘第一天（Day one）’。”

2006 年，亚马逊推出了 Amazon Web Services，从此开创出了云计算市场。过去的十一年，亚马逊在 Gartner 的云基础设施和平台服务魔力象限 (CIPS) 上一直处于遥遥领先的领导者位置。在云计算行业普遍的盈利困境下，今年 2 月亚马逊公布了上一财年第四季度数据，亚马逊云服务本季营收 178 亿美元，同比增长 40%，营收增速在 2021 年连续 4 个季度持续提升。

如今，云计算领域竞争对手均日渐成熟，市场竞争异常激烈，而亚马逊却能一直保持领先地位，一直被追赶，但从未被超越。如果从亚马逊云科技诞生与发展的过程来看，你会发现“基于客户需求，快速的产品更新与技术迭代”的 Day one 理念始终贯穿其中，从而能让亚马逊一路领跑云计算市场。

## 云计算的诞生

亚马逊云科技的技术思想实际诞生于 20 年前，非常具有前瞻性。当年亚马逊推出的 Prime 服务为用户数量带来了爆发性增长，于是亚马逊开始计划第二个名叫“Merchant.com”的项目，目的是为大型零售客户提供在线体验。

该项目投入了很多工程师，但没能如期交付。因为当时 Amazon.com 采用的是单体应用程序，庞大而且多平台组件交织在一起。为了给客户添加新功能，开发人员必须在这个单体程序上从零编辑和重写大量代码，主要是计算和数据库。此外，每次团队解决问题时，都仅限于解决自己项目中的问题，大家都在做着重复的工作。

为了加快应用程序的迭代过程，亚马逊以非常严格的方式理清业务并重新设计了架构，将单体程序重组为被称为“服务”的更小部分，还拆解了开发人员的职能层次结构，形成众多小型的“两个披萨团队”。每一个小团队都集中在特定的产品、服务或功能集上，赋予他们对应用程序特定部分的更多权限，以加快对自己负责的产品的决策过程。最后，Merchant.com 服务得以成功推出，亚马逊为用户提供了可自己设置价格、标题和可用性的 API，在没有任何推广的情况下，成千上万的开发者涌向这个 API。而这种用来构建 Merchant.com 应用程序的“微服务”思想，延续至今已经彻底改变了现代应用程序的构建方式。

分解应用程序和组织结构，这只是故事的一半，另一半是关于“基础架构”的。

2003 年，亚马逊网站工程经理 Black 写了一篇简短的论文，论述了一种重组亚马逊基础设施的方法，提出了“将虚拟服务器作为服务出售”的可能性。虽然与零售业务无关，但显然 Bezos 认可并推动了它。

2006 年，亚马逊云科技正式推出了他们的前三款产品：EC2（弹性计算机云）、S3（简单存储服务）、SQS（简单队列服务）。亚马逊将所有的 IT 基础设施都分化成了最小的单元，其中包括网络、存储、计算等。开发者可以自由选择这些单元，以及亚马逊云科技提供的软件服务，来构建自己的产品。

亚马逊 CTO [Werner Vogels 博士](https://www.quora.com/How-and-why-did-Amazon-get-into-the-cloud-computing-business-Rumor-has-it-that-they-wanted-to-lease-out-their-excess-capacity-outside-of-the-holiday-season-November-January-Is-that-true-10 "xxx")说，“亚马逊想用可扩展系统软件方面的专业知识，将原始基础设施通过服务接口的方式交付，让开发人员不再专注于构建、维护基础设施。这是一个关于‘创新’的故事，至于‘因产能过剩而发明云计算’的说法是不对的，因为 Amazon Web Services 推出之后的 2 个月内，就已经耗尽了 Amazon.com 所有的过剩产能。”

[随后十年](https://perspectives.mvdirona.com/2016/03/a-decade-of-innovation "xxx")，亚马逊逐渐推出了更多服务，Amazon Simple DB、Amazon CloudWatch、Amazon Route53、Amazon CloudFormation、Amazon Dynamo DB 等等。随着服务的完善，越来越多企业迁入了云中。作为独角兽迅速崛起的 [Slack 公司](https://aws.amazon.com/cn/solutions/case-studies/slack/?did=cr_card&trk=cr_card "xxx")，在 2015 年分享了他们的构建方式：使用 Amazon EC2 实例进行计算，用于 Amazon S3 存储用户上传的文件和静态资产，用 Elastic Load Balancing 来平衡 Amazon EC2 实例之间的工作负载，以及使用 Amazon Elastic Block Store (Amazon EBS) 对 Amazon EC2 实例上运行的 MySQL 实例进行夜间备份。

![](https://static001.geekbang.org/wechat/images/ca/caf82cee546ab5cd7659d9031e45c5dd.png)

_来源：_[_https://aws.amazon.com/cn/solutions/case-studies/slack_](https://aws.amazon.com/cn/solutions/case-studies/slack)  

如同 Slack 公司一样，[不同的企业](https://aws.amazon.com/cn/solutions/case-studies "xxx")可以根据自己的需求选择不同的服务，构成自己的 IT 架构，企业不用再自己解决基础问题，免去重复性造轮子的工作。可以说，云技术是亚马逊的技术发展到一定程度后，得到的一种资源优化方法，一种系统性的创新方法。

截至 2021 年，亚马逊云科技包含超过 245 种产品和服务，包括计算、存储、网络、数据库、分析、部署、管理、机器学习、开发者工具等。虽然数量众多，但这种模块化、单元化的技术解决方案，作为一个整体在设计上又特别协调和流畅。据[媒体报道](http://www.kejilie.com/tmtpost/article/BZZJZj.html "xxx")，有国内知名技术专家参加 2016 re:Invent 时感叹：“看 Amazon Web Services 的架构，有一种感觉仿佛整个 Amazon Web Services 是一个人做的一样，模块定义、复用、互通，看的真是一个赏心悦目。”

## 重构云底座：构建可进化的系统

### 计算

2006 年 8 月，Amazon Elastic Compute Cloud (EC2) 开放了 beta 测试，并从此启动了云上计算革命。如果没有这种计算能力上的创新，我们认为现在一些理所当然的事情——从外卖、快递中的调度计算，到生命科学中的基因计算，都是不可能被轻松实现的。

亚马逊最初的想法是为开发人员提供针对计算基础设施的按需访问，并让他们只为自己所使用的资源付费。

EC2 服务副总裁 Dave Brown 曾回忆道：“最初在南非的开普敦建立研发中心时，亚马逊云科技团队只有 8 个人，但我们当时做的事领先于时代。以至于当我们在 2006 年 8 月发布产品时，Reddit 和 Slashdot 上的大多数评论，都表示很难理解我们在做什么。一个世纪前，美国的很多大型制造企业自己发电为工厂供电，随着电网的普及，公司开始关闭自己的发电厂，因为他们可以随时随地获得更经济的电力资源。计算领域也应如此，就像早期电网的转变一样，算力可以在你需要的时候打开和关闭。”

亚马逊最初选择在开源 Xen 上进行修改定制的办法来实现 EC2 架构，通过 Xen hypervisor 虚拟化 CPU、存储和网络，并提供丰富的管理能力，让多个虚拟机 (VM) 在一台物理机器上运行，由管理程序提供虚拟机之间的隔离和多租户功能。

![](https://static001.geekbang.org/wechat/images/92/923f26ce519dfc920d92205185b40d31.png)

_Xen Hypervisor EC2 架构，来源：_[_https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html_](https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html)  

经过数年发展，EC2 成为了一个庞大、稳健的环境和数十亿美元的业务。虽然传统虚拟化架构已经被亚马逊优化到了极限，但是使用这种架构，一个实例中[多达 30% 的资源](https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html "xxx")被分配给了虚拟机管理程序以及网络、存储的监控运营。为了降低这些损耗，亚马逊云科技开始定制化专用硬件，从头开始重新发明底层 EC2 技术。

这不是一蹴而就的事情，2013 年，亚马逊发布了 EC2 C3 实例，将网络进程卸载到硬件（将功能从软件转移到硬件）。2014 年，亚马逊云科技与 Annapurna Labs 合作，再次推出了 EC2 C4 实例。C4 将 EBS 存储卸载到硬件中，但仍然依赖英特尔至强芯片作为控制器。据报道，亚马逊云科技在 2015 年斥资 3.5 亿美元收购了 Annapurna，引入 C5 实例，用 Nitro hypervisor 取代了 Xen，将虚拟机与 ASIC 紧密耦合。Werner Vogels 表示，这个里程碑卸载了控制平面和其余 I/O，使用近 100% 的处理来支持客户工作负载，还启用了裸机版本的计算，催生了与 VMware 的合作伙伴关系，以启动 VMware Cloud on AWS。2017 年，亚马逊正式推出完整版 Nitro。

Nitro 是一组自定义硬件和软件，目的是将虚拟机管理程序、网络和存储虚拟化转移到专用硬件，从而释放 CPU 以更高效地运行。

![](https://static001.geekbang.org/wechat/images/b9/b980a9cdf53f3ad88893240470232af9.png)

_2017 年的 Nitro 系统架构，来源：_[_https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html_](https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html)  

2018 年，亚马逊云科技推出了基于 Arm 的定制芯片 Graviton。据相关数据显示，基于 Arm 的 Graviton2 的性价比比基于 x86 的同类实例高 40%。这打破了对 x86 的依赖，开启了架构的新时代，现在亚马逊已经能够以多种配置支持密集型计算任务。这些举措为亚马逊云科技的其它创新奠定了基础，包括针对从训练到推理环节的机器学习和人工智能任务进行了优化。

自推出 Nitro 系统之后，EC2 实例也快速增加，现在 Amazon EC2 已经拥有超过 475 个实例。计算方式也不断更新，从 EC2 实例开始，逐渐支持容器和无服务器。一般云平台只提供 Kubernetes 托管，亚马逊云科技提供了 EKS（托管 Kubernetes 服务）、 ECS（自研）和 [Fargate](https://www.allthingsdistributed.com/2018/04/changing-calculus-containers-cloud.html "xxx") 三种容器管理工具。Amazon Lambda 也开创了无服务器计算时代，无服务器计算是一种按需提供后端服务的方法。无服务器提供者允许用户编写和部署代码，而不必担心底层基础设施。

### 存储

2006 年，亚马逊云科技推出了 S3 (Simple Storage Service) 服务，S3 定义了对象存储，是对象存储事实上的标准，具有划时代的意义。

S3 的推出时间实际上比 EC2 还早 6 个月，最初设计时的一个重要原则就是“简单”，所以当时只提供了 GetObject 和 PutObject 功能，核心 API 只有四个 REST 命令（GET、PUT、LIST 和 DELETE）。Werner Vogels 和亚马逊前技术副总裁 Tom Killalea 在[谈到 S3 的发展](https://dl.acm.org/doi/fullHtml/10.1145/3434571.3434573 "xxx")时，两人认为“‘尽可能的简单’是 S3 成功的关键之一，虽然这在当时颇具争议，但一个可进化的架构一开始不可能被设计得很复杂。”

开始时用户主要是用 S3 存储图像和视频数据，但随着时间的推移，越来越多的事务日志、parquet 文件、客户服务记录等数据被放进了 S3。然后用户想要的就不仅仅是“弹性存储”和“低成本”了，他们还希望能够让数百个应用程序使用这些共享数据集，于是亚马逊就添加了“Access Points”功能。[这个过程](https://www.youtube.com/watch?v=6ryVoGlgwU4 "xxx")足以说明 S3 的演进原则：用户希望用他们的数据来做什么，亚马逊云科技就添加什么功能或服务。通过技术和商业的服务，亚马逊云科技与用户建立了一个快速的反馈循环，成为一个快速成长的飞轮。

2006 年的时候，S3 只有 8 个服务（Services），到 2019 年，S3 已经拥有 262 个了。而且亚马逊致力让 S3 拥有更高性能、更低成本，推出了七种不同层次的存储产品：Standard、Intelligent Tiering、Infrequent Access、One-Zone Infrequent Access、 Glacier、Glacier Deep Archive，以及 S3 Intelligent Tiering。S3 Intelligent Tiering（智能分层）产品又分为“频繁访问”和“非频繁访问”两个层级，会自动将连续 30 天未访问的对象移动到“非频繁访问”层，降低了运营复杂度。2021 年，智能分层也从 2 个层级增加到了 3 个层级，添加了新推出的归档即时访问层，最经典的存储仍在不断进化。

![](https://static001.geekbang.org/wechat/images/c8/c8e84cdf5b278b22071cc950081e83af.png)

S3 存储本身具备了计算存储分离的特性，在云原生时代，非常适合作为[数据湖存储的核心](https://www.infoq.cn/article/0BPZIkj7qgxsT91khHue "xxx")。企业可以基于 Amazon S3 构建数据湖，同时利用原生 Amazon Web Services 服务，来运行人工智能或机器学习服务（SageMaker），从而可以更高效地处理各种结构化和非结构化数据。

S3 持久性设计为“11 个 9”(99.999999999%) ，意味着使用 Amazon S3 存储 10000000 个对象，则预期平均每 10000 年发生一次对象丢失。今年，亚马逊宣布 S3 存储的对象数量已经超过 200 万亿，每秒需要处理数千万个请求。如今，S3 已经演变为了庞大而健壮的分布式存储系统，为保持数据持久性，亚马逊于去年底宣布升级了 S3 的存储后端系统 ShardStore，[引入了“自动推理”方法](http://muratbuffalo.blogspot.com/2021/10/using-lightweight-formal-methods-to.html "xxx")，以保证“崩溃一致性”，即系统崩溃时数据仍能保持“11 个 9”的一致性状态。

ShardStore 实现比较复杂，包含超过 4 万行 rust 代码，使用 soft update 提供崩溃一致性，传统的验证方法速度无法跟上系统的开发迭代速度。亚马逊采用了轻量级形式化方法（lightweight formal methods）提高 ShardStore 的可靠性，自动生成一系列的操作调用键值存储系统的接口，同时检查操作过程中参考模型和 ShardStore 具体实现之间的行为和状态是否一致。基于上述方法，亚马逊成功在 ShardStore 找到并修复了 16 个重要的问题，涉及崩溃一致性和并发等方面的错误。亚马逊云科技还在 SOSP 大会上发表了一篇相关论文，并获得了[最佳论文奖](https://www.amazon.science/blog/aws-team-wins-best-paper-award-for-work-on-automated-reasoning "xxx")。

### 网络

网络是云计算业务最基础的支撑之一，亚马逊云科技拥有规模最大的全球骨干网络，现有 25 个区域，81 个可用区，14 个 Local Zone，17 个 Wavelength Zone，超过 300 个边缘站点，以及 108 个专线接入点。这些都是在 2006 年单一扁平子网的基础上，历经 16 年的不断创新和优化发展而来。

在全球化浪潮下，不少大型跨国企业会在全球设置多个站点，在构建全球化网络时，亚马逊云科技的用户可以利用 Amazon VPC 创建多个虚拟网络。传统的做法是利用 VPC Peering 功能，将区域内或者跨区域之间的 VPC 进行连接，利用 Direct Connect 或 VPN 实现非亚马逊云科技基础设施与亚马逊云科技的互联。但如果云上负载增多，管理工作将成倍增加，这时就可以采用集中管理链接的方案，亚马逊在 2018 年 Re:Invent 推出了 Amazon Transit Gateway。使用 Amazon Transit Gateway，可显著简化管理并降低运营成本，因为每个网络只需连接到 Transit Gateway，而不是连接到所有其他网络。

![](https://static001.geekbang.org/wechat/images/e3/e3584f3e4f9ad3e408c828e462e84813.png)

当网络在全球范围内延伸并且还需要混合各种技术时，企业构建、管理和监控它们的复杂度也会明显增加。在 2021 年 re:Invent 大会上，亚马逊云科技宣布了 Cloud WAN 全球网络托管服务。借助这项网络服务，企业可以借助于亚马逊的骨干网，使用 Cloud WAN 图形界面一键创建属于自己的全球网，实现设置中转网关或云连接，监控网络运行状况、安全性和性能等功能。

![](https://static001.geekbang.org/wechat/images/ca/ca2f0357b9b94b5413fb3d09249e812e.png)

_来源：https ://aws.amazon.com/cloud-wan/_

在 2021 年的 re:Invent 大会的[主题演讲](https://www.youtube.com/watch?v=8_Xs8Ik0h1w "xxx")和[博客文章](https://www.allthingsdistributed.com/2021/12/tech-prediction-for-2022-and-beyond.html "xxx")中，Werner Vogels 还提到了亚马逊对“无处不在的云（the everywhere cloud）”的愿景，即通过有针对性的硬件和解决方案将亚马逊云科技带到新的地区。

![](https://static001.geekbang.org/wechat/images/da/da56d34ebfcd66ad6f7e98f389ab8076.png)

Werner Vogels 在博客中写道：“我们将在 2022 年见证转变：云将在边缘上变得高度专业化。我们会为边缘提供量身定制的解决方案，那么无论是车间、餐厅、零售小店或偏远地区，都可以充分发挥云的优势。”

### 安全

没有安全这个保障，云的一切优势都将无从谈起。随着上百万的组织迁入云中，数据、流量更为集中，云早已成为安全攻防的主战场。

作为云计算的先驱，亚马逊首创的“[安全责任共担模型](https://pages.awscloud.com/apn-tv-authority-to-operate-ep-007.html?trk=apn-tv-carousel "xxx")”已经是云安全联盟中大家公认的事实上的行业标准，这个模型明确了云厂商和租户的安全边界，也明确了云厂商内部的安全责任。今年，亚马逊云科技再次提出了五层“[洋葱型防护方法论](https://www.infoq.cn/article/UvAgT2AfJZLwOWClBI9C "xxx")”：威胁检测与事件响应、身份认证与访问控制、网络与基础设施安全、数据保护与隐私、风险管控及合规。在 280 多项安全、合规服务及功能基础上，亚马逊云科技利用这五层防护体系，为客户提供全方位的安全服务。

### 可持续发展

联合国于 2015 年制定了一个全球框架《巴黎协定》，随后各缔约国纷纷制定了“碳中和”路径和目标，对地球环境的健康发展做出承诺。埃森哲的分析表明，倘若采用绿色方法迁移至公有云，全球二氧化碳排放量每年可减少 5,900 万吨，这相当于动动手指就能减少 2,200 万辆汽车的碳排放量！

亚马逊作为世界级科技巨头，引领了“绿色云”改造。亚马逊表示将提前十年达成《巴黎协定》，并在 2025 年实现 100% 可再生能源，而且还设计了一套从基础设施[到软件设计](https://www.infoq.cn/article/9diy3t8nemohhpu9qfe2 "xxx")的具有前瞻性的解决方案。这些举措也取得了显著的成果，2021 年，亚马逊云科技可持续发展架构副总裁 Adrian Cockcroft 表示：亚马逊云科技的基础设施能源效率比普通美国企业数据中心高出 3.6 倍。同时，亚马逊在执行相同任务时，可以减少 88% 的碳足迹。

## 重构云底座：面向客户和未来进行创新

诞生于 16 年前的亚马逊云科技，开创了一个全新的云计算领域，亚马逊的创新可以说对 IT 产业演进产生了革命性的影响。

虽然如今各云厂商竞争激烈，但亚马逊云科技却始终能处于市场的领先位置。据相关数据显示，亚马逊云科技在云基础设施服务提供商中的份额最大，为 33%，客户也早已超过百万，无论是技术巨头、银行还是政府，不同的组织都在使用 Amazon Web Services 来开发和部署自己的应用程序。

早期的典型用户有 Netflix，从 2009 年就开始采用 Amazon Web Services，2015 年中 Netflix 关闭了自己的最后一个数据中心。[纳斯达克](https://aws.amazon.com/cn/solutions/case-studies/nasdaq-case-study "xxx")从 2014 年就开始使用 Amazon Web Services 在云中存储股票交易所数据，今年再次增加了边缘解决方案的使用，将 Markets 逐步开始迁移到亚马逊云服务上。[NASA](https://aws.amazon.com/cn/partners/success/nasa-image-library "xxx") 于 2000 年开始，利用 Amazon Web Services 服务，提供对照片、视频和音频的在线访问，上周，[NASA 再次](https://aws.amazon.com/cn/blogs/publicsector/amazon-aws-reimagine-space-station-operations-logistics-orbital-reef "xxx")宣布采用亚马逊云服务为空间站构建“太空物流”基础设施体系…

这样的成绩归功于亚马逊不断地围绕客户业务进行技术创新，有业界专家认为，亚马逊云科技的一大亮点是能非常敏感地发现用户当前紧迫面临的是什么问题，并快速提供解决方案或者产品。这也像 Dave Brown 所说的：“我们可以应对几乎所有需求以及挑战，并且永远不会对客户说不。在亚马逊云科技，我们拥有‘顾客至尚’的文化，不仅满足他们当前需求，而且会预测他们未来的需求。”以客户为中心，不断进行创新，这也正是“第一天（Day one）”理念的一种体现。

面向未来的发展过程中，亚马逊云科技在这 16 年当中无疑有很多技术理念和决策经验值得我们借鉴和思考。

2022 年 4 月 20 日，亚马逊云科技将在线召开 INNOVATE 创新大会，这也是亚马逊首个专门针对云基础架构的技术大会，设置 6 大分会场、30+ 前沿技术主题，将**更全面、更详细**地分享亚马逊云科技的底层创新，解读不同业务场景的应用，帮助您构建数据驱动型企业！

![](https://static001.geekbang.org/wechat/images/1e/1e8b3e0cd66c11db7d6156c7ce257222.jpeg)

_参考链接：_

[https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html](https://www.allthingsdistributed.com/2020/09/reinventing-virtualization-with-nitro.html)  

[http://muratbuffalo.blogspot.com/2021/10/using-lightweight-formal-methods-to.html](http://muratbuffalo.blogspot.com/2021/10/using-lightweight-formal-methods-to.html)  

[https://www.youtube.com/watch?v=6ryVoGlgwU4](https://www.youtube.com/watch?v=6ryVoGlgwU4)  

[https://perspectives.mvdirona.com/2016/03/a-decade-of-innovation/](https://perspectives.mvdirona.com/2016/03/a-decade-of-innovation/)  

[https://thestack.technology/aws-shardstore-s3/](https://thestack.technology/aws-shardstore-s3/)  

[https://dl.acm.org/doi/fullHtml/10.1145/3434571.3434573](https://dl.acm.org/doi/fullHtml/10.1145/3434571.3434573)  

[https://www.allthingsdistributed.com/2021/12/tech-prediction-for-2022-and-beyond.html](https://www.allthingsdistributed.com/2021/12/tech-prediction-for-2022-and-beyond.html)  

[https://aws.amazon.com/cn/blogs/publicsector/amazon-aws-reimagine-space-station-operations-logistics-orbital-reef/](https://aws.amazon.com/cn/blogs/publicsector/amazon-aws-reimagine-space-station-operations-logistics-orbital-reef/)  

点个在看少个 bug 👇