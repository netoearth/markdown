译者注：

这篇文章很全面的罗列出了 [[Kubernetes]] 中涉及的网络知识，从 Linux 内核的网络内容，到容器、Kubernetes，一一进行了详细的说明。

文章篇幅有点长，不得不说，网络是很复杂很麻烦的一层，但恰恰这层多年来变化不大。希望翻译的内容对大家能有所帮助，有误的地方，也欢迎大家指正。

本文翻译获得 Learnk8s 的授权，原文 [Tracing the path of network traffic in Kubernetes](https://learnk8s.io/kubernetes-network-packets) 作者 Kristijan Mitevski。

___

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427680911539.jpg)

**TL;DR：** _本文将代理了解 Kubernetes 集群内外的数据流转。从最初的 Web 请求开始，一直到托管应用程序的容器。_

## 目录

-   [目录](https://atbug.com/#%E7%9B%AE%E5%BD%95)
-   [Kubernetes 网络要求](https://atbug.com/#kubernetes-%E7%BD%91%E7%BB%9C%E8%A6%81%E6%B1%82)
-   [Linux 网络命名空间如果在 pod 中工作](https://atbug.com/#linux-%E7%BD%91%E7%BB%9C%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E5%A6%82%E6%9E%9C%E5%9C%A8-pod-%E4%B8%AD%E5%B7%A5%E4%BD%9C)
-   [Pause 容器创建 Pod 中的网络命名空间](https://atbug.com/#pause-%E5%AE%B9%E5%99%A8%E5%88%9B%E5%BB%BA-pod-%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9C%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4)
-   [为 Pod 分配了 IP 地址](https://atbug.com/#%E4%B8%BA-pod-%E5%88%86%E9%85%8D%E4%BA%86-ip-%E5%9C%B0%E5%9D%80)
-   [检查集群中 pod 到 pod 的流量](https://atbug.com/#%E6%A3%80%E6%9F%A5%E9%9B%86%E7%BE%A4%E4%B8%AD-pod-%E5%88%B0-pod-%E7%9A%84%E6%B5%81%E9%87%8F)
-   [Pod 命名空间连接到以太网桥接器](https://atbug.com/#pod-%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E8%BF%9E%E6%8E%A5%E5%88%B0%E4%BB%A5%E5%A4%AA%E7%BD%91%E6%A1%A5%E6%8E%A5%E5%99%A8)
-   [跟踪同一节点上 pod 间的流量](https://atbug.com/#%E8%B7%9F%E8%B8%AA%E5%90%8C%E4%B8%80%E8%8A%82%E7%82%B9%E4%B8%8A-pod-%E9%97%B4%E7%9A%84%E6%B5%81%E9%87%8F)
-   [跟踪不同节点上 pod 间的通信](https://atbug.com/#%E8%B7%9F%E8%B8%AA%E4%B8%8D%E5%90%8C%E8%8A%82%E7%82%B9%E4%B8%8A-pod-%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1)
    -   [位运算的工作原理](https://atbug.com/#%E4%BD%8D%E8%BF%90%E7%AE%97%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86)
-   [容器网络接口 - CNI](https://atbug.com/#%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E6%8E%A5%E5%8F%A3---cni)
-   [检查 pod 到服务的流量](https://atbug.com/#%E6%A3%80%E6%9F%A5-pod-%E5%88%B0%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%B5%81%E9%87%8F)
-   [使用 Netfilter 和 Iptables 拦截和重写流量](https://atbug.com/#%E4%BD%BF%E7%94%A8-netfilter-%E5%92%8C-iptables-%E6%8B%A6%E6%88%AA%E5%92%8C%E9%87%8D%E5%86%99%E6%B5%81%E9%87%8F)
-   [检查服务的响应](https://atbug.com/#%E6%A3%80%E6%9F%A5%E6%9C%8D%E5%8A%A1%E7%9A%84%E5%93%8D%E5%BA%94)
-   [回顾](https://atbug.com/#%E5%9B%9E%E9%A1%BE)

## Kubernetes 网络要求

在深入了解 Kubernetes 中的数据流转之前，让我们先澄清下 Kubernetes 网络的要求。

Kubernetes 网络模型定义了一套基本规则：

-   **集群中的 pod 应该能够与任何其他 pod 自由通信**，而无需使用网络地址转换（NAT）。
-   在不使用 NAT 的情况下，**集群节点上运行的任意程序都应该能够与同一节点上的任意 pod 通信**。
-   **每个 pod 都有自己的 IP 地址**（IP-per Pod），其他 pod 都可以使用同一个地址进行访问。

这些要求不会将实现限制在单一方案上。

相反，他们概括了集群网络的特性。

在满足这些限制时，必须解决如下[挑战](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/network/networking.md)：

1.  _如何保证同一 pod 中的容器间的访问就像在同一主机上一样？_
2.  _Pod 能否访问集群中的其他 pod？_
3.  _Pod 能否访问服务（service）？以及服务可以负载均衡请求吗？_
4.  _Pod 可以接收来自集群外的流量吗？_

本文将专注于前三点，从 pod 内部网络或者容器间的通信说起。

## Linux 网络命名空间如果在 pod 中工作

我们想象下，有一个承载应用程序的主容器和另一个与它一起运行的容器。

在示例 Pod 中有一个 Nginx 容器和 busybox 容器：

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: container-1
      image: busybox
      command: ['/bin/sh', '-c', 'sleep 1d']
    - name: container-2
      image: nginx
```

在部署时，会出现如下情况：

1.  Pod 在节点上得到**自己的网络命名空间**。
2.  Pod **分配到一个 IP 地址**，两个容器间共享端口。
3.  **两个容器共享同一个网络命名空间**，在本地互相可见。

网络配置在后台很快完成。

然后，我们退后一步，是这理解_为什么_上面是容器运行所必须的。

[在 Linux 中，网络命名空间是独立的、隔离的逻辑空间。](https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/)

可以将网络命名空间堪称将物理网络接口分割成更小的独立部分。

每部分都可以单独配置，并使用自己的网络规则和资源。

这些可以包括防火墙规则、接口（虚拟或物理）、路由和其他所有与网络相关的内容。

1.  物理接口持有根命名空间。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427699864958.jpg)
    
2.  可以使用 Linux 网络命名空间创建隔离的网络。每个网络都是独立的，除非进行配置否则不会与其他命名空间通信。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427700613518.jpg)
    

物理接口必须处理最后的所有_真实_数据包，因此所有的虚拟接口都是从中创建的。

网络命名空间可以通过 [`ip-netns` 管理工具](https://man7.org/linux/man-pages/man8/ip-netns.8.html) 来管理，可以使用 `ip netns list` 列出主机上的命名空间。

> 请注意，创建的网络命名空间将会出现在 `/var/run/netns` 目录下，但 [Docker 并没有遵循这一点](https://www.packetcoders.io/how-to-view-the-network-namespaces-in-kubernetes/)。

例如，下面是 Kubernetes 节点的命名空间：

```
$ ip netns list
cni-0f226515-e28b-df13-9f16-dd79456825ac (id: 3)
cni-4e4dfaac-89a6-2034-6098-dd8b2ee51dcd (id: 4)
cni-7e94f0cc-9ee8-6a46-178a-55c73ce58f2e (id: 2)
cni-7619c818-5b66-5d45-91c1-1c516f559291 (id: 1)
cni-3004ec2c-9ac2-2928-b556-82c7fb37a4d8 (id: 0)
```

> 注意 `cni-` 前缀意味着命名空间的创建由 CNI 来完成。

当创建 pod 并分配给节点时，[CNI](https://github.com/containernetworking/cni#what-is-cni) 会：

1.  为其创建网络命名空间。
2.  分配 IP 地址。
3.  将容器连接到网络。

如果 pod 像上面的示例一样包含多个容器，则所有容器都被置于同一个命名空间中。

1.  创建 pod 时，CNI 为容器创建网络命名空间
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427706567489.jpg)
    
2.  然后分配 IP 地址
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427707025246.jpg)
    
3.  最后将容器连接到网络的其余部分
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427710287189.jpg)
    

_那么当列出节点上的容器时会看到什么？_

可以 SSH 到 Kubernetes 节点来查看命名空间：

```
$ lsns -t net
        NS TYPE NPROCS   PID USER     NETNSID NSFS                           COMMAND
4026531992 net     171     1 root  unassigned /run/docker/netns/default      /sbin/init noembed norestore
4026532286 net       2  4808 65535          0 /run/docker/netns/56c020051c3b /pause
4026532414 net       5  5489 65535          1 /run/docker/netns/7db647b9b187 /pause

```

`lsns` 命令会列出主机上_所有_的命名空间。

> 记住 Linux 中有[多种命名空间类型](https://man7.org/linux/man-pages/man7/namespaces.7.html)。

_Nginx 容器在哪？_

_那么 `pause` 容器又是什么？_

## Pause 容器创建 Pod 中的网络命名空间

从节点上的所有进程中找出 Nginx 容器：

```
$ lsns
        NS TYPE   NPROCS   PID USER            COMMAND
# truncated output
4026532414 net         5  5489 65535           /pause
4026532513 mnt         1  5599 root            sleep 1d
4026532514 uts         1  5599 root            sleep 1d
4026532515 pid         1  5599 root            sleep 1d
4026532516 mnt         3  5777 root            nginx: master process nginx -g daemon off;
4026532517 uts         3  5777 root            nginx: master process nginx -g daemon off;
4026532518 pid         3  5777 root            nginx: master process nginx -g daemon off;
```

该容器出现在了挂在（mount `mnt`）、Unix 分时系统（Unix time-sharing `uts`）和 PID（`pid`）命名空间中，但是并不在网络命名空间（`net`）中。

不幸的是，`lsns` 只显示了每个进程最低的 PID，不过可以根据进程 ID 进一步过滤。

可以通过以下内容检索Nginx 容器的所有命名空间：

```
$ sudo lsns -p 5777
       NS TYPE   NPROCS   PID USER  COMMAND
4026531835 cgroup    178     1 root  /sbin/init noembed norestore
4026531837 user      178     1 root  /sbin/init noembed norestore
4026532411 ipc         5  5489 65535 /pause
4026532414 net         5  5489 65535 /pause
4026532516 mnt         3  5777 root  nginx: master process nginx -g daemon off;
4026532517 uts         3  5777 root  nginx: master process nginx -g daemon off;
4026532518 pid         3  5777 root  nginx: master process nginx -g daemon off;
```

`pause` 进程再次出现，这次它劫持了网络命名空间。

_那是什么？_

**集群中的每个 pod 都有一个在后台运行的隐藏容器，被称为 `pause`**。

列出节点上的所有容器并过滤出 pause 容器：

```
$ docker ps | grep pause
fa9666c1d9c6   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_kube-dns-599484b884-sv2js…
44218e010aeb   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_blackbox-exporter-55c457d…
5fb4b5942c66   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_kube-dns-599484b884-cq99x…
8007db79dcf2   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_konnectivity-agent-84f87c…
```

将看到对于节点分配到的每个 pod，都有一个匹配的 `pause` 容器。

**该 `pause` 容器负责创建和维持网络命名空间。**

它包含的代码极少，部署后立即进入睡眠状态。

然而，[它在 Kubernetes 生态中的首当其冲，发挥着至关重要的作用。](https://www.ianlewis.org/en/almighty-pause-container)。

1.  创建 pod 时，CNI 会创建一个带有_睡眠_容器的网络命名空间
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427720335296.jpg)
    
2.  Pod 中的所有容器都会加入到它创建的网络命名空间中
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427721343650.jpg)
    
3.  此时 CNI 分配 IP 地址并将容器连接到网络
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427710287189.jpg)
    

_进入睡眠状态的容器有什么用？_

要了解它的实用性，我们可以想象下如示例一样有两个容器的 pod，但没有 `pause` 容器。

容器启动，CNI：

1.  为 Nginx 容器创建一个网络命名空间。
2.  把 busybox 容器加入到前面创建的网络命名空间中。
3.  为 pod 分配 IP 地址。
4.  将容器连接到网络。

_假如 Nginx 容器崩溃了会发生什么？_

CNI 将不得不_再次_完成所有流程，两个容器的网络都会中断。

由于 `sleep` 容器不太可能有任何 bug，因此创建网络命名空间通常是一个更保险、更健壮的选择。

**如果 pod 中的一个容器崩溃，其余的仍可以处理网络请求。**

## 为 Pod 分配了 IP 地址

前面提到 pod 和所有容器获得了同样的 IP。

`这是怎么配置的？`

__在 pod 网络命名空间中，创建一个接口并分配 IP 地址_。_

我们来验证下。

首先，找到 pod 的 IP 地址：

```
$ kubectl get pod multi-container-pod -o jsonpath={.status.podIP}
10.244.4.40
```

接下来，找到相关的网络命名空间。

由于网络命名空间是从物理接口创建的，需要访问集群节点。

> 如果你运行的是 minikube，可以通过 `minikube ssh` 访问节点。如果在云提供商中运行，应该有某种方法通过 SSH 访问节点。

进入后，可以找到创建的最新的网络命名空间：

```
$ ls -lt /var/run/netns
total 0
-r--r--r-- 1 root root 0 Sep 25 13:34 cni-0f226515-e28b-df13-9f16-dd79456825ac
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-4e4dfaac-89a6-2034-6098-dd8b2ee51dcd
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-7e94f0cc-9ee8-6a46-178a-55c73ce58f2e
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-7619c818-5b66-5d45-91c1-1c516f559291
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-3004ec2c-9ac2-2928-b556-82c7fb37a4d8
```

本示例中，它是 `cni-0f226515-e28b-df13-9f16-dd79456825ac`。此时，可以在该命名空间总执行 `exec` 命令：

```
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac ip a
# output truncated
3: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
    link/ether 16:a4:f8:4f:56:77 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.4.40/32 brd 10.244.4.40 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::14a4:f8ff:fe4f:5677/64 scope link
       valid_lft forever preferred_lft forever
```

**`10.244.4.40` 就是 pod 的 IP 地址。**

通过查找 `@if12` 中的 `12` 找到网络接口。

```
$ ip link | grep -A1 ^12
12: vethweplb3f36a0@if16: mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default
    link/ether 72:1c:73:d9:d9:f6 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

还可以验证 Nginx 容器是否从该命名空间中监听 HTTP 流量：

```
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      692698/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      692698/nginx: master
```

> 如果无法通过 SSH 访问集群的节点，可以试试 `kubectl exec` 进入到 busybox 容器，然后使用 `ip` 和 `netstat` 命令。

_太棒了！_

现在我们已经介绍了容器间的通信，接下来看看 Pod 与 Pod 直接如何建立通信。

## 检查集群中 pod 到 pod 的流量

当说起 pod 间通信时，会有两种可能：

1.  Pod 流量流向同一节点上的 pod。
2.  Pod 流量流量另一个节点上的 pod。

为了使整个设置正常工作，我们需要之前讨论过的虚拟接口和以太网桥接。

在继续之前，我们先讨论下他们的功能以及为什么他们时必需的。

**要完成 pod 与其他 pod 的通信，它必须先访问节点的根命名空间。**

这是使用连接 pod 和根命名空间的虚拟以太网对来实现的。

这些[虚拟接口设备](https://man7.org/linux/man-pages/man4/veth.4.html)（`veth` 中的 `v`）连接并充当两个命名空间间的隧道。

使用此 `veth` 设备，将一端连接到 pod 的命名空间，另一端连接到根命名空间。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427710287189.jpg)

这些 CNI 可以替你做，也可以手动操作：

```
$ ip link add veth1 netns pod-namespace type veth peer veth2 netns root
```

现在 pod 的命名空间有了可以访问根命名空间的隧道。

**节点上每个新建的 pod 都会设置如下所示的 veth 对。**

创建接口对时其中一部分。

其他的就是为以太网设备分配地址，并创建默认路由。

来看下如何在 pod 的命名空间中设置 `veth1` 接口：

```
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac ip addr add 10.244.4.40/24 dev veth1
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac ip link set veth1 up
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac ip route add default via 10.244.4.40
```

在节点侧，我们创建另一个 `veth2` 对：

```
$ ip addr add 169.254.132.141/16 dev veth2
$ ip link set veth2 up
```

可以像以前一样检查现有的 `veth` 对。

在 pod 的命名空间中，检查 `eth0` 接口的后缀。

```
$ ip netns exec cni-0f226515-e28b-df13-9f16-dd79456825ac ip link show type veth
3: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default
    link/ether 16:a4:f8:4f:56:77 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

这种情况下可以使用 `grep -A1 ^12` 进行查找（或者滚动到目标所在）：

```
$ ip link show type veth
# output truncated
12: cali97e50e215bd@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-0f226515-e28b-df13-9f16-dd79456825ac
```

> 也可以使用 `ip -n cni-0f226515-e28b-df13-9f16-dd79456825ac link show type veth` 命令。

注意 `3: eth0@if12` 和 `12: cali97e50e215bd@if3` 接口上的符号。

在 pod 命名空间中，`eth0` 接口连接到根命名空间中编号为 `12` 的接口。因此是 `@if12`。

在 `veth` 对的另一端，根命名空间连接到 pod 命名空间的 `3` 号接口。

接下来是连接 `veth` 对两端的桥接器（bridge）。

## Pod 命名空间连接到以太网桥接器

桥接器将位于根命名空间中的虚拟接口的每一端“绑定”。

**该桥接器将允许流量在虚拟对之间流动，并通过公共根命名空间。**

_理论时间。_

以太网桥接器位于[OSI 网络模型](https://en.wikipedia.org/wiki/OSI_model)的第二层。

[可以将桥接器看作一个虚拟交换机，接受来自不同命名空间和接口的连接。](https://ops.tips/blog/using-network-namespaces-and-bridge-to-isolate-servers/)

**以太网桥接器允许连接同一个节点上的多个可用网络。**

因此，可以使用该设置连接两个接口：从 pod 命名空间的 `veth` 连接到同一节点上另一个 pod 的 `veth`。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427769020626.jpg)

我们继续看下以太网桥接器和 veth 对的作用。

## 跟踪同一节点上 pod 间的流量

假设同一个节点上有两个 pod，Pod-A 想向 Pod-B 发送消息。

1.  由于目标不是同命名空间的容器，Pod-A 向其默认接口 `eth0` 发送数据包。这个接口与 `veth` 对的一端绑定，作为隧道。因此数据包将被转发到节点的根命名空间。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427770131482.jpg)
    
2.  以太网桥接器作为虚拟交换机，必须以某种方式将目标 pod IP（Pod-B）解析为其 MAC 地址。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427772473425.jpg)
    
3.  轮到ARP 协议上场了。当帧到达桥接器时，会向所有连接的设备发送 ARP 广播。桥接器喊道_谁有 Pod-B 的 IP 地址_。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427774225871.jpg)
    
4.  收到带有连接 Pod-B 接口的 MAC 地址的回复，然后此信息存储在桥接器 ARP 缓存（查找表）中。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427775056174.jpg)
    
5.  IP 和 MAC 地址的映射存储完成后，桥接器在表中查找，并将数据包转发到正确的短点。数据包到达根命名空间中 Pod- B 的 `veth`，然后从那快速到达 Pod-B 命名空间内的 `eth0` 接口。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427775056174.jpg)
    

有了这个，Pod-A 和 Pod-B 之间的通信取得了成功。

## 跟踪不同节点上 pod 间的通信

对于需要跨不同节点通信的 pod，需要额外的通信跳转。

1.  前几个步骤保持不变，直到数据包到达根命名空间并需要发送到 Pod- B。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427777182601.jpg)
    
2.  当目标地址不在本地网络中，数据包将被转发到本节点的默认网关。节点上退出或默认网关通常位于 `eth0` 接口上 – 将节点连接到网络的物理接口。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16427777968858.jpg)
    

**这次并不会发生 ARP 解析，因为源和目标 IP 在不同网络上。**

检查使用位运算（Bitwise）操作完成。

当目标 IP 不在当前网络上时，数据包将被转发到节点的默认网关。

### 位运算的工作原理

在确定数据包的转发位置时，源节点必须执行位运算。

[这也被称为与操作。](https://en.wikipedia.org/wiki/Bitwise_operation#AND)

作为复习，位与操作产生如下结果：

```
0 AND 0 = 0
0 AND 1 = 0
1 AND 0 = 0
1 AND 1 = 1
```

除了 1 与 1 以外的都是 false。

如果源节点的 IP 为 192.168.1.1，子网掩码为 /24，目标 IP 为 172.16.1.1/16，则按位与操作将确认他们不在同一网络上。

这意味着目标 IP 与数据包的源在不同的网络上，因此数据包将在默认网关中转发。

_数学时间。_

我们必须从二进制文件中的 32 位地址执行与操作开始。

先找出源和目标 IP 的网络。

```
| Type             | Binary                              | Converted          |
| ---------------- | ----------------------------------- | ------------------ |
| Src. IP Address  | 11000000.10101000.00000001.00000001 | 192.168.1.1        |
| Src. Subnet Mask | 11111111.11111111.11111111.00000000 | 255.255.255.0(/24) |
| Src. Network     | 11000000.10101000.00000001.00000000 | 192.168.1.0        |
|                  |                                     |                    |
| Dst. IP Address  | 10101100.00010000.00000001.00000001 | 172.16.1.1         |
| Dst. Subnet Mask | 11111111.11111111.00000000.00000000 | 255.255.0.0(/16)   |
| Dst. Network     | 10101100.00010000.00000000.00000000 | 172.16.0.0         |
```

位运算操作后，需要将目标 IP 与数据包源节点的子网进行比较。

```
| Type             | Binary                              | Converted          |
| ---------------- | ----------------------------------- | ------------------ |
| Dst. IP Address  | 10101100.00010000.00000001.00000001 | 172.16.1.1         |
| Src. Subnet Mask | 11111111.11111111.11111111.00000000 | 255.255.255.0(/24) |
| Network  Result  | 10101100.00010000.00000001.00000000 | 172.16.1.0  
```

进行位比较后，ARP 会检查其查询表来查找默认网关的 MAC 地址。

如果有条目，将立即转发数据包。

否则，先进行广播以确定网关的 MAC 地址。

1.  数据包现在路由到另一个节点的默认接口，我们叫它 Node-B。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428091589748.jpg)
    
2.  以相反的顺序。数据包现在位与 Node-B 的根命名空间，并到达桥接器，这里会进行 ARP 解析。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428095883798.jpg)
    
3.  收到带有连接 Pod-B 的接口 MAC地址的回复。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428097041936.jpg)
    
4.  这次桥接器通过 Pod-B 的 `veth` 设备将帧转发，并到达 Pod-B 自己的命名空间。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428097041936.jpg)
    

此时应该已经熟悉了 pod 之间的流量如何流转，接下来再探索下 CNI 如何创建上述内容。

## 容器网络接口 - CNI

[容器网络接口（CNI）关注当前节点的网络。](https://github.com/containernetworking/cni/blob/master/SPEC.md)

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428099347551.jpg)

**可以将 CNI 看作网络插件在解决 Kubernetes _某些_ 需求时要遵循的一套规则。**

然而，它不仅仅与 Kubernetes 或者特定网络插件关联。

可以使用如下 CNI：

-   [Calico](https://www.tigera.io/project-calico/)
-   [Cilium](https://cilium.io/)
-   [Flannel](https://github.com/flannel-io/flannel)
-   [Weave Net](https://www.weave.works/docs/net/latest/overview/)
-   [其他网络插件](https://github.com/containernetworking/cni#3rd-party-plugins)

他们都实现相同的 CNI 标准。

如果没有 CNI，你需要手动完成如下操作：

-   创建 pod（容器）的网络命名空间
-   创建接口
-   创建 veth 对
-   设置命名空间网络
-   设置静态路由
-   配置以太网桥接器
-   分配 IP 地址
-   创建 NAT 规则

还有太多其他需要手动完成的工作。

更不用说删除或重新启动 pod 时删除或调整上述所有内容了。

CNI 必须支持[四个不同的操作](https://github.com/containernetworking/cni/blob/master/SPEC.md#cni-operations)：

-   **ADD** - 将容器添加到网络
-   **DEL** - 从网络中删除容器
-   **CHECK** - 如果容器的网络出现问题，则返回错误
-   **VERSION** - 显示插件的版本

_让我们在实践中看看它是如何工作的。_

当 pod 分配到特定节点时，kubelet 本身不会初始化网络。

相反，它将任务交给了 CNI。

**然后，它指定了配置，并以 JSON 格式将其发送给 CNI 插件。**

可以在节点的 `/etc/cni/net.d` 目录中，找到当前 CNI 的配置文件：

```
$ cat 10-calico.conflist
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "datastore_type": "kubernetes",
      "mtu": 0,
      "nodename_file_optional": false,
      "log_level": "Info",
      "log_file_path": "/var/log/calico/cni/cni.log",
      "ipam": { "type": "calico-ipam", "assign_ipv4" : "true", "assign_ipv6" : "false"},
      "container_settings": {
          "allow_ip_forwarding": false
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "k8s_api_root":"https://10.96.0.1:443",
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    },
    {"type": "portmap", "snat": true, "capabilities": {"portMappings": true}}
  ]
}
```

**每个插件都使用不同类型的配置来设置网络。**

例如，Calico 使用 BGP 路由协议配对的第 3 层网络来连接 pod。

Cilium 在第 3 到 7 层使用 eBPF 配置覆盖网络。

与 Calico 一起，Cilium 支持设置网络策略来限制流量。

_那该如何选择呢？_

这取决于。

CNI 主要有两组。

**第一组中，可以找到使用基本网络设置（也称为扁平网络）的CNI**，并将集群 IP 池 中的IP 地址分配给 pod。

这可能会因为快速用尽可用的 IP 地址而成为负担。

**相反，另一种方法是使用覆盖网络。**

简而言之，覆盖网络是主（底层）网络之上的辅助网络。

**覆盖网络的工作原理是封装来自底层网络的所有数据包，这些数据包指向另一个节点上的 pod。**

覆盖网络的一项流行技术是 [VXLAN](https://en.wikipedia.org/wiki/Virtual_Extensible_LAN)，它允许在 L3 网络上隧道传输 L2 域。

_那么哪种更好？_

**没有唯一的答案，通常取决于你的需求。**

_你是在构建一个拥有数万个节点的大集群吗？_

可能覆盖网络更好。

_你是否在意更简单的设置和在嵌套网络中不失去检查网络流量的能力。_

扁平网络更适合你。

现在已经讨论了 CNI，让我们继续探索 Pod 到服务（service）的通信是如何完成的。

## 检查 pod 到服务的流量

由于 Kubernetes 环境下 pod 的动态特性，分配给 pod 的 IP 地址不是静态的。

**这些 IP 地址是短暂的，每次创建或者删除 pod 时都会发生变化。**

服务解决了这个问题，为连接到一组 pod 提供了稳定的机制。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428122010963.jpg)

默认情况下，在 Kubernetes 中创建服务时，会[为其预定并分配虚拟 IP](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies) 。

使用选择器将服务于目标 pod 进行管理。

_当删除 pod 并添加新 pod 时会发生什么？_

**该服务的虚拟 IP 保持不变。**

然而，无需敢于，流量将到达新创建的 pod。

换句话说，Kubernetes 中的服务类似于负载均衡器。

_但他们时如何工作的？_

## 使用 Netfilter 和 Iptables 拦截和重写流量

Kubernetes 中的服务基于两个 Linux 内核组件：

1.  Netfilter
2.  Iptables

**Netfilter 是一个框架，允许配置数据包过滤、创建 NAT或端口翻译规则，并管理网络中的流量。**

此外，它还屏蔽和阻止不请自来的连接访问服务。

**另一方面，Iptables 是一个用户空间程序，允许你配置 Linux 内核防火墙的 IP 数据包过滤器规则。**

iptables 使用不同的 Netfilter 模块实现。

你可以使用 iptables CLI 实时更改过滤规则，并将其插入 netfilters 的挂点。

过滤器组织在不同的表中，其中包含处理网络流量数据包的链。

每个协议都使用不同的内核模块和程序。

> 当提到 iptables 时，通常说的是 IPV4。对于 IPV6 的规则，CLI 是 ip6tables。

Iptables 有五种类型的链，每种链都直接映射到 Netfilter 钩子。

从 iptables 角度看是：

-   `PRE_ROUTING`
-   `INPUT`
-   `FORWARD`
-   `OUTPUT`
-   `POST_ROUTING`

对应映射到 Netfilter 钩子：

-   `NF_IP_PRE_ROUTING`
-   `NF_IP_LOCAL_IN`
-   `NF_IP_FORWARD`
-   `NF_IP_LOCAL_OUT`
-   `NF_IP_POST_ROUTING`

当数据包到达时，根据所处的阶段，会“出发” Netfilter 钩子，该钩子应用特定的 iptables 过滤。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428137183275.jpg)

_哎呀，看起来很复杂！_

不过不需要担心。

这就是为什么我们使用 Kubernetes，上面的所有内容都是通过使用服务来抽象的，一个简单的 YAML 定义就可以自动完成这些规则的设置。

如果对这些 iptables 规则感兴趣，可以登陆到节点并运行：

也可以使用[可视化工具](https://github.com/Nudin/iptable_vis)浏览节点上的 iptables 链。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428153624915.jpg)

_记住，可能会有数百条规则。想象下手动创建的难度。_

我们已经解释了相同和不同节点上的 pod 间如何通信。

在 Pod-to-Service 中，通信的前半部分保持不变。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428154678475.jpg)

当 Pod-A 发出请求时，希望到达 Pod-B（这种情况下，Pod-B 位与服务之后），转移的过程中会发生其他变化。

原始请求从 Pod-A 命名空间中的 `eth0` 接口出来。

从那里穿过 `veth` 对，到达根命名空间的以太网桥。

一旦到达桥接器，数据包立即通过默认网关转发。

与 Pod-to-Pod 部分一样，主机进行位比较，由于服务的 vIP 不是节点 CIDR 的一部分，数据包将立即通过默认网关转发出去。

如果查找表中尚没有默认网关的 MAC 地址，则会进行相同的 ARP 解析。

_现在魔法发生了。_

在数据包经过节点的路由处理之前，Netfilter 钩子 `NF_IP_PRE_ROUTING` 被触发并应用一条 iptables 规则。规则进行了 DNAT 转换，重写了 POD-A 数据包的目标 IP 地址。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428159345915.jpg)

原来服务 vIP 地址被重写称 POD-B 的IP 地址。

从那里，路由就像 Pod-to-Pod 直接通信一样。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428160707534.jpg)

然而，在所有这些通信之间，使用了第三个功能。

[这个功能被称为 conntrack](https://www.linuxtopia.org/Linux_Firewall_iptables/x1298.html)，或连接跟踪。

**Conntrack 将数据包与连接关联起来，并在 Pod-B 发送回响应时跟踪其来源。**

NAT 严重依赖 contrack 工作。

如果没有连接跟踪，它将不知道将包含响应的数据包发送回哪里。

使用 conntrack 时，数据包的返回路径可以轻松设置相同的源或目标 NAT 更改。

另一半使用相反的顺序执行。

Pod-B 接收并处理了请求，现在将数据发送回 Pod-A。

_此时会发生什么？_

## 检查服务的响应

现在 Pod-B 发送响应，将其 IP 地址设置为源地址，Pod-A IP 地址设置为目标地址。

1.  当数据包到达 Pod-A 所在节点的接口时，就会发生另一个 NAT
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428165510048.jpg)
    
2.  这次，使用 conntrack 更改源 IP 地址，iptables 规则执行 SNAT 将 Pod-B IP 地址替换为原始服务的 VIP 地址。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428166224192.jpg)
    
3.  从 Pod-A 来看像是服务发回的响应，而不是 Pod-B。
    
    ![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/22/16428166224192.jpg)
    

其他部分都一样；一旦 SNAT 完成，数据包到达根命名空间中的以太网桥接器，并通过 veth 对转发到 Pod-A。

## 回顾

让我们来总结下你在本文中学到的东西：

-   容器如何在本地或 pod 内通信。
-   当 pod 位于相同和不同的节点上时，Pod-to- Pod 如何通信。
-   Pod-to-Service - 当 pod 向 Kubernetes 服务背后的 pod 发送流量时。
-   Kubernetes 网络工具箱中有效通信所需的命名空间、veth、iptables、链、Netfilter、CNI、覆盖网络以及所有其他内容。