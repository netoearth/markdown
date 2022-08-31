之前写过一篇 [介绍了工具加速[[k8s/云原生]] Java 开发](https://mp.weixin.qq.com/s/CXJ1zUoktdI3c3CHVju0gA)。

其实日常工作中在集群上的操作也非常多，今天就来介绍我所使用的工具。

## kubectl-alias

使用频率最高的工具，我自己稍微修改了一下，加入了 `StatefulSet` 的支持。

这个是我的 [https://github.com/addozhang/kubectl-aliases](https://github.com/addozhang/kubectl-aliases)，基于 [https://github.com/ahmetb/kubectl-aliases](https://github.com/ahmetb/kubectl-aliases)。

比如输出某个 pod 的 json，`kgpoojson xxx` 等同于 `kubectl get pod xxx -o json`。

结合 [jq](https://stedolan.github.io/jq/) 使用效果更好 😂。

### 语法解读

-   **`k`**\=`kubectl`
    -   **`sys`**\=`--namespace kube-system`
-   commands:
    -   **`g`**\=`get`
    -   **`d`**\=`describe`
    -   **`rm`**\=`delete`
    -   **`a`**:`apply -f`
    -   **`ak`**:`apply -k`
    -   **`k`**:`kustomize`
    -   **`ex`**: `exec -i -t`
    -   **`lo`**: `logs -f`
-   resources:
    -   **`po`**\=pod, **`dep`**\=`deployment`, **`ing`**\=`ingress`, **`svc`**\=`service`, **`cm`**\=`configmap`, **`sec`\=`secret`**,**`ns`**\=`namespace`, **`no`**\=`node`
-   flags:
    -   output format: **`oyaml`**, **`ojson`**, **`owide`**
    -   **`all`**: `--all` or `--all-namespaces` depending on the command
    -   **`sl`**: `--show-labels`
    -   **`w`**\=`-w/--watch`
-   value flags (should be at the end):
    -   **`n`**\=`-n/--namespace`
    -   **`f`**\=`-f/--filename`
    -   **`l`**\=`-l/--selector`

## kubectx + kubens

[安装看这里](https://github.com/ahmetb/kubectx#installation)

`kubectx` 用于在不同的集群间进行快速切换。假如用 `kubectl`，你需要：

```
# context 列表
kubectl config current-context 
# 设置 context
kubectl config use-context coffee
```

![kubectx-demo](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/kubectxdemo.gif)

`kubens` 就是在不同 namespace 间快速切换的工具。用 `kubectl` 的话，需要：

```
# namespace 列表
kbuectl get ns
# kubectl config set-context --current --namespace=kube-system
```

![kubens-demo](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/kubensdemo.gif)

## k9s

没错，只比 k8s 多了个 1 😂。

[k9s](https://github.com/derailed/k9s) 提供了终端 UI 与 Kubernetes 集群进行编辑交互。本人常用的比如：

-   `F` 配置端口转发
-   `l` 输出 pod 日志
-   `e` 修改资源对象
-   `s` pod 终端交互模式
-   `y` yaml 方式输出资源对象
-   `d` describe 资源对象
-   `ctrl+d` 删除 pod

启动方式

```
# 指定 namespace 运行
k9s -n mycoolns
# 指定 context 运行
k9s --context coolCtx
# 只读模式运行
k9s --readonly
```

![k9s-pod](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/20210626102731.png)

键入问号“?” 就可以打开快捷操作指引。

![help](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/20210626103012.png)

## stern

[stern](https://github.com/wercker/stern) 可以用来 `tail` 集群上的多个 pod 和 pod 中多个[[容器]]的日志。不同的 pod 和[[容器]]以不同的颜色区分，方便 debug。

比如使用命令 `stern -l tier=control-plane -n kube-system` 可以输出 `kube-system` 命名空间下控制平面（`label` 为 `tier=control-plane`） pod 的日志。

![stern-control-plane](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/20210626104458.png)

命令行选项

```
Tail multiple pods and containers from Kubernetes

Usage:
  stern pod-query [flags]

Flags:
  -A, --all-namespaces             If present, tail across all namespaces. A specific namespace is ignored even if specified with --namespace.
      --color string               Color output. Can be 'always', 'never', or 'auto' (default "auto")
      --completion string          Outputs stern command-line completion code for the specified shell. Can be 'bash' or 'zsh'
  -c, --container string           Container name when multiple containers in pod (default ".*")
      --container-state string     If present, tail containers with status in running, waiting or terminated. Default to running. (default "running")
      --context string             Kubernetes context to use. Default to current context configured in kubeconfig.
  -e, --exclude strings            Regex of log lines to exclude
  -E, --exclude-container string   Exclude a Container name
      --field-selector string      Selector (field query) to filter on. If present, default to ".*" for the pod-query.
  -h, --help                       help for stern
  -i, --include strings            Regex of log lines to include
      --init-containers            Include or exclude init containers (default true)
      --kubeconfig string          Path to kubeconfig file to use
  -n, --namespace strings          Kubernetes namespace to use. Default to namespace configured in Kubernetes context. To specify multiple namespaces, repeat this or set comma-separated value.
  -o, --output string              Specify predefined template. Currently support: [default, raw, json] (default "default")
  -l, --selector string            Selector (label query) to filter on. If present, default to ".*" for the pod-query.
  -s, --since duration             Return logs newer than a relative duration like 5s, 2m, or 3h. Defaults to 48h.
      --tail int                   The number of lines from the end of the logs to show. Defaults to -1, showing all logs. (default -1)
      --template string            Template to use for log lines, leave empty to use --output flag
  -t, --timestamps                 Print timestamps
      --timezone string            Set timestamps to specific timezone (default "Local")
  -v, --version                    Print the version and exit

```

## Lens

[Lens](https://k8slens.dev/) 是用来控制 Kubernetes 的 IDE，开源且免费。

消除了集群操作的复杂性、提供了实时的可观察性、方便故障排查、支持多系统的桌面客户端、兼容多种集群。

![Lens](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/20210626113853.png)

## Infra App

[Infra App](https://infra.app/) 跟 Lens 差不多，UI 较 Lens 好些，但是功能就弱很多，类似 Lens 的只读模式。

免费版比收费版的区别只在于支持的集群数量，免费版只支持一个集群。

![Infra](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/20210626114417.png)

## kubefwd

[kubefwd](https://github.com/txn2/kubefwd)，这个一直有安装但是使用次数寥寥，因为应用之间的访问没有走 service，不过偶尔做些实验的时候会用的上。

> kubefwd 是一个用于端口转发Kubernetes中指定namespace下的全部或者部分pod的命令行工具。 kubefwd 使用本地的环回IP地址转发需要访问的service，并且使用与service相同的端口。 kubefwd 会临时将service的域条目添加到 /etc/hosts 文件中。

> 启动kubefwd后，在本地就能像在[[Kubernetes]]集群中一样使用service名字与端口访问对应的应用程序。

![kubefwd_ani](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/kubefwdani.gif)

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/06/26/16246803227515.jpg)

## 总结

善用工具可以提升效率，但并不是不可或缺的。

如果你有其他的工具，欢迎留言提出。