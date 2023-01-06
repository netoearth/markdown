## kubekey简介

kubeykey是KubeSphere基于Go 语言开发的[kubernetes](https://so.csdn.net/so/search?q=kubernetes&spm=1001.2101.3001.7020)集群安装工具，可以轻松、高效、灵活地单独或整体安装 Kubernetes 和 KubeSphere，底层使用 Kubeadm 在多个节点上并行安装 Kubernetes 集群，支持创建、扩缩和升级 Kubernetes 集群。  
KubeKey 提供内置高可用模式，支持一键安装高可用 Kubernetes 集群。KubeKey 不仅能帮助用户在线创建集群，还能作为离线安装解决方案。

KubeKey可以用于以下三种安装场景：

-   仅安装 Kubernetes集群
-   一键安装 Kubernetes 和 KubeSphere
-   已有Kubernetes集群，使用ks-installer 在其上部署 KubeSphere

![在这里插入图片描述](https://img-blog.csdnimg.cn/474ce94e77344d03bb14c96bedee5f09.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70)  
项目地址：[https://github.com/kubesphere/kubekey](https://github.com/kubesphere/kubekey)  
官方文档：[https://kubesphere.io/zh/docs/installing-on-linux/introduction/kubekey/](https://kubesphere.io/zh/docs/installing-on-linux/introduction/kubekey/)

## kubekey安装

下载kk二进制部署命令，您可以在任意节点下载该工具，比如准备一个部署节点，或者复用集群中已有节点：

```
wget https://github.com/kubesphere/kubekey/releases/download/v1.2.0-alpha.2/kubekey-v1.2.0-alpha.2-linux-amd64.tar.gz
tar -zxvf kubekey-v1.2.0-alpha.2-linux-amd64.tar.gz
mv kk /usr/local/bin/
```

或脚本直接从国内地址下载安装

```
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.0 sh -
mv kk /usr/local/bin/
```

查看kubekey版本

```
kk version
```

查看kubekey支持的kubernertes版本

```
kk version --show-supported-k8s
```

## 部署单节点集群

1、环境初始化，在所有节点上安装相关依赖

```
yum install -y socat conntrack ebtables ipset
```

2、所有节点关闭selinux和firewalld

```
setenforce 0 && sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
systemctl disable --now firewalld
```

3、所有节点时间同步

```
yum install -y chrony
systemctl enable --now chronyd
timedatectl set-timezone Asia/Shanghai
```

4、开始部署单节点kubernetes，对于 All-in-One，唯一的节点既是主节点，也是工作节点。

```
kk create cluster
```

5、同时部署kubernetes和kubesphere，可指定kubernetes版本或kubesphere版本

```
kk create cluster --with-kubernetes v1.20.4 --with-kubesphere v3.1.0
```

快速安装 KubeSphere 仅用于开发或测试，因为默认情况下它使用了基于 openEBS 的 Local Volume 提供储存服务。如果需要在生产环境安装，请参见[高可用配置安装](https://kubesphere.io/zh/docs/installing-on-linux/high-availability-configurations/ha-configuration/)。

您可以在 KubeSphere 安装前或安装后配置持久化储存服务。同时，KubeSphere 支持各种开源存储解决方案（例如 Ceph 和 GlusterFS）以及商业存储产品。有关在安装 KubeSphere 之前配置存储类型的详细说明，请参考[持久化存储配置](https://kubesphere.io/zh/docs/installing-on-linux/persistent-storage-configurations/understand-persistent-storage/)。

有关如何在安装 KubeSphere 之后配置存储类型，请参考[持久卷和存储类型](https://kubesphere.io/zh/docs/cluster-administration/persistent-volume-and-storage-class/)。

## 部署高可用集群

官方文档：[https://kubesphere.io/zh/docs/installing-on-linux/high-availability-configurations/internal-ha-configuration/](https://kubesphere.io/zh/docs/installing-on-linux/high-availability-configurations/internal-ha-configuration/)

KubeKey 作为一种集群安装工具，从版本 v1.2.1 开始，提供了内置高可用模式，支持一键部署高可用集群环境。KubeKey 的高可用模式实现方式称作本地负载均衡模式。具体表现为 KubeKey 会在每一个工作节点上部署一个负载均衡器（HAproxy），所有主节点的 Kubernetes 组件连接其本地的 kube-apiserver ，而所有工作节点的 Kubernetes 组件通过由 KubeKey 部署的负载均衡器反向代理到多个主节点的 kube-apiserver 。这种模式相较于专用到负载均衡器来说效率有所降低，因为会引入额外的健康检查机制，但是如果当前环境无法提供外部负载均衡器或者虚拟 IP（VIP）时这将是一种更实用、更有效、更方便的高可用部署模式。

多节点集群由至少一个主节点和一个工作节点组成。您可以使用任何节点作为任务机来执行安装任务，也可以在安装之前或之后根据需要新增节点，下面使用 KubeKey 内置 HAproxy 创建高可用集群。

部署环境：准备 6 台 Linux 机器，其中 3 台充当主节点，另外 3 台充当工作节点。下图展示了内置高可用模式的架构图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ab24cb61ba7437d81f94b8a86f3bd52.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAd2lsbGJsb2c=,size_20,color_FFFFFF,t_70,g_se,x_16)  
备注：/var/lib/docker 路径主要用于存储容器数据，在使用和操作过程中数据量会逐渐增加。因此，在生产环境中，建议为 /var/lib/docker 单独挂载一个硬盘。

创建示例配置文件，节点主机可不用提前修改，kubekey会基于集群配置文件自动纠正主机名。

```
kk create config
```

根据您的环境修改配置文件 config-sample.yaml，以下示例以部署3个master节点和3个node节点为例(不执行kubesphere部署，仅搭建kubernetes集群)：

```
cat > config-sample.yaml <<EOF
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: k8s-master1, address: 192.168.93.60, internalAddress: 192.168.93.60,user: root, password: 123456}
  - {name: k8s-master2, address: 192.168.93.61, internalAddress: 192.168.93.61,user: root, password: 123456}
  - {name: k8s-master3, address: 192.168.93.62, internalAddress: 192.168.93.62,user: root, password: 123456}
  - {name: k8s-node1, address: 192.168.93.63, internalAddress: 192.168.93.63,user: root, password: 123456}
  - {name: k8s-node2, address: 192.168.93.64, internalAddress: 192.168.93.64,user: root, password: 123456}
  - {name: k8s-node3, address: 192.168.93.65, internalAddress: 192.168.93.65,user: root, password: 123456}
  roleGroups:
    etcd:
    - k8s-master[1:3]
    master:
    - k8s-master[1:3]
    worker:
    - k8s-node[1:3]
  controlPlaneEndpoint:
    ##Internal loadbalancer for apiservers
    internalLoadbalancer: haproxy
    domain: lb.kubesphere.local
    address: ""      # The IP address of your load balancer.
    port: 6443
  kubernetes:
    version: v1.20.6
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
  addons: []
EOF
```

使用配置文件创建集群

```
export KKZONE=cn
kk create cluster -f config-sample.yaml | tee kk.log
```

创建完成后查看节点状态

```
[root@k8s-master1 ~]# kubectl get nodes -o wide
NAME          STATUS   ROLES                  AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k8s-master1   Ready    control-plane,master   4m58s   v1.20.6   192.168.93.60   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
k8s-master2   Ready    control-plane,master   3m58s   v1.20.6   192.168.93.61   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
k8s-master3   Ready    control-plane,master   3m58s   v1.20.6   192.168.93.62   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
k8s-node1     Ready    worker                 4m13s   v1.20.6   192.168.93.63   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
k8s-node2     Ready    worker                 3m59s   v1.20.6   192.168.93.64   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
k8s-node3     Ready    worker                 3m59s   v1.20.6   192.168.93.65   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64   docker://20.10.7
```

查看所有pod状态

```
[root@k8s-master1 ~]# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-8545b68dd4-rbshc   1/1     Running   2          3m48s
kube-system   calico-node-5k7b5                          1/1     Running   1          3m48s
kube-system   calico-node-6cv8z                          1/1     Running   1          3m48s
kube-system   calico-node-8rbjs                          1/1     Running   0          3m48s
kube-system   calico-node-d6wkc                          1/1     Running   0          3m48s
kube-system   calico-node-q8qp8                          1/1     Running   0          3m48s
kube-system   calico-node-rvqpj                          1/1     Running   0          3m48s
kube-system   coredns-7f87749d6c-66wqb                   1/1     Running   0          4m58s
kube-system   coredns-7f87749d6c-htqww                   1/1     Running   0          4m58s
kube-system   haproxy-k8s-node1                          1/1     Running   0          4m3s
kube-system   haproxy-k8s-node2                          1/1     Running   0          4m3s
kube-system   haproxy-k8s-node3                          1/1     Running   0          2m47s
kube-system   kube-apiserver-k8s-master1                 1/1     Running   0          5m13s
kube-system   kube-apiserver-k8s-master2                 1/1     Running   0          4m10s
kube-system   kube-apiserver-k8s-master3                 1/1     Running   0          4m16s
kube-system   kube-controller-manager-k8s-master1        1/1     Running   0          5m13s
kube-system   kube-controller-manager-k8s-master2        1/1     Running   0          4m10s
kube-system   kube-controller-manager-k8s-master3        1/1     Running   0          4m16s
kube-system   kube-proxy-2t5l6                           1/1     Running   0          3m55s
kube-system   kube-proxy-b8q6g                           1/1     Running   0          3m56s
kube-system   kube-proxy-dsz5g                           1/1     Running   0          3m55s
kube-system   kube-proxy-g2gxz                           1/1     Running   0          3m55s
kube-system   kube-proxy-p6gb7                           1/1     Running   0          3m57s
kube-system   kube-proxy-q44jp                           1/1     Running   0          3m56s
kube-system   kube-scheduler-k8s-master1                 1/1     Running   0          5m13s
kube-system   kube-scheduler-k8s-master2                 1/1     Running   0          4m10s
kube-system   kube-scheduler-k8s-master3                 1/1     Running   0          4m16s
kube-system   nodelocaldns-l958t                         1/1     Running   0          4m19s
kube-system   nodelocaldns-n7vkn                         1/1     Running   0          4m18s
kube-system   nodelocaldns-q6wjc                         1/1     Running   0          4m33s
kube-system   nodelocaldns-sfmcc                         1/1     Running   0          4m58s
kube-system   nodelocaldns-tvdbh                         1/1     Running   0          4m18s
kube-system   nodelocaldns-vg5t7                         1/1     Running   0          4m19s
```

## kubekey集群维护

1、添加节点

```
kk add nodes -f config-sample.yaml
```

2、 删除节点

```
kk delete node <nodeName> -f config-sample.yaml
```

3、删除集群

```
kk delete cluster

kk delete cluster [-f config-sample.yaml]
```

4、集群升级

```
kk upgrade [--with-kubernetes version] [--with-kubesphere version]

kk upgrade [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]
```