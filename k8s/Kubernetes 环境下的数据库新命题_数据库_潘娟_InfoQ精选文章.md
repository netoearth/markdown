伴随着互联网应用场景逐渐深入到生活的各个角落，为了确保前端用户的使用体验，对互联网产品的后端架构性能提出了更高的需求。如今，开发以及运维人员正在将工作重心和优化重点放在了后端基础设施的可用性、一致性、扩展性、弹性以及全面自动化管理等能够提升效率的技术能力层面。

## 背景：Kubernetes 环境中的微服务与数据库

-   **应用部署的变化**
    

一方面，在处处充斥着大数据以及高并发场景的今天，后台技术人员往往会花费更多精力在解决『大规模业务数据的存储与应用』等问题上，以确保数据库等基础设施能够实时提供最佳服务；另一方面，在云原生『大行其道』的今天，技术人员也需要根据业务发展，构思并设计能够满足未来业务诉求的云上服务架构。

而这种颠覆性的云原生技术，正是我们今天谈论的主要对象，它改变了从开发、交付、部署到维护的全部技术流程。如今，市场上的应用与业务已经普遍接受由各家云厂商所提出的『[XaaS](https://www.infoq.cn/article/riNy5Aeu3vQpD1635Sfu "xxx")』概念。与之伴随而来的，则是交付模式与架构形态的新变化。

-   **架构形态的变化**
    

在部署形态发生转变的同时，架构模式也从单一的整体式架构逐步向微服务架构演进。随着企业业务的持续扩张，前端的业务线也在持续发生着变化，这就要求后端系统能够快速响应前台每一条业务的单独诉求，而这在过去传统的架构模式中几乎是不可能实现的。

微服务通过将一项单体式服务切分为多个更小的单元，围绕业务领域组件来创建应用，使这些应用可独立地进行开发、管理和迭代。与此同时，在分散的组件中使用云架构和平台式部署、管理及服务功能，使产品交付变得更加简单。而随着 Service Mesh 的兴起，服务间的解耦以及 API 调用等操作也被更加简化，以适应业务微服务的部署、交付以及业务诉求。

而这些之所以能够兴起，正是 Kubernetes 在架构模式演进的过程中所提供的抽象平台与机制，这也是 Kubernetes 在全球流行的重要原因之一。

借用 [Kubernetes 官方文档](https://kubernetes.io/zh-cn/docs/concepts/overview/) 中的一段话，更能够体现 Kubernetes 的价值：

> _“Kubernetes 为你提供了一个可弹性运行分布式系统的框架。Kubernetes 会满足你对于扩展、故障转移、部署模式等多方面的要求。 例如，Kubernetes 可以轻松管理系统的 Canary 部署。”_
> 
> _（摘自“为什么需要 Kubernetes，它能做什么？”部分）_

Kubernetes 是管理微服务生命周期的理想化平台，那么作为有状态的服务，数据库在这种大环境下应具备哪些新特性呢？

-   **数据库层面的变化**
    

针对上述问题，应用层采用了微服务架构来作为解决方案。但数据库的情况却有些不同。在数据库层面，面对『蜂拥而至』数据体量，常见的解决方案是对数据库层进行数据分片，将数据库改造为分布式架构。

当前，不论是 MongoDB、Cassandra、Hbase、DynamoDB 等非关系型数据库还是 Cockroach、Google Spanner、Aurora 等新型数据库，分布式架构无处不在。分布式数据库需要将单体数据库分片为更小的单元，从而实现高性能和弹性伸缩。这也就是为什么 CockroachDB 兼容 PostgreSQL 协议；Vitess 为 MySQL 提供了分片特性；AWS 开发了 Aurora-MySQL 和 Aurora-PostgreSQL。

而对于企业用户数据库选型来讲，新型的数据库解决方案，不仅要解决分布式、单元化的问题，更要考虑『如何让客户从现有的 Oracle、MySQL、PostgreSQL、SQLServer 等数据库迁移至新数据库产品中』的问题，其原因在于：**传统关系型数据库仍然占据着庞大的市场份额，数据库替换的痛点是企业用户不得不面临的重要课题。**

## 需求：在 Kubernetes 上实现数据库的云中立

云的兴起意味着数据库面临着新的挑战。云平台遵循“随机应变”、“一切皆服务”、“开箱即用”的原则，正逐步改变用户对于传统研发、交付、部署、维护等技术流程。

以应用程序开发人员为例，在云原生环境下，开发人员通常会在云上或 Kubernetes 上交付应用。这是否意味着数据库也应该在云端或 Kubernetes 上部署？大多数用户应该都会这样认为，这也是为什么数据库即服务（[DBaaS](https://xie.infoq.cn/article/fa5af60ccf74ef156aa8830fa "xxx")）的概念越来越受欢迎。

然而，如果你是一家准备上云的企业，首先要确认的一定是哪家云厂商能够为业务提供长期的技术支持。但事实上，任何人都心知肚明这是很难实现的。因此为了将风险分摊开，在企业视角中，多云就成为了市面上最广泛的选择，而**将应用及数据库部署在 Kubernetes 上完成与云厂商的解耦和依赖**，正是解决这类问题的可选项之一。

这是因为 Kubernetes 本质上是容器编排的抽象层，具有高度的可配置性和可延展性，用户甚至可以在特定场景下自定义编码。例如，Kubernetes 上的挂载服务是由众多云供应商实现和提供。如果服务部署在 Kubernetes 上，应用就能够与 Kubernetes 实现交互，无需与不同类型的特定云服务或基础设施产生额外的交互。实践证明这种理念在无状态应用程序或微服务的领域非常适用。正是由于这些成功案例，**人们开始思考如何将数据库部署在 Kubernetes 上，进而实现云中立**。

不过这一方案的缺点在于，Kubernetes 天生适用于无状态应用，而非数据库及其它有状态应用，因此管理 Kubernetes 上的数据库或有状态的应用比管理无状态的应用层的难度更大。为了解决这一问题，可以基于 Kubernetes 的基础机制，使用 StatefulSet 和 Persistent Volume 等方法来实现桥接和适配。MongoDB、cockachdb、PostgreSQL 等数据库的 operator 就是采用了这种方法。

## 命题一：如何将单体数据库转换为更接近云原生的分布式数据库？

上述方案已经很常见，那还有其他的方案吗？答案是肯定的。本节将会演示另一种方法，将完成从『MySQL、Oracle、PostgreSQL』等单体数据库到分布式数据库系统的升级改造，并且以一种更加接近云原生的管理方式，来实现 Kubernetes 上的分布式数据库系统的部署、管理及使用。

顾名思义，这个方案主要分为两个步骤：首先将单体数据库转换为分布式数据库，其次将该分布式成功部署在 Kubernetes 环境中并实现有效管理。

![](https://static001.geekbang.org/infoq/35/352fd2192cd19083b79b2d6fa8f1e65b.png)

其核心理念在于利用数据库最本真的两大基础能力，即计算能力和存储能力，如上图所示。任何一款数据库必然具备这样的核心组件、核心能力；传统的 MySQL、PostgreSQL 和其它单节点数据库只是刚好将两个组件部署在一台服务器或容器上而已，而所谓的分布式数据库架构，即是采用存算分离的分布式架构，将存储节点、计算节点进行分布式化，部署在不同服务器节点上，进行内部组件的通讯，完成对上层业务的响应。

### **首先，使用数据分片能力打造分布式数据库**

作为一款分布式的数据库生态系统，ShardingSphere 提供两个客户端：ShardingSphere-Proxy 和 ShardingSphere-JDBC。用户可选择使用 ShardingSphere-Proxy 充当 “分布式” MySQL 或 PostgreSQL 的服务器端；或者使用 ShardingSphere-JDBC 作为 Java 应用的 Driver 端来完成同样的功能实现。无论采用哪种方式，用户都无需变更原来访问数据库的方式，这极大地降低了用户的学习曲线和迁移成本。

如果将单体数据库看做分片节点（即存储节点），将 ShardingSphere-Proxy 或 ShardingSphere-JDBC 看做全局的 Server 入口或代理端（即计算节点），他们的组合就是一种分布式数据库系统。如下图所示：

![](https://static001.geekbang.org/infoq/9f/9f0ddea653891a506878365aee176207.png)

此外，ShardingSphere 内置有 DistSQL （分布式 SQL），用于管理分片数据库、动态控制分布式数据库系统的工作负载，如 SQL 审计、读写分离、权限等。

例如，你可以使用 `CREATE TABLE t_order ()` SQL 在 MySQL 中创建一个新表。在 ShardingSphere-Proxy 中使用 `CREATE SHARDING TABLE RULE t-order ()` 就可以在新升级的分布式数据库系统中创建一个全局分片表。

### 其次，在 Kubernetes 层面部署分布式数据库

目前为止，我们已经解决了数据分布式问题，但我们如何将其部署在 Kubernetes 上呢？Shardingsphere-on-cloud 提供了 ShardingSphere-Operator-Chart 和 ShardingSphere-Chart，帮助用户在 Kubernetes 上部署 ShardingSphere-Proxy 和『云上 DBA』ShardingSphere-Operator 集群。

『ShardingSphere-Chart』这个 Charts 能够帮助用户使用 helm 命令在 Kubernetes 环境中部署 ShardingSphere-Proxy 集群，包括 Proxy 计算节点本身、治理中心、数据库连接驱动程序和 ShardingSphere-Operator。

作为『云上 DBA』，ShardingSphere-Operator 通过利用预定义的 CustomResourceDefinition，可以声明 ShardingSphere-Proxy 在 Kubernetes 上的部署结构，并持续监控运行状态，还可以基于 CPU 指标进行 Kubernetes 上的 HPA （横向自动扩容），并能够确保 ShardingSphere-Proxy 的高可用性，以维持所需计算服务节点的副本数量，提升整个分布式数据库系统的高可用性。

## 命题二：如何让分布式数据库系统变得无状态，真正实现分布式+云原生？

如上所述，数据库系统一般由计算节点和存储节点组成。当用户使用现有的生产环境的数据库作为新分布式数据库系统的存储节点时，ShardingSphere 则可以作为全局的计算节点，提供分布式数据库计算服务，即经典的计算存储分离的架构。这种架构在 Cloud native 场景下， 特别是 Kubernates 上有了重新的解读和应用。将计算节点作为无状态的应用部署在 Kubernates 上，并利用 Kubernetes 天然的平台能力来解决传统单点数据库分布式改造的课题。而存储节点可以部署在任意位置，可以是 Kubernetes 集群内、云的 RDS、私有环境等，真正实现分布式数据库存算架构的解耦和云化问题。

用户可以使用 ShardingSphere-Chart 以及 ShardingSphere-Operator-Chart 这两款工具轻松地部署和管理 ShardingSphere 集群，并在 Kubernetes 上创建自己的分布式数据库系统，无需关注单体数据库的位置，利用 Kubernetes 管理无状态计算节点，用户所创建的数据库可在任何公有云或私有云上，通过让 Kubernetes 上的 ShardingSphere (全局计算节点) 访问任意位置的单机数据库，从而对最终用户提供透明化的、完整的分布式数据库的解决方案，大幅降低升级改造成本，并提升整体的性能和存储能力。

在这类体系中 ShardingSphere-Proxy 将作为全局计算节点处理用户请求，从分片存储节点中获取本地结果集并进行计算。这意味着无需对生产环境的现有数据库集群进行危险操作，只需要将 ShardingSphere 导入到现有的数据库基础设施层，就可以将单机数据库存储节点与 ShardingSphere 全局计算服务器连接起来，形成一套完整的分布式数据库系统解决方案。

另一方面，为了提升该集群的可用性和自动扩缩容等特性，用户可使用 ShardingSphere-Operator 按业务需求量对 ShardingSphere-Proxy（计算节点）和数据库（存储节点）进行分别的扩缩容，真正实现按需分配、一键弹性。例如，有些用户只需要增强计算能力，ShardingSphere-Operator 将在几秒钟内自动扩容 ShardingSphere-Proxy，而不会对存储能力做任何改变，无危险操作、无付费成本。也有一些用户只关注存储容量，而对计算能力无过多期望，这时他们只需要快速启动更多的空数据库实例并执行 DistSQL 命令，ShardingSphere-Proxy 将对新旧数据库重新进行数据分片，以提高容量和性能。

采用 ShardingSphere 对现有数据库集群进行平滑分片，并以更为云原生的方式将其部署于 Kubernetes 上。与其关注如何从根本上打破当前的数据库基础设施，忙于重新寻找一个可以在 Kubernetes 上作为有状态应用进行有效管理的分布式数据库，我们不如从另一个角度思考这个问题：

『如何让分布式数据库系统变得“无状态”、充分利用现有的数据库集群、真正实现分布式+云原生？』

下面将展示两个真实场景案例：

1.  **Kubernetes 上的数据库**
    

首先使用 Helm charts 等方法将数据库（如 MySQL 和 PostgreSQL）部署到 Kubernetes 环境中，然后使用 ShardingSphere charts 成功部署了 ShardingSphere-Proxy 和 ShardingSphere-Operator 集群。完成部署后，用户可以使用原生的驱动访问方式连接 ShardingSphere-Proxy，使用 DistSQL 让 ShardingSphere-Proxy 感知到单机数据库，即分布式计算节点连接到存储节点，形成最终的分布式数据库解决方案。

![](https://static001.geekbang.org/infoq/15/15fd39ca5b8e41d1f48a03e740720ef0.png)

2.  **云端或本地数据库**
    

下图为云端、本地的数据库两种形态的部署架构。下图右半部分体现云上计算+云下存储的分布式数据库架构，即计算节点 ShardingSphere-Proxy 及 “云上 DBA” ShardingSphere-Operator 在 Kubernetes 上运行，但是数据库（存储节点）位于 Kubernetes 之外。

![](https://static001.geekbang.org/infoq/4c/4c0c14ebef6116f4cfd7cf0a49e011c3.png)

3.  **方案的优势与不足**
    

当然，这个世界上并不存在能够满足任何需求的技术银弹，每一款产品或解决方案，都会有其所擅长以及相对不足的领域。

#### 优势

-   利用现有数据库能力
    

在不破坏生产环境遗留数据库架构的前提下，用户可以平滑安全地构建分布式数据库系统。

-   高效平稳迁移
    

ShardingSphere 几乎没有停机时间帮助用户完成历史数据迁移及分布式改造。

-   DistSQL
    

ShardingSphere 提供 DistSQL 支持以原生数据库的方式（即 SQL）使用分布式数据库系统的分片、数据加密、流量治理等特性。

-   对计算存储能力单独进行灵活扩缩容
    

基于计算存储分离架构，用户可以真正实现『按需』单独分别对 ShardingSphere-Proxy （计算能力） 和 Databases （存储能力） 进行灵活扩缩容。

-   云原生运行和治理方式增强
    

由于 ShardingSphere-Proxy 本质上是一种无状态全局计算服务节点，同时充当全局数据库访问入口，因此更容易在 Kubernetes 上进行云原生的管理和部署。

-   多云或跨云
    

数据库作为有状态存储节点可以部署于 Kubernetes 或任意云端，避免单个云平台锁定。仅使用 ShardingSphere 连接节点就可构建分布式数据库系统。

-   数据库的其他特性
    

ShardingSphere 是一个围绕数据库的生态系统，除了数据分片，还提供数据加密、权限、读写分离、SQL 审计等特性等待为用户场景全面赋能。

-   用户可选择多个客户端或混合部署
    

ShardingSphere 根据用户需求提供两种客户端：ShardingSphereProxy 和 ShardingSphere-JDBC。通常，ShardingSphere-JDBC 性能更高，而 ShardingSphere-Proxy 支持所有的开发语言，提供分布式数据库集群的管理能力。ShardingSphere-JDBC 和 ShardingSphere-Proxy 混合部署可以进行优势互补。

-   开源支持
    

Apache ShardingSphere 是 Apache 基金会的一个顶级项目，开源至今已有五年以上。作为一个成熟的社区，Apache ShardingSphere 具备丰富的用户案例、文档和强大的社区支持。

#### 劣势

-   分布式事务
    

事务对于分布式数据库系统也至关重要。但是由于这种技术架构不是从存储层开发的，目前它依赖 XA 协议来协调数据源的事务处理，所以说并不算一个完美的分布式事务方案。

-   SQL 兼容性
    

部分 SQL 查询在存储节点（数据库）中表现良好，但在全新的分布式系统中会出现问题。我们开源社区仍在努力攻克这一难点。

-   全局一致性备份
    

虽然 ShardingSphere 定义为分布式数据库计算引擎，但多数用户倾向于将其视为分布式数据库。因此，用户需要考虑该分布式数据库系统全局备份一致性。ShardingSphere 正在针对这一特性进行研发，目前暂不支持（5.2.1），用户需要对数据库进行手动或用 RDS 备份。

-   成本增加
    

ShardingSphere 会接收所有请求，计算并发送至存储节点。每次查询都必然会增加成本，所有分布式数据库都会遇到这种情况。

## 实操指南

本节将演示如何使用 ShardingSphere 和 PostgreSQL RDS 创建分布式 PostgreSQL 数据库，以及用户如何对两个 PostgreSQL 实例进行数据分片。

以下演示过程中，ShardingSphere-Proxy 运行于 Kubernetes；PostgreSQL RDS 运行于 AWS。部署架构如下图所示。

![](https://static001.geekbang.org/infoq/6c/6cdfbcc2e00e50491e09febf1ad457cb.png)

演示主要包含以下内容：

1.  部署 ShardingSphere-Proxy 集群和 ShardingSphere-Operator。
    
2.  使用 DistSQL 构建分布式数据库表。
    
3.  测试 ShardingSphere-Proxy 集群（计算节点）的弹性伸缩能力和高可用。
    

#### 1\. 准备数据库 RDS

在 AWS 或任意云上创建两个 PostgreSQL RDS 实例作为存储节点。

#### 2\. 部署 ShardingSphere-Operator

下载 repo，在 Kubernetes 上创建一个名为 `sharding-test` 的命名空间。

```
Bash
```

复制代码

修改并部署 `shardingsphere-operator-cluster` 的 values.yaml 中的 `automaticScaling: true` 和 `proxy-frontend-database-protocol-type: PostgreSQL`。

![](https://static001.geekbang.org/infoq/5d/5d08dc612f7e1a822c1f17073fd10954.png)

![](https://static001.geekbang.org/infoq/b3/b33baa2cd03bdfe93ba0013c31db11c8.png)

以上操作会创建一个 ShardingSphere-Proxy 集群，其中包含 1 个 Proxy 实例、2 个 Operator 实例和 1 个 Proxy 治理实例，如下所示：

![](https://static001.geekbang.org/infoq/6a/6a7b54c332ba65adddd356de7b1c45c8.png)

#### 3\. 使用 DistSQL 创建分片表

(1) 登录 ShardingSphere-Proxy 并添加 PostgreSQL 实例。

```
Bash
```

复制代码

![](https://static001.geekbang.org/infoq/c7/c7cf884bcbad822ecc5bf4654d7813cd.png)

(2) 执行 DistSQL，用 MOD（user\_id, 4）创建一个分片表 `t_user`，显示这个逻辑表 `t_user` 的实际表。

![](https://static001.geekbang.org/infoq/0c/0c24a503f9bb997e89a39d9df755baa2.png)

(3) 插入测试行，在 ShardingSphere-Proxy 上执行查询，得出最终合并结果。

![](https://static001.geekbang.org/infoq/f4/f45e18a520e7aadfdde9cde4f9016745.png)

(4) 登录到两个 PostgreSQL 实例以获取它们的本地结果。

![](https://static001.geekbang.org/infoq/be/be173a2dccf64ed9486693d6e21c77d6.png)

![](https://static001.geekbang.org/infoq/d6/d6f9e2e0c74e9ac866f5b3df77d00827.png)

以上测试有助于理解 ShardingSphere 管理和分片数据库的功能,用户无需关注不同分片中的单个数据。

#### 4\. 测试 ShardingSphere-Proxy 集群（计算节点）的伸缩和高可用

如果用户认为新系统的 TPS 或 QPS 太高，可以考虑升级整个分布式数据库系统的计算能力。相较于其它分布式数据库系统，ShardingSphere-Proxy 增加计算节点的方式最为简单。ShardingSphere-Operator 能够基于 CPU 指标确保 ShardingSphere-Proxy 的可用性和弹性伸缩。此外，还可通过修改参数进行扩容或缩容，如下图所示：

![](https://static001.geekbang.org/infoq/24/246419b10ced9e412b23f107d194177d.png)

升级完成后用户将会获取两个 ShardingSphere-Proxy 实例，表明计算能力得到了增强。

如上，若用户需要更多的存储容量，可以采取以下步骤：

1.  在云端或本地部署启动额外的 PostgreSQL 实例；
    
2.  将新的存储节点添加到 ShardingSphere-Proxy 中；
    
3.  运行 DistSQL，使用 ShardingSphere 进行重新分片。
    

## 小结

本文聚焦 Kubernetes 上的一种全新分片数据库架构，利用分布式数据库存算分离的架构，借助现有单体数据库帮运维团队高效流畅地将数据库基础设施转化为现代数据库的分布式系统。通过在 Kubernetes 重新解读和应用分布式数据库计算存储分离这种传统架构，解决了有状态数据库在 Kubernetes 上的部署、治理、使用等问题。

如今，分布式数据库、云计算、开源、大数据、数字化转型都是热门概念。这些热词传递了新概念、新想法和新方案，对解决生产问题、满足生产需求大有裨益。但毕竟世界上没有完美的方案，我们要善于接受新的思想、通过权衡优劣、依据用户独特的场景需要选出最适合自己的方案才是技术选型的王道。

#### 关于作者

潘娟，SphereEx 联合创始人 &CTO、Apache Member & Incubator Mentor、Apache ShardingSphere PMC、AWS Data Hero、腾讯云 TVP、中国木兰开源社区导师。曾负责京东数科数据库智能平台的设计与研发，现专注于分布式数据库 & 中间件生态及开源领域。

参考资料：

\[1\] [https://www.verifiedmarketresearch.com/product/database-as-a-service-providers-market/](https://www.verifiedmarketresearch.com/product/database-as-a-service-providers-market/)  

\[2\] [https://kubernetes.io/docs/concepts/extend-kubernetes/](https://kubernetes.io/docs/concepts/extend-kubernetes/)  

\[3\] [https://shardingsphere.apache.org/document/current/en/overview/](https://shardingsphere.apache.org/document/current/en/overview/)  

\[4\] [https://github.com/apache/shardingsphere-on-cloud](https://github.com/apache/shardingsphere-on-cloud)  

\[5\] [https://shardingsphere.apache.org/document/5.2.0/en/user-manual/shardingsphere-proxy/migration/build/](https://shardingsphere.apache.org/document/5.2.0/en/user-manual/shardingsphere-proxy/migration/build/)