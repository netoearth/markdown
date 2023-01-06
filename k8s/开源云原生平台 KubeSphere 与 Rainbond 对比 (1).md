最近因为工作需要，需要找一个功能完善的云原生应用平台，经过自己筛选和朋友推荐，剩下 KubeSphere 和 Rainbond，这两个产品都是基于 Kubernetes 之上构建的云原生应用平台，功能都非常强大，但产品定位和功能侧重不同，本文将介绍我在选型过程中从各维度对比两款产品的过程记录。

## 产品定位对比

**KubeSphere** 是在 Kubernetes 之上构建的面向云原生应用的分布式操作系统，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。作为全栈的多租户容器平台，KubeSphere 提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。KubeSphere 为用户提供构建企业级 Kubernetes 环境所需的多项功能，例如多云与多集群管理、Kubernetes 资源管理、DevOps、应用生命周期管理、微服务治理（服务网格）、日志查询与收集、服务与网络、多租户管理、监控告警、事件与审计查询、存储管理、访问权限控制、GPU 支持、网络策略、镜像仓库管理以及安全管理等。

**Rainbond** 是一个云原生应用管理平台，使用简单，不需要懂容器、Kubernetes和底层复杂技术，支持管理多个Kubernetes集群，和管理企业应用全生命周期。主要功能包括应用开发环境、应用市场、微服务架构、应用交付、应用运维、应用级多云管理等。Rainbond 遵循 以应用为中心的设计理念，统一封装容器、Kubernetes和底层基础设施相关技术，让使用者专注于业务本身, 避免在业务以外技术上花费大量学习和管理精力。

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| Slogan | 面向云原生应用的混合云平台 | 云原生多云应用管理平台 |
| 抽象 | 容器和K8s概念和抽象为主，应用级抽象为辅 | 应用级抽象 |
| 定位 | 面向懂K8s相关技术的运维和开发 | 面向所有运维和开发，平台管理需要懂K8s |

**由于产品抽象不同，表现出来的概念和流程也有很大差异，KubeSphere主要是Kubernetes相关概念和抽象，使用和管理都需要懂Kubernetes相关体系知识，懂Kubernetes的人能快速上手，Rainbond应用级抽象，使用门槛很低，面向不懂Kubernetes的普通开发人员，平台管理跟KubeSphere一样都需要懂Kubernetes。**

## 开源社区活跃度对比

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 社区活跃度 | 论坛、微信群都活跃 | 微信 钉钉活跃 |
| Stars | 11003 | 3451 |
| 文档成熟度 | 很全面 | 很全面 |
| 版本迭代 | 近一年迭代了4个版本 | 近一年迭代了8个版本 |
| 开源 | 100% 开源 | 100% 开源 |

**KubeSphere 社区更加活跃些，毕竟是万星开源项目，用户遍布国内外。Rainbond 社区用户基本都是国内用户，Star上差了些不过Github、社区群也蛮活跃的。**

## 安装体验对比

**KubeSphere**

支持通过一条命令在 Linux 上快速安装 KubeSphere。

```
./kk create cluster --with-kubernetes v1.22.10 --with-kubesphere v3.3.0
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaBZ3QiakibNeicPO9StjcUwCWznziaibomg1yz3F5Ra5R2oiaNHOVtCn1ZHjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Rainbond**

支持通过一条命令在 Mac、Win、Linux 上快速安装 Rainbond。

```
docker run --privileged -d -p 7070:7070 -p 80:80 -p 6060:6060 rainbond/rainbond:v5.8.1-dind-allinone
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaE6OTqRAPNGEibHlVKrjIwolZStqAoPmWqic7LNAGzn517p0NdBVEqmHA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| Docker Desktop and ARM | 不支持 | 支持 |
| Linux | 支持 | 支持 |
| Kubernetes | 支持 | 支持 |
| 公有云、托管Kubernetes | 支持 | 支持 |
| 安装后组件数量 | 启动所有可拔插组件后 Pod 大概 55 个左右 | 大概 15 个 Pod |

