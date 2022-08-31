字节跳动[宣布](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FuNbT3Ss0rBYc9pqlZe3n8Q)正式开源 KubeWharf 项目。KubeWharf 是字节跳动基础架构团队在对 Kubernetes 进行了大规模应用和不断优化增强之后的技术结晶。这是一套以 Kubernetes 为基础构建的分布式操作系统，由一组云原生组件构成，专注于提高系统的可扩展性、功能性、稳定性、可观测性、安全性等，以支持大规模多租集群、在离线混部、存储和机器学习云原生化等场景。公告指出：

> “2016 年，字节跳动启用 Kubernetes 技术栈，开始对业务进行大规模容器化改造，到 2018 年，内部部署的容器单集群已经达到了上万个节点。时至今日，字节跳动实现云原生化的应用比例已超过 95%，我们计划和开源社区合作，逐步开放规模化云原生落地的工具和最佳实践。”

![](https://oscimg.oschina.net/oscnet/up-e3ab2e9ad18af7540374469d64b90fbd827.png) 

**项目地址：**[https://github.com/kubewharf](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkubewharf)

## 开源背景介绍

以 Kubernetes 为代表的云原生技术底座支撑了字节跳动业务的快速发展。从微服务场景开始，Kubernetes 逐渐演化，统一支撑了字节内部的大数据、机器学习以及存储服务等多种形态基础设施。从 2018 年至今，字节跳动的 Kubernetes 节点的规模增长了 10 倍以上。面对这样的增速，提高 Kubernetes 分布式操作系统的性能、资源利用率、可扩展性、可用性等愈发重要，KubeWharf 就是在这样的背景下诞生。

KubeWharf 第一批计划开源三个项目 ：

-   高性能元信息存储系统 KubeBrain
-   kube-apiserver 七层网关 KubeGateway
-   轻量级多租户方案 KubeZoo

## KubeWharf 首批开源项目

### KubeBrain

Kubernetes 是典型的中心化架构，元信息存储的性能对于集群的可扩展性和稳定性至关重要。在字节使用 Kubernetes 的过程中，随着集群规模增大到 1w 节点左右，etcd 逐渐成为制约集群可扩展性的瓶颈，经常出现读写延迟增高、OOM 等问题。

在分析了 etcd 的性能瓶颈和 Kubernetes 对于状态信息存储的需求之后，字节跳动基础架构团队自研了 KubeBrain，代替 etcd 作为 Kubernetes 的元数据存储系统，旨在提高 Kube-APIServer 存储的读写性能，增强集群稳定性。

下面是 KubeBrain 的一些特性：

-   **高性能：**通过读写逻辑的优化与底层存储引擎的改良，读写性能相较 etcd 有明显优势，且有效降低了 OOM 的风险。
    
-   **无状态：**KubeBrain 作为一个实现 API Server 所需要使用的存储服务端接口的组件进行存储接口的转换并不实际存储数据，实际的元数据存放在底层的存储引擎中，而 API Server 所需要监听的数据存放主节点内存中。
    
-   **扩展性：**KubeBrain 抽象了键值数据库接口，在此基础上实现存储 API Server 存储所需要使用的接口，具有指定特性的键值数据库均可适配存储接口。
    
-   **高可用：**KubeBrain 当前采用主从架构，主节点支持包括条件更新、读、事件监听在内所有操作，从节点支持读操作，基于 leader election 进行自动选主，实现高可用。
    
-   **兼容性：**兼容 etcd 接口，Kubernetes 可以无缝快速接入。
    
-   **水平扩容：**生产环境下，KubeBrain 通常使用分布式键值数据库存储数据，水平扩容包含两个层面：
    
    -   在 KubeBrain 的层面，可以通过增加从节点提高并发读性能
        
    -   在存储引擎层面，可以通过增加存储节点等手段提高读写性能和存储容量
        

### KubeGateway

kube-apiserver 组件是 Kubernetes 集群的所有外部请求访问入口，以及 Kubernetes 集群内部所有组件的协作枢纽。在使用 Nginx 做 API Server 的四层负载均衡器时，由于 client 和 API Server 通过 HTTP2 连接，容易造成负载不均衡，在 APIServer 重启后不均衡更为明显。同时，缺乏灵活的请求路由和治理能力，无法进行精细化的 API 流量控制。

KubeGateway 是为 kube-apiserver 的 HTTP2 流量专门设计并定制的七层负载均衡代理，目标是为大规模 Kubernetes 集群的 kube-apiserver 提供更灵活的稳定的流量治理方案。

下面是 KubeGateway 的一些特性：

-   **负载均衡：**为多个 kube-apiserver 进行请求级别的负载均衡，取代之前 “连接级别” 的负载均衡，使 apiserver 层可以做到真正意义上的水平扩展。
    
-   **请求治理：**为 kube-apiserver 提供流量特征定制的路由规则，可以通过 verb、apiGroup、resource、user、userGroup、serviceAccounts、nonResourceURLs 等信息来区分请求，进行差异化转发，可以满足将 Pod 和 Node 的请求分开处理、 apiserver 灰度升级等场景的需求。
    
-   **连接复用：**通过 HTTP2 连接复用，能收敛单个 kube-apiserver 实例上的 TCP 连接数，降低至少一个数量级。
    
-   **配置热更新：**路由等配置实时生效，不需要重启服务。可以避免使用 Nginx 时由于配置变更滚动升级导致所有连接都需要断开重连的问题。
    
-   **网关能力：**具备动态服务发现、限流、降级、熔断、缓存、黑白名单等网关能力。
    

### KubeZoo

社区现有的 Kubernetes 多租户方案各有其适用场景，但在租户体验，集群资源效率以及运维成本方面尚存在改进空间：基于 NameSpace 的多租户方案会把租户约束在特定的 NameSpace 下，租户无法自由使用 CRD、NameSpace 等集群级别的资源；基于 cluster 或 controller plane 隔离的多租户方案面临着资源利用率低，运维成本偏高等问题。

KubeZoo 是一个轻量级的 Kubenertes 多租户项目，基于协议转换的核心理念在一个物理的 K8s 控制面上虚拟出多个控制面，它具有以下特点：

-   **资源消耗低：**和租户独占集群或者 Master 控制面对比，KubeZoo 只需要一个 “网关”，无需为每一个租户起一个独立的控制面集群，资源消耗很少。且租户数量越多，资源消耗方面的收益更加显著。
    
-   **控制面隔离性高：**每个租户可以拥有独立且完整的 Kubernetes 集群视图，租户既可以使用 namespace scope 的资源，又可以使用 cluster scope 的资源，使用体验好。
    
-   **运维成本低：**KubeZoo 有效的减少了集群 / Master 管控面的数量，大大减少了运维、升级维护成本。
    
-   **高效率：**秒级创建租户，每个租户的集群创建相当于创建一个 Tenant 对象，省略了耗时的硬件资源分配和控制面初始化过程。
    

## 项目 RoadMap

目前 KubeWharf 开源了第一批的三个项目。字节方面表示，他们计划在未来进一步推动其走向完善：

-   **结合内外部用户需求，持续迭代已经开源的项目。**在 KubeBrain 项目上，持续增强稳定性、性能和易用性，探索多点写架构设计。在 KubeGateway 项目上，进一步完善 apiserver 的网关能力，并且尝试基于网关聚合的多 K8s 集群透明联邦。在 KubeZoo 项目上，结合 Virtual Kubelet / Kata 等技术探索公有云 Serverless Kubernetes 产品形态。
    
-   **继续开源其他内部项目。**我们会开源更多 Kubernetes 生态的项目，如在离线统一的高性能分布式调度器、混部管控系统等。以这些有差异化竞争力的云原生组件与技术为基础，推出 Kubernetes 发行版，持续输出在大规模多租集群、混部、大数据等关键场景的解决方案与最佳实践。
    

“我们也希望通过开源，输出云原生关键场景下更丰富的解决方案和最佳实践，对上游社区做出更多贡献，为云原生开发者提供工具、参考和新思路。”

**相关项目：**

-   KubeWharf: [https://github.com/kubewharf](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkubewharf)
-   KubeBrian：[https://github.com/kubewharf/kubebrain](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkubewharf%2Fkubebrain)
-   KubeGateway：[https://github.com/kubewharf/kubegateway](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkubewharf%2Fkubegateway)
-   KubeZoo：[https://github.com/kubewharf/kubezoo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fkubewharf%2Fkubezoo)