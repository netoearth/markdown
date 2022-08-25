在容器化的早期阶段，它们被设定为一种运行无状态应用的机制。

在过去的几年间，社区意识到[在容器中运行有状态工作负载](https://www.infoq.com/articles/distributed-systems-kubernetes/)的价值，而且像 Kubernetes 这样的编排器引入了必要的特性。

Kubernetes 提供了持久化卷（Persistent Volume，PV）架构以及像 StatefulSet 和 DaemonSet 这样的控制器，它们能够让我们创建[有状态工作负载的Pod](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/)，即便是在 Kubernetes 扩展和供应资源的时候，这些工作负载也能保持运行，并且能够确保现有的客户端连接不会中断。

这种方式虽然远远谈不当简单直接，但是能够行之有效，任何采用 Kubernetes 作为运行时基础设施的人都必须熟悉它。

在本文中，我将会阐述在 Kubernetes 中运行有状态应用的重要性，给出运行有状态应用的三个可选方案，并详细描述它们的运行机制。

## 什么是有状态应用？

有状态应用允许用户重复返回该应用并恢复之前的操作，比如电子邮件或者网上银行应用。有状态的应用会记录之前事务的上下文，这些上下文可能会对当前或未来事务产生影响。所以，有状态的应用必须确保每个用户始终访问同一个应用程序实例，或者有某种在实例之间同步数据的机制。

有状态进程的优点是，应用程序可以存储每个事务的历史和上下文，跟踪最近的活动、配置偏好和窗口位置等元素，并允许用户恢复事务。有状态的事务的表现就像始终和同一台服务器进行对话一样。

如今，大多数的应用都是有状态的。容器和微服务等技术的进步推动了基于云的应用开发，然而由于它们的动态性，使得有状态进程的管理更具挑战性。

## 容器化有状态应用的使用场景

在容器上运行有状态应用的需求正变得越来越大。容器化的应用可以简化复杂环境中的部署和运维，如边缘云计算和混合云环境。状态性对于持续集成和持续交付（CI/CD）也很重要，因为 CI/CD 流水线必须保持状态，以确保从开发到生产环境部署过程的连贯性。

容器化有状态应用的常见使用场景包括：

-   **机器学习运维（MLOps）**：在 MLOps 环境中，容器需要是有状态的，这样做有多个目的，包括共享推理和训练的结果以及训练 job 的检查点。
    
-   **AI 和数据分析处理**：数据处理和机器学习框架，如 Apache Spark、Hadoop、Kubeflow、Tensorflow 和 PyTorch，对容器化的支持在不断增强。这些平台必须反复处理大量的数据，需要有保持状态的机制。
    
-   **消息系统和数据库**：你可能更喜欢使用本地闪存来获取低延迟性，但是这会使得容器很难在不同的 worker 节点间进行移动，因为数据会持久化到节点上。高性能共享存储对各种应用都很重要，比如单实例数据库（如 MySQL）、内存数据库（如 Redis）、NoSQL 数据库（如 MongoDB）、业务关键型的应用（如 SAP 或 Oracle）以及消息应用（如 Kafka）。
    

## 在 Kubernetes 中实现有状态部署的三个可选方案

在 Kubernetes 集群中运行有状态的工作负载主要有三个可选方案，即在集群之外运行、作为集群旁的云服务或者在 Kubernetes 集群中运行。

### 1.在 Kubernetes 之外运行有状态的应用

一种常见的方式就是在 VM 或裸机中运行有状态的应用，并让 Kubernetes 中的资源与之进行通信。从集群中 pod 的角度来看，有状态应用会作为一个外部的集成。

这种方式的**好处**在于，它允许我们按照原样运行现有的有状态应用，无需重构或重新架构。如果应用能够根据 Kubernetes 集群中工作负载的需要进行扩展，那么我们就不需要 Kubernetes 复杂的自动扩展和资源供应机制。

这种方式的**缺点**在于，在集群外维护非 Kubernetes 的资源，这就需要我们有某种方式来监控进程、执行配置管理，并为应用执行负载均衡和服务发现。我们在 Kubernetes 之外搭建了一个并行的软件工作流，所以基本是在进行重复的工作。

### 2\. 以云服务的形式运行有状态的工作负载

第二种同样常见的方法是将有状态的应用作为托管云服务来运行。例如，如果你需要在一个容器化的应用中运行一个 SQL 数据库，并且应用在 AWS 上运行，那么你可以使用 Amazon 的 Relational Database Service（RDS）。托管数据库往往是可以进行弹性扩展的，所以随着 Kubernetes 资源的扩展，有状态的服务也可以适应不断增加的需求。

这种方式的**好处**在于，它的搭建过程非常容易，有状态工作负载的持续维护应该会非常简单，而且你使用的是一个与 Kubernetes 兼容的云原生资源。

这种方式的**缺点**在于，托管云服务是有成本的，它的定制能力通常会比较有限，并且不一定能提供你所需要的性能或延迟属性。同时，采取这种方式，会让你锁定到特定云供应商上。

### 3\. 在 Kubernetes 中运行有状态的工作负载

这种方式最难实现，但是从长远来看，它会带给我们最大的灵活性和运维效率。我们可以使用 Kubernetes 提供的两个原生控制器来运行有状态的应用，即 StatefulSet 和 DaemonSet。

#### StatefulSet 控制器

[StatefulSet](https://spot.io/resources/kubernetes-architecture/kubernetes-statefulsets-scaling-managing-persistent-apps/)是一个 Kubernetes 的控制器，它管理具有唯一身份标识的多个 pod，并且它们是不能互相交换的（这与常规的 Kubernetes Deployment 有所差异，在 Deployment 中，pod 是无状态的，可以根据需要经常销毁和重建）。

在 StatefulSet 中，每个 pod 都有一个持久化的、唯一的 ID。每个 pod 可以有自己的持久化存储卷。如果 Kubernetes 需要扩展和伸缩的话，它会保持与外部用户或者集群中其他应用的现有连接。

#### DaemonSet 控制器

[DaemonSet](https://www.bmc.com/blogs/kubernetes-daemonset/)是一个 pod，Kubernetes 能够确保它会在集群的所有节点，或者通过选择器定义的特定节点子集上运行。每当符合条件的节点被添加到集群中，这个 pod 都会在它上面启动。

对于需要以后台进程的形式运行的有状态应用来说，DaemonSet 非常有用，比如监控或日志聚合应用。一般来讲，DaemonSets 的灵活性较差，但是比 StatefulSet 更易于管理，资源的使用也更加可预测。

## Kubernetes 中的持久化存储

卷（volume）是一个 Kubernetes 实体，它提供了持久化的存储。Pod 中所有的容器可以共享卷。我们可以借助持久化卷，让运行在同一个 pod 中的多个服务使用同一个挂载的文件系统。

### 非持久化存储卷

在 Kubernetes 中，要授予容器对持久化存储的访问权，我们需要声明所需的卷以及所需的位置，以便于在容器的文件系统中挂载该卷。

Kubernetes 中的常规存储卷会有一个确定的生命周期：每个卷都与 pod 的生命周期绑定。当 pod 处于活跃状态的时候，卷会保持在 pod 内，如果重启 pod 的话，卷会被重置。这个模型不适合有状态的工作负载，这也是 Kubernetes 引入持久化卷（Persistent Volumes）概念的原因。

### PersistentVolumes (PV)

[Kubernetes PersistentVolumes（PV）](https://cloud.netapp.com/blog/kubernetes-persistent-storage-why-where-and-how)是存在于集群级别的存储对象。将 PV 绑定到集群上会扩展它们的生命周期，不再局限于 pod 的生命周期。因为 PV 位于集群级别，所以 pod 可以共享数据。我们可以扩展持久化卷的大小和规模，但是不能减少它的大小。

我们有两种方式来提供 PV：

-   **静态方式（Statically）** ：能够让我们预先分配存储资源。这样会假定集群可用的物理存储资源是静态的。
    
-   **动态方式（Dynamically）**：能够让我们扩展可用的存储空间，以满足不断增长的需求。我们可以通过使用 Kubernetes API 服务器启用 DefaultStorageClass admission 控制器来使用该方案。
    

### PersistentVolumeClaim（PVC）

PVC 能够让 Kubernetes 用户请求存储。它的运行方式与 pod 类似，只不过 pod 消费节点资源，而 PVC 消费 PV 资源。除此之外，与 pod 能够请求特定级别的资源一样，PVC 也可以请求特定的访问模式和大小。

PV 和 PVC 的主要差异在于：

|  | PV | PVC |
| --- | --- | --- |
| 谁来创建它们 | 只有集群管理员和Kubernetes（通过动态供应）能够创建PV。 | 开发人员和用户都能创建PVC。 |
| 资源类型 | PV是一种集群资源。 | PVC是对存储资源的请求。 |
| 消费 | PVC消费PV资源。 | Pod消费PVC。 |

## StatefulSets 和 DaemonSets

### StatefulSets

StatefulSet 是一个工作负载 API 对象，旨在管理有状态的应用。它能够管理 pod 集合的扩展和部署，并且能够保证这些 pod 的唯一性和顺序。

StatefulSet 可以帮助我们处理提供持久化的存储卷。请注意，即便 StatefulSet 中的单个 pod 很容易发生故障，有状态的工作负载也能对故障保持弹性。持久化的 pod 标识符能够将现有的卷与 Kubernetes 新供应的新 pod 进行匹配，以取代发生故障的 pod。

StatefulSet 是如下场景的理想选择：

-   稳定的、唯一的网络标识符。
    
-   有序、优雅的部署和扩展。
    
-   稳定的、持久化的存储。
    
-   有序的、自动的滚动更新。
    

如下是一个来自[Kubernetes文档](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)的样例，展示了 StatefulSet 组件。

这个例子使用`nginx`服务来控制一个网络域。该 StatefulSet 名为 web，它有一个 Spec，表明必须在特定 pod 中启动`nginx`容器的三个副本。它还声明，当使用由 PV Provisioner 提供的 PV 时，由`volumeClaimTemplates`提供稳定的存储。

```
apiVersion: v1
```

复制代码

### DaemonSets

DaemonSets 负责确保所有或特定节点上会运行 pod 的副本。一旦节点被添加到集群中，DaemonSet 所声明的 pod 就会添加到节点中。当节点在集群中移除时，DaemonSet pod 就会被垃圾回收掉。删除 DaemonSet 时，会清理掉它所创建的 pod。

如下是 DaemonSets 的常见使用场景：

-   在每个节点上运行集群存储的 daemon
    
-   在每个节点上运行日志收集的 daemon
    
-   在每个节点上运行节点监控的 daemon
    

针对每种 daemon 类型，你可以定义一个 DaemonSet 涵盖所有的节点。也可以为每种 daemon 类型定义多个 DaemonSets，针对不同类型的硬件使用不同的标记、内存和 CPU。

**创建 DaemonSet**

运行如下的命令在 Kubernetes 集群中创建 DaemonSet：

```
kubectl apply -f [Path to Daemonset spec].yaml
```

复制代码

**定义 DaemonSet 参数**

Kubernetes 允许我们使用 YAML 文件来描述 DaemonSet。下面的`daemonset.yaml`文件样例定义了一个运行`fluentd-elasticsearch` Docker 镜像的 DaemonSet。这个例子也来自官方[文档](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)。

```
apiVersion: apps/v1
```

复制代码

## Kubernetes 中有状态应用的最佳实践

到此为止，我介绍了在 Kubernetes 上运行有状态工作负载的几种方法。这里有一些建议，可以更有效地运行有状态的应用：

-   **有效利用命名空间**：最好是将每个有状态的应用分割到自己的命名空间中，以确保明确的隔离并且更易于进行资源管理。
    
-   **使用 ConfigMap**：所有的脚本和自定义配置应该放到 ConfigMap 中，以确保所有的应用配置都会以声明式的方式来进行处理。
    
-   **服务路由**：随着应用程序的增长，考虑服务路由的可管理性，应该倾向于使用 headless 服务而不是负载均衡器。
    
-   **Secret 管理**：明文 secret 会给生产应用带来严重的安全风险，要确保所有的 secret 都在一个强大的 secret 管理系统中理。
    
-   **谨慎规划存储**：确定应用的持久化存储需求，确保物理存储设备可供集群使用，并以确保每个应用组件所需存储资源的方式定义 Storage Classes 和 PVC。
    

## 结论

在本文中，我阐述了有状态容器化应用的基础知识，并介绍了如何在 Kubernetes 中管理有状态工作负载。这包括以下关键的构件：

-   **PersistentVolume（PV）**：允许我们定义持久化存储单元并将其挂载到 Kubernetes 集群中的 pod 上的构造。
    
-   **PersistentVolumeClaim（PVC）**：允许 pod 动态请求符合其要求的存储的机制。
    
-   **StatefulSet**：控制器，允许创建具有持久化 ID 的 pod，即便 Kubernetes 动态扩展集群中的应用，它也会保持原样。
    
-   **DaemonSets**：控制器，允许集群中的所有节点或特定子集上运行有状态的工作负载。
    

熟悉了这些构件后，你就可以直接在 Kubernetes 集群中创建安全的、可重复运行的有状态的工作负载了。就像 Kubernetes 中的所有内容一样，有状态的机制并不简单，需要时间来掌握，但当你掌握了这些机制后，它就会变得强大而可靠。稍微练习一下，你就能成为一个有状态 Kubernetes 的专家。

**作者简介：**

Gilad David Maayan 是一位技术作家，曾与 150 多家技术公司合作，包括 SAP、Imperva、三星 NEXT、NetApp 和 Check Point，制作技术和思想领导力相关的内容，为开发者和 IT 领导层阐明技术解决方案。

**原文链接：**

[Best Practices for Running Stateful Applications on Kubernetes](https://www.infoq.com/articles/kubernetes-stateful-applications/)