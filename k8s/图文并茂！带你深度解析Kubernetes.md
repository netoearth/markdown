导语 | 在[[k8s/云原生]]技术发展的浪潮之中，Kubernetes作为容器编排领域的事实标准和[[k8s/云原生]]领域的关键项目，其诞生与完善有着对应的技术历史背景，了解这个过程，对于系统的理解Kubernetes的核心思想、架构设计、实现原理等会很有帮助。  

在云原生技术发展的浪潮之中，Kubernetes伴随着容器技术的发展，成为了目前云时代的操作系统。Kubernetes作为容器编排领域的事实标准和云原生领域的关键项目，已经是[[k8s/云原生]]时代工程师最需要理解与实践的核心技术。

但技术的发展从来都不是一蹴而就，Kubernetes的诞生与完善也有其对应的技术历史背景，了解其诞生与发展的过程，对于更加系统的理解其核心思想、架构设计、实现原理等内容会大有帮助。因此，本文从Kubernetes的诞生背景与Why Kubernetes两个方面，来完成对Kubernetes的概述。

**一、Kubernetes诞生背景**

如果要了解Kubernetes的诞生，就绕不开整个云计算的发展历程。了解了云计算的发展的过程，就会明白，Kubernetes是云计算发展到一定程度的必然产物。

### （一）云计算发展历程

云计算发展历程的时间轴如下图所示，从物理机过渡到传统的IaaS阶段，进而发展为早期的PaaS，直至发展到如今的基于Kubernetes架构的新兴PaaS平台。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

用户使用资源的形态也由早期的物理机过渡到虚拟机，再进化到目前更轻量的Docker容器。本质上云计算实现的关键突破就在于资源使用方式的改变，其最初解决的核心的问题就是应用的托管即应用部署与管理问题。

### （二）早期物理机时代

云计算之前，开发者如需部署管理服务，需要根据需求，进行配置、管理与运维物理机。整体上维护困难，成本高昂，重复劳动，风险随机。以至于当年流传着运维的传统艺能：上线拜祖，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4AxQHnvVSlyFXZqCkicmmZ2LzLibcMlDolvCpJY5SIQmb34eY7lflG5Ug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在那个时代，应用部署与管理面临着以下诸多问题：

-   硬件、机房等维护成本高。各个团队独立搭建机群、运维机器。
    

-   应用部署、迁移、修复困难。缺少统一的部署发布平台；面对突发情况，缺少自动化工具，排查解决问题依赖人工，低效且成本巨大。
    

-   资源利用率低。物理机的平均资源率不到10%，有的甚至在5%左右，造成了资源的巨大浪费。
    

-   应用隔离性差。多业务混部在一台机器时，会产生干扰。例如：当某一应用资源使用率突然提升，会抢占其他应用的可用资源。
    

### （三）IaaS平台

Infrastructure as a service (IaaS) 基础设施即服务，用户可以按需去申请基础设施资源（包括：计算、存储、网络）。

IaaS商业化道路上的一个标志性事件：2006年AWS推出了EC2（亚马逊弹性云端运算），其基于Xen虚拟化技术，用户可以在web界面上配置、获取虚拟机资源，部署应用。通过规模化来降低边际成本。

-   #### 虚拟化技术
    

IaaS的底层核心技术是虚拟化技术。虚拟化技术是一种资源关联技术，是将计算机的各种实体资源，如服务器、网络、存储等，进行抽象、整合、管理与再分配的一种技术。最常用的一种方案是基于虚拟机（Hypervisor-based）的虚拟化实现。其通过一个软件层的封装，提供和物理硬件相同的输入输出表现，实现了操作系统和计算机硬件的解耦，将OS和计算机从一对一转变为多对多（实际上是一对多）的关系。该软件层称为虚拟机管理器（VMM/Hypervisor），它分为两大类：裸金属架构、宿主机架构。

裸金属架构：VMM直接安装和运行在物理机上；VMM自带虚拟内核的管理和使用底层的硬件资源。业界的Xen、VMWare ESX都是裸金属架构。

