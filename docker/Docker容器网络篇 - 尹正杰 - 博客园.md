　　　　　　　　　　　　　　**Docker容器网络篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Docker的网络模型概述**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017194216297-54287928.png)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
如上图所示，Docker有四种网络模型：
　　封闭式网络(Closed container)：
　　　　封闭式容器，只有本地回环接口(Loopback interface，和咱们服务器看到的lo接口类似)，无法与外界进行通信。

　　桥接式网络(Bridge container A):
　　　　桥接容器，除了有一块本地回环接口(Loopback interface)外，还有一块私有接口(Private interface)通过容器虚拟接口(Container virtual interface)连接到桥接虚拟接口(Docker bridge virtual interface)，之后通过逻辑主机接口(Logical host interface)连接到主机物理网络(Physical network interface)。　　　　桥接网卡默认会分配到172.17.0.0/16的IP地址段。　　　　如果我们在创建容器时没有指定网络模型，默认就是(Nat)桥接网络，这也就是为什么我们在登录到一个容器后，发现IP地址段都在172.17.0.0/16网段的原因啦。

　　联盟式网络(Joined container A | Joined container B):
　　　　每个容器都各有一部分名称空间(Mount,PID,User)，另外一部分名称空间是共享的(UTS,Net,IPC)。由于他们的网络是共享的，因此各个容器可以通过本地回环接口(Loopback interface)进行通信。除了共享同一组本地回环接口(Loopback interface)外，还有一块一块私有接口(Private interface)通过联合容器虚拟接口(Joined container virtual interface)连接到桥接虚拟接口(Docker bridge virtual interface)，之后通过逻辑主机接口(Logical host interface)连接到主机物理网络(Physical network interface)。
　　　　开放式容器(Open container)：
　　　　比联盟式网络更开放，我们知道联盟式网络是多个容器共享网络(Net),而开放式容器(Open contaner)就直接共享了宿主机的名称空间。因此物理网卡有多少个，那么该容器就能看到多少网卡信息。我们可以说Open container是联盟式容器的衍生。
　　　　
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.**容器虚拟化网络概述****

**1>.查看docker支持的网络模型**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
33d001d8b94d        bridge              bridge              local　　　　　　#如果我们创建一个容器式未指定网络模型，默认就是桥接式网络哟~
9f539144f682        host                host                local
e10670abb710        none                null                local
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.查看桥接式网络元数据信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "33d001d8b94d4080411e06c711a1b6d322115aebbe1253ecef58a9a70e05bdd7",
        "Created": "2019-10-18T17:27:49.282236251+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",　　　　　　#这里就是默认的桥接式网络的网段地址，既然式默认那自然式可以修改的。
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",　　　　　　#看这里，告诉咱们bridge默认的网卡名称为"docker0"
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.使用ip命令网络名称空间(netns)来模拟容器间通信**

**1>.查看帮助信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# rpm -q iproute
iproute-4.11.0-14.el7.x86_64
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ip netns help　　　　　　#注意：当我们使用ip命令去管理网络名称时，其它名称都是共享的，这和容器所说的六个名称空间都是隔离的有所不同哟~
Usage: ip netns list
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id
[root@node101.yinzhengjie.org.cn ~]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.添加2个网络名称空间**

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns add r1　　　　　　　　　　　　　　#添加一个r1网络名称空间
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns add r2
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns list                                     #查看已经存在的网络名称空间列表
r2
r1
[root@node103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig -a　　　　　　#我们发现创建的r1网络名称空间并没有网卡，仅有本地回环地址，目前还没有绑定任何网络接口设备。
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig -a                 #和r1同理。
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.创建虚拟网卡对**

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip link help　　　　　　　　　　#查看该命令的帮助信息
Usage: ip link add [link DEV] [ name ] NAME
                   [ txqueuelen PACKETS ]
                   [ address LLADDR ]
                   [ broadcast LLADDR ]
                   [ mtu MTU ] [index IDX ]
                   [ numtxqueues QUEUE_COUNT ]
                   [ numrxqueues QUEUE_COUNT ]
                   type TYPE [ ARGS ]

       ip link delete { DEVICE | dev DEVICE | group DEVGROUP } type TYPE [ ARGS ]

       ip link set { DEVICE | dev DEVICE | group DEVGROUP }
                      [ { up | down } ]
                      [ type TYPE ARGS ]
                      [ arp { on | off } ]
                      [ dynamic { on | off } ]
                      [ multicast { on | off } ]
                      [ allmulticast { on | off } ]
                      [ promisc { on | off } ]
                      [ trailers { on | off } ]
                      [ carrier { on | off } ]
                      [ txqueuelen PACKETS ]
                      [ name NEWNAME ]
                      [ address LLADDR ]
                      [ broadcast LLADDR ]
                      [ mtu MTU ]
                      [ netns { PID | NAME } ]
                      [ link-netnsid ID ]
              [ alias NAME ]
                      [ vf NUM [ mac LLADDR ]
                   [ vlan VLANID [ qos VLAN-QOS ] [ proto VLAN-PROTO ] ]
                   [ rate TXRATE ]
                   [ max_tx_rate TXRATE ]
                   [ min_tx_rate TXRATE ]
                   [ spoofchk { on | off} ]
                   [ query_rss { on | off} ]
                   [ state { auto | enable | disable} ] ]
                   [ trust { on | off} ] ]
                   [ node_guid { eui64 } ]
                   [ port_guid { eui64 } ]
              [ xdp { off |
                  object FILE [ section NAME ] [ verbose ] |
                  pinned FILE } ]
              [ master DEVICE ][ vrf NAME ]
              [ nomaster ]
              [ addrgenmode { eui64 | none | stable_secret | random } ]
                      [ protodown { on | off } ]

       ip link show [ DEVICE | group GROUP ] [up] [master DEV] [vrf NAME] [type TYPE]

       ip link xstats type TYPE [ ARGS ]

       ip link afstats [ dev DEVICE ]

       ip link help [ TYPE ]

