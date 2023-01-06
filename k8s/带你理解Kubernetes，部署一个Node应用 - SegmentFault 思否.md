## Kubernetes是什么?

Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能提供一个以“容器为中心的基础架构”。如果你曾经用过Docker容器技术部署容器，那么可以将Docker看成Kubernetes内部使用的低级别组件。Kubernetes不仅仅支持Docker，还支持Rocket，这是另一种容器技术。

通过Kubernetes你可以:

-   自动化容器的部署和复制
-   快速扩展应用
-   将容器组织成组，并且提供容器间的负载均衡
-   无缝对接新的应用功能

想理解Kubernetes集群，需要先搞明白其中的几个重要概念。

### Deployment

Deployment负责创建和更新应用，当创建Deployment后，Kubernetes master 会将Deployment创建好的应用实例调度到集群中的各个节点。  
应用实例创建完成后，Kubernetes Deployment Controller会持续监视这些实例。如果管理实例的节点被关闭或删除，那么 Deployment Controller将会替换它们，实现自我修复能力。

### Pod

创建Deployment时，Kubernetes会创建了一个Pod来托管应用。Pod是Kubernetes中一个抽象化概念，由一个或多个容器组合在一起得共享资源。Pod是独立运行的基本单位，包含一组容器和卷。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。  
当在Kubernetes上创建Deployment时，该Deployment将会创建具有容器的Pods（而不会直接创建容器），每个Pod将被绑定调度到Node节点上，并一直保持在那里直到被终止（根据配置策略）或删除。在节点出现故障的情况下，群集中的其他可用节点上将会调度之前相同的Pod。

### Node

一个Pod总是在一个（Node）节点上运行，Node是Kubernetes中的工作节点，可以是虚拟机或物理机。每个Node由 Master管理，Node上可以有多个pod，Kubernetes Master会自动处理群集中Node的pod调度，同时Master的自动调度会考虑每个Node上的可用资源。

### Replication Controller

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它。如果在运行中将副本总数改为5，Replication Controller会立刻启动2个新Pod，保证总数为5。

### Service

事实上，Pod是有生命周期的。当一个工作节点(Node)销毁时，节点上运行的Pod也会销毁，然后通过ReplicationController动态创建新的Pods来保持应用的运行。  
举个例子，考虑一个图片处理 backend，它运行了3个副本，这些副本是可互换的 —— 前端不需要关心它们调用了哪个 backend 副本。也就是说，Kubernetes集群中的每个Pod都有一个独立的IP地址，因此需要有一种方式来自动协调各个Pod之间的变化，以便应用能够持续运行。  
Kubernetes中的Service 是一个抽象的概念，它定义了Pod的逻辑分组和一种可以访问它们的策略，让你的这组Pods能被Service访问。借助Service，可以方便的实现服务发现与负载均衡。  
Service可以被指定四种类型：

-   ClusterIP - 在集群中内部IP上暴露服务。此类型使Service只能从群集中访问。
-   NodePort - 通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求 <NodeIP>:<NodePort>，可以从集群的外部访问一个 NodePort 服务。
-   LoadBalancer - 使用云提供商的负载均衡器（如果支持），可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。(一般常用此类型向外暴露端口，并做负载均衡)
-   ExternalName - 通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容，没有任何类型代理被创建。

### Label

你可以赋予标签（键值对）来标识你的Pod、Deployment、Service。之后就可以通过选择标签来做具体的指令。

## 使用Minikube在kubernetes中部署第一个应用

本文以 MAC OS X 为例进行。目标是将简单的Hello World Node.js应用部署在Kubernetes上运行。因此你可能需要Node.js环境。  
Minikube 是一个使我们很容易在本地运行 kubernetes 的工具，由Kubernetes社区开发。

### 准备工作

1.  Node.js
2.  Docker
3.  VM - VirtualBox
4.  Minikube
5.  Kubectl

对于Node.js、Docker、VirtualBox的安装，在这里不做详细介绍。可以直接到官网下载安装最新稳定版本。

### 创建Minikube集群

使用curl下载并安装最新版本Minikube：

```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && \
  chmod +x minikube && \
  sudo mv minikube /usr/local/bin/
```

使用Homebrew下载kubectl命令管理工具：

```
$ brew install kubectl
```

默认的VM驱动程序VirtualBox，因此可直接启动Minikube：

```
$ minikube start
```

接下来会打印出类似信息...

