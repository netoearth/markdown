### 文章目录

-   What is Kubernetes？
-   Kubernetes 架构示意简图
-   Kubernetes 环境搭建方式
-   Kubeadm 部署 Kubernetes
-   -   Kubernetes 部署环境要求
    -   Kubernetes 部署环境准备
    -   Kubernetes 安装具体步骤
    -   部署网络插件

## What is Kubernetes？

Kubernetes 这个单词来自于希腊语，含义是舵手或领航员。Kubernetes，也称为 K8S，其中 8 是代表中间 “ubernete” 的 8 个字符。  
官网描述如下图：生产级别的容器编排系统，是用于自动部署，扩展和管理容器化应用程序的开源系统。 （编排：按照一定的目的依次排列；调配、安排）。  
![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_aTbA.png)

K8S 是 CNCF 毕业的项目，本来 Kubernetes 是 Google 的内部项目，后来开源出来，又后来为了其茁壮成长，捐给了 Cloud Native Computing Foundation（CNCF：云原生计算基金会）  
我们在 github 上可以看到，Kubernetes 是采用 Go 语言开发的。Go（又称 Golang）是 Google 开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言。  
![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_Tvzs.png)

## Kubernetes 架构示意简图

![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_rTqj.png)

**Master（系统由控制面）**  
k8s 集群控制节点，对集群进行调度管理，接受集群外用户去集群操作请求；  
Master Node 由 API Server、Scheduler、ClusterState Store（ETCD 数据库）和 Controller MangerServer 所组成；

**Nodes (数据面)**  
集群工作节点，运行用户业务应用容器；  
Nodes 节点也叫 Worker Node，包含 kubelet、kube proxy 和 Pod（Container Runtime）；

## Kubernetes 环境搭建方式

部署 Kubernetes 环境（集群）主要有多种方式：  
1、kubeadm  
Kubeadm 是一个 K8s 部署工具，提供 kubeadm init 和 kubeadm join 两个操作命令，可以快速部署一个 Kubernetes 集群；  
2、minikube  
`minikube` 可以在本地运行 Kubernetes 的工具，minikube 可以在个人计算机（包括 Windows，macOS 和 Linux PC）上运行一个单节点 Kubernetes 集群，以便您可以试用 Kubernetes 或进行日常开发工作；  
3、二进制包方式  
从 Github 下载发行版的二进制包，手动部署安装每个组件，组成 Kubernetes 集群，步骤比较繁琐，但是能让你对各个组件有更清晰的认识；  
4、yum 安装方式  
通过 yum 安装 Kubernetes 的每个组件，组成 Kubernetes 集群，不过 yum 源里面的 k8s 版本已经比较老了；  
5、第三方工具：利用一些大神封装的工具进行 k8s 环境的安装；  
6、还有一种就是一些云服务公司的公用云平台 k8s。

## Kubeadm 部署 Kubernetes

本文将介绍以上的第一种方式进行示意部署讲解：  
kubeadm 是官方社区推出的一个用于快速部署 kubernetes 集群的工具，这个工具能通过两条指令完成一个 kubernetes 集群的部署；  
1、创建一个 Master 节点：

> kubeadm init

2、将 Node 节点加入到 Master 集群中：

> $ kubeadm join <Master 节点的 IP 和端口>

## Kubernetes 部署环境要求

1、一台或多台机器，操作系统 CentOS 7.x-86\_x64  
2、硬件配置：内存 2GB 以上，CPU 2 核或 CPU 2 核以上；  
3、集群内各个机器之间能相互通信（必须）；  
4、集群内各个机器可以访问外网，需要拉取镜像（非必须，也可手动下载需要的文件包）；  
5、禁止 swap 分区；

## Kubernetes 部署环境准备

**关闭防火墙**

```
systemctl stop firewalld
systemctl disable firewalld
```

**关闭 selinux**

```
sed -i 's/enforcing/disabled/' /etc/selinux/config  #永久
setenforce 0  #临时
```

**关闭 swap（k8s 禁止虚拟内存以提高性能）**

```
sed -ri 's/.*swap.*/#&/' /etc/fstab #永久
swapoff -a #临时
```

**在 master 添加 hosts （ip 地址根据自己预先设置的为准）**

```
cat >> /etc/hosts << EOF 192.168.52.100 k8smaster 192.168.52.101 k8snode EOF
```

