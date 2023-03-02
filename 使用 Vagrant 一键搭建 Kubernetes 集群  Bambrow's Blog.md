本文简要讲述了如何使用 Vagrant 一键搭建 Kubernetes 集群。本项目有针对国内网络进行优化的版本。请在开始前先安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 与 [Vagrant](https://www.vagrantup.com/docs/installation)。

## 简要说明

本项目默认安装具有一个 master 及两个 worker 的 Kubernetes 集群。使用的 IP 地址为：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>10.0.0.80  master</span><br><span>10.0.0.81  worker1</span><br><span>10.0.0.82  worker2</span><br></pre></td></tr></tbody></table>

如需要修改 worker 个数、IP 地址或者 Kubernetes 版本，所需修改的地方将在后文说明。

## 版本说明

本项目使用的 VirtualBox 及 Vagrant 版本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>VirtualBox: 6.1.20 r143896 (Qt5.6.2)</span><br><span>Vagrant: 2.2.16</span><br></pre></td></tr></tbody></table>

本项目使用的 Vagrant Box 为 [`ubuntu/bionic64`](https://app.vagrantup.com/ubuntu/boxes/bionic64)。

本项目搭建的集群版本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><span>kube-apiserver: v1.21.1</span><br><span>kube-proxy: v1.21.1</span><br><span>kube-controller-manager: v1.21.1</span><br><span>kube-scheduler: v1.21.1</span><br><span>pause: 3.4.1</span><br><span>coredns: v1.8.0</span><br><span>etcd: 3.4.13-0  </span><br></pre></td></tr></tbody></table>

## 使用方法

### 搭建集群

#### GitHub 版本

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>git <span>clone</span> https://github.com/bambrow/vagrant-k8s-cluster.git</span><br><span><span>cd</span> vagrant-k8s-cluster</span><br><span>vagrant up</span><br></pre></td></tr></tbody></table>

#### Gitee 版本（针对国内网络优化）

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>git <span>clone</span> https://gitee.com/bambrow/vagrant-k8s-cluster-cn.git</span><br><span><span>cd</span> vagrant-k8s-cluster-cn</span><br><span>vagrant up</span><br></pre></td></tr></tbody></table>

### 连接节点

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>vagrant ssh master</span><br><span>vagrant ssh worker1</span><br><span>vagrant ssh worker2</span><br></pre></td></tr></tbody></table>

如果一切启动顺利，可以进入 master 节点查看状态：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br></pre></td><td><pre><span>$ kubectl get nodes -o wide</span><br><span>NAME      STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME</span><br><span>master    Ready    control-plane,master   48m     v1.21.1   10.0.0.80     &lt;none&gt;        Ubuntu 18.04.5 LTS   4.15.0-147-generic   docker://20.10.12</span><br><span>worker1   Ready    &lt;none&gt;                 7m28s   v1.21.1   10.0.0.81     &lt;none&gt;        Ubuntu 18.04.5 LTS   4.15.0-147-generic   docker://20.10.12</span><br><span>worker2   Ready    &lt;none&gt;                 21m     v1.21.1   10.0.0.82     &lt;none&gt;        Ubuntu 18.04.5 LTS   4.15.0-147-generic   docker://20.10.12</span><br><span></span><br><span>$ kubectl -n kube-system get all -o wide</span><br><span>NAME                                           READY   STATUS    RESTARTS   AGE     IP              NODE      NOMINATED NODE   READINESS GATES</span><br><span>pod/calico-kube-controllers-6b9fbfff44-4jpvc   1/1     Running   0          50m     10.244.219.66   master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/calico-node-p4qb2                          1/1     Running   0          23m     10.0.0.82       worker2   &lt;none&gt;           &lt;none&gt;</span><br><span>pod/calico-node-tcsxt                          1/1     Running   0          9m47s   10.0.0.81       worker1   &lt;none&gt;           &lt;none&gt;</span><br><span>pod/calico-node-wl74r                          1/1     Running   0          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/coredns-558bd4d5db-78sjf                   1/1     Running   0          50m     10.244.219.67   master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/coredns-558bd4d5db-jft6t                   1/1     Running   0          50m     10.244.219.65   master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/etcd-master                                1/1     Running   0          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-apiserver-master                      1/1     Running   0          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-controller-manager-master             1/1     Running   8          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-proxy-h4vnv                           1/1     Running   0          9m47s   10.0.0.81       worker1   &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-proxy-jgd5h                           1/1     Running   1          23m     10.0.0.82       worker2   &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-proxy-pznsv                           1/1     Running   0          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span>pod/kube-scheduler-master                      1/1     Running   3          50m     10.0.0.80       master    &lt;none&gt;           &lt;none&gt;</span><br><span></span><br><span>NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR</span><br><span>service/kube-dns   ClusterIP   10.96.0.10   &lt;none&gt;        53/UDP,53/TCP,9153/TCP   50m   k8s-app=kube-dns</span><br><span></span><br><span>NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS    IMAGES</span><br><span>               SELECTOR</span><br><span>daemonset.apps/calico-node   3         3         3       3            3           kubernetes.io/os=linux   50m   calico-node   docker.io/calico/node:v3.21.2   k8s-app=calico-node</span><br><span>daemonset.apps/kube-proxy    3         3         3       3            3           kubernetes.io/os=linux   50m   kube-proxy    k8s.gcr.io/kube-proxy:v1.21.1   k8s-app=kube-proxy</span><br><span></span><br><span>NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                IMAGES</span><br><span>       SELECTOR</span><br><span>deployment.apps/calico-kube-controllers   1/1     1            1           50m   calico-kube-controllers   docker.io/calico/kube-controllers:v3.21.2   k8s-app=calico-kube-controllers</span><br><span>deployment.apps/coredns                   2/2     2            2           50m   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0           k8s-app=kube-dns</span><br><span></span><br><span>NAME                                                 DESIRED   CURRENT   READY   AGE   CONTAINERS                IMAGES</span><br><span>             SELECTOR</span><br><span>replicaset.apps/calico-kube-controllers-6b9fbfff44   1         1         1       50m   calico-kube-controllers   docker.io/calico/kube-controllers:v3.21.2   k8s-app=calico-kube-controllers,pod-template-hash=6b9fbfff44</span><br><span>replicaset.apps/coredns-558bd4d5db                   2         2         2       50m   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0           k8s-app=kube-dns,pod-template-hash=558bd4d5db</span><br></pre></td></tr></tbody></table>

### 暂停与启动集群

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>vagrant halt</span><br><span>vagrant up</span><br></pre></td></tr></tbody></table>

如需暂停或启动（包括销毁后重建）某个节点，可以使用 `vagrant halt|up master|worker1|worker2`。

### 销毁集群

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>vagrant destroy -f</span><br></pre></td></tr></tbody></table>

如需销毁某个节点，可以使用 `vagrant destroy -f master|worker1|worker2`。

## 特殊设置

### 修改 worker 个数

打开 `Vagrantfile`，修改如下设置：

Vagrantfile

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><span>...</span><br><span><span>- (1..2).each do |i|</span></span><br><span><span>+ (1..[number]).each do |i|</span></span><br><span>    config.vm.define "worker#{i}" do |worker|</span><br><span>      worker.vm.box = "ubuntu/bionic64"</span><br><span>      worker.vm.hostname = "worker#{i}"</span><br><span>...</span><br></pre></td></tr></tbody></table>

将 `[number]` 改为你需要的数目即可。请注意，在修改完毕后需要在 hosts 设置处添加或删除对应的行。例如，添加 `worker3` 后需要做如下修改：

Vagrantfile

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span>...</span><br><span>    echo "10.0.0.80  master" &gt;&gt; /etc/hosts</span><br><span>    echo "10.0.0.81  worker1" &gt;&gt; /etc/hosts</span><br><span>    echo "10.0.0.82  worker2" &gt;&gt; /etc/hosts</span><br><span><span>+   echo "10.0.0.83  worker3" &gt;&gt; /etc/hosts</span></span><br><span>...</span><br></pre></td></tr></tbody></table>

### 修改 IP 地址

打开 `Vagrantfile`，修改如下设置：

Vagrantfile

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>echo "10.0.0.80  master" &gt;&gt; /etc/hosts</span><br><span>echo "10.0.0.81  worker1" &gt;&gt; /etc/hosts</span><br><span>echo "10.0.0.82  worker2" &gt;&gt; /etc/hosts</span><br></pre></td></tr></tbody></table>

把三个 IP 设置为你想要的 IP 即可。然后，修改下面的 `master.vm.network` 与 `worker.vm.network`。请注意，`worker.vm.network` 的 IP 必须是连续的。

此外，还需要修改 `master.sh` 中的 `MASTER_IP`。

### 修改 Kubernetes 版本

首先，修改 `common.sh` 中的 `KUBERNETES_VERSION`。其次，需要修改 `master.sh` 中的 `KUBE_VERSION`。

如果你使用的是针对国内网络优化的 Gitee 版本，需要注意以下情况：

由于镜像的 image 版本问题，对于 Kubernetes v1.21.1，`coredns` 的 tag 有错误，因此在 `master.sh` 中有修复 tag 的语句：

master.sh

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span></span><br><span>COREDNS_VERSION=1.8.0</span><br><span>sudo docker pull registry.aliyuncs.com/google_containers/coredns:<span>$COREDNS_VERSION</span></span><br><span>sudo docker tag registry.aliyuncs.com/google_containers/coredns:<span>$COREDNS_VERSION</span> registry.aliyuncs.com/google_containers/coredns/coredns:v<span>$COREDNS_VERSION</span></span><br></pre></td></tr></tbody></table>

如果使用其他版本的 Kubernetes，这个问题可能不会出现，因此可以按需要修改 `COREDNS_VERSION` 或直接注释掉这几行即可。