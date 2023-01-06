**前言简述：**

        千呼万唤始出来，随着上一Docker和容器教程系列学习笔记发布，得到了许多B友的支持，于此同时在交流与留言中也指出了我文中某些错误，使得我及时查证，让我收获颇深，也促进了大家共同学习进步，所以笔者我加紧对 云原生最火的 Kubernetes 容器编排系统系列学习笔记，进行了整理发布，希望大家食用开心。

GIF

![](https://i0.hdslb.com/bfs/article/4a10568f28aa362fc3ddc7f871b07daf847767da.gif)

帅哥（靓仔）、美女，点个关注后续不迷路！

**本章目录**

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

0x00 基础简述

-   1.发展经历
    
-   2.简要介绍
    
-   3.系统架构
    

-   Borg 系统
    
-   Kubernetes 系统
    

0x01 组件详述

-   1.Kubernetes-Master
    
-   2.Kubernetes-Node
    
-   3.Kubernetes-插件
    
-   4.本章小结
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

作者: WeiyiGeek

原文地址：https://blog.weiyigeek.top/2020/4-22-468.html

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述:近些年由于Cloud云计算(公有云)以及大数据的发展促进了企业从传统转型到数字信息化再到上云, 其中运维部署应用技术也从物理机转向虚拟化再转向了容器化，而又随着分布架构应用的火热，以及对业务快速迭代的的需要，便推动了如今的Kubernetes分布式架构运维平台，它实现了对容器资源的编排与控制, 这也是本次学习的重中之重;

```
# 公有云类型
Infrastructure as a Service (基础设施及服务) ：阿里云、腾讯云、百度云、京东云、Google Cloud、AWS Cloud
 # 提供给消费者的服务是对所有计算基础设施的利用，包括处理CPU、内存、存储、网络和其它基本的计算资源，用户能够部署和运行任意软件，包括操作系统和应用程序。

Platform as a Service (平台及服务) ： 新浪云(SAE) | Google Cloud 平台(GCP)
 # 提供给消费者的服务是把客户采用提供的开发语言和工具（例如Java，python, .Net等开发环境）开发的或收购的应用程序部署到供应商的云计算基础设施上去。

Software as a Service (软件即服务) : Office 365(云办公) 、 腾讯文档
 # 提供给客户的服务是bai运营商运行在云计算基础设施上的应用程序，用户可以在各种设备上通过客户端界面访问，如浏览器。
```

**部署时代变迁流程**

描述:又回到部署时代变迁流程, 大致来说在部署应用程序的方式上，我们主要经历了三个时代：

-   (1)传统部署时代：早期企业直接将应用程序部署在物理机上。由于物理机上，我们也就很难合理地分配计算资源。例如：如果多个应用程序运行在同一台物理机上，可能发生这样的情况：其中的一个应用程序消耗了大多数的计算资源，导致其他应用程序不能正常运行。应对此问题的一种解决办法是，将每一个应用程序运行在不同的物理机上。然而，这种做法无法大规模实施，因为资源利用率很低，且企业维护更多物理机的成本昂贵。
    
-   (2)虚拟化部署时代：针对上述问题，虚拟化技术应运而生。用户可以在单台物理机的CPU上运行多个虚拟机(Virtual Machine)。
    

-   虚拟化技术使得，限制了应用程序之间的非法访问，进而提供了一定程度的安全性。
    
-   虚拟化技术，可以更容易地安装或更新应用程序，降低了硬件成本，因此可以更好地规模化实施。
    
-   每一个虚拟机可以认为是被虚拟化的物理机之上的一台完整的机器，其中运行了一台机器的所有组件，包括虚拟机自身的操作系统。
    

-   (3)容器化部署时代：容器与虚拟机类似，但是。因此，容器可以认为是轻量级的。
    

-   与虚拟机相似，每个容器拥有自己的文件系统、CPU、内存、进程空间等
    
-   运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦()
    
-   容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署
    

![WeiyiGeek.DeplomentMethodChange](https://i0.hdslb.com/bfs/article/71e7c17e95ff48fe5a6237e8987453787866ece6.png@942w_372h_progressive.webp)

容器因具有许多优势而变得流行起来,这里再老生重谈一下容器化对我们带来的诸多好处:

-   1.敏捷地创建和部署应用程序: 比VM创建容器镜像更快更方便;
    
-   2.持续开发、集成和部署:通过快速、简便的回滚(由于映像不变性)，提供可靠且频繁的容器映像生成和部署。
    
-   3.分离开发和运维的关注点: 降低了开发和运维的耦合度
    
-   4.可监控性: 操作系统级别的资源监控信息以及应用程序健康状态以及其他信号的监控信息
    
-   5.开发、测试、生产不同阶段的环境一致性: 一次build到处运行;
    
-   6.跨云服务商、跨操作系统发行版的可移植性: 在 Ubuntu、RHEL、CoreOS、本地、主要公共云和其他任何位置上运行。
    
-   7.以应用程序为中心的管理:问题的焦点则是在操作系统的逻辑资源上运行一个应用程序,而VM注重于在虚拟硬件上运行一个操作系统
    
-   8.松耦合、分布式、弹性、无约束的微服务:应用程序被切分成更小的、独立的微服务，并可以动态部署和管理，而不是一个部署在专属机器上的庞大的单片应用程序
    
-   9.资源隔离：确保应用程序性能不受干扰
    
-   10.资源利用：资源高效、高密度利用
    

**前世今生**

言归正传，让我们回顾一下 Kubernetesd 的前世今生，在KUbernetes出现前的一些资源管理器。

-   1.Apache MESOS 分布式的资源管理框架 (2019-8 Twitter 宣布弃用MESOS转向Kubernetes) -> 退出舞台
    
-   2.Docker Swarm 针对于 Docker 最薄弱的集群化、容器编排与服务构建 (2019-07 阿里云宣布 Docker Swarm从云平台构建选项中剔除) -> 即将退出舞台
    

-   优点: 系统资源占用少(几十兆) 、支持大规模集群化
    
-   缺点：功能单一滚动更新以及回滚需要运维人员自定义流程费时
    

-   3.Kubernetes 由 Google 基于 Borg (博格)10年的容器化基础架构采用GO语言进行重写发布后迅速在各个企业占有一席之地; -> 下一代分布式架构的王者
    

-   Google 每周运行数十亿个容器，Kubernetes 基于与之相同的原则来设计，能够在不扩张运维团队的情况下进行规模扩展。
    
-   无论是本地测试，还是跨国公司，Kubernetes 的灵活性都能让你在应对复杂系统时得心应手。
    
-   Kubernetes 是开源系统，可以自由地部署在企业内部，私有云、混合云或公有云，让您轻松地做出合适的选择。
    
-   Kubernetes 的生态系统更大、增长更快，有更多的支持、服务和工具可供用户选择,可以将Kubernetes看做为Docker的上层架构。
    

以下是使用 google trends 对比在对于上述的容器编排工具  、 、  三个关键词搜索热度的截图。

![WeiyiGeek.2019](https://i0.hdslb.com/bfs/article/9a7292bf1303f347c76efbbe9fb93d0c21ae624d.png@942w_291h_progressive.webp)

Q:那什么是Kubernetes系统?

> 答: Kubernetes (K8s)是一个用于自动化部署、扩展和管理容器化应用程序的开源系统。  
> 简单的说它就是一个全新基于容器技术的分布式架构方案，Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置()和自动化。Kubernetes 拥有一个庞大且快速增长的生态系统。

参考地址: https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/

**名称含义**  
描述: Kubernetes的名字起源于希腊语，含义是 、、，其logo即像一张渔网又像一个罗盘。  
Google于2014年将Brog系统开源并命名为Kubernetes，它是构建在 十五年运行大规模分布式系统的经验 基础之上，同时凝聚了社区的最佳创意和实践。

> PS深层含义: 既然Docker把自己定位是驮着集装箱在大海遨游的鲸鱼，那么Kubernetes则是掌控大海从而捕获和指引这条鲸鱼按照其设定的路线巡游，从这里可以看出Google对其打造了新一代容器世界的伟大蓝图;

kubernetes 官网: https://kubernetes.io

**Kubernetes 能完成什么工作**

Q: 为什么需要 Kubernetes? 它能做什么?

> 答：容器是打包和运行应用程序的好方式但是免不了容器发生故障，在生产环境中您需要管理运行应用程序的容器并确保不会停机；  
> 例如：当一个容器故障停机，需要另外一个容器需要立刻启动以替补停机的容器。类似的这种对容器的管理动作由系统来执行会更好更快速()。  
> Kubernetes针对此类问题提供了一个可弹性运行分布式系统的框架，可以使你非常健壮地运行分布式系统，它可以处理等多种需求。  
> 例如:Kubernetes 可以轻松管理系统的 Canary 部署。  
> Kubernets 也提供了完善的管理工具涵盖了开发/部署测试/运维监控的各个环节;

Q: Kubernetes 设计理念?  
描述:其功能与架构都遵循了以及微服务的架构，简化开发流程与运维的成本;

Q: 什么是容器编排技术?  
答:容器编排的技术定义是。与此相对应Kubernetes构建了一系列，以持续不断地将系统从当前状态调整到声明的目标状态。

比如: 如何从 A 达到 C，并不重要集中化的控制也就不需要了,就是这样的设计思想使得Kubernetes。

Q: K8s提供特性说明

-   服务发现和负载均衡：通过暴露容器的访问方式,并且可以在同组;
    
-   存储编排:可以自动挂载指定的存储系统，例如  等
    
-   自动发布和回滚: 描述已部署容器的所需状态并将以合适的速率调整容器的实际状态, 可以;
    
-   自愈: 重新启动发生故障、替换容器、kill杀死不满足自定义健康检查条件的容器 ()
    
-   密钥及配置管理: 可以存储和管理敏感信息(例如，密码、OAuth token、ssh密钥等), 您可以更新容器应用程序的密钥、配置等信息,;
    

**Kubernetes 不是什么**

描述：Kubernetes不是一个传统意义的、保罗万象的 PaaS(Platform as a Service)系统。  
它主要在，它提供了与PaaS相似的通用特性例如：等; 然而K8a并不是一个单一整体，这些特性都是可选、可插拔的()

Kubernetes提供用于搭建开发平台的基础模块，同时为用户提供了不同模块的选择性和多样性。

-   1.不限制应用程序的类型: K8s目的广泛支持不同类型的工作负载，包括：等类型的应用,简单的说;
    
-   2.不部署源码、不编译或构建应用程序: 可以作为部署平台参与到 CI/CD 流程，但是不涉及镜像构建和分发的过程 ，即由公司业务及技术要求决定；
    

-   译者注：可选的有 Jenkins / Gitlab Runner / docker registry / harbor 等
    

-   3.不提供应用程序级服务: 包括：中间件(例如，消息总线)、数据处理框架(例如Spark)、数据库(例如，mysql)、缓存(例如，Redis)，或者分布式存储(例如，Ceph)。;
    
-   4.不限定日志、监控、报警的解决方案: k8s提供一些样例展示如何与日志、监控、报警等组件集成，同时提供收集、导出监控度量(metrics)的一套机制，用户可根据自己的需求进行选择;
    

-   译者注：可选的有 ELK(日志) / Graphana() / Pinpoint() / Skywalking ()/ Metrics Server () / Prometheus () 等
    

-   5.不提供或者限定配置语言(例如jsonnet): 提供一组声明式的 API您可以按照自己的方式定义部署信息。
    

-   译者注：可选的有  等
    

-   6.不提供或限定任何机器的配置、维护、管理或自愈的系统。
    

-   译者注：在这个级别上，可选的组件有  等
    

-   7.实际上 Kubernetes , 因为它消除了容器编排的需求。
    

**总结简要，K8s优点与劣势**

-   (1) 优点
    

-   开源(由于东家是Google不用担心项目更新迭代)
    
-   轻量级、消耗资源小
    
-   贴近底层
    
-   弹性收缩
    
-   负载均衡
    

-   (2) 劣势
    

-   前期学习成本高, 体系庞大且冗杂;
    

**Kubernetes 版本号格式**  
Kubernetes 版本号格式遵循 Semantic Versioning 版本控制规则, 版本号格式为 x.y.z，其中 x 为大版本号，y 为小版本号，z 为补丁版本号。

一般 Kubernetes 项目会维护最近的三个小版本分支（例如:2020年11月 -> ）。

-   Kubernetes 1.19 及更高的版本将获得大约1年的补丁支持。
    
-   Kubernetes 1.18 及更早的版本获得大约9个月的补丁支持。
    

提供了有关 kubelet 与控制平面以及其他 Kubernetes 组件之间受支持的版本倾斜的更多信息：https://kubernetes.io/zh/docs/setup/release/version-skew-policy/

Q：如何学习Kubernetes系统?从哪几方面进行入手学习?  
答: 笔者最初学习时候由于K8s知识体系太庞大了导致零零散散的学习了一些基础知识, 但是越学到后面就越吃力，所以又不得重新学习一些基础知识，下面就是本人学习思路:

-   介绍说明：前世今生 Kubernetes基础介绍 Borg / Kubernetes 框架 KUbernetes关键字含义
    
-   基础概念：什么是 Pod 控制器类型 K8S 网络通讯模式
    
-   工具部署：单节点 构建 K8S 集群
    
-   资源清单：资源 掌握资源清单的语法 编写 Pod 掌握 Pod 的生命周期\*\*\*
    
-   Pod 控制器：掌握各种控制器的特点以及使用定义方式
    
-   服务发现：掌握 SVC 原理及其构建方式
    
-   存储：掌握多种存储类型的特点 并且能够在不同环境中选择合适的存储方案(有自己的简介)
    
-   调度器：掌握调度器原理 能够根据要求把Pod 定义到想要的节点运行
    
-   安全：集群的认证 鉴权 访问控制 原理及其流程
    
-   HELM：Linux yum 掌握 HELM 原理 HELM 模板自定义 HELM 部署一些常用插件
    
-   运维：修改Kubeadm 达到证书可用期限为 10年 能够构建高可用的 Kubernetes 集群
    
-   开发： Kubernetes 自开发实现特殊功能
    

Q: k8s适合人群学习研究?  
描述: 项目经理、软件架构师、软件工程师、测试工程师、运维工程师、网络安全工程师以及其它网络技术爱好者;

Q: 学习参考文档?  
答:入门必看文档系列以后看到K8s一律等同于Kubernetes只是方便国人发音;

-   K8S官方文档(推荐):https://kubernetes.io/docs/home/
    
-   Kuboard(国人文档):https://kuboard.cn/learning/
    

描述: 为了更好的了解与学习Kubernets就需借鉴对照Brog系统架构, 看出K8s如何基于Brog系统进行演变更新;

**1) Borg系统架构图如下:**  

**2) 组件简单说明:**

-   BorgMaster : 集群大脑主要负责各组件请求的分发，为了防止其单节点故障, 高可用集群副本数据最好是  奇数个节点;
    
-   Borglet : 工作节点提供各类资源运行对应的容器, 并实时与Paxos进行联系监听，如果有请求取出(消费者)去执行该任务;
    
-   Persistent Store : 简写 Paxos 它是Google的一个键值对数据库;
    
-   Scheduler : 调度器将任务调度的数据写入Persistent Store(Paxos)而不是直接与Borglet节点进行联系;
    
-   访问方式 : BorGCfg(Config File), Command-line Tools, Web Browsers;
    

**1) K8s系统架构图如下:**  

