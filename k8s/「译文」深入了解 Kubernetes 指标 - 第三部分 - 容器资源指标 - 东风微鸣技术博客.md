> 👉️**原文 URL:** [https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-3-container-resource-metrics-361c5ee46e66](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-3-container-resource-metrics-361c5ee46e66)
> 
> ✍️**作者:** [Bob Cotton](https://medium.com/@bob_cotton)
> 
> 📝**Description:**
> 
> This is part 3 of a multi-part series about all the metrics you can gather from you Kubernetes cluster. In part 2, I explained, and then demonstrated the USE method to select and examine the most…

这是关于你可以从 Kubernetes 集群中收集的所有指标的多部分系列的第三部分。

在 [第二部分](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-2-c869581e9f29) 中，我解释并演示了选择和检查节点 (Node) 上最重要资源的 USE 方法；内存、CPU、磁盘和网络。这一次，我将关注容器层面的指标。这些是 cAdvisor 所报告的指标。

## cAdvisor 的 Container 指标[](https://ewhisper.cn/posts/65005/#cAdvisor%20%E7%9A%84%20-Container-%20%E6%8C%87%E6%A0%87)

来自谷歌的 [cAdvisor](https://github.com/google/cadvisor) 项目最初是一个独立的项目，用于收集 **节点上运行的容器的资源和性能指标**。在 Kubernetes 中，**cAdvisor 被嵌入到 kubelet 中**。这是控制集群中每个节点上所有容器的过程。这很方便，因为你不需要在每个节点上运行另一个进程来收集容器指标。

kublet 在一个 `/metrics` 端点上以 Prometheus 阐述格式公开了它的所有运行时指标和所有 cAdvisor 指标。

cAdvisor 所提供的 “container” 指标最终是由底层的 Linux cgroup 实现报告的指标。就像节点指标一样，它们数量众多，内容详细。然而，我们关注的是容器对底层节点所提供资源的使用情况。具体来说，我们对 CPU、内存、网络和磁盘感兴趣。在处理资源的时候，同样，在选择这些指标的报告时，最好使用 USE 方法。

> 📝 **译者注:**
> 
> USE 方法更多地关注于监视性能，并被用作确定性能问题和其他系统瓶颈的根本原因的起点。理想情况下，在监视应用程序时，USE 方法和 RED 方法应该同时使用。  
> USE 方法关键指标为:
> 
> -   Utilization(使用率)
> -   Saturation(饱和度)
> -   Errors(错误)
> 
> RED 方法关键指标为:
> 
> -   Rate(比率)
> -   Errors(错误)
> -   Duration(持续时间)

## USE 方法和容器度量[](https://ewhisper.cn/posts/65005/#USE-%20%E6%96%B9%E6%B3%95%E5%92%8C%E5%AE%B9%E5%99%A8%E5%BA%A6%E9%87%8F)

作为快速提醒，USE 方法是指 Utilization(使用率)、Saturation(饱和度)和 Errors(错误)。请参考本系列的 [第二部分](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-2-c869581e9f29)，以深入了解这一方法。

因为这些指标的来源从节点（node\_exporter）变成了容器（cAdvisor），所以指标的名称也会改变。此外，每个指标都会对你集群中的所有容器进行报告。使用 Prometheus 中的 `sum` 方法将是必要的，以获得你的应用程序的整体视图。

在我开始谈论单个资源指标之前，我们需要谈谈 Kubernetes 中的一个功能，这将使计算饱和度 (Saturation) 更容易。这个功能就是资源 "requests" 和 “limits”。

## Kubernetes Requests 和 Limits[](https://ewhisper.cn/posts/65005/#Kubernetes-Requests-%20%E5%92%8C%20-Limits)

Kubernetes 系统的核心是一个调度器，将容器放在节点上。就像用不同大小的物品包装一堆不同大小的盒子一样，调度器需要知道节点的容量和被放置在这些节点上的容器的大小。如果不知道容器的 “大小 (size)”，你就很容易在集群中过度配置(over-provision) 节点，导致由于过度拥挤而出现性能问题。

请求 (Requests) 和限制 (limits) 作为 deployment 的一部分被应用到容器规范中。从 Kubernetes 1.10 开始，有两种资源类型可以设置请求和限制：CPU 和内存。CPU 被指定为 CPU 或核心的分数（低至 1/1000），内存被指定为字节(bytes)。

一个 **request** 是对你的容器将需要的该资源的最低数量的投标。请求并没有说你将使用多少资源，只是说你将需要多少。你在告诉调度器你的容器需要多少资源来完成它的工作。Request 用于 Kubernetes 调度器的调度。对于 CPU request，它们也被用来配置容器如何被 Linux 内核调度。关于这一点，在另一篇文章中会有更多介绍。

**Limit**是你的容器将使用的该资源的最大数量。Limit 必须大于或等于 Request 值。如果你只设置 Limit，那么 request 将与 limit 相同。

Limit 允许容器有一些余地，以超过资源请求。由于 Kubernetes 调度器不考虑 limits，因此 limits 给了你一个在节点上过度配置容器的旋钮。也就是说，如果你的容器超过了你的 limit，行动取决于资源资源类型；如果你超过了 CPU 的 limit，你将被遏制(throttled)，如果你超过了内存的限制，你将被杀死(killed)。

带着资源 request 和 limit 运行是一个 "[安全最佳实践](https://kubernetes.io/blog/2016/08/security-best-practices-kubernetes-deployment)"。

> 运行无资源约束的容器的选项使你的系统处于 DoS 或 " 嘈杂的邻居(noisy neighbor)" 场景的风险中。为了防止和减少这些风险，你应该定义资源配额(resource quotas)。

一旦你在 namespace 上设置 quotas，你将被迫对该 namespace 的每个容器应用设置 request 和 limit。

## Container CPU Utilization, Saturation, 和 Errors[](https://ewhisper.cn/posts/65005/#Container-CPU-Utilization-Saturation-%20%E5%92%8C%20-Errors)

对于 CPU 的利用率，Kubernetes 只为每个容器提供了三个指标:

1.  `container_cpu_user_seconds_total` — “user” time 的总数(即: 不在内核中的花费时间)
2.  `container_cpu_system_seconds_total` — “system” time 的总数 (i.e. 在内核中的花费时间)
3.  `container_cpu_usage_seconds_total` — 上述各项的综合. 在 Kubernetes 1.9 之前，所有节点的每个 CPU 都会报告这个情况。这在 1.1.0 中有所改变。

所有这些指标都是计数器，需要有一个 `rate` 应用于它们。这个查询将给我们提供每个容器正在使用的核心数量。对于整个系统中的所有该名称的容器:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sum(<span>rate</span>(<span>container_cpu_usage_seconds_total</span>[<span>5</span>m]))by (<span>container_name</span>)<br></code><p><i></i>LISP</p></pre></td></tr></tbody></table>

[![](https://pic-cdn.ewhisper.cn/img/2022/06/13/1d7e1ea58ee3e5a350871e44c4594a7f-20220613154402.png)](https://pic-cdn.ewhisper.cn/img/2022/06/13/1d7e1ea58ee3e5a350871e44c4594a7f-20220613154402.png)

当使用 CPU limits 运行时，计算饱和度会变得更容易，因为你已经定义了 CPU 使用的上限是什么。当一个容器超过其 CPU 限制时，Linux 运行时将 " 遏制(throttle)" 该容器，并在 `container_cpu_cfs_throttled_seconds_total` 系列中记录它被节制的时间量。这将作为一个计数器再次跟踪每个容器，因此采取`rate`:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sum(<span>rate</span>(<span>container_cpu_cfs_throttled_seconds_total</span>[<span>5</span>m])) by (<span>container_name</span>)<br></code><p><i></i>LISP</p></pre></td></tr></tbody></table>

这是一个重要的指标，在运行时要注意 CPU 的 limit（应该总是这样的！）。

与 node\_exporter 非常相似，cAdvisor 不会暴露 CPU 错误。

## 内存 Utilization, Saturation, 和 Errors[](https://ewhisper.cn/posts/65005/#%E5%86%85%E5%AD%98%20-Utilization-Saturation-%20%E5%92%8C%20-Errors)

在 cAdvisor 中跟踪的内存指标是由 node\_exporter 暴露的 43 个内存指标的一个子集。下面是容器的内存指标:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code>container_memory_cache <span>-- Number of bytes of page cache memory.</span><br>container_memory_rss <span>-- Size of RSS in bytes.</span><br>container_memory_swap <span>-- Container swap usage in bytes.</span><br>container_memory_usage_bytes <span>-- Current memory usage in bytes,       </span><br>                                including <span>all</span> memory regardless <span>of</span><br>                                <span>when</span> it was accessed.<br>container_memory_max_usage_bytes <span>-- Maximum memory usage recorded </span><br>                                    <span>in</span> bytes.<br>container_memory_working_set_bytes <span>-- Current working set in bytes.</span><br>container_memory_failcnt <span>-- Number of memory usage hits limits.</span><br>container_memory_failures_total <span>-- Cumulative count of memory </span><br>                                   allocation failures.<br></code><p><i></i>ADA</p></pre></td></tr></tbody></table>

你可能认为用 `container_memory_usage_bytes` 来跟踪内存利用率是很容易的，然而，这个指标也包括在内存压力下可能被驱逐的缓存(cache)（想想文件系统缓存）项目。更好的指标是`container_memory_working_set_bytes`，因为这就是 OOM killer 所关注的。

为了计算容器的内存利用率，我们使用:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sum(container_memory_working_set_bytes{<span>name</span>!~"POD"})  <span>by</span> (<span>name</span>)<br></code><p><i></i>PGSQL</p></pre></td></tr></tbody></table>

在上面的查询中，我们需要排除名称包含 “POD” 的容器。这是该容器的父组，并将跟踪 pod 中所有容器的统计信息。

容器的内存饱和度在运行内存限制时变得更容易（感觉到这里有一个主题？）我们将把饱和度定义为限制中的可用内存量:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>sum(<span>container_memory_working_set_bytes</span>) by (<span>container_name</span>) / sum(<span>label_join</span>(<span>kube_pod_container_resource_limits_memory_bytes</span>,<br>    <span>"container_name"</span>, <span>""</span>, <span>"container"</span>)) by (<span>container_name</span>)<br></code><p><i></i>LISP</p></pre></td></tr></tbody></table>

这里我们要连接两个系列，一个来自 cAdvisor，一个来自 kube-state-metrics。不幸的是，容器名称的标签没有对齐，但 PromQL 在这里用 `label_join` 来帮助。

内存错误不会被 kubelet 暴露出来。

## Disk Utilization, Saturation, 和 Errors[](https://ewhisper.cn/posts/65005/#Disk-Utilization-Saturation-%20%E5%92%8C%20-Errors)

cAdvisor 有 `container_fs_io_time_seconds_total` 和`container_fs_io_time_weighted_seconds_total`的系列，在处理磁盘 I/O 时，我们首先通过查看读和写来跟踪所有的磁盘利用率。这些应该在节点层面上跟踪类似的指标，但在我的安装中，这些指标总是为零。

最基本的磁盘 I/O 利用率是指读 / 写的字节数:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>sum(<span>rate</span>(<span>container_fs_writes_bytes_total</span>[<span>5</span>m])) by (<span>container_name</span>,device)<br>sum(<span>rate</span>(<span>container_fs_reads_bytes_total</span>[<span>5</span>m])) by (<span>container_name</span>,device)<br></code><p><i></i>LISP</p></pre></td></tr></tbody></table>

将这些加起来，得到每个容器的总体磁盘 I/O 利用率。

kubelet 并没有提供足够的细节来对容器磁盘饱和度或错误进行有意义的查询。

## 网络 Utilization, Saturation, 和 Errors[](https://ewhisper.cn/posts/65005/#%E7%BD%91%E7%BB%9C%20-Utilization-Saturation-%20%E5%92%8C%20-Errors)

在容器级别的网络利用率，你可以选择以字节或数据包的形式来测量发送和接收。这个查询有点不同，因为所有的网络核算都发生在 Pod 层面，而不是在容器层面

这个查询将按 pod 名称显示每个 pod 的网络利用率:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>sum(<span>rate</span>(<span>container_network_receive_bytes_total</span>[<span>5</span>m])) by (<span>name</span>)<br>sum(<span>rate</span>(<span>container_network_transmit_bytes_total</span>[<span>5</span>m])) by (<span>name</span>)<br></code><p><i></i>LISP</p></pre></td></tr></tbody></table>

同样，如果不知道最大的网络带宽是多少，网络的饱和度是无法定义的。cAdvisor 提供了 `container_network_receive_packets_dropped_total` 和`container_network_transmit_packets_dropped_total`。

cAdvisor 还显示了 `container_network_receive_errors_total`和 `container_network_transmit_errors_total`系列的错误数量。

## 结束语[](https://ewhisper.cn/posts/65005/#%E7%BB%93%E6%9D%9F%E8%AF%AD)

通过使用 cAdvisor，kubelet 为你的 Kubernetes 集群中的所有容器提供了大量的资源信息。通过利用率、饱和度和错误来查看这些资源，可以为调查资源限制和容量规划提供起点。