宿主机架构：物理机上首先会装一个操作系统，VMM安装和运行在操作系统上；在VMM再去装其他虚拟机操作系统，依赖与操作系统对硬件设备的支持与资源的管理。这种架构的好处是，VMM会变得非常简单，因为可以基于操作系统去管理系统资源，VMM只需要做额外的虚拟化工作。Oracle VirtualBox，VMWare Workstation、KVM都是这种架构，宿主机架构是目前虚拟化技术的主流架构。

下图中，对比了物理机架构与宿主机虚拟化架构的区别。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4Agfrvib6KempW1TibD1qWwHiaFia5ImCfedqFhTYuhkRibqcKhXS41eAaGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

虚拟化架构有如下的优势：

-   封装应用技术栈。虚拟化镜像中可以预装一些通用的软件与库，来减少应用对通用软件与库的依赖。
    

-   提高底层资源的隔离性。硬件层面做了隔离，虚拟机之间互不干扰。
    

-   动态调整机器、资源配置。虚拟化的配置可以动态升降配，用户可以按自己的需求调整。
    

-   提高资源利用率。资源使用率从平均不到10%提高到了15%左右。
    

-   #### OpenStack
    

当物理机转变为虚拟机之后，如何对多台虚拟机的资源进行管理与调度，成为了一个新的问题。

OpenStack给出了解决方案，它是一个开源的分布式的平台，能够统一管理多个服务器，按用户需求进行分配与调度虚拟机。其本质上是一组分配、管理虚拟机的自动化工具脚本。

目前，OpenStack已经发展成了IaaS的主流解决方案，即：OpenStack as IaaS。目前主流IaaS云服务厂商底层都是利用OpenStack技术。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4iaUzQicWFDARuCgyYw6ycrpNCLZBFfD7r39BwibDdvlI23GAtWoq1lH0w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

IaaS平台一定程度上提升了物理机的资源利用率，由物理机时代的低于10%，提升到了15%。但虚拟机对资源利用率的提升仍存在一定的局限性，其相对笨重，启动慢，自身消耗大（其完整运行了一套操作系统），自身加载就要消耗几百兆的内存资源。此外，虚拟机可以预装一些软件，一定程度简化了应用程序的依赖安装。但应用程序的部署与打包，仍然需要开发人员各自解决，仍未高效的完成应用部署与分发。

### （四）PaaS平台

Platform-as-a-Service (PaaS)平台即服务。PaaS提供了包括服务器、存储空间和网络等基础结构，但它并未包括中间件、开发工具、数据库管理系统等。PaaS旨在支持应用程序的完整生命周期：生成、部署、管理和更新，提供应用的托管能力。

在IaaS阶段，服务厂商只提供虚拟机，虚拟机之上的软件栈都由用户管理，包括操作系统、持久化层、中间层、用户程序。在IaaS层面用户只是减少了关心底层硬件，而PaaS层面希望能够进一步解放用户，让用户真正只需关注应用本身。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4VUZ01gUSynzOXCaNP6lGI8uQFFe6QsybOuMu7Agcib4nDTrQsQuHLqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

-   #### PaaS主要功能
    

目前一个成熟的PaaS平台应具备的主要功能，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4zuGWEp4NHxIGliaiaib9tT3Y4CTRr93C7ibia1MribhNdn4KBjcOI0WM7fFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

早期PaaS平台，更多关注运行时环境与依赖服务，而目前的PaaS平台新增大量的支持服务，包括：认证授权、系统日志、应用监控等，以上都是应用开发的常见需求。原则上：共用内容就应该抽象出统一通用的组件，由框架和平台来实现。让用户只关心逻辑或应用本身，避免重复造轮子。

-   #### PaaS早期代表Cloud Foundry
    

PaaS在成熟之前也经历了几个阶段，而PaaS早期的代表就不得不提Cloud Foundry。Cloud Foundry由VMWare开发，是第一款开源PaaS平台（2011年）。支持多种框架、语言、运行时环境、云平台及应用服务，使开发人员能够快速进行应用的部署，无需担心任何基础架构的问题。

它主要功能包括以下：

-   应用打包、分发部署
    

-   以容器的方式运行应用
    

-   均衡负载
    

-   服务监控、异常重启
    

Cloud Foundry的出现，其描绘了PaaS平台的初步形态，推动了PaaS的发展，具有划时代的意义。

