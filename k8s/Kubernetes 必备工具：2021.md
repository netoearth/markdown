有别于前些天的文章 - [常用的几款工具让 Kubernetes 集群上的工作更容易](https://mp.weixin.qq.com/s/uU2zmT5yyVcKZ5XmLSRqtg) 偏重于工具类来提升工作效率，今天这篇文章更加适合用来做选型时的参考。

文档翻译自 [Kubernetes Essential Tools: 2021](https://itnext.io/kubernetes-essential-tools-2021-def12e84c572)，篇幅较长，做了部分增删。

___

## 介绍

在本文中，我将尝试总结我最喜欢的 [Kubernetes](https://kubernetes.io/) 工具，并特别强调最新的和鲜为人知但我认为会非常流行的工具。

这只是我根据我的经验得出的个人清单，但为了避免偏见，我还将尝试提及每种工具的替代方案，以便你可以根据自己的需要进行比较和决定。我将尽可能缩短这篇文章并提供链接，以便你可以自行探索更多内容。我的目标是回答这个问题：“我如何在 Kubernetes 中做 X？” 通过描述不同软件开发任务的工具。

## K3D

[K3D](https://k3d.io/) 是我最喜欢的在笔记本电脑上运行 Kubernetes (K8s) 集群的方式。它非常**轻巧且**速度非常快。它是使用 **Docker** 围绕 [K3S](https://k3s.io/) 的包装器。所以，你只需要 Docker 来运行它并且资源使用率非常低。唯一的问题是**它不完全符合 K8s 标准**，但这不应该是本地开发的问题。对于测试环境，你可以使用其他解决方案。K3D 比 Kind 快，但 Kind 完全兼容。

### 备选

-   [**K3S**](https://k3s.io/) 物联网或者边缘计算
-   [**Kind**](https://kind.sigs.k8s.io/) 完全兼容 Kubernetes 的备选
-   [**MicroK8s**](https://microk8s.io/)
-   [**MiniKube**](https://minikube.sigs.k8s.io/docs/)

## Krew

[Krew](https://krew.sigs.k8s.io/) 是管理的必备工具 **Kubectl 插件**，这是一个必须有任何 K8S 用户。我不会详细介绍超过 145 个可用[插件](https://krew.sigs.k8s.io/plugins/)，但至少安装 [**kubens**](https://github.com/ahmetb/kubectx) 和 [**kubectx**](https://github.com/ahmetb/kubectx)。

## Lens

[Lens](https://k8slens.dev/) 是适用于 SRE、Ops 和开发人员的 K8s **IDE**。它适用于任何 Kubernetes 发行版：本地或云端。它快速、易于使用并提供实时可观察性。使用 Lens 可以非常轻松地管理多个集群。如果你是集群操作员，这是必须的。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263041512885.jpg)

### 备选

-   对于那些喜欢轻量级终端替代品的人来说，[K9s](https://k9scli.io/) 是一个很好的选择。K9s 会持续观察 Kubernetes 的变化，并提供后续命令来与你观察到的资源进行交互。

## Helm

[Helm](https://helm.sh/) 不需要介绍，它是 Kubernetes 最著名的包管理器。你应该在 K8s 中使用包管理器，就像在编程语言中使用它一样。Helm 允许你将应用程序打包到 [Charts](https://artifacthub.io/) 中，将复杂的应用程序抽象为易于定义、安装和更新的可重用简单组件。

它还提供了强大的模板引擎。Helm 很成熟，有很多预定义的 charts，很好的支持，而且很容易使用。

### 备选

-   [**Kustomize**](https://kustomize.io/) 是 helm 的一个更新和伟大的替代品，它不使用模板引擎，而是一个覆盖引擎，在其中你有基本的定义和覆盖在它们之上。

## ArgoCD

我相信 [GitOps](https://www.gitops.tech/) 是过去十年中最好的想法之一。在软件开发中，我们应该使用单一的事实来源来跟踪构建软件所需的所有移动部分，而 **Git** 是做到这一点的完美工具。我们的想法是拥有一个 Git 存储库，其中包含应用程序代码以及表示所需生产环境状态的基础设施 ([IaC](https://en.wikipedia.org/wiki/Infrastructure_as_Code)) 的声明性描述；以及使所需环境与存储库中描述的状态相匹配的自动化过程。

> GitOps: versioned CI/CD on top of declarative infrastructure. Stop scripting and start shipping.
> 
> — Kelsey Hightower

尽管使用 [Terraform](https://www.terraform.io/) 或类似工具，你可以实现基础设施即代码（[IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code)），但这还不足以将你在 Git 中的所需状态与生产同步。我们需要一种方法来持续监控环境并确保没有配置漂移。使用 Terraform，你将不得不编写脚本来运行`terraform apply`并检查状态是否与 Terraform 状态匹配，但这既乏味又难以维护。

Kubernetes 从头开始构建控制循环的思想，这意味着 Kubernetes 一直在监视集群的状态以确保它与所需的状态匹配，例如，运行的副本数量与所需的数量相匹配复制品。GitOps 的想法是将其扩展到应用程序，因此你可以将你的服务定义为代码，例如，通过定义 Helm Charts，并使用利用 K8s 功能的工具来监控你的应用程序的状态并相应地调整集群。也就是说，如果更新你的代码存储库或Helm Chart，生产集群也会更新。这是真正的[持续部署](https://en.wikipedia.org/wiki/Continuous_deployment)。核心原则是应用程序部署和生命周期管理应该自动化、可审计且易于理解。

对我来说，这个想法是革命性的，如果做得好，将使组织能够更多地关注功能，而不是编写自动化脚本。这个概念可以扩展到软件开发的其他领域，例如，你可以将文档存储在代码中以跟踪更改的历史并确保文档是最新的；或使用[ADR](https://github.com/jamesmh/architecture_decision_record)跟踪架构决策。

在我看来，在最好的 GitOps 工具 **Kubernetes** 是 [ArgoCD](https://argoproj.github.io/argo-cd/)。你可以在[此处](https://argoproj.github.io/argo-cd/core_concepts/)阅读更多信息。ArgoCD 是 **Argo** 生态系统的一部分，其中包括一些其他很棒的工具，其中一些我们将在稍后讨论。

使用 **ArgoCD**，你可以在代码存储库中拥有每个环境，你可以在其中定义该环境的所有配置。Argo CD 在指定的目标环境中自动部署所需的应用程序状态。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263046811430.jpg)

ArgoCD 被实现为一个 kubernetes 控制器，它持续监控正在运行的应用程序并将当前的实时状态与所需的目标状态（如 Git 存储库中指定的）进行比较。ArgoCD 报告并可视化差异，并且可以自动或手动将实时状态同步回所需的目标状态。

### 备选

-   [Flux](https://fluxcd.io/) 刚刚发布了一个具有许多改进的新版本。它提供了非常相似的功能。

## Argo 工作流（Workflows）和 Argo 事件（Events）

在 Kubernetes 中，你可能还需要运行批处理作业或复杂的工作流。这可能是你的数据管道、异步流程甚至 CI/CD 的一部分。最重要的是，你甚至可能需要运行对某些事件做出反应的驱动微服务，例如文件上传或消息发送到队列。对于所有这些，我们有 [Argo Workflows](https://argoproj.github.io/argo-workflows/) 和 [Argo Events](https://argoproj.github.io/argo-events/)。

尽管它们是独立的项目，但它们往往会被部署在一起。

Argo Workflows 是一个类似于 [Apache Airflow](https://airflow.apache.org/) 的编排引擎，但它是 Kubernetes 原生的。它使用自定义 CRD 来定义复杂的工作流程，使用 YAML 的步骤或 **DAG**，这在 K8s 中感觉更自然。它有一个漂亮的 UI、重试机制、基于 cron 的作业、输入和输出跟踪等等。你可以使用它来编排数据管道、批处理作业等等。

有时，你可能希望将管道与异步服务（如 **Kafka** 等流引擎、队列、webhooks 或深度存储服务）集成。例如，你可能想要对上传到 S3 的文件等事件做出反应。为此，你将使用 [Argo 事件（Event）](https://argoproj.github.io/argo-events/)。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263049343226.jpg)

这两个工具组合为你的所有管道需求提供了一个简单而强大的解决方案，包括 CI/CD 管道，它允许你在 Kubernetes 中本地运行 CI/CD 管道。

### 备选

-   对于 ML 管道，你可以使用 [Kubeflow](https://www.kubeflow.org/)。
-   对于 CI/CD 管道，你可以使用 [Tekton](https://tekton.dev/docs/pipelines/pipelines/)。

## Kaniko

我们刚刚看到了如何使用 Argo Workflows 运行 Kubernetes 原生 **CI/CD** 管道。一个常见的任务是构建 **Docker 镜像**，这在 Kubernetes 中通常是乏味的，因为构建过程实际上是在容器本身上运行的，你需要使用变通方法来使用主机的 Docker 引擎。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263050297533.jpg)

底线是你不应该使用 **Docker** 来构建你的镜像：改用 [**Kanico**](https://github.com/GoogleContainerTools/kaniko)。Kaniko 不依赖于 Docker 守护进程，而是完全在用户空间中执行 Dockerfile 中的每个命令。这使得在无法轻松或安全地运行 Docker 守护程序的环境中构建容器镜像成为可能，例如标准的 Kubernetes 集群。这消除了在 K8s 集群中构建镜像的所有问题。

## Istio

[Istio](https://istio.io/) 是市场上最著名的[服务网格](https://en.wikipedia.org/wiki/Service_mesh)，它是开源的并且非常受欢迎。我不会详细介绍什么是服务网格，因为它是一个巨大的话题，但是如果你正在构建[微服务](https://microservices.io/)，并且可能应该这样做，那么你将需要一个服务网格来管理通信、可观察性、错误处理、安全性。与其用重复的逻辑污染每个微服务的代码（译者：SDK 侵入），不如利用服务网格为你做这件事。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263052962147.jpg)

简而言之，服务网格是一个专用的基础设施层，你可以将其添加到你的应用程序中。它允许你透明地添加可观察性、流量管理和安全性等功能，而无需将它们添加到你自己的代码中。

如果 **Istio** 用于运行微服务，尽管你可以在任何地方运行 Istio 并使用微服务，但 Kubernetes 已被一次又一次地证明是运行它们的最佳平台。**Istio** 还可以将你的 K8s 集群扩展到其他服务，例如 VM，允许你拥有在迁移到 Kubernetes 时非常有用的混合环境。

### 备选

-   [**Linkerd**](https://linkerd.io/) 是一种更轻巧且可能更快的服务网格。Linkerd 从头开始为安全性而构建，包括[默认 mTLS](https://linkerd.io/2.10/features/automatic-mtls/)、[使用 Rust 构建的数据平面](https://github.com/linkerd/linkerd2-proxy)、[内存安全语言](https://github.com/linkerd/linkerd2-proxy)和[定期安全审计](https://github.com/linkerd/linkerd2/blob/main/SECURITY_AUDIT.pdf)等功能
-   [**Consul**](https://www.consul.io/) 是为任何运行时和云提供商构建的服务网格，因此它非常适合跨 K8s 和云提供商的混合部署。如果不是所有的工作负载都在 Kubernetes 上运行，这是一个不错的选择。

## Argo Rollouts

我们已经提到，你可以使用 Kubernetes 使用 Argo Workflows 或使用 Kanico 构建图像的类似工具来运行 CI/CD 管道。下一个合乎逻辑的步骤是继续并进行持续部署。由于涉及高风险，这在真实场景中是极具挑战性的，这就是为什么大多数公司只做持续交付，这意味着他们已经实现了自动化，但他们仍然需要手动批准和验证，这个手动步骤是这是因为团队**不能完全信任他们的自动化**。

那么，你如何建立这种信任以摆脱所有脚本并完全自动化从源代码到生产的所有内容？答案是：可观察性。你需要将资源更多地集中在指标上，并收集准确表示应用程序状态所需的所有数据。目标是使用一组指标来建立这种信任。如果你在 [Prometheus](https://prometheus.io/) 中拥有所有数据，那么你可以自动部署，因为你可以根据这些指标自动逐步推出应用程序。

简而言之，你需要比 K8s 开箱即用的[**滚动更新**](https://www.educative.io/blog/kubernetes-deployments-strategies)更高级的部署技术。我们需要使用[金丝雀部署](https://semaphoreci.com/blog/what-is-canary-deployment)进行渐进式交付。目标是逐步将流量路由到应用程序的新版本，等待收集指标，分析它们并将它们与预定义的规则进行匹配。如果一切正常，我们增加流量；如果有任何问题，我们会回滚部署。 要在 Kubernetes 中执行此操作，你可以使用提供 Canary 发布等的 Argo Rollouts 。

> [Argo Rollouts](https://kubernetes.io/docs/concepts/architecture/controller/) 是一个 [Kubernetes 控制器](https://kubernetes.io/docs/concepts/architecture/controller/)和一组 [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)，可提供高级部署功能，例如蓝绿、金丝雀、金丝雀分析、实验和向 Kubernetes 的渐进式交付功能。

尽管像 [Istio](https://istio.io/) 这样的服务网格提供 Canary 发布，但 Argo Rollouts 使这个过程变得更加容易并且以开发人员为中心，因为它是专门为此目的而构建的。除此之外，Argo Rollouts 可以与任何服务网格集成。

Argo Rollouts 功能：

-   蓝绿更新策略
-   金丝雀更新策略
-   细粒度、加权的流量转移
-   自动回滚和促销或人工判断
-   可定制的指标查询和业务 KPI 分析
-   入口控制器集成：NGINX、ALB
-   服务网格集成：Istio、Linkerd、SMI
-   指标提供者集成：Prometheus、Wavefront、Kayenta、Web、Kubernetes Jobs

### 备选

-   [Istio](https://istio.io/) 作为 Canary 版本的服务网格。Istio 不仅仅是一个渐进式交付工具，它还是一个完整的服务网格。Istio 不会自动部署，Argo Rollouts 可以与 Istio 集成来实现这一点。
-   [Flagger](https://flagger.app/) 与 Argo Rollouts 非常相似，并且与 [Flux](https://fluxcd.io/) 很好地集成在一起，因此如果你使用 Flux，请考虑使用 Flagger。
-   [Spinnaker](https://spinnaker.io/) 是 Kubernetes 的第一个持续交付工具，它具有许多功能，但使用和设置起来有点复杂。

## Crossplane

[**Crossplane**](https://crossplane.io/) 是我最喜欢的 K8s 工具，我对这个项目感到非常兴奋，因为它给 Kubernetes 带来了一个关键的缺失部分：像管理 K8s 资源一样管理第三方服务。这意味着，你可以使用 **YAML** 中定义的 K8s 资源来配置云提供商数据库，例如 [**AWS RDS**](https://aws.amazon.com/rds/) 或 **GCP Cloud SQL**，就像你在 K8s 中配置数据库一样。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263059248680.jpg)

使用 Crossplane，无需使用不同的工具和方法分离基础设施和代码。**你可以使用 K8s 资源定义一切**。这样，你就无需学习 [Terraform](https://www.terraform.io/) 等新工具并将它们分开保存。

> Crossplane 是一个开源 Kubernetes 附加组件，它使平台团队能够组装来自多个供应商的基础设施，并公开更高级别的自助 API 供应用程序团队使用，而无需编写任何代码。

**Crossplane** 扩展了你的 Kubernetes 集群，为你提供适用于任何基础架构或托管云服务的 **CRD**。此外，它允许你完全实现持续部署，因为与 Terraform 等其他工具相反，Crossplane 使用现有的 K8s 功能（例如控制循环）来持续观察你的集群并自动检测任何对其起作用的配置漂移。例如，如果你定义了一个托管数据库实例并且有人手动更改它，Crossplane 将自动检测问题并将其设置回以前的值。这将实施基础设施即代码和 GitOps 原则。Crossplane 与 ArgoCD 配合使用效果很好，它可以查看源代码并确保你的代码存储库是唯一的真实来源，并且代码中的任何更改都会传播到集群以及外部云服务。如果没有 Crossplane，你只能在 K8s 服务中实现 GitOps，而不能在不使用单独进程的情况下在云服务中实现，现在你可以做到这一点，这太棒了。

### 备选

-   [Terraform](https://www.terraform.io/) 是最著名的 IaC 工具，但它不是 K8s 原生的，需要新技能并且不会自动监视配置漂移。
-   [Pulumi](https://www.pulumi.com/) 是一种 Terraform 替代品，它使用开发人员可以测试和理解的编程语言工作。

## Knative

如果你在云中开发应用程序，你可能已经使用了一些无服务器技术，例如 [AWS Lambda](https://aws.amazon.com/lambda/) ，它是一种称为 [FaaS](https://en.wikipedia.org/wiki/Function_as_a_service) 的事件驱动范例。

我过去已经讨论过 [Serverless](https://en.wikipedia.org/wiki/Serverless_computing)，因此请查看我[之前的文章](https://itnext.io/scaling-my-app-serverless-vs-kubernetes-cdb8adf446e1)以了解更多信息。Serverless 的问题在于它与云提供商紧密耦合，因为提供商可以为事件驱动的应用程序创建一个很好的生态系统。

对于 Kubernetes，如果你希望将函数作为代码运行并使用事件驱动架构，那么你最好的选择是 [Knative](https://itnext.io/scaling-my-app-serverless-vs-kubernetes-cdb8adf446e1)。Knative 旨在在 Kubernetes 上运行函数，在 Pod 之上创建一个抽象。

特点：

-   针对常见应用程序用例的具有更高级别抽象的重点 API。
-   在几秒钟内建立一个可扩展、安全、无状态的服务。
-   松散耦合的功能让你可以使用所需的部分。
-   可插拔组件让你可以使用自己的日志记录和监控、网络和服务网格。
-   Knative 是可移植的：在 Kubernetes 运行的任何地方运行它，不用担心供应商锁定。
-   惯用的开发者体验，支持 GitOps、DockerOps、ManualOps 等常用模式。
-   Knative 可以与常见的工具和框架一起使用，例如 Django、Ruby on Rails、Spring 等等。

### 备选

-   [Argo Events](https://argoproj.github.io/argo-events/) 为 Kubernetes 提供了一个事件驱动的工作流引擎，可以与 AWS Lambda 等云引擎集成。它不是 FaaS，而是为 Kubernetes 提供了一个事件驱动的架构。
-   [OpenFaas](https://www.openfaas.com/)

## Kyverno

Kubernetes 提供了极大的灵活性，以赋予敏捷的自治团队权力，但能力越大，责任越大。必须有一组**最佳实践和规则**，以确保以一致且有凝聚力的方式来部署和管理符合公司政策和安全要求的工作负载。

有几种工具可以实现这一点，但没有一个是 Kubernetes 原生的…… 直到现在。[Kyverno](https://kyverno.io/) 是为 Kubernetes 设计的策略引擎，策略作为 Kubernetes 资源进行管理，并且不需要新的语言来编写策略。Kyverno 策略可以验证、改变和生成 Kubernetes 资源。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263062593080.jpg)

你可以应用有关最佳实践、网络或安全性的任何类型的策略。例如，你可以强制所有服务都有标签或所有容器都以非 root 身份运行。你可以在[此处](https://github.com/kyverno/policies/)查看一些政策示例。策略可以应用于整个集群或给定的命名空间。你还可以选择是只想审核策略还是强制执行它们以阻止用户部署资源。

### 备选

-   [Open Policy Agent](https://www.openpolicyagent.org/) 是著名的云原生基于策略的控制引擎。它使用自己的声明性语言，并且可以在许多环境中运行，而不仅仅是在 Kubernetes 上。它比 [Kyverno](https://kyverno.io/) 更难管理，但更强大。

## Kubevela

Kubernetes 的一个问题是开发人员需要非常了解和理解平台和集群配置。许多人会争辩说 **K8s 的抽象级别太低**，这会给只想专注于编写和交付应用程序的开发人员带来很多摩擦。

在开放式应用程序模型（[OAM](https://oam.dev/)）的设立是为了克服这个问题。这个想法是围绕应用程序创建更高级别的抽象，它独立于底层运行时。你可以在此处阅读规范。

> 专注于应用程序而不是容器或协调器，开放应用程序模型 \[OAM\] 带来了模块化、可扩展和可移植的设计，用于使用更高级别但一致的 API 对应用程序部署进行建模。

[Kubevela](https://kubevela.io/) 是 OAM 模型的一个实现。KubeVela 与运行时无关，可本地扩展，但最重要的是，以应用程序为中心 。在 Kubevela 中，应用程序是作为 Kubernetes 资源实现的一等公民。\*\*集群运营商（Platform Team）和开发者（Application Team）\*\*是有区别的。集群操作员通过定义组件（组成应用程序的可部署/可配置实体，如 Helm Chart）和特征来管理集群和不同的环境。开发人员通过组装组件和特征来定义应用程序。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263064905339.jpg)

KubeVela 是一个[云原生计算基金会](https://cncf.io/)沙箱项目，虽然它仍处于起步阶段，但它可以在不久的将来改变我们使用 Kubernetes 的方式，让开发人员无需成为 Kubernetes 专家即可专注于应用程序。但是，我确实对 **OAM** 在现实世界中的适用性有一些担忧，因为系统应用程序、ML 或大数据过程等一些服务在很大程度上依赖于低级细节，这些细节可能很难融入 OAM 模型中。

### 备选

-   [Shipa](https://www.shipa.io/getting-started/) 遵循类似的方法，使平台和开发团队能够协同工作，轻松将应用程序部署到 Kubernetes。

## Snyk

任何开发过程中一个非常重要的方面是安全性，这一直是 Kubernetes 的一个问题，因为想要迁移到 Kubernetes 的公司无法轻松实现其当前的安全原则。

[Snyk](https://snyk.io/) 试图通过提供一个可以轻松与 Kubernetes 集成的安全框架来缓解这种情况。它可以检测容器映像、你的代码、开源项目等中的漏洞。

### 备选

-   [Falco](https://falco.org/) 是 Kubernetes 的运行时安全线程检测工具。

## Velero

如果你在 Kubernetes 中运行工作负载并使用卷来存储数据，则需要创建和管理备份。[Velero](https://velero.io/) 提供简单的备份/恢复过程、灾难恢复机制和数据迁移。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263066379840.jpg)

与其他直接访问 Kubernetes etcd 数据库执行备份和恢复的工具不同，Velero 使用 Kubernetes API 来捕获集群资源的状态并在必要时恢复它们。此外，Velero 使你能够在配置的同时备份和恢复你的应用程序持久数据。

## Schema Hero

软件开发中的另一个常见过程是在使用关系数据库时管理**模式演变**。

[SchemaHero](https://schemahero.io/) 是一种开源数据库架构迁移工具，可将架构定义转换为可应用于任何环境的迁移脚本。它使用 Kubernetes 声明性来管理数据库模式迁移。你只需指定所需的状态，然后 **SchemaHero** 管理其余的。

### 备选

-   [LiquidBase](https://www.liquibase.org/) 是最著名的替代品。它更难使用，且不是 Kubernetes 原生的，但它具有更多功能。

## Bitnami Sealed Secrets

我们已经介绍了许多 **GitOps** 工具，例如 [ArgoCD](https://argoproj.github.io/argo-cd/)。我们的目标是将所有内容保留在 Git 中，并使用 Kubernetes 声明性来保持环境同步。我们刚刚看到我们如何（并且应该）在 Git 中保留真实来源，并让自动化流程处理配置更改。

在 Git 中通常很难保留的一件事是诸如数据库密码或 API 密钥之类的秘密，这是因为你永远不应该在代码存储库中存储秘密。一种常见的解决方案是使用外部保管库（例如 [AWS Secret Manager](https://aws.amazon.com/secrets-manager/) 或 [HashiCorp Vault](https://www.vaultproject.io/)）来存储机密，但这会产生很多摩擦，因为你需要有一个单独的流程来处理机密。理想情况下，我们希望有一种方法可以像任何其他资源一样安全地在 Git 中存储机密。

[**Sealed Secrets**](https://github.com/bitnami-labs/sealed-secrets) 旨在克服这个问题，允许你使用强加密将敏感数据存储在 Git 中。Bitnami **Sealed Secrets** 本地集成在 Kubernetes 中，允许你仅通过在 Kubernetes 中运行的 Kubernetes 控制器而不是其他任何人来解密密钥。控制器将解密数据并创建安全存储的原生 K8s 机密。这使我们能够将所有内容作为代码存储在我们的 repo 中，从而允许我们安全地执行持续部署，而无需任何外部依赖。

Sealed Secrets 由两部分组成：

-   集群端控制器
-   客户端实用程序：`kubeseal`

该 `kubeseal` 实用程序使用非对称加密来加密只有控制器才能解密的机密。这些加密的秘密被编码在一个 `SealedSecret` K8s 资源中，你可以将其存储在 Git 中。

### 备选

-   [AWS Secret Manager](https://aws.amazon.com/secrets-manager/)
-   [HashiCorp Vault](https://www.vaultproject.io/)）

## Capsule

许多公司使用**多租户**来管理不同的客户。这在软件开发中很常见，但在 Kubernetes 中很难实现。**命名空间**是将集群的逻辑分区创建为隔离切片的好方法，但这不足以安全地隔离客户，我们需要强制执行网络策略、配额等。你可以为每个名称空间创建网络策略和规则，但这是一个难以扩展的乏味过程。此外，租户将不能使用多个命名空间，这是一个很大的限制。

创建**分层命名空间**是为了克服其中一些问题。这个想法是为每个租户拥有一个父命名空间，为租户提供公共网络策略和配额，并允许创建子命名空间。这是一个很大的改进，但它在安全和治理方面没有对租户的本地支持。此外，它还没有达到生产状态，但 1.0 版预计将在未来几个月内发布。

当前解决此问题的常用方法是为每个客户创建一个集群，这是安全的并提供租户所需的一切，但这很难管理且非常昂贵。

[**Capsule**](https://github.com/clastix/capsule) 是一种为单个集群中的多个租户提供原生 Kubernetes 支持的工具。使用 Capsule，你可以为所有租户拥有一个集群。Capsule 将为租户提供 “几乎” 原生体验（有一些小限制），他们将能够创建多个命名空间并使用集群，因为它们完全可用，隐藏了集群实际上是共享的事实。

![Capsule 架构](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263070902872.jpg)

在单个集群中，Capsule Controller 在称为 **Tenant** 的轻量级 Kubernetes 抽象中聚合多个命名空间，这是一组 Kubernetes 命名空间。在每个租户内，用户可以自由创建他们的命名空间并共享所有分配的资源，而策略引擎则使不同的租户彼此隔离。

租户级别定义的网络和安全策略、资源配额、限制范围、RBAC 和其他策略由租户中的所有命名空间自动继承，类似于分层命名空间。然后用户可以自由地自治操作他们的租户，而无需集群管理员的干预。 Capsule 是 GitOps 就绪的，因为它是声明性的，并且所有配置都可以存储在 Git 中。

## vCluster

[vCluster](https://www.vcluster.com/) 在多租户方面更进了一步，它在 Kubernetes 集群内提供了虚拟集群。每个集群都在一个常规命名空间上运行，并且是完全隔离的。虚拟集群有自己的 API 服务器和独立的数据存储，所以你在 vCluster 中创建的每个 Kubernetes 对象都只存在于 vcluster 内部。此外，您可以将 kube 上下文与虚拟集群一起使用，以像使用常规集群一样使用它们。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263071651487.jpg)

只要您可以在单个命名空间内创建部署，您就可以创建虚拟集群并成为该虚拟集群的管理员，租户可以创建命名空间、安装 CRD、配置权限等等。

**vCluster** 使用 [**k3s**](https://k3s.io/) 作为其 API 服务器，使虚拟集群超轻量级且经济高效；由于 k3s 集群 100% 合规，虚拟集群也 100% 合规。**vClusters** 是超轻量级的（1 个 pod），消耗很少的资源并且可以在任何 Kubernetes 集群上运行，而无需对底层集群进行特权访问。与 **Capsule** 相比，它确实使用了更多的资源，但它提供了更多的灵活性，因为多租户只是用例之一。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/15/16263072079021.jpg)

## 其他工具

-   [kube-burner](https://github.com/cloud-bulldozer/kube-burner) 用于**压力测试**。它提供指标和警报。
-   混沌工程的 [Litmus](https://github.com/litmuschaos/litmus)。
-   [kubewatch](https://github.com/bitnami-labs/kubewatch) 用于监控，但主要关注基于 Kubernetes 事件（如资源创建或删除）的推送通知。它可以与 Slack 等许多工具集成。
-   [BotKube](https://www.botkube.io/) 是一个消息机器人，用于监控和调试 Kubernetes 集群。与 kubewatch 类似，但更新并具有更多功能。
-   [Mizu](https://getmizu.io/) 是一个 API 流量查看器和调试器。
-   [kube-fledged](https://github.com/senthilrch/kube-fledged) 是一个 Kubernetes 插件，用于直接在 Kubernetes 集群的工作节点上创建和管理容器镜像的缓存。因此，**应用程序 pod 几乎立即启动**，因为不需要从注册表中提取图像。

## 结论

在本文中，我们回顾了我最喜欢的 Kubernetes 工具。我专注于可以合并到任何 Kubernetes 发行版中的开源项目。我没有涵盖诸如 [OpenShift](https://www.openshift.com/) 或 Cloud Providers Add-Ons 之类的商业解决方案，因为我想让它保持通用性，但我鼓励您探索如果您在云上运行 Kubernetes 或使用商业工具，您的云提供商可以为您提供什么。

我的目标是向您展示您可以在 **Kubernetes** 中完成您在本地所做的一切。我还更多地关注鲜为人知的工具，我认为这些工具可能具有很大的潜力，例如 [Crossplane](https://crossplane.io/)、[Argo Rollouts](https://kubevela.io/)或 [Kubevela](https://kubevela.io/)。我更感兴趣的工具是 [vCluster](https://www.vcluster.com/)、[Crossplane](https://crossplane.io/) 和 ArgoCD/Workflows。