![WeiyiGeek.K8s系统架构图](https://i0.hdslb.com/bfs/article/d4ef56e4f421a56707f94c4456c3226a4ecf07b3.png@942w_437h_progressive.webp)

**2) 组件简单说明:**

-   Kubernetes Master (Control Plane)
    

-   Control Plane Components : 控制平面组件对集群做出全局决策(比如调度)，以及检测和响应集群事件(例如当不满足部署的 replicas 字段时启动新的 pod)
    
-   Kube-api-server : 所有服务请求访问统一入口, 基于HTTP协议进行的C/S架构开发；PS 由于HTTP协议本身支持多种请求方式和验证则没有必要采用TCP协议重写;
    
-   Kube-controller manager : 控制器组件其内部包含多个控制器，并且每个控制器都是一个单独的进程()
    
-   Cloud-controller manager : 云控制器管理器是1.8的alpha 特性在未来发布的版本中将 Kubernetes 与任何其他云集成的最佳方式， 它主要用于可以想做它与 kube-controller-manager 类似;
    
-   kube-scheduler : 调度器负责任务调度选择合适的节点进行分配任务
    
-   Etcd : 它是一个可信赖(自身支持集群化)的分布式(扩容缩方便)键值对存储服务数据库, 它能够为整个K8S分布式集群存储某些关键性数据, 并协助分布式集群的正常运转;
    