但其最终并未成为PaaS主流，是因为其存在一个核心不足：它只对应用和配置进行了打包，而没有打包整体依赖（所谓的整体依赖包括：中间环境、操作系统文件）。所以它的包在跨平台运行时，会出现运行失败的现象。这个问题非常致命。

而且，早期Cloud Foundry主要是针对单一Web应用的管理，对分布式应用所需的各项能力均未涉及，例如：服务发现、弹性扩缩等。

### （五）Docker

Docker公司的前身是dotCloud，它是2010年成立，提供Paas服务的平台。但当时Cloud Foundry做的相对完善和开放，2012年底dotClound濒临倒闭，创始人决定把内部的打包平台开源出去。因此，2013年3月dotCloud公司在github平台上开源了其内部的容器项目Docker。Github开源之后，受到了业界的热烈追捧，从而Docker大火。公司后来也改名为Docker。

Docker的成功，主要是通过镜像完美解决了开发、测试、生产环境不一致的问题。它的口号是：Build、Shipand Run any App、Anywhere，即一处构建，到处运行。

Docker的核心技术有三个：NameSpaces做视图隔离、Cgroups做资源限制，UnionFS联合文件系统，统一mount。通俗理解：NameSpaces、Cgroups通给进程设置属性，实现进程的隔离与限制，UnionFS给进程构造文件系统。这三项技术均有linux内核提供，Docker本身并没有创造新的技术。

但是Docker创造性的通过镜像整体打包了应用的依赖环境，包括：操作系统文件、中间依赖层、APP。

-   整体打包之后，镜像变大，又该如何优化？
    

Docker通过镜像分层复用的方式进行了优化。共用只读层，节省存储空间，提高镜像推送、拉取效率，镜像的操作是增量式。

-   当分层之后，在宿主机上如何合并多个层？
    

利用UnionFS实现合并，多个只读层加一个可写层mount成一个目录。并且上面的层会覆盖下面的层，当对底层的只读层修改时会采用写时复制策略（copy-on-write）。写时复制的含义：当另一个层第一次需要写入该文件时（在构建镜像或运行容器时），该文件会被复制到该读写层并被修改。该机制大大减少了容器的启动时间（启动时新建的可写层只有很少的文件写入），但容器运行后每次第一次修改的文件都需要先将整个文件复制到container layer中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw44cj00RvLWbHwbp4evr4ojkwI1WnHkBxKoLuMYjpsmlWRWkWBicJficrw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下图所示，Docker相比于虚拟机操作系统级的资源隔离，实现了进程级资源隔离，极大提升了资源利用率。具备以下特点：

-   进程级隔离，更轻量
    

-   更低消耗系统资源
    

-   更快速启动
    

-   更快交付与部署
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4stG2iaHquoUO068icLObZ4iarluPYia4cSu902msKiajO8zX4OjutoXMqcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### （六）容器编排

当Docker解决了应用打包的问题后，PaaS上应用大规模部署与管理的问题愈发突出。此时，业内明白：容器本身没有“价值”，有价值的是容器编排。

容器编排(Orchestration)：对Docker及容器进行更高级更灵活的管理，按照用户的意愿和整个系统的规则，完全自动化的处理好容器之间的各种关系（对象之间的关系远重要于对象本身）。

容器技术做为底层基础技术，只能用来创建和启动容器的小工具，最终只能充当平台项目的“幕后英雄”。用户最终部署的还是他们的网站、服务、数据库，甚至是云计算业务。这就需要一个真正的PaaS平台，让用户把自己的容器应用部署在此之上。

在以上的历史背景之下，2014年左右，Docker、Mesos、Google相继发布自己的PaaS平台，容器编排之争正式开始。

Docker发布了Swarm平台，Swarm擅长跟Docker生态无缝集成，docker用户可以低成本过渡。其最大亮点是使用Docker项目原有的容器管理API来完成集群管理。例如：单机Docker项目: docker run “我的容器”。集群Docker项目：docker run-H“我的Swarm集群API地址” “我的容器”。