TYPE := { vlan | veth | vcan | dummy | ifb | macvlan | macvtap |
          bridge | bond | team | ipoib | ip6tnl | ipip | sit | vxlan |
          gre | gretap | ip6gre | ip6gretap | vti | nlmon | team_slave |
          bond_slave | ipvlan | geneve | bridge_slave | vrf | macsec }
[root@node103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip link add name veth1.1 type veth peer name veth1.2　　　　　　#创建一对类型为虚拟以太网网卡(veth)，两端名称分别为veth1.1和veth1.2
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip link show　　　　　　#我们可以看到veth1.1@veth1.2表示veth1.1和veth1.2为一对网卡，目前他们都在宿主机且状态为DOWN。
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
4: veth1.2@veth1.1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 36:1e:54:37:69:78 brd ff:ff:ff:ff:ff:ff
5: veth1.1@veth1.2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 52:00:e0:83:48:cd brd ff:ff:ff:ff:ff:ff
[root@node103.yinzhengjie.org.cn ~]#
```

\[root@node103.yinzhengjie.org.cn ~\]# ip link add name veth1.1 type veth peer name veth1.2　　　　　　#创建一对类型为虚拟以太网网卡(veth)，两端名称分别为veth1.1和veth1.2

**4>.将虚拟网卡移动到指定名称空间并实现网络互通**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
4: veth1.2@veth1.1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 36:1e:54:37:69:78 brd ff:ff:ff:ff:ff:ff
5: veth1.1@veth1.2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:00:e0:83:48:cd brd ff:ff:ff:ff:ff:ff
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip link set dev veth1.2 netns r2　　　　#我们将veth1.2这块虚拟网卡移动到r2名称空间。
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip link show　　　　　　　　　　#此时，我们发现veth1.2不见啦，因此我们可以说一块网卡只能归属一个名称空间，注意观察此时veth1.1的名称由"veth1.1@veth1.2"变为"veth1.1@if4"了哟~
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
5: veth1.1@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:00:e0:83:48:cd brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ip link set dev veth1.2 netns r2　　　　#我们将veth1.2这块虚拟网卡移动到r2名称空间。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig -a　　　　#于此同时，我们上面说将宿主机的veth1.2移动到了r2名称空间里啦，那我们就来看看，发现的确存在。
lo: flags=8<LOOPBACK> mtu 65536
loop txqueuelen 1000 (Local Loopback)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

veth1.2: flags=4098<BROADCAST,MULTICAST> mtu 1500　　　　　　　　　　　　　　#果不其然，这里的确有该虚拟网卡设备呢！
ether 36:1e:54:37:69:78 txqueuelen 1000 (Ethernet)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ip link set dev veth1.2 name eth0　　　　#于此同时我们将veth1.2更名为eth0,便于规范化。
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig -a　　　　　　#veth1.2成功更名为eth0,默认网卡是没有激活的，因此我们需要使用-a选项查看哟
eth0: flags=4098<BROADCAST,MULTICAST> mtu 1500
ether 36:1e:54:37:69:78 txqueuelen 1000 (Ethernet)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

lo: flags=8<LOOPBACK> mtu 65536
loop txqueuelen 1000 (Local Loopback)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

[root@node103.yinzhengjie.org.cn ~]#
```

\[root@node103.yinzhengjie.org.cn ~\]# ip netns exec r2 ip link set dev veth1.2 name eth0　　　　#于此同时我们将veth1.2更名为eth0,便于规范化。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
5: veth1.1@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:00:e0:83:48:cd brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ifconfig veth1.1 10.1.0.1/24 up　　　　　　#给宿主机端的veth1.1虚拟网卡分配临时IP地址并激活该网卡
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ifconfig 　　　　　　　　　　　　　　　　　　　#此时我们可以使用ifconfig命令看到veth1.1的网卡配置信息啦
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 08:00:27:ef:75:60  txqueuelen 1000  (Ethernet)
        RX packets 15  bytes 3188 (3.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 80  bytes 8168 (7.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.103  netmask 255.255.255.0  broadcast 172.30.1.255
        ether 08:00:27:3a:da:a7  txqueuelen 1000  (Ethernet)
        RX packets 3905  bytes 300723 (293.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13059  bytes 1199271 (1.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth1.1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.1.0.1  netmask 255.255.255.0  broadcast 10.1.0.255
        ether 52:00:e0:83:48:cd  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ifconfig veth1.1 10.1.0.1/24 up　　　　　　#给宿主机端的veth1.1虚拟网卡分配临时IP地址并激活该网卡

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig -a
eth0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 36:1e:54:37:69:78  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig eth0 10.1.0.102/24 up　　　　#我们为r2网络名称空间的eth0虚拟网卡分配临时IP地址并激活
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.102  netmask 255.255.255.0  broadcast 10.1.0.255
        inet6 fe80::341e:54ff:fe37:6978  prefixlen 64  scopeid 0x20<link>
        ether 36:1e:54:37:69:78  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6  bytes 516 (516.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ip netns exec r2 ifconfig eth0 10.1.0.102/24 up　　　　#我们为r2网络名称空间的eth0虚拟网卡分配临时IP地址并激活

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ifconfig 　　　　　　　　　　　　　　#宿主机已激活网卡信息
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 10.0.2.15 netmask 255.255.255.0 broadcast 10.0.2.255
ether 08:00:27:ef:75:60 txqueuelen 1000 (Ethernet)
RX packets 15 bytes 3188 (3.1 KiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 80 bytes 8168 (7.9 KiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 172.30.1.103 netmask 255.255.255.0 broadcast 172.30.1.255
ether 08:00:27:3a:da:a7 txqueuelen 1000 (Ethernet)
RX packets 4280 bytes 331527 (323.7 KiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 13362 bytes 1235909 (1.1 MiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536
inet 127.0.0.1 netmask 255.0.0.0
loop txqueuelen 1000 (Local Loopback)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

veth1.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 10.1.0.1 netmask 255.255.255.0 broadcast 10.1.0.255
ether 52:00:e0:83:48:cd txqueuelen 1000 (Ethernet)
RX packets 18 bytes 1524 (1.4 KiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 10 bytes 868 (868.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig 　　　　　　　　#r2名称空间已激活网卡信息
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 10.1.0.102 netmask 255.255.255.0 broadcast 10.1.0.255
inet6 fe80::341e:54ff:fe37:6978 prefixlen 64 scopeid 0x20<link>
ether 36:1e:54:37:69:78 txqueuelen 1000 (Ethernet)
RX packets 10 bytes 868 (868.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 18 bytes 1524 (1.4 KiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]#


[root@node103.yinzhengjie.org.cn ~]# ping 10.1.0.102　　　　　　　　　　　　#我们发现宿主机可以和r2名称空间的eth0虚拟网卡通信啦。
PING 10.1.0.102 (10.1.0.102) 56(84) bytes of data.
64 bytes from 10.1.0.102: icmp_seq=1 ttl=64 time=0.037 ms
64 bytes from 10.1.0.102: icmp_seq=2 ttl=64 time=0.018 ms
64 bytes from 10.1.0.102: icmp_seq=3 ttl=64 time=0.022 ms
64 bytes from 10.1.0.102: icmp_seq=4 ttl=64 time=0.020 ms
64 bytes from 10.1.0.102: icmp_seq=5 ttl=64 time=0.019 ms
64 bytes from 10.1.0.102: icmp_seq=6 ttl=64 time=0.040 ms
64 bytes from 10.1.0.102: icmp_seq=7 ttl=64 time=0.022 ms
64 bytes from 10.1.0.102: icmp_seq=8 ttl=64 time=0.047 ms
^C
--- 10.1.0.102 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 7024ms
rtt min/avg/max/mdev = 0.018/0.028/0.047/0.010 ms
[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ping 10.1.0.102　　　　　　　　　　　　#我们发现宿主机可以和r2名称空间的eth0虚拟网卡通信啦。

**5>.实现两个虚拟网络名称空间的网络互通** 

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
5: veth1.1@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 52:00:e0:83:48:cd brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip link set dev veth1.1 netns r1　　　　　　#将veth1.1虚拟网卡移动到r1网络名称空间
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:ef:75:60 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:3a:da:a7 brd ff:ff:ff:ff:ff:ff
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig -a
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth1.1: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 52:00:e0:83:48:cd  txqueuelen 1000  (Ethernet)
        RX packets 18  bytes 1524 (1.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 868 (868.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ip link set dev veth1.1 netns r1　　　　　　#将veth1.1虚拟网卡移动到r1网络名称空间

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig -a
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth1.1: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 52:00:e0:83:48:cd  txqueuelen 1000  (Ethernet)
        RX packets 18  bytes 1524 (1.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 868 (868.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig veth1.1 10.1.0.101/24 up　　　　#为r1网络名称空间的veth1.1虚拟网卡分配IP地址并激活
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig 
veth1.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.101  netmask 255.255.255.0  broadcast 10.1.0.255
        inet6 fe80::5000:e0ff:fe83:48cd  prefixlen 64  scopeid 0x20<link>
        ether 52:00:e0:83:48:cd  txqueuelen 1000  (Ethernet)
        RX packets 18  bytes 1524 (1.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16  bytes 1384 (1.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# ip netns exec r1 ifconfig veth1.1 10.1.0.101/24 up　　　　#为r1网络名称空间的veth1.1虚拟网卡分配IP地址并激活

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ifconfig 
veth1.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.101  netmask 255.255.255.0  broadcast 10.1.0.255
        inet6 fe80::5000:e0ff:fe83:48cd  prefixlen 64  scopeid 0x20<link>
        ether 52:00:e0:83:48:cd  txqueuelen 1000  (Ethernet)
        RX packets 26  bytes 2196 (2.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 26  bytes 2196 (2.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r2 ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.102  netmask 255.255.255.0  broadcast 10.1.0.255
        inet6 fe80::341e:54ff:fe37:6978  prefixlen 64  scopeid 0x20<link>
        ether 36:1e:54:37:69:78  txqueuelen 1000  (Ethernet)
        RX packets 26  bytes 2196 (2.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 26  bytes 2196 (2.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# ip netns exec r1 ping 10.1.0.102　　　　#我们使用r1名称空间去ping 一下r2名称空间的虚拟网卡地址，发现是可以互通的！
PING 10.1.0.102 (10.1.0.102) 56(84) bytes of data.
64 bytes from 10.1.0.102: icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from 10.1.0.102: icmp_seq=2 ttl=64 time=0.019 ms
64 bytes from 10.1.0.102: icmp_seq=3 ttl=64 time=0.041 ms
64 bytes from 10.1.0.102: icmp_seq=4 ttl=64 time=0.019 ms
64 bytes from 10.1.0.102: icmp_seq=5 ttl=64 time=0.048 ms
64 bytes from 10.1.0.102: icmp_seq=6 ttl=64 time=0.020 ms
^C
--- 10.1.0.102 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 4999ms
rtt min/avg/max/mdev = 0.017/0.027/0.048/0.013 ms
[root@node103.yinzhengjie.org.cn ~]#
```

\[root@node103.yinzhengjie.org.cn ~\]# ip netns exec r1 ping 10.1.0.102　　　　#我们使用r1名称空间去ping 一下r2名称空间的虚拟网卡地址，发现是可以互通的！

**四.主机名及主机列表解析相关配置案例**

**1>.启动一个容器时指定网络模型**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge --rm busybox:latest　　　　#启动一个名称为test1的容器，网络模型为bridge,其中"-it"表示启动一个交互式终端,"-rm"表示程序允许结束后自动删除容器。
/ # 
/ # hostname 　　　　　　#大家初次看到这个主机名是否有疑虑为什么主机名会是一个随机字符串，其实不然，它默认是容器ID(CONTAINER ID)的名称，可以在开一个终端使用"docker ps"命令进行验证。
9441fef2264c
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --rm busybox:latest　　　　　　#我们在启动容器时使用"-h"参数来指定虚拟机的主机名，但该主机名并不会修改"docker ps"中的"CONTAINER ID"的名称哟~
/ # 
/ # hostname 
node102.yinzhengjie.org.cn
/ #
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.启动容器时指定dns地址**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --rm busybox:latest
/ # 
/ # hostname 
node102.yinzhengjie.org.cn
/ # 
/ # cat /etc/resolv.conf 
# Generated by NetworkManager
search yinzhengjie.org.cn
nameserver 172.30.1.254
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --rm busybox:latest
/ # 
/ # cat /etc/resolv.conf 
search yinzhengjie.org.cn
nameserver 114.114.114.114
/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --rm busybox:latest

**3>.启动容器时自定义dns search**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --rm busybox:latest
/ # 
/ # cat /etc/resolv.conf 
search yinzhengjie.org.cn
nameserver 114.114.114.114
/ # 
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --dns-search ilinux.io --rm busybox:latest
/ # 
/ # cat /etc/resolv.conf 
search ilinux.io
nameserver 114.114.114.114
/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --dns-search ilinux.io --rm busybox:latest

**4>.启动容器时自定义主机解析列表**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --dns-search ilinux.io --rm busybox:latest
/ # 
/ # cat /etc/hosts 
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    node102.yinzhengjie.org.cn node102
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --dns-search ilinux.io --add-host node101.yinzhengjie.org.cn:172.30.1.101 --rm busybox:latest
/ # 
/ # cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.30.1.101    node101.yinzhengjie.org.cn
172.17.0.2    node102.yinzhengjie.org.cn node102
/ # 
/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name test1 -it --network bridge -h node102.yinzhengjie.org.cn --dns 114.114.114.114 --dns-search ilinux.io --add-host node101.yinzhengjie.org.cn:172.30.1.101 --rm busybox:latest

**五.打开入站通信(opening inbound communication)**

**1>.“-p”选项的使用格式**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
-P<containerPort>　　将指定的容器端口映射至主机所有地址的一个动态端口。

-p<hostPort>:<containerPort>　　将容器端口<containerPort>映射至指定的主机端口<hostPort>。

-p<ip>::<containerPort>　　将指定的容器端口<containerPort>映射至主机指定<ip>的动态端口。

-p<ip>:<hostPort>:<containerPort>　　将指定的容器端口<containerPort>映射至主机指定<ip>的端口<hostPort>注意："动态端口"指随机端口，具体的映射结果可使用"docker port"命令查看。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.“-P <containerPort>”案例展示**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        26 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        42 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name myweb --rm -p 80 jason/httpd:v0.2　　　　#我们使用咱们之前自定义的镜像来做实验，启动容器后会自动启动httpd服务
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect myweb | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 172.17.0.2　　　　　　　　　　　　　　　　#另启动一个终端，可以正常访问咱们的容器服务。
<h1>Busybox httpd server.[Jason Yin dao ci yi you !!!]</h1>
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# curl 172.17.0.2　　　　　　　　　　　　　　　　#另启动一个终端，可以正常访问咱们的容器服务。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# iptables -t nat -vnL　　　　　　#注意观察"Chain Docker"那一列，其实我们发现所谓的端口暴露无非是Docker底层调用了DNAT实现的。
Chain PREROUTING (policy ACCEPT 2 packets, 473 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   25  2999 PREROUTING_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
   25  2999 PREROUTING_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
   25  2999 PREROUTING_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    3   156 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1 packets, 60 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  115  8297 OUTPUT_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 3 packets, 164 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    59 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    2   271 RETURN     all  --  *      *       192.168.122.0/24     224.0.0.0/24        
    0     0 RETURN     all  --  *      *       192.168.122.0/24     255.255.255.255     
    0     0 MASQUERADE  tcp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
    0     0 MASQUERADE  udp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
    0     0 MASQUERADE  all  --  *      *       192.168.122.0/24    !192.168.122.0/24    
  115  8130 POSTROUTING_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
  115  8130 POSTROUTING_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
  115  8130 POSTROUTING_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    2   104 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:32768 to:172.17.0.2:80　　　　　　#看到这一行了没有？我们发现docker其实底层调用了iptable来帮它实现端口暴露。

Chain OUTPUT_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING_ZONES (1 references)
 pkts bytes target     prot opt in     out     source               destination         
  103  6991 POST_public  all  --  *      ens33   0.0.0.0/0            0.0.0.0/0           [goto] 
   12  1139 POST_public  all  --  *      +       0.0.0.0/0            0.0.0.0/0           [goto] 

Chain POSTROUTING_ZONES_SOURCE (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain POST_public (2 references)
 pkts bytes target     prot opt in     out     source               destination         
  115  8130 POST_public_log  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
  115  8130 POST_public_deny  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
  115  8130 POST_public_allow  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain POST_public_allow (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain POST_public_deny (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain POST_public_log (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain PREROUTING_ZONES (1 references)
 pkts bytes target     prot opt in     out     source               destination         
   24  2940 PRE_public  all  --  ens33  *       0.0.0.0/0            0.0.0.0/0           [goto] 
    1    59 PRE_public  all  --  +      *       0.0.0.0/0            0.0.0.0/0           [goto] 

Chain PREROUTING_ZONES_SOURCE (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain PREROUTING_direct (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain PRE_public (2 references)
 pkts bytes target     prot opt in     out     source               destination         
   25  2999 PRE_public_log  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
   25  2999 PRE_public_deny  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
   25  2999 PRE_public_allow  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain PRE_public_allow (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain PRE_public_deny (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain PRE_public_log (1 references)
 pkts bytes target     prot opt in     out     source               destination         
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]#iptables -t nat  　　　　　　　　　　　#注意观察"Chain Docker"那一列，其实我们发现所谓的端口暴露无非是Docker底层调用了DNAT实现的。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```

[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
9c020aca98dc        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   2 minutes ago       Up 2 minutes        0.0.0.0:32768->80/tcp   myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker port myweb　　　　　　　　　　#我们发现docker帮我们做了一次映射，myweb容器将httpd服务绑定到了物理机所有网卡可用地址的32768端口进行服务暴露，我们可以直接使用物理机访问它，如下图所示。
80/tcp -> 0.0.0.0:32768
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# hostname -i
172.30.1.101
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191018075434431-1090017195.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
9c020aca98dc        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   11 hours ago        Up 11 hours         0.0.0.0:32768->80/tcp   myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker kill myweb　　　　　　　　#停止容器运行后iptables中的nat表记录也会跟着清除
myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# iptables -t nat -vnL
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination 
359 44547 PREROUTING_direct all -- * * 0.0.0.0/0 0.0.0.0/0 
359 44547 PREROUTING_ZONES_SOURCE all -- * * 0.0.0.0/0 0.0.0.0/0 
359 44547 PREROUTING_ZONES all -- * * 0.0.0.0/0 0.0.0.0/0 
3 156 DOCKER all -- * * 0.0.0.0/0 0.0.0.0/0 ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination 
118 8513 OUTPUT_direct all -- * * 0.0.0.0/0 0.0.0.0/0 
0 0 DOCKER all -- * * 0.0.0.0/0 !127.0.0.0/8 ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination 
1 59 MASQUERADE all -- * !docker0 172.17.0.0/16 0.0.0.0/0 
2 271 RETURN all -- * * 192.168.122.0/24 224.0.0.0/24 
0 0 RETURN all -- * * 192.168.122.0/24 255.255.255.255 
0 0 MASQUERADE tcp -- * * 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
0 0 MASQUERADE udp -- * * 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
0 0 MASQUERADE all -- * * 192.168.122.0/24 !192.168.122.0/24 
118 8346 POSTROUTING_direct all -- * * 0.0.0.0/0 0.0.0.0/0 
118 8346 POSTROUTING_ZONES_SOURCE all -- * * 0.0.0.0/0 0.0.0.0/0 
118 8346 POSTROUTING_ZONES all -- * * 0.0.0.0/0 0.0.0.0/0

Chain DOCKER (2 references)
pkts bytes target prot opt in out source destination 
0 0 RETURN all -- docker0 * 0.0.0.0/0 0.0.0.0/0

Chain OUTPUT_direct (1 references)
pkts bytes target prot opt in out source destination

Chain POSTROUTING_ZONES (1 references)
pkts bytes target prot opt in out source destination 
106 7207 POST_public all -- * ens33 0.0.0.0/0 0.0.0.0/0 [goto] 
12 1139 POST_public all -- * + 0.0.0.0/0 0.0.0.0/0 [goto]

Chain POSTROUTING_ZONES_SOURCE (1 references)
pkts bytes target prot opt in out source destination

Chain POSTROUTING_direct (1 references)
pkts bytes target prot opt in out source destination

Chain POST_public (2 references)
pkts bytes target prot opt in out source destination 
118 8346 POST_public_log all -- * * 0.0.0.0/0 0.0.0.0/0 
118 8346 POST_public_deny all -- * * 0.0.0.0/0 0.0.0.0/0 
118 8346 POST_public_allow all -- * * 0.0.0.0/0 0.0.0.0/0

Chain POST_public_allow (1 references)
pkts bytes target prot opt in out source destination

Chain POST_public_deny (1 references)
pkts bytes target prot opt in out source destination

Chain POST_public_log (1 references)
pkts bytes target prot opt in out source destination

Chain PREROUTING_ZONES (1 references)
pkts bytes target prot opt in out source destination 
358 44488 PRE_public all -- ens33 * 0.0.0.0/0 0.0.0.0/0 [goto] 
1 59 PRE_public all -- + * 0.0.0.0/0 0.0.0.0/0 [goto]

Chain PREROUTING_ZONES_SOURCE (1 references)
pkts bytes target prot opt in out source destination

Chain PREROUTING_direct (1 references)
pkts bytes target prot opt in out source destination

Chain PRE_public (2 references)
pkts bytes target prot opt in out source destination 
359 44547 PRE_public_log all -- * * 0.0.0.0/0 0.0.0.0/0 
359 44547 PRE_public_deny all -- * * 0.0.0.0/0 0.0.0.0/0 
359 44547 PRE_public_allow all -- * * 0.0.0.0/0 0.0.0.0/0

Chain PRE_public_allow (1 references)
pkts bytes target prot opt in out source destination

Chain PRE_public_deny (1 references)
pkts bytes target prot opt in out source destination

Chain PRE_public_log (1 references)
pkts bytes target prot opt in out source destination 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]#
```

\[root@node101.yinzhengjie.org.cn ~\]# docker kill myweb　　　　　　　　#停止容器运行后iptables中的nat表记录也会跟着清除

**3>.“\-p<hostPort>:<containerPort>”案例展示**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        26 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        42 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name myweb --rm -p 80:80 jason/httpd:v0.2　　　　　　#运行时阻塞在当前终端，需要重新开启一个终端查看容器运行情况
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
4e508f5351e9        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker port myweb　　　　　　　　#我们发现容器的80端口被绑定到物理机所有可用网卡的80端口啦。
80/tcp -> 0.0.0.0:80
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker port myweb　　　　　　　　#我们发现容器的80端口被绑定到物理机所有可用网卡的80端口啦，服务可以正常访问，如下图所示。

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191018184117002-1934934106.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
4e508f5351e9        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   3 minutes ago       Up 3 minutes        0.0.0.0:80->80/tcp   myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker kill myweb　　　　　　　　#终止myweb容器运行
myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker kill myweb　　　　　　　　#终止myweb容器运行

**4>.“\-p <ip>::<containerPort>”案例展示**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        36 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        2 days ago          1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name myweb --rm -p 172.30.1.101::80 jason/httpd:v0.2　　　　　　#启动容器时指定绑定的具体IP，另外启动一个终端查看对应的port信息
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                        N
AMES9dab293691ef        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   25 seconds ago      Up 24 seconds       172.30.1.101:32768->80/tcp   myweb
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker port myweb 　　　　　　　　#很明显，此时myweb容器的80端口被动态绑定到172.30.1.101这块网卡的32768端口啦
80/tcp -> 172.30.1.101:32768
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 172.30.1.101:32768　　　　　　#服务依然是可用正常访问的
<h1>Busybox httpd server.[Jason Yin dao ci yi you !!!]</h1>
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker kill myweb　　　　　　　　　　#停止容器运行
myweb
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker port myweb 　　　　　　　　#很明显，此时myweb容器的80端口被动态绑定到172.30.1.101这块网卡的32768端口啦

**4>.“-p<ip>:<hostPort>:<containerPort>”**

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name myweb --rm -p 172.30.1.101:8080:80 jason/httpd:v0.2　　　　#启动容器时指定具体IP和其对应的端口映射容器主机的80端口，运行时程序会阻塞。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                       N
AMES4bb5e9b96599        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   About a minute ago   Up About a minute   172.30.1.101:8080->80/tcp   
myweb[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker port myweb　　　　　　　　#发现配置生效啦，物理机访问结果如下图所示。
80/tcp -> 172.30.1.101:8080
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191018184618041-469109590.png)

**六.基于host的网络模型案例**

**1>.两个虚拟机公用相同网络名称空间**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it --rm busybox　　　　　　#启动第一个容器，注意观察IP地址
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:656 (656.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name c1 -it --rm busybox　　　　　　#启动第一个容器，注意观察IP地址

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c2 -it --network container:c1 --rm busybox　　　　#启动第二个容器，网络名称空间和c1容器共享，注意观察2个容器的IP地址，发现c2容器的IP地址和c1的竟然是一样的。
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:656 (656.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # echo "hello world" > /tmp/index.html
/ # 
/ # ls /tmp/
index.html
/ # 
/ # httpd -h /tmp/　　　　　　　　　　　　#我们在c2容器中启动一个http服务
/ # 
/ # netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address State 
tcp 0 0 :::80 :::* LISTEN 
/ #
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it --rm busybox
/ # 
.....
/ # wget -O - -q 127.0.0.1　　　　　　　　　　　　　　#我们在c1容器完全是可以访问到c2容器的服务的，因为它们公用的是相同的网络名称空间，但文件系统依旧是独立的，可以查看"/tmp"目录下是否有"index.html"文件
hello world
/ # 
/ # ls /tmp/　　　　　　　　　　　　　　　　　　　　　　　　#需要注意的是，它们的文件系统还是独立的，我们在c1的容器无法访问到c2容器的文件系统。
/ #
```

/ # wget -O - -q 127.0.0.1　　　　　　　　　　　　　　#我们在c1容器完全是可以访问到c2容器的服务的，因为它们公用的是相同的网络名称空间，但文件系统依旧是独立的，可以查看"/tmp"目录下是否有"index.html"文件 hello world

**2>.容器和宿主机公用相同的名称空间案例**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:5aff:fe50:fc34  prefixlen 64  scopeid 0x20<link>
        ether 02:42:5a:50:fc:34  txqueuelen 0  (Ethernet)
        RX packets 61  bytes 4728 (4.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 75  bytes 8346 (8.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.101  netmask 255.255.255.0  broadcast 172.30.1.255
        inet6 fe80::20c:29ff:febe:114d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:be:11:4d  txqueuelen 1000  (Ethernet)
        RX packets 9344  bytes 2086245 (1.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6433  bytes 634801 (619.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 100  bytes 10960 (10.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 100  bytes 10960 (10.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:a9:de:9b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 --network host -it --rm busybox　　　　　　　　#我们发现容器和宿主机使用的是相同网络名称空间，容器的所有网卡均和宿主机一样。
/ # 
/ # ifconfig 
docker0   Link encap:Ethernet  HWaddr 02:42:5A:50:FC:34  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:5aff:fe50:fc34/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:61 errors:0 dropped:0 overruns:0 frame:0
          TX packets:75 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:4728 (4.6 KiB)  TX bytes:8346 (8.1 KiB)

ens33     Link encap:Ethernet  HWaddr 00:0C:29:BE:11:4D  
          inet addr:172.30.1.101  Bcast:172.30.1.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:febe:114d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:9552 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6566 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:2104397 (2.0 MiB)  TX bytes:652471 (637.1 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:100 errors:0 dropped:0 overruns:0 frame:0
          TX packets:100 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:10960 (10.7 KiB)  TX bytes:10960 (10.7 KiB)

virbr0    Link encap:Ethernet  HWaddr 52:54:00:A9:DE:9B  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # 
/ # echo "hello container" > /tmp/index.html
/ # 
/ # ls /tmp/
index.html
/ # 
/ # httpd -h /tmp/　　　　　　　　#启动HTTPD服务
/ # 
/ # netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address State 
tcp 0 0 0.0.0.0:111 0.0.0.0:* LISTEN 
tcp 0 0 0.0.0.0:6000 0.0.0.0:* LISTEN 
tcp 0 0 192.168.122.1:53 0.0.0.0:* LISTEN 
tcp 0 0 0.0.0.0:22 0.0.0.0:* LISTEN 
tcp 0 0 127.0.0.1:631 0.0.0.0:* LISTEN 
tcp 0 0 127.0.0.1:25 0.0.0.0:* LISTEN 
tcp 0 0 :::111 :::* LISTEN 
tcp 0 0 :::80 :::* LISTEN 
tcp 0 0 :::6000 :::* LISTEN 
tcp 0 0 :::22 :::* LISTEN 
tcp 0 0 ::1:631 :::* LISTEN 
tcp 0 0 ::1:25 :::* LISTEN 
/ # 
/ #
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name c1 --network host -it --rm busybox　　　　　　　　#我们发现容器和宿主机使用的是相同网络名称空间，容器的所有网卡均和宿主机一样。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::6000                 :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 127.0.0.1:80　　　　　　　　#在容器内部启动httpd服务后，在宿主机上也是可以正常访问到的，因为它们使用的是相同的网络名称空间。
hello container
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# curl 127.0.0.1:80　　　　　　　　#在容器内部启动httpd服务后，在宿主机上也是可以正常访问到的，因为它们使用的是相同的网络名称空间。

**七.自定义docker0桥的网络属性信息**

**1>.修改配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl stop docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# vi /etc/docker/daemon.json 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 　　　　　　#注意，下面的核心选项为"bip",即"bridge ip"之意，用于指定docker0桥自身的IP地址；其它选项可通过此计算出来，当然DNS咱们得单独指定哈~
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "bip":"192.168.100.254/24",
  "dns":["219.141.139.10","219.141.140.10"]
}
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl start docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ifconfig docker0　　　　　　　　　　　　#重启docker服务后，发现配置生效啦！
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.100.254  netmask 255.255.255.0  broadcast 192.168.100.255
        inet6 fe80::42:5aff:fe50:fc34  prefixlen 64  scopeid 0x20<link>
        ether 02:42:5a:50:fc:34  txqueuelen 0  (Ethernet)
        RX packets 61  bytes 4728 (4.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 75  bytes 8346 (8.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.创建一个docker容器观察上一步的配置是否生效**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1  -it --rm busybox　　　　　　　　#和咱们想想的一样，配置已经生效啦！
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:64:01  
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:516 (516.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # cat /etc/resolv.conf 
search yinzhengjie.org.cn
nameserver 219.141.139.10
nameserver 219.141.140.10
/ # 
/ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.100.254 0.0.0.0         UG    0      0        0 eth0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0
/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name c1 -it --rm busybox　　　　　　　　#和咱们想想的一样，配置已经生效啦！

**八.修改docker默认的监听方式**

**1>.查看docker默认监听Unix Socket格式的地址**

```
[root@node101.yinzhengjie.org.cn ~]# ll /var/run/docker.sock 　　　　　　#我们知道Unix socket文件只支持本地通信，想要跨主机支持就得用别的方式实现啦！而默认就是基于Unix socket套接字实现。
srw-rw----. 1 root docker 0 Oct 19 19:13 /var/run/docker.sock
[root@node101.yinzhengjie.org.cn ~]# 
```

**2>.修改配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e249bc41f2fd        busybox             "sh"                10 minutes ago      Up 10 minutes                           c1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker kill c1
c1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl stop docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# vi /etc/docker/daemon.json 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "bip":"192.168.100.254/24",
  "dns":["219.141.139.10","219.141.140.10"],
  "hosts":["tcp://0.0.0.0:8888","unix:///var/run/docker.sock"]　　　　　　　　#此处我们绑定了docker启动基于tcp启动便于其它主机访问，基于unix套接字启动便于本地访问
}
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl start docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# netstat -ntl | grep 8888
tcp6       0      0 :::8888                 :::*                    LISTEN     
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# systemctl restart docker　　　　　　　　　　#记一次启动服务报错，附有详细解决流程。
Warning: docker.service changed on disk. Run 'systemctl daemon-reload' to reload units.
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl
-xe" for details.

[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# tail -100f /var/log/messages　　　　　　　　　　#启动docker服务时最好是一边启动服务一边查看日志信息。启动时发现有以下报错信息
......
Oct 19 19:52:03 node101 dockerd: unable to configure the Docker daemon with file /etc/docker/daemon.json: the following directives are 
specified both as a flag and in the configuration file: hosts: (from flag: [fd://], from file: [tcp://0.0.0.0:2375 unix:///var/run/docker.sock])

....
初次看到这个信息给我的感觉是配置文件出错了，但是始终找不到配置文件哪里有错，最后查看官方文档有关报错信息提到了类似的说明，链接地址为：https://docs.docker.com/config/daemon/

　　报错分析说咱们在配置文件指定了"hosts"这个key，“启动时始终使用主机标志dockerd。如果在中指定 hosts条目，则将daemon.json导致配置冲突并且Docker无法启动。”

具体解决步骤如下所示：
[root@node101.yinzhengjie.org.cn ~]# mkdir -pv /etc/systemd/system/docker.service.d/
mkdir: created directory ‘/etc/systemd/system/docker.service.d/’
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# vi /etc/systemd/system/docker.service.d/docker.conf
[root@node101.yinzhengjie.org.cn ~]#

[root@node101.yinzhengjie.org.cn ~]# cat /etc/systemd/system/docker.service.d/docker.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl daemon-reload　　　　　　　　#这个操作必须做
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# systemctl restart docker　　　　　　　　#经过上述操作，发现问题得到解决
[root@node101.yinzhengjie.org.cn ~]#

[root@node101.yinzhengjie.org.cn ~]# netstat -ntl | grep 8888
tcp6 0 0 :::8888 :::* LISTEN 
[root@node101.yinzhengjie.org.cn ~]#
```

\[root@node101.yinzhengjie.org.cn ~\]# systemctl restart docker　　　　　　　　　　#记一次启动服务报错，附有详细解决流程。

**3>.此时配置是否成功(docker客户端连接其它docker daemon进程)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              de25a81a5a0b        33 hours ago        98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1  -it --rm busybox　　　　　　　　#我们在node101.yinzhengjie.org.cn节点上运行一个容器，于此同时，我们可以在另一个节点来访问当前节点的docker服务
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:64:01  
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # cat /etc/resolv.conf 
search www.tendawifi.com yinzhengjie.org.cn
nameserver 219.141.139.10
nameserver 219.141.140.10
/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name c1 -it --rm busybox　　　　　　　　#我们在node101.yinzhengjie.org.cn节点上运行一个容器，于此同时，我们可以在另一个节点来访问当前节点的docker服务

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# systemctl start docker
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# systemctl stop docker　　　　　　　　　　#我们停掉node102.yinzhengjie.org.cn节点的docker服务
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker -H node101.yinzhengjie.org.cn:8888 image ls　　　　　　#我们访问node101.yinzhengjie.org.cn节点的docker服务，查看该节点的镜像发现是可以查看到数据的。
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              de25a81a5a0b        33 hours ago        98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker -H node101.yinzhengjie.org.cn:8888 container ls　　　　#同理访问容器信息也可以查看到！注意：端口号必须得指定，如果你设置的是2375端口则可以不指定端口哟~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
68db1d8192ad        busybox             "sh"                3 minutes ago       Up 3 minutes                            c1
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**九.创建自定义的网络模型**

**1>.查看已有的网络模型**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker network ls　　　　 #默认的网络模型，创建容器时若不指定网络模型，默认使用"bridge"
NETWORK ID          NAME                DRIVER              SCOPE
7ad23e4ff4b3        bridge              bridge              local
8aeb2bc6b3fe        host                host                local
7e83f7595aac        none                null                local
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker info | grep Network
Network: bridge host ipvlan macvlan null overlay　　　　　　　　　　#我们发现其实网络模型不仅仅只有上面默认的bridge,host和null，docker还支持macvlan以及overlay技术。
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.通过创建自定义的网络模型启动一个容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker network create --help　　　　#查看创建网络驱动的帮助信息

Usage:    docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which copying the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker network create --help　　　　#查看创建网络驱动的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
7ad23e4ff4b3        bridge              bridge              local
8aeb2bc6b3fe        host                host                local
7e83f7595aac        none                null                local
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker network create -d bridge --subnet "192.168.200.0/24" --gateway "192.168.200.254" mybr0　　　　　　#基于bridge创建一块mybr0的网络模型
3d42817e3691bc9f4275b6a222ef6d792b1e0817817e97af77d35dfdfbfe7e24
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker network ls　　　　　　　　#注意观察多出来了一行"mybr0"的信息
NETWORK ID          NAME                DRIVER              SCOPE
7ad23e4ff4b3        bridge              bridge              local
8aeb2bc6b3fe        host                host                local
3d42817e3691        mybr0               bridge              local
7e83f7595aac        none                null                local
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ifconfig 　　　　　　　　　　　　#注意观察多出来了一块网卡，而且网卡的地址就是咱们上面配置的"192.168.200.254"
br-3d42817e3691: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.200.254  netmask 255.255.255.0  broadcast 192.168.200.255
        ether 02:42:db:6f:71:5d  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.100.254  netmask 255.255.255.0  broadcast 192.168.100.255
        ether 02:42:16:4f:26:da  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 08:00:27:e0:bb:66  txqueuelen 1000  (Ethernet)
        RX packets 104555  bytes 139228111 (132.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13542  bytes 860664 (840.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.101  netmask 255.255.255.0  broadcast 172.30.1.255
        ether 08:00:27:c1:c7:46  txqueuelen 1000  (Ethernet)
        RX packets 2478  bytes 200047 (195.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1999  bytes 391091 (381.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker network create -d bridge --subnet "192.168.200.0/24" --gateway "192.168.200.254" mybr0　　　　　　#基于bridge创建一块mybr0的网络模型

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# ifconfig 
br-3d42817e3691: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.200.254  netmask 255.255.255.0  broadcast 192.168.200.255
        ether 02:42:db:6f:71:5d  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.100.254  netmask 255.255.255.0  broadcast 192.168.100.255
        ether 02:42:16:4f:26:da  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 08:00:27:e0:bb:66  txqueuelen 1000  (Ethernet)
        RX packets 104555  bytes 139228111 (132.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13542  bytes 860664 (840.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.101  netmask 255.255.255.0  broadcast 172.30.1.255
        ether 08:00:27:c1:c7:46  txqueuelen 1000  (Ethernet)
        RX packets 2623  bytes 211871 (206.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2102  bytes 408675 (399.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ifconfig br-3d42817e3691 down　　　　　　　　　　　　　　#关掉网卡后改名
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ip link set dev br-3d42817e3691 name docker1　　　　　　#将咱们自定义的网卡名称改为docker1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ifconfig -a
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.100.254  netmask 255.255.255.0  broadcast 192.168.100.255
        ether 02:42:16:4f:26:da  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker1: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.200.254  netmask 255.255.255.0  broadcast 192.168.200.255
        ether 02:42:db:6f:71:5d  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 08:00:27:e0:bb:66  txqueuelen 1000  (Ethernet)
        RX packets 104555  bytes 139228111 (132.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13542  bytes 860664 (840.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.101  netmask 255.255.255.0  broadcast 172.30.1.255
        ether 08:00:27:c1:c7:46  txqueuelen 1000  (Ethernet)
        RX packets 2716  bytes 219575 (214.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2165  bytes 419525 (409.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# ip link set dev br-3d42817e3691 name docker1　　　　　　#将咱们自定义的网卡名称改为docker1

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it --net mybr0 --rm busybox:latest　　　　#基于咱们自定义的网卡mybr0创建一个容器名称为c1
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:C8:01  
          inet addr:192.168.200.1  Bcast:192.168.200.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name c1 -it --net mybr0 --rm busybox:latest　　　　#基于咱们自定义的网卡mybr0创建一个容器名称为c1

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c2 -it --net bridge --rm busybox　　　　　　　　#使用默认的bridge网络创建一个c2的容器
/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:64:01  
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # ping 192.168.200.1　　　　　　　　　　#此时我们发现无法和咱们自定义的"mybr0"网络模型的容器c1进行通信，如果路由转发("/proc/sys/net/ipv4/ip_forward")参数是开启的话，估计问题就出现在iptables上了，需要自行添加放行语句。
PING 192.168.200.1 (192.168.200.1): 56 data bytes
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186