![file](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/14/16dc9c2ca14e12e4~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image "file")

本文首发于：微信公众号「运维之美」，公众号 ID：Hi-Linux。

「运维之美」是一个有情怀、有态度，专注于 Linux 运维相关技术文章分享的公众号。公众号致力于为广大运维工作者分享各类技术文章和发布最前沿的科技信息。公众号的核心理念是：分享，我们认为只有分享才能使我们的团体更强大。如果你想第一时间获取最新技术文章，欢迎关注我们！

公众号作者 Mike，一个月薪 3000 的杂工。从事 IT 相关工作 15+ 年，热衷于互联网技术领域，认同开源文化，对运维相关技术有自己独特的见解。很愿意将自己积累的经验、心得、技能与大家分享交流，篇篇干货不要错过哟。如果你想联系到我，可关注公众号获取相关信息。

前段时间，我们在 「[使用 Kind 在 5 分钟内快速部署一个 Kubernetes 高可用集群](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI3MTI2NzkxMA%3D%3D%26mid%3D2247488666%26idx%3D1%26sn%3Dde46a87cf59f8564ad75337b7c0cf7f6%26chksm%3Deac535b3ddb2bca5de2a923ec6cfa427e87f0135b31193efd1b904dff4c7b473f1dc9af82903%26token%3D212816651%26lang%3Dzh_CN%23rd "https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247488666&idx=1&sn=de46a87cf59f8564ad75337b7c0cf7f6&chksm=eac535b3ddb2bca5de2a923ec6cfa427e87f0135b31193efd1b904dff4c7b473f1dc9af82903&token=212816651&lang=zh_CN#rd")」一文中介绍了如何使用 `Kind` 这个开箱即可快速部署 `Kubernetes` 高可用集群的神器，相信不少同学用上这个神器后大大的降低了 `Kubernetes` 集群的部署难度和提高了 `Kubernetes` 集群的部署速度。不过有一点比较遗憾的是 `Kind` 当前仅仅支持在本地快速构建一个开发或者测试环境，目前暂时还是不支持在生产环境中部署 `Kubernetes` 高可用集群的。

今天，我们就要给大家介绍另一款可以支持在生产环境中部署 `Kubernetes` 高可用集群的利器 `Sealos`。

## 什么是 Sealos ？

`Sealos` 是一个 Go 语言开发的简单干净且轻量的 `Kubernetes` 集群部署工具，`Sealos` 能很好的支持在生产环境中部署高可用的 `Kubernetes` 集群。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/14/16dc9c2d4fd1ece1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

`Sealos` 架构图

**Sealos 特性与优势**

1.  支持离线安装，工具与部署资源包分离，方便不同版本间快速升级。
2.  证书有效期默认延期至 99 年。
3.  工具使用非常简单。
4.  支持使用自定义配置文件，可灵活完成集群环境定制。
5.  使用内核进行本地负载，稳定性极高，故障排查也极其简单。

## Sealos 设计原则和工作原理

### 1\. 为什么不使用 Ansilbe 实现

`Sealos 1.0` 版本时是使用 `Ansible` 实现的，这样在使用时就必须先安装 `Ansible` 及一些 `Python` 的依赖包和进行一些必须的相关环境配置，使用起来还是比较复杂的。

为了解决这个问题，目前新版本的 `Sealos` 采用二进制文件方式提供。新版本 `Sealos` 没有任何依赖，开箱即用。

> 文件分发与远程命令都通过调用对应 `SDK` 实现，不依赖其它任何环境。

### 2\. 为什么不用 KeepAlived 和 HAProxy 实现集群高可用

无论是通过 `KeepAlived` 还是 `HAProxy` 进行高可用集群调度都会存在以下一些劣势。

1.  软件源不一致可能导致容器中安装的软件版本也不一致，进而会引起相应检查脚本不生效等故障。
2.  可能因为系统依赖库问题，在某些特定环境下就直接无法完成安装。
3.  只依靠检测 `HAProxy` 进程是否存活是无法保证集群高可用的，正确的检测方式应该是判断 `ApiServer` 是否 `healthz` 状态。
4.  `Keepalived` 可能存在 `Cpu` 占满的情况。

### 3\. 本地负载为什么不使用 Envoy 或者 Nginx 实现

`Sealos` 高可用实现是通过本地负载方式完成的。本地负载实现方式有多种，比如：`IPVS`、`Envoy`、`Nginx` 等，而 `Sealos` 采用的是通过内核 `IPVS` 来实现的。

> 本地负载：在每个 `Node` 节点上都启动一个负载均衡，同时监听集群中的多个 `Master` 节点。