**设置网桥参数**

```
cat > /etc/sysctl.d/k8s.conf << EOF net.bridge.bridge-nf-call-ip6tables = 1 net.bridge.bridge-nf-call-iptables = 1 EOF
sysctl --system  #生效
```

**时间同步**

```
yum install ntpdate -y
ntpdate time.windows.com
```

## Kubernetes 安装具体步骤

所有服务器节点安装 `Docker、kubeadm、kubelet、kubectl，Kubernetes` 默认容器运行环境是 Docker，因此首先需要安装 Docker；  
1、安装 `Docker`  
更新 docker 的 yum 源

```
yum install wget -y
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```

安装指定版本的 docker（自行替换最新版本号 x.x.x.x）：

```
yum install docker-ce-x.x.x.x -y
```

配置加速器加速下载

```
/etc/docker/daemon.json
{
   
"registry-mirrors": ["https://cr.console.aliyun.com/"]
}
```

然后执行，不然会提示警告：

> systemctl enable docker.service

2、接下搭建：kubeadm、kubelet、kubectl , 添加 k8s 的阿里云 YUM 源

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF [kubernetes] name=Kubernetes baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64 enabled=1 gpgcheck=0 repo_gpgcheck=0 gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg EOF
```

3、安装 `kubeadm，kubelet 和 kubectl`

> yum install kubelet-1.24.1 kubeadm-1.24.1 kubectl-1.24.1 -y

然后执行，不然会提示警告：

> systemctl enable kubelet.service

查看有没有安装：

```
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl
```

查看安装的版本：

> kubelet –version

`Kubelet`：运行在 cluster 所有节点上，负责启动 POD 和容器；  
`Kubeadm`：用于初始化 cluster 的一个工具；  
`Kubectl`：kubectl 是 kubenetes 命令行工具，通过 kubectl 可以部署和管理应用，查看各种资源，创建，删除和更新组件；

此时应重启一下系统 reboot（centos）；

4、部署 Kubernetes Master 主节点（此命令在 master 机器上执行）

```
kubeadm init --apiserver-advertise-address=192.168.52.100 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.24.1 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
```

补充：service-cidr 的选取不能和 PodCIDR 及本机网络有重叠或者冲突，一般可以选择一个本机网络和 PodCIDR 都没有用到的私网地址段，比如 PODCIDR 使用 10.244.0.0/16, 那么 service cidr 可以选择 10.96.0.0/12，网络无重叠冲突即可；

接下来在 master 机器上执行：

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
```

接下来把 `node` 节点加入 Kubernetes master 中，在 Node 机器上执行；  
向集群添加新节点，执行的命令是 kubeadm init 最后输出的 `kubeadm join` 命令（如下图）：  
![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_L8oE.png)

```
kubeadm join 192.168.52.101：端口号 --token wa5bif.zfuvbesevdfvf4of \
    --discovery-token-ca-cert-hash sha256:87cf5828d54dd80da13c4b57c57360370ea0267a7cc3991989ca3006cf3e44d8
```

![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_qcBF.png)

## 部署网络插件

下载 `kube-flannel.yml` 文件 （flannel 作为 k8s 的集群中常用的网络组件，其 yml 文件的获取，建议去 github 中获取）

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

应用 `kube-flannel.yml` 文件得到运行时容器

> kubectl apply -f kube-flannel.yml （在 master 机器上执行）  
> ![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_HQ55.png)

然后查看节点状态： `kubectl get nodes` （在 master 机器上执行），STATUS:Ready 时，说明我们的 k8s 环境至此就搭建好了。

查看运行时容器 `pod` （一个 pod 里面运行了多个 docker 容器）

> kubectl get pods -n kube-system  
> ![在这里插入图片描述](https://static.oschina.net/uploads/img/202208/19170512_F7Fb.png)

以上就是用 `Kubeadm` 简单部署 `Kubernetes` 的过程，其中涉及到提前准备机器、环境、安装包等动作，过程中需要大家细心操作，部分组件、包等不能通过 wget 顺利下载时，建议手工下载。 最后，迎大家参考、亲测～

🍒如果您觉得博主的文章还不错或者有帮助的话，请关注一下博主，如果三连收藏支持就更好啦！谢谢各位大佬给予的鼓励！