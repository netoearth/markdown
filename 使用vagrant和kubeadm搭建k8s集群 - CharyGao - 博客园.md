## 1\. 准备环境

```
16G 4核CPU 主机一台
```

## 2\. 安装virtual box

## 3\. 安装vagrant

## 4\. Vagrantfile

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "k8s-master",
        :eth1 => "192.168.205.120",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node1",
        :eth1 => "192.168.205.121",
        :mem => "2048",
        :cpu => "1"
    },
    {
        :name => "k8s-node2",
        :eth1 => "192.168.205.122",
        :mem => "2048",
        :cpu => "1"
    }

]

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = opts[:mem]
        v.vmx["numvcpus"] = opts[:cpu]
      end
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end
      config.vm.network :private_network, ip: opts[:eth1]
    end
  end
  config.vm.provision "shell", privileged: true, path: "./setup.sh"
end


```

## 4\. setup.sh

```
#/bin/sh

# install some tools
sudo yum install -y vim telnet bind-utils wget


# install docker
#curl -fsSL get.docker.com -o get-docker.sh
#sh get-docker.sh

## 安装docker


# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum-config-manager --disable docker-ce-edge
yum-config-manager --disable docker-ce-test

# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce

# Step 4: 开启Docker服务
sudo service docker start

# Step 5: 更改cgroup driver
sudo bash -c ' cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF'




if [ ! $(getent group docker) ];
then 
    sudo groupadd docker;
else
    echo "docker user group already exists"
fi

sudo gpasswd -a $USER docker


sudo systemctl  daemon-reload
sudo systemctl restart docker

#rm -rf get-docker.sh

# open password auth for backup if ssh key doesn't work, bydefault, username=vagrant password=vagrant
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sudo systemctl restart sshd

sudo bash -c 'cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF'

# 也可以尝试国内的源 http://ljchen.net/2018/10/23/%E5%9F%BA%E4%BA%8E%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E7%AB%99%E5%AE%89%E8%A3%85kubernetes/

sudo setenforce 0

# install kubeadm, kubectl, and kubelet.
#sudo yum install -y kubelet kubeadm kubectl

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable docker && systemctl start docker
sudo systemctl enable kubelet && systemctl start kubelet


sudo bash -c 'cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
EOF'
sudo sysctl --system

sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo swapoff -a

sudo systemctl enable docker.service
sudo systemctl enable kubelet.service

sudo cat ./pull.sh  
for i in \`kubeadm config images list\`; do   
 imageName=${i#k8s.gcr.io/}  
 sudo docker pull registry.aliyuncs.com/google_containers/$imageName  
 sudo docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName  
 sudo docker rmi registry.aliyuncs.com/google_containers/$imageName  
done;

```

## 6\. 启动vagrant & vagrant命令

```
$ vagrant init      # 初始化
$ vagrant up        # 启动虚拟机  
$ vagrant halt      # 关闭虚拟机  
$ vagrant reload    # 重启虚拟机  
$ vagrant ssh       # SSH 至虚拟机  
$ vagrant suspend   # 挂起虚拟机  
$ vagrant resume    # 唤醒虚拟机  
$ vagrant status    # 查看虚拟机运行状态  
$ vagrant destroy   # 销毁当前虚拟机_

vagrant ssh NAME 登录主机
```

## 8\. 主节点上执行

```
sudo kubeadm init --pod-network-cidr 172.100.0.0/16 --apiserver-advertise-address 192.168.205.120
```

```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

安装网络插件

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

## 9\. node节点

```
sudo kubeadm join 192.168.205.120:6443 --token tte278.145ozal6u6e26ypm --discovery-token-ca-cert-hash sha256:cbb168e0665fe1b14e96a87c2da5dc1eeda04c70932ac1913d989753703277bb

