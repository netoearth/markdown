> 👉️**URL:** [https://medium.com/thermokline/comparing-k8s-load-balancers-2f5c76ea8f31](https://medium.com/thermokline/comparing-k8s-load-balancers-2f5c76ea8f31)
> 
> ✍ Author: Adam Dunstan| ThermoKline
> 
> 📆 Date：Oct 14, 2020
> 
> 📝**Description:**
> 
> Compare 3 open source load balancer controllers for kubernetes that can be used wiht any k8s distribution

> 🧠 **译者声明：**
> 
> 1.  请注意文章发布时间
> 2.  原文作者为 PureLB 的利益相关者，所以本文可能不尽客观。

## 词汇表[](https://ewhisper.cn/posts/29610/#%E8%AF%8D%E6%B1%87%E8%A1%A8)

| 英文 | 中文 | 备注 |
| --- | --- | --- |
| LoadBalancer | 负载均衡器 | 本文指 Kubernetes LoadBalancer |
| Allocator (controller) | 分配器（控制器） | MetalLB/PureLB 专有词汇 |
| speaker | 发言人 | MetalLB 专有词汇 |
| post failure | 陈旧 | MetalLB 专有词汇 |
| Service Groups | 服务组 | PureLB 专有词汇 |
|  |  |  |

在这篇文章中，我们讨论了三个开源的负载平衡器控制器，它们可以与任何 Kubernetes 的发行版一起使用。

-   [MetalLB.](https://metallb.universe.tf/) 流行的和最知名的 LoabBalancer Controller
-   [PureLB.](https://purelb.gitlab.io/docs/) 最新加入的。（完全公开，我参与了 PureLB 的开发）
-   [OpenELB](https://github.com/kubesphere/openelb)（之前叫：Porter）. 一个相对较近的新增项目，最初只关注路由的问题

添加一个实现服务类型 LoadBalancer 功能的控制器是简单、可扩展的集群操作所必需的关键网络组件。

-   启用对集群服务 / 应用的受控外部访问
-   外部资源是预先配置的
-   易于与自动化工作流程（CI/CD）整合

第一点是显而易见的，但是后两点在为你的集群设计负载均衡器解决方案时同样关键。根据部署模式的不同，负责确保集群可靠网络的团队与运行集群的团队不同是很常见的。预配置允许网络团队帮助完成设置，并将操作留给集群团队。这有利于与 CI/CD 的轻松集成，因为使用负载平衡器资源现在是标准的 k8s " 应用 " 定义和工作流程的一部分。

这些独立的负载均衡器可以在任何 k8s 环境中运行，而不像公有云中使用的集成负载均衡器或 K3s 中的 Klipper 等特定实施捆绑解决方案。

所有的负载均衡器控制器都暴露了服务，每个控制器如何实现这一点是不同的，这种差异影响了操作行为和故障模式。每个负载均衡器控制器都包括一个地址分配器和一个或多个向网络添加地址的机制，关键的区别是用于添加和管理地址的技术。

|  | MetalLB | PureLB | OpenELB(Porter) |
| --- | --- | --- | --- |
| IPAM | 内置的 | 内置的 & 外部的 | 内置的 |
| 使用 Linux 网络子系统 | no | yes | no |
| 本地地址 | yes | yes | 未完成 |
| 路由地址 | yes | yes | yes |
| 支持的协议 | BGP | any (BGP,OSPF,ISIS,RIP) | 部分 BGP |
| IPv4 & IPv6 | no | yes | no |
| 与路由类的 CNI 整合 | 困难 | 容易 | 有可能 |
| 使用 CRD 配置 | no | yes | yes |
| 冗余和故障转移 | 好 | 好 | 差 |

### 分配地址[](https://ewhisper.cn/posts/29610/#%E5%88%86%E9%85%8D%E5%9C%B0%E5%9D%80)

所有的负载均衡器都包括集成的 IP 地址管理（IPAM），用于将 IP 地址从池中分配给服务。它们都有类似的操作：它们观察 Services API 中没有 IP 地址的 LoadBalancer 类型，并以 IP 地址更新 kube-api。反过来，其他负载平衡器组件和 kube-proxy 也在观察服务 API 中包含 IP 地址的 LoadBalancer 类型的事件，并使用该信息来触发向网络添加地址。有一个共同的问题影响到分配器和其他组件的操作。服务 API 中用于 IP 地址的类型是一个字符串，包含 a.b.c.d 形式的地址（或者 IPv6 的 a::z）。网络中使用的 IP 地址通常被称为前缀，因为它包括子网掩码，例如 a.b.c.d/ 掩码（a::z/ 掩码）。在 golang 中，使用的类型是 ipNet，然而 kube-api 为这个字段使用了一个字符串。这是一个重要的细节，因为它必须由添加地址的过程来处理。

一般来说，有两种主要的操作类型。

### 向本地接口添加地址[](https://ewhisper.cn/posts/29610/#%E5%90%91%E6%9C%AC%E5%9C%B0%E6%8E%A5%E5%8F%A3%E6%B7%BB%E5%8A%A0%E5%9C%B0%E5%9D%80)

为本地接口添加地址是小型集群中的第一个要求，但在更大的集群中也可能是有用的，以公布需要从非 k8s 基础设施组件的本地访问的服务。然而，添加本地地址并不像它最初看起来那样简单。本地网络接口上的地址必须是唯一的，然而 k8s 以统一的方式协调它所有的节点。因此，必须使用一种机制来识别并确保一个地址只被添加到一个节点上，希望是在所需的网络上。如果 kube-proxy（用于在集群内编程分配的机制）被配置为 IPVS 模式，这就更加复杂了。

> IPVS。kube-proxy 中的 IPVS 实现通过减少对 iptables 的使用来增加可扩展性。在 iptables 输入链中不使用 PREROUTING，而是创建一个假的接口，叫做 kube-ipvs0。Kube-proxy 为 kube-ipvs0 添加负载均衡地址。默认情况下，Linux 内核将回答来自任何接口的 ARP/ND 请求，以获得任何接口上的地址。这种行为的原因是为了增加成功连接的概率。然而，当地址从本地子网分配时，这就产生了问题，因为所有的主机都响应 ARP/ND，导致重复的 ARP 信息。所有讨论的负载均衡器都要求将主机 ARP/ND 模式设置为 “严格”，这是对有多个接口的服务器的良好做法。

解决操作问题需要了解地址是如何被添加的以及被添加到哪个节点。了解 k8s 是如何允许向节点添加地址的，这一点很重要。一个使用节点的网络接口的 POD 被设置为使用`hostNetwork:true`，允许的能力为`NET_ADMIN`，在某些情况下设置为`NET_RAW`。它很容易弄清楚哪些 PODS 被设置为使用主机网络，POD 的地址将是节点的地址。

### 将地址添加到路由器上进行分配[](https://ewhisper.cn/posts/29610/#%E5%B0%86%E5%9C%B0%E5%9D%80%E6%B7%BB%E5%8A%A0%E5%88%B0%E8%B7%AF%E7%94%B1%E5%99%A8%E4%B8%8A%E8%BF%9B%E8%A1%8C%E5%88%86%E9%85%8D)

将分配的地址添加到网络路由表中是一个更可扩展和冗余的解决方案。路由允许同一个地址被多个 k8s 节点公布出来。这允许负载均衡器在交换机中填充路由表，以使用等价多路径，在流量进入集群之前提供一层数据包分配。多个路径被公布出来，使之成为一个更加冗余的解决方案。然而，重要的是要记住，服务负载均衡器需要与 kube-proxy（集群内流量分配机制）合作。

### 服务流量策略[](https://ewhisper.cn/posts/29610/#%E6%9C%8D%E5%8A%A1%E6%B5%81%E9%87%8F%E7%AD%96%E7%95%A5)

`externalTrafficPolicy`是一个重要的配置设置。它提供了两种选择，即如何将流量分配到集群和集群内。当设置为`Cluster`（默认）时，每个节点都被 kube-proxy 配置为接收并在整个集群内平均分配流量。当设置为本地时，kube-proxy 希望只在运行 POD 的节点上接收流量，本地设置被广泛用于外部应用，因为这种模式消除了 kube-proxy 对 NAT 的需求，导致源 IP 地址出现在 POD 上，而不是一个 "natted " 节点地址。负载均衡器必须适应这些行为，以实现可靠的流量交付。

### 开放源码[](https://ewhisper.cn/posts/29610/#%E5%BC%80%E6%94%BE%E6%BA%90%E7%A0%81)

所有这些负载均衡器控制器都是开源的，因此文档覆盖面也不一样。在许多情况下，要了解详细的操作，阅读源代码是必要的。

第一个被广泛部署的 LoadBalancer 是作为 Google 的一个 10% 的项目开始的，直到最近都是寻求开源软件的负载平衡解决方案的团队的唯一选择。它以两种不同的模式运行，即 L2 模式和 BGP 模式。

MetalLB 是通过 configmap 来配置的。

> 🧠 **译者注**:
> 
> MetalLB 现在已经有 Operator 了, 即也可以使用 CRD 方式进行配置, 20220219 发布了 v0.12.0 版本.  
> 所以在 " 配置 " 方面, 相比 PureLB 和 OpenELB 的差距已经在缩小或没有了.  
> 地址为: [https://github.com/metallb/metallb-operator](https://github.com/metallb/metallb-operator)  
> 但是目前 MetalLB Operator 还只有 amd64 版本的 image.  
> 另外, 目前 OpenShift 4.9 开始采用 MetalLB 及 MetalLB Operator 作为 BareMetal 的 LB 解决方案.  
> 感兴趣可以看这篇博文: [Deploying a high-availability, fault-tolerant Kubernetes Service on bare metal clusters with MetalLB BGP](https://cloud.redhat.com/blog/deploying-a-high-availability-fault-tolerant-kubernetes-service-on-baremetal-clusters-with-metallb-bgp)

该控制器由两个部分组成。

-   控制器。分配 IP 地址。每个集群有一个
-   发言人 (speaker)。配置节点网络。在所有节点上运行，提供对 IP 地址的访问

### 分配器（控制器）[](https://ewhisper.cn/posts/29610/#%E5%88%86%E9%85%8D%E5%99%A8%EF%BC%88%E6%8E%A7%E5%88%B6%E5%99%A8%EF%BC%89)

MetalLB 分配器是非常直接的。地址池是由 configmap 配置的，然而其模式的配置也是地址池配置的一部分。

控制器的行为是一致的，发言人实现了两种操作模式。在 MetalLB 中，这些模式在 configmap 池中被配置为 “协议”。

### **L2 Mode**[](https://ewhisper.cn/posts/29610/#L2-Mode)

在 MetalLB 中，有两个关键组件共同作用，将地址添加到单个节点上。第一个是使用成员列表的节点选举。这是一个简单的选举方案，使用服务名称来确保每个节点选择相同的赢家，即一个分配的 ip 地址将被回答的单一节点。

我使用了回答这个词，因为在 "L2 " 中，发言人进程实现了 ARP 和 ND。Kube-proxy 已经添加了必要的配置，将流量转发到目标 POD，ARP/ND 响应仍然是启动通信的必要条件。

在 L2 模式下，MetalLB 不使用 Linux 网络来回答 ARP/ND，处理是在发言人的 POD 可执行文件中实现的。因此它的 ARP/ND 处理对 Linux 及其网络工具来说是不可见的。识别哪个节点在回答 ARP/ND 请求，需要在其他网络主机上使用 APR/ND 工具，或者检查发言人 POD 的日志。

Metallb 不知道节点使用的地址范围，ARP/ND 过程会回答 ConfigMap 中配置的任何网络地址。这可能导致在 L2 模式下分配的地址无法到达。当节点网络跨越多个主机子网时，尤其需要注意，不能保证添加的地址会位于具有本地连接的节点上。

### **BGP Mode**[](https://ewhisper.cn/posts/29610/#BGP-Mode)

MetalLB 也在 BGP 模式下运行。BGP 提供了一种机制，将前缀（addr/mask）公布给邻近的路由器。使用路由的机制是利用 Linux 的第三层转发与路由协议相结合来提供目的地信息。

configmap 配置了 BGP 自治系统信息、对等体信息和池，由协议 bgp 标识。

发言人在每个节点上作为 daemonset 运行，从每个节点到配置的对等路由器建立 BGP 对等连接。AS 号码标识对等体为外部或内部。

当一个地址从池子里被分配出来时，每个发言人都会把这个地址作为一个主机路由来宣传。控制器只是从 kube-api 获取分配的字符串，并通过添加 /32 前缀使所有分配的地址成为主机路由，从而将其变成一个 ipnet。这是公布给 BGP 邻居的地址。这可能是个问题，因为有些路由器不接受 BGP 的 /32 路由。MetalLB 有一些 BGP 地址聚合功能，但这并不改变 /32 的广告，它只是告诉上游路由器进行聚合，对等路由器仍然会有 /32 路由。

MetalLB 有自己的 BGP 实现。这很重要，原因有三。

1.  MetalLB 使用节点的 IP 地址，是一个路由器。这意味着在默认网络命名空间中运行的另一个进程，如软件路由器，不能使用该主机地址与同一路由器对等。在较大的网络中，这可能导致非常复杂的 BGP 配置。
    
2.  BGP 功能没有办法与 Linux 路由表整合，只能用于宣传 MetalLB 创建的前缀。
    
3.  BGP 功能受限于 MetalLB 的支持水平，其他软件路由器中的功能需要专门为 MetalLB 开发。MetalLB 有一些额外的 BGP 功能，如聚合和社区支持，但没有被认为在标准路由器中必须的功能。
    

这两种模式都可以同时使用，每种模式都需要特定的配置。

### 流量策略[](https://ewhisper.cn/posts/29610/#%E6%B5%81%E9%87%8F%E7%AD%96%E7%95%A5)

MetalLB 支持这种设置，但是在 L2 模式下将 externalTrafficPolicy 设置为 local 可能会导致丢包，不应使用。在 BGP 模式下，只有有 POD 的节点才会被公布。流量将在运行 POD 的节点之间平均分配，但每个节点都将收到相同的份额，而不考虑该节点上的 pod 数量。

### 配置[](https://ewhisper.cn/posts/29610/#%E9%85%8D%E7%BD%AE%20-2)

MetalLB 是使用 configmaps 来配置的。在最初开发时，这是用于配置的主要机制，现在许多控制器仍然使用。configmaps 的问题是，在应用 POD 读取配置之前，没有办法验证它。因此，不正确的配置会导致 POD 失败或不正确的操作，" 失败后 " 只能通过查看个别 POD 的日志来调试 。

这进一步增加了 configmap 的复杂性，MetalLB 有一个奇怪的配置行为，值得在其部署前了解。不正确的配置变化会导致 configmap 被标记为过时。旧的配置作为注释存储在 configmap 中，继续被使用。例如，一旦定义并使用，改变地址池是很困难的。如果一个服务使用现有池中的地址，就不能改变池的配置。改变池子标志着配置的陈旧，MetalLB 继续使用同一个地址池，新的服务从旧的池子中分配。知道配置是否过时的唯一方法是检查 MetalLB POD 日志。如果发言人被手动重启或通过节点故障重启，旧的地址分配将保留，然而如果控制器被重启，所有服务将被重新编号为新的配置。如果控制器或发言人以无效的配置重新启动，并且配置无效，那么最后的配置将从 configmap 注释中加载。要注意通过检查发言人日志，确保每一个 configmap 的变化都是有效的。一个分配了大量地址的 " 陈旧 " 的配置可能很难在不遭受停机的情况下退掉。

MetalLB 的文档很充分，但很稀少。要了解它的运作方式，有必要阅读源代码。

### IPv6[](https://ewhisper.cn/posts/29610/#IPv6)

在检查源代码时，你会发现 MetalLB 的一些组件支持 IPv6，但作为一个系统，MetalLB 并不支持 IPv6。添加地址池会导致控制器和发言人中加载的配置失效，变得 “陈旧”。

### MetalLB 总结[](https://ewhisper.cn/posts/29610/#MetalLB-%20%E6%80%BB%E7%BB%93)

MetalLB 不使用原生的 Linux 网络，因此很难使用标准的 Linux 工具进行故障排除。它有自己独特的 BGP 实现，缺乏用于管理大规模 BGP 基础设施的功能。增加这些功能需要 BGP 协议的开发，虽然有可能，但要使 BGP 实现的功能齐全需要大量的开发工作。因为它是一个 BGP 路由器，使用 BGP for CNI 的节点不能同时将 BGP for CNI 和 BGP 从 MetalLB 对接到同一个上游路由器。有一些拓扑结构的解决办法，但它们给路由基础设施增加了很大的复杂性。

## PureLB[](https://ewhisper.cn/posts/29610/#PureLB)

PureLB 是最新的开源负载均衡器。检查代码会发现，它是作为 MetalLB 的一个分叉开始的，但是操作方式非常不同。PureLB 没有 " 操作 " 或协议模式，它自动从包含地址池的服务组分配地址给不同的接口。这些接口响应本地网络请求或通过独立的路由软件分配路由。

PureLB 是使用自定义资源（CRD）配置的。

该控制器由两部分组成。

-   分配器。分配 IP 地址。每个集群有一个
-   lbnodeagent . 配置节点网络。在所有节点上运行，提供对 IP 地址的访问。

PureLB 没有 “协议模式”，只是包含地址的服务组。PureLB 使用 Linux 网络功能来添加本地使用的地址，并由路由器重新分配。所需的配置很少，但在需要特定网络接口时，可以在自定义资源中覆盖接口默认值。

### 分配器[](https://ewhisper.cn/posts/29610/#%E5%88%86%E9%85%8D%E5%99%A8)

PureLB 的分配器包括一个集成的 IPAM，但也支持外部 IPAM 系统。目前唯一支持的外部 IPAM 是流行的开放源码 NetBox。分配器是使用自定义资源配置的。每个地址池都包含在一个独特的服务组中。服务组包含其他网络信息，允许 lbnodeagent 构建所需的前缀或添加的 IPNet。无论地址如何使用，分配器的行为都是一致的，不需要 " 协议 " 配置。

### 本地网络地址[](https://ewhisper.cn/posts/29610/#%E6%9C%AC%E5%9C%B0%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80)

在 PureLB 中，" 本地 " 地址是作为二级地址添加到节点主网络接口的地址。PureLB 通过识别哪个接口有默认路由来识别主网络适配器。当一个地址从地址池中分配，而该地址与主网络接口前缀相匹配时，PureLB 使用成员列表选出一个节点，并将该地址作为二级地址添加到接口上。

由于地址是作为二级网络接口添加的，Linux 网络响应 ARP/ND 请求，PureLB 并没有实现自己的 ARP/ND 进程。此外，使用标准的 Linux 网络工具，如 iproute2，地址是可见的，所以可以用标准的 Linux 工具找到地址的分配位置。

此外，当 PureLB 分配一个地址时，服务被注释为 IPAM 源和分配的地址。在本地地址的情况下，当选的节点和接口也被作为注释添加到服务中作为注释。

PureLB 知道节点网络，因此只有与本地网络相匹配的服务组池才会被应用于本地接口。这确保了已建立本地连接的地址可以被访问，需要路由的地址将使用虚拟接口机制。

### 虚拟网络地址[](https://ewhisper.cn/posts/29610/#%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80)

PureLB 将服务组池中与本地节点接口不匹配的地址添加到一个名为 kube-lb0 的虚拟接口。lbnodeagent 会在 POD 启动时添加该接口。

虚拟接口可用于添加任何将通过路由访问的网络，为了实现高效的路由，PureLB 允许在添加地址时使用默认或配置的地址聚合掩码。在服务组中添加的这个掩码被应用于创建添加到虚拟接口的 ipNet。Linux 内核负责将地址添加到接口并创建正确的路由。这使得在使用 externalTrafficPolicy: cluster 的情况下，一个子网可以被公布一次，或者在设置为本地时，一个主机路由可以被公布。网络设计者使用地址聚合来控制路由表的大小，支持聚合提供大规模的网络灵活性。

### 路由协议[](https://ewhisper.cn/posts/29610/#%E8%B7%AF%E7%94%B1%E5%8D%8F%E8%AE%AE)

PureLB 不直接实现任何需要添加软件路由器的路由协议，以分配添加到虚拟接口的地址的可达性。在集群没有路由软件用于基础设施或 CNI 的情况下，PureLB 提供一个捆绑的、预配置的 BIRD 路由器守护程序。路由器被配置为使用一个名为 " 重新分配 " 的通用功能来读取与虚拟接口相关的路由，并使用配置的路由协议进行分配。

利用这种机制，所使用的路由软件支持的任何路由协议都可以用来分配负载均衡器的地址，路由协议的开发与负载均衡器脱钩。成套的 BIRD 路由器支持 BGP、OSPF、RIP，用于 IPv4 和 IPv6。其他如 FRR 也支持 ISIS 和 IGRP，商业路由软件也可以使用。这种机制的主要好处之一是，路由软件提供了每个路由协议的完整实现，允许在设计网络和如何分配路由方面有最大的灵活性。在一个使用 OSPF 实现可达性的网络中，可以使用 OSPF 分配负载平衡器地址，而不需要增加 BGP。

当 CNI 没有被配置为使用隧道和 POD 网络跨越多个子网时，就需要路由协议。在某些情况下，CNI 的包括路由功能作为其操作的一部分。在这些情况下，PureLB 可以避免因试图在同一节点上运行两个路由实例而引起的问题。CNI 的路由过程只是重新分配来自虚拟接口的路由，并应用任何必要的路由策略，创造一个更简单、更自然的路由拓扑结构。

### 流量策略[](https://ewhisper.cn/posts/29610/#%E6%B5%81%E9%87%8F%E7%AD%96%E7%95%A5%20-2)

PureLB 支持虚拟网络地址的`externalTrafficPolicy:local`。只有当 POD 存在时，虚拟网络地址才会被添加到一个节点上。流量将在运行 POD 的节点之间平均分配，然而每个节点将收到相等的份额，无论该节点上有多少 POD。PureLB 不支持本地网络地址的`externalTrafficPolicy:local`，事实上，如果你试图将本地网络地址的 externalTrafficPolicy 设置为 local，PureLB 会将其重置为`Cluster`，以确保 kube-proxy 的配置正确，流量将到达所有选定的 POD

### 配置[](https://ewhisper.cn/posts/29610/#%E9%85%8D%E7%BD%AE%20-3)

PureLB 的配置是通过自定义资源。整个系统配置有一个 CR，每个服务组有一个 CR。由于每个 CR 都是独立的，并且 CR 可以在创建时进行验证，PureLB 保持了一致的配置，地址池可以根据需要进行更改。PureLB 永远不会改变已经分配的地址，它希望地址的改变是一种故意的行为，因此地址的改变需要添加 / 删除一个服务。

随着 PureLB 的运行，它向提供负载平衡服务的服务添加注释，以及更新服务事件日志，因此可以从服务中提取关键服务信息和状态，而不需要检查 POD 日志。

### IPv6[](https://ewhisper.cn/posts/29610/#IPv6-2)

PureLB 包括对 IPv6 的支持。定义了一个服务组，其中包括地址和网络信息，当使用该池时，会在本地或虚拟中添加 IPv6 地址。为了通过路由协议进行分配，需要在软件路由器中配置相应的 IPv6 路由协议，打包的 BIRD 路由器支持 IPv6。

### 总结[](https://ewhisper.cn/posts/29610/#%E6%80%BB%E7%BB%93%20-12)

这款新产品通过提供外部 IPAM、IPv6，更重要的是利用 Linux 网络，使自己与众不同。PureLB 没有试图重新发明网络和路由协议，而是能够使用所有的 Linux 和路由功能。在复杂的基础设施中，PureLB 更容易实现，因为在这些基础设施中，路由需要其他方面的可达性，如 CNI、存储或管理网络，并且已经在集群中存在。检查代码可以看出 PureLB 与分叉出来的 MetalLB 版本有多大区别，其简单性表现在它的代码行数只有一半左右。

## OpenELB（Porter）[](https://ewhisper.cn/posts/29610/#OpenELB%EF%BC%88Porter%EF%BC%89)

Porter 是一个相对较新的开源负载均衡器的补充。它是一个独立的开发项目。最初只有 BGP，最近加入了本地地址支持。Porter 的不同之处在于它在控制器和代理之间分配功能的方式。Porter 也实现了 BGP，但需要在本地网络上有一个 BGP 路由器，并有一个自定义的端口配置。

Porter 是使用自定义资源配置的。

该控制器由两部分组成。

-   porter-manager。分配地址和配置网络功能。
-   porter-agent。收集节点的具体信息，由 porter-manager 用来配置网络。

### 分配器[](https://ewhisper.cn/posts/29610/#%E5%88%86%E9%85%8D%E5%99%A8%20-2)

分配器是 porter-manager 的一部分。有一个 porter-manager 的单一实例。池被配置在称为 EIP 的自定义资源中。这包含地址和用于地址可达性的协议。

注意，porter 没有任何默认负载均衡器地址池的概念，注释总是需要的。

Porter 把网络配置集中在 porter-manager 中，porter-agent 只是收集事件，添加节点的具体信息，并把这些信息返回给 porter-manager。

### L2[](https://ewhisper.cn/posts/29610/#L2)

在 L2 模式下，Porter 不使用 Linux 网络。porter-manager 可执行程序实现了它自己的 ARP 进程。当一个地址从 EIP 池中分配时，porter-manager 代表目标节点回答对该地址的请求。服务的 ARP 过程集中在管理器中。虽然这种机制应该是有效的，但它并不严格符合 ARP 处理的运作方式。porter-manager 回答初始请求，因为它们是广播。然而，APR 缓存处理使用单播 ARP 消息来验证缓存条目，只有在单播 ARP 失败时才回落到广播。porter-manager 将不回答单播 ARP 请求。这增加了广播流量，并依赖于非标准的操作行为。跟踪硬件 mac 地址的第二层交换机安全机制可能会干扰这种操作，导致第二层功能失败。

使 L2 实现的问题更加复杂的是，当一个部署被扩展或存在一个守护集时，porter-manager 会错误地响应每个运行 pod 的节点的 mac 地址，有效地表现为网络上有重复的 ip 地址。唯一可以使用 L2 模式的时间是集群上的单个 POD。

最后，继续运行取决于 porter-manager，porter-manager 使用标准的 k8s POD 故障检测机制，因此 porter-manager 可能需要大量的时间来重新启动，影响到新的和现有的服务，因为它们将没有 ARP 响应。

### BGP[](https://ewhisper.cn/posts/29610/#BGP)

Porter 使用 gobgp 实现了 BGP，但是它实现了 gobgp 功能的一个非常小的子集。Porter BGP 的实现只具有与本地网段的路由器对等的必要功能。

Porter 从 porter-manager 节点为集群建立单个 BGP 对等体。BGP 对等体必须是在本地网段上，因为不可能配置 BGP 多跳。

当一个服务被定义时，porter-manager 会公布分配的地址，其下一跳是运行容器的节点。扩大部署，使 POD 分布在多个节点上，会导致为 ECMP 添加额外的路由。每个分配的地址都被转换为一个主机路由（/32）。然而，这种流量策略行为是不正确的，下面将详细介绍。

另外，Porter 声称支持 AddPath，这是一个为单一路由添加多个并发路径的机制。这个功能的使用似乎有些混乱，从 MetalLB 的一个帖子开始，建议这可能是一种机制，以更好地平衡流量分配到有多个 POD 的节点上。这不是 addpath 的目的，它被用来提供路由表的稳定性，在这种情况下，多个路径通常会被汇总。如果他们的实施方案使用 addpath 来为同一个节点增加更多的 ECMP 下一跳，这可能是某一特定厂商的行为。不管怎么说，大多数 BGP 实现都会显示 addpath 计数，我们没有看到 porter-manager 在上游路由器上更新这些计数以反映节点上的 POD 数量。

BGP 进程作为 porter-manager 的一部分运行，因此 porter manager 的故障会导致上游路由器的对等体宕机，软件路由器将撤回它从 porter 学到的所有路由。

为了解决一个节点上有两个路由器造成的问题，porter 用不同的端口号连接到上游的路由器。它解决了部分问题，必须注意手动配置路由器 -ID，而不是通过使用主机 IP 地址创建一个重复的路由器。

### 流量策略[](https://ewhisper.cn/posts/29610/#%E6%B5%81%E9%87%8F%E7%AD%96%E7%95%A5%20-3)

在 Porter 中改变 externalTrafficPolicy:local 确实改变了其操作。Porter 总是将流量发送到有 POD 的节点或节点。这违反了 kubernetes 描述的行为。在 BGP 模式下，所有的服务都应该被设置为 externalTafficPolicy:Local，因为这是 Porter 的行为方式，不管这个配置如何。这种设置将导致 kube-proxy 的正确配置，删除一个不必要的 NAT 步骤，并保留源 IP 地址。波特的第二层操作有多个 POD，导致重复的 IP 地址，因此进一步讨论这个设置如何影响现有的不正确的网络行为是不相关的。

### 配置[](https://ewhisper.cn/posts/29610/#%E9%85%8D%E7%BD%AE%20-4)

配置是通过自定义资源，每个 EIP 包含一个地址范围和相关协议。

L2 不需要进一步的配置，但是有两个 BGP 的 CR，`BgpConf`和 `BgpPeer`。`BgpConf` 要求定义`RouterID`，这进一步强调了 BGP 的有限实施。按照惯例，这将是主机设备上的一个地址。然而，这将是混乱的，因为 Kubernetes 安排了这个 POD，因此应该使用另一个地址作为静态标识符。与大多数实现不同的是，Porter 没有一个机制来自动选择一个有效的地址。

在讨论配置问题时，Porter 的文档非常有限。它可以从文档中安装和配置，然而，如果你的评估需要你弄清楚它是如何工作的，请准备阅读 golang 代码。

### IPv6[](https://ewhisper.cn/posts/29610/#IPv6-3)

Porter 没有对 IPv6 的支持。扫描代码时没有发现有任何 IPv6 支持。

### 总结[](https://ewhisper.cn/posts/29610/#%E6%80%BB%E7%BB%93%20-13)

构建这些负载平衡器协调器并不容易！K8s 网络与主机和路由协议的结合需要大量的知识和技能。因此，我赞扬开发者的工作。然而，Porter 在功能和操作上存在问题，除非你愿意做大量的额外开发，否则很难选择它作为一个解决方案。