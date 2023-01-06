## 文章目录

-   [环境说明](https://www.treesir.pub/post/kubekey-install-k8s/#%E7%8E%AF%E5%A2%83%E8%AF%B4%E6%98%8E)
    -   [概述](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%A6%82%E8%BF%B0)
    -   [环境说明](https://www.treesir.pub/post/kubekey-install-k8s/#%E7%8E%AF%E5%A2%83%E8%AF%B4%E6%98%8E-1)
-   [`online` 模式部署](https://www.treesir.pub/post/kubekey-install-k8s/#online-%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2)
    -   [下载源码 编译 kubekey](https://www.treesir.pub/post/kubekey-install-k8s/#%E4%B8%8B%E8%BD%BD%E6%BA%90%E7%A0%81-%E7%BC%96%E8%AF%91-kubekey)
    -   [创建配置文件](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    -   [启动集群](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4)
    -   [offline 部署](https://www.treesir.pub/post/kubekey-install-k8s/#offline-%E9%83%A8%E7%BD%B2)
-   [部署前准备](https://www.treesir.pub/post/kubekey-install-k8s/#%E9%83%A8%E7%BD%B2%E5%89%8D%E5%87%86%E5%A4%87)
    -   [解压文件](https://www.treesir.pub/post/kubekey-install-k8s/#%E8%A7%A3%E5%8E%8B%E6%96%87%E4%BB%B6)
    -   [创建集群配置文件](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    -   [修改默认生成的配置文件展示](https://www.treesir.pub/post/kubekey-install-k8s/#%E4%BF%AE%E6%94%B9%E9%BB%98%E8%AE%A4%E7%94%9F%E6%88%90%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%B1%95%E7%A4%BA)
    -   [节点环境初始化](https://www.treesir.pub/post/kubekey-install-k8s/#%E8%8A%82%E7%82%B9%E7%8E%AF%E5%A2%83%E5%88%9D%E5%A7%8B%E5%8C%96)
    -   [上传镜像至私服中](https://www.treesir.pub/post/kubekey-install-k8s/#%E4%B8%8A%E4%BC%A0%E9%95%9C%E5%83%8F%E8%87%B3%E7%A7%81%E6%9C%8D%E4%B8%AD)
    -   [启动集群](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4-1)
-   [扩展集群角色的配置](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A9%E5%B1%95%E9%9B%86%E7%BE%A4%E8%A7%92%E8%89%B2%E7%9A%84%E9%85%8D%E7%BD%AE)
    -   [nginx 负载均衡配置](https://www.treesir.pub/post/kubekey-install-k8s/#nginx-%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E9%85%8D%E7%BD%AE)
    -   [更改kubekey 配置文件](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%9B%B4%E6%94%B9kubekey-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    -   [执行添加节点](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A7%E8%A1%8C%E6%B7%BB%E5%8A%A0%E8%8A%82%E7%82%B9)
        -   [初始化新添加的节点](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9D%E5%A7%8B%E5%8C%96%E6%96%B0%E6%B7%BB%E5%8A%A0%E7%9A%84%E8%8A%82%E7%82%B9)
        -   [执行节点添加](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A7%E8%A1%8C%E8%8A%82%E7%82%B9%E6%B7%BB%E5%8A%A0)
-   [更具体操作请参考下述文档](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%9B%B4%E5%85%B7%E4%BD%93%E6%93%8D%E4%BD%9C%E8%AF%B7%E5%8F%82%E8%80%83%E4%B8%8B%E8%BF%B0%E6%96%87%E6%A1%A3)

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E7%8E%AF%E5%A2%83%E8%AF%B4%E6%98%8E)环境说明

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%A6%82%E8%BF%B0)概述

> [KubeSphere](https://kubesphere.io/) 是在 [Kubernetes](https://kubernetes.io/) 之上构建的面向云原生应用的**分布式操作系统**，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。
> 
> [KubeKey](https://github.com/kubesphere/kubekey)（由 Go 语言开发且`代码开源`）是一种全新的安装工具，替代了以前使用的基于 ansible 的安装程序。KubeKey 提供灵活的安装选择，可以仅安装 `Kubernetes`，也可以同时安装 Kubernetes 和 KubeSphere。
> 
> 使用场景：
> 
> -   仅安装 Kubernetes；
> -   使用一个命令同时安装 Kubernetes 和 KubeSphere；
> -   扩缩集群；
> -   升级集群；
> -   安装 Kubernetes 相关的插件（Chart 或 YAML）。

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E7%8E%AF%E5%A2%83%E8%AF%B4%E6%98%8E-1)环境说明

-   操作系统: `CentOS Linux release 7.8.2003 (Core)`
-   服务器 `cpu` 架构: `x86_64`

## [](https://www.treesir.pub/post/kubekey-install-k8s/#online-%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2)`online` 模式部署

> 此次演示，准备两台只安装了操作系统和配置了ip地址的主机。
> 
> -   `node1`: 192.168.8.30
> -   `node2`: 192.168.8.31

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">yum install -y git 

git clone https://github.com/kubesphere/kubekey.git <span>\
</span><span></span><span>&amp;&amp;</span> <span>cd</span> kubekey 

./build.sh -p  <span># 执行编译</span>

ls  -lh output/kk <span># 编译结束后，会在 output 目录下，生成可执行程序 kk </span>
-rwxr-xr-x <span>1</span> root root 57M 4月  <span>26</span> 15:17 output/kk

output/kk version
version.BuildInfo<span>{</span>Version:<span>"latest+unreleased"</span>, GitCommit:<span>"22d2e7648569a7e538cbbd6505ce9aca4180d6a1"</span>, GitTreeState:<span>"clean"</span>, GoVersion:<span>"go1.14.7"</span><span>}</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)创建配置文件

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">scp output/kk 192.168.8.30:/usr/local/bin/ <span># copy 编译好的命令至 node1</span>

<span>[</span>root@localhost ~<span>]</span><span># kk version  # node1</span>
version.BuildInfo<span>{</span>Version:<span>"latest+unreleased"</span>, GitCommit:<span>"22d2e7648569a7e538cbbd6505ce9aca4180d6a1"</span>, GitTreeState:<span>"clean"</span>, GoVersion:<span>"go1.14.7"</span><span>}</span>

./kk create config --with-kubernetes v1.17.9  <span># 创建 v1.17.9 版本的配置文件，版本列表请查看此链接https://github.com/kubesphere/kubekey/blob/master/docs/kubernetes-versions.md</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4)启动集群

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span><span>17
</span><span>18
</span><span>19
</span><span>20
</span><span>21
</span><span>22
</span><span>23
</span><span>24
</span><span>25
</span><span>26
</span><span>27
</span><span>28
</span><span>29
</span><span>30
</span><span>31
</span><span>32
</span><span>33
</span><span>34
</span><span>35
</span><span>36
</span><span>37
</span><span>38
</span><span>39
</span><span>40
</span><span>41
</span><span>42
</span><span>43
</span><span>44
</span><span>45
</span><span>46
</span><span>47
</span><span>48
</span><span>49
</span><span>50
</span><span>51
</span><span>52
</span><span>53
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">cat config-sample.yaml  <span># 配置文件展示</span>

apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - <span>{</span>name: node1, address: 192.168.8.30, internalAddress: 192.168.8.30, user: root, password: 123456<span>}</span> <span># 将对应主机的密码填入</span>
  - <span>{</span>name: node2, address: 192.168.8.31, internalAddress: 192.168.8.31, user: root, password: 123456<span>}</span>
  roleGroups: <span># 配置机器角色</span>
    etcd:
    - node1  
    master:
    - node1
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: <span>""</span>
    port: <span>6443</span>
  kubernetes:
    version: v1.17.9
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: <span>[]</span>
    insecureRegistries: <span>[]</span>
  addons: <span>[]</span>
  
kk init os -f ./config-sample.yaml  <span># 初始化集群机器，此操作会安装系统依赖，开启lpvs模块等。前提是确保对应node的网络通畅。</span>

kk create cluster -f ./config-sample.yaml  <span># 创建集群</span>

kubectl get po --all-namespaces <span># 等待 30s 左右就会初始化完成</span>
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-7b4f58f565-6wdl2   0/1     Pending   <span>0</span>          11s
kube-system   calico-node-f99fc                          0/1     Running   <span>0</span>          11s
kube-system   calico-node-lnhc8                          1/1     Running   <span>0</span>          11s
kube-system   coredns-74d59cc5c6-nrdsn                   0/1     Pending   <span>0</span>          46s
kube-system   coredns-74d59cc5c6-zftdf                   0/1     Pending   <span>0</span>          46s
kube-system   kube-apiserver-node1                       1/1     Running   <span>0</span>          41s
kube-system   kube-controller-manager-node1              1/1     Running   <span>0</span>          41s
kube-system   kube-proxy-l245l                           1/1     Running   <span>0</span>          14s
kube-system   kube-proxy-mgvfx                           1/1     Running   <span>0</span>          46s
kube-system   kube-scheduler-node1                       1/1     Running   <span>0</span>          41s
kube-system   nodelocaldns-k2qnm                         1/1     Running   <span>0</span>          46s
kube-system   nodelocaldns-rz8h9                         1/1     Running   <span>0</span>          14s
</code></pre></td></tr></tbody></table>

[![image-20210426160358377](https://cdn.treesir.pub/img/image-20210426160358377.png)](https://cdn.treesir.pub/img/image-20210426160358377.png)

[![image-20210426160850178](https://cdn.treesir.pub/img/image-20210426160850178.png)](https://cdn.treesir.pub/img/image-20210426160850178.png)

## [](https://www.treesir.pub/post/kubekey-install-k8s/#offline-%E9%83%A8%E7%BD%B2)offline 部署

> -   [参考文档](https://kubesphere.com.cn/forum/d/2034-kubekey-kubesphere-v300)
> 
> 上述文档中，离线部署已 `非常详细`，下面演示步骤中，且 `着重` 展示去掉 `无用` 操作的优化项。
> 
> 文档中 `kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz`文件下载，那个下载太慢了 (限速)
> 
> -   [百度云盘链接](https://pan.baidu.com/s/1WBYN3bFkKe-6KqCJWbQQcQ) : (bfvp)。

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E9%83%A8%E7%BD%B2%E5%89%8D%E5%87%86%E5%A4%87)部署前准备

> 环境与`online` 部署环境基本一致，只是网络是 `无法访问外网` 的。

-   **取消机器默认路由，使其无法访问外网**

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">ip route add default via 192.168.8.253 <span># 将默认路由指向一个不存在的网关</span>

ping -c3  223.5.5.5
PING 223.5.5.5 <span>(</span>223.5.5.5<span>)</span> 56<span>(</span>84<span>)</span> bytes of data.
From 192.168.8.31 <span>icmp_seq</span><span>=</span><span>1</span> Destination Host Unreachable
From 192.168.8.31 <span>icmp_seq</span><span>=</span><span>2</span> Destination Host Unreachable
From 192.168.8.31 <span>icmp_seq</span><span>=</span><span>3</span> Destination Host Unreachable
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E8%A7%A3%E5%8E%8B%E6%96%87%E4%BB%B6)解压文件

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">tar xf kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz 
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)创建集群配置文件

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">./kk create config --with-kubernetes v1.17.9 
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E4%BF%AE%E6%94%B9%E9%BB%98%E8%AE%A4%E7%94%9F%E6%88%90%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%B1%95%E7%A4%BA)修改默认生成的配置文件展示

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span><span>17
</span><span>18
</span><span>19
</span><span>20
</span><span>21
</span><span>22
</span><span>23
</span><span>24
</span><span>25
</span><span>26
</span><span>27
</span><span>28
</span><span>29
</span><span>30
</span><span>31
</span><span>32
</span><span>33
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - <span>{</span>name: node1, address: 192.168.8.30, internalAddress: 192.168.8.30, user: root, password: 123456<span>}</span>
  - <span>{</span>name: node2, address: 192.168.8.31, internalAddress: 192.168.8.31, user: root, password: 123456<span>}</span>
  roleGroups:
    etcd:
    - node1
    master: 
    - node1
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: <span>""</span>
    port: <span>"6443"</span>
  kubernetes:
    version: v1.17.9
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: <span>[]</span>
    insecureRegistries: <span>[]</span>
    privateRegistry: dockerhub.kubekey.local  <span># 添加使用 kubekey 创建的私服</span>
  addons: <span>[]</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E8%8A%82%E7%82%B9%E7%8E%AF%E5%A2%83%E5%88%9D%E5%A7%8B%E5%8C%96)节点环境初始化

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span><span>9
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">./kk init os -f config-sample.yaml -s ./dependencies/

docker load &lt; kubesphere-images-v3.0.0/registry.tar  <span># 加载私服镜像</span>

./kk init os -f config-sample.yaml -s ./dependencies/ --add-images-repo <span># 启动镜像私服</span>

docker ps <span># 可以看到私服已经启动了</span>
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
741cbeff3dee        registry:2          <span>"/entrypoint.sh /etc…"</span>   <span>19</span> seconds ago      Up <span>17</span> seconds       0.0.0.0:443-&gt;443/tcp, 5000/tcp   kubekey-registry
</code></pre></td></tr></tbody></table>

[![image-20210426170119026](https://cdn.treesir.pub/img/image-20210426170119026.png)](https://cdn.treesir.pub/img/image-20210426170119026.png)

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E4%B8%8A%E4%BC%A0%E9%95%9C%E5%83%8F%E8%87%B3%E7%A7%81%E6%9C%8D%E4%B8%AD)上传镜像至私服中

> 此步骤比较漫长，可以更具实际使用情况，进行删除 `images-list-v3.0.0.txt` 中的镜像列表 和 路径下的 `*.tar` 结尾的镜像压缩包

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span>cd</span> kubesphere-images-v3.0.0 <span>\
</span><span></span><span>&amp;&amp;</span>./push-images.sh  dockerhub.kubekey.local  <span># 加载路径下的所有 *.tar 结尾的镜像压缩包，并根据 images-list-v3.0.0.txt 中的镜像列表，进行上传至刚才创建的私服中。</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4-1)启动集群

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">./kk create cluster -f config-sample.yaml

kubectl get po --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-677cbc8557-c9n9b   1/1     Running   <span>0</span>          48s
kube-system   calico-node-bhjwf                          1/1     Running   <span>0</span>          48s
kube-system   calico-node-f278s                          1/1     Running   <span>0</span>          24s
kube-system   coredns-79878cb9c9-g5drn                   1/1     Running   <span>0</span>          63s
kube-system   coredns-79878cb9c9-lsh7n                   1/1     Running   <span>0</span>          63s
kube-system   kube-apiserver-node1                       1/1     Running   <span>0</span>          58s
kube-system   kube-controller-manager-node1              1/1     Running   <span>0</span>          58s
kube-system   kube-proxy-5zkpg                           1/1     Running   <span>0</span>          63s
kube-system   kube-proxy-c986s                           1/1     Running   <span>0</span>          24s
kube-system   kube-scheduler-node1                       1/1     Running   <span>0</span>          58s
kube-system   nodelocaldns-p5fkv                         1/1     Running   <span>0</span>          63s
kube-system   nodelocaldns-phh4s                         1/1     Running   <span>0</span>          24s
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A9%E5%B1%95%E9%9B%86%E7%BE%A4%E8%A7%92%E8%89%B2%E7%9A%84%E9%85%8D%E7%BD%AE)扩展集群角色的配置

> 示例将，对上面部署的集群进行扩展，分别扩展 `master` & `worker` 各一台，扩展中 master apiserver 负载均衡器将使用 `nginx` 承担

**本次新增节点列表如下所示**

| 主机名称 | 集群角色 | IP地址 |
| --- | --- | --- |
| node3 | master | 192.168.8.32 |
| node4 | worker | 192.168.8.33 |
| manage | `lb` (nginx) | 192.168.8.88 |

## [](https://www.treesir.pub/post/kubekey-install-k8s/#nginx-%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E9%85%8D%E7%BD%AE)nginx 负载均衡配置

**安装**

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">yum install -y wget

mv /etc/yum.repos.d<span>{</span>,.bak<span>}</span> <span>\
</span><span></span><span>&amp;&amp;</span> mkdir -p /etc/yum.repos.d <span>\
</span><span></span><span>&amp;&amp;</span> curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo <span>\
</span><span></span><span>&amp;&amp;</span> wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo <span>\
</span><span></span><span>&amp;&amp;</span> yum clean all <span>\
</span><span></span><span>&amp;&amp;</span> yum makecache fast


yum install -y nginx*
</code></pre></td></tr></tbody></table>

**负载均衡配置**

<table><tbody><tr><td><pre tabindex="0"><code><span>  1
</span><span>  2
</span><span>  3
</span><span>  4
</span><span>  5
</span><span>  6
</span><span>  7
</span><span>  8
</span><span>  9
</span><span> 10
</span><span> 11
</span><span> 12
</span><span> 13
</span><span> 14
</span><span> 15
</span><span> 16
</span><span> 17
</span><span> 18
</span><span> 19
</span><span> 20
</span><span> 21
</span><span> 22
</span><span> 23
</span><span> 24
</span><span> 25
</span><span> 26
</span><span> 27
</span><span> 28
</span><span> 29
</span><span> 30
</span><span> 31
</span><span> 32
</span><span> 33
</span><span> 34
</span><span> 35
</span><span> 36
</span><span> 37
</span><span> 38
</span><span> 39
</span><span> 40
</span><span> 41
</span><span> 42
</span><span> 43
</span><span> 44
</span><span> 45
</span><span> 46
</span><span> 47
</span><span> 48
</span><span> 49
</span><span> 50
</span><span> 51
</span><span> 52
</span><span> 53
</span><span> 54
</span><span> 55
</span><span> 56
</span><span> 57
</span><span> 58
</span><span> 59
</span><span> 60
</span><span> 61
</span><span> 62
</span><span> 63
</span><span> 64
</span><span> 65
</span><span> 66
</span><span> 67
</span><span> 68
</span><span> 69
</span><span> 70
</span><span> 71
</span><span> 72
</span><span> 73
</span><span> 74
</span><span> 75
</span><span> 76
</span><span> 77
</span><span> 78
</span><span> 79
</span><span> 80
</span><span> 81
</span><span> 82
</span><span> 83
</span><span> 84
</span><span> 85
</span><span> 86
</span><span> 87
</span><span> 88
</span><span> 89
</span><span> 90
</span><span> 91
</span><span> 92
</span><span> 93
</span><span> 94
</span><span> 95
</span><span> 96
</span><span> 97
</span><span> 98
</span><span> 99
</span><span>100
</span><span>101
</span><span>102
</span><span>103
</span><span>104
</span><span>105
</span><span>106
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">cat nginx.conf

user nginx nginx<span>;</span>
worker_processes auto<span>;</span>

error_log /data/wwwlogs/error_nginx.log crit<span>;</span>
pid /var/run/nginx.pid<span>;</span>
worker_rlimit_nofile 51200<span>;</span>
load_module <span>"/usr/lib64/nginx/modules/ngx_stream_module.so"</span><span>;</span>

events <span>{</span>
  use epoll<span>;</span>
  worker_connections 51200<span>;</span>
  multi_accept on<span>;</span>
<span>}</span>

http <span>{</span>
  include mime.types<span>;</span>
  default_type application/octet-stream<span>;</span>
  server_names_hash_bucket_size 128<span>;</span>
  client_header_buffer_size 32k<span>;</span>
  large_client_header_buffers <span>4</span> 32k<span>;</span>
  client_max_body_size 1024m<span>;</span>
  client_body_buffer_size 10m<span>;</span>
  sendfile on<span>;</span>
  tcp_nopush on<span>;</span>
  keepalive_timeout 120<span>;</span>
  server_tokens off<span>;</span>
  tcp_nodelay on<span>;</span>

  fastcgi_connect_timeout 300<span>;</span>
  fastcgi_send_timeout 300<span>;</span>
  fastcgi_read_timeout 300<span>;</span>
  fastcgi_buffer_size 64k<span>;</span>
  fastcgi_buffers <span>4</span> 64k<span>;</span>
  fastcgi_busy_buffers_size 128k<span>;</span>
  fastcgi_temp_file_write_size 128k<span>;</span>
  fastcgi_intercept_errors on<span>;</span>

  <span>#Gzip Compression</span>
  gzip on<span>;</span>
  gzip_buffers <span>16</span> 8k<span>;</span>
  gzip_comp_level 6<span>;</span>
  gzip_http_version 1.1<span>;</span>
  gzip_min_length 256<span>;</span>
  gzip_proxied any<span>;</span>
  gzip_vary on<span>;</span>
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon<span>;</span>
  gzip_disable <span>"MSIE [1-6]\.(?!.*SV1)"</span><span>;</span>

  <span>##Brotli Compression</span>
  <span>#brotli on;</span>
  <span>#brotli_comp_level 6;</span>
  <span>#brotli_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;</span>

  <span>##If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can save some latency.</span>
  <span>#open_file_cache max=1000 inactive=20s;</span>
  <span>#open_file_cache_valid 30s;</span>
  <span>#open_file_cache_min_uses 2;</span>
  <span>#open_file_cache_errors on;</span>

<span>######################## default ############################</span>
  include vhost/*.conf<span>;</span>
<span>}</span>

stream <span>{</span>
    upstream kubekey_apiserver <span>{</span>
        least_conn<span>;</span>
        server 192.168.8.30:6443 <span>max_fails</span><span>=</span><span>3</span> <span>fail_timeout</span><span>=</span>5s<span>;</span>
        server 192.168.8.32:6443 <span>max_fails</span><span>=</span><span>3</span> <span>fail_timeout</span><span>=</span>5s<span>;</span>
    <span>}</span>
    server <span>{</span>
        listen     6443<span>;</span>
        proxy_pass kubekey_apiserver<span>;</span>
    <span>}</span>
<span>}</span>


mkdir -p /data/wwwlogs/ <span>\
</span><span></span><span>&amp;&amp;</span> chmod <span>777</span> -R /data/wwwlogs/ <span># 创建日志目录</span>


nginx -t <span># 检查配置文件</span>
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf <span>test</span> is successful

sed -i <span>'s#SELINUX=enforcing#SELINUX=disabled#g'</span> /etc/selinux/config  <span># 关闭 selinux，防止启动报错</span>
grep -i  ^selinux<span>=</span> /etc/selinux/config 
setenforce <span>0</span>
getenforce

service firewalld stop <span>\
</span><span></span><span>&amp;&amp;</span> systemctl disable firewalld <span># 关闭防火墙</span>

service nginx start <span>\
</span><span></span><span>&amp;&amp;</span> systemctl <span>enable</span> nginx  <span># 启动 nginx 并设置开机自启</span>


ss -lntp<span>|</span>grep <span>":6443"</span>  <span># 检查端口</span>
LISTEN     <span>0</span>      <span>128</span>          *:6443                     *:*                   users:<span>((</span><span>"nginx"</span>,pid<span>=</span>8513,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8512,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8511,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8510,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8509,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8508,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8507,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8506,fd<span>=</span>5<span>)</span>,<span>(</span><span>"nginx"</span>,pid<span>=</span>8495,fd<span>=</span>5<span>))</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%9B%B4%E6%94%B9kubekey-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)更改kubekey 配置文件

> 修改后的配置文件展示

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span><span>17
</span><span>18
</span><span>19
</span><span>20
</span><span>21
</span><span>22
</span><span>23
</span><span>24
</span><span>25
</span><span>26
</span><span>27
</span><span>28
</span><span>29
</span><span>30
</span><span>31
</span><span>32
</span><span>33
</span><span>34
</span><span>35
</span><span>36
</span><span>37
</span><span>38
</span><span>39
</span><span>40
</span><span>41
</span><span>42
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">cat config-sample.yaml 

apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - <span>{</span>name: node1, address: 192.168.8.30, internalAddress: 192.168.8.30, user: root, password: 123456<span>}</span>
  - <span>{</span>name: node2, address: 192.168.8.31, internalAddress: 192.168.8.31, user: root, password: 123456<span>}</span>
  - <span>{</span>name: node3, address: 192.168.8.32, internalAddress: 192.168.8.32, user: root, password: 123456<span>}</span>
  - <span>{</span>name: node4, address: 192.168.8.33, internalAddress: 192.168.8.33, user: root, password: 123456<span>}</span>
  roleGroups:
    etcd:  <span># 这里的 etcd 的节点数需要是奇数,所以也将 node2 也加入至 etcd集群中。</span>
    - node1
    - node2 
    - node3
    master:
    - node1
    - node3
    worker:
    - node1
    - node2
    - node3
    - node4
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: <span>"192.168.8.88"</span>
    port: <span>"6443"</span>
  kubernetes:
    version: v1.17.9
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: <span>[]</span>
    insecureRegistries: <span>[]</span>
    privateRegistry: dockerhub.kubekey.local  <span># 添加使用 kubekey 创建的私服</span>
  addons: <span>[]</span>
</code></pre></td></tr></tbody></table>

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A7%E8%A1%8C%E6%B7%BB%E5%8A%A0%E8%8A%82%E7%82%B9)执行添加节点

### [](https://www.treesir.pub/post/kubekey-install-k8s/#%E5%88%9D%E5%A7%8B%E5%8C%96%E6%96%B0%E6%B7%BB%E5%8A%A0%E7%9A%84%E8%8A%82%E7%82%B9)初始化新添加的节点

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">./kk init os -f config-sample.yaml -s ./dependencies/ --add-images-repo <span># 示例适用上面离线安装的步骤，如 online部署，取消 -s 选项</span>
</code></pre></td></tr></tbody></table>

[![image-20210427113449801](https://cdn.treesir.pub/img/image-20210427113449801.png)](https://cdn.treesir.pub/img/image-20210427113449801.png)

### [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%89%A7%E8%A1%8C%E8%8A%82%E7%82%B9%E6%B7%BB%E5%8A%A0)执行节点添加

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">service firewalld stop <span>\
</span><span></span><span>&amp;&amp;</span> systemctl disable firewalld  <span># 关闭防火墙，防止加入节点时端口不同</span>

sed -i <span>'s#SELINUX=enforcing#SELINUX=disabled#g'</span> /etc/selinux/config  <span># 关闭 selinux，防止启动报错</span>
grep -i  ^selinux<span>=</span> /etc/selinux/config 
setenforce <span>0</span>
getenforce 

./kk add nodes -f ./config-sample.yaml 

kubectl get nodes -o wide
NAME    STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
node1   Ready    master,worker   8d      v1.17.9   192.168.8.30   &lt;none&gt;        CentOS Linux <span>7</span> <span>(</span>Core<span>)</span>   3.10.0-1127.el7.x86_64   docker://19.3.13
node2   Ready    worker          8d      v1.17.9   192.168.8.31   &lt;none&gt;        CentOS Linux <span>7</span> <span>(</span>Core<span>)</span>   3.10.0-1127.el7.x86_64   docker://19.3.13
node3   Ready    master,worker   2m23s   v1.17.9   192.168.8.32   &lt;none&gt;        CentOS Linux <span>7</span> <span>(</span>Core<span>)</span>   3.10.0-1127.el7.x86_64   docker://19.3.13
node4   Ready    worker          2m26s   v1.17.9   192.168.8.33   &lt;none&gt;        CentOS Linux <span>7</span> <span>(</span>Core<span>)</span>   3.10.0-1127.el7.x86_64   docker://19.3.13
</code></pre></td></tr></tbody></table>

> 在上面示例中，执行的前两步，可以在 github 中找到对应 kubekey仓库，clone 最新源码，在源码中做对应添加后，再执行编译，这样我们就不需要每台多去执行了。

[![image-20210427162435941](https://cdn.treesir.pub/img/image-20210427162435941.png)](https://cdn.treesir.pub/img/image-20210427162435941.png)

[![image-20210427162412720](https://cdn.treesir.pub/img/image-20210427162412720.png)](https://cdn.treesir.pub/img/image-20210427162412720.png)

[![image-20210427162558416](https://cdn.treesir.pub/img/image-20210427162558416.png)](https://cdn.treesir.pub/img/image-20210427162558416.png)

## [](https://www.treesir.pub/post/kubekey-install-k8s/#%E6%9B%B4%E5%85%B7%E4%BD%93%E6%93%8D%E4%BD%9C%E8%AF%B7%E5%8F%82%E8%80%83%E4%B8%8B%E8%BF%B0%E6%96%87%E6%A1%A3)更具体操作请参考下述文档

-   [https://kubesphere.io/zh/docs/installing-on-linux/introduction/kubekey/](https://kubesphere.io/zh/docs/installing-on-linux/introduction/kubekey/)
-   [https://github.com/kubesphere/kubekey](https://github.com/kubesphere/kubekey)

文章作者 Yang'zun

上次更新 2021-04-26

许可协议 [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)