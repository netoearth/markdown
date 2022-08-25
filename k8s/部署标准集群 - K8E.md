### On this page

-   -   [初始化网络 内置Cilium 安装工具](https://getk8e.com/docs/install/200-quick-start/#%E5%88%9D%E5%A7%8B%E5%8C%96%E7%BD%91%E7%BB%9C-%E5%86%85%E7%BD%AEcilium-%E5%AE%89%E8%A3%85%E5%B7%A5%E5%85%B7)

## 部署标准集群

使用k8e快速部署[[Kubernetes]]集群服务

作为YAML工程师，经常需要使用[[Kubernetes]]集群来验证很多技术化场景，如何快速搭建一套完整标准化的集群至关重要。罗列当前能快速部署[[Kubernetes]] 集群的工具有很多种，例如官方首当其冲有**kubeadm**工具，[[云原生]]社区有**sealos**作为一键部署的最佳方案，熟悉起来后部署都非常快。但是你是否考虑过并不是每一个YAML工程师都需要非常了解集群组件的搭配。这里，我给大家推荐的工具是基于单个文件的免配置的部署方式，对比kubeadm和sealos方案，去掉了对 [[Kubernetes]] 官方组件镜像的依赖，并且把[[Kubernetes]]相关的核心扩展推荐组件也都集成到这个二进制包中，通过软链接暴露，让环境依赖更少，这个安装工具就是**k8e**(可以叫 ‘kuber easy’ 或 K8易) 。k8e是基于当前主流上游[[Kubernetes]]发行版 k3s做的优化封装和裁剪。去掉对IoT的依赖，目标就是做最好的服务器版本的发行版本。并且和上游保持一致，可以自由扩展。

**K8e v1** 组件架构图：

![k8e-arch](https://getk8e.com/k8e-arch.png)

-   准备主机（**2Core/4G RAM**是最小标配）清单如下：

| 主机名 | 配置 | 数量 | 网络 | 类型 |
| --- | --- | --- | --- | --- |
| k8e-test1 | 2Core vCPU/8G RAM | 1 | 172.25.1.56 | master/etcd |
| k8e-test2 | 2Core vCPU/8G RAM | 1 | 172.25.1.57 | master/etcd |
| k8e-test3 | 2Core vCPU/8G RAM | 1 | 172.25.1.58 | master/etcd |
| k8e-test4 | 2Core vCPU/8G RAM | 1 | 172.25.1.59 | agent |

以下端口需要在主机上开放: (https://[[kubernetes]].io/docs/reference/ports-and-protocols/)

_**Control plane**_

| Protocol | Direction | Port Range | Purpose | Used By |
| --- | --- | --- | --- | --- |
| TCP | Inbound | 6443 | Kubernetes API server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| TCP | Inbound | 10252 | kube-controller-manager | Self |

_**Worker node(s)**_

| Protocol | Direction | Port Range | Purpose | Used By |
| --- | --- | --- | --- | --- |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 30000-32767 | NodePort Services | All |

k8e 默认支持 [[kubernetes]] 1.21 主线版本, 以下操作默认在root帐号下操作：

部署步骤如下：

1.  启动 k8s 集群过程：

-   注意主机系统必须满足：**Linux kernel >= 4.9.17**，方便支持eBPF网络
-   第一台，是引导服务(注意：第一台主机IP就是**api-server**的IP)：

```
# 定义环境变量
# K8E_TOKEN   集群token，用来传输tls证书使用
# K8E_NODE_NAME  主机hostname，用来表示etcd中的唯一别名
# K8E_CLUSTER_INIT  启用内置etcd做为存储

curl -sfL https://getk8e.com/install.sh | K8E_TOKEN=ilovek8e INSTALL_K8E_EXEC="server --cluster-init --write-kubeconfig-mode=666" sh -
```

此时，单机版的k8s集群就启动完毕，做技术验证环境足够。但是如果你需要搭建高可用版本的k8s集群，就需要启用3台master节点，这是可根据一下步骤增加master节点。遵循**CAP原理** 高可用集群，需要配置3台以上的etcd实例才能保证集群状态的一致性。

第二台master节点配置如下：

```
curl -sfL https://getk8e.com/install.sh | K8E_TOKEN=ilovek8e K8E_URL=https://172.25.1.56:6443 INSTALL_K8E_EXEC="server --write-kubeconfig-mode=666" sh -s -
```

第三台master节点配置如下

```
curl -sfL https://getk8e.com/install.sh | K8E_TOKEN=ilovek8e K8E_URL=https://172.25.1.56:6443 INSTALL_K8E_EXEC="server --write-kubeconfig-mode=666" sh -s -
```

此时，3台master节点配置完毕，通过命令`kubelet get node` 获得节点情况。

2.  添加工作负载节点agent

```
curl -sfL https://getk8e.com/install.sh | K8E_TOKEN=ilovek8e K8E_URL=https://172.25.1.56:6443 sh -
```

### 初始化网络 内置Cilium 安装工具

查看cilium网络状态

```
[root@ip-172-31-24-246 ~]# export KUBECONFIG=/etc/k8e/k8e.yaml
[root@ip-172-31-24-246 ~]# cilium status
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         disabled
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

DaemonSet         cilium             Desired: 2, Ready: 2/2, Available: 2/2
Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium             Running: 2
                  cilium-operator    Running: 1
Cluster Pods:     3/3 managed by Cilium
Image versions    cilium             quay.io/cilium/cilium:v1.10.5: 2
                  cilium-operator    quay.io/cilium/operator-generic:v1.10.5: 1
```

_**Note**_ : 主控节点三台内置了LB，启动集群后，把第一台关闭，计算节点agent也可以自动切换到第二台。不用担心集群受到影响。对于管理控制台想访问api-server地址，可以放一个nginx汇聚三台master节点的ip就可以。如果想做VIP，可以考虑采用kube-vip来实现。

默认kubeconfig放在 /etc/k8e/k8e.yaml中。

_**Note**_ : 对于集群高可用场景下控制面kube-api-server，需要一个VIP入口，这里需要注意。因为k8e是默认加入了TLS证书在6443端口上，所以需要在启动k8e的时候给VIP也签名一个证书，这样管理端才能有效的访问这个K8S集群进行管理。配置参数如下：

```
# --tls-san value   (listener) Add additional hostname or IP as a Subject Alternative Name in the TLS cert
# bootstrap server和其它server都需要配置如下参数：
curl -sfL https://getk8e.com/install.sh | INSTALL_K8E_EXEC="--tls-san 192.168.1.1" sh -
```

_**Note**_ : 支持启用Dockerd[[容器]]引擎，需要修改/etc/systemd/systemd/k8e.service 文件加入参数`--docker`就可以切换成功，后面Pod内的[[容器]]就是由docker来执行。如下：

```
# --docker这个参数就是切换[[[[[[[[[[[[[[[[[[[[[[[[容器]]]]]]]]]]]]]]]]]]]]]]]]引擎到docker
curl -sfL https://getk8e.com/install.sh | INSTALL_K8E_EXEC="--docker" sh -
```

_**Note**_ : 清理k8e，请使用k8e内置提供的脚本来清理。默认以上安装在root下安装才可以顺利清理k8e文件 [清理k8e点这里 →](https://getk8e.com/docs/install/230-clean-k8e/)