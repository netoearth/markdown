### 介绍

[Helm](https://www.helm.sh/)是 Kubernetes 的包管理器，它允许开发人员和操作员更轻松地在 Kubernetes 集群上配置和部署应用程序。

Helm 包被称为_charts_，它们包含资源定义的模板，这些模板可以用最少的用户工作来部署和配置给定的应用程序。使用模板，您可以通过传入变量定义而不修改实际图表来管理图表、其设置和行为。自定义资源定义以及对已部署定义的修改，Helm 会自动管理。具有可能自定义的已部署图表称为_发布_。

在本教程中，您将设置 Helm 3 并学习如何安装、升级、回滚和管理图表和版本。您还将学习创建和打包您自己的图表，以及设置图表存储库，其中托管您可以立即安装的图表。

## 先决条件

-   启用了基于角色的访问控制 (RBAC) 的 Kubernetes 集群。有关发布的更多信息，您可以查看[Helm 发布](https://github.com/helm/helm/releases)页面。
    
-   `kubectl`安装在本地机器上的命令行工具，配置为连接到集群。您可以`kubectl` [在官方文档中](https://kubernetes.io/docs/tasks/tools/install-kubectl/)阅读有关安装的更多信息。
    

您可以使用以下命令测试连接：

```
  kubectl cluster-info
```

如果您没有收到任何错误，则您已连接到集群。如果您使用 访问多个集群`kubectl`，请务必通过运行以下命令来验证您是否选择了正确的集群上下文：

```
  kubectl config get-contexts
```

输出将列出可用的配置：

```
Output    CURRENT   NAME                    CLUSTER                 AUTHINFO                      NAMESPACE
  *         do-fra1-helm3-example   do-fra1-helm3-example   do-fra1-helm3-example-admin
```

这里，星号 ( `*`) 表示我们已连接到`do-fra1-helm3-example`集群。要切换集群，请运行：

```
  kubectl config use-context context-name
```

连接到正确的集群后，继续执行步骤 1 以开始安装 Helm。

在本节中，您将使用官方提供的 shell 脚本[安装 Helm 3](https://helm.sh/docs/intro/install/)。

首先导航到`/tmp`，您将通过运行以下命令来存储安装脚本：

```
cd /tmp
```

使用以下命令下载脚本：

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

您可以`get_helm.sh`在文本编辑器中检查以确保它是安全的。

通过将其权限设置为以下内容使其可执行：

```
chmod u+x get_helm.sh
```

最后，运行它来安装 Helm 3：

```
./get_helm.sh
```

您将收到类似于以下内容的输出：

```
OutputDownloading https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```

你的机器上安装了 Helm 3。您现在将了解图表存储库以及如何使用它们。

## 第 2 步 – 设置图表存储库

Helm 图表存储在任何人都可以托管的图表存储库中。默认情况下，Helm 3 没有预配置任何存储库。先前版本的 Helm 包括一个中央策划图表存储库；然而，Helm 3 的设计故意演变为图表开发人员管理他们自己的存储库，允许更多的自由和更快的发布。这意味着对于您希望使用的每个图表，您需要确保将托管存储库添加到您的 Helm 安装中。

为了帮助您找到正确的存储库，您可以使用[ArtifactHub.io](https://artifacthub.io/)，这是一个由[CNCF](https://www.cncf.io/)管理的开源网站，[用于](https://www.cncf.io/)对 Helm 图表及其存储库进行编目。它还跟踪其他 CNCF 项目使用的流行和有用的图表，因此它与`stable`以前版本的 Helm 工作的存储库不同。对于常见的项目，例如 Nginx 入口或监控工具，它是一个很好的来源。

您可以通过主页搜索要安装的图表。搜索`nginx`将显示与其相关的所有索引图表。

![ArtifactHub - Nginx 搜索](https://www.gingerdoc.com/wp-content/uploads/tutorials/step2a.png)

您将安装由 Kubernetes 团队管理的社区版。搜索`ingress-nginx`以在您的结果中找到它。选择它以访问其页面。

![ArtifactHub - ingress-nginx 搜索](https://www.gingerdoc.com/wp-content/uploads/tutorials/step2b.png)

![ArtifactHub - ingress-nginx 页面](https://www.gingerdoc.com/wp-content/uploads/tutorials/step2c.png)

每个图表都应该有一个详细说明它的作用，加上用于将其存储库添加到您的安装和安装图表的命令。如果没有，您仍然可以通过按页面右侧的**INSTALL**按钮来获取必要的命令。

![ArtifactHub - ingress-nginx 安装命令](https://www.gingerdoc.com/wp-content/uploads/tutorials/step2d.png)

您可以单击命令旁边的蓝色按钮来复制它。对第一个命令执行此操作，然后运行它：

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

要将存储库添加到 Helm，请运行`helm repo add`. 它接受的参数是存储库的名称及其位置。

输出将是：

```
Output"ingress-nginx" has been added to your repositories
```

当你添加一个新的 repo 时，你需要通过运行让 Helm 知道它包含什么：

```
helm repo update
```

您将收到以下输出，表明更新成功：

```
OutputHang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈Happy Helming!⎈
```

在此步骤中，您了解了 ArtifactHub 及其提供的功能。您还向 Helm 安装添加了一个新存储库。在下一步中，您将安装 Helm 图表。

## 第 3 步 – 安装 Helm Chart

在上一节中，您已为`ingress-nginx`图表添加了存储库。现在，您将把它安装到您的集群中。

每个图表都有配置变量，您可以设置这些变量来修改其行为。这些变量存储在称为`values.yaml`图表的一部分的文件中。除非您已将图表下载到您的机器上，否则您必须运行以下命令来查看它：

```
helm show values chart_name
```

要显示 的可用变量`ingress-nginx`，请替换`chart_name`：

```
helm show values ingress-nginx/ingress-nginx
```

输出将很长，显示`values.yaml`for的内容`ingress-nginx`。

要安装图表，您可以使用`helm install`：

```
helm install release_name repository/chart_name
```

一个_版本_是图表的部署实例，在这里你调用它`ingress-nginx`。

此命令将使用变量的默认值将图表安装到您的集群。如果您想修改其中的一些，您可以使用以下命令传入新的变量值`--set`：

```
helm install ingress-nginx/ingress-nginx --set variable_name=variable_value
```

您可以`--set`根据需要对任意数量的变量进行重复。由于我们现在不会自定义它，请按原样运行：

```
helm install ingress-nginx ingress-nginx/ingress-nginx
```

输出将类似于以下内容：

```
OutputNAME: ingress-nginx
LAST DEPLOYED: Wed Feb 24 10:12:37 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w ingress-nginx-controller'
...
```

请注意，`NAME`对应于您指定的版本名称。Helm 还列出了常见信息，例如发布状态和部署它的命名空间。该`NOTES`部分因图表而异，通常包含使用图表资源时的一些常见陷阱的快速入门指南或警告。在这里，它注意到正在创建负载均衡器，可能需要一些时间才能完成。

要检查部署的图表，请使用`helm list`：

```
helm list
```

您会发现这`ingress-nginx`是目前唯一部署的图表：

```
OutputNAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
ingress-nginx   default         1               2021-02-24 10:12:37.281049711 +0000 UTC deployed        ingress-nginx-3.23.0    0.44.0
```

您可以通过运行以下命令找到它在集群中拥有的服务：

```
kubectl get services
```

输出将类似于：

```
OutputNAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.245.211.81   46.101.68.67   80:30704/TCP,443:30700/TCP   7m19s
ingress-nginx-controller-admission   ClusterIP      10.245.50.17    <none>         443/TCP                      7m19s
kubernetes                           ClusterIP      10.245.0.1      <none>         443/TCP                      83m
```

现在您已将一个版本部署到集群，您将在部署时修改其配置。

## 第 4 步 – 升级版本

部署发布后，您无需在需要更改其配置时将其完全拆除并重新部署。您可以使用该`helm upgrade`命令使用新版本的图表升级版本，或设置新设置。

该`ingress-nginx`图表公开了`controller.replicaCount`控制已部署控制器 Pod 数量的变量。默认情况下，它设置为 1，您可以通过列出可用的 pod 来验证：

```
kubectl get pods
```

你会发现只有一个：

```
OutputNAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-7fc74cf778-kjtst   1/1     Running   0          12m
```

如果你想部署更多的冗余（例如，三个），你可以通过运行`upgrade`来发布和设置变量`3`：

```
helm upgrade ingress-nginx ingress-nginx/ingress-nginx --set controller.replicaCount=3 --reuse-values
```

您还传入`--reuse-values`，它指示 Helm 将您的更改基于已部署的版本，保留以前的配置。

在输出中，Helm 将修改修订以表示版本已升级：

```
OutputNAME: ingress-nginx
LAST DEPLOYED: Wed Feb 24 12:07:54 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
...
```

您可以通过运行列出可用的 pod：

```
kubectl get pods
```

您会发现列出了三个 pod，而不是一个：

```
OutputNAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-7fc74cf778-4hk9g   1/1     Running   0          18s
ingress-nginx-controller-7fc74cf778-kjtst   1/1     Running   0          22m
ingress-nginx-controller-7fc74cf778-wz595   1/1     Running   0          18s
```

接下来，您将回滚更改并完全删除版本。

## 步骤 5 — 回滚和删除版本

当您`upgrade`发布版本时，其修订号会增加。在内部，Helm 存储发布的所有修订版本，允许您在需要时返回到先前的修订版本。

要将 Pod 的数量恢复为一个，您可以`helm upgrade`再次运行并手动设置数量，因为这是一个很小的变化。但是，在处理具有许多变量的较大图表时，手动还原是不可行的，应该自动化。

要回滚版本，请使用`helm rollback`：

```
helm rollback release_name release_revision
```

您可以使用它`ingress-nginx`通过回滚到修订版本来还原您所做的更改`1`：

```
helm rollback ingress-nginx 1
```

您将收到以下输出，表明操作成功：

```
OutputRollback was a success! Happy Helming!
```

您可以通过列出版本来检查当前版本：

```
helm list
```

您会发现修订版现在是`3`，而不是`1`：

```
OutputNAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
ingress-nginx   default         3               2021-02-24 12:43:21.523664768 +0000 UTC deployed        ingress-nginx-3.23.0    0.44.0
```

Helm 将每个更改（包括回滚）视为对发布的新修订。您可以`3`通过运行以下命令检查已部署的 pod 数量来检查修订是否等于第一个：

```
kubectl get pods
```

你会发现只有一个：

```
OutputNAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-7fc74cf778-kjtst   1/1     Running   0          41m
```

要删除版本及其所有修订，您可以使用`helm delete`：

```
helm delete release_name
```

由于您不再需要它，请`ingress-nginx`通过运行以下命令删除：

```
helm delete ingress-nginx
```

输出将是：

```
Outputrelease "ingress-nginx" uninstalled
```

您可以列出发行版以检查是否没有发行版：

```
helm list
```

输出表将没有行：

```
OutputNAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

```

现在该版本已被删除，您可以在未来的部署中重复使用它的名称。

## 第 6 步 -（可选）创建自定义图表

在此可选步骤中，您将了解如何创建自定义图表、将资源定义放置在何处以及如何将其打包以供进一步分发。

您将创建一个名为 的新图表`example-chart`。运行以下命令来创建它：

```
helm create example-chart
```

这将创建一个名为`example-chart`以下文件和结构的新目录：

示例图表/

```
charts/
templates/
├─ tests/
│  ├─ test-connection.yaml
├─ deployment.yaml
├─ hpa.yaml
├─ ingress.yaml
├─ NOTES.txt
├─ service.yaml
├─ serviceaccount.yaml
├─ _helpers.tpl
Chart.yaml
values.yaml
```

您的图表将安装在目标集群上的资源定义位于该`templates`目录中。默认情况下 Helm 创建的作为起点的部署 Nginx 入口控制器。尽管它们的文件扩展名是 YAML，但它们使用 Go 的模板语法通过您可以传入的公开变量来保持可定制性。您可以`service.yaml`通过运行以下命令来显示内容进行验证：

```
cat example-chart/templates/service.yaml
```

你会发现它有模板指令来生成双大括号包围的值：

```
OutputapiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
```

引用的变量向用户公开并在`values.yaml`. `NOTES`部署后 Helm 显示的文本存储在 中`NOTES.txt`，并且也是模板化的。图表元数据，例如正在部署的软件的名称、版本和版本，在 中指定`Chart.yaml`：

示例图表/Chart.yaml

```
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes

...
type: application

...
version: 0.1.0

...
appVersion: "1.16.0"
```

要检查什么头盔将部署，您可以传递`--dry-run`，并`--debug`以`helm install`在图表目录指向：

```
helm install example-chart --dry-run --debug ./example-chart
```

输出将很长，并包含将应用于您的集群的所有最终资源定义。在处理图表时，您可以使用`helm upgrade`将新版本推送到 Kubernetes。

当需要共享完成的图表时，您可以通过运行以下命令将其打包以进行分发：

```
helm package ./example-chart
```

输出将是：

```
OutputSuccessfully packaged chart and saved it to: .../example-chart-0.1.0.tgz
```

打包的图表可以像添加存储库中的图表一样安装：

```
helm install example-chart example-chart-0.1.0.tgz
```

在此步骤中，您已创建并部署了自定义图表。您还打包了它，并了解了它的结构。

## 结论

您现在知道如何使用 Helm 安装和升级部署到 Kubernetes 集群的软件。您已经添加了图表存储库，并了解了它们的重要性以及 ArtifactHub 如何帮助您找到它们。您还创建了一个新的自定义图表并了解了版本修订以及如何在必要时回滚。

有关创建自定义图表的更多信息，请访问[官方指南](https://helm.sh/docs/chart_template_guide/getting_started/)。