```

## 10.修改节点IP

[https://blog.csdn.net/qianghaohao/article/details/98588427](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.csdn.net%2Fqianghaohao%2Farticle%2Fdetails%2F98588427) 参考这里

vi /etc/sysconfig/kubelet KUBELET\_EXTRA\_ARGS="--node-ip=<eth1 网口 IP>"

"--node-ip=192.168.205.120"

## 10\. 故障排查

查看kubelet服务是否启动成功

```
sudo systemctl status kubelet
```

查看服务日志

```
sudo journalctl -xefu kubelet
```

## kubeadm + vagrant 部署多节点 k8s 的一个坑

## kubeadm + vagrant 部署多节点 k8s 的一个坑

之前写过一篇[「使用 kubeadm 搭建 kubernetes 集群」](https://blog.csdn.net/qianghaohao/article/details/88676241)教程，教程里面使用 Vagrant 启动 3 个节点，1 个 master，2 个 node 节点，后来使用过程中才慢慢发现还是存在问题的。具体问题表现是：

1.  kubectl get node -o wide 查看到节点 IP 都是：10.0.2.15；

```
[root@master ~]# kubectl get node -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION              CONTAINER-RUNTIME
master   Ready    master   5m11s   v1.15.1   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
node1    Ready    <none>   2m31s   v1.15.1   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
```

2.  kubect get pod 可以查看 pod，pod 也运行正常，但是无法查看 pod 日志，也无法 kubectl exec -it 进入 pod。具体报错如下：

```
[root@master ~]# kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5754944d6c-gnqjg   1/1     Running   0          66s
nginx-deployment-5754944d6c-mxgn7   1/1     Running   0          66s

[root@master ~]# kubectl logs nginx-deployment-5754944d6c-gnqjg
Error from server (NotFound): the server could not find the requested resource ( pods/log nginx-deployment-5754944d6c-gnqjg)

[root@master ~]# kubectl exec -it nginx-deployment-5754944d6c-gnqjg sh
error: unable to upgrade connection: pod does not exist
```

带着上面两个问题，于是网上搜索一番，找到了根因并得以解决：  
[https://github.com/kubernetes/kubernetes/issues/60835](https://github.com/kubernetes/kubernetes/issues/60835)

#### 主要原因

Vagrant 在多主机模式时每个主机的 eth0 网口 ip 都是 10.0.2.15，这个网口是所有主机访问公网的出口，用于 nat 转发。而 eth1才是主机真正的 IP。kubelet 在启动时默认读取的是 eth0 网卡的 IP，因此在集群部署完后 kubect get node -o wide 查看到节点的 IP 都是 10.0.2.15。

```
[vagrant@master ~]$ ip addr
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85708sec preferred_lft 85708sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:25:1b:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.26.10/24 brd 192.168.26.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe25:1b45/64 scope link
       valid_lft forever preferred_lft forever
...
```

#### 解决方法

上面知道问题的根本原因是 k8s 节点 IP 获取不对导致访问节点出现问题，那么解决方法就是调整 kubelet 参数设置正确的  
IP 地址：  
编辑 /etc/sysconfig/kubelet 文件，KUBELET\_EXTRA\_ARGS 环境变量添加 `--node-ip` 参数：

```
KUBELET_EXTRA_ARGS="--node-ip=<eth1 网口 IP>"
```

然后重启 kubelet：systemctl restart kubelet  
执行 kubectl get node -o wide 发现节点 IP 已经改变成了KUBELET\_EXTRA\_ARGS 变量指定的 IP。

```
[root@master ~]# kubectl get node -o wide
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION              CONTAINER-RUNTIME
master   Ready    master   24m   v1.15.1   192.168.26.10   <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
node1    Ready    <none>   21m   v1.15.1   10.0.2.15       <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
```

用同样的方法修改其他节点 IP 即可。  
为了方便，这里提供了一个命令，自动化上面步骤：

```
echo KUBELET_EXTRA_ARGS=\"--node-ip=`ip addr show eth1 | grep inet | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}/" | tr -d '/'`\" > /etc/sysconfig/kubelet
systemctl restart kubelet
```

## 使用 kubeadm 搭建 kubernetes 集群

## 使用 kubeadm 搭建 [kubernetes](https://so.csdn.net/so/search?from=pc_blog_highlight&q=kubernetes) 集群