Mesos平台，擅长大规模集群的调度与管理。它是Apache基金会下的一个开源集群管理器，最初是由Berkeley分校开发的。它为应用程序提供了跨集群的资源管理和调度API。之后转向支持PaaS业务，推出了Marathon项目。它是一个高度成熟的PaaS项目，旨在让用户便捷管理一个数万级别的物理机集群，可使用容器在这个集群里自由部署应用。

Google推出的是Kubernetes平台，整个系统的前身是Borg系统，Kubernetes平台是Google在容器化基础设施领域十多年来实践经验的沉淀与升华。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

经过近3年的角逐，容器编排之争的胜利者是Kubernetes。  

-   2017年9月，Mesos宣布支持Kubernetes。
    

-   2017年10月，Docker官方支持Kubernetes。
    

-   2018年3月，Kubernetes正式从CNCF毕业，开始一统江湖。（所谓毕业是指这个产品可以直接使用在生产环境）
    

-   目前，Kubernetes已经成为容器编排领域的事实标准。
    

Kubernetes，读者一定会有一个疑问：为什么最后是Kubernetes？

每个人对这个问题，都有一些自己的理解，本文从技术方面对该问题进行了阐述。

**二、Why Kubernetes**

Kubernetes源于希腊语，意为“舵手”。k8s缩写是因为k和s之间有八个字符的原因。它是google在2015开源的容器调度编排的平台。它是建立在Google大规模运行生产工作负载（Borg系统）十几年经验的基础上， 结合了社区中最优秀的想法和实践，已经成为了目前容器编排的事实标准。

其实看到Docker和Kubernetes的Logo，就可以很快明白Kubernetes的作用。Docker的Logo是一条鲸鱼船，运载着许多封装好的集装箱(container)，代表着一次打包到处运行的意图。而Kubernetes的Logo就是这条船的方向舵！

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

对于Why Kubernetes？很多人都有自己的理解，接下来笔者从技术的角度，阐述一下自己的观点。Kubernetes技术上的成功，个人认为核心在于三个关键点：

-   成熟的技术前身
    

-   优秀的框架架构
    

-   良好的核心设计
    

### （一）Kubernetes前身

Kubernetes的基础特性，并不是几个工程师突然“拍脑袋”想出来的东西，而是 Google 公司在容器化基础设施领域多年来实践经验的沉淀与升华。这个实践与升华的过程，就是Kubernetes的前身是Borg系统。

Borg系统一直以来都被誉为Google内部最强大的“秘密武器”，是Google整个基础设施的核心依赖。很多应用框架已经运行在Borg上多年，其中包括了内部的MapReduce、GFS、BigTable、Megastore等，上层应用程序更是有这些耳熟能详的产品：Gmail、Google Docs、Google Search等。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

其架构图如下所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

架构分析：

-   集群分为Master节点与Worker节点。
    

-   Master节点由多台机器构成，一主多备。
    

-   BorgMaster由主进程和scheduler进程组成，主进程处理clientRpc请求，scheduler负责调度tasks。
    

-   Borglet是Worker节点上的代理进程，用于启停tasks。
    

根据2015年4月google发布的Large-scale clustermanagement at Google with Borg，与其2020年7月发布的Borg: the nextgeneration，两篇论文中的数据表明：Borg系统通过对在线任务与离线任务进行混合部署，可以节约20%-30%的资源，极大提高了资源利用率。下表是2011年与2019年的Borg集群，与2015年AWS、Facebool、Twitter数据中心资源利用率的对比图。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

对于成熟高效的Borg系统，继承者Kubernetes从中获得了宝贵的经验：

-   Pods。Pod是Kubernetes中调度的单位。它是一个或多个容器在其中运行的资源封装。保证属于同一 Pod的容器可以一起调度到同一台计算机上，并且可以通过本地卷共享状态。Borg有一个类似的抽象，称为alloc（“资源分配”的缩写）。
    

-   Service。Kubernetes使用服务抽象支持命名和负载均衡：带名字的服务，会映射到由标签选择器定义的一组动态Pod集。集群中的任何容器都可以使用服务名访问服务。
    

-   Labels。通过使用标签组织Pod，Kubernetes比Borg支持更灵活的集合，标签是用户附加到Pod（实际上是系统中的任何对象）的任意键值对。
    

