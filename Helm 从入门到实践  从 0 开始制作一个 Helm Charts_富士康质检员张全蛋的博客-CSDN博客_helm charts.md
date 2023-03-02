## **Chart模板**

___

![](https://img-blog.csdnimg.cn/9d675e3fa9be478b9c78978f7accbaa0.png)

## **快速制作一个chart**

___

```
[root@master ~]# mkdir -p testchart/templates [root@master testchart]# lstemplates
```

Chart.yaml 这里就是chart相关的描述信息，从以创建的chart当中将文件拷贝过来

```
[root@master testchart]# cp ../mychart/Chart.yaml .[root@master testchart]# vim Chart.yaml [root@master testchart]# cat Chart.yaml sion: v2: testchart       #chart名字ption: A Helm chart for Kubernetes: applicationn: 0.1.0      #chart的版本号，sion: 1.16.0  #当前应用的版本[root@master testchart]# helm listNAMESPACEREVISIONUPDATED                                STATUS  CHART        APP VERSIONdefault  3       2022-06-24 22:19:33.782165096 +0800 CSTdeployedmychart-0.1.01.16.0 
```

 拷贝values.yml

```
[root@master testchart]# cp ../mychart/values.yaml .[root@master testchart]# vim values.yaml [root@master testchart]# cat values.yaml aCount: 1orLabels: nginx:sitory: nginx: "1.17"
```

这个目录下存放着模板文件。

这样就将差异化的地方，通过模板语法，进行做了一个替换，那当helm执行的时候，会将具体的字段替换为具体值。

```
[root@master templates]# cat deployment.yaml sion: apps/v1: Deploymentta:ls:p: web: {{ .Release.Name }}:icas: {{ .Values.replicaCount }}ctor:tchLabels:app: {{ .Values.selectorLabels }}late:tadata:labels:  app: {{ .Values.selectorLabels }}ec:containers:- image: {{ .Values.image.repository }}:{{ .Values.image.tag }}  name: nginx
```

```
[root@master templates]# cat service.yaml sion: v1: Serviceta:: {{ .Release.Name }}:ctor:p: {{ .Values.selectorLabels }}s:- port: 80otocol: TCPrgetPort: 80
```

 {{ .Release.Name }}这个引用的是helm install  **Release.Name**  testchart/， **Release.Name是什么那么** {{ .Release.Name }}的值就是什么。

```
[root@master ~]# helm install testchart testchart/: testchartEPLOYED: Sat Jun 25 00:16:15 2022ACE: default: deployedON: 1UITE: None:testchart
```

NOTES：hello testchart    这个将NOTES.txt文件里面的内容进行展示

```
[root@master templates]# helm list   NAMESPACEREVISIONUPDATED                                STATUS  CHART          APP VERSIONartdefault  1       2022-06-25 00:16:15.693048986 +0800 CSTdeployedtestchart-0.1.01.16.0 [root@master templates]# kubectl get pod                         READY   STATUS    RESTARTS   AGEart-db749865c-lfqxj      1/1     Running   0          65s[root@master templates]# kubectl get svc        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGEart     ClusterIP   10.109.132.249   <none>        80/TCP         16m
```

## **Chart模板：内置对象**

___

在上面示例中，模板文件中.Release、.Values是Helm内置对象，顶级开头写。

**Release对象：**获取发布记录信息 

![](https://img-blog.csdnimg.cn/9f3dd7e9573f4472a1a3a9f68dea7c97.png)

**Values对象：** 为Chart模板提供值，这个对象的值有4个来源：

-   chart包中的values.yaml文件
-   helm install或者helm upgrade的-f或者--values参数传入的自定义yaml文件
-   \--set参数传入值

**Chart对象：** 可以通过Chart对象访问Chart.yaml文件的内容，例如： {{ .Chart.AppVersion }}

```
[root@master ~]# cat mychart/Chart.yaml | grep appVsion: 1.16.0
```

## **Chart模板：调试** 

___

使用helm install提供了--dry-run和--debug调试参数，帮助你验证模板正确性，并把渲染后的模板打印出来，而不会真正的去部署。

```
# helm install --dry-run web mychart
```

## **如何制作自己的 Helm Charts**

___

我们平时在日常生活中会经常在不同的平台上与各种各样的应用打交道，比如从苹果的 App Store 里下载的淘宝、高德、支付宝等应用，或者是在 PC 端安装的Word、Photoshop、Steam。这些各类平台上的应用程序，对用户而言，大多只需要点击安装就可使用。

然而，在云（[Kubernetes](https://so.csdn.net/so/search?q=Kubernetes&spm=1001.2101.3001.7020)）上，部署一个应用往往却不是那么简单。如果想要部署一个应用程序到云上，首先要准备好它所需要的环境，打包成 Docker 镜像。进而把镜像放在部署文件（Deployment）中、配置服务（Service）、应用所需的账户（ServiceAccount）及权限（Role）、命名空间（Namespace）、密钥信息（Secret）、可持久化存储（PersistentVolumes）等资源。也就是编写一系列互相相关的 YAML 配置文件，将它们部署在 Kubernetes 集群上。

但是即便应用的开发者可以把这些[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)镜像存放在公共仓库中，并且将部署所需的 YAML 资源文件提供给用户，用户仍然需要自己去寻找这些资源文件，并把它们一一部署。倘若用户希望修改开发者提供的默认资源，比如使用更多的副本（Replicas）或是修改服务端口（Port），他还需要自己去查应该在这些资源文件的哪些地方修改，更不用提版本变更与维护会给开发者和用户造成多少麻烦了。可见最原始的 Kubernetes 应用形态并不便利。

## **Helm 与 Helm Chart**

___

在这样的大环境下，有一系列基于 Kubernetes 的应用包管理工具横空出世。而我们今天的主角 Helm，就是这其中最受欢迎的选择之一。

![图片](https://img-blog.csdnimg.cn/img_convert/57f0ae22c40a1e3eb99b4f5114669d56.png)

开发者按照 Helm Chart 的格式，将应用所需的资源文件包装起来，通过模版化（Templating）的方式将一些可变字段（比如我们之前提到的暴露哪个端口，使用多少副本）暴露给用户，最后将封装好的应用包，也就是 Helm Chart，集中存放在统一的仓库中供用户浏览下载。

站在用户角度，用户只需要一行简单的命令就可以完成应用的安装、卸载与升级。对于安装之后状态，也可以通过 `helm list` 或者是原生的 kubectl 进行查询。

```
$ helm install redis stable/redis$ kubectl get pods                READY       STATUS     RESTARTS           AGE-master-0        1/1         Running    0                  63s-slave-0-0       1/1         Running    0                  63s-slave-1-0       1/1         Running    0                  13s$ helm delete redis
```

## **创作 Helm Chart**

___

那么站在开发者的角度上，我们应该如何去创作一个 Helm 应用包呢？

-   **准备工作**
    

首先我们需要一个准备部署的镜像。这个镜像可以是一个 Java 程序、一个 Python 脚本、甚至是一个空的 linux 镜像跑几条命令。

![图片](https://img-blog.csdnimg.cn/img_convert/a91d5061a044b5b15a305e5257a37b4c.png)

这里我们使用一个简单的基于 golang 的。该服务通过读取环境变量 `USERNAME` 获得用户自己定义的名称，然后监听 80 端口。对于任意 HTTP 请求，返回 `Hello ${USERNAME}`。比如如果设置 `USERNAME=world`（默认场景），该服务会返回 `Hello world`。

然后我们使用 对镜像进行打包。**先对 Golang 代码进行编译，然后将编译后的程序放在基于 alpine 的镜像中，以缩小镜像体积。**

![图片](https://img-blog.csdnimg.cn/img_convert/3aaf87dd3696fbe74adc123f0973eafe.png)

在 Docker 构建好镜像之后，我们把镜像上传到仓库中，比如 Docker Hub 或是阿里云容器镜像仓库。准备工作做好之后，我们就可以开始创作 Helm Chart 了。

-   **开始创作**
    

运行 `helm create my-hello-world`，会得到一个 helm 自动生成的空 chart。这个 chart 里的名称是`my-hello-world`。**需要注意的是，Chart 里面的 my-hello-world 名称需要和生成的 Chart 文件夹名称一致。****如果修改 my-hello-world，则需要做一致的修改****。** 现在，我们看到 Chart 的文件夹目录如下:

```
-hello-world├── charts├── Chart.yaml├── templates│   ├── deployment.yaml│   ├── _helpers.tpl│   ├── ingress.yaml│   ├── NOTES.txt│   └── service.yaml└── values.yaml
```

在根目录下的 Chart.yaml 文件内，声明了当前 Chart 的名称、版本等基本信息，这些信息会在该 Chart 被放入仓库后，供用户浏览检索。比如我们可以把 Chart 的 Description 改成 "My first hello world helm chart"。在 Chart.yaml 里有两个跟版本相关的字段，**其中 version 指明的是 Chart 的版本，也就是我们应用包的版本，而 appVersion 指明的是内部实际使用的应用版本。**

```
[root@node1 mychart]# vim Chart.yaml sion: 1.18.0n: 0.1.1[root@node1 mychart]# helm upgrade --set image.tag=1.18 mychart mychart/[root@node1 ~]# helm history mychartONUPDATED                 STATUS    CHART        APP VERSIONDESCRIPTION       Sat Aug 13 06:50:47 2022supersededmychart-0.1.01.16.0     Install complete  Sat Aug 13 06:54:03 2022supersededmychart-0.1.01.16.0     Upgrade complete  Sat Aug 13 06:56:21 2022supersededmychart-0.1.01.16.0     Rollback to 1     Sat Aug 13 07:00:17 2022deployed  mychart-0.1.11.18.0     Upgrade complete[root@node1 nginxchart]# helm list NAMESPACEREVISIONUPDATED                                STATUS  CHART        APP VERSIONtdefault  4       2022-08-13 07:00:17.802406484 +0800 CSTdeployedmychart-0.1.11.18.0 
```

在 templates 文件夹内存放了各类应用部署所需要使用的 YAML 文件，比如 Deployment 和 Service。在我们当前的应用内，我们只需要一个 deployment，而有的应用可能包含不同组件，需要多个 deployments，那么我们就可以在 templates 文件夹下放置 deploymentA、deploymentB 等。同样的，**如果我们需要配置 serviceaccount、secret、volumes 等内容，也可以在里面添加相应的配置文件。** 

```
sion: apps/v1beta2: Deploymentta:: {{ template "my-hello-world.fullname" . }}s:{{ include "my-hello-world.labels" . | indent 4 }}:cas: {{ .Values.replicaCount }}tor:chLabels:pp.kubernetes.io/name: {{ include "my-hello-world.name" . }}pp.kubernetes.io/instance: {{ .Release.Name }}ate:adata:abels: app.kubernetes.io/name: {{ include "my-hello-world.name" . }} app.kubernetes.io/instance: {{ .Release.Name }}c:ontainers: - name: {{ .Chart.Name }}   image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"   imagePullPolicy: {{ .Values.image.pullPolicy }}   env:     - name: USERNAME       value: {{ .Values.Username }}......
```

Helm Chart 对于应用的打包，不仅仅是将 Deployment 和 Service 以及其它资源整合在一起。我们看到 deployment.yaml 和 service.yaml 文件被放在 templates/ 文件夹下，**相较于原生的 Kubernetes 配置，多了很多渲染所用的可注入字段。**比如在 deployment.yaml 的 `spec.replicas` 中，使用的是 `.Values.replicaCount` 而不是 Kubernetes 本身的静态数值。这个用来控制应用在 Kubernetes 上应该有多少运行副本的字段，在不同的应用部署环境下可以有不同的数值，而这个数值便是由注入的 `Values` 提供。

```
aCount: 1:itory: somefive/hello-worldolicy: IfNotPresenterride: ""meOverride: ""e:: ClusterIP: 80......me: AppHub
```

在根目录下我们看到有一个 `values.yaml` 文件，这个文件提供了应用在安装时的默认参数。在默认的 `Values` 中，我们看到 `replicaCount: 1` 说明该应用在默认部署的状态下只有一个副本。

为了使用我们要部署应用的镜像，我们看到 deployment.yaml 里在 `spec.template.spec.containers` 里，`image` 和 `imagePullPolicy` 都使用了 `Values` 中的值。

**其中 `image` 字段由 `.Values.image.repository` 和 `.Chart.AppVersion` 组成。看到这里，同学们应该就知道我们需要变更的字段了：一个是位于 values.yaml 内的 `image.repository`，另一个是位于 Chart.yaml 里的 `AppVersion`。我们将它们与我们需要部署应用的 docker 镜像匹配起来。这里我们把 values.yaml 里的 `image.repository` 置成 `somefive/hello-world`，把 Chart.yaml 里的 `AppVersion`设置成 `1.0.0` 即可。**

类似的，我们可以查看 service.yaml 内我们要部署的服务，其中的主要配置也在 values.yaml 中。默认生成的服务将 80 端口暴露在 Kubernetes 集群内部。我们暂时不需要对这一部分进行修改。

由于部署的 hello-world 服务会从环境变量中读取 `USERNAME` 环境变量，我们将这个配置加入 deployment.yaml。

```
- name: {{ .Chart.Name }}  image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"  imagePullPolicy: {{ .Values.image.pullPolicy }}  env:  - name: USERNAME    value: {{ .Values.Username }}
```

现在我们的 deployment.yaml 模版会从 values.yaml 中加载 `Username` 字段，因此相应的，我们也在 values.yaml 中添加 `Username: AppHub`。现在，我们的应用就会从 values.yaml 中读取 Username 把它放入镜像的环境变量中启动了。

-   **校验打包**
    

在准备好我们的应用后，**我们可以使用 Helm lint 来粗略地检查一下制作的 Chart 有没有什么语法上的错误。如果没有问题的话，我们就可以使用 helm package 命令对我们的 Chart 文件夹进行打包**，打包后我们可以得到一个 my-hello-world-0.1.0.tgz 的应用包。这个便是我们完成的应用了。

```
$ helm lint --strict my-hello-world1 chart(s) linted, 0 chart(s) failed[INFO] Chart.yaml: icon is recommended$ helm package my-hello-world
```

我们可以使用 `helm install` 命令尝试安装一下刚刚做好的应用包，然后用 kubectl 查看一下运行 pod 的状态。我们可以通过 port-forward 命令将该 pod 的端口映射到本地端口上，这个时候我们就可以通过访问 localhost 来访问部署好的应用了。

```
$ helm install my-hello-world-chart-test my-hello-world-0.1.0.tgz$ kubectl get pods                                          READY    STATUS     RESTARTS    AGE-hello-world-chart-test-65d6c7b4b6-ptk4x-0  1/1      Running    0           4m3s$ kubectl port-forward my-hello-world-chart-test-65d6c7b4b6-ptk4x 8080:80$ curl localhost:8080 AppHub
```

-   **参数重载**
    

有的同学可能会有疑惑，虽然我们应用开发者把可配置的信息暴露在的 values.yaml 中，用户使用应用时想要修改该怎么办呢？答案很简单，用户只需要在 install 时使用 set 参数设置想要覆盖的参数即可。应用开发者在 Chart 的 values 配置中只是提供了默认的安装参数，用户也可以在安装时指定自己的配置。类似的，如果用户可以用 upgrade 命令替代 install，实现在原有部署好的应用的基础上变更配置。

```
$ helm install my-app my-hello-world-0.1.0.tgz --set Username="Cloud Native"$ helm install my-super-app my-hello-world-0.1.0.tgz -f my-values.yaml$ helm upgrade my-super-app my-hello-world-0.1.0.tgz -f my-new-values.yaml
```

-   **修改提示信息**
    

我们注意到在安装 Chart 指令运行后，屏幕的输出会出现：

```
:. Get the application URL by running these commands:  export POD_NAME=$(kubectl get pods -l "app=my-hello-world,release=my-hello-world-chart-test2" -o jsonpath="{.items[0].metadata.name}")  echo "Visit http://127.0.0.1:8080 to use your application"  kubectl port-forward $POD_NAME 8080:80
```

这里的注释是由 Chart 中的 `templates/NOTES.txt` 提供的。我们注意到原始的 NOTES 中，所写的 `"app={{ template "my-hello-world.name" . }},release={{ .Release.Name }}"` 和我们的 deployment.yaml 中所写的配置不太一样。我们可以把它改成 `"app.kubernetes.io/name={{ template "my-hello-world.name" . }},app.kubernetes.io/instance={{ .Release.Name }}"`，将 values.yaml 中的 `version` 更新成 `0.1.1`。然后重新打包 Chart（运行 `helm package`）。得到新的 my-hello-world-0.1.1.tgz 之后，升级原有 Chart（运行 `helm upgrade my-hello-world-chart-test2 my-hello-world-0.1.1.tgz --set Username="New Chart"`），就能看到更新过后的 NOTES 了。

```
:. Get the application URL by running these commands:  export POD_NAME=$(kubectl get pods -l "app.kubernetes.io/name=my-hello-world,app.kubernetes.io/instance=my-hello-world-chart-test2" -o jsonpath="{.items[0].metadata.name}")  echo "Visit http://127.0.0.1:8080 to use your application"  kubectl port-forward $POD_NAME 8080:80
```

## **应用分享**

___

![图片](https://img-blog.csdnimg.cn/img_convert/6dfcec18b1640f2ac2949e50c78049c2.png)

那么制作完成的应用如何和其他人分享呢？Helm 官方推出的 ChartMuseum 提供了 Chart 仓库的构建方法，使用它可以创建自己的 Chart 仓库。然而自行维护一个仓库本身成本不小，而且对于用户而言如果每一个开发者都是自己的仓库，他就需要将所需应用对应的仓库都加入自己的检索列表中，很不利于应用的传播与分享。

![图片](https://img-blog.csdnimg.cn/img_convert/5dea43151010edea2f678c0f1b65531a.png)

开放云原生应用中心 Cloud Native App Hub，同步了各类应用，同时还提供了开发者上传应用的渠道。

在我们的开放云原生应用中心中，应用来自两个渠道。一方面，我们定期从一些国外的知名 Helm 仓库同步 Chart 资源，在同步的过程中，会对 Chart 内部使用的一部分 Docker 镜像进行同步替换（例如 gcr.io 或者 quay.io 的镜像），方便国内用户访问使用；另一方面，我们和 Helm 官方库一样在 上接受开发者通过 Pull Request 的形式提交自己的应用。提交成功的应用会在短期内同步至云原生应用中心，和其他官方应用展示在一起供其他用户使用。

希望大家共同参与，让开放云原生应用中心更加丰富，惠及更多人！

![图片](https://img-blog.csdnimg.cn/img_convert/6460c85f7ca9cbe8c11692c94b0675a6.png)