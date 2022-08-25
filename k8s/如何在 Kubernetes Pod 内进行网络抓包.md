使用 Kubernetes 时，经常会遇到一些棘手的网络问题需要对 Pod 内的流量进行抓包分析。然而所使用的镜像一般不会带有 _tcpdump_ 命令，过去常用的做法简单直接暴力：登录到节点所在节点，使用 _root_ 账号进入容器，然后安装 _tcpdump_。抓到的包有时还需要拉回本地，使用 Wireshark 进行分析。而且整个过程非常繁琐，跨越几个环境。

正好前几天也做了一次抓包问题排查，这次就介绍一下快速进行网络抓包的几种方法。

## TL;DR

几种方法各有优缺点，且都不建议在生产环境使用。假如必须使用，个人倾向于 kubectl debug 临时容器的方案，但这个方案也有不足。

-   使用额外容器：这种方案为了 Pod 添加一个额外的容器，使用了静态编译的 tcpdump 进行抓取，借助了多容器共享网络空间的特性，适合 distroless 容器。缺点是需要修改原来的 Pod，调式容器重启会引起 Pod 重启。
-   _kubectl plugin ksniff_：一个 kubectl 插件。支持特权和非特权容器，可以将捕获内容重定向到 wireshark 或者 tshark。非特权容器的实现会稍微复杂。
-   _kubectl debug_ 临时容器：该方案对于 distroless 容器有很好的支持，临时容器退出后也不会导致 Pod 重启。缺点是 1.23 的版本临时容器才进入 beta 阶段；而且笔者在将捕获的数据重定向到本地的 Wireshark 时会报数据格式不支持的错误。

## 环境

使用 k3d 创建 k3s 集群，这里版本选择 1.23:

```
$ k3d cluster create test --image rancher/k3s:v1.23.4-k3s1
```

抓包的对象使用 [Pipy](https://github.com/flomesh-io/pipy) 运行的一个 _echo_ 服务（返回请求的 body 内容）：

```
$ kubectl run echo --image addozhang/echo-server --image-pull-policy IfNotPresent
```

为了方便访问，创建一个 _NodePort_ Service：

```
$ kubectl expose pod echo --name echo --port 8080 --type NodePort
```

## 挂载容器

在之前的文章我们介绍[调试 distroless 容器的几种方法](https://atbug.com/debug-distroless-container-on-kubernetes/)时曾用过修改 Pod 添加额外容器的方式，新的容器使用镜像 `addozhang/static-dump` 镜像。这个镜像中加入了**静态编译**的 _tcpdump_。

修改后的 Pod：

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: echo
  name: echo
spec:
  containers:
  - image: addozhang/echo-server
    imagePullPolicy: IfNotPresent
    name: echo
    resources: {} 
  - image: addozhang/static-dump
    imagePullPolicy: IfNotPresent
    name: sniff
    command: ['sleep', '1d']
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

重新部署后，就可以使用下面命令将抓取网络包并重定向到本地的 Wireshark：

```
$ kubectl exec -i echo -c sniff -- /static-tcpdump -i eth0 -U -w - | wireshark -k -i -
```

![debug-container](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/04/03/debugcontainer.gif)

## kubectl plugin ksniff

[ksniff](https://github.com/eldadru/ksniff) 是一个 kubectl 插件，利用 _tcpdump_ 和 _Wireshark_ 对 Pod 中的网络包实现远程抓取。使用这种方法既可以借助 Wireshark 的强大功能，又能降低对 Pod 的影响。

ksniff 的实现是上传一个**静态编译的**_tcpdump_ 到 Pod 中，然后将 _tcpdump_ 的输出重定向到本地的 Wireshark 进行调试。

核心可以理解成 `tcpdump -w - | wireshark -k -i -`，与前面使用 debug 容器的方案类似。

### 安装

通过 [krew](https://github.com/GoogleContainerTools/krew) 安装：

```
$ kubectl krew install sniff
```

或者下载发布包，手动安装：

```
$ unzip ksniff.zip
$ make install
```

### 特权模式容器

使用说明参考 [ksniff 官方说明](https://github.com/eldadru/ksniff#usage)，这里我们只需要执行如下命令，默认就会重定向到 Wireshark，不需要显示地指定：

```
$ kubectl sniff echo -n default -f "port 8080"
```

![ksniff](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/04/03/ksniff.gif)

除了使用 Wireshark，可以使用其命令行模式的 _tshark_：

```
$ kubectl sniff echo -n default -f "port 8080" -o - | tshark -r -
```

### 非特权模式容器

对于无特权的容器，就无法使用上面的方法了，会收到如下的错误提示：

```
INFO[0000] command: '[/tmp/static-tcpdump -i any -U -w - port 8080]' executing successfully exitCode: '1', stdErr :'static-tcpdump: any: You don't have permission to capture on that device
(socket: Operation not permitted)
```

不过，Ksniff 对此类容器也提供了支持。通过添加 `-p` 参数，ksniff 会创建一个新的可以访问节点上 Docker Daemon 的 pod，然后将容器附加到目标容器的网络命名空间，并执行报文捕获。

注意，笔者使用的是 k3s 的环境，执行命令时需要通过参数指定 Docker Daemon 的 socket 地址 `--socket /run/k3s/containerd/containerd.sock`

```
$ kubectl sniff echo -n default -f "port 8080" --socket /run/k3s/containerd/containerd.sock -p | wireshark -k -i -
```

![ksniff-priviledged](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/04/03/ksniffpriviledged.gif)

## kubectl debug 临时容器

接下来也是之前介绍过的 _kubectl debug_ ，也就是为 Pod 添加[临时容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/ephemeral-containers/)。

同样我们可以通过这种方法对 Pod 的网络进行抓包，临时容器我们使用 `addozhang/static-dump` 镜像。

```
$ kubectl debug -i echo --image addozhang/static-dump --target echo -- /static-tcpdump -i eth0 
```

![ephermeral-sniff-stdout](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/04/03/ephermeralsniffstdout.gif)

大家能发现这里将捕获的内容直接输出在标准输出中了，而不是重定向到本地的 Wireshark。

原本临时容器应该是其中最接近完美的方案：不需上传任何文件目标容器、无需修改 Pod、无需重启、无需特权、支持 distroless 容器。然而，当尝试重定向到 Wireshark 或者 tshark 的时候，会遇到 _Data written to the pipe is neither in a supported pcap format nor in pcapng format._ 问题。

最后经过一番折腾，也未能解决该问题。有解决了问题的朋友，也麻烦评论告知一下。感谢！