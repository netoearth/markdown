![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-1.png)

[本文永久链接](https://tonybai.com/2022/08/15/developing-kubernetes-operators-in-go-part1) – https://tonybai.com/2022/08/15/developing-kubernetes-operators-in-go-part1

> 注：文章首图基于《Kubernetes Operators Explained》修改

[几年前，我还称Kubernetes为服务编排和容器调度领域的事实标准](https://tonybai.com/2018/10/17/imooc-course-kubernetes-practice-go-online/)，如今K8s已经是这个领域的“霸主”，地位无可撼动。不过，虽然Kubernetes发展演化到今天已经变得非常复杂，但是Kubernetes最初的数据模型、应用模式与扩展方式却依然有效。并且像[Operator这样的应用模式和扩展方式](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)日益受到开发者与运维者的欢迎。

我们的平台内部存在有状态(stateful)的后端服务，对有状态的服务的部署和运维是k8s operator的**拿手好戏**，是时候来研究一下operator了。

### 一. Operator的优点

[kubernetes operator的概念最初来自CoreOS](https://web.archive.org/web/20170129131616/https://coreos.com/blog/introducing-operators.html) – 一家被红帽(redhat)收购的容器技术公司。

CoreOS在引入Operator概念的同时，也给出了Operator的第一批参考实现：[etcd operator](https://web.archive.org/web/20170224100544/https://coreos.com/blog/introducing-the-etcd-operator.html)和[prometheus operator](https://web.archive.org/web/20170224101137/https://coreos.com/blog/the-prometheus-operator.html)。

> 注：[etcd](https://etcd.io/)于2013年由CoreOS以开源形式发布；[prometheus](https://prometheus.io/)作为首款面向云原生服务的时序数据存储与监控系统，由SoundCloud公司于2012年以开源的形式发布。

下面是CoreOS对Operator这一概念的诠释：**Operator在软件中代表了人类的运维操作知识，通过它可以可靠地管理一个应用程序**。

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-4.png)  

图：CoreOS对operator的诠释(截图来自CoreOS官方博客归档)

Operator出现的初衷就是用来解放运维人员的，如今Operator也越来越受到云原生运维开发人员的青睐。

那么operator好处究竟在哪里呢？下面示意图对使用Operator和不使用Operator进行了对比：

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-2.png)

通过这张图，即便对operator不甚了解，你也能大致感受到operator的优点吧。

我们看到在使用operator的情况下，对有状态应用的伸缩操作(这里以伸缩操作为例，也可以是其他诸如版本升级等对于有状态应用来说的“复杂”操作)，运维人员仅需一个简单的命令即可，运维人员也无需知道k8s内部对有状态应用的伸缩操作的原理是什么。

在没有使用operator的情况下，运维人员需要对有状态应用的伸缩的操作步骤有深刻的认知，并按顺序逐个执行一个命令序列中的命令并检查命令响应，遇到失败的情况时还需要进行重试，直到伸缩成功。

我们看到operator就好比一个内置于k8s中的经验丰富运维人员，时刻监控目标对象的状态，把复杂性留给自己，给运维人员一个简洁的交互接口，同时operator也能降低运维人员因个人原因导致的操作失误的概率。

不过，operator虽好，但开发门槛却不低。开发门槛至少体现在如下几个方面：

-   对operator概念的理解是基于对k8s的理解的基础之上的，而k8s自从2014年开源以来，变的日益复杂，理解起来需要一定时间投入；
-   从头手撸operator很verbose，几乎无人这么做，大多数开发者都会去学习相应的开发框架与工具，比如：[kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)、[operator framework sdk](https://sdk.operatorframework.io/)等；
-   operator的能力也有高低之分，operator framework就提出了一个包含**五个等级的operator能力模型(CAPABILITY MODEL)**，见下图。使用Go开发高能力等级的operator需要对[client-go](https://github.com/kubernetes/client-go)这个kubernetes官方go client库中的API有深入的了解。

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-3.png)  

图：operator能力模型(截图来自operator framework官网)

当然在这些门槛当中，对operator概念的理解既是基础也是前提，而理解operator的前提又是对kubernetes的诸多概念要有深入理解，尤其是resource、resource type、API、controller以及它们之间的关系。接下来我们就来快速介绍一下这些概念。

### 二. Kubernetes resource、resource type、API和controller介绍

Kubernetes发展到今天，其本质已经显现：

-   Kubernetes就是一个“数据库”(数据实际持久存储在etcd中)；
-   其API就是“sql语句”；
-   API设计采用基于resource的Restful风格, resource type是API的端点(endpoint)；
-   每一类resource(即Resource Type)是一张“表”，Resource Type的spec对应“表结构”信息(schema)；
-   每张“表”里的一行记录就是一个resource，即该表对应的Resource Type的一个实例(instance)；
-   Kubernetes这个“数据库”内置了很多“表”，比如Pod、Deployment、DaemonSet、ReplicaSet等；

下面是一个Kubernetes API与resource关系的示意图：

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-5.png)

我们看到resource type有两类，一类的namespace相关的(namespace-scoped)，我们通过下面形式的API操作这类resource type的实例：

```
VERB /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE - 操作某特定namespace下面的resouce type中的resource实例集合
VERB /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE/NAME - 操作某特定namespace下面的resource type中的某个具体的resource实例
```

另外一类则是namespace无关，即cluster范围(cluster-scoped)的，我们通过下面形式的API对这类resource type的实例进行操作：

```
VERB /apis/GROUP/VERSION/RESOURCETYPE - 操作resouce type中的resource实例集合
VERB /apis/GROUP/VERSION/RESOURCETYPE/NAME - 操作resource type中的某个具体的resource实例
```

我们知道Kubernetes并非真的只是一个“数据库”，它是服务编排和容器调度的平台标准，它的基本调度单元是Pod(也是一个resource type)，即一组容器的集合。那么Pod又是如何被创建、更新和删除的呢？这就离不开控制器(controller)了。**每一类resource type都有自己对应的控制器(controller)**。以pod这个resource type为例，它的controller为ReplicasSet的实例。

控制器的运行逻辑如下图所示：

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-6.png)  

图：控制器运行逻辑(引自《Kubernetes Operators Explained》一文)

控制器一旦启动，将尝试获得resource的当前状态(current state)，并与存储在k8s中的resource的期望状态（desired state，即spec)做比对，如果不一致，controller就会调用相应API进行调整，尽力使得current state与期望状态达成一致。这个达成一致的过程被称为**协调(reconciliation)**，协调过程的伪代码逻辑如下：

```
for {
    desired := getDesiredState()
    current := getCurrentState()
    makeChanges(desired, current)
}
```

> 注：k8s中有一个object的概念？那么object是什么呢？它类似于Java Object基类或Ruby中的Object超类。不仅resource type的实例resource是一个(is-a)object，resource type本身也是一个object，它是kubernetes concept的实例。

有了上面对k8s这些概念的初步理解，我们下面就来理解一下Operator究竟是什么！

### 三. Operator模式 = 操作对象(CRD) + 控制逻辑(controller)

如果让运维人员直面这些内置的resource type(如deployment、pod等)，也就是前面“使用operator vs. 不使用operator”对比图中的第二种情况, 运维人员面临的情况将会很复杂，且操作易错。

那么如果不直面内置的resource type，那么我们如何自定义resource type呢, Kubernetes提供了Custom Resource Definition，CRD(在coreos刚提出operator概念的时候，crd的前身是Third Party Resource, TPR)可以用于自定义resource type。

根据前面我们对resource type理解，定义CRD相当于建立新“表”(resource type)，一旦CRD建立，k8s会为我们自动生成对应CRD的API endpoint，我们就可以通过yaml或API来操作这个“表”。我们可以向“表”中“插入”数据，即基于CRD创建Custom Resource(CR)，这就好比我们创建Deployment实例，向Deployment“表”中插入数据一样。

和原生内置的resource type一样，光有存储对象状态的CR还不够，原生resource type有对应controller负责协调(reconciliation)实例的创建、伸缩与删除，CR也需要这样的“协调者”，即我们也需要定义一个controller来负责监听CR状态并管理CR创建、伸缩、删除以及保持期望状态(spec)与当前状态(current state)的一致。这个controller不再是面向原生Resource type的实例，而是**面向CRD的实例CR的controller**。

有了自定义的操作对象类型(CRD)，有了面向操作对象类型实例的controller，我们将其打包为一个概念：“Operator模式”，operator模式中的controller也被称为operator，它是在集群中对CR进行维护操作的主体。

### 四. 使用kubebuilder开发webserver operator

> 假设：此时你的本地开发环境已经具备访问实验用k8s环境的一切配置，通过kubectl工具可以任意操作k8s。

**再深入浅出的概念讲解都不如一次实战对理解概念更有帮助**，下面我们就来开发一个简单的Operator。

前面提过operator开发非常verbose，因此社区提供了开发工具和框架来帮助开发人员简化开发过程，目前主流的包括operator framework sdk和kubebuilder，前者是redhat开源并维护的一套工具，支持使用go、ansible、helm进行operator开发(其中只有go可以开发到能力级别5的operator，其他两种则不行)；而kubebuilder则是kubernetes官方的一个sig(特别兴趣小组)维护的operator开发工具。目前基于operator framework sdk和go进行operator开发时，operator sdk底层使用的也是kubebuilder，所以这里我们就直接使用kubebuilder来开发operator。

按照operator能力模型，我们这个operator差不多处于2级这个层次，我们定义一个Webserver的resource type，它代表的是一个基于nginx的webserver集群，我们的operator支持创建webserver示例(一个nginx集群)，支持nginx集群伸缩，支持集群中nginx的版本升级。

下面我们就用kubebuilder来实现这个operator！

#### 1\. 安装kubebuilder

这里我们采用源码构建方式安装，步骤如下：

```
$git clone git@github.com:kubernetes-sigs/kubebuilder.git
$cd kubebuilder
$make
$cd bin
$./kubebuilder version
Version: main.version{KubeBuilderVersion:"v3.5.0-101-g5c949c2e",
KubernetesVendor:"unknown",
GitCommit:"5c949c2e50ca8eec80d64878b88e1b2ee30bf0bc",
BuildDate:"2022-08-06T09:12:50Z", GoOs:"linux", GoArch:"amd64"}
```

然后将bin/kubebuilder拷贝到你的PATH环境变量中的某个路径下即可。

#### 2\. 创建webserver-operator工程

接下来，我们就可以使用kubebuilder创建webserver-operator工程了：

```
$mkdir webserver-operator
$cd webserver-operator
$kubebuilder init  --repo github.com/bigwhite/webserver-operator --project-name webserver-operator

Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.12.2
go: downloading k8s.io/client-go v0.24.2
go: downloading k8s.io/component-base v0.24.2
Update dependencies:
$ go mod tidy
Next: define a resource with:
kubebuilder create api
```

> 注：–repo指定go.mod中的module root path，你可以定义你自己的module root path。

#### 3\. 创建API，生成初始CRD

Operator包括CRD和controller，这里我们就来建立自己的CRD，即自定义的resource type，也就是API的endpoint，我们使用下面kubebuilder create命令来完成这个步骤：

```
$kubebuilder create api --version v1 --kind WebServer
Create Resource [y/n]
y
Create Controller [y/n]
y
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
api/v1/webserver_types.go
controllers/webserver_controller.go
Update dependencies:
$ go mod tidy
Running make:
$ make generate
mkdir -p /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin
test -s /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen || GOBIN=/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.9.2
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
Next: implement your new API and generate the manifests (e.g. CRDs,CRs) with:
$ make manifests
```

之后，我们执行make manifests来生成最终CRD对应的yaml文件：

```
$make manifests
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
```

此刻，整个工程的目录文件布局如下：

```
$tree -F .
.
├── api/
│   └── v1/
│       ├── groupversion_info.go
│       ├── webserver_types.go
│       └── zz_generated.deepcopy.go
├── bin/
│   └── controller-gen*
├── config/
│   ├── crd/
│   │   ├── bases/
│   │   │   └── my.domain_webservers.yaml
│   │   ├── kustomization.yaml
│   │   ├── kustomizeconfig.yaml
│   │   └── patches/
│   │       ├── cainjection_in_webservers.yaml
│   │       └── webhook_in_webservers.yaml
│   ├── default/
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager/
│   │   ├── controller_manager_config.yaml
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus/
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac/
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── role_binding.yaml
│   │   ├── role.yaml
│   │   ├── service_account.yaml
│   │   ├── webserver_editor_role.yaml
│   │   └── webserver_viewer_role.yaml
│   └── samples/
│       └── _v1_webserver.yaml
├── controllers/
│   ├── suite_test.go
│   └── webserver_controller.go
├── Dockerfile
├── go.mod
├── go.sum
├── hack/
│   └── boilerplate.go.txt
├── main.go
├── Makefile
├── PROJECT
└── README.md

14 directories, 40 files
```

#### 4\. webserver-operator的基本结构

忽略我们此次不关心的诸如leader election、auth\_proxy等，我将这个operator例子的主要部分整理到下面这张图中：

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-7.png)

