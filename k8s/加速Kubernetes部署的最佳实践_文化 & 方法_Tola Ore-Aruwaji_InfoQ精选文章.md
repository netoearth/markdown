-   刘雅梦

-   凌敏

-   2021 年 8 月 30 日
-   本文字数：3611 字
    
    阅读完需：约 12 分钟
    

![加速Kubernetes部署的最佳实践](https://static001.infoq.cn/resource/image/84/24/84566ecab76f6c51567b68616f34da24.jpeg)

> 在本文中，我们将介绍扩展 Pod、副本控制器（Replication Controller）以及加速 Kubernetes 部署（Deployment）的最佳实践。

Kubernetes Deployment 是对 Pod 所期望状态的描述。你可以使用 Deployment 来发布新的应用程序或微服务，或者更新现有的应用程序或微服务。Deployment 可以扩展 Pod 的副本数，可以以可控的方式来发布更新后的代码，或者在必要时回滚到早期的部署版本。

如果你的应用程序是无状态的（Stateless），则可以进行水平扩展（Horizontally Scale）。无状态应用程序意味着你的应用程序没有状态，它没有任何本地文件写入并不保留任何本地会话。

如果你有两个 Pod，其中一个 Pod 需在本地写东西，那么这两个 Pod 就会不同步。这就意味着每个 Pod 都有自己的状态，并且你不能杀死它。如果两个 Pod 始终具有相同的文件，那么它们实际上不会有自己的状态，并且向其中一个 Pod 发出的请求结果始终与向另一个 Pod 发出的请求结果相同，那么该 Pod 可能就是无状态的。

-   **无状态的（Stateless）**：应用程序没有状态。它没有任何本地文件写入并不保留任何本地会话。
    
-   所有的传统数据库（MYSQL、 PostgreSQL）都是**有状态的（stateful）**。它们具有不能在多个实例上进行拆分的数据库文件。
    

大多数的 Web 应用程序都可以被设置成无状态的：

-   会话管理需要在容器外完成。因此，如果你想从 Web 应用程序上获取点击量，并且想要保留访问者的信息，那么你需要使用外部服 务，你不能将这些数据存储在容器中。 你可以使用 Memcache、Redis 甚至数据库来存储会话。
    
-   任何需要保存的文件都不能在容器上进行本地保存，因为如果你关停并重启容器，文件将会丢失。 因此，任何需要保存的文件都需存储在容器外的某些共享存储或外部服务（AWS）上。在 AWS 上，你可以将其保存在 S3 对象存储上。
    
-   那些有状态的应用程序无法进行水平扩展，但你可以在单个容器中运行它们，并进行**垂直扩展（Vertically Scale）**，分配更多 CPU/内存/磁盘。
    

Kubernetes 中的扩展可以使用 Replication Controller 来完成。

Replication Controller（副本控制器）将确保始终运行指定数量的 Pod 副本。例如，你可以指定 5 个 Pod 副本，它将会运行同一 Pod，但会运行 5 次。

使用 Replication Controller 创建的 Pod 如果出现失败、被删除或被终止，则该 Pod 将会**被自动替换**。

例如，如果你告诉 Kubernetes 运行五（5）个 Pod，但由于某个节点崩溃了，只有 4 个 Pod 能正常运行，那么 Kubernetes 将会在另外的一个节点上再另外启动一个该 Pod 的实例。

如果你只是想要确**保始终有 1 个 Pod 能运行**，即使是在重启之后，那么建议你也使用 Replication Controller。然后，你可以运行只有 **1 个副本**的 Replication Controller。这样可以确保该 Pod 始终会运行，即使是在节点崩溃时。

## 举个例子

在下面的例子中，我们会将应用复制 2 次：

Replication Controller 也有一个规范（spec）。在这个 spec 中，我们有 2 个副本，这是应用程序 hello world 的 Replication Controller，这就是为什么我们将选择器（selector）设置为 `app:helloworld` ，它将选择带有`app:helloworld`的标签。

然后我们就有一个 Pod 的定义模板。Pod 定义也有 `metadata` ，也有标签，与你在选择器下看到的相同。

最后，我们有了 `pod specification` 。还有一个名为 `docker-demo` 容器及镜像，并且暴露了端口 `5000` 。

```
apiVersion: v1
```

复制代码

因此，Replication-Controller 要做的就是运行两次模板中定义的 Pod。

-   完成配置文件后，可以使用 `kubectl create -f replicationcontroller.yaml` 进行创建
    
-   可以使用 `kubectl get replicationcontroller` 来确认检查你的副本控制器是否正在运行
    
-   通过这个，你可以看到我们是如何水平扩展 Pod 的。我们有多个 Pod 正在运行，你还可以在它前面放置一个服务，比如负载平衡器，使其他软件或客户可以访问你的多个 Pod。如果你配置的某个 Pod 崩溃，控制器将会自动重新编排这些 Pod。
    
-   如果要移除了其中的某个 Pod，你会看到副本控制器会再创建一个新的 Pod。
    
-   你可以看到被终止 Pod，以及正在创建的那个新 Pod。
    

![](https://static001.infoq.cn/resource/image/9f/b7/9f4353b8967431fc592f8b8f40aaf2b7.png)

-   接下来你会看到 Pod 被终止后又创建了一个新的 Pod。从下图可以看出，在我们删除最后一个 Pod 后的 43 秒后，一个新的 Pod 被创建。
    

![](https://static001.infoq.cn/resource/image/5f/30/5f63bayy72cf5977a358c0b98ac8e330.png)

-   你可以随时对这些 Pod 进行扩展，可以通过 `kubectl scale --replicas=4 -f (filename.yaml)` 命令来实现，并且最终你会看到控制器进行了相应扩展。
    
-   回到你的终端，检查一下，你会发现由于进行了扩容操作，已经创建了 4 个 Pod。
    
-   另一种进行扩展的方式是使用 `kubectl get rc` 来获取副本控制器的名称，然后使用 `kubectl scale--replications=1 rc/helloworld controller` 来对其进行扩展，运行该命令再次进行扩展。
    
-   再次检查你的 Pod，你会看到一些 Pod 已经终止，现在只有 1 个 Pod 正在运行。
    
-   正如我们前面讨论的那样，你可以看到，你只能在 Pod 是无状态的情况下进行扩展。如果 Pod 是有状态的，那么你将无法执行这些操作。
    
-   你可以使用 `kubectl delete rc/helloworld-controller` 来删除你的副本制控制器，最后一个 Pod 将会被终止。
    
-   这些扩展操作都以后端 `etcd` 的形式被保存在 Kubernetes 中，它保存了所有这些设置，如副本的数量。你无需总是将这些内容写入到 `yaml` 文件中。
    

## Kubernetes Deployment 与 Replication Set

-   **Replica Set**（副本集）是下一代 Replication Controller（副本控制器）。它支持了一个新的选择器，该选择器可以根据一组值来进行筛选。例如，环境可以是“dev”或“qa”，利用副本集，进行更复杂的选择匹配。
    
-   Deployment 对象使用的是 Replica Set，而不是 Replication Controller。
    

让我们看一下 Kubernetes 中的 **Deployment：**

-   Deployment 是 Kubernetes 中的一个声明，通过它你可以对应用程序进行部署和更新。使用 Deployment 对象时，定义应用程序的**状态（state）**。然后，Kubernetes 将确保集群与你所期待的状态相符。
    
-   仅使用 Replication Controller 或 Replication Set 来部署应用程序可能会很麻烦。更新和回滚可能需要太多的手工工作。
    
-   Deployment 对象更易于使用，并能为你提供更多可能性。
    

那么这些可能性是什么呢?

-   创建部署（例如部署应用程序）
    
-   更新部署（例如部署新版本）
    
-   执行滚动更新（零停机部署）
    
-   回滚到以前的版本。
    
-   暂停/恢复部署（例如，仅部署到指定百分比）
    

```
apiVersion: extensions/v1beta1
```

复制代码

对于上面的 Deployment，我们将其命名为 `helloworld-deployment` ，其中包含 3 个副本，容器规范与我们之前在副本集中使用的规范类似。

![](https://static001.infoq.cn/resource/image/86/28/867640241b9a992eeea6e13a3a3c8e28.png)

所有副本都是最新的，并且运行的是我们要求的最新版 `docker-demo` 。

-   你还可以使用 `kubectl get rs` 获取 `replica set`  
    

![](https://static001.infoq.cn/resource/image/5e/0a/5e739472f0dded29939ab1cb9009130a.png)

你无需自己创建副本集，Kubernetes 会自动为你创建。

-   检查你的 Pod，并获取副本。
    

![](https://static001.infoq.cn/resource/image/63/83/6356b901a140a2c1de7c1ed3567d3383.png)

-   显示 Pod 中的标签。
    

![](https://static001.infoq.cn/resource/image/c7/cf/c7180cf123bcb93d4edd4489fb51e3cf.png)

-   使用 `kubectl expose deployment hello-world-deployment — type=NordPort` 发布你的部署
    

![](https://static001.infoq.cn/resource/image/02/69/029ba4f52a6744058bf1a36a89434e69.png)

-   使用 `kubectl get service` 获取服务，你将看到该新创建部署的服务。
    

![](https://static001.infoq.cn/resource/image/77/f2/77e830823553c363a4539b7ae1ca4bf2.png)

-   使用 `kubectl set image hello-world-deployment docker-demo=tolatemitope/docker-demo:2`设置新镜像
    
-   使用 `kubectl rollout status hello-world-deployment` 获取 rollout 状态。
    

## 适用于 Kubernetes Deployment 的命令

-   `kubectl get deployments` -> 获取当前部署的信息
    
-   `kubectl get rs` -> 获取副本集信息。
    
-   `kubectl get pods --show-labels` -> 获取 pods，并显示附加在 pods 上的标签。
    
-   `kubectl rollout status deployment/(name of deployment)` ->获取部署状态
    
-   `kubectl set image deployment/helloworld-deployment docker-demo=docker-demo:2` -> 使用版本号为 2 的新镜像运行 docker-demo
    
-   `kubectl edit deployment/helloworld-deployment` -> 编辑部署对象
    
-   `kubectl rollout status deployment/helloworld-deployment` ->获取 rollout 状态
    

如果你需将镜像更新到新版本，那么可以使用 rollout 状态来查看部署的进度。

-   `kubectl rollout history deployment/helloworld-deployment` -> 获取 rollout 历史记录
    

如果你发布了应用程序的新版本，例如版本 3，并且你希望恢复到版本 2，则可以使用：

-   `kubectl rollout undo deployment/helloworld-deployment` -> 回滚到前一个版本。
    

回滚到特定版本可以使用：

-   `kubectl rollout undo deployment/helloworld-deployment --to-revision=n` -> 回滚到某个更具体的版本。
    

**原文链接：**

[https://levelup.gitconnected.com/best-practices-for-accelerating-kubernetes-deployments-ece46ccd174c](https://levelup.gitconnected.com/best-practices-for-accelerating-kubernetes-deployments-ece46ccd174c)  

2021 年 8 月 30 日 14:283240

文章版权归极客邦科技InfoQ所有，未经许可不得转载。