-   Ip-per-Pod。Borg容器只能共享主机网络，必须将端口作为调度的资源。在Kubernetes中IP是以Pod为单位分配的，一个Pod内部的所有容器共享一个网络堆栈。
    

### （二）Kubernetes架构

-   #### 整体架构
    

Kubernets整体架构，如下所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

整个系统由控制面(Master)与数据面(Worker Node)组成。Master核心组件：

-   API Server。集群控制的唯一入口，它是各个组件通信的中心枢纽。
    

-   controller-mananger。负责编排，用于调节系统状态。内置了多种控制器（DeploymentController、- ServiceController、NodeController、HPAController等）是Kubernetes维护业务和集群状态的最核心组件。
    

-   scheduler。集群的调度器，它负责在Kubernetes集群中为Pod资源对象找到合适节点并使其在该节点上运行。
    

-   etcd。用于存储Kubernetes集群的数据与状态信息。
    

Kubernetes架构具备高可用：一方面Master节点高可用；另一方面所部署的业务也是高可用的。系统高可用的核心在于冗余部署，当某一个节点或程序出现异常时，其他节点或程序能分担或替换工作。Master节点高可用，主要由以下几个方面的设计实现：

-   Master由多台服务器构成。
    

-   API Server多实例同时工作，负载均衡。
    

-   etcd多节点，一主多从。
    

-   controller-manager与scheduler抢主实现。
    

Work Node节点由以下组件组成：

-   kubelet：负责Pod对应容器的创建、启停等任务，是部署在Node上的一个agent。
    

-   kube-proxy：实现Service通信与负载均衡机制。
    

-   容器运行时(如Docker)：负责本机的容器创建和管理。
    

-   #### API Server中心枢纽
    

Kubernetes中API Server的核心功能是提供Kubernetes各类资源对象（如Pod、RC、Service等）的增、删、改、查及Watch等HTTP REST接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。除此之外，它还是集群管理的API入口，提供了完备的集群安全机制。API Server是由多实例同时工作，各个组件通过负载均衡连到具体的API Server实例上。

如下所示，各组件与API Server通信时，采用List-Watch机制，通过API server获取etcd配置与状态信息，进而触发行为。以下图为例是kubectl创建一个deployment时，各个组件与API Server的流程交互。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Api Server的作用：

-   集群控制、访问的唯一入口，统一的认证、流量控制、鉴权等。
    

-   ectd数据的缓存层，请求不会轻易穿透到etcd。
    

-   集群中各个模块的中心枢纽，各个模块之间解耦。
    

-   便于模块插件的扩展（其他模块List、Watch、Update ApiServer即可实现扩展功能）。
    

### （三）Kubernetes核心设计

Kubernetes取的巨大的成功，与它良好的核心设计紧密相关。笔者认为Kubernetes有三大核心设计：

-   编排抽象。容器平台核心点不在于创建和调度容器，而是在上层架构抽象出各种对象，便于去统一管理。Kubernetes创造性的抽象出了各个编排的关系，例如亲密关系(Pod对象)、访问关系（Service对象）等。
    

-   声明式API。声明式API是整个系统自动化的核心要点，kubernetes提供了以声明式API的方式将抽象对外暴露，同时也便于了用户管理对象。
    

-   开放插件。支持系统资源插件化（比如计算、存储、网络）；同时也支持用户自定义CRD和开发Operator。
    

-   #### Pod对象
    

Kubernetes在对象抽象方面，核心创新在于Pod对象的设计。容器设计本身是一种“单进程”模型。该表述不是指容器里只能启动一个进程，而是指容器无法管理多个进程。只有容器内PID=1的进程生命周期才受到容器管理（该进程退出后，容器也会退出），其他进程都是PID=1的进程的子进程。根据容器设计模式，传统架构中多个紧密配合的业务进程（例如业务进程与日志收集进程，业务进程与业务网络代理进程）应该部署成多个容器。但这些容器之间存在亲密的关系，需要一起调度和直接共享某些资源（网络和存储）。