图中的各个部分就是使用kubebuilder生成的**operator的基本结构**。

webserver operator主要由CRD和controller组成：

-   CRD

图中的左下角的框框就是上面生成的CRD yaml文件：config/crd/bases/my.domain\_webservers.yaml。CRD与api/v1/webserver\_types.go密切相关。我们在api/v1/webserver\_types.go中为CRD定义spec相关字段，之后make manifests命令可以解析webserver\_types.go中的变化并更新CRD的yaml文件。

-   controller

从图的右侧部分可以看出，controller自身就是作为一个deployment部署在k8s集群中运行的，它监视CRD的实例CR的运行状态，并在Reconcile方法中检查预期状态与当前状态是否一致，如果不一致，则执行相关操作。

-   其它

图中左上角是有关controller的权限的设置，controller通过serviceaccount访问k8s API server，通过role.yaml和role\_binding.yaml设置controller的角色和权限。

#### 5\. 为CRD spec添加字段(field)

为了实现Webserver operator的功能目标，我们需要为CRD spec添加一些状态字段。前面说过，CRD与api中的webserver\_types.go文件是同步的，我们只需修改webserver\_types.go文件即可。我们在WebServerSpec结构体中增加Replicas和Image两个字段，它们分别用于表示webserver实例的副本数量以及使用的容器镜像：