**KubeSphere和Rainbond安装都很简单。KubeSphere 自研的 KubeKey 安装工具，在服务上安装 K8s 和 KubeSphere 很方便。KubeSphere 的可拔插组件这个设计还蛮好的，Allinone安装之后有 5  个 Pod 左右，能满足基本运行需求，需要其他功能就通过可拔插开启，开启所有组件后 Pod 大概 60 个左右。Rainbond 能支持在 Mac M1 Docker Desktop 上安装，这个安装体验还蛮好的可以在本地开发，Rainbond 启动后 Pod 大概15个左右，内存占用1G 左右。**

## 应用部署功能对比

**KubeSphere**

KubeSphere对接git仓库部署源码，支持 Source-to-Image (S2I) 标准工作流将源码打包成镜像，并部署在 Kubernetes 集群中。支持 Java、Python、Node，其他语言可通过自定义 S2I 实现源码构建。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaooWiasyiaw0EAQ9TibQWZZW0W0MKoFJGyY8g3Zjw8eU9dAL4qYmwr2I0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

KubeSphere采用 Binary-to-Image (B2I) 标准工作流将二进制打包成镜像，并部署在 Kubernetes 集群中。支持通过 Jar、War、二进制

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URamxIduQuUGHwN2W6DS12SRDoHCFicJEsVRYOebck8rjoRYG0moicPDP3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

