## TL;DR

在本篇，我们分别使用了 Kubernetes 原生的网络策略和 Cilium 的网络策略实现了 Pod 网络层面的隔离。不同的是，前者只提供了基于 L3/4 的网络策略；后者支持 L3/4、L7 的网络策略。

通过网络策略来提升网络安全，可以极大降低了实现和维护的成本，同时对系统几乎没有影响。

尤其是基于 eBPF 技术的 Cilium，解决了内核扩展性不足的问题，从内核层面为工作负载提供安全可靠、可观测的网络连接。

## 目录

-   [TL;DR](https://atbug.com/#tldr)
-   [目录](https://atbug.com/#%E7%9B%AE%E5%BD%95)
-   [背景](https://atbug.com/#%E8%83%8C%E6%99%AF)
-   [示例应用](https://atbug.com/#%E7%A4%BA%E4%BE%8B%E5%BA%94%E7%94%A8)
-   [Kubernetes 网络策略](https://atbug.com/#kubernetes-%E7%BD%91%E7%BB%9C%E7%AD%96%E7%95%A5)
    -   [测试](https://atbug.com/#%E6%B5%8B%E8%AF%95)
    -   [思考](https://atbug.com/#%E6%80%9D%E8%80%83)
-   [Cilium 网络策略](https://atbug.com/#cilium-%E7%BD%91%E7%BB%9C%E7%AD%96%E7%95%A5)
    -   [Cilium 简介](https://atbug.com/#cilium-%E7%AE%80%E4%BB%8B)
    -   [测试](https://atbug.com/#%E6%B5%8B%E8%AF%95-1)

## 背景

为什么说 Kubernetes 网络存在安全隐患？集群中的 Pod 默认是未隔离的，也就是 Pod 之间的网络是互通的，可以互相通信的。

这里就会有问题，比如由于数据敏感服务 B 只允许特定的服务 A 才能访问，而服务 C 无法访问 B。要禁止服务 C 对服务 B 的访问，可以有几种方案：

-   在 SDK 中提供通用的解决方案，实现白名单的功能。首先请求要带有来源的标识，然后服务端可以接收规则设置放行特定标识的请求，拒绝其他的请求。
-   [[k8s/云原生]]的解决方案，使用服务网格的 RBAC、mTLS 功能。RBAC 实现原理与应用层的 SDK 方案类似，但是属于基础设施层的抽象通用方案；mTLS 则会更加复杂一些，在连接握手阶段进行身份验证，涉及证书的签发、验证等操作。

以上两种方案各有利弊：

-   SDK 的方案实现简单，但是规模较大的系统会面临升级推广困难、多语言支持成本高等问题。
-   服务网格的方案是基础设施层的通用方案，天生支持多语言。但是对于未落地网格的用户来说，架构变化大，成本高。如果单纯为了解决安全问题，使用网格方案性价比又很低，且不说现有网格实现等落地难度大及后期的使用维护成本高。

继续向基础设施下层找方案，从网络层入手。Kubernetes 提供了的[网络策略 _NetworkPolicy_](https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/)，则可以实现“网络层面的隔离”。

## 示例应用

在进一步演示 _NetworkPolicy_ 的方案之前，先介绍用于演示的示例应用。我们使用 Cilium 在互动教程 [Cilium getting started](https://play.instruqt.com/isovalent/tracks/cilium-getting-started) 中使用的“星球大战”场景。

这里有三个应用，星战迷估计不会陌生：

-   死星 _deathstar_：在 `80` 端口提供 web 服务，有 2 个 副本，通过 Kubernetes Service 的负载均衡为**帝国战机**对外提供”登陆“服务。
-   钛战机 _tiefighter_：执行登陆请求。
-   X翼战机 _xwing_：执行登陆请求。

![desc](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/02/13/desc.png)

> 如图所示，我们使用了 _Label_ 对三个应用进行了标识：`org` 和 `class`。在执行网络策略时，我们会使用这两个标签识别负载。

```
# app.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: deathstar
  labels:
    app.kubernetes.io/name: deathstar
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    org: empire
    class: deathstar
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deathstar
  labels:
    app.kubernetes.io/name: deathstar
spec:
  replicas: 2
  selector:
    matchLabels:
      org: empire
      class: deathstar
  template:
    metadata:
      labels:
        org: empire
        class: deathstar
        app.kubernetes.io/name: deathstar
    spec:
      containers:
      - name: deathstar
        image: docker.io/cilium/starwars
---
apiVersion: v1
kind: Pod
metadata:
  name: tiefighter
  labels:
    org: empire
    class: tiefighter
    app.kubernetes.io/name: tiefighter
spec:
  containers:
  - name: spaceship
    image: docker.io/tgraf/netperf
---
apiVersion: v1
kind: Pod
metadata:
  name: xwing
  labels:
    app.kubernetes.io/name: xwing
    org: alliance
    class: xwing
spec:
  containers:
  - name: spaceship
    image: docker.io/tgraf/netperf
```

## Kubernetes 网络策略

可以通过[官方文档](https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/)获取更多详细信息，这里我们直接放出配置：

```
# native/networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      org: empire
      class: deathstar
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          org: empire
    ports:
    - protocol: TCP
      port: 80
```

-   `podSelector` ：表示要应用网络策略的工作负载均衡，通过 _label_ 选择到了 _deathstar_ 的 2 个 Pod。
-   `policyTypes` ：表示流量的类型，可以是 `Ingress` 或 `Egress` 或两者兼具。这里使用 `Ingress`，表示对选择的 _deathstar_ Pod 的入站流量执行规则。
-   `ingress.from`：表示流量的来源工作负载，也是使用 `podSelector` 和 _Label_ 进行选择，这里选中了 `org=empire` 也就是所有“帝国的战机”。
-   `ingress.ports`：表示流量的进入端口，这里列出了 _deathstar_ 的服务端口。

接下来，我们测试下。

### 测试

先准备环境，我们使用 [K3s](https://k3s.io/) 作为 Kubernetes 环境。但由于 K3s 默认的 CNI 插件 Flannel 不支持网络策略，我们需要换个插件，这里选择 [Calico](https://www.tigera.io/project-calico/)，即 K3s + Calico 的方案。

先创建一个单节点的集群：

```
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=10.42.0.0/16 --disable-network-policy --disable=traefik" sh -
```

此时，所有的 Pod 都处于 `Pending` 状态，因为还需要安装 Calico：

```
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

待 Calico 成功运行后，所有的 Pod 也会成功运行。

接下来就是部署应用：

```
kubectl apply -f app.yaml
```

**执行策略前**，执行下面的命令看看“战机能否登陆死星”：

```
kubectl exec tiefighter -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed

kubectl exec xwing -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
```

从结果来看，两种 ”战机“（Pod 负载）都可以访问 _deathstar_ 服务。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/02/13/test.png)

此时执行网络策略：

```
kubectl apply -f native/networkpolicy.yaml
```

再次尝试”登陆“，_xwing_ 的登陆请求会停在那（需要使用 _ctrl+c_ 退出，或者请求时加上 `--connect-timeout 2`）。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/02/13/3.png)

### 思考

使用 Kubernetes 网络策略实现了我们想要的，从网络层面为服务增加了白名单的功能，这种方案没有改造成本，对系统也几乎无影响。

Cilium 还没出场就结束了？我们继续看：

有时我们的服务会对外暴露一些管理端点，由系统调用执行一些管理上的操作，比如热更新、重启等。这些端点是不允许普通服务来调用，否则会造成严重的后果。

比如示例中，_tiefighter_ 访问了 _deathstar_ 的管理端点 `/exhaust-port`：

```
kubectl exec tiefighter -- curl -s -XPUT deathstar.default.svc.cluster.local/v1/exhaust-port
Panic: deathstar exploded

goroutine 1 [running]:
main.HandleGarbage(0x2080c3f50, 0x2, 0x4, 0x425c0, 0x5, 0xa)
        /code/src/github.com/empire/deathstar/
        temp/main.go:9 +0x64
main.main()
        /code/src/github.com/empire/deathstar/
        temp/main.go:5 +0x85
```

出现了 _Panic_ 错误，检查 Pod 你会发现 _dealthstar_ 挂了。

**Kubernetes 的网络策略仅能工作在 L3/4 层，对 L7 层就无能为力了。**

还是要请出 Cilium。

## Cilium 网络策略

**由于 Cilium 涉及了 Linux 内核、网络等众多知识点，要讲清实现原理篇幅极大。故这里仅摘取了官网的介绍，后期希望有时间再写一篇关于实现的。**

### Cilium 简介

> [Cilium](https://cilium.io/) 是一个开源软件，用于提供、保护和观察容器工作负载（[[k8s/云原生]]）之间的网络连接，由革命性的内核技术 [eBPF](https://ebpf.io/) 推动。

**eBPF 是什么？**

> Linux 内核一直是实现监控/可观测性、网络和安全功能的理想地方。 不过很多情况下这并非易事，因为这些工作需要修改内核源码或加载内核模块， 最终实现形式是在已有的层层抽象之上叠加新的抽象。 eBPF 是一项革命性技术，它能在内核中运行沙箱程序（sandbox programs）， 而无需修改内核源码或者加载内核模块。

> 将 Linux 内核变成可编程之后，就能基于现有的（而非增加新的）抽象层来打造更加智能、 功能更加丰富的基础设施软件，而不会增加系统的复杂度，也不会牺牲执行效率和安全性。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/02/13/16446950885360.jpg)

我们来看下 Cilium 的网络策略：

```
# cilium/networkpolicy-L4.yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "rule1"
spec:
  description: "L7 policy to restrict access to specific HTTP call"
  endpointSelector:
    matchLabels:
      org: empire
      class: deathstar
  ingress:
  - fromEndpoints:
    - matchLabels:
        org: empire
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
```

与 Kubernetes 的原生网络策略差异不大，参考前面的介绍也都看懂，我们直接进入测试。

### 测试

由于 Cilium 本身就实现了 CNI，所以之前的集群就不能用了，先卸载集群：

```
k3s-uninstall.sh
# ！！！切记要清理之前的 cni 插件
sudo rm -rf /etc/cni/net.d
```

还是使用同样的命令创建单节点的集群：

```
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=10.42.0.0/16 --disable-network-policy --disable=traefik" sh -

# cilium 会使用该变量
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

接下来安装 Cilium CLI：

```
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-amd64.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz{,.sha256sum}

cilium version
cilium-cli: v0.10.2 compiled with go1.17.6 on linux/amd64
cilium image (default): v1.11.1
cilium image (stable): v1.11.1
cilium image (running): unknown. Unable to obtain cilium version, no cilium pods found in namespace "kube-system"
```

安装 Cilium 到集群：

待 Cilium 成功运行：

```
cilium status
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         disabled
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium             Running: 1
                  cilium-operator    Running: 1
Cluster Pods:     3/3 managed by Cilium
Image versions    cilium-operator    quay.io/cilium/operator-generic:v1.11.1@sha256:977240a4783c7be821e215ead515da3093a10f4a7baea9f803511a2c2b44a235: 1
                  cilium             quay.io/cilium/cilium:v1.11.1@sha256:251ff274acf22fd2067b29a31e9fda94253d2961c061577203621583d7e85bd2: 1
```

部署应用：

```
kubectl apply -f app.yaml
```

待应用启动后测试服务调用：

```
kubectl exec tiefighter -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
kubectl exec xwing -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
```

执行 L4 网络策略：

```
kubectl apply -f cilium/networkpolicy-L4.yaml
```

再次尝试“登陆”死星，_xwing_ 战机同样无法登陆，说明 L4 层的规则生效。

我们再尝试 L7 层的规则：

```
# cilium/networkpolicy-L7.yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "rule1"
spec:
  description: "L7 policy to restrict access to specific HTTP call"
  endpointSelector:
    matchLabels:
      org: empire
      class: deathstar
  ingress:
  - fromEndpoints:
    - matchLabels:
        org: empire
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "POST"
          path: "/v1/request-landing"
```

执行规则：

```
kubectl apply -f cilium/networkpolicy-L7.yaml
```

这回，使用 _tiefighter_ 调用死星的管理接口：

```
kubectl exec tiefighter -- curl -s -XPUT deathstar.default.svc.cluster.local/v1/exhaust-port
Access denied
# 登陆接口工作正常
kubectl exec tiefighter -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
```

这回返回了 _Access denied_，说明 L7 层的规则生效了。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/02/13/4.png)