```
// api/v1/webserver_types.go

// WebServerSpec defines the desired state of WebServer
type WebServerSpec struct {
    // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
    // Important: Run "make" to regenerate code after modifying this file

    // The number of replicas that the webserver should have
    Replicas int `json:"replicas,omitempty"`

    // The container image of the webserver
    Image string `json:"image,omitempty"`

    // Foo is an example field of WebServer. Edit webserver_types.go to remove/update
    Foo string `json:"foo,omitempty"`
}
```

保存修改后，**执行make manifests**重新生成config/crd/bases/my.domain\_webservers.yaml

```
$cat my.domain_webservers.yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.9.2
  creationTimestamp: null
  name: webservers.my.domain
spec:
  group: my.domain
  names:
    kind: WebServer
    listKind: WebServerList
    plural: webservers
    singular: webserver
  scope: Namespaced
  versions:
  - name: v1
    schema:
      openAPIV3Schema:
        description: WebServer is the Schema for the webservers API
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: WebServerSpec defines the desired state of WebServer
            properties:
              foo:
                description: Foo is an example field of WebServer. Edit webserver_types.go
                  to remove/update
                type: string
              image:
                description: The container image of the webserver
                type: string
              replicas:
                description: The number of replicas that the webserver should have
                type: integer
            type: object
          status:
            description: WebServerStatus defines the observed state of WebServer
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
```

