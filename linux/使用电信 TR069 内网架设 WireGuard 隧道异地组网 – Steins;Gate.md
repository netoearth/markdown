TR069 内网是运营商用于下发光猫管理网络的内网，同一运营商同一省份的内网中所有光猫互通，并且可以跑到链路速度。

TR069 内网用于架设异地组网隧道不占用宽带带宽，是同一运营商同一省份异地组网的比较好的选择。虽然光猫只有 1 个千兆网口，但是即便使用光猫的百兆口进行架设，也能跑到 100 Mbps 的速度，享受专线一般的待遇。

本文选用 WireGuard 来作为异地组网方案，是因为 WireGuard 支持在内核角度配置 FwMark 参数直接为 WireGuard 的数据流量打上连接标记，以简化相关路由表的配置。

每个节点需要：

-   电信千兆光猫（百兆猫也可能可以实现，但是没有测试）
-   内网1台双网卡的Linux系统的服务器（经过测试的系统有CentOS 7 & 8，Debian 10，Ubuntu 18.04）
-   相同运营商相同省份的宽带（经过测试的有电信和联通宽带）
-   用于配置 DDNS 的域名，使用 Cloudflare 或阿里云解析
-   宽带拨号地址采用公网或 CGN 内网 IP 均可

本方案的大致配置思路如下。接下来为具体操作步骤。

1.  在光猫配置好 TR069 内网与网口的桥接。
2.  使用网线连上桥接后的光猫网口与内网服务器，使用 DHCP 获取 IP 地址，并屏蔽 DHCP 默认网关与 DNS 服务器的获取。
3.  配置 DDNS ，读取服务器使用的连接 TR069 的网卡 IP 并更新 Cloudflare 或阿里云的 DNS 解析配置。
4.  使用守护进程动态更新路由表的默认网关。
5.  配置 WireGuard ，使用 Fwmark 选项为流量打上连接标记，以此做策略路由，让 WireGuard 发往其他节点的流量从 TR069 内网的网卡出。

