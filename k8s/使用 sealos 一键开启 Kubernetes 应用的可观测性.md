### 一键开启可观测

DeepFlow 基于 eBPF 的零侵扰特性，能让 Kubernetes 及其上应用轻松开启可观测性能力。此前部署 DeepFlow 需要开发者掌握一定的 kubernetes、helm 等相关知识，且安装部署依赖在线镜像仓库和 helm repo。sealos 完备的系统依赖处理方案让 kubernetes、上层应用可以轻松运行在任何操作系统上，并且以其集群镜像、images-shim等方案让任何应用的离线部署变得非常丝滑。今天我们很高兴的和大家分享，sealos 4.0 已经支持了 DeepFlow 集群镜像，从此任何人在任何场景、任何环境可无障碍开启 Kubernetes 应用的可观测性。

基于 sealos 的能力，现在仅需一条命令即可拉起 DeepFlow

```
sealos run \
    labring/kubernetes:v1.24.0 \
    labring/calico:v3.22.1 \
    labring/helm:v3.8.2 \
    labring/openebs:v1.9.0 \
    labring/deepflow:v6.1.1 \
    --masters <Your master IP address> \
    --nodes <Your worker IP address> \
    -p <Your SSH root password>
```

回显内容如下（隐去了部分 sealos 输出）：

```
Release "deepflow" has been upgraded. Happy Helming!
NAME: deepflow
LAST DEPLOYED: Fri Aug  5 14:38:07 2022
NAMESPACE: deepflow
STATUS: deployed
REVISION: 2
NOTES:
██████╗ ███████╗███████╗██████╗ ███████╗██╗      ██████╗ ██╗    ██╗
██╔══██╗██╔════╝██╔════╝██╔══██╗██╔════╝██║     ██╔═══██╗██║    ██║
██║  ██║█████╗  █████╗  ██████╔╝█████╗  ██║     ██║   ██║██║ █╗ ██║
██║  ██║██╔══╝  ██╔══╝  ██╔═══╝ ██╔══╝  ██║     ██║   ██║██║███╗██║
██████╔╝███████╗███████╗██║     ██║     ███████╗╚██████╔╝╚███╔███╔╝
╚═════╝ ╚══════╝╚══════╝╚═╝     ╚═╝     ╚══════╝ ╚═════╝  ╚══╝╚══╝


An automated observability platform for cloud-native developers.


# deepflow-agent Port for receiving trace, metrics, and log


deepflow-agent service: deepflow-agent.deepflow
deepflow-agent Host listening port: 38086


# Get the Grafana URL to visit by running these commands in the same shell


NODE_PORT=$(kubectl get --namespace deepflow -o jsonpath="{.spec.ports[0].nodePort}" services deepflow-grafana)
NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
echo -e "Grafana URL: http://$NODE_IP:$NODE_PORT  \nGrafana auth: admin:deepflow"
```

执行上述回显中最后三行命令，即可获取访问 Grafana 的链接，查看 Kubernetes 上所有 Node、Pod、Service 的指标、分布式追踪、日志数据。

任意微服务的应用 RED 和网络性能指标（在线文档和 Demo）：

![](https://p6.toutiaoimg.com/img/tos-cn-i-qvj2lq49k0/270586d97cc2494a8fc11685586fd7a3~tplv-tt-shrink:640:0.jpg)

任意微服务的分布式调用链追踪，适用于 Linux Kernel 4.14+（在线文档和 Demo）：

![](https://p26.toutiaoimg.com/img/tos-cn-i-qvj2lq49k0/e6e08a827fc047c8a4d442d24ac37748~tplv-tt-shrink:640:0.jpg)

任意微服务的应用请求日志、网络流日志（在线文档和 Demo）：

![](https://p6.toutiaoimg.com/img/tos-cn-i-qvj2lq49k0/64abc8f0f2ba451fbf8f1cffa3b13cdf~tplv-tt-shrink:640:0.jpg)

### 什么是sealos

sealos 是以 kubernetes 为内核的云操作系统发行版, 目前社区内的 sealos、lvscare、image-cri-shim、endpoints-operator 等项目致力于使用云原生的方案解决用户的各种痛点问题；laf 是一款对标腾讯云开发的杀手级函数计算应用。

**sealos 核心特性：**

-   管理集群生命周期
    -   快速安装高可用 Kubernetes 集群
    -   添加/删除节点
    -   清理集群、备份与自动恢复等
-   下载和使用完全兼容 OCI 标准的分布式应用
    -   DeepFlow, OpenEBS, MinIO, Ingress, PostgreSQL, MySQL, Redis 等
-   定制化分布式应用
    -   用 Dockerfile 构建分布式应用镜像，保存所有的依赖
    -   发布分布式应用镜像到 Docker Hub
    -   融合多个应用构建专属的云平台

**GitHub 地址：🔗[GitHub – labring/sealos: sealos is a kubernetes distribution. Let’s sealos run kubernetes:v1.24.0 in 3 minutes!](https://github.com/labring/sealos)**  

sealos 一经发布便获得了众多开发者的一致好评，从 kubernetes 一键安装部署、扩缩容，到率先提出集群镜像的概念及加入images-shim方案，都一直在为开发者丝滑使用 kubernetes 而努力。

**引用 sealos 官网的介绍：**

> 从现在开始，把你数据中心所有机器想象成一台”抽象”的超级计算机，sealos 就是用来管理这台超级计算机的操作系统，kubernetes 就是这个操作系统的内核！
> 
> 云计算从此刻起再无 IaaS PaaS SaaS 之分，只有云操作系统驱动（CSI CNI CRI 实现）云操作系统内核（kubernetes）和分布式应用组成。

### 什么是DeepFlow

DeepFlow 是一款开源的高度自动化的可观测性平台，是为云原生应用开发者建设可观测性能力而量身打造的全栈、全链路、高性能数据引擎。DeepFlow 使用 eBPF、WASM、OpenTelemetry 等新技术，创新的实现了 AutoTracing、AutoMetrics、AutoTagging、SmartEncoding 等核心机制，帮助开发者提升埋点插码的自动化水平，降低可观测性平台的运维复杂度。利用 DeepFlow 的可编程能力和开放接口，开发者可以快速将其融入到自己的可观测性技术栈中。

**GitHub 地址：🔗[GitHub – deepflowys/deepflow: DeepFlow is an automated observability platform for cloud-native developers.](https://github.com/deepflowys/deepflow)**  

访问 DeepFlow Demo，体验高度自动化的可观测性新时代。

![](https://p3.toutiaoimg.com/img/tos-cn-i-qvj2lq49k0/e78070beb6434008a8098a80e76c4f48~tplv-tt-shrink:640:0.jpg)

![](https://p9.toutiaoimg.com/img/tos-cn-i-qvj2lq49k0/c639882032a34e21a954bfe750bc47a8~tplv-tt-shrink:640:0.jpg)