Kubernetes抽象出一个Pod对象，是一组（一个或多个）容器， 这些容器共享存储、网络等， 这些容器是相对紧密的耦合在一起的。Pod是Kubernetes内创建和管理的最小可调度单元，调度过程是按Pod整体所需资源一起进行调度的。Pod本身只是逻辑上的概念，在容器管理这层并不认识Pod对象。

Pod的实现需要使用一个中间容器（Infra容器），在这个Pod中，Infra容器永远是第一个被创建的容器，用户定义的其他容器通过Join Network Namespace的方式与Infra容器关联在一起。抽象一个中间容器的原因在于各个业务容器是对等的，其启动没有严格的先后顺序，需借助中间容器实现共享网络和存储的目的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4HhJKmUfCMCcs9JTiajYNAFvjGbm9NbW5Fk29LovG2DRBSzmArTPSMibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中，Node、Pod与容器三者关系，如下图所示。Node表示一台机器，可调度多个Pod，而一个Pod内又能包含多个容器。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4j4n2gkvagCh83vENI5Fm1VBFPw92wFpkjutBA86xNMpPnnrLf2Jmfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

至此，再来通过Kubernetes中各个对象的关联关系来更为深刻的理解Pod的意义。下图可以看出，Pod其实是整个编排过程中操作的核心，很多对象直接或间接的同Pod相关联。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4at4Sict5HZSPkXl1NhGLxTyLuAiaZ1ztx07YX5wQlibLaRnFFotTCIxVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

-   #### Service对象
    

Kubernetes编排抽象的另一个核心对象是Service对象，它统一的解决了集群内服务发现与负载均衡。Service是对一组提供相同功能的Pod的抽象，为其提供了一个统一的入口。Service通过标签选择服务后端，匹配标签的Pod IP和端口列表组成endpoints，由kube-proxy负责将请求负载到相关的endpoints。

下图是kube-proxy通过iptables模式来实现Service的过程，Service对象有一个虚拟clusterIP，集群内请求访问clusterIP时，会由iptables规则负载均衡到后端endpoints。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

-   #### 声明式API
    

Declarative（声明式设计）指的是一种软件设计理念和编程方式，描述了目标状态，由工具自行判断当前状态并执行相关操作至目标状态。声明式强调What，目标是什么。而Imperative(命令式)需要用户描述一系列详细指令来达到期望的目标状态。命令式强调How，具体如何做。

下图描绘了一个场景：目标副本数为3。对于声明式而言，用户设定目标为3，系统获取当前副本数为2，系统判定当前值与目标值的差为1，便自行加1，最终实现副本数为3的目标状态。而对于命令式，需用户判断当前副本数为2，用户给出指令副本+1，系统接收用户指令，执行副本数+1操作，最终系统副本数为3。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

kubernetes的一大核心设计就是采用了声明式API，利用该设计思想有效的实现了系统的自动化运行。Kubernetes声明式API指定了集群期望的运行状态，集群控制器会通过List&Watch机制来获取当前状态，并根据当前状态自动执行相应的操作至目标状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4JlgreOUk9TUJJqgxJx1B5VOeib5ur9C5Rf0vDJljjV6ODLaVZrAwDgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Kubernetes中，用户通过提交定义好的API对象来声明期望状态，系统允许有多个API写端，以PATCH方式对API对象进行修改。Kubectl工具支持三种对象管理方式：命令式命令行、命令式对象配置(yaml)、声明式对象配置(yaml)。举例如下：命令式命令行：

```
kubectl run nginx –image nginx
```

命令式对象配置：

```
kubectl create –f nginx.yaml
```

以上先kubectl create再kubectl replace的操作，与命令式命令行不存在本质区别。只是把具体命令写入yaml配置文件中而已。声明式对象配置：

```
kubectl apply –f nginx.yaml
```

Kubernetes推荐使用：声明式对象配置(YAML)。kubectl replace执行过程是通过新的YAML文件中的API对象来替换原有的API对象，而Kubectl apply执行了一个对原有API对象的PATCH操作。除此之外，YAML配置文件用于Kubernetes对象的定义，还会有以下收益：

-   便捷性：不必添加大量的参数到命令行中执行命令。
    

-   灵活性：YAML可以创建比命令行更加复杂的结构。
    

