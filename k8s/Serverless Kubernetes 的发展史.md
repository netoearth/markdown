伴随着云原生的发展，从早先的单机版 Docker 到 Kubernetes 的编排领域的一统江湖，再到云上托管 Kubernetes，技术风雨变化。今天我们就沿着历史的脉络，一起看一下 Serverless Kubernetes 的发展史。

## 故事，从 Docker 讲起

故事虽然从 Docker 讲起，但我们不能忽视了 IaaS 先辈们在前面的披荆斩棘，以及云计算大佬们很早就确定的云计算发展规划。

在十几年前，先辈们从按照用户使用（云平台提供能力）维度，将云分为三层：

-   IaaS：Infrastructure as a Service，基础设施即服务，提供虚拟机或者其他基础资源作为服务提供给用户；
    
-   PaaS：Platform as a Service，平台即服务，将平台作为服务提供给用户，譬如在平台中可以随用随取各种中间件服务；
    
-   SaaS：Software as a Service，软件即服务，将应用作为服务提供给用户，譬如邮件服务。
    

如下图所示，从 IaaS 到 PaaS，用户（开发和运维）越来越少地感知基础资源，更加关注到业务当中。

![](https://static001.infoq.cn/resource/image/02/32/02bb6ce3314f0d3647c8deb750e9c032.png)

 让专业的人做专业的事情，从而发挥整体的最大效率。譬如一个初创的互联网买菜公司，没有必要自己去建机房、采购硬件、配置网络存储以及安装操作系统等与业务无关的事情，更应该把精力放到业务的开发和运营上面。

经过十几年的发展，IaaS 已经比较成熟，各种基础资源，如 ECS、VPC、EBS 等已经深入人心，但 PaaS 的发展却非常缓慢。

早在 2008 年，Google 就推出了 App Engine 服务，想打造一个开发平台，让开发者只需要编写业务代码就可以在 App Engine 上面运行。这个思想过于超前，开发者还不能完全接受。除了公有云以外，开源社区 PaaS 平台也在左突右冲。其中，IBM 的 Cloud Foundry 和 Redhat 的 OpenShift 最为出名，他们都希望提供一个应用快速发布的平台，但也都是不温不火，反而因为各种兼容问题越来越难使用。

直到 2013 年 Docker 的诞生，一个对开发者充分友好、一个命令可以拉起一个服务，并极致简单的操作方式，让 Docker 一下成为社区最受欢迎的开源项目之一。

Docker 的优势主要体现在：Docker 镜像将应该依赖的环境和应用打包成一个压缩文件，这个文件可以在任何安装了 Docker 的机器上面直接运行，解决了应用从开发、测试到生产各个环节部署问题，并且能够保障环境的一致性。

![](https://static001.infoq.cn/resource/image/0b/b4/0b853d801443d8c3e98b1d3de5f298b4.png)

Docker 的成功在于极致的简单操作而非技术的创新，像 cgroup、namespace 这些技术早就加入内核特性了。所以，Cloud Foundry 早先并没有把 Docker 看做竞争对手，因为这些技术早就在 Cloud Foundry 上使用了。反而，Dcoker 镜像这个无心插柳的功能，让 Docker 真正实现了“Build once, Run anywhere”。

## Kubernetes 确定江湖地位

最初的 Docker 是单机版本，面对大规模部署的场景时需要一套管理平台，就像 OpenStack 管理 VM 一样。

容器管理平台初期也是百家争鸣，譬如 Mesos、Swarm 等，但它们都没有脱离 IaaS 固有思维，还是停留在把容器当做虚拟机管理。直到 Kubernetes 的出现，才真正开始一统江湖。这里除了 Google 的背书以及脱胎于 Borg 的成熟架构以外，更重要的是 Kubernetes 在诞生之初就已经想好了容器如何管理（ Replica set）以及如何对外提供服务（Service）。

其中，最令人惋惜的就是 Docker 公司自家的管理系统 Swarm。当时的 Docker 虽然已经斩落头角，但 Docker 公司本身却没有实现盈利。于是公司推出了 Swarm 企业版，虽然 Swarm 后期也引入了很多 Kubernetes 的概念，但无奈大势已去，云原生的生态已经围绕 Kubernetes 蓬勃发展。

![](https://static001.infoq.cn/resource/image/be/13/be1be8c8046952f7387d69b79a72fe13.png)

Kubernetes 虽然由 Google 主导，但却保持了足够的开放性，将资源的管理抽象出接口规范，譬如针对容器运行时的 CRI、针对网络的 CNI、针对存储的 CSI，以及设备管理 Device Plugins 和各种准入控制、CRD 等。Kubernetes 正逐渐演变成云操作系统，各种云原生组件就是运行在这个操作系统之上的系统组件。

## 公有云托管 Kubernetes

虽然 Kubernetes 确定了领导地位，但 Kubernetes 的运维却并非那么容易。在这种背景下，公有云尝试纷纷推出了云上 Kubernetes 托管服务，比如国内的阿里云在 2017 年就推出了托管 Master 的方案：ACK（Alibaba Cloud Container Service for Kubernetes）。

在 ACK 中，Kubernetes 管理组件的安装和运维托管给公有云，使用 ECS 或者裸金属作为 Kubernetes 的计算节点，极大地减少了 Kubernetes 用户的使用成本。用户从云平台获取一个 kubeconfig 文件便可以直接通过 kubectl 命令行或者 Restful API 管理集群。

![](https://static001.infoq.cn/resource/image/98/65/98409a7a69ab43c95727a96f24a91a65.png)

如果需要扩容集群容量，只需要调整 ECS 个数，新创建的 ECS 会自动注册到 Kubernetes Master。不仅如此，ACK 还支持一键升级集群版本和各种插件。ACK 将繁杂的运维工作转移到云上，并且借助云的弹性能力，可以做到分钟级别的资源扩展。

## 将免运维和弹性进行到底

公有云相对私有云更加关注成本，因为在私有云中，用户的基础设施成本基本是固定的，用户不可能下线一个服务后去机房停一台服务器。与之相反，公有云则提供了按量付费的模式。

如果集群里面运行任务大部分都是 long run 并且资源需求是固定的任务，使用 ACK 没有问题，但如果是大量 job 类型的任务或者存在突发流量的情况，ACK 这种临时扩容虚拟机在虚拟机上启用容器方案在弹性方面有所欠缺。

比如某在线教育公司，每天晚上 7-9 点上课高峰期会临时扩容几万个 Pod，如果使用 ACK 就需要预先评估这些 Pod 的容量，然后再折算成 ECS 的算力，提前购买对应数量的计算节点加入到 Kubernetes 里面，并且还需要在 9 点之后将这些 ECS 释放掉，操作非常繁琐。

那么，有没有一种既能兼容 Kubernetes 使用方式，又能够秒级启动 Pod，并且按照 Pod 维度计费（ACK 按照 Node 维度计费）的方案呢？

AWS 率先提出 Fargate，可以在没有真实 Node 的情况下，以 Pod 的维度加入到 [[Kubernetes]] 集群。阿里云在 2018 年也推出了类似的产品 ECI（Elastic Container Instance），每个 ECI 就是一个 Pod，这不过这个 Pod 是托管在云上的。

[[Kubernetes]] 使用 ECI 有两种方式 ：一种是 ASK（Alibaba Serverless [[Kubernetes]]），另一种是 ACK + Virtual Node 的方案。在 ASK 中，计算节点完全变成了 Virtaul Node。Virtaul Node 是一个虚拟的无限容量的计算节点，负责 ECI 生命周期管理。Virtaul Node 会注册到 [[Kubernetes]] 里面，对于 [[Kubernetes]] 来说，它就是一个普通的 Node 节点。用户只需要提交原生的 [[Kubernetes]] Yaml 便可以创建出 Pod，完全兼容 [[Kubernetes]] 的使用。

![](https://static001.infoq.cn/resource/image/ea/8d/eaf9181bc686b02e6e77624d16ab728d.png)

Virtaul Node 还可以与普通 ACK 节点混用，用户可以将 long run 的任务调度到 ECS 节点上运行，然后利用 ECI 的快速启动（10s 内拉起容器），将突发或者短周期任务调度到 ECI 上面，从而达到成本最优。

![](https://static001.infoq.cn/resource/image/a6/54/a62c48180cb59c81aa1283c01a248154.png)

目前 ECI 已经被很多互联网以及人工智能公司所采用。在后续的文章中，我们将逐步分享几个典型用户在迁移 ECI 过程中遇到的技术问题和挑战。

总结一下，我们今天从技术发展的角度回顾了容器和 k8s 的发展历程，可以看到公共技术正逐渐沉淀到底层，无论是 k8s 还是 ServiceMesh，都在分别尝试将服务管理和流量管理下沉到基础设施中。但这些组件本身也存在管理成本，所以演化出云上托管。未来，随着技术的下沉，云计算提供的能力将不断上移、提供更加全面和丰富能力，让开发专注在业务之上。

_本文节选自阿里云技术专家陈晓宇的《深度揭秘阿里云 Serverless [[Kubernetes]]》系列专题。本专栏将主要围绕如何在 Serverless [[Kubernetes]] 场景中实现秒级扩容，以及在大规模并发启动中遇到的各种技术挑战、难点以及解决方案，系统地揭秘阿里云 Serverless [[Kubernetes]] 的发展、架构以及核心技术。_

**作者简介：**

陈晓宇，阿里云技术专家，负责阿里云弹性容器（ECI）底层研发工作，曾出版《深入浅出 Prometheus》 和 《云计算那些事儿》。