KubeSphere 支持自定义持续构建的流水线![图片](https://mmbiz.qpic.cn/mmbiz_jpg/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URasicK0MWbYGm5aJbC37wyvvvVs86ib1v7ROmdal6OQra8Vt4y3QI9ynkA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**Rainbond**

Rainbond支持对接和整合 Gitlab、Github、Gitee、SVN，实现统一入口![图片](https://mmbiz.qpic.cn/mmbiz_jpg/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaHLIaFd1KlhMvtFE9XSMurriaQ32dpbTfR18l2yPv7M5meseWVT3Vl9Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

Rainbond 的构建支持自动识别源代码类型，支持自动识别 Java Maven、Java Gradle、Java Jar、Java War、Python、PHP、.NetCore、Golang、NodeJS、Static HTML。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URa2aYMEzpymrf3YsibSZic0vRYmYXdPR1bYFZZUfVEqnKOCmRyaouRhHbA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

每种识别的开发语言支持设置环境相关信息，并自动构建成容器镜像。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaNjO0sFJRibERSxyJiabNADzaQ6HEXHIKESiaF6etp1wZtT3oPjV1OPHpw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 源码部署 | 支持 Java、Python、Node | 支持自动识别 Java Maven、Java Gradle、Java Jar、Java War、Python、PHP、.NetCore、Golang、NodeJS、Static HTML |
| 二进制部署 | Jar、War | Jar、War |
| 容器镜像 | 支持容器镜像部署 | 支持容器镜像、docker run、docker compose部署 |
| Kubernetes 应用 | Yaml、Helm | Yaml、Helm |
| 持续交付 | 支持GitOps和自定义流水线步骤 | 支持GitOps |

**KubeSphere 兼容Kubernetes体系，应用部署使用S2I和B2I，KubeSphere自定义流水线功能非常强大，配置灵活。Rainbond 应用部署不需要懂容器和Kubernetes，支持常见的源代码，并自动识别和构建，使用非常简单。**

## 微服务架构功能对比

**KubeSphere**

KubeSphere的微服务架构基于 Istio 实现，支持微服务的流量可视化管理。![图片](https://mmbiz.qpic.cn/mmbiz_jpg/z9BgVMEm7YvcZNt4LxCuLpiaXqHlI6URaMRcA2jTNKK9dvVAMOqIibr0DVtic3ZicSbDRmBicmTDRELhH8ckJYYHgzA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

基于Jaeger的调用链分析

**Rainbond**

Rainbond的微服务架构拓扑和服务编排，通过图形化的编排，添加组件之间的依赖关系，添加后也会注入服务之间的连接信息等。拓扑图可以展示服务之间的关系，用颜色区分服务的状态等。

微服务实时性能分析

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 服务网格支持 | Istio | 内置、Istio、Linkerd |
| 服务拓扑图 | 流量拓扑图 | 微服务依赖关系和服务状态展示 |
| 服务治理 | 熔断、限流 | 插件实现熔断断路器和限流 |
| 微服务可观测性 | 调用链分析 | 通过插件扩展非常多可观测性：性能分析、pinpoint、skywalking、Jaeger等 |
| 微服务编排 | 代码编排 | “拖拉拽”的形式编排微服务依赖关系 |

**KubeSphere 完全依赖 Istio 实现微服务架构，对Istio的功能支持非常完整，KubeSphere弥补了Istio没有图形化的控制面板的不足，简化了 Istio 的上手难度，服务之间的拓扑图是根据流量走向自动生成的，可以直观的看到服务间流量。**

**Rainbond 的服务网格、服务治理、可观测性都是通过插件体系支持的，传统应用开启服务网格插件，马上就能支持微服务架构，服务治理和可观测性也只需要开启相应插件，Rainbond内置了很多插件，有需要还可以自行扩展，可以将自己趁手的工具添加进来，另外，图形化手动编排服务是个特色，不用改代码就可以动态调整依赖关系。**

## 应用市场功能对比

**KubeSphere**

内置应用商店有 30 个应用可直接安装。

基于 Helm Chart 创建应用模板。

发布 Helm Chart 应用模板。

**Rainbond**

内置应用商店有 90+ 应用可直接安装。

支持用户将已经部署好的应用一键发布至应用市场，无需编写复杂的YAML。可以一键发布应用模型内所有元数据，例如依赖关系、配置文件、存储信息等。

支持应用离线导出导入，支持导出 Rainbond App 应用包、Docker Compose 应用包、非容器环境应用包。

支持基于 Rainbond 应用市场一键安装和一键升级，升级会包含应用模型内所有元数据，包括依赖关系等。

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 应用模板 | Helm | Rainbond 应用模版、Helm |
| 应用发布 | 上传 Helm Chart | 一键发布到应用市场 |
| 应用安装 | 一键安装 | 一键安装 |
| 应用升级 | 整体升级 | 部分升级或整体升级 |
| 离线导入导出 | 不支持 | 离线导出多种格式包 |
| 内置应用 | 30 可用应用 | 90+ 可用应用 |

**在应用市场这块Rainbond的功能比KubeSphere强大很多，易用性也好很多。KubeSphere 在应用市场这块是基于标准的 Helm 实现的，在应用发布、安装、升级这套流程里是按照标准的 Helm 应用规范实现，制作 Helm Chart 门槛比较高，功能也受限于Helm。Rainbond 的应用市场 定义了自己的应用模型规范，也支持Helm Chart转成Rainbond的应用模型，应用发布支持一键发布由几十个服务组成的应用，无需编写复杂的YAML，离线导出是在企业软件交付场景非常实用的功能。**

## Kubenetes 多集群管理功能对比

**KubeSphere**

KubeSphere支持对接多个 K8s 集群，支持各种云厂商托管 K8s 集群以及私有云、混合云等。借助 KubeSphere的图形化 Web 控制台，用户可以管理底层的基础架构，例如添加或删除集群。可以使用相同的方式管理部署在任何基础架构上的异构集群。支持跨集群应用分发，资源整合等。支持通过图形化界面管理节点，监控集群状态、应用资源监控、集群告警和通知等。

集群监控

**Rainbond**

支持对接多个 K8s 集群，支持各种云厂商托管 K8s 集群以及私有云、混合云等。支持用户通过控制台添加或删除集群，支持跨集群应用分发。

通过grafana扩展的集群和节点监控

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 多集群管理 | 支持对接多个 K8s 集群 | 支持对接多个 K8s 集群 |
| 集群管理 | 存储管理、节点管理 | 命令行管理 |
| 集群监控和可视化 | 丰富的监控 | 通过grafana扩展的监控 |
| 多租户 | 从平台角色、企业空间角色、项目角色三个维度定义多租户  
支持企业空间、项目进行资源限额，支持多租户的逻辑隔离、网络隔离 | 从企业角色、团队角色两个维度定义多租户  
支持对团队的资源限额，支持多租户的逻辑隔离 |

**KubeSphere 在多集群管理这块比Rainbond体验好，有丰富的监控和可观测性，管理存储和节点在控制台全部完成，Rainbond在集群管理这块需要在命令行下管理，监控功能也弱一些。**

## 应用运维功能对比

### 基本管理

**KubeSphere**

支持对工作负载、容器组级别的管理，支持工作负载的YAML编辑、版本回滚、删除、重新创建等。

支持对容器级别的日志查询过滤，支持全局的日志查询过滤。

KubeSphere 采用 Kubernetes 原生模式进行应用访问，可通过 NodePort、LoadBalancer、Ingress实现外部访问。支持扩展第三方负载均衡控制器以及 Ingress 控制器

**Rainbond**

支持对应用、组件级别的管理，支持应用批量启动、重启、更新、关闭、删除以及组件的操作，支持应用和组件级别的环境变量、版本回滚等。

组件日志实时查看和筛选

Rainbond采用统一的应用网关，支持配置HTTP路由规则和HTTPS证书

由 Rainbond Gateway 统一封装访问，支持http、tcp、udp、grpc访问组件。

|   
 | KubeSphere | Rainbond |
| --- | --- | --- |
| 基本管理 | 支持对工作负载级别的管理 | 支持对应用、组件级别的管理 |
| 日志 | 支持容器组日志查询过滤和全局日志查询过滤 | 支持组件日志查询过滤 |
| 监控 | 支持工作负载级别的告警以及自定义监控图 | 支持组件级别的监控以及图表，告警可扩展 |
| 伸缩 | 支持手工和自动 | 支持手工和自动 |
| 网关 | 支持 NodePort、LoadBalancer 和 Nginx Ingress | 由 Rainbond Gateway 统一封装访问，支持http、tcp、udp、grpc |

**对于基本管理来说 KubeSphere 是原生 K8s 的一些管理，比如删除Pod、编辑YAML、配置环境、自定伸缩等，同样 Rainbond 展现的是应用级概念，比如：在K8s里没有关闭的概念，而在Rainbond里应用不用了直接关闭，想用了再启动，Rainbond做了很多应用级的概念转化，对于不动K8s的开发人员更加容易接受。**

**KubeSphere 在网关这块同样也是遵循了 K8s 原生的模式，通过 NodePort、LoadBalancer、Ingress实现外部访问，并通过图形化操作简化了 YAML 的操作，优点是可以扩展更多第三方 Ingress 控制器，例如 Traefik 等。Rainbond 网关则是通过 Rainbond Gateway 统一封装实现外部访问，简化了用户的操作，一键开启外部访问，同时也能配置 HTTP 的路由规则等，使用的体验非常好。**

## 总结

总体来说，KubeSphere 和 Rainbond 都很成熟，也都有大量开源使用用户，只是定位不同，所以适用场景也会不同。

KubeSphere 在兼容 Kubernetes 生态方面做的非常好，包装和整合了很多云原生的工具，并扩展了对 Kubernetes 和开源工具的管理能力，对于想要管理 Kubernetes 集群的系统管理员是个好的工具，熟悉 Kubernetes 的工程师也可以自行扩展 KubeSphere 的能力。但对开发人员来开发和管理应用来说，门槛比较高，需要学习和理解的概念非常多。

Rainbond 屏蔽了底层复杂的技术，基于应用级抽象，在 Rainbond 的产品闭环里，体验非常好。这适用普通的开发人员开发和管理应用，对于不熟悉 Kubernetes 用户快速起步也是一个不错的选择，在企业软件交付上 Rainbond 非常擅长。但在对 Kubernetes 的系统管理上功能有欠缺。

由于个人知识和经验有限，如有理解不对的地方，还请见谅。

> 本文为作者`张齐`原创投稿，版权归原作者所有。欢迎投稿，投稿请添加微信好友：`iEverything`。