`Sealos` 选择通过内核 `IPVS` 来实现主要有以下几个原因：

-   如果使用 `Envoy` 等需要在每个节点上都跑一个进程，消耗更多资源。虽然 `IPVS` 实际上也会多跑一个 `lvscare` 进程 ，但是 `lvscare` 只是负责管理 `IPVS` 规则，原理和 `Kube-Proxy`类似。真正的流量直接从内核层面走，不需要把数据包先走到用户态中去处理。
-   使用 `Envoy` 存在启动优先级的问题，比如：Join 集群时，如果负载均衡没有建立，Kubelet 就会启动失败。使用 `IPVS` 则不会存在这样的问题，因为我们可以在 Join 集群前先建立好转发规则。

**3.1 本地内核负载工作原理**

`Sealos` 通过本地内核负载的方式实现每个 `Node` 节点负载均衡访问所有 `Master` 节点，具体参见下图。

```
  +----------+                       +---------------+  virturl server: 127.0.0.1:6443
  | mater0   |<----------------------| ipvs nodes    |    real servers:
  +----------+                      |+---------------+            10.103.97.200:6443
                                    |                             10.103.97.201:6443
  +----------+                      |                             10.103.97.202:6443
  | mater1   |<---------------------+
  +----------+                      |
                                    |
  +----------+                      |
  | mater2   |<---------------------+
  +----------+复制代码
```

在所有 `Node` 节点上启动一个包含 `lvscare` 进程的 `Static Pod` 对 `IPVS` 进行守护。 如果检测到 `ApiServer` 不可用时，`Sealos` 会自动清理掉所有 `Node` 节点上对应的主节点 `IPVS` 转发规则。直到 `Master` 节点恢复正常时，再自动生成对应规则。为了实现以上功能，我们在 `Node` 节点上增加了下面这些内容。

```
# 增加了一个 lvscare 的 Static Pod
$ cat /etc/kubernetes/manifests

# 自动创建的一些 IPVS 规则
$ ipvsadm -Ln            

# 增加了对虚拟 IP 的地址解析
$ cat /etc/hosts                  复制代码
```

### 4\. 为什么要定制 Kubeadm

-   解决默认证书有效期只有一年的问题。
-   更方便的实现本地负载。
-   核心的功能均集成到 Kubeadm 中了，Sealos 只管分发和执行上层命令，相对就更轻量了。

### 5\. Sealos 执行流程

1.  通过 `SFTP` 或者 `Wget` 命令把离线安装包拷贝到目标机器上，包括所有 `Master` 和 `Node` 节点。
2.  在 `Master 0` 节点上执行 `kubeadm init` 命令。
3.  在其它 `Master` 节点上执行 `kubeadm join` 命令并设置控制面。这个过程中多个 `Master` 节点上的 `Etcd` 会自动组成一个 `Etcd` 集群，并启动相应控制组件。
4.  所有 `Node` 节点都加入到集群中，这个过程中会在 `Node` 节点上进行 `IPVS` 转发规则和 `/etc/hosts` 配置。

> `Node` 节点对 `ApiServer` 的访问均是通过域名进行的。因为 `Node` 节点需要通过 `虚拟 IP` 连接到多个 `Master` 上，但是每个 `Node` 节点的 `Kubelet` 与 `Kube-Proxy` 访问 `ApiServer` 的地址是不同的，所以这里使用域名来解析每个节点上 `ApiServer` 不同的 `IP` 地址。

## 使用 Sealos 部署高可用 Kubernetes 集群

### 1\. 安装相关环境依赖

通过 `Sealos` 进行 `Kubernetes` 集群部署，你需要先准备好以下环境。

1.  在所有要部署的机器上，先完成 `Docker` 的安装和启动。
2.  下载 `Kubernetes` 离线安装包。
3.  下载最新版本 `Sealos`。
4.  对所有服务器进行时间同步。

> Sealos 项目地址：https://github.com/fanux/sealos/releases
> 
> Kubernetes 离线安装包：https://github.com/sealstore/cloud-kernel/releases/

### 2\. 通过 Sealos 部署高可用 Kubernetes 集群

目前 `Sealos` 已经支持最新版本 `Kubernetes 1.16.0` 的高可用集群安装。

#### 2.1 Sealos 常用参数说明