一旦定义完CRD，我们就可以将其安装到k8s中：

```
$make install
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
test -s /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize || { curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash -s -- 3.8.7 /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin; }
{Version:kustomize/v3.8.7 GitCommit:ad092cc7a91c07fdf63a2e4b7f13fa588a39af4f BuildDate:2020-11-11T23:14:14Z GoOs:linux GoArch:amd64}
kustomize installed to /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/webservers.my.domain created
```

检查安装情况：

```
$kubectl get crd|grep webservers
webservers.my.domain                                             2022-08-06T21:55:45Z
```

#### 6\. 修改role.yaml

在开始controller开发之前，我们先来为controller后续的运行“铺平道路”，即设置好相应权限。

我们在controller中会为CRD实例创建对应deployment和service，这样就要求controller有操作deployments和services的权限，这样就需要我们修改role.yaml，增加service account: controller-manager 操作deployments和services的权限：

```
// config/rbac/role.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: manager-role
rules:
- apiGroups:
  - my.domain
  resources:
  - webservers
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - my.domain
  resources:
  - webservers/finalizers
  verbs:
  - update
- apiGroups:
  - my.domain
  resources:
  - webservers/status
  verbs:
  - get
  - patch
  - update
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - apps
  - ""
  resources:
  - services
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```

修改后的role.yaml先放在这里，后续与controller一并部署到k8s上。

#### 7\. 实现controller的Reconcile(协调)逻辑

kubebuilder为我们搭好了controller的代码架子，我们只需要在controllers/webserver\_controller.go中实现WebServerReconciler的Reconcile方法即可。下面是Reconcile的一个简易流程图，结合这幅图理解代码就容易的多了：

