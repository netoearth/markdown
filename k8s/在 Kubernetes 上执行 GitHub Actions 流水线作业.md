[GitHub Actions](https://github.com/features/actions) 是一个功能强大、“免费” 的 CI（持续集成）工具。

与[之前介绍的 Tekton](https://atbug.com/tekton-pipeline-practice/) 类似，GitHub Actions 的核心也是 Pipeline as Code 也就是所谓的流水线即代码。二者不同的是，GitHub Actions 本身就是一个 CI 平台，用户可以使用代码来定义流水线并在平台上运行；而 Tekton 本身是一个用于构建 CI/CD 平台的开源框架。

Pipeline as Code，既然与代码扯上了关系。那流水线的定义就可繁可简了，完全看需求。小到一个 GitHub Pages，大到流程复杂的项目都可以使用 GitHub Actions 来构建。

本篇文章不会介绍如何使用 GitHub Actions 的，如果还未用过的同学可以浏览下官方的[文档](https://docs.github.com/en/actions/quickstart)。今天主要来分享下如何在 Kubernetes 上的自托管资源来执行流水线作业。

## 0x01 背景

在介绍 GitHub Actions 的时候，免费带上了引号，为何？其作为一个 CI 工具，允许用户定义流水线并在平台上运行，需要消耗计算、存储、网络等资源，这些运行流水线的机器称为 Runner。GitHub 为不同类（等）型（级）的用户每月提供了不同的免费额度（额度用完后，每分钟 0.008 美元。），见下图。不同类型的主机，分钟数的消耗倍数也不同：Linux 为 1、macOS 为 10、Windows 为 2。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220709-at-224556.png)

拿免费用户来看，每月 2000 分钟看似也不少。比如笔者个人就是拿来构建下博客静态页面，以及几个简单的应用，每个月也用不了太多。但对于企业或者组织来说，尤其是当流水线的触发频繁（每次代码提交触发）、或者项目的单元测试耗时很长（bug 引起的或者项目本身的复杂度所致），积少成多也会变成一笔不小的开支。

那有没有办法使用自己的资源来运行流水线呢？有。GitHub Actions Runner 分为两种：Github 托管的 Runner 和自托管的 Runner。我们可以将自己的资源作为自托管的 Runner 来运行流水线，而且还可以借助 Kubernetes 的能力来管理这些 Runner。

同时，自托管 Runner 也适合那些对 CI 有更高要求的用户，比如更高性能、更多类型的计算资源等等。

## 0x02 准备工作

### Kubernetes 集群

我们使用 k3s 快速创建一个单节点的集群。

```
export INSTALL_K3S_VERSION=v1.22.11+k3s2
curl -sfL https://get.k3s.io | sh -s - --disable traefik --write-kubeconfig-mode 644 --write-kubeconfig ~/.kube/config
```

### 使用 Github Actions 构建的项目

这里使用之前做的一个 graalvm+maven 的基础镜像[仓库](https://github.com/addozhang/docker-graalvm-maven)来进行测试：https://github.com/addozhang/docker-graalvm-maven。

### GitHub Access Token

参考[文档](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)，创建 Access Token。

_注意：鉴于本演示的需要，分配完全的 repo 访问权限。_

![access token](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220710-at-000541.png)

## 0x03 创建 Runner

GitHub Actions Runner 的程序源码是[开源](https://github.com/actions/runner)的，GitHub 托管的 Runner 也是使用该程序运行的。

Runner 可以是某个仓库使用，也可以由组织下的所有仓库共享。鉴于这里演示用的项目，我们为上面提到的项目仓库创建 Runner。

Runner 在启动时，会通过 [GitHub API](https://docs.github.com/en/rest/actions/self-hosted-runners) 将自己注册到 GitHub Actions；然后不断发送请求到 GitHub 查看是否分配了流水线作业，如有则会执行流水线作业；Runner 停止运行时，需要进行注销的操作。这里的一系列操作，都会用到上面创建的 Access Token。

要在 Kubernetes 上运行，我们要将 Runner 应用以及上面的逻辑打成镜像，并以 Deployment 的方式部署到 Kubernetes 中。然后通过 [workflow 作业](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_job) webhook 查看排队的作业数来决定是否增加或者减少 Deployment 的副本数。

听起来就有些复杂，但确实是实现的基本方式。这里我们使用一个 Kubernetes controller [actions-runner-controller/actions-runner-controller](https://github.com/actions-runner-controller/actions-runner-controller) 来实现所有的流程。

actions-runner-controller 提供了三种 CRD：

-   Runner：可以理解为 `Pod`，该 Runner 只能执行一次作业。
-   RunnerDeployments：可以理解为 `Deployment`，可以设置要创建的 Runner 数量。Runner 在执行完作业后会销毁，然后 Controller 会创建新的 Runner 等待作业调度。
-   RunnerSets：可以理解为 `StatefulSet`，也是基于 `StatefulSet` 来创建 Pod（即 Runner），提供 `StatefulSet` 的特性。

更多用法，可以参考 [actions-runner-controller 官方文档](https://github.com/actions-runner-controller/actions-runner-controller#usage)。

### 部署 Cert Manager

actions-runner-controller 的运行，需要使用 cert-mananger。通过下面的命令来部署：

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
```

检查 pod 是否成功启动：

```
kubectl get pods -n cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-66b646d76-7b9fz               1/1     Running   0          18s
cert-manager-cainjector-59dc9659c7-rzlrl   1/1     Running   0          18s
cert-manager-webhook-7d8f555998-6zkqp      1/1     Running   0          18s
```

### 启动 Controller

接下来就是启动 controller，本文发布时的最新版本是 v0.25.0。使用下面的命令部署 controller：

```
kubectl create -f https://github.com/actions-runner-controller/actions-runner-controller/releases/download/v0.25.0/actions-runner-controller.yaml
```

> 注：这里使用的 `create` 而非 `apply`。使用 `apply` 会报类似下面的错误： Error from server (Invalid): error when creating “[https://github.com/actions-runner-controller/actions-runner-controller/releases/download/v0.25.0/actions-runner-controller.yaml":](https://github.com/actions-runner-controller/actions-runner-controller/releases/download/v0.25.0/actions-runner-controller.yaml%22:) CustomResourceDefinition.apiextensions.k8s.io “runnerdeployments.actions.summerwind.dev” is invalid: metadata.annotations: Too long: must have at most 262144 bytes

此时，pod 无法启动，还需要为其创建 Secret 来提供 GitHub Access Token：

```
export GITHUB_TOKEN=<TOKEN_HERE>
kubectl create secret generic controller-manager \
    -n actions-runner-system \
    --from-literal=github_token=${GITHUB_TOKEN}
```

现在可以看到 pod 可以成功运行：

```
kubectl get pods -n actions-runner-system
NAME                                  READY   STATUS    RESTARTS   AGE
controller-manager-58c598f64d-27xn6   2/2     Running   0          40s
```

### 创建 Runner

执行下面的命令，创建一个名为 `building-runner` 的 `Runner` CR。

```
kubectl apply -f - <<EOF
apiVersion: actions.summerwind.dev/v1alpha1
kind: Runner
metadata:
  name: building-runner
spec:
  repository: addozhang/docker-graalvm-maven
  env: []
EOF
```

此时，会发现 controller 创建了一个同名的 pod：

```
kubectl get pods -l actions-runner="" -n actions-runner-system
NAME              READY   STATUS    RESTARTS   AGE
building-runner   2/2     Running   0          16s
```

在仓库的 `Settings/Actions/Runners` 中可以看到同名的 Runner，处于 `idle` 状态。

![runner list](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220710-at-003748.png)

假如此时启动流水线作业，会发现作业并没有调度该 Runner 上。这是因为还没有将作业 Runner 的 label 设置为该 runner 的 label：`self-hosted`、`Linux`、 `X64`。

修改流水线定义，将 `runs-on` 指定为 `[self-hosted, linux, X64]`：

![custom labels](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220710-at-093403.png)

然后可以查看作业成功调度到该 runner 上运行：

![schedule job](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220710-at-093804.png)

现在再去看 pod 的状态，其中的 `runner` 容器是 `Completed`：

```
kubectl get pods -l actions-runner="" -n actions-runner-system
NAME              READY   STATUS     RESTARTS   AGE
building-runner   1/2     Running   0          3m53s
```

此时，再去触发流水线执行，作业会一直等待可用的 runner 来执行：

![queued job](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/07/10/20220710-at-010913.png)

runner 无法重复使用，怎么办？

## 0x04 可重用 Runner

`RunnerDeployment` 要派上用场了，前面提到可以将 `RunnerDeployments` 理解为 `Deployment`，可以设置要创建的 Runner 数量。Runner 在执行完作业后会销毁，然后 Controller 会创建新的 Runner 等待作业调度。

```
kubectl apply -f - <<EOF
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: building-runner
spec:
  template:
    spec:
      repository: addozhang/docker-graalvm-maven
      env: []
EOF
```

使用上面的命令创建 `RunnerDeployment` 之后，controller 会创建新的 pod（runner）并得到作业的调度；完成作业的执行后，pod `building-runner-9k4cj-ml46v` 被销毁，新的 pod `building-runner-9k4cj-f8twz` 被创建并等待作业的调度：

```
kubectl get events --sort-by='metadata.creationTimestamp'
...
3m7s        Normal    PodCreated                 runner/building-runner-9k4cj-ml46v         Created pod 'building-runner-9k4cj-ml46v'
3m6s        Normal    Scheduled                  pod/building-runner-9k4cj-ml46v            Successfully assigned actions-runner-system/building-runner-9k4cj-ml46v to ubuntu-dev1
3m7s        Normal    RegistrationTokenUpdated   runner/building-runner-9k4cj-ml46v         Successfully update registration token
3m6s        Normal    Pulling                    pod/building-runner-9k4cj-ml46v            Pulling image "summerwind/actions-runner:latest"
3m3s        Normal    Started                    pod/building-runner-9k4cj-ml46v            Started container docker
3m3s        Normal    Pulled                     pod/building-runner-9k4cj-ml46v            Successfully pulled image "summerwind/actions-runner:latest" in 3.06531539s
3m3s        Normal    Created                    pod/building-runner-9k4cj-ml46v            Created container runner
3m3s        Normal    Started                    pod/building-runner-9k4cj-ml46v            Started container runner
3m3s        Normal    Created                    pod/building-runner-9k4cj-ml46v            Created container docker
3m3s        Normal    Pulled                     pod/building-runner-9k4cj-ml46v            Container image "docker:dind" already present on machine
2m6s        Normal    RegistrationTokenUpdated   runner/building-runner-9k4cj-f8twz         Successfully update registration token
2m6s        Normal    PodCreated                 runner/building-runner-9k4cj-f8twz         Created pod 'building-runner-9k4cj-f8twz'
2m5s        Normal    Scheduled                  pod/building-runner-9k4cj-f8twz            Successfully assigned actions-runner-system/building-runner-9k4cj-f8twz to ubuntu-dev1
2m5s        Normal    Pulling                    pod/building-runner-9k4cj-f8twz            Pulling image "summerwind/actions-runner:latest"
2m5s        Normal    Killing                    pod/building-runner-9k4cj-ml46v            Stopping container docker
2m3s        Normal    Pulled                     pod/building-runner-9k4cj-f8twz            Successfully pulled image "summerwind/actions-runner:latest" in 2.50468863s
2m3s        Normal    Created                    pod/building-runner-9k4cj-f8twz            Created container runner
2m3s        Normal    Started                    pod/building-runner-9k4cj-f8twz            Started container runner
2m3s        Normal    Pulled                     pod/building-runner-9k4cj-f8twz            Container image "docker:dind" already present on machine
2m3s        Normal    Created                    pod/building-runner-9k4cj-f8twz            Created container docker
2m3s        Normal    Started                    pod/building-runner-9k4cj-f8twz            Started container docker
```

前面创建 `RunnerDeployment` 的时候没有指定副本数，也就是 runner 的数量，controller 只创建了一个 Runner。假设这个 `RunnerDeployment` 是某个组织下多个项目共用的，多个项目同时执行作业会怎样？

这里我重新运行 3 个已经完成的作业，模拟作业的并发：

```
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token ${GITHUB_TOKEN}" https://api.github.com/repos/addozhang/docker-graalvm-maven/actions/runs/2643982502/rerun
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token ${GITHUB_TOKEN}" https://api.github.com/repos/addozhang/docker-graalvm-maven/actions/runs/2643982274/rerun
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token ${GITHUB_TOKEN}" https://api.github.com/repos/addozhang/docker-graalvm-maven/actions/runs/2643982274/rerun
```

或者 push 几个空的提交到仓库（不推荐，会有 commit 历史。测试项目随意）：

```
git commit --allow-empty -m "trigger action"
git push
```

去 Actions 列表会发现，只有一个作业在运行，其他两个都是 `queued` 的等待状态。前面的作业执行完成后，后一个作业才会被执行。

不支持并发？既然是类似 Deployment 的方式运行 runner，那是否有类似 HPA（水平 pod 自动扩缩容）的功能？

## 0x05 自动伸缩

actions-runner-controller 在 3 个 Runner 的 CRD 之外，还提供了类似 HPA 的 CRD `HorizontalRunnerAutoscaler`，简称 HRA。

HRA 可以根据指标 `PercentageRunnersBusy` 或者 `TotalNumberOfQueuedAndInProgressWorkflowRuns` 来对 runner 进行扩缩容，或者基于 GitHub Events（webhook）来进行扩缩容。这两种都各有优缺点，前者指标是通过 GitHub API 轮训等待的作业数，实现简单，但时效性差；后者基于事件触发时效性更佳，但是实现复杂，需要对外暴露访问端点接收 GitHub Event。

这里为了演示，使用基于指标的方式进行扩缩容。

```
kubectl apply -f - <<EOF
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: building-runner-autoscaler
spec:
  scaleDownDelaySecondsAfterScaleOut: 30
  scaleTargetRef:
    name: building-runner
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: PercentageRunnersBusy
    scaleUpThreshold: '0.75'
    scaleDownThreshold: '0.25'
    scaleUpFactor: '2'
    scaleDownFactor: '0.5'
EOF
```

还是通过 API 触发作业模拟并发执行，由于轮训间隔比较长（默认 1 分钟），自动扩容需要等待一段时间：

```
kubectl get pods -l actions-runner="" -n actions-runner-system
NAME                          READY   STATUS    RESTARTS   AGE
building-runner-k6jlc-dk7gc   2/2     Running   0          1m34s
building-runner-k6jlc-pbwnz   2/2     Running   0          54s
building-runner-k6jlc-nlptp   2/2     Running   0          54s
```

## 0x06总结

GitHub Actions 是个很强大的 CI 工具，结合自托管的 Runner 可以在付出较低成本的基础上有更好的体验。本文也只是从满足需求的出发，对 “Runner on Kubernetes” 进行了探索。对一个工具从会用到用好，还有很长的路要走。但满足当前需求，已是足矣。

有兴趣的同学可以更进一步思考：

-   如何限制 Runner 运行时的资源占用
-   持久化如何实现，比如 Maven 构建时的本地库，Nodejs 的 node\_mudules 如何避免重复下载
-   自动扩缩容能否更加高效