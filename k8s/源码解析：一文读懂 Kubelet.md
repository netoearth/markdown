本文主要介绍 kubelet 功能、核心组件，以及启动流程的源码分析，总结了 kubelet 的工作原理。

## kubelet 简介

![Kubernetes 的架构图](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/15/20210613091144.png)

从官方的架构图中很容易就能找到 `kubelet`

执行 `kubelet -h` 看到 kubelet 的功能介绍：

-   kubelet 是每个 Node 节点上都运行的主要“节点代理”。使用如下的一个向 apiserver 注册 Node 节点：主机的 `hostname`；覆盖 `host` 的参数；或者云提供商指定的逻辑。
-   kubelet 基于 `PodSpec` 工作。`PodSpec` 是用 `YAML` 或者 `JSON` 对象来描述 Pod。Kubelet 接受通过各种机制（主要是 apiserver）提供的一组 `PodSpec`，并确保里面描述的容器良好运行。

除了由 apiserver 提供 `PodSpec`，还可以通过以下方式提供：

-   文件
-   HTTP 端点
-   HTTP 服务器

kubelet 功能归纳一下就是上报 Node 节点信息，和管理（创建、销毁）Pod。 功能看似简单，实际不然。每一个点拿出来都需要很大的篇幅来讲，比如 Node 节点的计算资源，除了传统的 CPU、内存、硬盘，还提供扩展来支持类似 GPU 等资源；Pod 不仅仅有容器，还有相关的网络、安全策略等。

## kubelet 架构

![2021-06-14-21-55-08](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/15/20210614215508.png)

### 重要组件

kubelet 的架构由 N 多的组件组成，下面简单介绍下比较重要的几个：

#### PLEG

即 **Pod Lifecycle Event Generator**，字面意思 Pod 生命周期事件（`ContainerStarted`、`ContainerDied`、`ContainerRemoved`、`ContainerChanged`）生成器。

其维护着 Pod 缓存；定期通过 `ContainerRuntime` 获取 Pod 的信息，与缓存中的信息比较，生成如上的事件；将事件写入其维护的通道（channel）中。

#### PodWorkers

处理事件中 Pod 的同步。核心方法 `managePodLoop()` 间接调用 `kubelet.syncPod()` 完成 Pod 的同步：

-   如果 Pod 正在被创建，记录其延迟
-   生成 Pod 的 API Status，即 `v1.PodStatus`：从运行时的 status 转换成 api status
-   记录 Pod 从 `pending` 到 `running` 的耗时
-   在 `StatusManager` 中更新 pod 的状态
-   杀掉不应该运行的 Pod
-   如果网络插件未就绪，只启动使用了主机网络（host network）的 Pod
-   如果 static pod 不存在，为其创建镜像（Mirror）Pod
-   为 Pod 创建文件系统目录：Pod 目录、卷目录、插件目录
-   使用 `VolumeManager` 为 Pod 挂载卷
-   获取 image pull secrets
-   调用容器运行时（container runtime）的 `#SyncPod()` 方法

#### PodManager

存储 Pod 的期望状态，kubelet 服务的不同渠道的 Pod

#### StatsProvider

提供节点和容器的统计信息，有 `cAdvisor` 和 `CRI` 两种实现。

#### ContainerRuntime

顾名思义，容器运行时。与遵循 CRI 规范的高级容器运行时进行交互。

#### Deps.PodConfig

PodConfig 是一个配置多路复用器，它将许多 Pod 配置源合并成一个单一的一致结构，然后按顺序向监听器传递增量变更通知。

配置源有：文件、apiserver、HTTP

#### `#syncLoop`

接收来自 `PodConfig` 的 Pod 变更通知、定时任务、`PLEG` 的事件，以及 `ProbeManager` 的事件，将 Pod 同步到**期望状态**。

#### PodAdmitHandlers

Pod admission 过程中调用的一系列处理器，比如 eviction handler（节点内存有压力时，不会驱逐 QoS 设置为 `BestEffort` 的 Pod）、shutdown admit handler（当节点关闭时，不处理 pod 的同步操作）等。

#### OOMWatcher

从系统日志中获取容器的 OOM 日志，将其封装成事件并记录。

#### VolumeManger

VolumeManager 运行一组异步循环，根据在此节点上调度的 pod 确定需要附加/挂载/卸载/分离哪些卷并执行操作。

#### CertificateManager

处理证书轮换。

#### ProbeManager

实际上包含了三种 Probe，提供 probe 结果缓存和通道。

-   LivenessManager
-   ReadinessManager
-   StartupManager

#### EvictionManager

监控 Node 节点的资源占用情况，根据驱逐规则驱逐 Pod 释放资源，缓解节点的压力。

#### PluginManager

PluginManager 运行一组异步循环，根据此节点确定哪些插件需要注册/取消注册并执行。如 CSI 驱动和设备管理器插件（Device Plugin）。

##### CSI

Container Storage Interface，由存储厂商实现的存储驱动。

##### 设备管理器插件（Device Plugin）

