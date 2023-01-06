kind 在创建集群的时候实际上使用的是 kubeadm 所以还可以修改 kubeadm 的配置来修改默认的镜像地址，节点标签污点等信息，除此之外还可以配置 PV/PVC CNI 插件等配置.

[![](https://s4.51cto.com/oss/202105/07/aec84dd7bed5340b16aa30b468f3c4c7.png)](https://s4.51cto.com/oss/202105/07/aec84dd7bed5340b16aa30b468f3c4c7.png)

上篇文章中我们讲到了 k8s 中有哪些扩展点，开始 Operator 开发相关之前我们需要先把本地环境搭建好

## 一、依赖检查

开始之前需要先检查一下

1.是否已经安装好了 Docker 环境

-   如果还没安装 Mac/Windows 的用户可以直接安装 Docker Desktop
-   Windows 用户推荐使用 WSL2

2.是否已经安装好了 Golang 环境, Go 版本最好 >= 1.15

-   如果没有可以参考官方文档进行安装

## 二、使用 Kind 搭建本地开发环境

### 2.1 安装

如果本地存在 Golang 环境可以直接执行下方命令进行安装

```
GO111MODULE="on" go get sigs.k8s.io/kind@v0.10.0 && kind create cluster 
```

如果不想通过源码安装可以查看官方文档直接安装编译好的二进制文件

执行下方命令可以输出 kind 的版本就表示安装好了

```
❯ kind version 
kind v0.10.0 go1.16 linux/amd64 
```

### 2.2 创建一个 K8s 集群

使用下方命令即可创建一个简单的单节点 K8s 集群

集群的创建时间和你的网络环境以及机器的性能有关系，kind 会去 docker hub 上拉取 kindest/node 镜像，这个镜像大概有 400M 的大小，当出现下方的提示的时候就说明集群创建好了

```
❯ kind create cluster 
Creating cluster "kind" ... 
 ✓ Ensuring node image (kindest/node:v1.20.2) 🖼 
 ✓ Preparing nodes 📦 
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind" 
You can now use your cluster with: 
 
kubectl cluster-info --context kind-kind 
 
Thanks for using kind! 😊 
```

我们可以使用 kind get clusters 获取我们创建的集群列表，kind 支持创建多个集群，不手动指定集群名称的时候默认名为 kind ，我们也可以像下面一样在创建集群的时候使用 --name 指定集群名称

```
kind create cluster --name mohuishou 
```

可以看到我们两个集群都创建好了

```
❯ kind get clusters 
kind 
mohuishou 
```

现在我们只需要一个集群，所以可以先使用 kind delete clusters mohuishou 将刚刚创建的集群先删除掉

### 2.3 使用集群

下面我们来看一下 kubectl 的常用命令是否都可以正常使用，并且部署一个简单的 web 服务试试

### 查看集群信息

```
❯ kubectl  cluster-info --context kind-kind 
Kubernetes master is running at https://127.0.0.1:41801 
KubeDNS is running at https://127.0.0.1:41801/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy 
 
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'. 
```

可以看到我们 k8s master 地址和 dns 的地址，注意一般情况下我们只创建一个集群不需要指定 --context cluster name 但是创建了多个集群时这个基本就是必须的一个命令了。如果不加这个参数可能会报下面的错误

```
The connection to the server localhost:8080 was refused - did you specify the right host or port? 
```

这是因为 kubectl默认连接的 apiserver 地址是 localhost:8080但是我们的 apiserver 地址不是这个所以报错。

**为什么我们使用 --context cluster-name 就可以了呢?**

这是因为 kind 在创建集群的时候会修改 $HOME/.kube/config 的配置，将集群的 apiserver 地址，证书等相关信息都自动写入进去了

**每次命令都要加上这个参数好麻烦怎么办?**

我们可以使用 kubectl config use-context context-name 来进行设置，设置好了之后我们后续就不用每次都加上 –context 的参数了，同时还可以通过 kubectl config current-context 查询我们当前默认操作的集群是哪一个

### 查看集群节点列表

可以发现我们部署的是一个单节点的 v1.20.2 的集群

```
❯ kubectl get no 
NAME                 STATUS   ROLES                  AGE   VERSION 
kind-control-plane   Ready    control-plane,master   20m   v1.20.2 
```

### 部署一个 Nginx 服务

使用下方代码创建一个 nginx.yml 文件，然后使用 kubectl apply -f nginx.yml 就可以完成部署了

```
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: nginx-deployment 
  labels: 
    app: nginx 
spec: 
  replicas: 3 
  selector: 
    matchLabels: 
      app: nginx 
  template: 
    metadata: 
      labels: 
        app: nginx 
    spec: 
      containers: 
        - name: nginx 
          image: nginx:1.14.2 
          ports: 
            - containerPort: 80 
```

查看当前 deployment 的状态

```
❯ kubectl get deployment 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE 
nginx-deployment   0/3     3            0           39s 
```

查看 pod 的状态

```
❯ kubectl get pods          
NAME                                READY   STATUS    RESTARTS   AGE 
nginx-deployment-66b6c48dd5-2s5cb   1/1     Running   0          84s 
nginx-deployment-66b6c48dd5-8wf8b   1/1     Running   0          84s 
nginx-deployment-66b6c48dd5-zc6vd   1/1     Running   0          84s 
```

由于我们没有做服务暴露，所以是不能直接访问对应的服务的，我们可以用 kubectl提供的端口转发功能来讲流量从本地转发给 k8s 集群

```
❯ kubectl port-forward nginx-deployment-66b6c48dd5-2s5cb 30080:80 
Forwarding from 127.0.0.1:30080 -> 80 
Forwarding from [::1]:30080 -> 80 
Handling connection for 30080 
```

我们访问 http://localhost:30080 就会发现这个熟悉的 nginx 界面

[![](https://s4.51cto.com/oss/202105/07/168a84ec700c4a6ee7539fa77a0b9cbe.png)](https://s4.51cto.com/oss/202105/07/168a84ec700c4a6ee7539fa77a0b9cbe.png) 

image-20210424210804343

到这里可以发现我们集群已经是可以使用了

### 2.4 进阶使用

2.4.1 创建一个多节点的集群

kind 默认创建的是一个单节点的集群，我们可以通过修改配置创建一个高可用的集群，我们创建一个 3 个 master 节点，两个 worker 节点的集群

```
kind create cluster --name mohuishou-ha --config kind-ha.yml 
```

```
kind: Cluster 
apiVersion: kind.x-k8s.io/v1alpha4 
nodes: 
  - role: control-plane 
  - role: control-plane 
  - role: control-plane 
  - role: worker 
  - role: worker 
```

 看一下节点即可验证

```
❯ kubectl --context kind-mohuishou-ha get no                  
NAME                          STATUS   ROLES                  AGE     VERSION 
mohuishou-ha-control-plane    Ready    control-plane,master   16m     v1.20.2 
mohuishou-ha-control-plane2   Ready    control-plane,master   14m     v1.20.2 
mohuishou-ha-control-plane3   Ready    control-plane,master   13m     v1.20.2 
mohuishou-ha-worker           Ready    <none>                 5m52s   v1.20.2 
mohuishou-ha-worker2          Ready    <none>                 5m52s   v1.20.2 
```

2.4.2 Load Image

一般来说我们在 k8s 内部署应用需要先把容器镜像推送到到镜像仓库当中，这样在本地开发的时候相对来说会比较麻烦，特别是镜像比较大的时候，往返会有两次网络消耗，为了解决这个问题我们可以使用 kind load 镜像的功能直接把镜像。

```
kind load docker-image my-custom-image-0 my-custom-image-1 --name kind 
```

排坑指南：load image 成功，但是部署 pod 报错

```
Failed to pull image "controller:latest": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/controller:latest": failed to resolve reference "docker.io/library/controller:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed 
```

这个问题之前卡了我很久，还是有点坑

1.进入节点查看: 镜像是否存在

-   获取节点名

```
▶ kubectl get nodes 
NAME                 STATUS   ROLES                  AGE    VERSION 
kind-control-plane   Ready    control-plane,master   2d1h   v1.20.2 
```

-   进入终端

```
docker exec -it kind-control-plane bash 
```

-   查看镜像是否存在，kind 创建的集群使用的是 containerd 所以我们使用 crictl 命令来获取

```
crictl img | grep controller 
docker.io/library/controller   latest    421cbf77618ba       72.1MB 
```

2.如果镜像存在，也就是我们上面看到的情况，这时候就要检查一下我们部署的 yaml 文件

```
kubectl -n node-pool-operator-system  get deployment -o yaml | grep imagePullPolicy 
```

是否存在: imagePullPolicy: Always 如果我们没有改 yaml 的话默认会是这个配置，这个配置会导致每次都去镜像仓库拉取镜像，改成 imagePullPolicy: IfNotPresent 就可以了

3.如果镜像不存在：这时候要检查一下有没有指定 cluster-name，在存在多个集群的情况下可能没有加载到我们想要的集群，加上 --name cluster-name 即可

### 总结

kind 在创建集群的时候实际上使用的是 kubeadm 所以还可以修改 kubeadm 的配置来修改默认的镜像地址，节点标签污点等信息，除此之外还可以配置 PV/PVC CNI 插件等配置，如果有需求的话可以查阅 kind 的官方文档

不过要注意的是 kind 不支持给运行的集群添加节点，如果需要多节点集群的话得提前规划好节点数量

### 参考文献

-   kind – Configurationhttps://kind.sigs.k8s.io/docs/user/configuration/
-   Docker Desktophttps://www.docker.com/products/docker-desktop
-   WSL2https://docs.microsoft.com/en-us/windows/wsl/install-win10
-   Getting Started - go.dev https://learn.go.dev/
-   使用 Kind 搭建你的本地 Kubernetes 集群https://segmentfault.com/a/1190000018720465

[![](https://s6.51cto.com/oss/202105/07/bc78ac401c4b3e471a3dc3d7fccfcf99.png)](https://s6.51cto.com/oss/202105/07/bc78ac401c4b3e471a3dc3d7fccfcf99.png)