-   可维护性：YAML文件可以通过源头控制，跟踪每次操作；并且对象配置可以存储在源控制系统中（比如Git）；对象配置同时也提供了用于创建对象的模板。
    

-   #### 开放插件
    

Kubernetes的设计初衷就是支持可插拔的架构，解决PaaS平台使用不方便、不易扩展等问题。为了便于系统的扩展，Kubernetes中开放了以下接口可对系统资源（计算、网络、存储）插件进行扩展，可分别对接不同的后端来实现自己的业务逻辑。

CRI（Container Runtime Interface）：容器运行时接口，提供计算资源。CRI接口设计的一个重要原则是只关注接口本身，而不关心具体实现，kubelet就只需要跟这个接口打交道。而作为具体的容器项目，比如Docker、rkt、containerd、kata container它们就只需要自己提供一个该接口的实现，然后对kubelet暴露出gRPC服务即可。简单来说，CRI主要作用就是实现了kubelet和容器运行时之间的解耦。  

CNI（Container Network Interface）：容器网络接口，提供网络资源。跨主机容器间的网络互通已经成为基本要求，K8S网络模型要求所有容器都可以直接使用IP地址与其他容器通信，而无需使用NAT；所有宿主机都可以直接使用IP地址与所有容器通信，而无需使用NAT。反之亦然。容器自己的IP地址，和别人（宿主机或者容器）看到的地址是完全一样的。K8S提供了一个插件规范，称为容器网络接口(CNI)，只要满足K8S的基本需求，第三方厂商可以随意使用自己的网络栈，通过使用overlay网络来支持多子网或者一些个性化使用场景，所以出现很多优秀的CNI插件被添加到Kubernetes集群中。

CSI（Container Storage Interface）：容器存储接口，提供存储资源。K8S将存储体系抽象出了外部存储组件接口，第三方存储厂商可以发布和部署公开的存储插件，而无需接触Kubernetes核心代码，同时为Kubernetes用户提供了更多的存储选项。例如：AWS、NFS、Ceph。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw4ctffBUIQ0yWcibNQVv6gqX3iag9yPFWTMnKFrds0WufNv23amEibPGIHg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Kubernetes除了对系统资源可插件扩展外，也可以自定义CRD（Custom Resource Definition）来扩展API对象，同时也支持编写Operator对CRD进行控制。例如：对于一些有状态应用（etcd），可以定义新的CRD对象，并编写特定的Operator（本质上是新的controller）去实现控制逻辑。

Kubernetes的调度器Scheduler也是可以扩展的，可以部署自定义的调度器，在整个集群中还可以同时运行多个调度器实例，通过 pod.Spec.schedulerName 来选择具体指定调度器（默认使用内置的调度器）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw40Q3ay0jC3PoHYzC6vcWA0PKkrapfQyKhJks5shR295gBY91fmRxQNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**三、小结**

根据以上两个章节的阐述，对于文章开头的经典问题：如何才能有效的部署与管理应用？到Kubernets大放异彩的今天，已经给出了答案：

-   应用部署与管理的问题，利用Docker+Kubernetes的方式已经完美解决。
    

-   Kubernetes的强大功能还进一步提供了：均衡负载、服务发现、弹性扩缩、自动修复等分布式应用管理的能力。
    

感谢Kubernetes，将开发、运维人员从繁重的应用部署与管理工作中解放出来。到目前为止，Kubernetes已经成为了容器编排的事实标准，是新一代的基于容器技术的PaaS平台的重要底层框架。

Kubernetes的成熟，拉开了轰轰烈烈的云原生技术发展的大幕！

## **参考资料：**

1.Kubernetes 文档

2.Kubernetes权威指南 第五版

3.深入剖析 Kubernetes-张磊

4.Docker与k8s的前世今生

5.https://draveness.me/understanding-kubernetes/

 **作者简介**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/VY8SELNGe96jEyaJCib0qpPqfNTh5WTw462DzOXJzx4UeeLT6HMEqpd1A9ibOIU8EvzOtibJ3m2tEM2px3YSRFC2Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**尹飞**

腾讯后台开发工程师

腾讯后台开发工程师，专注于后台游戏开发，擅长分布式开发，有丰富的C++、Lua、Go语言使用经验。