[Acorn](https://acorn.io/)是Rancher创始人推出的一个新的应用程序部署框架，它非常接近我对运行在Kubernetes上的开发环境的期望。

长期以来，我一直主张一种简化的方法来开发和部署以Kubernetes为目标的应用程序。我之前[强调](https://thenewstack.io/azure-container-apps-do-we-need-yet-another-managed-container-service/)需要一个可移植的、透明的、开源的应用程序层，该应用程序层将始终部署在开发人员笔记本电脑中的Minikube集群或在公共云中配置的大型多节点集群内运行。

作为高人气Kubernetes发行版K3s缔造者Darren Shepherd及其团队的开发成果，Acorn从Rancher身上继承了不少颇受云原生社区好评的设计原则。这是一套开源、简单、轻量化且可移植的框架，用于在Kubernetes上部署和扩展微服务。

使用Acorn的开发人员和运维人员无需了解Kubernetes的具体细节。如果他们了解Volumes、Secrets、ConfigMaps和Ingress等内部结构，那将非常棒。Acorn用自己的类JSON领域特定语言（DSL）抽象了Kubernetes的复杂性，以描述基于微服务设计模式的现代应用程序。

像Cloud Foundry这样的PaaS的目标是将代码推送到运行时并使用URL。Acorn正是专注于接受源代码或容器镜像并发布端点的工作流程。在后台，它完成了与Kubernetes API协商以创建资源和连接它们所需的管道的繁重工作。

尽管Amazon Web Services的[App Runner](https://aws.amazon.com/apprunner/)、[Azure Container Apps](https://azure.microsoft.com/en-us/services/container-apps/)和[Google Cloud Run](https://cloud.google.com/run)等努力为部署容器化工作负载带来类似PaaS的体验，但它们仅限于公有云环境并且不可移植。Acorn是少数几个可以从开发人员笔记本电脑上运行的[Kind](https://kind.sigs.k8s.io/)集群无缝扩展到云中的多节点集群的框架之一。

本文分析了Acorn的架构，并深入了解Acorn部署如何转换为Kubernetes对象。

让我们详细了解一下。

### 在Minikube中设置环境

在Mac上安装[Minikube](https://minikube.sigs.k8s.io/docs/)并在其上启用Nginx Ingress。Ingress是Acorn最重要的先决条件之一。

```
minikube addons enable ingress

```

[![1.png](http://dockone.io/uploads/article/20220829/e4d006cc93aacde112553a2fc8b5d423.png "1.png")](http://dockone.io/uploads/article/20220829/e4d006cc93aacde112553a2fc8b5d423.png)

使用Homebrew安装Acorn CLI，并检查其版本以确保已安装。

```
brew install acorn-io/cli/acorn

```

[![2.png](http://dockone.io/uploads/article/20220829/1ad6e9d7cd9878a3c8df1cc0056107ef.png "2.png")](http://dockone.io/uploads/article/20220829/1ad6e9d7cd9878a3c8df1cc0056107ef.png)

我们现在准备在Minikube中安装Acorn。运行`acorn init`以配置Minikube。

[![3.png](http://dockone.io/uploads/article/20220829/ab65bc787a6ccbff0a19c9cfde9e48df.png "3.png")](http://dockone.io/uploads/article/20220829/ab65bc787a6ccbff0a19c9cfde9e48df.png)

在Kubernetes集群中安装Acorn会创建一组资源来处理应用程序的构建时间和运行时要求。让我们从namespaces开始。

[![4.png](http://dockone.io/uploads/article/20220829/76cbcf285c62e84e257a1705a6685833.png "4.png")](http://dockone.io/uploads/article/20220829/76cbcf285c62e84e257a1705a6685833.png)

`acorn-system` namespaces包含API和控制器，它们是运行时环境的组件。在开发环境中运行时，相同的namespaces可以选择运行镜像构建器和镜像注册表。另一个namespaces `acorn`是为应用程序保留的，我们将在下一节中探讨。

[![5.png](http://dockone.io/uploads/article/20220829/a4b87282bc586661bd0b7ba33774bd86.png "5.png")](http://dockone.io/uploads/article/20220829/a4b87282bc586661bd0b7ba33774bd86.png)

安装程序仅在集群中创建一个自定义资源定义（CRD）。CRD `AppInstance.internal.acorn.io`映射到集群内运行的Acorn应用程序上。

[![6.png](http://dockone.io/uploads/article/20220829/23dbf24819f59e5beeda6e3169b7660d.png "6.png")](http://dockone.io/uploads/article/20220829/23dbf24819f59e5beeda6e3169b7660d.png)

Acorn API服务器通过聚合与Kubernetes API服务器关联。Acorn CLI与API服务器`api.acorn.io`对话。由于Acorn利用Kubernetes API聚合，CLI只需要Acorn API组的RBAC权限。

[![7.png](http://dockone.io/uploads/article/20220829/e2980911e900b8f87f76d583b2abea32.png "7.png")](http://dockone.io/uploads/article/20220829/e2980911e900b8f87f76d583b2abea32.png)

API服务器将入站请求传递给Acorn控制器，该控制器将应用程序定义转换为适当的Kubernetes资源，例如Deployments、ConfigMaps、Secrets和Volumes。控制器负责通过创建和终止下游Kubernetes资源来管理Acorn应用程序的生命周期。

[![8.png](http://dockone.io/uploads/article/20220829/05942e01a4afc70c0fc78e89fc6d17c8.png "8.png")](http://dockone.io/uploads/article/20220829/05942e01a4afc70c0fc78e89fc6d17c8.png)

  

### 部署Acorn应用程序

让我们从基于Nginx镜像的单个Web服务器创建最简单的Acorn应用程序开始。

在一个空目录中创建一个`Acornfile`包含以下内容的目录：

```
containers: {
web: {
image: "nginx"
ports: {
publish: "80/http"
}
}
} 

```

该定义是不言自明的。我们基于Docker注册表中的Nginx镜像启动一个名为“web”的容器，并使其在端口80上可用。

使用以下命令运行Acornfile：

```
acorn run  .

```

[![9.png](http://dockone.io/uploads/article/20220829/6161637d4cd1b1f1c22a62e06662e100.png "9.png")](http://dockone.io/uploads/article/20220829/6161637d4cd1b1f1c22a62e06662e100.png)

由于我们没有为应用程序提供名称，因此Acorn为应用程序指定了一个随机名称，即proud-silence。

当我们调用run命令时，Acorn创建了一个OCI清单并将其推送到在namespace内运行的内部注册表服务`acorn-system`上。也可以为这些OCI工件使用外部注册表。

[![10.png](http://dockone.io/uploads/article/20220829/1db2abe22b3045af61dfa0e6bb9f19b2.png "10.png")](http://dockone.io/uploads/article/20220829/1db2abe22b3045af61dfa0e6bb9f19b2.png)

让我们通过运行以下命令获取访问应用程序的URL：

```
acorn apps

```

[![11.png](http://dockone.io/uploads/article/20220829/4e843d6cd27eaa238e7b86481afd01d1.png "11.png")](http://dockone.io/uploads/article/20220829/4e843d6cd27eaa238e7b86481afd01d1.png)

让我们访问Web服务器来测试应用程序。

```
curl  -H  "host: web.proud-silence.local.on-acorn.io"  `minikube ip`

```

现在，让我们看看这个简单的应用程序对我们的Kubernetes集群做了什么。

首先，我们注意到有一个新的namespace作为应用程序的边界。

[![12.png](http://dockone.io/uploads/article/20220829/6eee1429d74cef39bf2437a3c10da809.png "12.png")](http://dockone.io/uploads/article/20220829/6eee1429d74cef39bf2437a3c10da809.png)

让我们检查一下这个namespace。正如预期的那样，app run命令已经创建了一个Kubernetes Deployment、ReplicaSet、Pod和一个集群IP服务。

[![13.png](http://dockone.io/uploads/article/20220829/312e7f868a3a1475f804aa50edfe62cf.png "13.png")](http://dockone.io/uploads/article/20220829/312e7f868a3a1475f804aa50edfe62cf.png)

集群IP服务通过一个Ingress资源暴露给外界，我们稍后会探讨。

当我们检查`acorn` namespace时，我们会找到CRD的实例AppInstance。

```
kubectl get appinstances -n acorn

```

[![14.png](http://dockone.io/uploads/article/20220829/dc7d84da6637826a45d47308d695811f.png "14.png")](http://dockone.io/uploads/article/20220829/dc7d84da6637826a45d47308d695811f.png)

重温Ingress的想法来暴露Web应用程序，让我们看看我们是否可以在应用程序namespace中找到一个Ingress资源。

```
kubectl get ingress  -n  proud-silence-f6db18f8-c38

```

[![15.png](http://dockone.io/uploads/article/20220829/154e1a039d3fa4794c19e67638b1ab9c.png "15.png")](http://dockone.io/uploads/article/20220829/154e1a039d3fa4794c19e67638b1ab9c.png)

每个“发布”端口的Acorn应用程序都会在Kubernetes中创建一个关联的入口对象。

由于应用程序按预期运行，它现在可以被标记并推送到外部注册表。管理工作负载的运营团队可以将其部署到生产集群，而无需了解应用程序的内部结构。

Acorn让我着迷的是它的简单性和便携性。它将Kubernetes视为部署、扩展和运行应用程序的理想运行时环境，无需任何假设。它不会篡改集群并部署最少的资源集，足以运行微服务。从某种意义上说，它是真正可移植的，当我们从开发集群上下文切换到另一个并部署应用程序时，它会被推送到生产集群。

Acorn深受Docker的影响，并遵循一些通用的模式来运行多容器应用程序。与Cloud Foundry一样，它也支持绑定现有服务，例如部署在其他应用程序中的数据库和缓存。

一旦Acorn支持直接从包含Acornfile的Git存储库进行部署的能力，DevOps就可以非常轻松地管理基于微服务的应用程序。

在本系列的下一部分中，我将展示一个基于Acorn的微服务应用程序的真实示例，该应用程序在开发环境和生产集群中运行。敬请关注。

**原文链接：[Acorn, a Lightweight, Portable PaaS for Kubernetes](https://thenewstack.io/acorn-a-lightweight-portable-paas-for-kubernetes/)（翻译：李鹏）**