```
Starting local Kubernetes v1.9.4 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

验证kubectl是否安装成功：

```
$ kubectl cluster-info
```

会打印出类似的信息：

```
Kubernetes master is running at https://192.168.99.100:8443
```

### 创建Node.js应用

编写应用程序。将这段代码保存在一个名为hellonode的文件夹中，文件名server.js:

```
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(3000);
```

启动应用：

```
$ node server.js
```

现在可以在 [http://localhost](https://link.segmentfault.com/?enc=3oVkdv8HzF3j6kmH61ZAog%3D%3D.mUtN3buPAyAZcsXfHqXaxDwjHDpDkeeO3TgT48fcbx8%3D):3000 中查看到“Hello World！”消息。  
Ctrl-C停止正在运行的Node.js服务器。

### 创建Docker容器镜像

在hellonode文件夹中创建一个Dockerfile命名的文件。

```
FROM node:8.10.0
EXPOSE 3000
COPY server.js .
CMD node server.js
```

我们使用Minikube，而不是将Docker镜像push到registry，可以使用与Minikube VM相同的Docker主机构建镜像，以使镜像自动存在。为此，请确保使用Minikube Docker守护进程：

```
$ eval $(minikube docker-env)
```

注意：如果不在使用Minikube主机时，可以通过运行`eval $(minikube docker-env -u)`来撤消此更改。

使用Minikube Docker守护进程build Docker镜像：

```
$ docker build -t hello-node:v1 .
```

### 创建Deployment

Kubernetes Deployment 是检查Pod的健康状况，如果它终止，则重新启动一个Pod的容器，Deployment管理Pod的创建和扩展。

使用kubectl run命令创建Deployment来管理Pod。

```
$ kubectl run hello-node --image=hello-node:v1 --port=3000
```

查看Deployment：

```
$ kubectl get deployments
```

输出：

```
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           5s
```

查看Pod：

```
$ kubectl get pods
```

输出：

```
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-6ddb5576c9-644xn   1/1       Running   0          1m
```

查看deployment的详细信息：

```
$ kubectl describe deployment
```

### 创建Service

Pod只能通过Kubernetes群集内部IP访问。要使hello-node容器能从Kubernetes虚拟网络外部访问，须要使用Kubernetes Service暴露Pod。

我们可以使用kubectl expose命令将Pod暴露到外部环境：

```
$ kubectl expose deployment hello-node --type=LoadBalancer
```

查看Service：

```
$ kubectl get services
```

输出：

```
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.110.94.17   <pending>     8080:32081/TCP   2m
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP          6d
```

查看详细信息：

```
$ kubectl describe service hello-node
```

输出：

```
Name:                     hello-node
Namespace:                default
Labels:                   run=hello-node
Annotations:              <none>
Selector:                 run=hello-node
Type:                     LoadBalancer
IP:                       10.110.94.17
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32081/TCP
Endpoints:                172.17.0.5:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

通过--type=LoadBalancer 来在群集外暴露Service，在支持负载均衡的云提供商上，将配置外部IP(`EXTERNAL-IP`，在Minikube显示为：`<pending>`)地址来访问Service。在Minikube上，该LoadBalancer type使服务可以通过minikube Service 命令访问。

```
$ minikube service hello-node
```

将打开浏览器，在本地IP地址为应用提供服务，显示“Hello World”的消息。

可以查看到日志：

```
$ kubectl logs <pod-name>  //exp: kubectl logs hello-node-6ddb5576c9-644xn
// 可通过 kubectl get pods 查看pod-name
```

### 扩展实例

根据线上需求，扩容和缩容是常会遇到的问题。Scaling 是通过更改 Deployment 中的副本数量实现的。一旦有多个实例，就可以滚动更新，而不会停止服务。通过kubectl scale指令来扩容和缩容。

```
$ kubectl scale deployments/hello-node --replicas=4
```

在查看pods：

```
$ kubectl get pods
```

输出：

```
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-9f5f775d6-6qdmn   1/1       Running   0          3s
hello-node-9f5f775d6-9mrm6   1/1       Running   0          3s
hello-node-9f5f775d6-jxb8z   1/1       Running   0          3s
hello-node-9f5f775d6-tx8kg   1/1       Running   0          11m
```

总共有4个实例，那么就可通过Service 的 `--type=LoadBalancer` 进行负载均衡。

### 更新应用程序

编辑server.js文件以返回新消息：

```
response.end('Hello World Again!');
```

docker build新版本镜像：

```
$ docker build -t hello-node:v2 .
```

Deployment更新镜像：

```
$ kubectl set image deployment/hello-node hello-node=hello-node:v2
```

再次在浏览器查看消息：

```
$ minikube service hello-node
```

### 清理删除

删除在群集中创建的资源：

```
$ kubectl delete service hello-node
$ kubectl delete deployment hello-node
```

查看pods:

```
$ kubectl get pods
```

输出：

```
NAME                         READY     STATUS        RESTARTS   AGE
hello-node-9f5f775d6-6qdmn   1/1       Terminating   0          6m
hello-node-9f5f775d6-9mrm6   1/1       Terminating   0          6m
hello-node-9f5f775d6-jxb8z   1/1       Terminating   0          6m
hello-node-9f5f775d6-tx8kg   1/1       Terminating   0          18m
```

全部在停止中... 稍等一分钟，再查看，输出 `No resources found.`

### 停止

```
$ minikube stop
```

输出：

```
Stopping local Kubernetes cluster...
Machine stopped.
```

完毕