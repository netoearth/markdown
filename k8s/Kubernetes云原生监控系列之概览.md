![图片](https://mmbiz.qpic.cn/mmbiz_png/u8wIoFASVo7ZG1iaI4d0vcCvI4I0m5OibKpyNDjWoRv9SiaTkiaN6p26NBiaUOxxbpkno3Vh0f6x5RJamjBUSBewIVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

云原生包含了开源软件、云计算和应用架构的元素。云计算解决开源软件的运行门槛问题，同时降低了运维成本和基础架构成本; 云原生给出了更多的应用架构规范, 更聚焦于能力和生态。 随着 Kubernetes 在基础设施的大规模落地，监控的需求也发生了变化，建设一套更符合云原生规范的监控体系势在必行。

## 一 监控需求变化

-   指标周期变短: 相比物理机时代，基础设施动态化，Pod销毁重建非常频繁，监控指标跟随 Pod 的生命周期
    
-   指标数量增加: 随着微服务化流行，指标的数量也大幅增长，研发工程师也更愿意埋点，获取服务状态；各种采集器层出不穷，指标应采尽采
    
-   指标维度更加丰富：物理机时代监控多从资源视角出发，更关注机器、交换机、中间件的采集；新的监控维度更加丰富，维度标签动辄十几个几十个
    
-   基础设施复杂度变高，监控难度增加：Kubernetes 组件和应用架构模型都需要投入时间去了解学习。Kubernetes 本身组件都通过 /metrics 接口暴露了监控数据，但是缺少体系化的文档指导和最佳实践总结
    
-   自动发现更重要：相比物理机时代的静态采集，自动发现采集目标的能力变得更重要
    

这个系列的文章重点讲解 Kubernetes 云原生监控体系，讲解 Kubernetes 本身各个组件的监控，以及运行在 Kubernetes 上的业务应用的监控。

## 二 采集目标

### 2.1 从 Kubernetes 架构看采集目标

![图片](https://mmbiz.qpic.cn/mmbiz_png/u8wIoFASVo7ZG1iaI4d0vcCvI4I0m5OibKPufsgUDlUQhXgOuXRkyWXswrLicW2aOhYzrRBNaWFRpt2lsV1hfGukQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

-   控制面组件监控，包括 kube-apiserver、kube-controller-manager、kube-scheduler、etcd 的监控
    
-   node 组件监控，包括 kubelet、kube-proxy 等组件的监控
    
-   Kubernetes对象监控（kube-state-metrics，后文简称KSM）
    
-   宿主节点资源监控
    
-   业务自身监控，业务自身的状态监控，白盒埋点监控
    

### 2.2 简单梳理监控指标

#### 2.2.1 kube-apiserver

kube-apiserver 是 Kubernetes 的总入口，其他 Kubernetes 组件都是通过与kube-apiserver 交互实现各种功能

-   要处理各种 API 调用，需要重点关注吞吐、延迟、错误率这些指标
    
-   缓存对象数据，需要关注自身的 CPU 和内存使用率
    

#### 2.2.2 kube-controller-manager

kube-controller-manager 负责监听对象状态，并与期望状态对比，如果状态不一致就进行调谐（reconcile)

-   主要关注各个 controller 的任务数、队列深度
    
-   与 kube-apiserver 交互的请求数、耗时、错误数
    

#### 2.2.3 kube-scheduler

kube-scheduler 经过一些打分算法，给 Pod 选取合适的 Node

-   调度框架的各个扩展点、算法的耗时
    
-   调度队列的任务数、队列深度
    
-   与 kube-apiserver 交互的请求数、耗时、错误数
    

#### 2.2.4 etcd

etcd 主要存储各种 Kubernetes 的对象数据，etcd 与 kube-apiserver 直接交互

-   关注 etcd 处理的请求量、请求耗时、cache 命中情况
    
-   etcd 集群的状态，是否在做 snapshot、是否在做数据 defrag、leader/learner 以及切换次数
    
-   etcd 的 db size ，wal 写入状态等
    
-   etcd 各种操作的耗时、数量情况
    
-   etcd 的 CPU、内存、fd 使用情况
    

#### 2.2.5 Node节点

-   Node本身的资源（cpu 内存 硬盘 网络 fd等）
    
-   Node上容器资源（cpu 内存 硬盘 网络 fd等）
    
-   Node上组件的监控，kubelet、kube-proxy、dockerd、containerd 等
    

#### 2.2.6 KSM

-   workloads(工作负载)指标，比如业务部署了多少个 deployment，部署了多少个pod，各个 pod 的状态如何
    
-   资源 quota 还剩多少
    

#### 2.2.7 业务监控

-   业务本身提供的 metrics 接口，来暴露业务指标
    
-   业务进程自身的通用监控指标，比如进程的CPU、内存、IO、句柄等
    

### 三 采集方式

#### 3.1 权限

监控对象明确后，就要确定如何收集指标。在收集具体指标前，首先要搞定的就是权限。Kubernetes 大部分指标的采集都是需要 kube-apiserver 授权的。关于 Kubernetes 的 authorization，主要包含RBAC、ABAC、Node Authorization、Webhook Authorization。

-   RBAC：Role-based access control 是基于角色的访问控制
    
-   ABAC：Atrribute-based access control 是基于属性的访问控制
    
-   Node Authorization：节点鉴权，专门用于 kubelet 发出的 API 请求进行鉴权
    
-   Webhook Authorization：webhook 是一种 HTTP 回调，kube-apiserver 配置webhook 时， 会设置回调 webhook 的规则，这些规则中包含了调用的 api group、version、operation、scope等信息。
    

Node Authorization 和 Webhook Authorization是两种专门的鉴权模型。ABAC 是用户和权限绑定；RBAC 是用户和角色绑定，角色和权限绑定。RBAC 比 ABAC 多了一个角色，更加灵活，后续我们主要是 RBAC 鉴权模式进行采集指标。

#### 3.2 自动发现

搞定鉴权模式后，我们面临的下一个问题是自动发现。对于控制面组件，类似prometheus 的 static config 可以满足需求， 对于使用 pod 部署的组件，我们都可以利用 k8s 自身的特性来动态发现目标，进而采集数据。比如，创建 service，然后利用 service 动态发现对应 endpoint 的变化，这样 pod 的 ip 发生变化也不影响采集。如果 公司已经有其他成熟的服务发现机制，也可以直接利用，比如consul。

#### 3.3 如何部署采集器

搞定了鉴权和自动发现，接下来要考虑采集器以何种方式采集了。

-   控制面组件，首先推荐使用 deployment方式 + 自动发现; 如果控制面组件是二进制方式部署, 可以用 deployment+ consul（或者静态抓取方式）；
    
-   node节点，首先推荐使用 daemonset 方式采集 node 指标和容器基础指标；
    
-   KSM，首先推荐使用单副本的 deployment 方式 + 自动发现；
    
-   业务监控，首先推荐使用 sidecar 模式来采集；其次使用集中式的 deployment方式采集；
    

部署yaml文件，我们已经放在k8s目录下:

-   deployment.yaml 以deployment方式部署
    
-   sidecar.yaml 以sidecar方式部署
    
-   deamonset.yaml 以daemonset方式部署
    
-   scrape\_with\_cafile.yaml 在pod内抓取指标
    
-   scrape\_with\_cafile.yaml 用cafile方式抓取指标
    
-   scrape\_with\_kubeconfig.yaml 用kube-config 文件方式抓取指标
    
-   scrape\_with\_token.yaml 用token方式抓取指标
    

### 四 监控大盘与报警规则

按照梳理的指标，已经将k8s 部分组件的大盘梳理完了，具体可以参考categraf k8s目录下对应的文件。

-   apiserver-dash.json
    
-   cm-dash.json
    
-   sheducler-dash.json
    
-   etcd-dash.json
    
-   coredns-dash.json
    

本文先将 Kubernetes 环境下的监控整体工作做了一个简单介绍，大家先有一个感性认识。后续我们会针对具体的步骤做详细的介绍，欢迎大家转发、点赞、关注和收藏。

## 关于夜莺监控

夜莺监控是一款开源云原生监控分析系统，采用 All-In-One 的设计，集数据采集、可视化、监控告警、数据分析于一体，与云原生生态紧密集成，提供开箱即用的企业级监控分析和告警能力，已有众多企业选择将 Prometheus + AlertManager + Grafana 的组合方案升级为使用夜莺监控。夜莺监控，由滴滴开发和开源，并于 2022 年 5 月 11 日，捐赠予中国计算机学会开源发展委员会（CCF ODC），为 CCF ODC 成立后接受捐赠的第一个开源项目。

夜莺监控项目文档站点：https://n9e.github.io/ 欢迎加入，一起打造最好用的云原生监控系统；也欢迎大家在 github 上 star 夜莺项目：https://github.com/ccfos/nightingale 及时了解社区动态。

加入夜莺交流社群请添加微信：flashcats

点击阅读原文即可访问夜莺的github ~