### kubeadm 简介

[kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/) 是 [k8s 官方](https://kubernetes.io/)提供的用于快速部署 k8s 集群的命令行工具，也是官方推荐的最小化部署 k8s 集群的最佳实践，比起直接用二进制部署能省去很多工作，因为该方式部署的集群的各个组件以 docker 容器的方式启动，而各个容器的启动都是通过该工具配自动化启动起来的。

kubeadm 不仅仅能部署 k8s 集群，还能很方便的管理集群，比如集群的升级、降级、集群初始化配置等管理操作。

kubeadm 的设计初衷是为新用户提供一种便捷的方式来首次试用 Kubernetes， 同时也方便老用户搭建集群测试他们的应用。

kubeadm 的使用案例：

-   新用户可以从 kubeadm 开始来试用 Kubernetes。
-   熟悉 Kubernetes 的用户可以使用 kubeadm 快速搭建集群并测试他们的应用。
-   大型的项目可以将 kubeadm 和其他的安装工具一起形成一个比较复杂的系统。

### 安装环境要求

-   一台或多台运行着下列系统的机器:
    -   Ubuntu 16.04+
    -   Debian 9
    -   CentOS 7
    -   RHEL 7
    -   Fedora 25/26
    -   HypriotOS v1.0.1+
    -   Container Linux (针对1800.6.0 版本测试)
-   每台机器 2 GB 或更多的 RAM (如果少于这个数字将会影响您应用的运行内存)
-   2 CPU 核心或更多(节点少于 2 核的话新版本 kubeadm 会报错)
-   集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)
-   禁用 Swap 交换分区。（Swap 分区必须要禁掉，否则安装会报错）

### 准备环境

本文使用 kubeadm 部署一个 3 节点的 k8s 集群：1 个 master 节点，2 个 node 节点。各节点详细信息如下：

| Hostname | IP | OS 发行版 | 内存（GB） | CPU（核） |
| --- | --- | --- | --- | --- |
| k8s-master | 192.168.10.100 | Centos7 | 2 | 2 |
| k8s-node-1 | 192.168.10.101 | Centos7 | 2 | 2 |
| k8s-node-2 | 192.168.10.102 | Centos7 | 2 | 2 |

