## 前言[](https://ewhisper.cn/posts/33903/#%E5%89%8D%E8%A8%80%20-2)

K8S 性能优化系列文章，本文为第四篇：Kubernetes Node 性能优化参数最佳实践。

系列文章：

1.  [《K8S 性能优化 - OS sysctl 调优》](https://ewhisper.cn/posts/5019/)
2.  [《K8S 性能优化 - K8S APIServer 调优》](https://ewhisper.cn/posts/51187/)
3.  [《K8S 性能优化 - 大型集群 CIDR 配置》](https://ewhisper.cn/posts/21327)

## 两个参数[](https://ewhisper.cn/posts/33903/#%E4%B8%A4%E4%B8%AA%E5%8F%82%E6%95%B0)

控制可以为 K8S Node 调度的最大 pod 数量的两个参数: `podsPerCore` 和 `maxPods`。

当两个参数都被设置时，其中较小的值限制了节点上的 pod 数量。超过这些值可导致：

-   CPU 使用率增加。
-   减慢 pod 调度的速度。
-   根据节点中的内存数量，可能出现内存耗尽的问题。
-   耗尽 IP 地址池。
-   资源过量使用，导致用户应用程序性能变差。

⚠️ **重要**

在 Kubernetes 中，包含单个容器的 pod 实际使用两个容器。第二个容器用来在实际容器启动前设置联网(pause)。因此，运行 10 个 pod 的系统实际上会运行 20 个容器。

`podsPerCore` 根据节点中的处理器内核数来设置节点可运行的 pod 数量。例如：在一个有 4 个处理器内核的节点上将 `podsPerCore` 设为 10 ，则该节点上允许的最大 pod 数量为 40。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>kubeletConfig:</span><br>  <span>podsPerCore:</span> <span>10</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

将 `podsPerCore` 设置为 `0` 可禁用这个限制。默认为 `0`。`podsPerCore` 不能超过 `maxPods`。

`maxPods` 把节点可以运行的 pod 数量设置为一个固定值，而不需要考虑节点的属性。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>kubeletConfig:</span><br>   <span>maxPods:</span> <span>250</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

EOF