## **TL;DR**

### **Draft**

-   向 K8S 集群部署代码（自动“构建-推送-部署”）。
-   使用 Draft 打包支持的语言 的代码可以不编写 Dockerfile 或者 K8S 元数据文件直接进行部署。
-   需要 draft 以及 helm 客户端，集群要部署 tiller，本地 Docker，Docker 仓库。

### **Gitkube**

-   向 K8S 集群部署代码（自动“构建-推送-部署”）。
-   Git 推送触发部署，本机无依赖。
-   Git 仓库中需要提供 Dockerfile 以及 K8S 元数据文件，集群中需部署 gitkube。

### **Helm**

-   在 K8S 集群上对 Chart（其中包含一个应用的所有 K8S 资源定义文件）进行部署和管理。
-   提供了很多通用应用（例如 [MySQL](https://cloud.tencent.com/product/cdb?from=10680)、Mediawiki 等）的 Chart。
-   客户端需要 Helm，服务端需要 Tiller，Chart 定义可以在本地也可以在仓库中保存。

### **Ksonnet**

-   在 jsonnet 上定义 K8S 元数据文件，然后进行部署。
-   可以对通用模式（例如 Deployment + Service）和应用栈（例如 [Redis](https://cloud.tencent.com/product/crs?from=10680)）进行复用。
-   需要 jsonnet 知识，安装 ksonnet 客户端。

### **Metaparticle**

-   使用 Metaparticle 支持的语言 编写代码，然后部署到 K8S 集群（自动“构建-推送-部署”）
-   在应用的代码中直接定义容器化和 K8S 相关内容，傻瓜化的编写过程，无需编写 Dockerfile 或者 Yaml。
-   需要本地 Docker 部署，需要相关语言的库。

### **Skaffold**

-   向 K8S 集群部署代码（自动“构建-推送-部署”）。
-   监控源代码变更，变更发生后就会触发“构建-推送-部署”过程，Pipeline 可配置。
-   需要 Skaffold 客户端、Dockerfile、K8S 元数据文件、Skaffold 元数据文件，本地 Docker 以及私库。

___

下面进入一点细节。

当今的 Kubernetes 炙手可热，用户们寻求更多的方式和流程来进行 Kubernetes 集群上的应用部署。`kubectl` 已经成为底层工具，用户需要更易用的流程。Draft、Gitkube、Helm、Ksonnet、MetaParticle 以及 Skaffold 都是用来帮助开发人员在 Kubernetes 上进行应用构建和部署的工具。

Draft、Gitkube 和 Skaffold 减轻了开发人员的负担，在构建应用的过程中，能够更快的在 Kubernetes 上运行起来。Helm 和 Ksonnet 提供了定义应用、更新版本、选择不同集群等功能，在应用构建完成，进入发布就绪状态之后，这两个工具可以提高部署能力。Metaparticle 是比较独特的一个，他把包含 yaml、dockerfile 这些东西集成到业务代码之中。

所以用户自身的用例中如何进行选择？

## **正文**

### **Draft**

> 在任何 Kubernetes 集群上简化应用的开发和部署。

顾名思义，Draft 让面向 Kubernetes 的应用开发变得简单。官方宣称，对于运行在 Kubernetes 上的应用，Draft 这一工具是帮助开发过程而非部署的。Draft 文档中推荐使用 Helm 进行应用部署。

他的目标是：开发人员还在开发调试之中的本地的代码，不经提交到版本控制系统，直接运行到 Kubernetes 集群上。开发人员对 Draft 发布的应用变更满意之后，才提交给版本控制系统。

Draft 不是用来在生产环境上进行部署的，他的用意就是在于快速推进面向 Kubernetes 环境的开发过程。他内部使用 Helm 来进行变更，因此他和 Helm 的集成是非常紧密的。

#### **架构**

如上图所示，`Draft` 客户端是一个关键组件。它感知代码的变化，然后从 `Repo` 中获取对应的 `Pack`。`Pack` 是一个 Dockerfile 和 Helm chart 的合体，他们一起定义了应用的运行环境。`Pack` 定义之后保存在 `Repo` 中。用户可以定义自己的 `Pack` 和 `Repo`，这两个对象可以保存在本地，也可以在 Git 仓库之中。

只要有对应的 `Pack`，任何一个包含源码的目录都可以进行部署。使用 `draft create` 处理目录之后，会在目录中添加 Dockerfile、Helm chart 以及 draft.toml 文件，`draft up` 能够构建 Docker 镜像，推送到私库，然后使用 Helm Chart 部署应用。每次代码变更之后，再次执行这一命令，就会产生一个新的部署。

`draft connect` 命令能够进行端口转发，以此在本地获取容器的日志。他还能够和 `nginx-ingress` 集成，为上面部署的应用提供域名。

#### **从 0 到 Kubernetes**

下面是一个用 Draft 把 Python 应用运行到 K8S 集群上的步骤。可以从官方文档获得更详细的指导。

-   先决条件 $ helm init $ draft init $ draft config set registry docker.io/myusername $ git clone https://github.com/Azure/draft $ cd draft/examples/example-python $ draft create $ draft up ## 代码修改 $ draft up
    -   Kubernetes 集群（包括 kubectl）
    -   Helm 客户端
    -   Draft 客户端
    -   Docker
    -   Docker 镜像库

#### **用例**

-   开发运行在 Kubernetes 上的应用。
-   用于在提交到版本控制之前的“内部流程”。
-   预 CI：应用完成 Draft 过程之后，可以由 CI/CD 接管。
-   不应该用在生产环境部署环节。

### **Gitkube**

> 使用 git push 构建 Docker 镜像并在 Kubernetes 上进行部署。

Gitkube 是一个用来构建 Docker 镜像并向 Kubernetes 上部署的工具，他的起点就是 `git push`，不像 Draft，他不需要客户端，只需在集群上独立运行。

任何带有 Dockerfile 的代码仓库，都可以使用 gitkube 进行部署。Gitkube 安装和部署在集群之上，开发人员可以获取一个包含 git URL 的 CRD。开发人员推送到仓库的代码，会触发集群一端的 Docker Build 以及 Kubectl 发布流程。可以使用 kubectl 或 helm 等类似工具给应用创建应用元数据。

Gitkube 的重点是即插即用的安装过程，以及沿用既有的知名工具（git 以及 kubectl）。对需要部署的仓库没有什么假设。Docker build 的上下文以及 Dockerfile 所在路径，都可以进行配置。Git 连接认证是通过 SSH 公钥进行的。任何时候代码发生变更、提交和推送，都会触发后面的构建和部署过程。

#### **架构**

集群侧有三个组件，一个远程 CRD 可以定义针对一个远端 URL 发生 Push 的时候如何应对，`gitkubed` 构建 Docker 镜像并更新部署，`gitkube-controller` 会监控 CRD，随变化更新 `gitkubed`。

在集群上创建这些对象之后，开发者就可以使用 kubectl 来创建应用的定义了。创建一个 `remote` 对象，告诉 gitkube，当 git push 发生时该做什么。Gitkube 把远程 url 写回到 `remote` 对象的状态字段中。

#### **从 0 到 Kubernetes**

-   先决条件
    -   Kubernetes 集群（包括 kubectl）。
    -   git。
    -   集群上安装好 gitkube （`kubectl create`）。

下面是将应用提交到 Kubernetes 的步骤，也包含了 gitkube 的安装过程。

> 已经过时

```
$ git clone https://github.com/hasura/gitkube-example
$ cd gitkube-example
$ kubectl create -f k8s.yaml
$ cat ~/.ssh/id_rsa.pub | awk '$0="  - "$0' >> "remote.yaml"
$ kubectl create -f remote.yaml
$ kubectl get remote example -o json | jq -r '.status.remoteUrl'
$ git remote add example [remoteUrl]
$ git push example master
## 编辑代码
## 提交和推送
```

#### **用例**

-   使用 Git 进行简单的部署，无需 Docker Build。
-   在 Kubernetes 上开发应用。
-   开发过程中，WIP 分支可以多次提交，迅速反馈。

### **Helm**

> Kubernetes 的包管理系统。

Helm 使用一种称为 `Chart` 的形式，来管理 Kubernetes 上的应用。Helm 为应用创建 YAML 并进行版本化操作，这样可以对包含 Deployment 在内的所有对象进行回滚。Chart 可以包含 Deployment、Service 以及 Configmap 等。Chart 的模板允许用户方便的修改部署细节，另外还支持带有依赖关系的复杂应用。

Helm 的主要目标是在生产环境中部署和管理应用程序。对比 Draft 和 Gitkube，Helm 不是用来开发的，而是用来部署的。另外现在有大量的预构建 `Chart` 可以供 Helm 使用。

#### **架构**

首先看看 Chart。我们之前说过，Chart 之中包含一系列的信息，这些信息是部署应用到 Kubernetes 中的必要条件。其中可能包含 Deployment、Service、Configmap、Secret 以及 Ingress 等。所有的定义都是以 Yaml 文件模板的形式出现，另外还包含嵌套的依赖 Chart。Chart 可以在 Chart 仓库中发布。

Helm 有两个主要组件，分别是 Helm 客户端和 Tiller 服务器。客户端用于管理 Chart 和仓库，并且和 Tiller 服务器进行通信，来完成对 Chart 的部署和管理。

Tiller 组件运行在集群上，和 Kubernetes API 服务器打交道，进行对象的实际操作。

Helm 不处理源码，用户需要使用 CI/CD 系统来构建镜像，然后用 Helm 来部署合适的镜像。

#### **从 0 到 Kubernetes**

-   先决条件
    -   Kubernetes 集群
    -   Helm 客户端

接下来是一个在 Kubernetes 集群上使用 Helm 部署 Wordpress 博客的例子：

```
$ helm init
$ helm repo update
$ helm install stable/wordpress
## 更新版本
$ helm upgrade [release-name] [chart-name]
```

#### **用例**

-   打包：包含多个 Kubernetes 对象的复杂应用可以集中在一起。
-   可复用的 Chart 仓库。
-   可以轻易部署在多个环境上。
-   可嵌套的结构，能够解决依赖关系。
-   参数化的模板。
-   容易复用。
-   持续交付的最后一公里。
-   只能部署已经构建完成的镜像。
-   具备生命周期管理能力，可以管理多个 Kubernetes 对象的升级和回滚。

### **Ksonnet**

> 一个支持客户按操作的框架，提供可扩展的 Kubernetes 配置。

Ksonnet 是为 Kubernetes 定义应用配置的另一种方法。它并没有使用 Kubernetes 世界中常用的 YAML 语言，改用一种称为 Jsonnet 的 JSON 模板语言。Ksonnet 客户端最终会渲染出 YAML 文件并提交给集群。

这一系统的主要功能就是定义可复用的组件，并利用该工具渐进式的进行程序构建。

#### **架构**

基础的构建单位被称为 `part`，`part` 可以协作构成 `prototype`。一个 `prototype` 配合参数之后，就成为了一个 `component`，`component` 可以聚合在一起，成为一个 `application`。`application` 可以部署到多个 `environment` 之中。

最基础的流程就是使用 `ks init` 命令创建一个应用目录，使用 `ks generate` 生成（或者也可以自行编写）`component` 的元数据文件，使用 `ks apply <env>` 命令可以把应用部署到集群/环境之中。可以用 `ks env` 命令来管理不同的环境。

简而言之，Ksonnet 帮助用户定义和管理应用，他把应用视作一系列使用 Jsonnet 的组件进行管理，并部署在不同的 Kubernetes 集群上。

跟 Helm 类似，Ksonnet 不和源码发生关系，他是一个使用 Jsonnet 为 Kubernetes 定义应用的工具。

#### **从 0 到 Kubernetes**

-   先决条件
    -   Kubernetes 集群
    -   ksonnet 客户端

接下来是一个留言板例子：

```
$ ks init
$ ks generate deployed-service guestbook-ui \
    --image gcr.io/heptio-images/ks-guestbook-demo:0.1 \
    --type ClusterIP
$ ks apply default
## 变更
$ ks apply default
```

#### **用例**

-   使用 Jsonnet 编写配置很有弹性。
-   打包：复杂配置可以用匹配组件的方式集成起来。
-   可复用的组件和原型库：避免重复。
-   方便的多环境部署。
-   CD 的最后一步。

### **Metaparticle**

> 为容器和 Kubernetes 而生的云原生标准库。

Metaparticle 将自己定位于云原生应用的标准库，他内置了经过验证的分布式系统模式，而开发人员可以用习惯的编程语言通过原语的方式方便的采用这些先进模式。

他提供了简易的语言接口，帮助用户构建可以容器化并部署到 Kubernetes 上的应用，这些应用会直接兼容[负载均衡](https://cloud.tencent.com/product/clb?from=10680)等基础设施。无需自行编写 Dockerfile 或者 Kubernetes 元数据文件，所有相关内容都在代码中的用原语来体现。

例如一个 Python Web 应用，可以给 main 函数加入一个叫做 `containerize` 的 Decorator（从 metaparticle 中 import）。当执行这段 Python 代码的时候，会构建 Docker 镜像并部署到 Decorator 参数中提到的 Kubernetes 集群上。缺省集群定义来自 kubectl 上下文。所以切换环境就和切换当前上下文是等价的。

在 NodeJS、Java 以及 .NET 上也提供了类似的原语。另外还正在开发更多的语言支持。

#### **架构**

各种语言的 `metaparticle` 库都包含所需的原语，绑定了构建 Docker 镜像、推送到私库、创建 Kubenretes yaml 文件并在集群上部署的代码。

Metaparticle 包中内置了各种语言用来构建容器的支持。而 Metaparticle Sync 则包含了在不同机器上运行的不同容器进行同步的能力。

目前支持的语言包括：JavaScript/NodeJS、Python、Java 以及 .NET。

#### **从 0 到 Kubernetes**

-   先决条件
    -   Kubernetes 集群。
    -   特定语言的 Metaparticle 库。
    -   Docker。
    -   Docker 私库。

一个只包含相关内容的 Python 例子，可以使用这些代码构建 Docker 镜像，并在 Kubernetes 上进行部署。

```
@containerize(
   'docker.io/your-docker-user-goes-here',
   options={        'ports': [8080],        'replicas': 4,        'runner': 'metaparticle',        'name': 'my-image',        'publish': True
   })def main():
   Handler = MyHandler
   httpd = SocketServer.TCPServer(("", port), Handler)
   httpd.serve_forever()
```

#### **用例**

-   只想开发应用，不想担心 Kubernetes YAML 或者 Dockerfile。
-   不想掌握多种工具和文件格式，又想搭上容器和 Kubernetes 快车的开发人员。
-   快速开发可复制可负载均衡的服务。
-   在多个分布式副本之间进行同步，例如加锁、或者选举操作。
-   简单开发云原生模式的应用，例如分片系统。

### **Skaffold**

> 简单可重复的 Kubernetes 开发。

Skaffold 能够处理构建镜像、推送镜像以及在 Kubernetes 上进行部署。跟 Gitkube 类似，任何包含 Dockerfile 的目录都可以用 Skaffold 部署到 kubernetes 集群上。

Skaffold 会在本地构建 Docker 镜像，推送到私库，然后使用 `skaffold` 客户端进行部署。他还会监测目录，如此一来，目录中的代码一旦发生变化，就会触发重新构建和部署。这个过程还会从容器中获取日志。

可以使用 YAML 文件来构建、推送、部署的 Pipeline，所以开发者可以混合使用合适的工具，例如 Docker build 和 Google Container Builder，Kubectl 和 Helm 等。

#### **架构**

Skaffold 客户端做了所有的工作。他会查找一个叫做 `skaffold.yaml` 的文件，其中包含了必须完成的任务。一个典型的例子就是在 `skaffold dev` 运行的目录中查找 Dockerfile 构建 Docker 镜像，并使用 sha256 进行标记，推送镜像，把镜像设置到 Kubernetes 元数据文件之中，最后发布到集群上。这一系列动作会被目录中的变更所触发。来自部署容器的日志会出现在同一个 Watch 窗口中。

Skaffold 和 Draft 和 Gitkube 很像，但是更具弹性，如上图所示，他能管理不同的“构建-推送-部署”流程。

#### **从 0 到 Kubernetes**

-   先决条件
    -   Kubernetes 集群
    -   Skaffold 客户端
    -   Docker
    -   Docker 镜像库

下面的步骤，部署一个 Go 编写的 Hello World 应用：

```
$ git clone https://github.com/GoogleCloudPlatform/skaffold
$ cd examples/getting-started
## 编辑 skaffold.yaml，加入 Docker 仓库
$ skaffold dev
## 打开新终端: 编辑代码
```

#### **用例**

-   方便部署。
-   迭代构建——持续的构建-发布流程。
-   为 Kubernetes 开发应用。
-   在 CICD 流程中定义“构建-推送-部署”流程。

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_fdf2bb3e033a)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。