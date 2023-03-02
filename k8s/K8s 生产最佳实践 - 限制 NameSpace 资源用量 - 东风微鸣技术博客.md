## 前言[](https://ewhisper.cn/posts/34662/#%E5%89%8D%E8%A8%80)

想象一下这个场景：多个系统运行在同一套 K8s 集群上，有重要系统，也有不太重要的系统。但是某一天，某个不重要的系统突然占用了该 K8s 集群的所有资源，导致该集群上的其他系统的正常运行受到影响。本文介绍了 Kubernetes 平台如何管理容量，以及作者对管理员的注意事项和建议。

## Kubernetes 资源限制概述[](https://ewhisper.cn/posts/34662/#Kubernetes-%20%E8%B5%84%E6%BA%90%E9%99%90%E5%88%B6%E6%A6%82%E8%BF%B0)

我们寿险了解 Kubernetes 平台如何在容器和节点级别应用资源约束。 为了讨论合理规模，我们将专门关注 CPU 和内存，尽管还有其他因素需要考虑。

可以为每个容器和 Pod 指定 resource requests 和 limits。 Requests 是为 pod 预留的有保证的资源，而 limits 则是旨在保护集群整体架构的安全措施。 在 Kubernetes 中，pod 的 requests 和 limits 之间的关系被配置为服务质量（QoS）。 在节点上，kubelet（可以监控资源的代理）将此信息传递给容器运行时，容器运行时使用内核 cgroups 来应用资源约束。

要调度新的 pod，Kubernetes 调度程序将确定可用节点上的有效位置，并考虑现有 pod 资源限制。 Kubernetes 会预配置一些系统预留，以留出资源供操作系统和 Kubernetes 系统组件使用（具体如下图）。 剩余量被定义为可分配的，调度程序将其视为节点的容量。 调度器可以基于所有单元的合计资源请求来调度单元到节点的容量。 请注意，所有单元的聚合资源限制可以大于节点容量，这种做法称为超额使用（超配 or 超卖）。