Kubernetes 提供了一个 设备插件框架，你可以用它来将系统硬件资源发布到 Kubelet。

供应商可以实现设备插件，由你手动部署或作为 DaemonSet 来部署，而不必定制 Kubernetes 本身的代码。目标设备包括 GPU、高性能 NIC、FPGA、 InfiniBand 适配器以及其他类似的、可能需要特定于供应商的初始化和设置的计算资源。

## kubelet 的启动流程

要分析 kubelet 的启动流程，可以从 kubelet 运行方式着手。找一个 Node 节点，很容易就能找到 kubelet 的进程。由于其是以 `systemd` 的方式启动，也可以通过 `systemctl` 查看其状态。

### kubelet 启动命令

kubelet 的启动命令（minikube 环境）

```
$ ps -aux | grep '/kubelet' | grep -v grep
root        4917  2.6  0.3 1857652 106152 ?      Ssl  01:34  13:05 /var/lib/minikube/binaries/v1.21.0/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=1.21.0 --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.64.5
```

或者

```
$ systemctl status kubelet.service
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; disabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sun 2021-06-13 01:34:42 UTC; 11h ago
       Docs: http://kubernetes.io/docs/
   Main PID: 4917 (kubelet)
      Tasks: 15 (limit: 38314)
     Memory: 39.4M
     CGroup: /system.slice/kubelet.service
             └─4917 /var/lib/minikube/binaries/v1.21.0/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=1.21.0 --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.64
```

### 源码分析

从 `git@github.com:kubernetes/kubernetes.git` 仓库获取代码，使用最新的 `release-1.21` 分支。

-   `cmd/kubelet/kubelet.go:35` 的 `main` 方法为程序入口。
    -   调用 `NewKubeletCommand` 方法，创建 command
    -   执行 command
        -   `cmd/kubelet/app/server.go:434` 的 `Run` 方法。
            -   调用 `RunKubelet` 方法。
                -   调用 `createAndInitKubelet` 方法，创建并初始化 kubelet
                    -   `pkg/kubelet/kubelet.go` 的 `NewMainKubelet` 方法，创建 kubelet的 各种组件。共十几个组件，见 [kubelet 的构架](https://atbug.com/#kubelet-%E6%9E%B6%E6%9E%84)。
                    -   调用 `BirtyCry` 方法：放出 `Starting` 事件
                    -   调用 `StartGarbageCollection` 方法，开启 `ContainerGC` 和 `ImageGC`
                -   调用 `startKubelet` 方法（大量使用 goroutine 和通道）
                    -   goroutine：`kubelet.Run()`
                        -   初始化模块
                            -   metrics 相关
                            -   创建文件系统目录目录
                            -   创建容器日志目录
                            -   启动 `ImageGCManager`
                            -   启动 `ServerCertificateManager`
                            -   启动 `OOMWatcher`
                            -   启动 `ResourceAnalyzer`
                        -   goroutine：`VolumeManager.Run()` 开始处理 Pod Volume 的卸载和挂载
                        -   goroutine：状态更新 `fastStatusUpdateOnce()` （更新 Pod CIDR -> 更新 `ContainerRuntime` 状态 -> 更新 Node 节点状态）
                        -   goroutine： `NodeLeaseController.Run()` 更新节点租约
                        -   goroutine：`podKiller.PerformPodKillingWork` 杀掉未被正确处理的 pod
                        -   `StatusManager.Start()` 开始向 apiserver 更新 Pod 状态
                        -   `RuntimeClassManager.Start()`
                        -   `PLEG.Start()`：持续从 `ContainerRuntime` 获取 Pod/容器的状态，并与 kubelet 本地 cache 中的比较，生成对应的 `Event`
                        -   `syncLoop()` 重点，**_持续监控并处理来自文件、apiserver、http 的变更_**。包括 Pod 的增加、更新、优雅删除、非优雅删除、调和。
            -   启动 server，暴露 `/healthz` 端点
            -   通知 `systemd` `kuberlet` 服务已经启动

## kubelet 的工作原理

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/15/20210615000611.png)

1.  来静态文件、apiserver 以及 HTTP 请求的 Pod 配置变更，被发送到 `kubelet.syncLoop`
2.  PLEG 会定期通过容器运行时获取节点上 Pod 的状态，与其缓存中的 Pod 信息进行比较，封装成事件，进入 PLEG 的通道
3.  定期检查工作队列中的 Pod
4.  ProbeManager 的通道中的 Pod
5.  以上 1~4，都会进入 `syncLoopIteration`，并从对应的通道中获取到对应 Pod，将 Pod 的信息保存到 `PodManager`；然后分发给 `PodWorker`，[完成一些列的同步工作](https://atbug.com/#PodWorkers)。

## 总结

kubelet 启动流量就讲到这里，虽然复杂，还是有迹可循。只要了解了 kubelet 在 Kubernetes 中的定位及角色，就很容易理解其工作流量。

后面会再深入分析 Pod 创建及启动流程。