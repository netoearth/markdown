本文简要介绍了 NFS 服务器的搭建以及在 Kubernetes 中的使用 Storage Class 进行 NFS 挂载的操作步骤。

NFS 是 Network File System 的缩写，是 Sun 公司于 1984 年开发的一种分布式文件系统协议。它的核心功能就是可以通过网络，让不同的客户端可以彼此访问共同的文件系统，来实现文件的共享。像许多其他的协议，NFS 建立在开放的网络计算的远程过程调用（RPC）之上。NFS 服务器可以让客户端将网络远程的 NFS 服务器分享的目录，直接挂载到本地端的机器当中。本地端的机器通过直接读写挂载的目录，就可以同步到 NFS 服务器之上 [\[1\]](https://bambrow.com/20210908-nfs-storage-class/#fn1)。

有关 NFS 的更多介绍，可以参看[维基百科](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)与[这篇文章](http://cn.linux.vbird.org/linux_server/0330nfs.php)。本文也重度参考了后者。

### NFS 客户端与服务端通讯过程

NFS 的数据传输基于 [RPC 协议](https://zh.wikipedia.org/zh-cn/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)，RPC 为 Remote Procedure Call（远程过程调用）的简写。该协议允许运行于一台计算机的程序调用另一个地址空间（通常为一个开放网络的一台计算机）的子程序，而程序员就像调用本地程序一样，无需额外地为这个交互作用编程（无需关注细节）。RPC 是一种服务器 - 客户端（Client/Server）模式，经典实现是一个通过发送请求、接受回应进行信息交互的系统。

RPC 可以统一管理 NFS 随机的服务端口，并且统一对外的端口为 111。NFS 启动之前需要先启动 RPC，这样 NFS 才能够到 RPC 去注册端口信息。客户端的 RPC 可以通过向服务端的 RPC 请求获取服务端的 NFS 端口信息。当获取到了 NFS 端口信息后，就会以实际端口进行数据的传输。

客户端与服务端的通讯过程如下 [\[2\]](https://bambrow.com/20210908-nfs-storage-class/#fn2)：

1.  首先服务器端启动 RPC 服务，并开启 111 端口
2.  服务器端启动 NFS 服务，并向 RPC 注册端口信息
3.  客户端启动 RPC（portmap）服务，向服务端的 RPC（portmap）服务请求服务端的 NFS 端口
4.  服务端的 RPC（portmap）服务反馈 NFS 端口信息给客户端
5.  客户端通过获取的 NFS 端口来建立和服务端的 NFS 连接并进行数据的传输

### NFS 安装与配置

#### 组件安装

NFS 安装需要两个基础组件，分别是 RPC 主程序 `rpcbind` 与 NFS 主程序 `nfs-utils`。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><span></span><br><span>yum install -y rpcbind</span><br><span>yum install -y nfs-utils</span><br><span></span><br><span>systemctl start rpcbind</span><br><span>systemctl <span>enable</span> rpcbind</span><br><span>systemctl start nfs-server</span><br><span>systemctl <span>enable</span> nfs-server</span><br><span>systemctl start nfs-secure</span><br><span>systemctl <span>enable</span> nfs-secure</span><br><span></span><br><span>firewall-cmd --permanent --add-service=nfs</span><br><span>firewall-cmd --reload</span><br></pre></td></tr></tbody></table>

#### 服务端配置

1.  新建文件夹：`mkdir /public; chmod 777 /public`
2.  修改 `/etc/exports` 并添加 `/public 192.168.37.0/24(rw,sync)`（其中 `192.168.37.0/24` 需要改为用户自己机器集群的 IP 段）
3.  重载 NFS 服务：`systemctl reload nfs`

对于配置文件的详解请看[这里](http://cn.linux.vbird.org/linux_server/0330nfs.php#nfsserver_exports)。简单来讲，最前面的 `/public` 为共享的目录，后面的 IP 为允许访问客户端的 IP 网段，其余参数配置在后面的括号中指定，用逗号隔开。`rw` 表示可读写，`ro` 表示只读；`sync` 表示数据写入内存与硬盘，可以保证不丢失，`async` 则优先保存到内存，再保存到硬盘，可以保证高效率，但可能会丢失数据 [\[3\]](https://bambrow.com/20210908-nfs-storage-class/#fn3)。

#### 客户端挂载配置

1.  使用 `showmount -e 192.168.37.128` 查看 NFS 服务器的共享信息
2.  在客户端创建目录 `mkdir /mnt/public`
3.  修改 `/etc/fstab` 并添加 `192.168.37.128:/public /mnt/public nfs defaults 0 0`
4.  使用 `mount -a` 使配置生效
5.  在服务器上使用 `touch /public/test`，可以看到客户端的 `/mnt/public` 中也出现了此文件，挂载成功

## Kubernetes Storage Class

本小节假设读者已经对 Kubernetes 有基础的了解。在熟悉 [Storage Class](https://kubernetes.io/zh/docs/concepts/storage/storage-classes/) 的概念之前，建议先熟悉 [Volume](https://kubernetes.io/zh/docs/concepts/storage/volumes/) 与 [Persistent Volume](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/) 的概念。

持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者使用存储类（Storage Class）来动态供应。PV 持久卷和普通的 Volume 一样，也是使用 Volume 插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。Storage Class 为管理员提供了描述存储类的方法，可以实现动态创建 PV。每个 StorageClass 都包含 `provisioner`、`parameters` 和 `reclaimPolicy` 字段，这些字段会在 StorageClass 需要动态分配 PersistentVolume 时会使用到。

StorageClass 对象的命名很重要，用户使用这个命名来请求生成一个特定的类。 当创建 StorageClass 对象时，管理员设置 StorageClass 对象的命名和其他参数，一旦创建了对象就不能再对其更新。每个 StorageClass 都有一个制备器（Provisioner），用来决定使用哪个卷插件制备 PV。该字段必须指定。

自动创建的 PV 会以 `${namespace}-${pvcName}-${pvName}` 这样的名字出现在 NFS 服务器的共享目录中，如果它被回收，则会改为 `archived-${namespace}-${pvcName}-${pvName}` 这样的名字 [\[4\]](https://bambrow.com/20210908-nfs-storage-class/#fn4)。

### 通过 Helm 安装

#### 安装 Helm

安装 Helm 的方法十分简单，只需遵循[官网的安装步骤](https://helm.sh/zh/docs/intro/install/)即可。

#### Helm Chart 简介

Helm Chart 可以理解为 Helm 包，它包含在 Kubernetes 集群内部运行应用程序，工具或服务所需的所有资源定义。你可以把它看作是 Homebrew formula，Apt dpkg，或 Yum RPM 在 Kubernetes 中的等价物。Repository（仓库） 是用来存放和共享 Charts 的地方。Release 是运行在 Kubernetes 集群中的 Chart 的实例，一个 Chart 通常可以在同一个集群中安装多次。每一次安装都会创建一个新的 Release[\[5\]](https://bambrow.com/20210908-nfs-storage-class/#fn5)。

Helm Chart 里通常会有以下部分 [\[6\]](https://bambrow.com/20210908-nfs-storage-class/#fn6)：

1.  `Chart.yaml`：Chart 的元数据信息
2.  `values.yaml`：定义 Chart 默认的配置信息
3.  `charts/`：手动管理的 Chart 依赖信息
4.  `templates/`：模板 YAML 文件，定义所有需要用到的 Kubernetes 对象

关于更详细的解释，可以参考[这篇文章](https://docs.bitnami.com/kubernetes/faq/administration/understand-helm-chart/)。

#### 通过 Helm 直接安装 `nfs-subdir-external-provisioner`

此部分参考 [GitHub 项目](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)。

首先新建一个 `namespace`：`kubectl create namespace nfs-provisioner`，随后执行：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/</span><br><span>helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -n nfs-provisioner --<span>set</span> nfs.server=192.168.37.128 --<span>set</span> nfs.path=/public</span><br></pre></td></tr></tbody></table>

安装完毕后通过 `kubectl get all -n nfs-provisioner` 检查是否正常。

#### 通过第三方镜像安装 `nfs-subdir-external-provisioner`

在网络不好的情况下，在添加 Chart Repository 后，需要下载并解压修改 Chart 的配置，使用替换的镜像源。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/</span><br><span>helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner</span><br><span>tar zxvf nfs-subdir-external-provisioner-4.0.13.tgz</span><br></pre></td></tr></tbody></table>

解压完毕，修改文件夹里的 `values.yaml`（下面只列举开头部分）：

values.yaml

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><span><span>replicaCount:</span> <span>1</span></span><br><span><span>strategyType:</span> <span>Recreate</span></span><br><span></span><br><span><span>image:</span></span><br><span>  <span>repository:</span> <span>eipwork/nfs-subdir-external-provisioner</span></span><br><span>  <span>tag:</span> <span>v4.0.2</span></span><br><span>  <span>pullPolicy:</span> <span>IfNotPresent</span></span><br><span><span>imagePullSecrets:</span> []</span><br><span></span><br><span><span>nfs:</span></span><br><span>  <span>server:</span> <span>192.168</span><span>.37</span><span>.128</span></span><br><span>  <span>path:</span> <span>/public</span></span><br><span>  <span>mountOptions:</span></span><br><span>  <span>volumeName:</span> <span>nfs-subdir-external-provisioner-root</span></span><br><span></span><br><span></span><br></pre></td></tr></tbody></table>

随后新建一个 `namespace`：`kubectl create namespace nfs-provisioner`，然后运行 `helm install nfs-subdir-external-provisioner . -n nfs-provisioner` 安装 `nfs-subdir-external-provisioner`。

安装完毕后通过 `kubectl get all -n nfs-provisioner` 检查是否正常。

### 测试 NFS Storage Class

#### 创建 `test-pvc.yaml`

test-pvc.yaml

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><span><span>kind:</span> <span>PersistentVolumeClaim</span></span><br><span><span>apiVersion:</span> <span>v1</span></span><br><span><span>metadata:</span></span><br><span>  <span>name:</span> <span>test-claim</span></span><br><span>  <span>annotations:</span></span><br><span>    <span>volume.beta.kubernetes.io/storage-class:</span> <span>"nfs-client"</span> </span><br><span><span>spec:</span></span><br><span>  <span>accessModes:</span></span><br><span>    <span>-</span> <span>ReadWriteMany</span></span><br><span>  <span>resources:</span></span><br><span>    <span>requests:</span></span><br><span>      <span>storage:</span> <span>1Mi</span></span><br></pre></td></tr></tbody></table>

#### 检查创建的 PV

使用 `kubectl apply -f test-pvc.yaml` 创建 PVC，随后使用 `kubectl get pvc` 查看 PVC 的信息，可以看到状态为 Bound。`kubectl get pv` 也可以看到对应的 PV 状态也为 Bound。

在 NFS 服务器上使用 `ls /public` 查看对应生成的文件夹，名称应该符合 `${namespace}-${pvcName}-${pvName}` 这样的规则。再使用 `kubectl delete -f test-pvc.yaml` 删除创建的 PVC 与 PV，可以看到对应的文件夹已经改为 `archived-${namespace}-${pvcName}-${pvName}` 的名称。

#### 更改默认的 Storage Class

默认的 Storage Class 会在 `kubectl get sc` 的结果中以 `default` 标记。标记默认的方法：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>kubectl patch sc &lt;your-class-name&gt; -p <span>'{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'</span></span><br></pre></td></tr></tbody></table>

取消标记：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>kubectl patch sc &lt;your-class-name&gt; -p <span>'{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'</span></span><br></pre></td></tr></tbody></table>

___

1.  [https://cloud.tencent.com/developer/article/1328694](https://cloud.tencent.com/developer/article/1328694) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref1)
    
2.  [http://zhangguangzhi.top/2017/08/23/14、NFS 服务配置 /](http://zhangguangzhi.top/2017/08/23/14%E3%80%81NFS%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE/) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref2)
    
3.  [https://blog.csdn.net/qq\_38265137/article/details/83146421](https://blog.csdn.net/qq_38265137/article/details/83146421) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref3)
    
4.  [https://www.cnblogs.com/panwenbin-logs/p/12196286.html](https://www.cnblogs.com/panwenbin-logs/p/12196286.html) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref4)
    
5.  [https://helm.sh/zh/docs/intro/using\_helm/](https://helm.sh/zh/docs/intro/using_helm/) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref5)
    
6.  [https://helm.sh/zh/docs/chart\_template\_guide/getting\_started/](https://helm.sh/zh/docs/chart_template_guide/getting_started/) [↩︎](https://bambrow.com/20210908-nfs-storage-class/#fnref6)