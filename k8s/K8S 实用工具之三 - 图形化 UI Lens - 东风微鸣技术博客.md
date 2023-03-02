## 开篇[](https://ewhisper.cn/posts/33163/#%E5%BC%80%E7%AF%87%20-4)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

-   第一篇：《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》
-   第二篇：《[K8S 实用工具之二 - 终端 UI K9S](https://ewhisper.cn/posts/48546/)》

像我这种，`kubectl` 用的不是非常溜，经常会碰到以下情况：

-   忘记命令，先敲 `--help`，再敲命令，效率低
-   忘记加 `-n` 指定 namespace
-   太长的命令经常记错或敲错，比如 `kubectl exec -it...`
-   无法快速将日志、yaml 复制出来
-   对于 CRD 类资源，记不住 CRD type，查不到相关信息
-   无法掌握集群的健康及监控状态
-   Windows 机器命令行不好用
-   ……

如果你的工作机（前置机、跳板机、操作机、堡垒机…）是 Windows 桌面环境。那么我强烈推荐你使用这个 K8S 实用工具：图形化 UI **[Lens](https://k8slens.dev/)**。

Kubernetes IDE（集成开发环境），可用于：

-   开发
-   调试
-   DevOps
-   运维
-   监控

Lens 是你唯一需要的 IDE ，它可以用来控制你的 Kubernetes 集群。它建立在开源和免费的基础上。

[![Lens UI](https://pic-cdn.ewhisper.cn/img/2021/11/25/14ff5741981146206b76db42a53b1e7c-header-screenshot.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/14ff5741981146206b76db42a53b1e7c-header-screenshot.png "Lens UI")

一个为那些每天使用 Kubernetes 工作的人设计的 IDE，漂亮且强大。

### 💪 Lens 优势[](https://ewhisper.cn/posts/33163/#%F0%9F%92%AA-Lens-%20%E4%BC%98%E5%8A%BF)

-   💡 **移除复杂性**：不需要学习 kubectl 命令就可以探索和导航 Kubernetes 集群。对于刚起步的开发者来说是非常棒的。
-   👁️ **实时可观察性**：实时查看实时统计、事件、日志流。没有转圈圈的加载，刷新或等待屏幕更新。
-   🔨 **定位和调试**：在仪表板上查看错误和警告，然后单击查看详细信息。再次单击以查看日志或获取命令行。
-   💻️ **在你的个人电脑上运行**：MacOS, Windows 和 Linux 上的独立应用程序。1 分钟安装。不需要在集群中安装任何东西。
-   💚 **开源免费**：Lens 基于开源平台，拥有活跃的社区，并得到 Kubernetes 和云原生生态系统先锋的支持。
-   ⎈ **可和任何 Kubernetes 一起工作**：使用 EKS, AKS, GKE, Minikube, Rancher, k0s, k3s, OpenShift…？他们所有都可以正常运行。只需为您想要使用的集群导入 kubeconfigs。

## 个人使用体验[](https://ewhisper.cn/posts/33163/#%E4%B8%AA%E4%BA%BA%E4%BD%BF%E7%94%A8%E4%BD%93%E9%AA%8C)

### 无障碍使用[](https://ewhisper.cn/posts/33163/#%E6%97%A0%E9%9A%9C%E7%A2%8D%E4%BD%BF%E7%94%A8)

Lens 有一个 **统一的目录（Catalog）**。将所有集群、服务、工作负载、工具、自动化和相关资源集中在一起，以便轻松访问。

而且在 Catalog 上，可以很方便进行 **浏览和组织**。使用搜索、过滤、分类和标签来访问你需要工作的资源比以往任何时候都更容易。

[![Lens Catalog](https://pic-cdn.ewhisper.cn/img/2021/11/25/c681572cc2752dbb6568ea7bdbbef802-image-20211125214532292.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/c681572cc2752dbb6568ea7bdbbef802-image-20211125214532292.png "Lens Catalog")

### 效率高[](https://ewhisper.cn/posts/33163/#%E6%95%88%E7%8E%87%E9%AB%98)

Lens 的特色是左边有一列，叫做：**Hotbar**。就是主导航，允许用户在桌面应用程序中构建适合自己的「工作流」和「自动化」。用户可以通过分配不同的标签、颜色和图标来自定义 Hotbar 中的项目，以方便回忆。比如这样：

[![Lens Hotbar](https://pic-cdn.ewhisper.cn/img/2021/11/25/0a0591037fc1e42ad75af3ee2b1a4334-image-20211125215551512.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/0a0591037fc1e42ad75af3ee2b1a4334-image-20211125215551512.png "Lens Hotbar")

它还有类似 VSCode 的 **命令面板**。命令面板允许用户执行特定的键盘快捷键，从而使最常见的任务变得更容易。在使用 Lens 时提高可访问性和效率。Windows 的快捷键是：`Ctrl + Shift + p`。如下图：

[![Lens 命令面板](https://pic-cdn.ewhisper.cn/img/2021/11/25/4a7df2bfe57505667d8e8538f22c56eb-image-20211125215735380.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/4a7df2bfe57505667d8e8538f22c56eb-image-20211125215735380.png "Lens 命令面板")

### 监控整合[](https://ewhisper.cn/posts/33163/#%E7%9B%91%E6%8E%A7%E6%95%B4%E5%90%88)

Lens **内置的可视化**。Lens 与 Prometheus 集成，可以通过总容量、实际使用、请求和限制可视化和查看资源使用指标（包括 CPU、内存、网络和磁盘）的趋势。为每个 k8s 资源自动生成详细的可视化。如下图：

[![Lens CPU 内存 pod](https://pic-cdn.ewhisper.cn/img/2021/11/25/aa3c09b2e3d235cd0029355448348e60-image-20211125220144313.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/aa3c09b2e3d235cd0029355448348e60-image-20211125220144313.png "Lens CPU 内存 pod")

[![Lens Node 监控](https://pic-cdn.ewhisper.cn/img/2021/11/25/7ae139e0b39d115aa01485d5262208fd-image-20211125220303331.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/7ae139e0b39d115aa01485d5262208fd-image-20211125220303331.png "Lens Node 监控")

[![Lens Overview](https://pic-cdn.ewhisper.cn/img/2021/11/25/6888806b62f8c6e96c80067ba9ee0683-image-20211125220409836.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/6888806b62f8c6e96c80067ba9ee0683-image-20211125220409836.png "Lens Overview")

[![Lens Pod 监控](https://pic-cdn.ewhisper.cn/img/2021/11/25/8f716df0daa5e6b6ff55b68f4f2057e0-image-20211125220528814.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/8f716df0daa5e6b6ff55b68f4f2057e0-image-20211125220528814.png "Lens Pod 监控")

### 所有 K8S 资源触手可达[](https://ewhisper.cn/posts/33163/#%E6%89%80%E6%9C%89%20-K8S-%20%E8%B5%84%E6%BA%90%E8%A7%A6%E6%89%8B%E5%8F%AF%E8%BE%BE)

**智能终端** 功能。Lens 智能终端自带 kubectl 和 helm，自动同步 kubectl 的版本，以匹配当前选择的 K8S 集群 API 版本。Lens 会自动分配 kubeconfig 上下文来匹配当前选择的 K8s 集群。

[![Lens 智能终端](https://pic-cdn.ewhisper.cn/img/2021/11/25/150c79b3ff74356c168ce67acff4dffb-image-20211125221022602.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/150c79b3ff74356c168ce67acff4dffb-image-20211125221022602.png "Lens 智能终端")

### K8S 资源模板[](https://ewhisper.cn/posts/33163/#K8S-%20%E8%B5%84%E6%BA%90%E6%A8%A1%E6%9D%BF)

自带全量 K8S 资源模板，而且是有丰富信息的模板，直接在模板上照猫画虎就可完成各类资源的创建，妈妈再也不用担心我忘记 K8S Resources 的 Spec 了！

[![Lens 资源模板](https://pic-cdn.ewhisper.cn/img/2021/11/25/cd6a941acedb3564316cc5fe26772fc3-image-20211125225429603.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/cd6a941acedb3564316cc5fe26772fc3-image-20211125225429603.png "Lens 资源模板")

### 快速部署[](https://ewhisper.cn/posts/33163/#%E5%BF%AB%E9%80%9F%E9%83%A8%E7%BD%B2)

**Helm Chart**。Lens 自带 Helm Chart 管理，允许发现和快速部署数以千计的公开可用的 Helm Chart 和管理您自己的存储库。探索已安装的 Helm Chart ，只需一次点击即可修订和升级。

如下图：

[![Lens Helm Chart 仓库](https://pic-cdn.ewhisper.cn/img/2021/11/25/87582951d7619a5812b1fd5040c973b9-image-20211125221401388.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/87582951d7619a5812b1fd5040c973b9-image-20211125221401388.png "Lens Helm Chart 仓库")

[![Helm Chart 一键升级](https://pic-cdn.ewhisper.cn/img/2021/11/25/34f407291bbc360db7ac18406aa63d81-image-20211125221511377.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/34f407291bbc360db7ac18406aa63d81-image-20211125221511377.png "Helm Chart 一键升级")

[![Lens Helm 已安装资源展示](https://pic-cdn.ewhisper.cn/img/2021/11/25/63b0e35c8335c6ebff1bbbeaa40a5a97-image-20211125221556109.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/63b0e35c8335c6ebff1bbbeaa40a5a97-image-20211125221556109.png "Lens Helm 已安装资源展示")

### 插件！[](https://ewhisper.cn/posts/33163/#%E6%8F%92%E4%BB%B6%EF%BC%81)

**支持插件**。轻松地从社区和云本地生态系统供应商添加 Lens 扩展或构建自己的。Lens Extensions 用于添加自定义功能和服务，以加速与 Kubernetes 和其他云原生技术集成的所有技术的开发流程。

这里推荐几个实用的插件：

#### lens-certificate-info[](https://ewhisper.cn/posts/33163/#lens-certificate-info)

查看证书信息。查看含有证书信息的 Secret，效果如下：

[![lens-certificate-info](https://pic-cdn.ewhisper.cn/img/2021/11/25/0a92f833a0cb6368b29dbe62915517d1-image-20211125222329093.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/0a92f833a0cb6368b29dbe62915517d1-image-20211125222329093.png "lens-certificate-info")

#### lens-debug-tools[](https://ewhisper.cn/posts/33163/#lens-debug-tools)

配合 [K8S 1.16 的新功能](https://www.shogan.co.uk/kubernetes/enabling-and-using-ephemeral-containers-on-kubernetes-1-16/)，可以在想要调试的 Pod 里插入带有丰富工具集的 Sidecar（为了追求 Size，一般镜像都是非常精简，导致常用命令缺失，调试困难），方便调试。

还可以配置调试用的镜像，还贴心的给了 3 个推荐：

| Name | Description | Link |
| --- | --- | --- |
| busybox | Default value | [https://hub.docker.com/\_/busybox](https://hub.docker.com/_/busybox) |
| markeijsermans/debug |  | [https://hub.docker.com/r/markeijsermans/debug](https://hub.docker.com/r/markeijsermans/debug) |
| praqma/network-multitool |  | [https://hub.docker.com/r/praqma/network-multitool](https://hub.docker.com/r/praqma/network-multitool) |

安装完成后 Pod 页面会多一个按钮：

[![Lens Debug Pod 按钮](https://pic-cdn.ewhisper.cn/img/2021/11/25/6f4977426b1c92010484647998c622ca-image-20211125222731180.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/6f4977426b1c92010484647998c622ca-image-20211125222731180.png "Lens Debug Pod 按钮")

有 2 种模式：

一种是「Run as debug pod」，就是在同一台 Node 上启动一个新 pod，可以用来分析调试与 Node 有关的问题。自动执行的命令如下：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl run loki-promtail-5d5h8-debug -n loki-stack -it --image=busybox --restart=Never  --attach  --overrides=<span>'{ \"spec\": { \"nodeName\": \"izuf656om146vu1n6pd6lpz\" } }'</span> --labels=createdBy=lens-debug-extension --rm<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

另一种是「Run as emepheral container」，需要启用 K8S 1.16 的新功能才能使用。直接是在要调试的 Pod 里启动一个 Debug Sidecar，就可以分析调试与 Node、Pod 有关的问题。自动执行的命令如下：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl debug -i -t -n loki-stack loki-promtail-5d5h8 --image=busybox --target promtail --attach<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### @nevalla/kube-resource-map[](https://ewhisper.cn/posts/33163/#nevalla-kube-resource-map)

资源拓扑图。这个真的是心头好。来看看 Monitoring 的拓扑图：

从 Helm，到 StatefulSet，到 Pod，到 SVC，再到 ConfigMap、Secret，一应俱全。

[![Lens 拓扑图](https://pic-cdn.ewhisper.cn/img/2021/11/25/91b271a5ec54043ebe25355b74cd79b2-image-20211125224803962.png)](https://pic-cdn.ewhisper.cn/img/2021/11/25/91b271a5ec54043ebe25355b74cd79b2-image-20211125224803962.png "Lens 拓扑图")

## ✍ 总结[](https://ewhisper.cn/posts/33163/#%E2%9C%8D-%20%E6%80%BB%E7%BB%93)

Lens 是一个**Kubernetes IDE**，在桌面环境下使用它，来开发、调试、DevOps、运维和监控。

它有很多强大的功能，其中：Catalog、Hotbar、命令面板、监控、智能终端、资源模板、Helm Chart 管理和插件 这些功能一定要试一试，体验飞升！

一起使用吧~ 🤓🤓🤓