首先，获取电信光猫的超级管理员密码。大部分地区的电信光猫的超级管理员用户名密码为：用户名 `telecomadmin` 密码 `telecomadmin` 。若这个密码不可用或是其他运营商的宽带，请自行在网上搜索获取超级管理员密码的方式，这里本文不再叙述。  
若之前光猫是路由模式，直接在浏览器输入 `http://192.168.1.1` 即可访问光猫后台。若光猫是桥接模式（即使用路由器进行 PPPoE 拨号），则需要使用一台电脑，用网线插在光猫的任意网口上面，并设置 IP 地址为 `192.168.1.0/24` 网段内的地址进行操作。  
[](https://www.kryii.com/wp-content/uploads/2021/10/tr069-1.jpeg)

[![](https://www.kryii.com/wp-content/uploads/2021/10/tr069-1.jpeg)](https://www.kryii.com/wp-content/uploads/2021/10/tr069-1.jpeg)

进入光猫后台界面之后，进入网络—连接界面，找到连接名称含有 `TR069` 字样的连接。记录下来这里的 VLAN ID 与 802.1p 数值，并新建一个连接。设置如下：

[

![](https://www.kryii.com/wp-content/uploads/2021/10/tr069-2.jpg)

](https://www.kryii.com/wp-content/uploads/2021/10/tr069-2.jpg)

-   封装类型：IPoE
-   业务类型：其他
-   连接模式：桥接
-   启用VLAN：是
-   VLAN ID：上面记录的VLAN ID
-   802.1p优先级策略：使用指定值
-   802.1p：上面记录的数值
-   LAN端口绑定：选1个没用的口，不要选IPTV口。 如果没有可选项的话去检查一下拨号的连接是否占满全部端口，是的话取消1个

配置完成之后，保存即可。之后使用1台设备用网线连接选择的网口进行测试，使用 DHCP 自动获取IP。如果能获取到 `10.0.0.0/7` 范围的地址（开头是10或11的地址均可），即算配置成功。

光猫配置完成之后，用网线连接光猫新的连接绑定的网口和服务器的1个网口。然后进行网络配置。这里的要点是使用 DHCP 获取 IP 地址，但是不使用 DHCP 的默认网关和 DNS 服务器。  
以下将根据不同的操作系统分别讨论。本文假定连接光猫 TR069 内网的网卡为  
`eth1` ，请根据实际情况进行更改。

这个操作系统最简单，编辑 `/etc/netplan/50-cloud-init.yaml`加入以下配置即可。

```
        eth1:            dhcp4: true            dhcp4-overrides:                use-routes: false                use-dns: false                use-hostname: false
```

编辑完成后，`sudo netplan try` 敲2下回车确认即可。

编辑 `/etc/sysconfig/network-scripts/ifcfg-eth1` ，并修改以下条目即可。若该文件不存在请自行创建。

```
BOOTPROTO="dhcp"ONBOOT="yes"DEFROUTE="no"PEERDNS="no"
```

之后，使用 `ifup eth1` 启动网卡即可。

首先编辑 `/etc/network/interfaces` 并增加以下条目。

```
auto eth1iface eth1 inet dhcp
```

之后编辑 `/etc/dhcp/dhclient.conf` 并增加以下条目，使 DHCP 客户端只获取地址和网段，不获取其他信息。

```
interface "eth1" {    request subnet-mask, broadcast-address;}
```

编辑完成后，使用命令 `ifup eth1` 启动网卡即可。

由于 TR069 内网获得的 IP 地址不固定，我们需要 DDNS 来持续更新域名解析来获取对端地址来连接。和传统调用 API 获取 IP 地址的 DDNS 不同，我们这里需要获取网卡的 IP 地址进行上报。  
使用以下命令安装 Python 环境以及本方案需要的所有包，并下载 DDNS 主程序代码。

```
yum -y install epel-releaseyum copr enable jdoss/wireguardyum -y install python3 git wireguard-dkms wireguard-tools subnetcalcgit clone https://github.com/NewFuture/DDNS /usr/src/DDNS
```

```
add-apt-repository ppa:wireguard/wireguardapt updateapt -y install python3 git wireguard resolvconf subnetcalcgit clone https://github.com/NewFuture/DDNS /usr/src/DDNS
```

```
echo "deb http://mirrors.163.com/debian/ unstable main" > /etc/apt/sources.list.d/unstable.listprintf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstableapt updateapt -y install python3 git wireguard resolvconf subnetcalcgit clone https://github.com/NewFuture/DDNS /usr/src/DDNS
```

我们需要使用一个脚本来获取某个网卡的 IP 地址。创建 `/usr/bin/get-interface-ip` 文件如下。

```
#!/bin/bashifconfig $1 | grep -oP 'inet ((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}' | sed 's/inet //g'
```

之后运行命令 `chmod +x /usr/bin/get-interface-ip` 赋予该脚本可执行权限。  
接下来，创建 DDNS 的配置文件。本文使用 Cloudflare 进行示范。若使用AliDDNS或其他支持的DDNS，参考项目文档进行配置即可。创建文件 `/usr/src/DDNS/config.json` 如下。

```
{  "$schema": "https://ddns.newfuture.cc/schema.json",  "debug": false,  "dns": "cloudflare",  "id": "Cloudflare 邮箱地址",  "index4": "shell:/usr/bin/get-interface-ip eth1",  "index6": "default",  "ipv4": [    "example1.example.com"  ],  "ipv6": [],  "proxy": null,  "token": "Cloudflare 的 API KEY"}
```

创建完成之后，使用命令 `/usr/bin/python3 /usr/src/DDNS/run.py -c /usr/src/DDNS/config.json` 测试是否可以正常上报 DDNS 。若成功，则在 `/etc/crontab` 中添加以下条目来添加 DDNS 的计划任务。

```
* * * * * /usr/bin/python3 /usr/src/DDNS/run.py -c /usr/src/DDNS/config.json
```

TR069 内网 DHCP 到的网段也并不固定，默认网关也与公网并不相通。因此在上面的步骤中我们没有让 DHCP 获取默认网关，避免损坏系统的路由表导致服务器与公网连接中断。而 TR069 内网的网关为 DHCP 到的网段的第1个可用地址。  
我们利用这一点，使用 subnetcalc 工具计算网段第一个可用地址作为默认网关，并将其配置为 table 305 路由表的默认网关，在配置 WireGuard 的时候会用到。  
创建脚本 `/usr/bin/set-tr069-gateway` ，内容如下。并使用命令 `chmod +x /usr/bin/set-tr069-gateway` 赋予可执行权限。

```
#!/bin/bashIP_REGEXP='((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}'LOCAL_IP=$(ifconfig $1 | grep -oP "inet $IP_REGEXP" | sed 's/inet //g')LOCAL_MASK=$(ifconfig $1 | grep -oP "mask $IP_REGEXP" | sed 's/mask //g')GATEWAY=$(subnetcalc $LOCAL_IP/$LOCAL_MASK -n | grep 'Host Range' | grep -oP $IP_REGEXP | head -n 1)ROUTE_TXT="default via $GATEWAY dev $1 table 305" echo $ROUTE_TXT bash -c "ip route replace $ROUTE_TXT || ip route add $ROUTE_TXT"
```

添加之后，输入 `/usr/bin/set-tr069-gateway eth1` 将该脚本运行一次。确认无误之后，在 `/etc/crontab` 中添加以下条目来添加该脚本的计划任务。

```
* * * * * root /usr/bin/set-tr069-gateway eth1
```

完成上面的基础工作之后，我们就可以着手配置 WireGuard 了。首先，使用以下命令生成 WireGuard 的一对密钥。

```
wg genkey | tee /root/privatekey | wg pubkey > /root/publickey
```

之后，创建 WireGuard 配置文件 `/etc/wireguard/wg0.conf` 如下。这里我们使用了 FwMark 参数，使内核自动对 WireGuard 流量打上连接标记，而利用策略路由，将符合 TR069 网段并且打上连接标记的流量从 TR069 内网的网卡流出。

```
[Interface]Address = 192.168.11.1/24PrivateKey = <服务器私钥，/root/privatekey 的内容>ListenPort = 22000FwMark = 0x131PostUp = ip rule add pref 305 to 10.0.0.0/7 fwmark 0x131 lookup 305PostDown = ip rule del pref 305 [Peer]PublicKey = <对端公钥，对方服务器的 /root/publickey 的内容>AllowedIPs = 192.168.11.2/32Endpoint = example2.example.com:22000PersistentKeepalive = 25
```

这里 Address 字段为服务器 WireGuard 内网地址。每个 Peer 的内网地址写在对应的 AllowedIPs 字段下。Endpoint 为对端服务器的配置的 DDNS 域名。若对端服务器同时承载网络的网关，在网段不冲突的情况下，可以在 AllowedIPs 写上对方所在的网段，而让网段互通。  
本文只演示两个节点的情况。若有多个节点，按照格式增加 Peer 即可。  
配置完成后，使用以下命令启动 WireGuard，即可通过 TR069 内网连通对方服务器，享受 100Mbps 对等的专线连接。

```
wg-quick up wg0systemctl enable wg-quick@wg0
```

至此，我们已经实现了利用电信光猫 TR069 内网来搭建 WireGuard 隧道，实现服务器之间快速稳定的通信。  
通信速度实际上受制于光猫的百兆网口的链路速度。事实上，如果不在意下行速度的话，完全可以把宽带移动到百兆口上，而用千兆口进行 TR069 内网通信，得到一条千兆对等隧道。一般而言，对于服务器的场景下，使用下行带宽不如上行带宽多，大多数情况下下行不会超过上行。深圳电信 500M 宽带就有 50M 的上行。  
此外，这个方案并不需要宽带是开通的。只要宽带曾经开通过，即便被停机，仍然可以用这个方案架设隧道到其他能上网的地方来蹭网用。不过需要解决 DNS 的问题。这里在没有宽带的情况下，没有办法正常解析域名。可能需要一些其他的配置来解决这个问题。