```
--master   Master 节点服务器地址列表
--node     Node 节点服务器地址列表
--user     服务器 SSH 用户名
--passwd   服务器 SSH 用户密码
--pkg-url  离线包所在位置，可以是本地目录，也可以是一个 HTTP 地址
--version  指定需要部署的 Kubernetes 版本
--pk       指定 SSH 私钥所在位置，默认为 /root/.ssh/id_rsa

Other flags:

 --kubeadm-config string   kubeadm-config.yaml 用于指定自定义 kubeadm 配置文件
 --vip string              virtual ip (default "10.103.97.2") 本地负载时虚拟 IP ，不推荐修改，集群外不可访问复制代码
```

#### 2.2 部署一个单主节点的 Kubernetes 集群

通过 `Sealos` 部署 `Kubernetes` 集群是非常简单的 ，通常只需以下两条指令就可以完成安装。

```
$ wget https://github.com/fanux/sealos/releases/download/v2.0.7/sealos && \
    chmod +x sealos && mv sealos /usr/bin 

$ sealos init --passwd YOUR_SERVER_PASSWD \
    --master 192.168.0.2  --master 192.168.0.3  --master 192.168.0.4  \
    --node 192.168.0.5 \
    --pkg-url https://sealyun.oss-cn-beijing.aliyuncs.com/cf6bece970f6dab3d8dc8bc5b588cc18-1.16.0/kube1.16.0.tar.gz \
    --version v1.16.0复制代码
```

如果你的服务器已经配置了 `SSH` 免密登陆，你可以直接使用对应密钥进行部署。

```
$ sealos init --master 192.168.0.2 \
    --node 192.168.0.3 \
    --pkg-url https://YOUR_HTTP_SERVER/kube1.15.0.tar.gz \
    --pk /root/kubernetes.pem \
    --version v1.16.0复制代码
```

如果你需要其它 `Kubernetes` 版本离线包，可到 `Sealos` 官网 http://store.lameleg.com/ 进行下载。

#### 2.3 部署一个多主节点的高可用 Kubernetes 集群

```
$ sealos init --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \
    --node 192.168.0.5 \
    --user root \
    --passwd your-server-password \
    --version v1.16.0 \
    --pkg-url /root/kube1.16.0.tar.gz     复制代码
```

#### 2.4 验证部署是否成功

```
$ kubectl get node
NAME                      STATUS   ROLES    AGE     VERSION
izj6cdqfqw4o4o9tc0q44rz   Ready    master   2m25s   v1.16.0
izj6cdqfqw4o4o9tc0q44sz   Ready    master   119s    v1.16.0
izj6cdqfqw4o4o9tc0q44tz   Ready    master   63s     v1.16.0
izj6cdqfqw4o4o9tc0q44uz   Ready    <none>   38s     v1.16.0

$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                              READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5cbcccc885-9n2p8          1/1     Running   0          3m1s
kube-system   calico-node-656zn                                 1/1     Running   0          93s
kube-system   calico-node-bv5hn                                 1/1     Running   0          2m54s
kube-system   calico-node-f2vmd                                 1/1     Running   0          3m1s
kube-system   calico-node-tbd5l                                 1/1     Running   0          118s
kube-system   coredns-fb8b8dccf-8bnkv                           1/1     Running   0          3m1s
kube-system   coredns-fb8b8dccf-spq7r                           1/1     Running   0          3m1s
kube-system   etcd-izj6cdqfqw4o4o9tc0q44rz                      1/1     Running   0          2m25s
kube-system   etcd-izj6cdqfqw4o4o9tc0q44sz                      1/1     Running   0          2m53s
kube-system   etcd-izj6cdqfqw4o4o9tc0q44tz                      1/1     Running   0          118s
kube-system   kube-apiserver-izj6cdqfqw4o4o9tc0q44rz            1/1     Running   0          2m15s
kube-system   kube-apiserver-izj6cdqfqw4o4o9tc0q44sz            1/1     Running   0          2m54s
kube-system   kube-apiserver-izj6cdqfqw4o4o9tc0q44tz            1/1     Running   1          47s
kube-system   kube-controller-manager-izj6cdqfqw4o4o9tc0q44rz   1/1     Running   1          2m43s
kube-system   kube-controller-manager-izj6cdqfqw4o4o9tc0q44sz   1/1     Running   0          2m54s
kube-system   kube-controller-manager-izj6cdqfqw4o4o9tc0q44tz   1/1     Running   0          63s
kube-system   kube-proxy-b9b9z                                  1/1     Running   0          2m54s
kube-system   kube-proxy-nf66n                                  1/1     Running   0          3m1s
kube-system   kube-proxy-q2bqp                                  1/1     Running   0          118s
kube-system   kube-proxy-s5g2k                                  1/1     Running   0          93s
kube-system   kube-scheduler-izj6cdqfqw4o4o9tc0q44rz            1/1     Running   1          2m43s
kube-system   kube-scheduler-izj6cdqfqw4o4o9tc0q44sz            1/1     Running   0          2m54s
kube-system   kube-scheduler-izj6cdqfqw4o4o9tc0q44tz            1/1     Running   0          61s
kube-system   kube-sealyun-lvscare-izj6cdqfqw4o4o9tc0q44uz      1/1     Running   0          86s复制代码
```