![在这里插入图片描述](https://img2020.cnblogs.com/blog/572188/202110/572188-20211030003137812-36557207.png)

### kubeadm 安装 k8s 集群完整流程

-   使用 Vagrant 启动 3 台符合上述要求的虚拟机
-   调整每台虚拟机的服务器参数
-   各节点安装 docker、kubeadm、kubelet、kubectl 工具
-   使用 kubeadm 部署 master 节点
-   安装 Pod 网络插件（CNI）
-   使用 kubeadm 部署 node 节点

接下来我们依次介绍每步的具体细节：

#### 使用 Vagrant 启动 3 台符合上述要求的虚拟机

Vagrant 的使用在这里不具体介绍了，如需了解请点击[这里](https://blog.csdn.net/qianghaohao/article/details/80038096)。  
本文用到的 Vagrantfile:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# author: qhh0205

$num_nodes = 2

Vagrant.configure("2") do |config|
  # k8s 主节点定义及初始化配置
  config.vm.define "k8s-master" do | k8s_master |
    k8s_master.vm.box = "Centos7"
    k8s_master.vm.hostname = "k8s-master"
    k8s_master.vm.network "private_network", ip: "192.168.10.100"
    k8s_master.vm.provider "virtualbox" do | v |
      v.name = "k8s-master"
      v.memory = "2048"
      v.cpus = 2
    end
  end

  # k8s node 节点定义及初始化配置
  (1..$num_nodes).each do |i|
      config.vm.define "k8s-node-#{i}" do |node|
        node.vm.box = "Centos7"
        node.vm.hostname = "k8s-node-#{i}"
        node.vm.network "private_network", ip: "192.168.10.#{i+100}"
        node.vm.provider "virtualbox" do |v|
          v.name = "k8s-node-#{i}"
          v.memory = "2048"
          v.cpus = 2
        end
      end
  end
end
```

进入 Vagrantfile 文件所在目录，执行如下命令启动上述定义的 3 台虚拟机：

```
vagrant up
```

#### 调整每台虚拟机的服务器参数

1.  禁用 swap 分区：  
    临时禁用：`swapoff -a`  
    永久禁用：`sed -i '/swap/s/^/#/g' /etc/fstab`  
    swap 分区必须禁止掉，否则 `kubadm init` 自检时会报如下错误：

```
[ERROR Swap]: running with swap on is not supported. Please disable swap
```

2.  将桥接的 IPv4 流量传递到 iptables 的链：

```
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system
```

如果不进行这一步的设置，`kubadm init` 自检时会报如下错误：

```
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
```

3.  关闭网络防火墙：

```
systemctl stop firewalld
systemctl disable firewalld
```

4.  禁用 SELinux：  
    临时关闭 selinux（不需要重启主机）: `setenforce 0`  
    永久关闭 selinux（需要重启主机才能生效）：`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

#### 各节点安装 docker、kubeadm、kubelet、kubectl 工具

##### 安装 Docker

配置 Docker yum 源（阿里 yum 源）：

```
curl -sS -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装 docker：

```
yum install --nogpgcheck -y yum-utils device-mapper-persistent-data lvm2
yum install --nogpgcheck -y docker-ce
systemctl enable docker && systemctl start docker
```

##### 安装 kubeadm、kubelet、kubectl 工具

配置相关工具 yum 源（阿里 yum 源）：

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装 kubeadm、kubelet、kubectl：  
其实 kubeadm、kubelet、kubectl 这三个工具的版本命名是一致的（和 k8s 版本命名一致），我们可以指定安装特定的版本，即安装指定版本的 k8s 集群。

查看哪些版本可以安装：  
`yum --showduplicates list kubeadm|kubelet|kubectl`

在这里我们安装 `1.13.2` 版本：

```
yum install -y kubeadm-1.13.2 kubelet-1.13.2 kubectl-1.13.2
# 设置 kubelet 开机自启动: kubelet 特别重要，如果服务器重启后 kubelet
# 没有启动，那么 k8s 相关组件的容器就无法启动。在这里不需要把 kubelet 启动
# 起来，因为现在还启动不起来，后续执行的 kubeadm 命令会自动把 kubelet 拉起来。
systemctl enable kubelet
```

#### 使用 kubeadm 部署 master 节点

登陆 master 节点执行如下命令：

```
kubeadm init --kubernetes-version v1.13.2 --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.10.100
```

参数说明：  
`--kubernetes-version`: 安装指定版本的 k8s 版本，该参数和 kubeadm 的版本有关，特定版本的 kubeadm 并不能安装所有版本的 k8s，最好还是 kubeadm 的版本和该参数指定的版本一致。

`--image-repository`: 该参数仅在高版本（具体哪个版本没仔细查，反正在 1.13.x 中是支持的）的 kubeadm 中支持，用来设置 kubeadm 拉取 k8s 各组件镜像的地址，默认拉取的地址是：[k8s.gcr.io](https://blog.csdn.net/qianghaohao/article/details/k8s.gcr.io)。众所周知 [k8s.gcr.io](https://blog.csdn.net/qianghaohao/article/details/k8s.gcr.io) 国内是无法访问的，所以在这里改为阿里云镜像仓库。

`--pod-network-cidr`: 设置 pod ip 的网段 ，网段之所以是 `10.244.0.0/16`，是因为后面安装 flannel 网络插件时，yaml 文件里面的 ip 段也是这个，两个保持一致，不然可能会使得 Node 间 Cluster IP 不通。这个参数必须得指定，如果这里不设置的话后面安装 flannel 网络插件时会报如下错误：

```
E0317 17:02:15.077598       1 main.go:289] Error registering network: failed to acquire lease: node "k8s-master" pod cidr not assigned
```

`--apiserver-advertise-address`: API server 用来告知集群中其它成员的地址，这个参数也必须得设置，否则 api-server 容器启动不起来，该参数的值为 master 节点所在的本地 ip 地址。

___

题外话：像之前没有 `--image-repository` 这个参数时，大家为了通过 kubeadm 安装 k8s 都是采用"曲线救国"的方式：先从别的地方把同样的镜像拉到本地（当然镜像的 tag 肯定不是 [k8s.gcr.io/xxxx），然后将拉下来的镜像重新打个](http://k8s.gcr.io/xxxx%EF%BC%89%EF%BC%8C%E7%84%B6%E5%90%8E%E5%B0%86%E6%8B%89%E4%B8%8B%E6%9D%A5%E7%9A%84%E9%95%9C%E5%83%8F%E9%87%8D%E6%96%B0%E6%89%93%E4%B8%AA) tag，tag 命名成和执行 `kubeadm init` 时真正拉取镜像的名称一致（比如：[k8s.gcr.io/kube-controller-manager-amd64:v1.13.2）。这么做显然做了很多不必要的工作，幸好现在有了](http://k8s.gcr.io/kube-controller-manager-amd64:v1.13.2%EF%BC%89%E3%80%82%E8%BF%99%E4%B9%88%E5%81%9A%E6%98%BE%E7%84%B6%E5%81%9A%E4%BA%86%E5%BE%88%E5%A4%9A%E4%B8%8D%E5%BF%85%E8%A6%81%E7%9A%84%E5%B7%A5%E4%BD%9C%EF%BC%8C%E5%B9%B8%E5%A5%BD%E7%8E%B0%E5%9C%A8%E6%9C%89%E4%BA%86) `--image-repository` 这个参数能自定义 kubeadm 拉取 k8s 相关组件的镜像地址了。

___

执行上面命令后，如果出现如下输出（截取了部分），则表示 master 节点安装成功了：

```
...
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.10.100:6443 --token jm6o42.9ystvjarc6u09pjp --discovery-token-ca-cert-hash sha256:64405f3a90597e0ebf1f33134649196047ce74df575cb1a7b38c4ed1e2f94421
```

根据上面输出知道：要开始使用集群普通用户执行下面命令：

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

现在就可以使用 kubectl 访问集群了：

```
[vagrant@k8s-master ~]$ kubectl get node
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   NotReady   master   13m   v1.13.2
```

可以看出现在 master 节点还是 `NotReady` 状态，这是因为默认情况下，为了保证 master 的安全，master 是不会被分配工作负载的。你可以取消这个限制通过输入（不建议这样做，我们后面会向集群中添加两 node 工作节点）：

```
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

#### 安装 Pod 网络插件（CNI）

Pod 网络插件有很多种，具体见这里：[https://kubernetes.io/docs/concepts/cluster-administration/addons/，我们选择部署](https://kubernetes.io/docs/concepts/cluster-administration/addons/%EF%BC%8C%E6%88%91%E4%BB%AC%E9%80%89%E6%8B%A9%E9%83%A8%E7%BD%B2) [Flannel](https://github.com/coreos/flannel) 网络插件：

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### 使用 kubeadm 部署 node 节点

前面已经将 master 节点部署完了，接下来部署 node 节点就很简单了，在 node节点执行如下命令将自己加到 k8s 集群中（复制 master 节点安装完后的输出）：

```
kubeadm join 192.168.10.100:6443 --token jm6o42.9ystvjarc6u09pjp --discovery-token-ca-cert-hash sha256:64405f3a90597e0ebf1f33134649196047ce74df575cb1a7b38c4ed1e2f94421
```

出现如下输出（截取了部分）表示成功将 node 添加到了集群：

```
...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

在 master 节点查看 node 状态，如果都为 Ready，则表示集群搭建完成：

```
[vagrant@k8s-master ~]$ kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   14m   v1.13.2
k8s-node-1   Ready    <none>   3m    v1.13.2
```

用同样的方法把另一个节点也加入到集群中。

### 相关资料

[https://purewhite.io/2017/12/17/use-kubeadm-setup-k8s/](https://purewhite.io/2017/12/17/use-kubeadm-setup-k8s/)  
[https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)  
[https://k8smeetup.github.io/docs/admin/kubeadm/](https://k8smeetup.github.io/docs/admin/kubeadm/)

## Kubeadm 结合 Vagrant 自动化部署最新版 Kubernetes 集群

之前写过一篇搭建 k8s 集群的教程：[「使用 kubeadm 搭建 kubernetes 集群」](https://qhh.me/2019/03/19/%E4%BD%BF%E7%94%A8-kubeadm-%E6%90%AD%E5%BB%BA-kubernetes-%E9%9B%86%E7%BE%A4/)，教程中用到了 kubeadm 和 vagrant，但是整个过程还是手动一步一步完成：`创建节点--> 节点配置、相关软件安装 --> 初始化 master 节点 --> node 节点加入 master 节点`。其实这个过程完全可以通过 Vagrant 的配置器自动化来实现，达到的目的是启动一个 k8s 只需在 Vagrant 工程目录执行：vagrant up 即可一键完成集群的创建。

本文主要介绍如何使用 Kubeadm 结合 Vagrant 自动化 k8s 集群的创建，在了解了 kubeadm 手动搭建 [kubernetes](https://so.csdn.net/so/search?from=pc_blog_highlight&q=kubernetes) 集群的过程后，自动化就简单了，如果不了解请参见之前的文章：[「使用 kubeadm 搭建 kubernetes 集群」](https://qhh.me/2019/03/19/%E4%BD%BF%E7%94%A8-kubeadm-%E6%90%AD%E5%BB%BA-kubernetes-%E9%9B%86%E7%BE%A4/)，梳理下整个过程，在此不做过多介绍。下面大概介绍下自动化流程：

1.  首先抽象出来每个节点需要执行的通用脚本，完成一些常用软件的安装、docker 安装、kubectl 和 kubeadm 安装，还有一些节点的系统级配置：  
    具体实现脚本见：[https://github.com/qhh0205/kubeadm-vagrant/blob/master/install-centos.sh](https://github.com/qhh0205/kubeadm-vagrant/blob/master/install-centos.sh)
    
2.  编写 Vagrantfile，完成主节点的初始化安装和 node 节点加入主节点。但是有个地方和之前手动安装不太一样，为了自动化，我们必须在 kubeadm 初始化 master 节点之前生成 TOKEN（使用其他任意主机的 kubeadm 工具生成 TOKEN 即可），然后自动化脚本统一用这个 TOKEN 初始化主节点和从节点加入。Vagrantfile 具体实现见：[https://github.com/qhh0205/kubeadm-vagrant/blob/master/Vagrantfile](https://github.com/qhh0205/kubeadm-vagrant/blob/master/Vagrantfile)
    

完整的 Vagrant 工程在这里：[https://github.com/qhh0205/kubeadm-vagrant](https://github.com/qhh0205/kubeadm-vagrant)  
使用 kubeadm + vagrant 自动化部署 k8s 集群，基于 Centos7 操作系统。该工程 fork 自 [kubeadm-vagrant](https://github.com/coolsvap/kubeadm-vagrant), 对已知问题进行了修复：节点设置正确的 IP 地址[「set-k8s-node-ip.sh」](https://github.com/qhh0205/kubeadm-vagrant/blob/master/set-k8s-node-ip.sh)。否则使用过程中会出现问题，具体问题见这里：[「kubeadm + vagrant 部署多节点 k8s 的一个坑」](https://qhh.me/2019/08/06/kubeadm-vagrant-%E9%83%A8%E7%BD%B2%E5%A4%9A%E8%8A%82%E7%82%B9-k8s-%E7%9A%84%E4%B8%80%E4%B8%AA%E5%9D%91/)。其他一些调整：节点初始化脚本更改、Vagrantfile 添加 Shell 脚本配置器，运行初始化脚本。

> 默认：1 个 master 节点，1 个 node 节点，可以根据需要修改 Vagrantfile 文件，具体见工程 [README.md](https://github.com/qhh0205/kubeadm-vagrant/blob/master/README.md) 说明。