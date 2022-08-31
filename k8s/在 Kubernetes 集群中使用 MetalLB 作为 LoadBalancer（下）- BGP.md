在上一篇[《[[在 Kubernetes 集群中使用 MetalLB 作为 LoadBalancer（上）- Layer2]]》](https://atbug.com/load-balancer-service-with-metallb/)中，我们使用 MetalLB 的 Layer2 模式作为 LoadBalancer 的实现，将 Kubernetes 集群中的服务暴露到集群外。

还记得我们在 Configmap 中为 MetalLB 分配的 IP 地址池么？

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.30-192.168.1.49    
```

这里分配的 `192.168.1.30-192.168.1.49` IP 段正好是在笔者的家庭网络中，当我们用 `192.168.1.30` 可以成功访问服务。

之前有提过 Layer2 的缺点时还漏了一点，除了故障转移过程中对可用性有影响且存在单点网络瓶颈，还有就是客户端需要与地址池位于同一个子网（假如将地址池改为 `192.168.1.30-192.168.1.49`，服务将无法访问）。不过在实验环境或者像笔者这样的 homelab 环境来说，前两个都不算是问题，后一个则在网络配置时稍微麻烦一些。

虽然缺点很明显，但是 Layer2 模式有更强的通用性，不像 BGP 模式需要支持 BGP 的路由。但是这些都挡不住笔者的探（强）索（迫）欲（症），因为还有一个 OpenWrt 软路由运行在[我的 Proxmox 虚拟机](https://atbug.com/deploy-vm-on-proxmox-with-terraform/)中。这个 OpenWrt 以软路由的方式，通过 `192.168.1.2` 对外提供路由服务，通过安装路由软件套件来支持 BGP。

正式开始之前，先看下什么是 BPG 以及相关的术语。已经了解，或者觉得太抽象的同学可以直接跳过，待看完demo的再回头看。

## 什么是 BGP

BGP 是边界网关协议（Border Gateway Protocol）的缩写。

> 边界网关协议是互联网上一个核心的去中心化自治路由协议。它通过维护IP路由表或“前缀”表来实现自治系统（AS）之间的可达性，属于矢量路由协议。BGP不使用传统的内部网关协议（IGP）的指标，而使用基于路径、网络策略或规则集来决定路由。因此，它更适合被称为矢量性协议，而不是路由协议。
> 
> BGP的邻居关系（或称通信对端/对等实体，peer）是通过人工配置实现的，对等实体之间通过TCP端口179建立会话交换数据。BGP路由器会周期地发送19字节的保持存活（keep-alive）消息来维护连接（默认周期为60秒）。在各种路由协议中，只有BGP使用TCP作为传输层协议。
> 
> 同一个AS自治系统中的两个或多个对等实体之间运行的BGP被称为iBGP（Internal/Interior BGP）。归属不同的AS的对等实体之间运行的BGP称为eBGP（External/Exterior BGP）。在AS边界上与其他AS交换信息的路由器被称作边界路由器（border/edge router），边界路由器之间互为eBGP对端。在Cisco IOS中，iBGP通告的路由距离为200，优先级比eBGP和任何内部网关协议（IGP）通告的路由都低。其他的路由器实现中，优先级顺序也是eBGP高于IGP，而IGP又高于iBGP。
> 
> iBGP和eBGP的区别主要在于转发路由信息的行为。例如，从eBGP peer获得的路由信息会分发给所有iBGP peer和eBGP peer，但从iBGP peer获得的路由信息仅会分发给所有eBGP peer。所有的iBGP peer之间需要全互联。

这里提到了三个名词：自治系统（AS）、内部网关协议（IGP）和外部网关协议（EGP）。

### 自治系统 AS

我们看下来自维基百科的介绍：

> 自制系统（Autonomous system，缩写 AS），是指在互联网中，一个或多个实体管辖下的所有IP 网络和路由器的组合，它们对互联网执行共同的路由策略。自治系统编号都是16位长的整数，这最多能被分配给65536个自治系统。自治系统编号被分成两个范围。第一个范围是公开的ASN，从1到64511，它们可在互联网上使用；第二个范围是被称为私有编号的从64512到65535的那些，它们仅能在一个组织自己的网络内使用。

简单理解，电信、移动、联通都有自己的 AS 编号，且不只一个，有兴趣的可以查看维基百科中的[中国互联网骨干网](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E4%BA%92%E8%81%94%E7%BD%91%E9%AA%A8%E5%B9%B2%E7%BD%91)条目。

除了互联网公开的 ASN 以外，私有的编号可以在内部使用。比如我可以我的家庭网络中使用私有编号创建几个 AS。

### 内部路由协议 IGP

引用百科中的内容，不是本篇的重点因此不做过多介绍。

> 内部路由协议（Interior Gateway Protocol 缩写为 IGP）是指在一个自治系统（AS）内部所使用的一种路由协议。

### 外部网关协议 EGP

外部网关协议（Exterior Gateway Protocol，错写 EGP）是一个已经过时互联网路由协议。已由 BPG 取代。

### BPG 的由来

BPG 是为了替换 EGP 而创建的，而除了应用于 AS 外部，也可以应用在 AS 内部。因此又分为 EBGP 和 IBGP。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/03/07/16465782510374.jpg)

说了这么多可能有些抽象，直接上 demo 吧。

## Demo

环境还是使用之前的，按照预先设想我们希望创建两个 AS：65000 和 65001。前者作为路由器和客户端所在的 AS，而后者是我们集群服务 LoadBalancer IP 所在的 AS。

![bgp](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/03/07/bgp.png)

我们要先让 OpenWrt 支持 BGP。

### OpenWrt 支持 BGP

为了让 OpenWrt 支持 BGP，这里要用到路由软件套件 [Quagga](https://www.quagga.net/)。Quagga 提供了 OSPFv2、OSPFv3、RIP v1 v2、RIPng 和 BGP-4 的实现。

Quagga 架构由核心守护进程和 zebra 组成，后者作为底层 Unix 内核的抽象层，并通过 Unix 或者 TCP 向 Quagga 客户端提供 Zserv API。正是这些 Zserv 客户端实现了路由协议，并将路由的更新发送给 zebra 守护进程。当前 Zserv 的实现是：

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/03/07/20220306-at-232428.png)

Quagga 的守护进程可以通过网络可访问的 CLI（简称 _vty_）进行配置。 CLI 遵循与其他路由软件类似的风格。还额外提供了一个工具 _vtysh_，充当了所有守护进程的聚合前端，允许在一个地方管理所有 Quagga 守护进程的所有功能。

执行下面的命令即可完成安装：

```
$ opkg update && opkg install quagga quagga-zebra quagga-bgpd quagga-vtysh
```

成功安装之后，会自动启动并监听端口：

```
$ netstat -lantp | grep -e 'zebra\|bgpd'
tcp        0      0 0.0.0.0:2601            0.0.0.0:*               LISTEN      2984/zebra
tcp        0      0 0.0.0.0:2605            0.0.0.0:*               LISTEN      3000/bgpd
tcp        0      0 :::2601                 :::*                    LISTEN      2984/zebra
tcp        0      0 :::2605                 :::*                    LISTEN      3000/bgpd
```

这里并没有看到 _bpgd_ 用于接收路由信息而监听的 `179` 端口，这是因为该路由还没有分配 AS。不着急，让我们使用命令 `vtysh`进入 _vty_ 进行配置：

```
$ vtysh
OpenWrt# conf t
OpenWrt(config)# router bgp 65000
OpenWrt(config-router)# neighbor 192.168.1.5 remote-as 65001
OpenWrt(config-router)# neighbor 192.168.1.5 description ubuntu-dev1
OpenWrt(config-router)# neighbor 192.168.1.6 remote-as 65001
OpenWrt(config-router)# neighbor 192.168.1.6 description ubuntu-dev2
OpenWrt(config-router)# exit
OpenWrt(config)# exit
```

在 vty 中使用 `show ip bgp summary` 命令查看：

```
OpenWrt# show ip bgp summary
BGP router identifier 192.168.1.2, local AS number 65000
RIB entries 0, using 0 bytes of memory
Peers 2, using 18 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.5     4 65001       0       0        0    0    0 never    Active
192.168.1.6     4 65001       0       0        0    0    0 never    Active

Total number of neighbors 2

Total num. Established sessions 0
Total num. of routes received     0
```

此时我们再去查看端口监听，就可以看到 bgpd 已经在监听 `179` 端口了：

```
$ netstat -lantp | grep -e 'zebra\|bgpd'
tcp        0      0 0.0.0.0:179             0.0.0.0:*               LISTEN      3000/bgpd
tcp        0      0 0.0.0.0:2601            0.0.0.0:*               LISTEN      2984/zebra
tcp        0      0 0.0.0.0:2605            0.0.0.0:*               LISTEN      3000/bgpd
tcp        0      0 :::179                  :::*                    LISTEN      3000/bgpd
tcp        0      0 :::2601                 :::*                    LISTEN      2984/zebra
tcp        0      0 :::2605                 :::*                    LISTEN      3000/bgpd
```

BGP 的路由设置好之后，就是 MetalLB 的部分了。

### MetalLB BGP 模式

我们更新一下 configmap：

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    peers:
    - peer-address: 192.168.1.2
      peer-asn: 65000
      my-asn: 65001
    address-pools:
    - name: default
      protocol: bgp
      addresses:
      - 192.168.0.30-192.168.0.49    
```

更新之后，你会发现 _Service_ 的 _EXTERNAL-IP_ 并没有重新分配，MetalLB 的控制器并没有自动生效配置。我们删除控制器 pod 进行重启：

```
$ kubectl delete po -n metallb-system -l app=metallb,component=controller
pod "controller-66445f859d-vss2t" deleted
```

此时可以看到 _Service_ 分配到了新的 IP：

```
$ kubectl get svc -n default
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
kubernetes   ClusterIP      10.43.0.1       <none>         443/TCP          25m
nginx-lb     LoadBalancer   10.43.188.185   192.168.0.30   8080:30381/TCP   21m
nginx2-lb    LoadBalancer   10.43.208.169   192.168.0.31   8080:32319/TCP   21m
```

检查 speaker POD 的日志，可以看到与 peer 192.168.1.2 之间的通信已经开始，并对外发布了 IP 地址的公告：

```
{"caller":"level.go:63","configmap":"metallb-system/config","event":"peerAdded","level":"info","msg":"peer configured, starting BGP session","peer":"192.168.1.2","ts":"2022-03-06T22:56:17.336335657Z"}
{"caller":"level.go:63","configmap":"metallb-system/config","event":"configLoaded","level":"info","msg":"config (re)loaded","ts":"2022-03-06T22:56:17.336366122Z"}
struct { Version uint8; ASN16 uint16; HoldTime uint16; RouterID uint32; OptsLen uint8 }{Version:0x4, ASN16:0xfde8, HoldTime:0xb4, RouterID:0xc0a80102, OptsLen:0x1e}
{"caller":"level.go:63","event":"sessionUp","level":"info","localASN":65001,"msg":"BGP session established","peer":"192.168.1.2:179","peerASN":65000,"ts":"2022-03-06T22:56:17.337341549Z"}
{"caller":"level.go:63","event":"updatedAdvertisements","ips":["192.168.0.30"],"level":"info","msg":"making advertisements using BGP","numAds":1,"pool":"default","protocol":"bgp","service":"default/nginx-lb","ts":"2022-03-06T22:56:17.341939983Z"}
{"caller":"level.go:63","event":"serviceAnnounced","ips":["192.168.0.30"],"level":"info","msg":"service has IP, announcing","pool":"default","protocol":"bgp","service":"default/nginx-lb","ts":"2022-03-06T22:56:17.341987657Z"}
{"caller":"level.go:63","event":"updatedAdvertisements","ips":["192.168.0.31"],"level":"info","msg":"making advertisements using BGP","numAds":1,"pool":"default","protocol":"bgp","service":"default/nginx2-lb","ts":"2022-03-06T22:56:17.342041554Z"}
{"caller":"level.go:63","event":"serviceAnnounced","ips":["192.168.0.31"],"level":"info","msg":"service has IP, announcing","pool":"default","protocol":"bgp","service":"default/nginx2-lb","ts":"2022-03-06T22:56:17.342056076Z"}
```

然后可以在 _vty_ 中查看路由表：

```
OpenWrt# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, P - PIM, A - Babel, N - NHRP,
       > - selected route, * - FIB route

K>* 0.0.0.0/0 via 192.168.1.1, br-lan
C>* 127.0.0.0/8 is directly connected, lo
B>* 192.168.0.30/32 [20/0] via 192.168.1.5, br-lan, 00:00:06
B>* 192.168.0.31/32 [20/0] via 192.168.1.5, br-lan, 00:00:06
C>* 192.168.1.0/24 is directly connected, br-lan
```

从表中我们可以找到 `192.168.0.30/32` 和 `192.168.0.31/32` 两条 BGP 的路由。

### 测试

我们使用新的 IP 访问服务：

```
$ curl -I 192.168.0.30:8080
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Sun, 06 Mar 2022 23:10:33 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
Connection: keep-alive
ETag: "61f01158-267"
Accept-Ranges: bytes
```

## 总结

至此，我们已经试过了 MetalLB 的两种模式：Layer2 有很强的通用性，不需要其他任何的依赖，但是缺点也明显；BGP 模式除了依赖支持 BGP 的路由，其他方面则没有任何限制，并且没有可用性的问题。

BGP 应该是 LoadBalancer 的终极模式，但是 Layer2 也不是毫无用处。大家还是要看使用的场景来理性的选择，比如 homelab 中使用我会选择 Layer2 模式。