三个问题，回顾前情提要。

ServiceMesh 解决什么问题？

![Istio，灰度发布从未如此轻松！！！_Istio](https://img.moregeek.xyz/s7/images/blog/202012/25/caeb6770cb572955e3f94cfb7efd5fc6.png)

SM 本质是业务服务与底层技术体系的解耦：

-   一个进程实现业务逻辑（不管是调用方，还是服务提供方），biz，即上图白色方块
    
-   一个进程实现底层技术体系，proxy，即上图蓝色方块
    

画外音：负载均衡、监控告警、服务发现与治理、调用链… 等诸多基础设施，都放到这一层实现。

什么是 Istio？

Istio 是 ServiceMesh 的产品化落地。

Istio 的分层架构设计如何？

![Istio，灰度发布从未如此轻松！！！_Istio_02](https://img.moregeek.xyz/s9/images/blog/202012/25/22eae4c169989ee2a2a82be7916991c4.jpeg)

Istio 采用实施与控制分离的数据平面与控制平面两层架构。

数据平面

-   envoy(proxy)：负责高效转发与策略落地 \[核心\]

控制平面

-   mixer：适配组件，数据平面与控制平面通过它交互
    
-   pilot：策略配置组件 \[核心\]
    
-   citadel：安全组件
    
-   galley：底层平台（例如：K8S）解耦组件
    

整个架构的核心是 envoy 与 pilot。

今天起，聊聊 Istio 的流控，典型如灰度发布。

就如同 ServiceMesh 的设计初衷，是技术体系与业务服务解耦一样，Istio 流控模型的本质，是流量控制与服务实例扩展的解耦，更具体的：

-   用户只需要通过控制平面中的 Pilot 设定期望流量要以什么规则进行路由
    
-   不需要规定服务实例 (service pods) 如何接收
    
-   数据平面 Envoy 将从 Pilot 中获取规则和命令，然后落地各类分流策略
    

![Istio，灰度发布从未如此轻松！！！_Istio_03](https://img.moregeek.xyz/s6/images/blog/202012/25/a1042562f6c1194111953dca348c9155.png)

如上图所示，最开始时，ServiceA 访问旧版的 ServiceB。

画外音，业务与底层解耦：

（1）灰色圆形为业务 Svc 服务；

（2）紫色六边形为 Envoy 代理；

（3）服务与代理之间都是本地访问；

（4）跨网段之间都是 Envoy 代理交互（蓝色箭头）；

如何进行灰度发布呢？

![Istio，灰度发布从未如此轻松！！！_Istio_04](https://img.moregeek.xyz/s8/images/blog/202012/25/62f8ce031d1fe1de7c9d996ec34fda33.png)

如上图所示，服务 A 调用服务 B，服务 B 要发布一个灰度版本，需要 5% 的流量打到服务 B 的新版本，只需要：

（1）部署服务 B 的新版本；

（2）控制平面 Pilot 上进行策略配置，策略同步到 Envoy；

（3）数据平面 Envoy 接收到策略配置，实时分流策略；

画外音：图形上没有画出 Pilot 和 Envoy 的交互。

搞定，这个过程业务服务与流量控制策略完全解耦，完美！

除了基于按流量比例分流的灰度发布，基于应用层的灰度发布通过 Istio 也非常容易实现。

![Istio，灰度发布从未如此轻松！！！_Istio_05](https://img.moregeek.xyz/s5/images/blog/202012/25/92b0cd3588655521df0c39ebde52989c.png)

如上图所示，服务 B 要发布一个灰度版本，需要把 iPhone 的流量打到 B 的新版本，操作流程完全一样（部署服务，Pilot 控制，Envoy 实施），非常方便。

如果 Envoy 原来只支持按照流量比例分流，不支持基于应用层协议分流，此时只需要：

（1）升级 Envoy 的分流策略，以及策略控制端 Pilot；

（2）调用方服务 A 不需要升级；

（3）服务方服务 B 也不需要升级；

业务与底层基础设施完全解耦，完美！

画外音：这是 Service Mesh 的核心理念之一，详见《\_\_ServiceMesh 究竟解决什么问题》。

如果是用传统微服务框架的方式，需要框架升级，调用方与服务方均需要配合升级与重启。

最近下班都比较晚，今天先写到这里。Pilot 的分层架构如何，它又是如何与 Envoy 配合实现流控的，且听下回分解。

思路比结论重要。

调研：

大伙升级一个流控策略，业务服务要升级，要重启么？

上篇： Service Mesh - Istio实战篇（上） 收集指标并监控应用 在可观察性里，指标是最能够从多方面去反映系统运行状况的。因为指标有各种各样，我们可以通过多维数据分析的方式来对系统的各个维度进行一个测量和监控。 Istio 默认是通过自带的 Promethuse 和 Grafana 组件来完成指标的收集和展示，但是监控系统这样的基础工具，通常在每个公司的生产环境上都是必备的，

项目准备和构建过程 典型的 CI/CD 过程 - DevOps GitOps 持续交付过程 GitOps：一种集群管理和应用分发的持续交付方式 GitOps与典型的CI/CD不同，其中最大的不同点在于使用 Git 作为信任源，保存声明式基础架构（declarative infrastructure）和应用程序 以 Git 作为交付过程（pipeline）的中心，配置文件如k8s的yaml文件都

官方文档： Security Concept 守卫网格：配置TLS安全网关 Istio 1.5 的安全更新： SDS （安全发现服务）趋于稳定、默认开启 对等认证和请求认证配置分离 自动 mTLS 从 alpha 变为 beta，默认开启 Node agent 和 Pilot agent 合并， 简化 Pod 安全策略的配置 支持 first-party-jwt （ServiceAccou

ServiceMesh(2)上一篇介绍了《ServiceMesh 究竟解决什么问题？》，当微服务架构体系越来越复杂的时候，需要将 “业务服务” 和“基础设施”解耦，将一个微服务进程一分为二：一个进程实现业务逻辑，biz，即上图白色方块一个进程实现底层技术体系，proxy，即上图蓝色方块，负载均衡、监控告警、服务发现与治理、调用链… 等诸多基础设施，都放到这一层实现如此解耦之后：biz 不管是调用服

What is IstioIstio is a implementation of service mesh. It design to provide La) traffic route option between servicesb) monitor the traffics go through different serivce  for different metrics e.g ht

随着微服务的发展，越来越多复杂问题都找打了第三方组件的解决方案，但是对于以往的单体架构的微服务化可能需要进行大量的重构，而且服务间的网络调用变得十分复杂，就需要一个完善的服务治理工具。而又因为 K8s 和微服务的完美结合，使其解决了微服务的编排部署的问题，但是服务治理还是过于繁琐，而且 k8s 对于服务治理方面不够完善，所以提出了服务网格的概念，通过代理微服务的网络，从而实现网络的追踪和监控。基本

前情提要：《ServiceMesh 究竟解决什么问题》《Istio 究竟是什么？》《Istio 分层架构设计？》Istio 架构体系中，流控 (Traffic Management) 虽然是数据平面的 Envoy Proxy 实施的，但整个架构的核心其实在于控制平面的 Pilot。灰度发布的过程在《Istio，灰度发布》一文中已经有过描述，今天重点说说 Pilot 和 Envoy 的交互流程与内部

看到这个问题，我想起当初玩魔兽世界的时候，25H难度的脑残吼的血量已经超过了21亿，所以那时候副本的BOSS都设计成了转阶段、回血的模式，因为魔兽的血量是int型，不能超过2^32大小。估计暴雪的设计师都没想到几个资料片下来血量都超过int上限了，以至于大家猜想才会有后来的属性压缩。这些都是题外话，只是告诉你数据量大了是有可能达到上限的而已，回到Mysql自增ID上限的问题，可以分为两个方面来说。

你应该从网上看过太多的文章说缓存穿透怎么解决？无非就是布隆过滤器，缓存空值什么的。但是，更深入的一个问题，缓存空值有没有问题？如果缓存的空值太多怎么办？如果用的redis，那么太多的空值会不会打爆你的redis？如果用的本地缓存，会不会打爆你的内存？继而引发的问题就是还是会打爆你的数据库。从线上问题说起前不久，我们线上环境压测，在QPS压倒2W之后RT达到了几十秒，排查后发现是redis的连接数不

 ServiceMesh(3)前篇：《ServiceMesh 究竟解决什么问题》《什么是 Istio，ServiceMesh 最流行落地》Istio 是 ServiceMesh 的产品化落地：它帮助微服务之间建立连接，帮助研发团队更好的管理与监控微服务，并使得系统架构更加安全它帮助微服务分层解耦，解耦后的 proxy 层能够更加专注于提供基础架构能力，例如：（1）服务发现 (discovery)（

ServiceMesh(2)上一篇介绍了《ServiceMesh 究竟解决什么问题？》，当微服务架构体系越来越复杂的时候，需要将 “业务服务” 和“基础设施”解耦，将一个微服务进程一分为二：一个进程实现业务逻辑，biz，即上图白色方块一个进程实现底层技术体系，proxy，即上图蓝色方块，负载均衡、监控告警、服务发现与治理、调用链… 等诸多基础设施，都放到这一层实现如此解耦之后：biz 不管是调用服

世界顶级公司的前端面试都问些什么\[每日前端夜话(0x0D)\] 京程一灯 前端先锋 原文：http://davidshariff.com/blog/preparing-for-a-front-end-web-development-interview-in-2017/作者：David Shariff （亚马逊 UI基础架构团队Leader）翻译：疯狂的技术宅 在过去的几年里，我在亚马逊和雅