-   Kubernetst Nodes
    

-   Kubelet : 通过CRI(Cinatiner/Runtime/Interface)直接跟容器引擎(Docker)交互实现容器的生命周期管理
    
-   Kube-proxy : 负责写入规则至 IPTABLES、IPVS 实现服务映射访问的
    
-   Container Runtime : 容器运行环境是负责运行容器的软件Kubernetes 支持多个容器运行环境: docker、 containerd、CRI-O 以及任何实现 Kubernetes CRI 容器运行环境接口。
    

-   Cloud ：共有云、私有云
    

PS : 不管是K8S的master节点还是Nodes节点都需要依赖容器引擎但不限于docker(主流默认)或者其它的一些容器引擎(podman)

参考地址：https://kubernetes.io/zh/docs/concepts/overview/components/

补充记录时间:

PS : etcd Storage 有  版本, 先K8S集群中使用的是etcd v3版本, v2版本已在K8s v1.11中弃用, 更多信息请参考官网或者博客中;

-   v3 : Database 存储在磁盘便于数据的持久化从而保证数据不会在误操作的情况下丢失
    
-   v2 : Memory 存储在内存存储数据随着机器重启而导致数据丢失
    

![WeiyiGeek.etcd实现架构](https://i0.hdslb.com/bfs/article/a7a2a4df4afd5d253cb5371860a4d9c6abd6a283.png@942w_500h_progressive.webp)

组件说明:

-   HTTP SERVER : 与K8S一样也是采用HTTP协议进行服务请求提交入口
    
-   Raft : 实时读写的信息
    
-   WAL : 预写日志
    
-   Entry : 实体信息
    
-   Snapshot : 快照信息(按照一定的时间将某个时间节点的大版本与其后的增量子版本进行整合备份便于后期数据恢复)
    
-   Store : 将数据存入到本地存储中进行数据的持久化  
    etcd 官方地址：https://etcd.io/docs/v3.4.0/
    

通过上面的简单的K8s组件说明，下面详述了 Kubernetes 的主要组件(该章节了解其组件是了解其K8s架构的基础)  

Q:Control Plane Components控制平面组件的作用说明

> 答:它是集群的控制平台组件()，主要负责集群中的全局决策()和探测并响应集群事件();  
> 控制平面组件可以运行于集群中的任何机器上，但是为了简洁性该组件;

**Master节点下组件的介绍:**

-   1.kube-apiserver : 主要用于提供 用来控制平台的前端可以进行水平扩展(k8S中资源的增删改查操作入口)，比如我们上面提到的  等k8s管理工具基于此实现对 Kubernetes 集群的管理
    
-   2.etcd : 支持一致性和高可用的键值对存储组件，Kubernetes集群的所有配置信息都存储在 etcd 中,所以etcd 数据库通常需要有个备份计划;
    
-   3.kube-scheduler : 主要用于任务调度，此组件监控所有新创建尚未分配到节点上的 Pod，并且自动选择为 Pod 选择一个合适的节点去运行。
    

-   影响调度的因素有：;
    

-   4.kube-controller-manager: 在主节点上运行控制器的组件(资源对象的自动化控制中心)，，但是为了降低复杂度，这些里，该模块包含的控制器有：
    

-   节点控制器(Node Controller):;
    
-   副本控制器(Replication Controller) :;
    
-   端点控制器(Endpoints Controller):;
    
-   服务帐户和令牌控制器(Service Account & Token Controllers): 负责为新的;
    

-   5.cloud-controller-manager: 云控制器管理器是 1.8 的 alpha 特性,这是将 Kubernetes 与任何其他云集成的最佳方式。 云控制器管理器允许您，并将与云平台交互的组件与仅与集群交互的组件分离开来即。与 kube-controller-manager 类似，cloud-controller-manager 将若干逻辑上独立的 控制回路组合到同一个可执行文件中，供你以同一进程的方式运行。 你可以对其执行水平扩容(运行不止一个副本)以提升性能或者增强容错能力。
    
    以下控制器可以有云提供商依赖:
    

```
(1) 节点 (Node) 控制器(Node Controller)：当某一个节点停止响应时，调用云供应商的接口，以检查该节点的虚拟机是否已经被云供应商删除
#译者注：私有化部署Kubernetes时，我们不知道节点的操作系统是否删除，所以在移除节点后，要自行通过 kubectl delete node 将节点对象从 Kubernetes 中删除

(2) 路由 (Router) 控制器 (Route Controller)：在云供应商的基础设施中设定网络路由（`即用于在底层云基础架构中设置路由`）
#译者注：私有化部署Kubernetes时，需要自行规划Kubernetes的拓扑结构，并做好路由配置，例如 安装Kubernetes单Master节点 中所作的

(3) 服务(Service) 控制器 (Service Controller)：`创建、更新、删除`云供应商提供的负载均衡器
#译者注：私有化部署Kubernetes时，不支持 LoadBalancer 类型的 Service，如需要此特性，需要创建 NodePort 类型的 Service，并自行配置负载均衡器

(4) 数据卷 (Volume) 控制器：`创建、绑定、挂载`数据卷，并协调云供应商编排数据卷
#译者注：私有化部署Kubernetes时，需要自行创建和管理存储资源，并通过Kubernetes的存储类、存储卷、数据卷等与之关联
```

-   ccm(简称)使得云供应商的代码和 Kubernetes 的代码可以各自独立的演化，使得K8核心代码不在高度依赖于云供应商的代码的; ccm(简称)与kcm(简称)一样。
    
-   注意在进行K8s集群安装时候可能默认不会安装,通过可以更好地与云供应商结合，例如在阿里云的 Kubernetes 服务里，您可以在云控制台界面上轻松点击鼠标，即可完成 Kubernetes 集群的创建和管理。在私有化部署环境时，您必须自行处理更多的内容。
    

补充说明:  
Q:什么是alpha阶段?  
答:即开发内部测试阶段;

描述：Node 组件运行在每一个节点上(包括 worker 节点或者 master 节点)，。

Node下组件的介绍:

-   1.kubelet: 此组件是运行在每一个集群节点上的代理程序 ，负责对容器的生命周期进行管理并且与Master节点密切卸载实现集群的基本管理工作；
    

-   它通过多种途径获得 PodSpec 定义，并确保 PodSpec 定义中所描述的容器处于运行和健康的状态。
    
-   注意:Kubelet它是不管理。
    

-   2.kube-proxy: 此组件是一个网络代理程序，运行在集群中的每一个节点上，是实现概念的重要部分。
    

-   它维护节点上的网络规则使得您可以在集群内、集群外正确地与 Pod 进行网络通信, 同时它也是负载均衡中最重要的点；
    
-   如果有可用的操作系统包过滤层kube-proxy将使用它，否则kube-proxy将转发流量本身。
    

-   3.容器引擎: 负责运行容器Kubernetes支持多种容器引擎：Docker 、containerd 、cri-o 、rktlet 以及任何实现了 Kubernetes CRI (容器运行环境接口) 的容器引擎
    

补充说明:

Q:什么是引擎?  
答:创建和管理容器的工具，通过读取镜像来生成容器，并负责从仓库拉取镜像或提交镜像到仓库中；

Q:多种容器引擎介绍

-   (1) Docker Engine : 是一种开源容器化技术，用于构建和容器化应用程序()，包含以下组件;  
    

-   (2) Containerd Engine : 是容器运行环境的核心引擎()，可以实现对容器的各种操作(启动，停止等)和网络和存储配置()，它提供了标准化的接口方便各种平台集成，并且还可以将运行环境(Runtime)做成可插拔(Plugable)。
    

前面我们说过在Node节点中Kubelet主要负责Pod的创建、启动、监控、重启、销毁;但它并不是直接面向最终用户, 主要是用于集成到更上层的系统里, 比如 等容器编排系统。

![WeiyiGeek.CRI](https://i0.hdslb.com/bfs/article/7d1c121b383e9464629a97d41f59c1b2b859f20c.png@837w_473h_progressive.webp)

PS : 例如 Docker 是 Kubernetes 社区支持或生态系统中用来在本地计算机上设置 Kubernetes 集群的一种工具。

描述：插件使用 Kubernetes 资源（DaemonSet、 Deployment等）实现集群功能。 因为这些插件，插件中命名空间域的。  
下面描述了一些经常用到的 addons 更多插件请参考K8s的 Addons List

-   1.CoreDNS: 可以为集群中的SVC创建一个域名IP的对应关系解析，它实际上就是一个 DNS 服务器和环境中的其他 DNS 服务器一起工作，它是实现负载均衡最重要的一环，所有 K8s 集群都应该有 ; 在容器启动时为 Kubernetes 服务提供 DNS 记录,即自动将该 DNS 服务器加入到容器的 DNS 搜索列表中；
    
-   2.Web UI(Dashboard):是一个Kubernetes集群的 Web ()管理界面。它使用户通过该界面管理集群中运行的应用程序以及集群本身并进行故障排除。。
    
-   3.Kuboard:是一款基于Kubernetes的微服务管理界面，相较于 Dashboard 来说Kuboard的特点 ：
    
-   4.Container Resource Monitoring(容器资源监控): 将容器的度量指标(metrics)记录在时间序列数据库中，并提供了 UI 界面查看这些数据;
    
-   5.Cluster-level Logging(集群级别的日志记录): 机制负责将容器的日志存储到一个统一存储中，并提供搜索浏览的界面;
    
-   6.Prometheus: 提供K8S集群的监控能力，将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面。
    
-   7.ELK / EFK : 提供 K8S 集群日志统一分析接入平台，负责将容器的日志数据 保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口。
    
-   8.Federation ：提供一个可以跨集群中心多K8S统一管理功能
    
-   9.Ingress: 官方只能实现四层代理,但是他可以实现七层代理，例如等前端代理;
    

Q:什么是Cluster DNS  
答:() 是一个 DNS 服务器，是对您已有环境中其他 DNS 服务器的一个补充，存放了 Kubernetes Service 的 DNS 记录。

描述: 前面我们说了k8s能够对容器化软件进行部署管理，在不停机的前提下, Kubernetes是;

简单的说: 如果项目需要多机器节点的微服务架构，并且采用，那么k8s可以帮助我们屏蔽掉集群的复杂性，自动选择最优资源分配方式进行部署;

下图描述的是拥有一个Master(主)节点和六个Worker(工作)节点的k8s集群, 可以通平面化查看其K8s组件展示；

-   : 负责管理集群以及协调集群中的所有活动
    

-   运行着集群管理的一组进程:，其负责Pod调度,弹性收缩，以及应用程序安全控制，维护应用程序的状态，扩展和更新应用程序。
    

-   : 即图中的Node是VM(虚拟机)或物理计算机，作为集群中的工作节点运行着真正的应用程序，简单的说它就是充当k8s集群中的工作计算机。
    

-   每个Worker节点都运行着一组进程: ,他们负责Pod创建、启动监控、重启、销毁以及实现应用的负载均衡；
    
-   每个Worker节点还安装了用于处理容器操作的工具: 例如Docker或者Container.io;
    
-   工作负载是在 Kubernetes 上运行的应用程序。  
    

![WeiyiGeek.简单集群](https://i0.hdslb.com/bfs/article/19b2d54c197908f4d050bb04ed14d67e568a5272.png@942w_371h_progressive.webp)

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

原文地址：https://blog.weiyigeek.top/2019/9-1-121.html

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

本文至此完毕，更多技术文章，尽情期待下一章节！

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

**已发布的相关历史文章（点击即可进入）**

[1.我在B站学云原生之Docker容器技术基础知识介绍](https://www.bilibili.com/read/cv15180540)

[2.我在B站学云原生之Docker容器环境安装实践](https://www.bilibili.com/read/cv15181036)

[3.我在B站学云原生之Docker容器三大核心概念介绍](https://www.bilibili.com/read/cv15181760)

[4.我在B站学云原生之Docker容器数据持久化介绍与实践](https://www.bilibili.com/read/cv15182308)

[5.我在B站学云原生之Docker容器网络介绍与实践](https://www.bilibili.com/read/cv15185166)

[6.我在B站学云原生之Docker容器Registry私有镜像仓库搭建实践](https://www.bilibili.com/read/cv15219863)

[7.我在B站学云原生之Docker容器Dockerfile镜像构建浅析与实践](https://www.bilibili.com/read/cv15220707)

[8.我在B站学云原生之Docker容器镜像构建最佳实践浅析](https://www.bilibili.com/read/cv15220861)

[9.我在B站学云原生之Docker容器优化镜像体积缩小技巧实践](https://www.bilibili.com/read/cv15226873)

[10.我在B站学云原生之Docker容器技术进阶知识介绍](https://www.bilibili.com/read/cv15227279)

[11.我在B站学云原生之Docker容器编排工具docker-compose安装使用实践](https://www.bilibili.com/read/cv15227639)

[12.我在B站学云原生之Docker容器底层原理浅析](https://www.bilibili.com/read/cv15228563)

[13.我在B站学云原生之Docker容器镜像构建存储原理浅析与实践](https://www.bilibili.com/read/cv15229214)

[14.我在B站学云原生之Docker容器Registry私有镜像仓库安全配置与GC回收实践](https://www.bilibili.com/read/cv15237911)

[15.我在B站学云原生之Docker镜像安全最佳实践](https://www.bilibili.com/read/cv15553799)

[16.我在B站学云原生之Docker容器安全最佳实践](https://www.bilibili.com/read/cv15554240)

[17.我在B站学云原生之Docker容器相关辅助工具使用介绍](https://www.bilibili.com/read/cv15669979)

[18.我在B站学云原生之Docker容器安装运行所遇异常问题解决集合](https://www.bilibili.com/read/cv15670482)

[19.我在B站学云原生之Harbor企业级私有镜像存储仓库入门实践](https://www.bilibili.com/read/cv15720450)

[20.我在B站学云原生之如何使用Skopeo工具做一个优雅的镜像搬运工](https://www.bilibili.com/read/cv15720705)

[1.我在B站学云原生之快速拥抱下一代容器引擎Podman来替代Docker容器](https://www.bilibili.com/read/cv15723446)

[2.我在B站学云原生之快速拥抱下一代容器引擎Podman常用命令浅析与简单配置](https://www.bilibili.com/read/cv15723670)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】或者 个人公众号【WeiyiGeek】联系我。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源于【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 个人博客。

**个人主页:** 【 https://weiyigeek.top 】  

**博客地址:** 【 https://blog.weiyigeek.top 】

![](https://i0.hdslb.com/bfs/article/813dfe5de3d7e2dc0ff2ce8c38ec097255fe26b1.png@942w_480h_progressive.webp)

https://weiyigeek.top - Always keep a beginner's mind, don't forget the beginner's mind

专栏书写不易，如果您觉得这个专栏还不错的，请给这篇专栏【点个赞、投个币、收个藏、关个注，转个发、留个言】，这将对我的肯定，谢谢！。

-   echo  "【点个赞】，动动你那粗壮的拇指或者芊芊玉手，亲！"
    
-   printf("%s", "【投个币】，万水千山总是情，投个硬币行不行，亲！")
    
-   fmt.Printf("【收个藏】，阅后即焚不吃灰，亲！")
    
-   System.out.println("【关个注】，后续浏览查看不迷路哟，亲！")
    
-   console.info("【转个发】，让更多的志同道合的朋友一起学习交流，亲！")
    
-   cout << "【留个言】，文章写得好不好、有没有错误，一定要留言哟，亲! " << endl;
    

GIF

![](https://i0.hdslb.com/bfs/article/11a629d1bc4369dc810216c5dedac871136167d7.gif@1s.webp)

谢谢，各位帅哥、美女四连支持！！这就是我的动力！

 更多网络安全、系统运维、应用开发、全栈文章，尽在【个人博客 - https://blog.weiyigeek.top】站点，谢谢支持！