#### 2.5 最简单粗暴的视频教程

如果你觉得上面的教程还是不够直观，现在就给你一个更简单粗暴的学习方式。猛击[这里](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI3MTI2NzkxMA%3D%3D%26mid%3D2247488905%26idx%3D1%26sn%3Dc42f9baeb1a48970ded0f42a520c91a5%26chksm%3Deac534a0ddb2bdb6cb072ec6e2ba60cf53a5447d7f36aa8e8b6a48ca9877ab82837213252133%26token%3D2039868521%26lang%3Dzh_CN%23rd "https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247488905&idx=1&sn=c42f9baeb1a48970ded0f42a520c91a5&chksm=eac534a0ddb2bdb6cb072ec6e2ba60cf53a5447d7f36aa8e8b6a48ca9877ab82837213252133&token=2039868521&lang=zh_CN#rd")的视频，开始吧！

#### 2.6 升级 Kubernetes 集群版本

`Kubernetes` 集群目前处于一个高速迭代期，每个新版本的发布都提供了不少新的特性。升级 `Kubernetes` 集群版本也就成了家常便饭，`Sealos` 也为大家提供非常方便的功能来帮助大家快速完成 `Kubernetes` 集群升级。`Kubernetes` 集群升级大致需要以下几个步骤：

1.  升级所有节点的 `Kubeadm` 并导入新的镜像。
2.  升级 `Master` 节点上的 `Kubelet`。
3.  升级其它 `Master` 节点。
4.  升级 `Node` 节点。
5.  验证集群状态。

**2.6.1 升级 Kubeadm**

这一步主要用于更新 `Kubeadm`、`Kubectl`、`Kubelet` 等二进制文件，并导入新版本的镜像。升级方法很简单，只需复制离线包到所有节点并执行以下命令。

```
$ cd kube/shell && sh init.sh复制代码
```

**2.6.2 升级 Master 节点上的 Kubelet**

升级 `Kubelet` 还是很简单的，只需要把新版本的 `Kubelet` 复制到 `/usr/bin` 目录下替换旧版本，然后重启 `Kubelet` 服务即可。

```
$ kubeadm upgrade plan
$ kubeadm upgrade apply v1.16.0

# 重启 Kubelet
$ systemctl restart kubelet复制代码
```

其中最重要的 `kubeadm upgrade apply` 命令主要完成以下一些操作。

-   验证集群是否可升级并执行版本升级策略。
-   确认离线包中相关镜像是否可用。
-   对控制组件的容器进行升级，失败就回滚。
-   对 `Kube-DNS` 和 `Kube-Proxy` 进行升级。
-   创建新的证书文件并备份旧的证书文件。

**2.6.3 升级其它 Master 节点**

```
$ kubeadm upgrade apply复制代码
```

**2.6.4 升级 Node 节点**

升级 `Node` 节点前，首先要驱逐节点。

```
$ kubectl drain $NODE --ignore-daemonsets复制代码
```

其次，是更新 `Kubelet` 的配置文件和升级 `Node` 节点的 `Kubelet`。

```
$ kubeadm upgrade node config --kubelet-version v1.16.0

# 同样是替换二进制文件并重启 Kubelet
$ systemctl restart kubelet复制代码
```

最后，恢复 `Node` 节点为可调度状态。

```
$ kubectl uncordon $NODE复制代码
```

**2.6.5 验证集群是否升级成功**

```
$ kubectl get nodes复制代码
```

如果输出的节点的版本信息是和升级的版本一致的话，一切就搞定了！

### 3\. 集群清理

如果你需要快速清理已部署的 `Kubernetes` 集群环境，你可以使用下面的命令快速完成。

```
$ sealos clean \
    --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \
    --node 192.168.0.5 \
    --user root \
    --passwd your-server-password复制代码
```

至此，使用 `Sealos` 快速部署一个生产级别的 `Kubernetes` 高可用集群的基本方法就介绍完了。如果你对 `Sealos` 非常的感兴趣，你还可以去官网探索更多高级功能哟！

对于在生产环境中快速部署 `Kubernetes` 高可用集群，你还有哪些更好用高效的方法呢？欢迎大家在留言讨论哟！![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/14/16dc9c2daf0ecbfe~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)