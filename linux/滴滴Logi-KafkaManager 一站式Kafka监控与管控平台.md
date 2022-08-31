![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYg37bHOBNU8aibIN90QB2m9icxIX28qmL45XJOoNOOd23GCgvYlZrEODg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**桔妹导读：** LogI-KafkaManager脱胎于滴滴内部多年的Kafka运营实践经验，是面向Kafka用户、Kafka运维人员打造的共享多租户Kafka云平台。专注于Kafka运维管控、监控告警、资源治理等核心场景，经历过大规模集群、海量大数据的考验。内部满意度高达90%的同时，还与多家知名企业达成商业化合作。 

滴滴内部统一使用 kafka 作为大数据的数据通道，当前滴滴共有几十个 kafka 集群，450+ 的节点，20k+ 的 kafka topic，每天2w亿+的消息量；每周500+UV用户，需要完成 topic 创建、申请、指标查看等操作；每天运维人员还有大量集群、topic运维操作。因此我们需要构建一个Kafka的管控平台来承载这些需求。

在平台建设的初期，我们调研了社区同类产品的使用情况，在调研中发现，外部同类产品无论在监控指标的完善程度、运维管控的能力亦或是使用的体验、还是整体的安全管控上都无法很好的满足我们的需求，因此自建滴滴 kafka 云管控平台势在必行。

经过前期产品同学的反复调研和设计、中期研发同学高质量的实现、后期针对用户体验反馈的持续迭代，kafka 云平台上线后广受内部用户和运维人员的好评，使用满意度达到90分。与此同时，还进行了开源(https://github.com/didi/Logi-KafkaManager)和商业化输出，并被多家企业用户成功采购。

免费体验地址：http://117.51.150.133:8080/kafka ，账户admin/admin。

_**1.**_ 

**需求分析**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在 Kafka 云平台建设的初期，我们对之前所面临的问题和需求进行了归纳分析，主要有以下几点：

-   **数据安全性**
    

由于绝大多数用户使用的kafka topic 都会由公共集群来承载数据的生产和消费，而当前 kafka 引擎在 topic 级别的安全管控手段较为薄弱，任何人只要知道kafka集群地址和相应的topic便可进行消费。这无疑会造成数据泄漏的安全风险，因此数据的安全性成首要被解决的问题。

-   **服务的稳定性**
    

滴滴内部绝大部分的 topic 是在共享集群上，共享集群下多 topic 之间存在着相互影响的问题。如某个 topic 的流量突增可能会大面积地影响其他 topic ，从而导致业务的抖动和集群的不稳定。因此在共享集群下，kafka 服务的稳定性也成为了一个必须被解决的问题。

-   **用户使用友好性**
    

滴滴内部每周有大量的用户需要进行 topic 的创建、消费、扩partiton、指标查看等操作，用户高频使用的功能需要做的足够的友好和容易上手，这样才能最大的简化用户的操作和减低日常开发和运维人员的答疑成本。因此用户高频操作的平台化支撑，则成为接下来需要解决的问题。

-   **问题定位高效性**
    

在日常使用 kafka topic 的过程中，会有大量的问题需要查看和定位，如：消息生产和消费的速度、消息堆积的程度、partition的均匀度、topic的分布信息、broker的负载信息、副本的同步信息等，如何使用户和运维人员快速高效的定位问题、处理问题，是重点需要解决的问题。

-   **运维管控便利性**
    

在日常的运维中，会存在着大量的集群、topic的运维操作，如：集群的部署、升级、扩缩容、topic的迁移、leader rebalance等高频高危的操作，怎么样在提升运维操作效率的同时，还要保证高危操作不会影响集群稳定性，这个也是需要去重点考虑。

_**2.**_ 

**整体设计**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从以上的需求分析可以看出，滴滴 kafka 云平台建设需要解决的问题比较多元，因此在设计之初就需要对整体有一个清晰的思路和规划，为此我们定义了一个核心设计原则，并对业务进行了合理的分层用以指导我们后续的产品设计和代码开发。

**********▍********1. 核心设计原则******

在平台的整体设计上，我们制定了“一点三化”的设计原则：

-   **一点：**以安全和稳定为核心点，建设 kafka 的网关系统，针对 topic 的生产/消费提供安全校验，同时提供多租户的隔离方案解决共享集群下多 topic 相互影响的问题；
    
-   **平台化：**着重建设 kafka 云平台，反复进行需求调研和产品设计，提炼用户和运维的高频操作，将这些操作都通过平台实现，降低用户的使用成本；
    
-   **可视化：**提升topic/集群监控、运维过程中指标的可观察性，所有指标尽量能在平台上可以直观体现，方便使用者及时感知集群运行状态，快速定位问题；
    
-   **专家化：**将日常集群运维的经验沉淀在平台上，形成专家服务能力和智能化能力，进一步降低 kafka 集群的维护成本，提升整体稳定性。
    

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYWfJFFxNZbTqkAeHxfpAhvA7BgBKOo9DMBoyvwbhicWg3sEtLibhmEj8Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**********▍********2. 业务分层架构******

在滴滴 kafka manager 具体的业务设计上，我们采取分层设计，从下至上分为以下几层：

-   **资源层：**滴滴 kafka 引擎和 kafka manager 除了 zookpeer 之外只依赖 msyql，依赖精简，部署方便；
    
-   **引擎层：**当前滴滴 kafka 引擎版本是2.5，我们在此基础上开发了一些自己的特性，如磁盘过载保护，并且完全兼容开源社区的 kafka；
    
-   **网关层：**引擎层之上我们设计了网关层，网关层的设计非常巧妙，主要提供：安全管控、topic 限流、服务发现、降级能力，具体详见后文“安全性”的内容；
    
-   **服务层：**基于kafka gateway 我们在 kafka manager 上提供了丰富的功能，主要有：topic 管理、监控管理、集群管理等；
    
-   **平台层：**对外提供了一套 web 平台，分别针对普通用户和运维用户，提供不同的功能页面，尽可能的将一些日常使用中的高频操作在平台上进行承接，降低用户的使用成本。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYyEREXiaV022HStZnmia2MlGo9amCdJEtjkIegPlZBOTml9HJj8qr0T3g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**********▍********3. 应用逻辑架构******

在实际的应用部署和关联上，整体的 kafka manager 和 kafka 引擎、kafka gateway 之间的逻辑关系比较简单，具体如下图所示：

-   kafka gateway 包括两大块功能：服务发现、元数据网关，详细介绍见后面的元数据网关设计一节。
    
-   kafka manager 提供我们开发的特色功能，如：topic管理、监控管理、集群管理，以及相应的 web 平台，普通用户和研发运维人员日常操作接触最多，最高频的操作都将这上面完成。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhY3mf8wN1uNRKEr3CCeicsjddoibDtKicvyiahGUfsRJayqP9m2BVibsAg8jg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

_**3.**_ 

**安全性**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)以kafka 数据的安全性和集群使用过程中的稳定性保障是我们在构思和设计整个 kafka 云平台时首要考虑的问题，为此我们设计了一套 kafka 元数据网关和多租户的隔离模型来解决这些问题。

  

******▍********1. 元数据网关设计**

用户在接入原始的 kafka 集群时没有安全管控，无法有效的管理用户的使用行为，对数据安全和集群稳定性造成严重的风险，因此我们设计了一套 kafka 集群元数据网关系统来解决安全问题。

kafka gateway 系统除了解决数据的安全风险还附带了以下作用：

-   提供 kafka 集群的服务发现服务节点，这样用户在使用 kafka 集群服务时，当 kafka cluster boostrap 地址变更时，则不需要用户改动 kafka 的连接地址，不用重启 kafka client，保障集群的变更对用户透明；
    
-   提供 kafka topic 生产和消费的限流能力，避免用户无限制的使用集群的流量，流量大的用户会耗尽系统资源从而影响其他用户的使用，造成集群的节点故障；
    

需要注明说明一点，kafka gateway 的设计很巧妙的将这些功能实现在 kafka 引擎内部。因为 kafka 作为一个消息系统，其本身流量是非常的巨大，如果 kafka gateway 作为一个单独应用存在，所有流量都先通过 gateway 再进入 kafka 引擎，则对 kafka gateway 机器资源的消耗是非常巨大的，基本等同于需要增加一倍的机器资源，并且整个流程的耗时也会增加。所以在设计之初，就把 kafka gateway 和 kafka 引擎合在一起，这便是滴滴 kafka 引擎的特色能力所在。

搭建好 kafka gateway 系统之后，用户使用 kafka topic 的流程变为如下：

-   在 kafka manager 上申请租户(appid)，获取到对应的租户密码(app-password)；
    
-   利用 appid 申请创建新的 topic，或者申请已存在的 topic 的生产和消费权限；
    
-   使用 kafka client 时设置好对应的 appid 和 app-password，相应的 topic 鉴权，限流都在 kafka broker 端完成。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYQQibUbPdGCHN3YNlDrlYV7MfrQknqxtBn5zVe1IuqJH9sTlQ94jRwSg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

******▍********2. 多租户隔离模型**

在滴滴内部，绝大部分的 topic 是在共享集群上的，在没有管控的情况下，topic 的各个分区是随机的分布在不同的 broker 上，随着集群中 topic 数量的增加，topic 流量的变大，某个具体 topic 的抖动有可能会影响到其他的 topic，因此共享集群下的 topic 隔离就是非常必要的，为此我们结合 kafka gateway 在 kafka manager 上设计了一套多租户隔离模型来解决这个问题，具体操作如下：

-   对将kafka 集群中的各个 broker 进行划分，一组 broker 被分成一个 region，这个 region 在业务上被抽象为一个逻辑集群；
    
-   用户在申请创建新的 topic 时需要选择一个逻辑集群，这样 topic 的分区只能创建在一个逻辑集群上，也就是一组 broker 上；
    
-   具体 topic 消息的生产和消费就只会和一组 broker 发生关系，如果某个 topic 的抖动也只会影响特定 broker 上的其他 topic，从而将影响限定在一定的范围内。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYxm3cGJMKPa1zeIzOaiaBM3yXFrzafkdlk1FfrUMcQXb0MYB8QMAy6Vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

_**4.**_ 

**平台化**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

滴滴 kafka manager 平台建设之初就把易用性作为主要目标，因此在产品设计上非常注重用户的使用体验，前期通过反复的用户调研和内部讨论，最终提炼出普通用户和运维用户的高频操作，将这些操作都通过平台实现，降低用户的使用成本。

-   **方便的日常用户使用**
    

1.   用户只关注自己的Topic的信息(默认)，以减少不必要信息的干扰；
    
2.   足够的指标信息，以保证用户能自助解决问题的同时减少不必要信息的干扰；
    

-   **高效的运维操作**
    

1.  提炼用户、运维人员创建Topic、调整配额、扩展分区、集群迁移、集群重启等高频运维操作，将高频操作平台化，简化用户操作，大大降低运维成本。
    
2.  提供整体集群直观的全局视角，以提高排查问题的效率以及对集群规模的直观感知，并提供详尽的局部视角，以提高排查问题的效率；
    

-   **友好的生态**
    
    内部版本与滴滴监控系统打通，开源版本与滴滴开源的夜莺监控告警系统打通，集成监控告警、集群部署、集群升级等能力，形成运维生态，凝练专家服务，使运维更高效。 
    

-   **体贴的细节**  
    
     kafka 云平台的建设，它有着自己的设计理念，如：应用、权限、限流等；kafka 集群的 broker 和 topic 的上也存在着各种指标，操作任务，审批流程等，这些都会对用户的使用造成困惑。虽然产品同学在反复的体验和持续的优化迭代，但是为了进一步的方便用户使用，我们在各种用户可能感觉到困惑指标含义的地方，设置了大量的提示说明，让用户不用出页面就可以基本解决问题，整个使用过程力争如丝般顺滑。
    

-   **出众的颜值**
    
    常见的内部工具型产品往往都是以满足功能和需求为主，很多都是功能在页面上的堆砌，kafka 云平台在建设之初，在产品设计和前端开发上也力求更高的标准，最终整体的风格相比以往提升巨大，一上线就被用户称赞为 “大厂出品” ，相比目前几款主流开源的 kafka 管理平台，在页面美观程度上大大超出其他同类产品，可以说是“我花开后百花杀”。
    

_**5.**_ 

**可视化**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

运维人员在日常进行kafka 集群维护和稳定性保障过程中，对集群和 topic 的各项指标都非常关注，因此指标采集展示的准确性和及时性变得非常重要。为此我们将指标的可视化也作为 kafka manager 建设的核心目标。

-   **黄金指标定义**
    

针对 broker 和 topic 日常监控和诊断问题过程中最主要的指标进行筛选，这些指标被定义为黄金指标，如：topic 的 messageIn、byteIn等，并在平台的相关页面上进行高优高亮展示，便于用户在日常使用过程中，快速诊断和定位集群可能存在的问题。同时我们还根据 broker 和 topic 的指标监控，制定了一套用于快速识别其运行状态的健康分算法，将多个指标进行加权计算，最终得到一个健康分数值，健康分的高低则直观的展示了 broker 和 topic 实际运行过程中的状态，便于用户和运维快速从多个 broker 和 topic 中识别。

-   **业务过程数据化**
    

增加基于 topic 生产消费各环节的耗时统计，支持动态开启与关闭，帮助用户自助排查问题；关键指标业务运行过程化，不同分位数性能指标的监控，方便历史问题回溯诊断。

-   **服务端强管控**
    

服务端记录客户端连接版本和类型，连接强感知，集群强管控，元信息的基石；controller 强管控，指定的主机列表，记录 controller 历史运行节点；记录 topic 的压缩格式，应用与业务元信息，业务强感知。

_**6.**_ 

**专家化**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

基于之前全面的kafka数据采集，和运维同学在日常操作过程中的一些经验总结，我们将高频的问题和操作沉淀形成特有的专家服务，来智能诊断 kafka 集群和 topic 的健康状态，并提供对应的处理方案。 

kafka manager 的专家服务能提供了以下功能：

-   **分区热点迁移**
    

1.  **背景：**Kafka只具备Topic迁移的能力，但是不具备自动均衡的能力，Region内部分Broker压力非常大，但是部分Broker压力又很低，高低不均。如果不立即处理，轻则无法继续承接用户的需求，重则由于部分Broker压力过大导致集群出现稳定性问题。
    
2.  **解决方案：**
    

1.  在滴滴 kafka 引擎上增加 topic 在不同磁盘上分布的指标；
    
2.  kafka manager 通过定时采集的 topic 指标来分析 topic在不同磁盘上的分布均匀度；
    
3.  针对不均匀的 topic 发起迁移操作，并在迁移时进行流量控制，保证迁移不会影响集群的稳定性。
    

-   **分区不足扩容**
    

1.  **背景：**kafka 集群的 topic 相关的元信息存储在 zookpeer 上，最终由 controller 将其同步到各个broker。如果元信息过大，controller 同步的压力就会随之上升成为整个集群的瓶颈点，如果遇到某台 broker 出现问题，整个集群的数据同步就非常慢，从而影响稳定性。
    
2.  **解决方案：**
    

1.  在用户申请创建 topic 的时候，平台进行分区数的管控，分区数不能设置的特别大，然而随着 topic 流量的上升，topic 分区可能不足。如果遇到 topic 流量突增，超过了单分区能承载的能力，会导致分区所在的Broker出现压力，如cpu和load升高等问题，客户端也会出现发送堆积的情况。
    
2.  基于现有的机型以及客户端的消费发送能力的压测，我们定义了单分区的承载流量，同时我们实时对每个 topic 的流量进行监控，当某个 topic 的峰值流量持续的达到和超过阈值之后，会对该 topic 进行标记或者告警，定义为分区不足的 topic。
    
3.  针对监控发现出来的分区不足的 topic，由运维人员手动进行扩分区，或者 kafka manager 根据当前集群整个容量情况自动进行扩分区。
    

-   **topic资源治理**
    

1.  **背景：**公共集群中用户申请创建的 topic，它们的使用情况是各不相同的，还存在着部分 topic 根本不使用还占据集群元数据的情况，这对本来就十分拥挤的公共集群造成很大的资源浪费和稳定性风险，因此针对集群中的不使用的 topic进行治理就非常必要。
    
2.  **解决方案:**
    

1.  实时对集群中所有 topic 的流量进行采集监控，智能的分析 topic 的流量趋势，如果发现 topic 的流量持续在一段时间内为0，则进行标记或者告警。
    
2.  针对监控发现的一定周期无流量、无生产消费链接的topic，进行通知和下线清理操作。
    

_**7.**_ 

**同类对比**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们来和外部类似的产品进行一个简要的功能对比如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYaIHEyVJrMNwjicjyL3zxiboTxNf9siaLmM0p3aAGLpyy7XfoqLr3qSjcw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

经过简单的对比，我们可以看到，经过平台化、可视化、智能化、安全建设之后，滴滴kafka manager在安全性、用户体验、监控、运维管控上都有着显著的优势，也提供了很多特色的功能，极大的方便了用户和运维人员的日常使用，欢迎大家Star和体验https://www.didiyun.com/production/logi-KafkaManager.html

_**8.**_ 

**展望**

![图片](https://mmbiz.qpic.cn/mmbiz_png/jE5bOw22iaBtFvdK9icOj3ibAXa8W3tqZ2lQDEaA5XcCDJ5cVIic2221PzXcw0oo69kvia8ojgPZnEV4jPxZURBln4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Logi-Kafka-Manager 已在滴滴内部上线使用，未来我们除了在平台化、可视化、智能化基础上持续改进，还会在以下几个方面持续努力：

-   **平滑的跨集群迁移**
    

我们将基于 MirrorMaker + KafkaGateWay 打造一套 kafka Topic 跨集群平滑迁移能力，在 kafka manager 平台上为 kafka topic 运维管控提供更为高效的迁移能力，进一步提升运维效率。

-   **更多的指标监控**
    

进一步的支持 kafka 2.5+ 最新版本的管控，并且新增更多的关键指标的监控，从而让 kafka manager 指标展示和问题定位的能力更强大。

-   **持续回馈开源社区**
    

作为在滴滴内部经过大量复杂场景验证过的 kafka 管控平台，我们会持续对其进行核心业务抽象，剥离滴滴内部依赖，回馈社区，我们也希望热心的社区同学和我们交流想法，共同提升滴滴 Logi-KafkaManager 的功能和体验

### 

**开源团队**

****▬****

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1wBZCGiaYqBHIwTIY5c03m7flz9BMUhhYiaIHXF8pGdMhm6iaZ94dg3FBvfR7buXWyMlK7cMic16zXUHckoOpllvWw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/jQAw5F0TaSrfDralUuA6ct1sM0fHgUxzNA3kQX5EodyqEmZDqQCdQiaezXLt3t1gVwbDECiaRgWFdVnqe1hfpULg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)