[![K8s Node 资源分配](https://pic-cdn.ewhisper.cn/img/2022/09/30/fd056bae3946f804978f97ce17a3626b-K8s_Node_Allocated_Resources.png)](https://pic-cdn.ewhisper.cn/img/2022/09/30/fd056bae3946f804978f97ce17a3626b-K8s_Node_Allocated_Resources.png "K8s Node 资源分配")

在管理节点容量时，我们要尽量避免两种情况。 在第一种情况下，实际内存利用率达到容量，并且 kubelet 基于驱逐信号触发节点压力驱逐。 如果节点在 kubelet 可以回收内存之前用完内存，则节点 oom-killer 将做出响应，根据从每个 pod 的 QoS 计算的 oom\_score\_adj 值选择要删除的 pod。 因此，构成这些 pod 的应用程序会受到影响。

在 CPU 上过量使用的底层机制与内存的行为不同，因为它将 CPU 时间分配给各个容器。 高 CPU 利用率会导致 CPU 节流 (CPU throttling)，但不会触发节点压力驱逐，也不会自动导致 Kubernetes 终止 Pod。 但是也请注意，CPU 耗尽仍可能导致应用程序 pod 降级、活动探测失败并重新启动。

我们还希望避免另一种情况。 在节点级别，requests 是有保证的资源，并且必须小于容量，因为 Kubernetes 调度程序不会超额预订。 如果 requests 明显且始终大于实际使用的资源，则多余的容量基本上未被使用。 虽然可能需要为高峰处理时间保留资源，但管理员应在这一点与运行可能不需要的过剩容量的重复成本之间进行平衡。 根据实际使用情况配置请求是一种平衡行为，应考虑应用程序的风险管理（平衡可用性和成本）.

## Kubernetes 管理员能做啥[](https://ewhisper.cn/posts/34662/#Kubernetes-%20%E7%AE%A1%E7%90%86%E5%91%98%E8%83%BD%E5%81%9A%E5%95%A5)

Kubernetes 管理员的一个主要关注点是管理和合理调整集群容量，我们可以在 Web 上利用 Prometheus + Grafana 仪表盘和命令行捕获集群利用率指标，供管理员使用。

但是 Kubernetes 管理员还面临一个棘手的大问题：正在运行的应用程序（的资源管理）。 解决特定问题的应用程序可以由不同的开发人员以不同的方式编写，从而导致不同的性能（比如 java 编写的可能消耗内存较多，golang 消耗内存相对较少）。 每个应用程序都是独特的，没有一种适合所有应用程序的方法。 管理员对开发人员的应用程序的控制能力较弱，在大型企业中，单个管理团队可能很难接触到众多的开发团队。 因此，管理员的重点应该是 **设置护栏**，以允许开发人员（在护栏内）调整自己的应用程序。

## 配置 LimitRange[](https://ewhisper.cn/posts/34662/#%E9%85%8D%E7%BD%AE%20-LimitRange)

绕了这么久，终于进入正题了。

为此，管理员可以为每个 NameSpace 配置不同的 LimitRanges，为开发人员提供针对各个容器和 pod 的建议大小限制。 以下是 LimitRange 的示例。 由于每个集群和应用程序都有不同的业务和风险要求，因此各位读者实际应用时数字会有所不同。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>LimitRange</span><br><span>metadata:</span><br>  <span>name:</span> <span>"resource-limits"</span><br><span>spec:</span><br>  <span>limits:</span><br>  <span>-</span> <span>max:</span><br>      <span>cpu:</span> <span>"2"</span><br>      <span>memory:</span> <span>4Gi</span><br>    <span>min:</span><br>      <span>cpu:</span> <span>125m</span><br>      <span>memory:</span> <span>128Mi</span><br>    <span>type:</span> <span>Pod</span><br>  <span>-</span> <span>default:</span><br>      <span>cpu:</span> <span>"0.5"</span><br>      <span>memory:</span> <span>1Gi</span><br>    <span>defaultRequest:</span><br>      <span>cpu:</span> <span>250m</span><br>      <span>memory:</span> <span>256Mi</span><br>    <span>max:</span><br>      <span>cpu:</span> <span>"2"</span><br>      <span>memory:</span> <span>4Gi</span><br>    <span>maxLimitRequestRatio:</span><br>      <span>cpu:</span> <span>"25"</span><br>      <span>memory:</span> <span>"4"</span><br>    <span>min:</span><br>      <span>cpu:</span> <span>125m</span><br>      <span>memory:</span> <span>128Mi</span><br>    <span>type:</span> <span>Container</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在 Kubernetes 中进行开发的良好实践是创建微服务应用程序，而不是大型的巨石应用程序。 为了鼓励微服务的开发，应该应用 limits 来约束 pod 的最大大小。 节点的物理容量可能会决定此最大大小，因为它应该可以轻松地容纳几个最大的 pod。 还是类似这个图：

[![1 个 K8s node 应该可以轻松地容纳几个最大的 pod](https://pic-cdn.ewhisper.cn/img/2022/09/30/fd056bae3946f804978f97ce17a3626b-K8s_Node_Allocated_Resources.png)](https://pic-cdn.ewhisper.cn/img/2022/09/30/fd056bae3946f804978f97ce17a3626b-K8s_Node_Allocated_Resources.png "1 个 K8s node 应该可以轻松地容纳几个最大的 pod")

让我们继续上面的 LimitRange 示例。 最小 pod 和容器大小可能由正在运行的应用程序的需求确定，管理员不必强制执行。 为了简单起见，我们还鼓励开发人员在每个 pod 上运行一个容器（一个典型的例外是使用 sidecar 容器，如 Istio 的 sidecar）。 因此，上面的示例对 pod 和 container 使用相同的资源值。

默认 requests 和 limits 作为开发人员的建议值。 **未显式声明容器大小的工作负载资源（即 pod）将继承默认值**。 作为一种好的做法，开发人员应明确定义工作负载资源中的资源请求和限制，而不采用默认值。

CPU 和内存的 `maxLimitRequestRatio` 是开发人员的突发准则。 在开发环境中，当原型应用程序经常空闲运行，但在使用时需要合理的按需资源时，高 CPU maxLimitRequestRatio 会很好地工作。 开发人员可能只在工作时间工作，在自己的 IDE 中离线编码，偶尔测试单个微服务，或者测试 CI/CD 管道的不同阶段。 相比之下，如果许多最终用户在一天中同时访问应用程序，您将看到更高的基准利用率。 这可能更接近您的生产环境，并可能降低 maxLimitRequestRatio（可能是事件 1：1 的请求限制）。 由于管道各阶段的不同利用率模式将导致不同的请求和限制，因此在生产之前使用模拟工作负载进行测试以确定适当的单元大小非常重要。

开发人员将使用 maxLimitRequestRatio 作为适当调整大小的准则。 Kubernetes 调度程序基于资源请求做出调度决策，因此开发人员应配置资源 requests 以反映实际使用情况。 然后，基于应用程序的风险状况，开发人员将配置 limits 以遵守 maxLimitRequestRatio。 将 maxLimitRequestRatio 设置为 1 的管理员强制开发人员将 requests 配置为等于限制，这在生产中可能是理想的，以降低风险并优先考虑稳定性。

在本文前面，我们比较了内存和 CPU，并描述了这两种资源在负载下的不同行为，高内存可能导致 pod 逐出或从内存不足的情况重新启动。 因此，最好是谨慎行事，并为不同环境的内存配置较低的 maxLimitRequestRatio，以防止应用程序 pod 重新启动。 为 OpenJDK pod 配置内存时还应注意其他事项。 （如果没配置相应动态调整的参数）, 容器和 pod 内部的 JVM heap 对容器的请求和限制一无所知，但应用于前者的资源约束将影响后者。

## 配置 ResourceQuota[](https://ewhisper.cn/posts/34662/#%E9%85%8D%E7%BD%AE%20-ResourceQuota)

管理员还可以配置 ResourceQuotas，它为 NameSpace 提供基于容量的限制，以指导开发人员根据预测的估计值来调整应用程序的规模。 下面是一个 ResourceQuota 示例。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>ResourceQuota</span><br><span>metadata:</span><br>  <span>name:</span> <span>compute-resources</span><br><span>spec:</span><br>  <span>hard:</span><br>    <span>limits.memory:</span> <span>20Gi</span><br>    <span>requests.cpu:</span> <span>"4"</span><br>    <span>requests.memory:</span> <span>20Gi</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在应用程序 NameSpace 的初始创建过程中，开发团队应与管理员一起预测其应用程序大小并应用适当的配额。 管理员应根据服务、副本的数量和 pod 的估计大小来预测应用程序大小。 为了简化对众多 NameSpace 的管理，管理员可考虑类似 AWS 的方法作为起始准则，其中 small、medium、large、xlarge 应用程序被给予对应的预定配额。

应用程序跨 CI/CD 管道的各个阶段进行运行，每个阶段都位于不同的集群或 NameSpace 中，并具有自己的配置配额。 在不考虑性能和高可用性的开发和测试 NameSpace 中，应用程序应配置最小的 pod，并为每个服务配置 1 个 pod 副本，以减少资源使用。 另一方面，在生产集群或 NameSpace 中，应使用更大的 pod 和每个服务至少 2 个单元副本，以处理更高的业务量并提供高可用性。 通过使用 CI/CD 管道中的模拟工作负载进行压力和性能测试，开发人员可以在生产发布之前确定适当的生产 pod 大小、副本数量和配额。

管理员应针对未来的扩展制定配额预算，并考虑应用程序的使用模式、峰值容量和已配置的 pod 或节点的 autoscaler（如果有）。 例如，可在快速添加新微服务的开发 NameSpace、用于确定适当生产 pod 大小的性能测试 NameSpace、或使用 HPA 来调整峰值容量的生产 NameSpace 中分配附加配额。 管理员应针对上述各种情况和其他情况提供足够的配额开销，同时平衡基础架构容量的风险并保护架构容量。

管理员和开发人员都应该预期会随着时间的推移调整配额。 开发人员无需管理员的帮助即可收回配额，方法是查看每项服务并减少 Pod requests 或 limits 以匹配实际使用量。 如果开发人员已经采取了这些步骤，但仍然需要额外的配额，那么他们应该联系管理员。 管理员应将开发人员的定期配额请求作为一个机会，根据以前预测的估计值分析实际消耗量，并相应地确认或调整配额大小和新的预测估计值。

另外再介绍在调整配额大小时的一些次要注意事项。 在确定 CPU 和内存的配额比率时，应考虑节点容量，以便有效地利用两者。 例如，m5.2xlarge 类型的 AWS EC2 实例为 8 个 vCPU、32 GiB RAM。 由 m5.2xlarge 节点组成的集群可以按照每 4 GB RAM 对应 1 个 vCPU 的比例分配应用配额（**不考虑节点的系统保留空间**），从而高效地使用 CPU 和内存。 如果应用程序工作负载（即 CPU 或内存密集型）与节点大小不匹配，则可以考虑使用不同的节点大小。

管理员对何时应用和不应用配额的 CPU limits 一直存在争议，这里我们将提供一些考虑事项，而不是正式的指导。 正如我们前面所讨论的，pod 的 CPU 不足会导致节流，但不一定会导致 pod 终止。 **如果管理员倾向于过量使用并利用节点上的所有可用 CPU，则不应设置配额的 CPU limits**。 **相反，应设置配额 (resource quota) 的 CPU limits，以减少过度使用和应用程序性能风险** ，这可能是一个业务和成本决策，而不是技术决策。 与生产环境相比，开发环境可以容忍更高的风险和不可预测的性能，因此管理员可以考虑 **将 CPU limits 应用于生产而不是开发**。

最后，在某些特殊情况下，不建议应用配额。 应用配额的目的是让管理员能够对自定义开发的应用程序的容量规划进行一定程度的控制。 配额不应应用于 Kubernetes 自身的组件，因为这些项目需要预先配置的资源量。 出于类似的原因，配额也不应适用于第三方供应商提供的企业版应用程序。

## 总结[](https://ewhisper.cn/posts/34662/#%E6%80%BB%E7%BB%93)

在本文中，我们介绍了 Kubernetes 平台如何通过资源约束保护架构，包括：

-   Pod 的 requests 和 limits
-   Node 的资源分配
-   NameSpace 级别的针对 Pod 和容器的 LimitRange
-   NameSpace 级别的 ResourceQuota

并提供了在应用程序 NameSpace 中应用 limits 和 quota 的保护措施时的合理调整注意事项。 每个应用的风险偏好和 Kubernetes 集群的容量各不相同，需要综合考量再实施。

## 参考文档[](https://ewhisper.cn/posts/34662/#%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

-   [Kubernetes instance calculator (learnk8s.io)](https://learnk8s.io/kubernetes-instance-calculator)
-   [限制范围 | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/policy/limit-range/)
-   [资源配额 | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/)