![](https://tonybai.com/wp-content/uploads/developing-kubernetes-operators-in-go-part1-8.png)

下面是对应的Reconcile方法的代码：

```
// controllers/webserver_controller.go

func (r *WebServerReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := r.Log.WithValues("Webserver", req.NamespacedName)

    instance := &mydomainv1.WebServer{}
    err := r.Get(ctx, req.NamespacedName, instance)
    if err != nil {
        if errors.IsNotFound(err) {
            // Request object not found, could have been deleted after reconcile request.
            // Return and don't requeue
            log.Info("Webserver resource not found. Ignoring since object must be deleted")
            return ctrl.Result{}, nil
        }

        // Error reading the object - requeue the request.
        log.Error(err, "Failed to get Webserver")
        return ctrl.Result{RequeueAfter: time.Second * 5}, err
    }

    // Check if the webserver deployment already exists, if not, create a new one
    found := &appsv1.Deployment{}
    err = r.Get(ctx, types.NamespacedName{Name: instance.Name, Namespace: instance.Namespace}, found)
    if err != nil && errors.IsNotFound(err) {
        // Define a new deployment
        dep := r.deploymentForWebserver(instance)
        log.Info("Creating a new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
        err = r.Create(ctx, dep)
        if err != nil {
            log.Error(err, "Failed to create new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
            return ctrl.Result{RequeueAfter: time.Second * 5}, err
        }
        // Deployment created successfully - return and requeue
        return ctrl.Result{Requeue: true}, nil
    } else if err != nil {
        log.Error(err, "Failed to get Deployment")
        return ctrl.Result{RequeueAfter: time.Second * 5}, err
    }

    // Ensure the deployment replicas and image are the same as the spec
    var replicas int32 = int32(instance.Spec.Replicas)
    image := instance.Spec.Image

    var needUpd bool
    if *found.Spec.Replicas != replicas {
        log.Info("Deployment spec.replicas change", "from", *found.Spec.Replicas, "to", replicas)
        found.Spec.Replicas = &replicas
        needUpd = true
    }

    if (*found).Spec.Template.Spec.Containers[0].Image != image {
        log.Info("Deployment spec.template.spec.container[0].image change", "from", (*found).Spec.Template.Spec.Containers[0].Image, "to", image)
        found.Spec.Template.Spec.Containers[0].Image = image
        needUpd = true
    }

    if needUpd {
        err = r.Update(ctx, found)
        if err != nil {
            log.Error(err, "Failed to update Deployment", "Deployment.Namespace", found.Namespace, "Deployment.Name", found.Name)
            return ctrl.Result{RequeueAfter: time.Second * 5}, err
        }
        // Spec updated - return and requeue
        return ctrl.Result{Requeue: true}, nil
    }

    // Check if the webserver service already exists, if not, create a new one
    foundService := &corev1.Service{}
    err = r.Get(ctx, types.NamespacedName{Name: instance.Name + "-service", Namespace: instance.Namespace}, foundService)
    if err != nil && errors.IsNotFound(err) {
        // Define a new service
        srv := r.serviceForWebserver(instance)
        log.Info("Creating a new Service", "Service.Namespace", srv.Namespace, "Service.Name", srv.Name)
        err = r.Create(ctx, srv)
        if err != nil {
            log.Error(err, "Failed to create new Servie", "Service.Namespace", srv.Namespace, "Service.Name", srv.Name)
            return ctrl.Result{RequeueAfter: time.Second * 5}, err
        }
        // Service created successfully - return and requeue
        return ctrl.Result{Requeue: true}, nil
    } else if err != nil {
        log.Error(err, "Failed to get Service")
        return ctrl.Result{RequeueAfter: time.Second * 5}, err
    }

    // Tbd: Ensure the service state is the same as the spec, your homework

    // reconcile webserver operator in again 10 seconds
    return ctrl.Result{RequeueAfter: time.Second * 10}, nil
}
```

这里大家可能发现了：**原来CRD的controller最终还是将CR翻译为k8s原生Resource，比如service、deployment等。CR的状态变化(比如这里的replicas、image等)最终都转换成了deployment等原生resource的update操作**，这就是operator的精髓！理解到这一层，operator对大家来说就不再是什么密不可及的概念了。

有些朋友可能也会发现，上面流程图中似乎没有考虑CR实例被删除时对deployment、service的操作，的确如此。不过对于一个7×24小时运行于后台的服务来说，我们更多关注的是其变更、伸缩、升级等操作，删除是优先级最低的需求。

#### 8\. 构建controller image

controller代码写完后，我们就来构建controller的image。通过前文我们知道，这个controller其实就是运行在k8s中的一个deployment下的pod。我们需要构建其image并通过deployment部署到k8s中。

kubebuilder创建的operator工程中包含了Makefile，通过make docker-build即可构建controller image。docker-build使用golang builder image来构建controller源码，不过如果不对Dockerfile稍作修改，你很难编译过去，因为默认GOPROXY在国内无法访问。这里最简单的改造方式是使用vendor构建，下面是改造后的Dockerfile：

```
# Build the manager binary
FROM golang:1.18 as builder

ENV GOPROXY https://goproxy.cn
WORKDIR /workspace
# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum
COPY vendor/ vendor/
# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
#RUN go mod download

# Copy the go source
COPY main.go main.go
COPY api/ api/
COPY controllers/ controllers/

# Build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -mod=vendor -a -o manager main.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
#FROM gcr.io/distroless/static:nonroot
FROM katanomi/distroless-static:nonroot
WORKDIR /
COPY --from=builder /workspace/manager .
USER 65532:65532

ENTRYPOINT ["/manager"]
```

下面是构建的步骤：

```
$go mod vendor
$make docker-build

test -s /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen || GOBIN=/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.9.2
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
go fmt ./...
go vet ./...
KUBEBUILDER_ASSETS="/home/tonybai/.local/share/kubebuilder-envtest/k8s/1.24.2-linux-amd64" go test ./... -coverprofile cover.out
?       github.com/bigwhite/webserver-operator    [no test files]
?       github.com/bigwhite/webserver-operator/api/v1    [no test files]
ok      github.com/bigwhite/webserver-operator/controllers    4.530s    coverage: 0.0% of statements
docker build -t bigwhite/webserver-controller:latest .
Sending build context to Docker daemon  47.51MB
Step 1/15 : FROM golang:1.18 as builder
 ---> 2d952adaec1e
Step 2/15 : ENV GOPROXY https://goproxy.cn
 ---> Using cache
 ---> db2b06a078e3
Step 3/15 : WORKDIR /workspace
 ---> Using cache
 ---> cc3c613c19c6
Step 4/15 : COPY go.mod go.mod
 ---> Using cache
 ---> 5fa5c0d89350
Step 5/15 : COPY go.sum go.sum
 ---> Using cache
 ---> 71669cd0fe8e
Step 6/15 : COPY vendor/ vendor/
 ---> Using cache
 ---> 502b280a0e67
Step 7/15 : COPY main.go main.go
 ---> Using cache
 ---> 0c59a69091bb
Step 8/15 : COPY api/ api/
 ---> Using cache
 ---> 2b81131c681f
Step 9/15 : COPY controllers/ controllers/
 ---> Using cache
 ---> e3fd48c88ccb
Step 10/15 : RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -mod=vendor -a -o manager main.go
 ---> Using cache
 ---> 548ac10321a2
Step 11/15 : FROM katanomi/distroless-static:nonroot
 ---> 421f180b71d8
Step 12/15 : WORKDIR /
 ---> Running in ea7cb03027c0
Removing intermediate container ea7cb03027c0
 ---> 9d3c0ea19c3b
Step 13/15 : COPY --from=builder /workspace/manager .
 ---> a4387fe33ab7
Step 14/15 : USER 65532:65532
 ---> Running in 739a32d251b6
Removing intermediate container 739a32d251b6
 ---> 52ae8742f9c5
Step 15/15 : ENTRYPOINT ["/manager"]
 ---> Running in 897893b0c9df
Removing intermediate container 897893b0c9df
 ---> e375cc2adb08
Successfully built e375cc2adb08
Successfully tagged bigwhite/webserver-controller:latest
```

> 注：执行make命令之前，先将Makefile中的IMG变量初值改为IMG ?= bigwhite/webserver-controller:latest

构建成功后，执行make docker-push将image推送到镜像仓库中(这里使用了docker公司提供的公共仓库)。

#### 9\. 部署controller

之前我们已经通过make install将CRD安装到k8s中了，接下来再把controller部署到k8s上，我们的operator就算部署完毕了。执行make deploy即可实现部署：

```
$make deploy
test -s /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen || GOBIN=/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.9.2
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
test -s /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize || { curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash -s -- 3.8.7 /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin; }
cd config/manager && /home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize edit set image controller=bigwhite/webserver-controller:latest
/home/tonybai/test/go/operator/kubebuilder/webserver-operator/bin/kustomize build config/default | kubectl apply -f -
namespace/webserver-operator-system created
customresourcedefinition.apiextensions.k8s.io/webservers.my.domain unchanged
serviceaccount/webserver-operator-controller-manager created
role.rbac.authorization.k8s.io/webserver-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/webserver-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/webserver-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/webserver-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/webserver-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/webserver-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/webserver-operator-proxy-rolebinding created
configmap/webserver-operator-manager-config created
service/webserver-operator-controller-manager-metrics-service created
deployment.apps/webserver-operator-controller-manager created
```

我们看到deploy不仅会安装controller、serviceaccount、role、rolebinding，它还会创建namespace，也会将crd安装一遍。也就是说deploy是一个完整的operator安装命令。

> 注：使用make undeploy可以完整卸载operator相关resource。

我们用kubectl logs查看一下controller的运行日志：

```
$kubectl logs -f deployment.apps/webserver-operator-controller-manager -n webserver-operator-system
1.6600280818476188e+09    INFO    controller-runtime.metrics    Metrics server is starting to listen    {"addr": "127.0.0.1:8080"}
1.6600280818478029e+09    INFO    setup    starting manager
1.6600280818480284e+09    INFO    Starting server    {"path": "/metrics", "kind": "metrics", "addr": "127.0.0.1:8080"}
1.660028081848097e+09    INFO    Starting server    {"kind": "health probe", "addr": "[::]:8081"}
I0809 06:54:41.848093       1 leaderelection.go:248] attempting to acquire leader lease webserver-operator-system/63e5a746.my.domain...
I0809 06:54:57.072336       1 leaderelection.go:258] successfully acquired lease webserver-operator-system/63e5a746.my.domain
1.6600280970724037e+09    DEBUG    events    Normal    {"object": {"kind":"Lease","namespace":"webserver-operator-system","name":"63e5a746.my.domain","uid":"e05aaeb5-4a3a-4272-b036-80d61f0b6788","apiVersion":"coordination.k8s.io/v1","resourceVersion":"5238800"}, "reason": "LeaderElection", "message": "webserver-operator-controller-manager-6f45bc88f7-ptxlc_0e960015-9fbe-466d-a6b1-ff31af63a797 became leader"}
1.6600280970724993e+09    INFO    Starting EventSource    {"controller": "webserver", "controllerGroup": "my.domain", "controllerKind": "WebServer", "source": "kind source: *v1.WebServer"}
1.6600280970725305e+09    INFO    Starting Controller    {"controller": "webserver", "controllerGroup": "my.domain", "controllerKind": "WebServer"}
1.660028097173026e+09    INFO    Starting workers    {"controller": "webserver", "controllerGroup": "my.domain", "controllerKind": "WebServer", "worker count": 1}
```

可以看到，controller已经成功启动，正在等待一个WebServer CR的相关事件(比如创建)！下面我们就来创建一个WebServer CR!

#### 10\. 创建WebServer CR

webserver-operator项目中有一个CR sample，位于config/samples下面，我们对其进行改造，添加我们在spec中加入的字段：

```
// config/samples/_v1_webserver.yaml 

apiVersion: my.domain/v1
kind: WebServer
metadata:
  name: webserver-sample
spec:
  # TODO(user): Add fields here
  image: nginx:1.23.1
  replicas: 3
```

我们通过kubectl创建该WebServer CR：

```
$cd config/samples
$kubectl apply -f _v1_webserver.yaml
webserver.my.domain/webserver-sample created
```

观察controller的日志：

```
1.6602084232243123e+09  INFO    controllers.WebServer   Creating a new Deployment   {"Webserver": "default/webserver-sample", "Deployment.Namespace": "default", "Deployment.Name": "webserver-sample"}
1.6602084233446114e+09  INFO    controllers.WebServer   Creating a new Service  {"Webserver": "default/webserver-sample", "Service.Namespace": "default", "Service.Name": "webserver-sample-service"}
```

我们看到当CR被创建后，controller监听到相关事件，创建了对应的Deployment和service，我们查看一下为CR创建的Deployment、三个Pod以及service：

```
$kubectl get service
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes                 ClusterIP   172.26.0.1     <none>        443/TCP        22d
webserver-sample-service   NodePort    172.26.173.0   <none>        80:30010/TCP   2m58s

$kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
webserver-sample   3/3     3            3           4m44s

$kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
webserver-sample-bc698b9fb-8gq2h   1/1     Running   0          4m52s
webserver-sample-bc698b9fb-vk6gw   1/1     Running   0          4m52s
webserver-sample-bc698b9fb-xgrgb   1/1     Running   0          4m52s
```

我们访问一下该服务：

```
$curl http://192.168.10.182:30010
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

服务如预期返回响应！

#### 11\. 伸缩、变更版本和Service自愈

接下来我们来对CR做一些常见的运维操作。

-   副本数由3变为4

我们将CR的replicas由3改为4，对容器实例做一次扩展操作：

```
// config/samples/_v1_webserver.yaml 

apiVersion: my.domain/v1
kind: WebServer
metadata:
  name: webserver-sample
spec:
  # TODO(user): Add fields here
  image: nginx:1.23.1
  replicas: 4
```

然后通过kubectl apply使之生效：

```
$kubectl apply -f _v1_webserver.yaml
webserver.my.domain/webserver-sample configured
```

上述命令执行后，我们观察到operator的controller日志如下：

```
1.660208962767797e+09   INFO    controllers.WebServer   Deployment spec.replicas change {"Webserver": "default/webserver-sample", "from": 3, "to": 4}
```

稍后，查看pod数量：

```
$kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
webserver-sample-bc698b9fb-8gq2h   1/1     Running   0          9m41s
webserver-sample-bc698b9fb-v9gvg   1/1     Running   0          42s
webserver-sample-bc698b9fb-vk6gw   1/1     Running   0          9m41s
webserver-sample-bc698b9fb-xgrgb   1/1     Running   0          9m41s
```

webserver pod副本数量成功从3扩为4。

-   变更webserver image版本

我们将CR的image的版本从nginx:1.23.1改为nginx:1.23.0，然后执行kubectl apply使之生效。

我们查看controller的响应日志如下：

```
1.6602090494113188e+09  INFO    controllers.WebServer   Deployment spec.template.spec.container[0].image change {"Webserver": "default/webserver-sample", "from": "nginx:1.23.1", "to": "nginx:1.23.0"}
```

controller会更新deployment，导致所辖pod进行滚动升级：

```
$kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
webserver-sample-bc698b9fb-8gq2h   1/1     Running             0          10m
webserver-sample-bc698b9fb-vk6gw   1/1     Running             0          10m
webserver-sample-bc698b9fb-xgrgb   1/1     Running             0          10m
webserver-sample-ffcf549ff-g6whk   0/1     ContainerCreating   0          12s
webserver-sample-ffcf549ff-ngjz6   0/1     ContainerCreating   0          12s
```

耐心等一小会儿，最终的pod列表为：

```
$kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
webserver-sample-ffcf549ff-g6whk   1/1     Running   0          6m22s
webserver-sample-ffcf549ff-m6z24   1/1     Running   0          3m12s
webserver-sample-ffcf549ff-ngjz6   1/1     Running   0          6m22s
webserver-sample-ffcf549ff-t7gvc   1/1     Running   0          4m16s
```

-   service自愈：恢复被无删除的Service

我们来一次“误操作”，将webserver-sample-service删除，看看controller能否帮助service自愈：

```
$kubectl delete service/webserver-sample-service
service "webserver-sample-service" deleted
```

查看controller日志：

```
1.6602096994710526e+09  INFO    controllers.WebServer   Creating a new Service  {"Webserver": "default/webserver-sample", "Service.Namespace": "default", "Service.Name": "webserver-sample-service"}
```

我们看到controller检测到了service被删除的状态，并重建了一个新service！

访问新建的service：

```
$curl http://192.168.10.182:30010
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

可以看到service在controller的帮助下完成了自愈！

### 五. 小结

本文对Kubernetes Operator的概念以及优点做了初步的介绍，并基于kubebuilder这个工具开发了一个具有2级能力的operator。当然这个operator离完善还有很远的距离，其主要目的还是帮助大家理解operator的概念以及实现套路。

相信你阅读完本文后，对operator，尤其是其基本结构会有一个较为清晰的了解，并具备开发简单operator的能力！

文中涉及的源码可以在[这里](https://github.com/bigwhite/experiments/tree/master/webserver-operator)下载 – https://github.com/bigwhite/experiments/tree/master/webserver-operator。

### 六. 参考资料

-   kubernetes operator 101, Part 1: Overview and key features – https://developers.redhat.com/articles/2021/06/11/kubernetes-operators-101-part-1-overview-and-key-features
-   Kubernetes Operators 101, Part 2: How operators work – https://developers.redhat.com/articles/2021/06/22/kubernetes-operators-101-part-2-how-operators-work
-   Operator SDK: Build Kubernetes Operators – https://developers.redhat.com/blog/2020/04/28/operator-sdk-build-kubernetes-operators-and-deploy-them-on-openshift
-   kubernetes doc: Custom Resources – https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/
-   kubernetes doc: Operator pattern – https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
-   kubernetes doc: API concepts – https://kubernetes.io/docs/reference/using-api/api-concepts/
-   Introducing Operators: Putting Operational Knowledge into Software 第一篇有关operator的文章 by coreos – https://web.archive.org/web/20170129131616/https://coreos.com/blog/introducing-operators.html
-   CNCF Operator白皮书v1.0 – https://github.com/cncf/tag-app-delivery/blob/main/operator-whitepaper/v1/Operator-WhitePaper\_v1-0.md
-   Best practices for building Kubernetes Operators and stateful apps – https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps
-   A deep dive into Kubernetes controllers – https://docs.bitnami.com/tutorials/a-deep-dive-into-kubernetes-controllers
-   Kubernetes Operators Explained – https://blog.container-solutions.com/kubernetes-operators-explained
-   书籍《Kubernetes Operator》 – https://book.douban.com/subject/34796009/
-   书籍《Programming Kubernetes》 – https://book.douban.com/subject/35498478/
-   Operator SDK Reaches v1.0 – https://cloud.redhat.com/blog/operator-sdk-reaches-v1.0
-   What is the difference between kubebuilder and operator-sdk – https://github.com/operator-framework/operator-sdk/issues/1758
-   Kubernetes Operators in Depth – https://www.infoq.com/articles/kubernetes-operators-in-depth/
-   Get started using Kubernetes Operators – https://developer.ibm.com/learningpaths/kubernetes-operators/
-   Use Kubernetes operators to extend Kubernetes’ functionality – https://developer.ibm.com/learningpaths/kubernetes-operators/operators-extend-kubernetes/
-   memcached operator – https://github.com/operator-framework/operator-sdk-samples/tree/master/go/memcached-operator

___

[“Gopher部落”知识星球](https://wx.zsxq.com/dweb2/index/group/51284458844544)旨在打造一个精品Go学习和进阶社群！高品质首发Go技术文章，“三天”首发阅读权，每年两期Go语言发展现状分析，每天提前1小时阅读到新鲜的Gopher日报，网课、技术专栏、图书内容前瞻，六小时内必答保证等满足你关于Go语言生态的所有需求！2022年，Gopher部落全面改版，将持续分享Go语言与Go应用领域的知识、技巧与实践，并增加诸多互动形式。欢迎大家加入！

![img{512x368}](http://image.tonybai.com/img/tonybai/gopher-tribe-zsxq-small-card.png)  
![img{512x368}](http://image.tonybai.com/img/tonybai/go-programming-from-beginner-to-master-qr.png)

![img{512x368}](http://image.tonybai.com/img/tonybai/go-first-course-banner.png)  
![img{512x368}](http://image.tonybai.com/img/tonybai/imooc-go-column-pgo-with-qr.jpg)

[我爱发短信](https://51smspush.com/)：企业级短信平台定制开发专家 https://51smspush.com/。smspush : 可部署在企业内部的定制化短信平台，三网覆盖，不惧大并发接入，可定制扩展； 短信内容你来定，不再受约束, 接口丰富，支持长短信，签名可选。2020年4月8日，中国三大电信运营商联合发布《5G消息白皮书》，51短信平台也会全新升级到“51商用消息平台”，全面支持5G RCS消息。

著名云主机服务厂商DigitalOcean发布最新的主机计划，入门级Droplet配置升级为：1 core CPU、1G内存、25G高速SSD，价格5$/月。有使用DigitalOcean需求的朋友，可以打开这个[链接地址](https://m.do.co/c/bff6eed92687)：https://m.do.co/c/bff6eed92687 开启你的DO主机之路。

Gopher Daily(Gopher每日新闻)归档仓库 – https://github.com/bigwhite/gopherdaily

我的联系方式：

-   微博：https://weibo.com/bigwhite20xx
-   博客：tonybai.com
-   github: https://github.com/bigwhite

![](http://image.tonybai.com/img/tonybai/iamtonybai-wechat-qr.png)

商务合作方式：撰稿、出书、培训、在线课程、合伙创业、咨询、广告合作。

© 2022, [bigwhite](https://tonybai.com/). 版权所有.

Related posts:

1.  [Kubernetes Deployment故障排除图解指南](https://tonybai.com/2019/12/08/k8s-deployment-troubleshooting/ "Kubernetes Deployment故障排除图解指南")
2.  [在Kubernetes集群上部署高可用Harbor镜像仓库](https://tonybai.com/2017/12/08/deploy-high-availability-harbor-on-kubernetes-cluster/ "在Kubernetes集群上部署高可用Harbor镜像仓库")
3.  [在Kubernetes上如何基于自定义指标实现应用的自动缩放](https://tonybai.com/2019/10/11/autoscaling-apps-on-kubernetes/ "在Kubernetes上如何基于自定义指标实现应用的自动缩放")
4.  [理解Kubernetes网络之Flannel网络](https://tonybai.com/2017/01/17/understanding-flannel-network-for-kubernetes/ "理解Kubernetes网络之Flannel网络")
5.  [Kubernetes集群中Service的滚动更新](https://tonybai.com/2017/02/09/rolling-update-for-services-in-kubernetes-cluster/ "Kubernetes集群中Service的滚动更新")