sealos 项目地址：[https://github.com/fanux/sealos](https://github.com/fanux/sealos)

本文教你如何用一条命令构建 k8s 高可用集群且不依赖 haproxy 和 keepalived，也无需 ansible。通过内核 `ipvs` 对 apiserver 进行负载均衡，并且带 `apiserver` 健康检测。架构如下图所示：

![](https://hugo-picture.oss-cn-beijing.aliyuncs.com/2020-04-20-sealos.webp)

本项目名叫 sealos，旨在做一个简单干净轻量级稳定的 kubernetes 安装工具，能很好的支持高可用安装。 其实把一个东西做的功能强大并不难，但是做到极简且灵活可扩展就比较难。 所以在实现时就必须要遵循这些原则。下面介绍一下 sealos 的设计原则：

## 1. 设计原则

___

sealos 特性与优势：

-   支持离线安装，工具与资源包（二进制程序 配置文件 镜像 yaml文件等）分离,这样不同版本替换不同离线包即可
-   证书延期
-   使用简单
-   支持自定义配置
-   内核负载，极其稳定，因为简单所以排查问题也极其简单

### 为什么不用 ansible？

1.0 版本确实是用 `ansible` 实现，但是用户还是需要先装 ansile，装 ansible 又需要装 `python` 和一些依赖等，为了不让用户那么麻烦把 ansible 放到了容器里供用户使用。如果不想配置免密钥使用用户名密码时又需要 `ssh-pass` 等，总之不能让我满意，不是我想的极简。

所以我想就来一个二进制文件工具，没有任何依赖，文件分发与远程命令都通过调用 sdk 实现所以不依赖其它任何东西，总算让我这个有洁癖的人满意了。

### 为什么不用 keepalived haproxy？

`haproxy` 用 static pod 跑没有太大问题，还算好管理，`keepalived` 现在大部分开源 ansible 脚本都用 yum 或者 apt 等装，这样非常的不可控，有如下劣势：

-   源不一致可能导致版本不一致，版本不一直连配置文件都不一样，我曾经检测脚本不生效一直找不到原因，后来才知道是版本原因。
-   系统原因安装不上，依赖库问题某些环境就直接装不上了。
-   看了网上很多安装脚本，很多检测脚本与权重调节方式都不对，直接去检测 haproxy 进程在不在，其实是应该去检测 apiserver 是不是 healthz 的，如果 apiserver 挂了，即使 haproxy 进程存在，集群也会不正常了，就是伪高可用了。
-   管理不方便，通过 prometheus 对集群进行监控，是能直接监控到 static pod 的但是用 systemd 跑又需要单独设置监控，且重启啥的还需要单独拉起。不如 kubelet 统一管理来的干净简洁。
-   我们还出现过 keepalived 把 CPU 占满的情况。

所以为了解决这个问题，我把 keepalived 跑在了容器中(社区提供的镜像基本是不可用的) 改造中间也是发生过很多问题，最终好在解决了。

总而言之，累觉不爱，所以在想能不能甩开 haproxy 和 keepalived 做出更简单更可靠的方案出来，还真找到了。。。

### 本地负载为什么不使用 envoy 或者 nginx？

我们通过本地负载解决高可用问题。

**本地负载**：在每个 node 节点上都启动一个负载均衡，上游就是三个 master。

如果使用 `envoy` 之类的负载均衡器，则需要在每个节点上都跑一个进程，消耗的资源更多，这是我不希望的。ipvs 实际也多跑了一个进程 `lvscare`，但是 `lvscare` 只是负责管理 ipvs 规则，和 `kube-proxy` 类似，真正的流量还是从很稳定的内核走的，不需要再把包丢到用户态中去处理。

在架构实现上有个问题会让使用 `envoy` 等变得非常尴尬，就是 `join` 时如果负载均衡没有建立那是会卡住的，`kubelet` 就不会起来，所以为此你需要先启动 envoy，意味着你又不能用 static pod 去管理它，同上面 keepalived 宿主机部署一样的问题，用 static pod 就会相互依赖，逻辑死锁，鸡说要先有蛋，蛋说要先有鸡，最后谁都没有。

使用 `ipvs` 就不一样，我可以在 join 之前先把 ipvs 规则建立好，再去 join 就可以了，然后对规则进行守护即可。一旦 apiserver 不可访问了，会自动清理掉所有 node 上对应的 ipvs 规则， 等到 master 恢复正常时添加回来。

### 为什么要定制 kubeadm?

首先是由于 `kubeadm` 把证书过期时间写死了，所以需要定制把它改成 `99` 年，虽然大部分人可以自己去签个新证书，但是我们还是不想再依赖个别的工具，就直接改源码了。

其次就是做本地负载时修改 `kubeadm` 代码是最方便的，因为在 join 时我们需要做两个事，第一是 join 之前先创建好 ipvs 规则，第二是创建 static pod。如果这块不去定制 kubeadm 就把报静态 pod 目录已存在的错误，忽略这个错误很不优雅。 而且 kubeadm 中已经提供了一些很好用的 `sdk` 供我们去实现这个功能。

且这样做之后最核心的功能都集成到 kubeadm 中了，`sealos` 就单单变成分发和执行上层命令的轻量级工具了，增加节点时我们也就可以直接用 kubeadm 了。

## 2. 使用教程

___

### 安装依赖

-   安装并启动 docker
-   下载 [kubernetes 离线安装包](https://github.com/sealstore/cloud-kernel/releases/)
-   下载 [最新版本 sealos](https://github.com/fanux/sealos/releases)
-   支持 kubernetes 1.14.0+

### 安装

多 master HA 只需执行以下命令：

```
$ sealos init --master 192.168.0.2 \
  --master 192.168.0.3 \
  --master 192.168.0.4 \
  --node 192.168.0.5 \
  --user root \
  --passwd your-server-password \
  --version v1.14.1 \
  --pkg-url /root/kube1.14.1.tar.gz 
```

然后，就没有然后了。。。没错，你的高可用集群已经装好了，是不是觉得一脸懵逼？就是这么简单快捷！

单 master 多 node：

```
$ sealos init --master 192.168.0.2 \
  --node 192.168.0.5 \ 
  --user root \
  --passwd your-server-password \
  --version v1.14.1 \
  --pkg-url /root/kube1.14.1.tar.gz 
```

使用免密钥或者密钥对：

```
$ sealos init --master 172.16.198.83 \
  --node 172.16.198.84 \
  --pkg-url https://sealyun.oss-cn-beijing.aliyuncs.com/free/kube1.15.0.tar.gz \
  --pk /root/kubernetes.pem # this is your ssh private key file \
  --version v1.15.0
```

参数解释：

```
--master   master服务器地址列表
--node     node服务器地址列表
--user     服务器ssh用户名
--passwd   服务器ssh用户密码
--pkg-url  离线包位置，可以放在本地目录，也可以放在一个 http 服务器上，sealos 会 wget 到安装目标机
--version  kubernetes 版本
--pk       ssh 私钥地址，配置免密钥默认就是 /root/.ssh/id_rsa
```

其他参数：

```
--kubeadm-config string   kubeadm-config.yaml kubeadm 配置文件，可自定义 kubeadm 配置文件
--vip string              virtual ip (default "10.103.97.2") 本地负载时虚拟 ip，不推荐修改，集群外不可访问
```

检查安装是否正常：

```
$ kubectl get node
NAME                      STATUS   ROLES    AGE     VERSION
izj6cdqfqw4o4o9tc0q44rz   Ready    master   2m25s   v1.14.1
izj6cdqfqw4o4o9tc0q44sz   Ready    master   119s    v1.14.1
izj6cdqfqw4o4o9tc0q44tz   Ready    master   63s     v1.14.1
izj6cdqfqw4o4o9tc0q44uz   Ready    <none>   38s     v1.14.1

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
kube-system   kube-sealyun-lvscare-izj6cdqfqw4o4o9tc0q44uz      1/1     Running   0          86s
```

### 增加节点

先获取 join command，在 master 上执行：

```
$ kubeadm token create --print-join-command
```

可以使用超级 kubeadm，但是 join 时需要增加一个 `--master` 参数：

```
$ cd kube/shell && init.sh
$ echo "10.103.97.2 apiserver.cluster.local" >> /etc/hosts   # using vip
$ kubeadm join 10.103.97.2:6443 --token 9vr73a.a8uxyaju799qwdjv \
  --master 10.103.97.100:6443 \
  --master 10.103.97.101:6443 \
  --master 10.103.97.102:6443 \
  --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

也可以用 sealos join 命令：

```
$ sealos join --master 192.168.0.2 \
  --master 192.168.0.3 \
  --master 192.168.0.4 \
  --vip 10.103.97.2 \
  --node 192.168.0.5 \
  --user root \
  --passwd your-server-password \
  --pkg-url /root/kube1.15.0.tar.gz
```

### 使用自定义 kubeadm 配置文件

有时你可能需要自定义 kubeadm 的配置文件，比如要在证书里加入域名 `sealyun.com`。

首先需要获取配置文件模板：

```
$ sealos config -t kubeadm >>  kubeadm-config.yaml.tmpl
```

然后修改 `kubeadm-config.yaml.tmpl` 即可，将 `sealyun.com` 添加到配置中：

```
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: {{.Version}}
controlPlaneEndpoint: "apiserver.cluster.local:6443"
networking:
  podSubnet: 100.64.0.0/10
apiServer:
        certSANs:
        - sealyun.com # 这是新增的域名
        - 127.0.0.1
        - apiserver.cluster.local
        {{range .Masters -}}
        - {{.}}
        {{end -}}
        - {{.VIP}}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
        excludeCIDRs: 
        - "{{.VIP}}/32"
```

**注意：其它部分不用修改，sealos 会自动填充模板里面的内容。**

最后在部署时使用 `--kubeadm-config` 指定配置文件模板即可：

```
$ sealos init --kubeadm-config kubeadm-config.yaml.tmpl \
  --master 192.168.0.2 \
  --master 192.168.0.3 \
  --master 192.168.0.4 \
  --node 192.168.0.5 \
  --user root \
  --passwd your-server-password \
  --version v1.14.1 \
  --pkg-url /root/kube1.14.1.tar.gz 
```

### 版本升级

本教程以 `1.14` 版本升级到 `1.15` 为例，其它版本原理类似，懂了这个其它的参考 [官方教程](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-14/)即可。

#### 升级过程

1.  升级 kubeadm，所有节点导入镜像
2.  升级控制节点
3.  升级 master(控制节点)上的 kubelet
4.  升级其它 master(控制节点)
5.  升级 node
6.  验证集群状态

#### 升级 kubeadm

把离线包拷贝到所有节点执行 `cd kube/shell && sh init.sh`。这里会把 kubeadm、kubectl、kubelet 的二进制文件都更新掉，而且会导入高版本镜像。

#### 升级控制节点

```
$ kubeadm upgrade plan
$ kubeadm upgrade apply v1.15.0
```

重启 kubelet：

```
$ systemctl restart kubelet
```

其实 kubelet 升级很简单粗暴，我们只需要把新版本的 kubelet 拷贝到 /usr/bin 下面，重启 kubelet service 即可，如果程序正在使用不让覆盖那么就停一下 kubelet 再进行拷贝，kubelet bin 文件在 `conf/bin` 目录下。

#### 升级其它控制节点

#### 升级 node

驱逐节点（要不要驱逐看情况, 喜欢粗暴的直接来也没啥）：

```
$ kubectl drain $NODE --ignore-daemonsets
```

更新 kubelet 配置：

```
$ kubeadm upgrade node config --kubelet-version v1.15.0
```

然后升级 kubelet。同样是替换二进制再重启 kubelet service。

```
$ systemctl restart kubelet
```

召回失去的爱情：

#### 验证

如果版本信息都对的话基本就升级成功了。

#### kubeadm upgrade apply 干了啥？

1.  检查集群是否可升级
2.  执行版本升级策略 哪些版本之间可以升级
3.  确认镜像是否存在
4.  执行控制组件升级，如果失败就回滚，其实就是 apiserver、controller manager、scheduler 等这些容器
5.  升级 kube-dns 和 kube-proxy
6.  创建新的证书文件，备份老的如果其超过 180 天

### 源码编译

因为使用了 `netlink` 库，所以推荐在容器内进行编译，只需一条命令：

```
$ docker run --rm -v $GOPATH/src/github.com/fanux/sealos:/go/src/github.com/fanux/sealos -w /go/src/github.com/fanux/sealos -it golang:1.12.7  go build
```

如果你使用的是 go mod，则需要指定通过 `vendor` 编译：

### 卸载

```
$ sealos clean \
  --master 192.168.0.2 \
  --master 192.168.0.3 \
  --master 192.168.0.4 \
  --node 192.168.0.5 \
  --user root \
  --passwd your-server-password
```

## 3. sealos 实现原理

___

### 执行流程

-   通过 `sftp`或者 `wget` 把离线安装包拷贝到目标机器上（masters 和 nodes）。
-   在 master0 上执行 `kubeadm init`。
-   在其它 master 上执行 kubeadm join 并设置控制面，这个过程会在其它 master 上起动 `etcd` 并与 `master0` 的 etcd 组成集群，并启动控制平面的组件（apiserver、controller 等）。
-   join node 节点，会在 node 上配置 ipvs 规则，配置 /etc/hosts 等。

所有对 apiserver 的请求都是通过域名进行访问，因为 node 需要通过虚拟 ip 连接多个 master，每个节点的 kubelet 与 kube-proxy 访问 apiserver 的虚拟地址是不一样的，而 kubeadm 又只能在配置文件中指定一个地址，所以使用一个域名但是每个节点解析的 IP 不同。当 IP 地址发生变化时仅需要修改解析地址即可。

### 本地内核负载

通过这样的方式实现每个 node 上通过本地内核负载均衡访问 masters：

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
  +----------+
```

在 node 上起了一个 `lvscare` 的 static pod 去守护这个 ipvs，一旦 apiserver 不可访问了，会自动清理掉所有 node 上对应的 ipvs 规则， master 恢复正常时添加回来。

所以在你的 node 上加了三个东西，可以直观的看到：

```
$ cat /etc/kubernetes/manifests   # 这下面增加了 lvscare 的 static pod
$ ipvsadm -Ln                     # 可以看到创建的ipvs规则
$ cat /etc/hosts                  # 增加了虚拟IP的地址解析
```

### 定制 kubeadm

sealos 对 kubeadm 改动非常少，主要是延长了证书过期时间和扩展了 join 命令。下面主要讲讲对 join 命令的改造。

首先 join 命令增加 `--master` 参数用于指定 master 地址列表：

```
lagSet.StringSliceVar(
&locallb.LVScare.Masters, "master", []string{},
"A list of ha masters, --master 192.168.0.2:6443  --master 192.168.0.2:6443  --master 192.168.0.2:6443",
)
```

这样就可以拿到 master 地址列表去做 ipvs 负载均衡了。

如果不是控制节点且不是单 master，那么就只创建一条 ipvs 规则，控制节点上不需要创建，连自己的 apiserver 即可：

```
if data.cfg.ControlPlane == nil {
fmt.Println("This is not a control plan")
if len(locallb.LVScare.Masters) != 0 {
locallb.CreateLocalLB(args[0])
}
} 
```

然后再去创建 lvscare static pod 来守护 ipvs：

```
if len(locallb.LVScare.Masters) != 0 {
locallb.LVScareStaticPodToDisk("/etc/kubernetes/manifests")
}
```

\*\*所以哪怕你不使用 sealos，也可以直接用定制过的 kubeadm 去部署集群，只是麻烦一些。\*\*下面给出安装步骤。

kubeadm 配置文件：

```
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: v1.14.0
controlPlaneEndpoint: "apiserver.cluster.local:6443" # apiserver DNS name
apiServer:
        certSANs:
        - 127.0.0.1
        - apiserver.cluster.local
        - 172.20.241.205
        - 172.20.241.206
        - 172.20.241.207
        - 172.20.241.208
        - 10.103.97.1          # virturl ip
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
        excludeCIDRs: 
        - "10.103.97.1/32" # 注意：如果不加这个，kube-proxy 就会清理你的规则
```

在 `master0`（假设 vip 地址为 10.103.97.100）上执行以下命令：

```
$ echo "10.103.97.100 apiserver.cluster.local" >> /etc/hosts # 解析的是 master0 的地址
$ kubeadm init --config=kubeadm-config.yaml --experimental-upload-certs  
$ mkdir ~/.kube && cp /etc/kubernetes/admin.conf ~/.kube/config
$ kubectl apply -f https://docs.projectcalico.org/v3.6/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

在 `master1`（假设 vip 地址为 10.103.97.101）上执行以下命令：

```
$ echo "10.103.97.100 apiserver.cluster.local" >> /etc/hosts #解析的是 master0 的地址,为了能正常 join 进去
$ kubeadm join 10.103.97.100:6443 --token 9vr73a.a8uxyaju799qwdjv \
    --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 \
    --experimental-control-plane \
    --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07 

$ sed "s/10.103.97.100/10.103.97.101/g" -i /etc/hosts  # 解析再换成自己的地址，否则就都依赖 master0 的伪高可用了
```

在 `master2`（假设 vip 地址为 10.103.97.102）上执行以下命令：

```
$ echo "10.103.97.100 apiserver.cluster.local" >> /etc/hosts
$ kubeadm join 10.103.97.100:6443 --token 9vr73a.a8uxyaju799qwdjv \
    --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 \
    --experimental-control-plane \
    --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07  

$ sed "s/10.103.97.100/10.103.97.102/g" -i /etc/hosts
```

在 node 上 join 时加上 `--master` 参数指定 master 地址列表：

```
$ echo "10.103.97.1 apiserver.cluster.local" >> /etc/hosts   # 需要解析成虚拟 ip
$ kubeadm join 10.103.97.1:6443 --token 9vr73a.a8uxyaju799qwdjv \
    --master 10.103.97.100:6443 \
    --master 10.103.97.101:6443 \
    --master 10.103.97.102:6443 \
    --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

### 离线包结构分析

```
.
├── bin  # 指定版本的二进制文件，只需要这三个，其它组件跑在容器里
│   ├── kubeadm
│   ├── kubectl
│   └── kubelet
├── conf
│   ├── 10-kubeadm.conf  # 这个文件新版本没用到，我在 shell 里直接生成，这样可以检测 cgroup driver
│   ├── dashboard
│   │   ├── dashboard-admin.yaml
│   │   └── kubernetes-dashboard.yaml
│   ├── heapster
│   │   ├── grafana.yaml
│   │   ├── heapster.yaml
│   │   ├── influxdb.yaml
│   │   └── rbac
│   │       └── heapster-rbac.yaml
│   ├── kubeadm.yaml # kubeadm 的配置文件
│   ├── kubelet.service  # kubelet systemd 配置文件
│   ├── net
│   │   └── calico.yaml
│   └── promethus
├── images  # 所有镜像包
│   └── images.tar
└── shell
    ├── init.sh  # 初始化脚本
    └── master.sh # 运行master脚本
```

-   `init.sh` 脚本会将 bin 目录下的二进制文件拷贝到 `$PATH` 下面，并配置好 systemd，关闭 swap 和防火墙等等，然后导入集群所需要的镜像。
-   `master.sh` 主要执行了 kubeadm init。
-   conf 目录下面包含了 kubeadm 的配置文件，calico yaml 文件等等。
-   sealos 会调用上面的两个脚本，所以大部分兼容。不同版本都可以通过微调脚本来保持兼容。

\-------他日江湖相逢 再当杯酒言欢-------