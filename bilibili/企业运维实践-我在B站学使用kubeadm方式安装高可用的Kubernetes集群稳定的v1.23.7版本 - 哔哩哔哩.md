

**本章目录：**

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

0x00 前言简述

0x01 环境准备

-   主机规划
    
-   软件版本
    
-   网络规划
    

0x02 安装部署

-   1.准备基础主机环境配置
    
-   2.负载均衡管理ipvsadm工具安装与内核加载
    
-   3.高可用HAProxy与Keepalived软件安装配置
    
-   4.容器运行时containerd.io安装配置
    
-   5.安装源配置与初始化集群配置准备
    
-   6.使用kubeadm安装部署K8S集群
    
-   7.部署配置 Calico 网络插件
    

0x03 集群辅助插件部署

-   1.集群中基于nfs的provisioner的动态持卷环境部署
    
-   2.集群中安装metrics-server获取客户端资源监控指标
    
-   3.集群管理原生UI工具kubernetes-dashboard安装部署
    
-   4.集群管理K9S客户端工具安装使用
    
-   5.集群服务Service七层负载均衡ingress环境搭建部署
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

作者: WeiyiGeek

首发地址：https://blog.weiyigeek.top/2022/6-7-664.html


![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

## 0x00 前言简述

描述: 在我博客以及前面的文章之中讲解Kubernetes相关集群环境的搭建说明, 随着K8S及其相关组件的迭代, 与读者当前接触的版本有所不同，在上一章中我们一起实践了【使用二进制方式进行安装部署高可用的K8S集群V1.23.6】(https://blog.weiyigeek.top/2022/5-7-654.html) , 所以本章将实践使用kubeadm方式部署搭建高可用的kubernetes集群V1.23.7，此处仍然按照ubuntu 20.04系统以及haproxy、keepalive、containerd、etcd、kubeadm、kubectl 等相关工具插件【最新或者稳定的版本】进行实践，这里不再对k8s等相关基础知识做介绍，如有新入门的童鞋，请访问如下【博客文章】(https://blog.weiyigeek.top/tags/k8s/) 或者【B站专栏】([https://www.bilibili.com/read/readlist/rl520875?spm\_id\_from=333.999.0.0](https://www.bilibili.com/read/readlist/rl520875?spm_id_from=333.999.0.0)) 按照顺序学习。

**Kubernetes 简述**  
Kubernetes (后续简称k8s)是 Google(2014年6月) 开源的一个容器编排引擎，使用Go语言开发，它支持自动化部署、大规模可伸缩、以及云平台中多个主机上的容器化应用进行管理。其目标是让部署容器化的应用更加简单并且高效，提供了资源调度、部署管理、服务发现、扩容缩容、状态 监控、维护等一整套功能, 努力成为跨主机集群的自动化部署、自动化扩展以及运行应用程序容器的平台，它支持一些列CNCF毕业项目，包括 Containerd、calico 等 。

**其它方案安装K8S集群，知识**扩展**文章:**

1.使用二进制方式部署v1.23.6的K8S集群实践(上)【 https://mp.weixin.qq.com/s/sYpHENyehAnGOQQakYzhUA 】

2.使用二进制方式部署v1.23.6的K8S集群实践(下)

【 https://mp.weixin.qq.com/s/-ksiNJG6v4q47ez7\_H3uYQ 】

## 0x01 环境准备

## 主机规划

温馨提示: 同样此处使用的是 Ubuntu 20.04 操作系统, 该系统已做安全加固和内核优化符合等保2.0要求【SecOpsDev/Ubuntu-InitializeSecurity.sh at master · WeiyiGeek/SecOpsDev (github.com)】, 如你的Linux未进行相应配置环境可能与读者有些许差异, 如需要进行(windows server、Ubuntu、CentOS)安全加固请参照如下加固脚本进行加固, 请大家疯狂的 star 。

加固脚本地址:【 https://github.com/WeiyiGeek/SecOpsDev/blob/master/OS-操作系统/Linux/Ubuntu/Ubuntu-InitializeSecurity.sh 】

![](https://i0.hdslb.com/bfs/article/23e44ffa97c736ad04f9559e412d85f4d045cf9e.png@938w_338h_progressive.webp)

## 软件版本

**操作系统**

-   Ubuntu 20.04 LTS - 5.4.0-92-generic
    

**高可用软件**

-   ipvsadm - 1:1.31-1
    
-   haproxy - 2.0.13-2
    
-   keepalived - 1:2.0.19-2
    

**ETCD数据库**

-   etcd - v3.5.4
    

**容器运行时**

-   containerd.io - 1.6.6
    

**Kubernetes**

-   kubeadm - v1.23.7
    
-   kube-apiserver - v1.23.7
    
-   kube-controller-manager - v1.23.7
    
-   kubectl - v1.23.7
    
-   kubelet - v1.23.7
    
-   kube-proxy - v1.23.7
    
-   kube-scheduler - v1.23.7
    

**网络插件&辅助软件**

-   calico - v3.22
    
-   coredns - v1.9.1
    
-   kubernetes-dashboard - v2.5.1
    
-   k9s - v0.25.18
    

## 网络规划

![](https://i0.hdslb.com/bfs/article/5114c016ad8d969838ca9aab96bb26306660ba15.png@639w_231h_progressive.webp)

## 0x02 安装部署

## 1.准备基础主机环境配置

步骤 01.【所有主机】主机名设置按照上述主机规划进行设置。

```
# 例如, 在10.20.176.212主机中运行。
hostnamectl set-hostname devtest-master-212
# 例如, 在10.20.176.213主机中运行。
hostnamectl set-hostname devtest-master-213
# 例如, 在10.20.176.214主机中运行。
hostnamectl set-hostname devtest-master-214

# 例如, 在10.10.107.215主机中运行。
hostnamectl set-hostname devtest-work-215
```

步骤 02.【所有主机】将规划中的主机名称与IP地址进行硬解析。  

```
sudo tee -a /etc/hosts <<'EOF'
10.20.176.211 slbvip.k8s.devtest
10.20.176.212 devtest-master-212
10.20.176.213 devtest-master-213
10.20.176.214 devtest-master-214
10.20.176.215 devtest-work-215
EOF
```

步骤 03.验证每个节点上IP、MAC 地址和 product\_uuid 的唯一性,保证其能相互正常通信

```
# 使用命令 ip link 或 ifconfig -a 来获取网络接口的 MAC 地址
ifconfig -a
# 使用命令 查看 product_uuid 校验
sudo cat /sys/class/dmi/id/product_uuid
```

步骤 04.【所有主机】系统时间同步与时区设置

```
date -R
sudo ntpdate ntp.aliyun.com
sudo timedatectl set-timezone Asia/Shanghai
# 或者
# sudo dpkg-reconfigure tzdata
sudo timedatectl set-local-rtc 0
timedatectl
```

步骤 05.【所有主机】禁用系统交换分区

```
swapoff -a && sed -i 's|^/swap.img|#/swap.ing|g' /etc/fstab
# 验证交换分区是否被禁用
free | grep "Swap:"
```

步骤 06.【所有主机】系统内核参数调整

```
# 禁用 swap 分区
egrep -q "^(#)?vm.swappiness.*" /etc/sysctl.conf && sed -ri "s|^(#)?vm.swappiness.*|vm.swappiness = 0|g"  /etc/sysctl.conf || echo "vm.swappiness = 0" >> /etc/sysctl.conf
# 允许转发
egrep -q "^(#)?net.ipv4.ip_forward.*" /etc/sysctl.conf && sed -ri "s|^(#)?net.ipv4.ip_forward.*|net.ipv4.ip_forward = 1|g"  /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

# - 允许 iptables 检查桥接流量
egrep -q "^(#)?net.bridge.bridge-nf-call-iptables.*" /etc/sysctl.conf && sed -ri "s|^(#)?net.bridge.bridge-nf-call-iptables.*|net.bridge.bridge-nf-call-iptables = 1|g" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
egrep -q "^(#)?net.bridge.bridge-nf-call-ip6tables.*" /etc/sysctl.conf && sed -ri "s|^(#)?net.bridge.bridge-nf-call-ip6tables.*|net.bridge.bridge-nf-call-ip6tables = 1|g" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
```

步骤 07.【所有主机】禁用系统防火墙

`ufw disable && systemctl disable ufw && systemctl stop ufw`

## 2.负载均衡管理ipvsadm工具安装与内核加载

步骤 01.安装ipvs模块以及负载均衡相关依赖。

```
# 查看可用版本
sudo apt-cache madison ipvsadm
  # ipvsadm |   1:1.31-1 | http://mirrors.aliyun.com/ubuntu focal/main amd64 Packages

# 安装
sudo apt -y install ipvsadm ipset sysstat conntrack

# 锁定版本
apt-mark hold ipvsadm
  # ipvsadm set on hold.
```

步骤 02.将模块加载到内核中(开机自动设置-需要重启机器生效)

```
tee /etc/modules-load.d/k8s-.conf <<'EOF'
# netfilter
br_netfilter

# containerd
overlay

# nf_conntrack
nf_conntrack

# ipvs
ip_vs
ip_vs_lc
ip_vs_lblc
ip_vs_lblcr
ip_vs_rr
ip_vs_wrr
ip_vs_sh
ip_vs_dh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_tables
ip_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
xt_set
EOF
```

步骤 03.手动加载模块到Linux内核中。

```
mkdir -vp /etc/modules.d/
tee /etc/modules.d/k8s-ipvs.modules <<'EOF'
#!/bin/bash
# netfilter 模块 允许 iptables 检查桥接流量
modprobe -- br_netfilter
# containerd
modprobe -- overlay
# nf_conntrack
modprobe -- nf_conntrack
# ipvs
modprobe -- ip_vs
modprobe -- ip_vs_lc
modprobe -- ip_vs_lblc
modprobe -- ip_vs_lblcr
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- ip_vs_dh
modprobe -- ip_vs_fo
modprobe -- ip_vs_nq
modprobe -- ip_vs_sed
modprobe -- ip_vs_ftp
modprobe -- ip_tables
modprobe -- ip_set
modprobe -- ipt_set
modprobe -- ipt_rpfilter
modprobe -- ipt_REJECT
modprobe -- ipip
modprobe -- xt_set
EOF

# 权限设置、并加载到系统之中
chmod 755 /etc/modules.d/k8s-ipvs.modules && bash /etc/modules.d/k8s-ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack
  # ip_vs_sh               16384  0
  # ip_vs_wrr              16384  0
  # ip_vs_rr               16384  0
  # ip_vs                 155648  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
  # nf_conntrack          139264  1 ip_vs
  # nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
  # nf_defrag_ipv4         16384  1 nf_conntrack
  # libcrc32c              16384  5 nf_conntrack,btrfs,xfs,raid456,ip_vs

sysctl --system
```

温馨提示: 在 kernel 4.19 版本及以上将使用 nf\_conntrack 模块, 则在 4.18 版本以下则需使用nf\_conntrack\_ipv4 模块。

## 3.高可用HAProxy与Keepalived软件安装配置

描述: 由于是测试学习环境, 此处我未专门准备两台HA服务器, 而是直接采用master节点机器，如果是正式环境建议独立出来。

步骤 01.【Master节点机器】安装下载 haproxy (HA代理健康检测) 与 keepalived (虚拟路由协议-主从)。

```
# 查看可用版本
sudo apt-cache madison haproxy keepalived
  #  haproxy | 2.0.13-2ubuntu0.5 | http://mirrors.aliyun.com/ubuntu focal-security/main amd64 Packages
  # keepalived | 1:2.0.19-2ubuntu0.2 | http://mirrors.aliyun.com/ubuntu focal-updates/main amd64 Packages

# 安装
sudo apt -y install haproxy keepalived

# 锁定版本
apt-mark hold haproxy keepalived
```

步骤 02.【Master节点机器】进行 HAProxy 配置，其配置目录为 `/etc/haproxy/`，所有节点配置是一致的。

```
sudo cp /etc/haproxy/haproxy.cfg{,.bak}
tee /etc/haproxy/haproxy.cfg<<'EOF'
global
  user haproxy
  group haproxy
  maxconn 2000
  daemon
  log /dev/log local0
  log /dev/log local1 err
  chroot /var/lib/haproxy
  stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
  stats timeout 30s
  # Default SSL material locations
  ca-base /etc/ssl/certs
  crt-base /etc/ssl/private
  # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
  ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
  ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
  ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
  log     global
  mode    http
  option  httplog
  option  dontlognull
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s
  # errorfile 400 /etc/haproxy/errors/400.http
  # errorfile 403 /etc/haproxy/errors/403.http
  # errorfile 408 /etc/haproxy/errors/408.http
  # errorfile 500 /etc/haproxy/errors/500.http
  # errorfile 502 /etc/haproxy/errors/502.http
  # errorfile 503 /etc/haproxy/errors/503.http
  # errorfile 504 /etc/haproxy/errors/504.http

# 注意: 管理HAproxy (可选)
# frontend monitor-in
#   bind *:33305
#   mode http
#   option httplog
#   monitor-uri /monitor

# 注意: 基于四层代理, 1644 3为VIP的 ApiServer 控制平面端口, 由于是与master节点在一起所以不能使用6443端口.
frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

# 注意: Master 节点的默认 Apiserver 是6443端口
backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server devtest-master-212 10.20.176.212:6443 check
  server devtest-master-213 10.20.176.213:6443 check
  server devtest-master-214 10.20.176.214:6443 check
EOF
```

步骤 03.【Master节点机器】进行 KeepAlived 相关配置 ，其配置目录为 `/etc/haproxy/`

```
# 创建配置目录，分别在各个master节点执行。
mkdir -vp /etc/keepalived
# __ROLE__ 角色: MASTER 或者 BACKUP
# __NETINTERFACE__ 宿主机物理网卡名称 例如我的ens32
# __IP__ 宿主机物理IP地址
# __VIP__ 虚拟VIP地址
sudo tee /etc/keepalived/keepalived.conf <<'EOF'
! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL
script_user root
  enable_script_security
}
vrrp_script chk_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 5
  weight -5
  fall 2
  rise 1
}
vrrp_instance VI_1 {
  state __ROLE__
  interface __NETINTERFACE__
  mcast_src_ip __IP__
  virtual_router_id 51
  priority 101
  advert_int 2
  authentication {
    auth_type PASS
    auth_pass K8SHA_KA_AUTH
  }
  virtual_ipaddress {
    __VIP__
  }
  # HA 健康检查
  # track_script {
  #   chk_apiserver
  # }
}
EOF

# 此处将 devtest-master-212 配置为 Master (devtest-master-212 主机上执行)
# devtest-master-212 10.20.176.212 => MASTER
sed -i -e 's#__ROLE__#MASTER#g' \
  -e 's#__NETINTERFACE__#ens32#g' \
  -e 's#__IP__#10.20.176.212#g' \
  -e 's#__VIP__#10.20.176.211#g' /etc/keepalived/keepalived.conf

# devtest-master-213 10.20.176.213 => BACKUP  (devtest-master-213 主机上执行)
sed -i -e 's#__ROLE__#BACKUP#g' \
  -e 's#__NETINTERFACE__#ens32#g' \
  -e 's#__IP__#10.20.176.213#g' \
  -e 's#__VIP__#10.20.176.211#g' /etc/keepalived/keepalived.conf

# devtest-master-214 10.20.176.214 => BACKUP  (devtest-master-214 主机上执行)
sed -i -e 's#__ROLE__#BACKUP#g' \
  -e 's#__NETINTERFACE__#ens32#g' \
  -e 's#__IP__#10.20.176.214#g' \
  -e 's#__VIP__#10.20.176.211#g' /etc/keepalived/keepalived.conf
```

温馨提示: 注意上述的健康检查是关闭注释了的，你需要将K8S集群建立完成后再开启。

`track_script { chk_apiserver }`  

步骤 04.【Master节点机器】进行配置 KeepAlived 健康检查文件。

```
sudo tee /etc/keepalived/check_apiserver.sh <<'EOF'
#!/bin/bash
err=0
for k in $(seq 1 3)
do
  check_code=$(pgrep haproxy)
  if [[ $check_code == "" ]]; then
    err=$(expr $err + 1)
    sleep 1
    continue
  else
    err=0
    break
  fi
done

if [[ $err != "0" ]]; then
  echo "systemctl stop keepalived"
  /usr/bin/systemctl stop keepalived
  exit 1
else
  exit 0
fi
EOF
sudo chmod +x /etc/keepalived/check_apiserver.sh
```

步骤 05.【Master节点机器】启动 haproxy 、keepalived 相关服务及测试VIP漂移。

```
# 重载 Systemd 设置 haproxy 、keepalived 开机自启以及立即启动
sudo systemctl daemon-reload
sudo systemctl enable --now haproxy && sudo systemctl enable --now keepalived
  # Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
  # Executing: /lib/systemd/systemd-sysv-install enable haproxy
  # Synchronizing state of keepalived.service with SysV service script with /lib/systemd/systemd-sysv-install.
  # Executing: /lib/systemd/systemd-sysv-install enable keepalived

# 在 devtest-master-212 主机中发现vip地址在其主机上。
root@devtest-master-212:~$ ip addr
  # 2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
  #   link/ether 00:50:56:8a:57:69 brd ff:ff:ff:ff:ff:ff
  #   inet 10.20.176.212/24 brd 10.20.176.255 scope global ens32
  #       valid_lft forever preferred_lft forever
  #   inet 10.20.176.211/24 scope global ens32
  #       valid_lft forever preferred_lft forever
  #   inet6 fe80::250:56ff:fe8a:5769/64 scope link
  #       valid_lft forever preferred_lft forever

# 其它两台Master主机上通信验证。
root@devtest-master-213:~$ ping 10.20.176.211
  # PING 10.20.176.211 (10.20.176.211) 56(84) bytes of data.
  # 64 bytes from 10.20.176.211: icmp_seq=1 ttl=64 time=0.161 ms
root@devtest-master-214:~$ ping 10.20.176.211
```

![WeiyiGeek. KeepAlived-地址存活验证](https://i0.hdslb.com/bfs/article/0f27cd0b2443175ee65330e658cbd6f5d0536f50.png@942w_284h_progressive.webp)

然后我们可以手动验证VIP漂移,我们将该服务器上keepalived停止掉。

```
root@devtest-master-212:~$ pgrep haproxy
  # 6120
  # 6121
root@devtest-master-212:~$ /usr/bin/systemctl stop keepalived

# 此时,发现VIP已经飘到devtest-master-212主机中
root@devtest-master-215:~$ ip addr show ens32
  # 2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
  #     link/ether 00:0c:29:93:28:61 brd ff:ff:ff:ff:ff:ff
  #     inet10.20.176.215/24 brd 10.10.107.255 scope global ens32
  #       valid_lft forever preferred_lft forever
  #     inet 10.20.176.211/32 scope global ens32
  #       valid_lft forever preferred_lft forever
```

至此，HAProxy 与 Keepalived 配置就告一段落了。

## 4.容器运行时containerd.io安装配置

步骤 01.分别在master与work节点上安装containerd

```
# 1.卸载旧版本
sudo apt-get remove docker docker-engine docker.io containerd runc

# 2.更新apt包索引并安装包以允许apt在HTTPS上使用存储库
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

# 3.添加Docker官方GPG密钥 # -fsSL
curl https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 4.通过搜索指纹的最后8个字符进行密钥验证
sudo apt-key fingerprint 0EBFCD88

# 5.设置稳定存储库
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# 6.安装特定版本在repo中列出可用的版本
sudo apt-cache madison containerd.io
  # containerd.io |    1.6.6-1 | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
  # containerd.io |    1.6.4-1 | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
  # containerd.io |   1.5.11-1 | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages

# 7.使用第二列中的版本字符串安装特定的版本，例如: 1.6.6-1
sudo apt-get install containerd.io=1.6.6-1
# 离线安装: apt install -y ./containerd.io_1.6.6-1_amd64.deb
```

步骤 02.下载安装后在【所有节点】进行containerd 配置，此处将创建并修改 config.toml 文件.

```
# 为 containerd 生成默认配置
mkdir -vp /etc/containerd
containerd config default >/etc/containerd/config.toml
ls /etc/containerd/config.toml
  # /etc/containerd/config.toml

# pause 镜像源
sed -i "s#k8s.gcr.io/pause#registry.cn-hangzhou.aliyuncs.com/google_containers/pause#g"  /etc/containerd/config.toml

# 使用 SystemdCgroup
sed -i 's#SystemdCgroup = false#SystemdCgroup = true#g' /etc/containerd/config.toml

# docker.io mirror
sed -i '/registry.mirrors]/a\ \ \ \ \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]' /etc/containerd/config.toml
sed -i '/registry.mirrors."docker.io"]/a\ \ \ \ \ \ \ \ \ \ endpoint = ["https://05f073ad3c0010ea0f4bc00b7105ec20.mirror.swr.myhuaweicloud.com","https://xlx9erfu.mirror.aliyuncs.com","https://docker.mirrors.ustc.edu.cn"]' /etc/containerd/config.toml

# gcr.io mirror
sed -i '/registry.mirrors]/a\ \ \ \ \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]' /etc/containerd/config.toml
sed -i '/registry.mirrors."gcr.io"]/a\ \ \ \ \ \ \ \ \ \ endpoint = ["https://gcr.mirrors.ustc.edu.cn"]' /etc/containerd/config.toml

# k8s.gcr.io mirror
sed -i '/registry.mirrors]/a\ \ \ \ \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]' /etc/containerd/config.toml
sed -i '/registry.mirrors."k8s.gcr.io"]/a\ \ \ \ \ \ \ \ \ \ endpoint = ["https://gcr.mirrors.ustc.edu.cn/google-containers/","https://registry.cn-hangzhou.aliyuncs.com/google_containers/"]' /etc/containerd/config.toml

# quay.io mirror
sed -i '/registry.mirrors]/a\ \ \ \ \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]' /etc/containerd/config.toml
sed -i '/registry.mirrors."quay.io"]/a\ \ \ \ \ \ \ \ \ \ endpoint = ["https://quay.mirrors.ustc.edu.cn"]' /etc/containerd/config.toml
```

步骤 03.修改配置后【所有节点】重载containerd服务并查看相关服务及其版本。

```
# 配置重载与服务重启
systemctl daemon-reload && systemctl restart containerd.service
systemctl status -l containerd.service
  # ● containerd.service - containerd container runtime
  #    Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
  #    Active: active (running) since Thu 2022-06-16 05:22:50 UTC; 5 days ago
  #      Docs: https://containerd.io
  #  Main PID: 790 (containerd)
  #     Tasks: 51
  #    Memory: 76.3M
  #    CGroup: /system.slice/containerd.service
  #            ├─ 790 /usr/bin/containerd

# 客户端版本查看
root@devtest-master-212:/var/cache/apt/archives# ctr version
# Client:
#   Version:  1.6.6
#   Revision: 10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
#   Go version: go1.17.11
# Server:
#   Version:  1.6.6
#   Revision: 10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
#   UUID: 71a28bbb-6ed6-408d-a873-e394d48b35d8

root@devtest-master-212:/var/cache/apt/archives# runc -v
  # runc version 1.1.2
  # commit: v1.1.2-0-ga916309
  # spec: 1.0.2-dev
  # go: go1.17.11
  # libseccomp: 2.5.1
```

## 5.安装源配置与初始化集群配置准备

步骤 01.【所有节点】Kubernetes 安装源配置及其kubelet、kubeadm、kubectl工具下载安装

```
# (1) gpg 签名下载导入
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

# (2) Kubernetes 安装源
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt update
# 其它方式:
# (2) 设置稳定存储库
# sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"

# (3) K8S可用版本,此处可以看见最新的是 1.24.1 , 由于博主实践的K8S是为开发测试环境所搭建，则此处选择 1.23.7 版本。
apt-cache madison kubelet | more
  kubelet |  1.24.1-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
  kubelet |  1.24.0-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
  kubelet |  1.23.7-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages

# (4) 下载安装指定版本的kubelet、kubeadm、kubectl
K8S_VERSION="1.23.7-00"
sudo apt install kubelet=${K8S_VERSION} kubeadm=${K8S_VERSION} kubectl=${K8S_VERSION}
```

步骤 02.【所有节点】重载systemd守护进程并将 kubelet 设置成开机启动

```
systemctl daemon-reload
systemctl restart containerd.service
systemctl enable kubelet && systemctl start kubelet
```

步骤 03.【devtest-master-212】为了节约拉取实践我们可以在某一台master节点上先拉取所K8S集群所需要的镜像。

```
# 列出所需镜像
kubeadm config images list --kubernetes-version=1.23.7
  # k8s.gcr.io/kube-apiserver:v1.23.7
  # k8s.gcr.io/kube-controller-manager:v1.23.7
  # k8s.gcr.io/kube-scheduler:v1.23.7
  # k8s.gcr.io/kube-proxy:v1.23.7
  # k8s.gcr.io/pause:3.6
  # k8s.gcr.io/etcd:3.5.1-0
  # k8s.gcr.io/coredns/coredns:v1.8.6

# 使用阿里提供的镜像源进行拉取v1.23.7版本依赖的相关镜像
for i in $(kubeadm config images list --kubernetes-version=1.23.7 --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers -v 5);do
  ctr -n k8s.io images pull ${i}
done
  # registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.23.7
  # registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.23.7
  # registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.23.7
  # registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.23.7
  # registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6
  # registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.1-0
  # registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.6

# 从 container.io 中的镜像导出本地
for images in $(ctr -n k8s.io i ls name~=registry.cn-hangzhou.aliyuncs.com/google_containers -q);do
  temp_name=${images##*/}
  tar_name=$(echo $temp_name |  tr ':' '_')
  echo "export ${images} >> $tar_name.tar"
  ctr -n k8s.io images export ${tar_name}.tar ${images}
done

# 从本地导入镜像到 container.io 中
for tarfile in $(ls);do
    $ ctr images import --base-name foo/bar foobar.tar
  ctr -n k8s.io images export ${tar_name}.tar ${images}
done
```

步骤 04.导入到各个节点后我们可以通过如下命令查看导入的镜像列表信息。

```
# 方式1
ctr -n k8s.io i ls name~=registry.cn-hangzhou.aliyuncs.com/google_container

# 方式2
crictl images | grep "registry.cn-hangzhou.aliyuncs.com/google"
registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64               3.0                 99e59f495ffaa       314kB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns                   v1.8.6              a4ca41631cc7a       13.6MB
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd                      3.5.1-0             25f8c7f3da61c       98.9MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver            v1.23.7             03c169f383d97       32.6MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager   v1.23.7             e34d4a6252edd       30.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy                v1.23.7             b1aa05aa5100e       39.3MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler            v1.23.7             ed0ccfa052ab4       15.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause                     3.6                 6270bb605e12e       302kB
```

## 6.使用kubeadm安装部署K8S集群

步骤 01.【devtest-master-212 节点】生成K8S初始化yaml配置文件，并根据实际情况进行编辑

```
$ kubeadm config print init-defaults > kubeadm-init-default.yaml
$ vim kubeadm-init-default.yaml

# 此处为v1.23.x 适用的集群初始化配置清单
tee kubeadm-init-cluster.yaml <<'EOF'
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: devtes.httpweiyigeektop
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.20.176.212   # 关键点: 当前节点地址
  bindPort: 6443                    # 关键点: API端口默认即可
nodeRegistration:
  criSocket: /run/containerd/containerd.sock  # 关键点: 指定containerd为运行时
  imagePullPolicy: IfNotPresent
  name: devtest-master-212          # 关键点: 当前节点名称
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:                         # 关键点: 证书中心包含的SANs列表一般包含VIP与各节点的主机名称
  - slbvip.k8s.devtest
  - localhost
  - devtest-master-212
  - devtest-master-213
  - devtest-master-214
  - 10.20.176.211
  - 10.20.176.212
  - 10.20.176.213
  - 10.20.176.214
  - 10.66.66.2
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers  # 关键点: 集群所需镜像仓库源加快拉取
kind: ClusterConfiguration
kubernetesVersion: v1.23.7                      # 关键点: K8S集群版本此处与我们提前下载的镜像版本必须一致否则将会重新拉取指定K8S集群所需镜像。
controlPlaneEndpoint: slbvip.k8s.devtest:16443  # 关键点: 高可用VIP地址的域名与haproxy代理端口。
networking:
  dnsDomain: cluster.test                       # 关键点: 集群dns根域默认cluster.local
  serviceSubnet: 10.96.0.0/12                   # 关键点: services 服务子网段
  podSubnet: 10.66.0.0/16                       # 关键点: pod 服务子网段
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs                                      # 关键点: 启用IPVS支持
ipvs:
  excludeCIDRs:
  - 10.66.66.2/32                               # 关键点: 排出指定IP
EOF
```

步骤 02.【devtest-master-212 节点】准备好初始化yaml清单后, 通过 kubeadm init 命令进行集群的初始化，并将日志输入到kubeadm-init.log。

```
kubeadm init --config=kubeadm-init-cluster.yaml --v=5 | tee kubeadm-init.log
# --config 指定yaml配置文件
# --v 指定日志等级 debug

# 当执行后出现如下提示表明节点初始化安装成功，可以按照指定提示进行执行。
Your Kubernetes control-plane has initialized successfully!
# To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Alternatively, if you are the root user, you can run:
  export KUBECONFIG=/etc/kubernetes/admin.conf

# You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

# 【控制节点加入命令】
# You can now join any number of control-plane nodes by copying certificate authorities and service account keys on each node and then running the following as root:
  kubeadm join slbvip.k8s.devtest:16443 --token devtes.httpweiyigeektop \
        --discovery-token-ca-cert-hash sha256:4e5b3acb1d821bbd3976355f7b12b1daaa44e1fc38cbdfb2f86f4078d9507f22 \
        --control-plane

# 【工作节点加入命令】
# Then you can join any number of worker nodes by running the following on each as root:
kubeadm join slbvip.k8s.devtest:16443 --token devtes.httpweiyigeektop \
        --discovery-token-ca-cert-hash sha256:4e5b3acb1d821bbd3976355f7b12b1daaa44e1fc38cbdfb2f86f4078d9507f22
```

步骤 03.将【devtest-master-212】节点中`/etc/kubernetes/`目录中的如下文件进行复制打包，并通过http协议将其分享给其它机器进行下载

```
# 将ca证书拷贝到指定目录
mkdir -vp /tmp/pki/etcd && cp /etc/kubernetes/pki/ca.* /etc/kubernetes/pki/sa.* /etc/kubernetes/pki/front-proxy-ca.* /tmp/pki
cp /etc/kubernetes/pki/etcd/ca.* /tmp/pki/etcd/

# 压缩依赖依赖的ca证书目录
tar -zcvf pki.tar.gz pki/

# 使用 python 搭建一个临时 http 服务
python3 -m http.server 8080

# 在其它【节点】进入到/tmp/执行如下命令进行下载
cd /tmp/ && wget 10.20.176.212:8080/pki.tar.gz
tar -zxvf /tmp/pki.tar.gz -C /etc/kubernetes/

# 查看解压后的复制到/etc/kubernetes/目录中的文件相关结构。
tree /etc/kubernetes/
/etc/kubernetes/
└── pki
    ├── ca.crt
    ├── ca.key
    ├── etcd
    │   ├── ca.crt
    │   └── ca.key
    ├── front-proxy-ca.crt
    ├── front-proxy-ca.key
    ├── sa.key
    └── sa.pub
2 directories, 8 files
```

步骤 04.在其余【master】节点中执行如下 kubeadm join 命令添加新的 Master 节点到K8S集群控制面板中, 加入成功后如下图所示:

```
kubeadm join slbvip.k8s.devtest:16443 \
--control-plane \
--token devtes.httpweiyigeektop \
--discovery-token-ca-cert-hash sha256:4e5b3acb1d821bbd3976355f7b12b1daaa44e1fc38cbdfb2f86f4078d9507f22 \
--cri-socket /run/containerd/containerd.sock
```

![](https://i0.hdslb.com/bfs/article/dbf1adc640063a7885e365f5215a509fad673d2b.png@942w_623h_progressive.webp)

温馨提示: 在`v1.23.x`如果要使用containerd运行时必须`--cri-socket`指定其运行时socket。

步骤 05.在其余【work】节点中执行如下 kubeadm join 命令添加新的 Work 节点到K8S集群中, 加入成功后如下图所示:

```
kubeadm join slbvip.k8s.devtest:16443 \
  --token devtes.httpweiyigeektop \
  --discovery-token-ca-cert-hash sha256:4e5b3acb1d821bbd3976355f7b12b1daaa44e1fc38cbdfb2f86f4078d9507f22 \
  --cri-socket /run/containerd/containerd.sock
```

![WeiyiGeek.work-join](https://i0.hdslb.com/bfs/article/861270a7a0fa7695686bea26d0a7d57c79072a23.png@942w_254h_progressive.webp)

步骤 06.全部加入到集群后我们可以通过如下命令进行查看添加的节点以及其节点角色设置，最终执行结果如下图所示。

```
# 集群节点查看
root@devtest-master-212:~$ kubectl get node
  # NAME                 STATUS   ROLES                  AGE     VERSION
  # devtest-master-212   Ready    control-plane,master   15m     v1.23.7
  # devtest-master-213   Ready    control-plane,master   5m19s   v1.23.7
  # devtest-master-214   Ready    control-plane,master   2m7s    v1.23.7
  # devtest-work-215     Ready    <none>                 14m     v1.23.7

# 集群节点角色设置，此处将devtest-work-215节点的ROLES设置为 work。
root@devtest-master-212:~$ kubectl label node devtest-work-215 node-role.kubernetes.io/work=
  # node/devtest-work-215 labeled
```

![WeiyiGeek.cluster-roles](https://i0.hdslb.com/bfs/article/fdf75c093bdec7c8876d0e55a55038b589361908.png@942w_84h_progressive.webp)

步骤 07.查看集群信息、集群组件、以及集群中所有Pod进行查看, 执行结果如下

```
root@devtest-master-212:~# kubectl cluster-info
  Kubernetes control plane is running at https://slbvip.k8s.devtest:16443
  CoreDNS is running at https://slbvip.k8s.devtest:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
  To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

root@devtest-master-212:~# kubectl get cs
  Warning: v1 ComponentStatus is deprecated in v1.19+
  NAME                 STATUS    MESSAGE                         ERROR
  scheduler            Healthy   ok
  controller-manager   Healthy   ok
  etcd-0               Healthy   {"health":"true","reason":""}

root@devtest-master-212:~# kubectl get pod -A
```

![WeiyiGeek.cluster-info](https://i0.hdslb.com/bfs/article/274aa7a06241d1806566beb19bac786bcaffcecb.png@942w_504h_progressive.webp)

步骤 08.开启所有【master】中keepalived的健康检查,并重启keepalived.service服务

```
vim /etc/keepalived/keepalived.conf
...
# 开启 HA 健康检查，例如
  track_script {
    chk_apiserver
  }

systemctl restart keepalived.service
```

步骤 09.然后我们为kubectl添加命令自动补齐不全功能，执行如下代码即可。

```
apt install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

步骤 10.最后我们可以部署一个nginx的Pod验证集群环境, 并通过curl进行访问，但是此时你会发现只有在【devtest-work-215】主机上才能访问该nginx，而其它节点访问时会报`Connection timed out`, 其原因可看下面的温馨提示。

```
kubectl run nginx --image nginx:latest --port=80
  pod/nginx created

kubectl get pod -o wide
  NAME    READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
  nginx   1/1     Running   0          68s   10.22.0.5   devtest-work-215   <none>           <none>

curl -I http://10.22.0.5
  curl: (28) Failed to connect to 10.22.0.5 port 80: Connection timed out
```

温馨提示: 这是由于我们没有为集群配置网络插件，而常用的网络插件是flannel、calico（`下面将会以此为例`）以及当下cilium。

步骤 11.特别注意此处为了能在后续演示中将Pod应用调度到Master节点上,此时我们需要分别去除控制节点的污点。

```
kubectl taint node devtest-master-212 node-role.kubernetes.io/master=:NoSchedule-
  # node/devtest-master-212 untainted

kubectl taint node devtest-master-213 node-role.kubernetes.io/master=:NoSchedule-
  # node/devtest-master-213 untainted

kubectl taint node devtest-master-214 node-role.kubernetes.io/master=:NoSchedule-
  # node/devtest-master-214 untainted
```

## 7.部署配置 Calico 网络插件

描述: 在节点加入到集群时有时你会发现其节点状态为 NotReady, 以及前面部署Pod无法被其它节点机器进行代理转发访问，所以部署calico插件可以让Pod与集群正常通信。

步骤 01.在【devtest-master-212】节点上拉取最新版本的 calico 当前最新版本为 v3.22, 官方项目地址 (https://github.com/projectcalico/calico)

```
# 拉取 calico 部署清单
wget https://docs.projectcalico.org/v3.22/manifests/calico.yaml
```

步骤 02.修改 calico.yaml 文件的中如下 K/V, 即 Pod 获取IP地址的地址池, 从网络规划中我们设置为 `10.66.0.0/16`, 注意默认情况下如下字段是注释的且默认地址池为`192.168.0.0/16`。

```
$ vim calico.yaml
# The default IPv4 pool to create on startup if none exists. Pod IPs will be
# chosen from this range. Changing this value after installation will have
# no effect. This should fall within `--cluster-cidr`.
- name: CALICO_IPV4POOL_CIDR
  value: "10.66.0.0/16"
```

![WeiyiGeek.CALICO_IPV4POOL_CIDR](https://i0.hdslb.com/bfs/article/251d87f9b372714cf6bae29659e1a17e5e459844.png@942w_270h_progressive.webp)

步骤 03.执行 kubectl apply 命令部署 calico 到集群之中。

```
kubectl apply -f calico.yaml
  # configmap/calico-config created
  # ....
  # poddisruptionbudget.policy/calico-kube-controllers created
```

步骤 04.查看calico网络插件在各节点上部署结果,状态为Running表示部署成功。

```
kubectl get pod -n kube-system -o wide| head -n 6
  # NAME                                         READY   STATUS    RESTARTS        AGE     IP              NODE                 NOMINATED NODE   READINESS GATES
  # calico-kube-controllers-6b77fff45-w29rq      1/1     Running   0               8m5s    10.66.35.65     devtest-master-214   <none>           <none>
  # calico-node-7gxdc                            1/1     Running   0               8m5s    10.20.176.213   devtest-master-213   <none>           <none>
  # calico-node-tsk6w                            1/1     Running   0               8m5s    10.20.176.212   devtest-master-212   <none>           <none>
  # calico-node-vnkhg                            1/1     Running   0               8m5s    10.20.176.215   devtest-work-215     <none>           <none>
  # calico-node-xh2dw                            1/1     Running   0               8m5s    10.20.176.214   devtest-master-214   <none>           <none>
```

![](https://i0.hdslb.com/bfs/article/dc46c58025fbd14e4bf542342cf24d10bab30b44.png@942w_96h_progressive.webp)

步骤 05.此时删除 nginx 的 Pod 再重新创建一个Pod应用，并在非【devtest-work-215 】节点上进行访问测试

```
root@devtest-master-212:/opt/init/k8s# kubectl run nginx --image=nginx:latest --port=80
  # pod/nginx created

root@devtest-master-212:/opt/init/k8s# kubectl get pod nginx
  # NAME    READY   STATUS    RESTARTS   AGE
  # nginx   1/1     Running   0          10s

root@devtest-master-212:/opt/init/k8s# kubectl get pod nginx -o wide
  # NAME    READY   STATUS    RESTARTS   AGE   IP   NODE   NOMINATED NODE       READINESS GATES
  # nginx   1/1     Running   0          13s   10.66.53.66   devtest-work-215   <none>   <none>

root@devtest-master-212:/opt/init/k8s# curl -I 10.66.53.66
  # HTTP/1.1 200 OK
  # Server: nginx/1.21.5
  # Date: Wed, 15 Jun 2022 11:39:48 GMT
  # Content-Type: text/html
  # Content-Length: 615
  # Last-Modified: Tue, 28 Dec 2021 15:28:38 GMT
  # Connection: keep-alive
  # ETag: "61cb2d26-267"
  # Accept-Ranges: bytes
```

至此，高可用的K8S集群部署完毕，在下节中将实践演示在集群中安装 Metrics Server 、kubernetes-dashboard、nfs-provisioner、ingress 等安装实践。

## 0x03 集群辅助插件部署

## 1.集群中基于nfs的provisioner的动态持卷环境部署

描述: 在K8S集群中我们常常会使用动态持久卷对应用数据进行持久化保存，例如应用配置、依赖插件、应用数据以及应用访问日志等，所以动态持久卷的重要性不言而喻，此小节将讲解如何搭建以及nfs存储服务以及使用`kubernetes-sigs`提供的\[nfs-subdir-external-provisioner\] (https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)项目针对已存在的 nfs 服务器进行 provisioner 部署。

**Q: 什么是nfs-subdir-external-provisioner?**

> NFS subdir external provisioner 是一个自动配置器，它使用您现有的和已配置的 NFS 服务器来支持通过 Persistent Volume Claims 动态配置 Kubernetes Persistent Volumes。  
> 持久卷配置为 namespace−namespace−{pvcName}-${pvName}。

温馨提示：在进行此环境搭建时，请注意您必须已经有配置好的 NFS 服务器，为了保持与早期部署文件的向后兼容性，NFS Client Provisioner 的命名在部署 YAML 中保留为 nfs-storage-provisioner。

项目地址: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

步骤 01.我在一台Ubuntu 20.04主机上配置 NFS 存储服务器(IP地址为10.20.176.102)，如果你已有存储服务器提供的NFS服务器则可以略过此步骤，其它操作系统安装部署请看出我的这篇博客文章\[NFS网络文件系统基础配置与使用\] (https://blog.weiyigeek.top/2019/5-16-104.html)

```
root@devtest-master-212:/opt/init/k8s# kubectl run nginx --image=nginx:latest --port=80
  # pod/nginx created

root@devtest-master-212:/opt/init/k8s# kubectl get pod nginx
  # NAME    READY   STATUS    RESTARTS   AGE
  # nginx   1/1     Running   0          10s

root@devtest-master-212:/opt/init/k8s# kubectl get pod nginx -o wide
  # NAME    READY   STATUS    RESTARTS   AGE   IP   NODE   NOMINATED NODE       READINESS GATES
  # nginx   1/1     Running   0          13s   10.66.53.66   devtest-work-215   <none>   <none>

root@devtest-master-212:/opt/init/k8s# curl -I 10.66.53.66
  # HTTP/1.1 200 OK
  # Server: nginx/1.21.5
  # Date: Wed, 15 Jun 2022 11:39:48 GMT
  # Content-Type: text/html
  # Content-Length: 615
  # Last-Modified: Tue, 28 Dec 2021 15:28:38 GMT
  # Connection: keep-alive
  # ETag: "61cb2d26-267"
  # Accept-Ranges: bytes
```

步骤 02.【所有节点】在客户端尝试挂载搭建的NFS服务器，并分别在节点机器挂载到`/storage/nfs`目录中

```
# 查看 NFS 挂载目录
showmount -e 127.0.0.1
  # Export list for 127.0.0.1:
  # /app/storage/nfs *

# 挂载 NFS 服务器到指定节点目录的/storage/nfs目录
mkdir -vp /storage/nfs
# 临时挂载
mount -t nfs 10.20.176.102:/app/storage/nfs /storage/nfs
# 永久挂载将挂载配置写入到/etc/fstab中
tee -a /etc/fstab <<"EOF"
10.20.176.102:/app/storage/nfs /storage/nfs nfs defaults 0 0
EOF
mount -a

# 查看挂载信息
$ mount -l
$ df -Th | grep "/storage/nfs"
10.20.176.102:/app/storage/nfs nfs4   97G   11G   82G  12% /storage/nfs

# 如果不使用了则可执行如下命令进行卸载挂载点
umount /storage/nfs
```

![WeiyiGeek.挂载NFS服务到各个节点](https://i0.hdslb.com/bfs/article/811936a8e9e01f8cfb00f0aeb4c39662b28690f0.png@942w_207h_progressive.webp)

步骤 03.helm3 部署工具快速安装

```
# helm3 安装
$ wget https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz && tar -zxf helm-v3.9.0-linux-amd64.tar.gz
$ tar -zxf helm-v3.9.0-linux-amd64.tar.gz
$ cp linux-amd64/helm /usr/local/bin/helm3
```

步骤 04.在Github中下载nfs-subdir-external-provisioner的release压缩包或者使用helm3指定仓库进行安装。

-   方式1.release 压缩包
    

```
$ wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/releases/download/nfs-subdir-external-provisioner-4.0.16/nfs-subdir-external-provisioner-4.0.16.tgz
$ tar -zxvf nfs-subdir-external-provisioner-4.0.16.tgz
# 按照指定需求编辑如下values值
$ egrep -v '^$|#' values.yaml
replicaCount: 1
strategyType: Recreate
image:
  repository: registry.cn-hangzhou.aliyuncs.com/weiyigeek/nfs-subdir-external-provisioner
  tag: v4.0.2
  pullPolicy: IfNotPresent
imagePullSecrets: []
nfs:
  server: 10.20.176.102
  path: /app/storage/nfs
  mountOptions:
  volumeName: nfs-subdir-external-provisioner-root
  reclaimPolicy: Retain
storageClass:
  create: true
  provisionerName: cluster.local/nfs-storage-subdir-provisioner
  defaultClass: false
  name: nfs-storage
  allowVolumeExpansion: true
  reclaimPolicy: Delete
  archiveOnDelete: true
  onDelete:
  pathPattern:
  accessModes: ReadWriteOnce
  annotations: {}
leaderElection:
  enabled: true
rbac:
  create: true
podSecurityPolicy:
  enabled: false
podAnnotations: {}
podSecurityContext: {}
securityContext: {}
serviceAccount:
  create: true
  annotations: {}
  name:
resources: {}
nodeSelector: {}
tolerations: []
affinity: {}
labels: {}

# 指定下载解压目录进行chat图表安装
$ helm3 install nfs-storage nfs-subdir-external-provisioner/
  NAME: nfs-storage
  LAST DEPLOYED: Mon Jun 20 15:41:50 2022
  NAMESPACE: default
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
```

-   方式2.helm3指定仓库
    

```
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
$ helm nfs-storage nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=10.20.176.102 \
  --set nfs.path=/app/storage/nfs

# 更新
$ helm3 upgrade nfs-storage nfs-subdir-external-provisioner/
```

温馨提示: 由于nfs-subdir-external-provisioner镜像是k8s.gcr.io域名下国内可能无法拉取，此处我已经拉取到阿里云镜像仓库中`registry.cn-hangzhou.aliyuncs.com/weiyigeek/nfs-subdir-external-provisioner:v4.0.2`，如果你需要自行拉取请参考此篇使用Aliyun容器镜像服务对海外镜像仓库中镜像进行拉取构建 ()

步骤 05.查看部署的nfs-storage动态持久卷以及其pod状态

```
$ helm3 list
  NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                    APP VERSION
  nfs-storage     default         1               2022-06-20 15:41:50.960642233 +0800 CST deployed        nfs-subdir-external-provisioner-4.0.16   4.0.2

$ kubectl get storageclasses.storage.k8s.io
  NAME          PROVISIONER                                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
  nfs-storage   cluster.local/nfs-storage-subdir-provisioner   Delete          Immediate           true                   17s

$ kubectl get pod
  NAME                                                           READY   STATUS    RESTARTS   AGE
  nfs-storage-nfs-subdir-external-provisioner-77f4d85f74-kljck   1/1     Running   0          33s
```

步骤 06.如想卸载helm3安装的nfs-storage动态卷执行如下命令即可。

`$ helm3 uninstall nfs-storage`

`release "nfs-storage" uninstalled`

至此，基于NFS的provisioner 动态持久卷的安装部署到此结束。

温馨提示: 如果需要部署多个nfs持久卷, 则我们需要更改values.yaml中如下字段，并使用不同的名称。

```
$ helm3 list
  NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                    APP VERSION
  nfs-storage     default         1               2022-06-20 15:41:50.960642233 +0800 CST deployed        nfs-subdir-external-provisioner-4.0.16   4.0.2

$ kubectl get storageclasses.storage.k8s.io
  NAME          PROVISIONER                                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
  nfs-storage   cluster.local/nfs-storage-subdir-provisioner   Delete          Immediate           true                   17s

$ kubectl get pod
  NAME                                                           READY   STATUS    RESTARTS   AGE
  nfs-storage-nfs-subdir-external-provisioner-77f4d85f74-kljck   1/1     Running   0          33s
```

正常情况下显示出的日志信息:

```
$ kubectl logs -f nfs-dev-nfs-subdir-external-provisioner-cf7684f8b-fzl9h
  # I0714 03:27:35.390909       1 leaderelection.go:242] attempting to acquire leader lease  default/cluster.local-nfs-dev-subdir-provisioner...
  # I0714 03:27:35.400777       1 leaderelection.go:252] successfully acquired lease default/cluster.local-nfs-dev-subdir-provisioner
  # I0714 03:27:35.400856       1 event.go:278] Event(v1.ObjectReference{Kind:"Endpoints", Namespace:"default", Name:"cluster.local-nfs-dev-subdir-provisioner", UID:"34019115-d09e-433d-85d6-c254c5492ce5", APIVersion:"v1", ResourceVersion:"5510886", FieldPath:""}): type: 'Normal' reason: 'LeaderElection' nfs-dev-nfs-subdir-external-provisioner-cf7684f8b-fzl9h_b0aaa93a-f7fc-4931-b0cf-e04f42cefd9a became leader
  # I0714 03:27:35.400943       1 controller.go:820] Starting provisioner controller cluster.local/nfs-dev-subdir-provisioner_nfs-dev-nfs-subdir-external-provisioner-cf7684f8b-fzl9h_b0aaa93a-f7fc-4931-b0cf-e04f42cefd9a!
  # I0714 03:27:35.501130       1 controller.go:869] Started provisioner controller cluster.local/nfs-dev-subdir-provisioner_nfs-dev-nfs-subdir-external-provisioner-cf7684f8b-fzl9h_b0aaa93a-f7fc-4931-b0cf-e04f42cefd9a!
```

## 2.集群中安装metrics-server获取客户端资源监控指标

描述: 通常在集群安装完成后,我们需要对其设置网络、持久卷存储等插件, 除此之外我们还需安装metrics-server以便于获取Node与Pod相关资源消耗等信息，否则你在执行`kubectl top`命令时会提示`error: Metrics API not available`, 所以本小节将针对Metrics-server的安装进行讲解。

项目地址: https://github.com/kubernetes-sigs/metrics-server

**Q: 什么是metrics-server?**

> Metrics Server 是 Kubernetes 内置自动缩放管道的可扩展、高效的容器资源指标来源。  
> Metrics Server 从 Kubelets 收集资源指标，并通过 Metrics API 在 Kubernetes apiserver 中公开它们，供 Horizontal Pod Autoscaler 和 Vertical Pod Autoscaler 使用。

> 简单的说: Metrics Server 是集群解析监控数据的聚合器,安装后用户可以通过标准的API（/apis/metrics.k8s.io）来访问监控数据，此处值得注意的是Metrics-Server并非kube-apiserver的一部分,而是通过Aggregator这种插件机制,在独立部署的情况下同kube-apiserver一起统一对外服务的，当进行api请求时kube-aggregator统一接口会分析访问api具体的类型，帮我们负载到具体的api上。

温馨提示: 我们可以通过 `kubectl top` 命令来访问 Metrics API 获取资源监控相关数据。

温馨提示: 注意 Metrics API 只可以查询当前度量数据,并不保存历史数据。

**安装使用**

步骤 01.Metrics Server 可以直接从 YAML 清单安装，也可以通过官方 Helm 图表安装。

```
# (1) 下载 YAML 清单
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O metrics-server.yaml
root@devtest-master-212:/opt/init/k8s# ls
  calico.yaml   kubeadm-init-cluster.yaml  kubeadm-init.log  metrics-server.yaml

# (2) 提前下载相应的镜像加快部署
grep "image:" metrics-server.yaml
  # image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1

# (3) 由于其镜像国内无法访问此处我们采用阿里云k8s.gcr.io镜像源
sed -i 's#k8s.gcr.io/metrics-server#registry.cn-hangzhou.aliyuncs.com/google_containers#g' metrics-server.yaml

# (4) 部署资源清单
kubectl apply -f metrics-server.yaml
  # serviceaccount/metrics-server created
  ......
  # apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

步骤 02.不论是在二进制方式或kubeadm方式安装k8s集群, 都在部署 metrics-server 资源清单后 Pod 状态为 0/1 并报出`annot validate certificate for 10.10.107.223 because it doesn't contain any IP SANs`错误问题解决。

-   错误信息:
    

```
$ kubectl describe pod -n kube-system metrics-server-6ffc8966f5-cf2qh
# Warning  Unhealthy  8s (x17 over 2m27s)  kubelet  Readiness probe failed: HTTP probe failed with statuscode: 500

$ kubectl logs -f --tail 50 -n kube-system metrics-server-6ffc8966f5-cf2qh
# E0520 11:13:17.379944       1 scraper.go:140] "Failed to scrape node" err="Get \"https://10.10.107.226:10250/metrics/resource\": x509: cannot validate certificate for 10.10.107.226 because it doesn't contain any IP SANs" node="node-1"
# E0520 11:13:17.382948       1 scraper.go:140] "Failed to scrape node" err="Get \"https://10.10.107.223:10250/metrics/resource\": x509: cannot validate certificate for 10.10.107.223 because it doesn't contain any IP SANs" node="master-223"
```

-   问题原因: 由于 metrics-server 未获得TLS Bootstrap 签发证书的导致访问各节点资源时报错。
    
-   解决办法: 启用 TLS BootStrap 证书签发
    

```
# 1.分别在 Master 与 Node 节点中启用TLS BootStrap 证书签发，在 kubelet 的 yaml 配置中追加入如下K/V.
# 方式1.Kubeadm 搭建的集群
$ vim /var/lib/kubelet/config.yaml
...
serverTLSBootstrap: true

# 方式2.二进制搭建的集群(注意此路径根据你的kubelet.service进行配置), 此处我们定义的路径为 /etc/kubernetes/cfg/kubelet-config.yaml
# /var/lib/kubelet/config.yaml
$ tee -a /etc/kubernetes/cfg/kubelet-config.yaml <<'EOF'
serverTLSBootstrap: true
EOF

# kubeadm 安装方式可以使用如下
tee -a /var/lib/kubelet/config.yaml <<'EOF'
serverTLSBootstrap: true
EOF

# 2.最后分别重启各个节点kubelet服务即可
systemctl daemon-reload && systemctl restart kubelet.service

# 3.查看节点的证书签发请求
kubectl get csr
  # NAME        AGE   SIGNERNAME                      REQUESTOR                        REQUESTEDDURATION   CONDITION
  # csr-6w6xp   7s    kubernetes.io/kubelet-serving   system:node:devtest-master-212   <none>              Pending
  # csr-bpqzl   5s    kubernetes.io/kubelet-serving   system:node:devtest-master-213   <none>              Pending
  # csr-gtksm   3s    kubernetes.io/kubelet-serving   system:node:devtest-work-215     <none>              Pending
  # csr-vzfs7   4s    kubernetes.io/kubelet-serving   system:node:devtest-master-214   <none>              Pending

# 4.同意签发所有节点的CSR请求
kubectl certificate approve $(kubectl get csr | grep -v NAME | grep "Pending"  | cut -d " " -f 1)
  # certificatesigningrequest.certificates.k8s.io/csr-6w6xp approved
  # certificatesigningrequest.certificates.k8s.io/csr-bpqzl approved
  # certificatesigningrequest.certificates.k8s.io/csr-gtksm approved
  # certificatesigningrequest.certificates.k8s.io/csr-vzfs7 approved
```

![WeiyiGeek.证书签发请求](https://i0.hdslb.com/bfs/article/174aeb05899ac691d43850f20435d4cbd2ff1400.png@942w_102h_progressive.webp)

步骤 03.Metrics Server 默认是安装在kube-system名称空间下，我们可以查看其deployment、Pod运行以及SVC情况。

```
# 1.部署清单状态查看
kubectl get deploy,pod,svc -n kube-system -l k8s-app=metrics-server
  # NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
  # deployment.apps/metrics-server   1/1     1            1           5h37m

  # NAME                                  READY   STATUS    RESTARTS   AGE
  # pod/metrics-server-6ffc8966f5-c4jvn   1/1     Running   0          14m

  # NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
  # service/metrics-server   ClusterIP   10.111.197.94   <none>        443/TCP   5h37m

# 2.注册到K8S集群中的 metrics.k8s.io API 查看
kubectl get apiservices.apiregistration.k8s.io  | grep "metrics"
  # v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        14h
```

步骤 04.查看各个节点以及Pod的资源指标（CPU/MEM）

```
$ kubectl top node
  # NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
  # devtest-master-212   219m         2%     2793Mi          17%
  # devtest-master-213   211m         2%     2417Mi          15%
  # devtest-master-214   203m         1%     2467Mi          7%
  # devtest-work-215     90m          1%     1601Mi          10%

# 默认名称空间中Pod的资源信息
$ kubectl top pod
  # NAME    CPU(cores)   MEMORY(bytes)
# nginx   0m           10Mi
```

## 3.集群管理原生UI工具kubernetes-dashboard安装部署

描述:Kubernetes Dashboard是Kubernetes集群的通用、基于web的UI。它允许用户管理集群中运行的应用程序并对其进行故障排除，以及管理集群本身。

项目地址: https://github.com/kubernetes/dashboard/

步骤 01.从Github中拉取dashboard部署资源清单，当前最新版本v2.6.0 【2022年6月22日 16:46:37】

```
# 下载资源清单并部署 dashboard
$ wget -L https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml -O dashboard.yaml
$ kubectl apply -f dashboard.yaml
$ grep "image:" dashboard.yaml
  # image: kubernetesui/dashboard:v2.6.0
  # image: kubernetesui/metrics-scraper:v1.0.8

# 或者一条命令搞定部署
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
  # serviceaccount/kubernetes-dashboard created
  ......
  # deployment.apps/dashboard-metrics-scraper created
```

步骤 02.使用kubectl get命令查看部署的dashboard相关资源是否正常。

```
$ kubectl get pod,svc -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-6f669b9c9b-vfmzp   1/1     Running   0          73m
kubernetes-dashboard-67b9478795-2gwmm        1/1     Running   0          73m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.106.168.38    <none>        8000/TCP   73m
service/kubernetes-dashboard        ClusterIP   10.106.191.197   <none>        443/TCP    73m

# 编辑 service/kubernetes-dashboard 服务将端口通过nodePort方式进行暴露为30443。
$ kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
# service/kubernetes-dashboard edited
apiVersion: v1
kind: Service
.....
spec:
.....
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
    nodePort: 31443  # 新增
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort     # 修改
```

步骤 03.默认仪表板部署包含运行所需的最小RBAC权限集，而要想使用dashboard操作集群中的资源，通常我们还需要自定义创建kubernetes-dashboard管理员角色。  
权限控制参考地址: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md

```
# 创建后最小权限的Token(只能操作kubernetes-dashboard名称空间下的资源)
kubectl get sa -n kubernetes-dashboard kubernetes-dashboard
kubectl describe secrets -n kubernetes-dashboard kubernetes-dashboard-token-jhdpb | grep '^token:'|awk '{print $2}'
```

Kubernetes Dashboard 支持几种不同的用户身份验证方式：

-   Authorization header
    
-   Bearer Token (默认)
    
-   Username/password
    
-   Kubeconfig file (默认)
    

温馨提示: 此处使用Bearer Token方式, 为了方便演示我们向 Dashboard 的服务帐户授予管理员权限 (Admin privileges), 而在生产环境中通常不建议如此操作, 而是指定一个或者多个名称空间下的资源进行操作。

```
tee rbac-dashboard-admin.yaml <<'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kubernetes-dashboard
EOF

kubectl apply -f rbac-dashboard-admin.yaml
  # serviceaccount/dashboard-admin created
  # clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created

# 或者 两条命令搞定
# kubectl create serviceaccount -n devtest devtest-ns-admin
# kubectl create clusterrolebinding devtest-ns-admin --clusterrole=admin --serviceaccount=devtest:devtest-ns-admin
```

步骤 04.获取 sa 创建的 dashboard-admin 用户的 secrets 名称并获取认证 token ，用于上述搭建的dashboard 认证使用。

```
kubectl get sa -n kubernetes-dashboard dashboard-admin -o yaml | grep "\- name" | awk '{print $3}'
  # dashboard-admin-token-crh7v
kubectl describe secrets -n kubernetes-dashboard dashboard-admin-token-crh7v | grep "^token:" | awk '{print $2}'
  #  获取到认证Token
```

步骤 05.利用上述 Token 进行登陆Kubernetes-dashboard的UI,其地址为node节点加上31443,例如此处`https://10.20.176.212:31443/#/workloads?namespace=default`。

![WeiyiGeek.Kubernetes-dashboard](https://i0.hdslb.com/bfs/article/e6c8ee93983c2f06affda6bce0b6881196daa092.png@942w_819h_progressive.webp)

## 4.集群管理K9S客户端工具安装使用

描述: k9s 是用于管理 Kubernetes 集群的 CLI, K9s 提供了一个终端 UI 来与您的 Kubernetes 集群进行交互。通过封装 kubectl 功能 k9s 持续监视 Kubernetes 的变化并提供后续命令来与您观察到的资源进行交互，直白的说就是k9s可以让开发者快速查看并解决运行 Kubernetes 时的日常问题。  
官网地址: https://k9scli.io/  
参考地址: https://github.com/derailed/k9s

此处,以安装二进制包为例进行实践。

```
# 1. 利用 wget 命令 -c 短点续传和 -b 后台下载
wget -b -c https://github.com/derailed/k9s/releases/download/v0.25.18/k9s_Linux_x86_64.tar.gz

# 2.解压并删除多余文件
tar -zxf k9s_linux_x86_64.tar.gz
rm  k9s_linux_x86_64.tar.gz  LICENSE  README.md

# 3.拷贝 kubernetes 控制配置文件到加目录中
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

# 4.直接运行即可，如果你对vim操作比较熟悉，那么恭喜你了你很快能上手k9s.
/stroage/nfs# ./k9s

# 5.退出k9s指令
:quit
```

更多使用技巧请参考: \[K9s之K8s集群管理工具实践尝试\] (https://blog.weiyigeek.top/2022/1-1-582.html)

## 5.集群服务Service七层负载均衡ingress环境搭建部署

**Q: 什么是Ingress?**  
A: Ingress 是管理对集群中服务的提供外部访问的 API 对象,Ingress 控制器负责实现 Ingress，通常使用负载均衡器，但它也可以配置边缘路由器或其他前端来帮助处理流量，它可以将来自集群外部的 HTTP 和 HTTPS 路由转发到集群内部的 Service 中。

Ingress 只是一个统称，其由 Ingress 和 Ingress Controller 两部分组成。

-   Ingress 用作将原来需要手动配置的规则抽象成一个 Ingress 对象，使用 YAML 格式的文件来创建和管理。
    
-   Ingress Controller 用作通过与 Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化。
    

使用 Ingress 控制器可以轻松实现外部URL访问集群内部服务、负载均衡、代理转发、支持配置SSL/TLS并提供基于名称的虚拟主机，值得注意的是 Ingress 不会暴露任意端口或协议，通过使用 `Service.Type=NodePort` 或 `Service.Type=LoadBalancer`类型的服务向向 Internet 公开 HTTP 和 HTTPS 的访问服务

**Q: 常用 Ingress 控制器有那些?**  
其中ingress controller目前主要有两种基于nginx服务的ingress controller (四层或者七层)和基于traefik的ingress controller(目前支持http和https协议)，其它更多适用于Kubernetes的ingress控制器可以参考地址\[https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers/#其他控制器\]

-   ingress-Nginx : 用于 Nginx Kubernetes Ingress 控制器能够与 NGINX Web 服务器（作为代理）一起使用 (推荐)
    
-   ingress-Traefik ：由 Traefik Kubernetes Ingress 提供程序是一个用于 Traefik 代理的Ingress控制器。
    
-   ingress-istio : Istio Ingress 是一个基于Istio的Ingress控制器。
    

温馨提示: 理想情况下所有 Ingress 控制器都应符合参考规范。实际上各种 Ingress 控制器的操作略有不同,请参考相应Ingress的控制器官方文档。

如下图所示的一个简单的示例，客户端请求访问外部URL地址, Ingress 将其所有流量发送到一个Service中, 后端 Pod 提供服务端响应通过路由进行返回给客户端。

![WeiyiGeek.Ingress](https://i0.hdslb.com/bfs/article/03da8e2f236bc355c57b481aedc894599c710e00.png@942w_564h_progressive.webp)

温馨提示: 基于nginx服务的ingress controller根据不同的开发公司，又分为k8s社区的ingres-nginx和nginx公司的nginx-ingress，在此根据github上的活跃度和关注人数，我们选择的是k8s社区的ingres-nginx。

-   k8s社区提供的ingress，github地址如下：https://github.com/kubernetes/ingress-nginx
    
-   nginx社区提供的ingress，github地址如下：https://github.com/nginxinc/kubernetes-ingress
    

**Q: K8S社区提供 Ingress 规则有哪些?**

-   host : 虚拟主机名称, 主机名通配符主机可以是精确匹配（例如"foo.bar.com"）或通配符（例如“ \*.foo.com”）
    
-   paths : URL访问路径。
    
-   pathType : Ingress 中的每个路径都需要有对应的路径类型（Path Type）
    
-   backend : 是 Service 文档中所述的服务和端口名称的组合与规则的 host 和 path 匹配的对 Ingress 的 HTTP（和 HTTPS ）请求将发送到列出的 backend, 一般情况可以单独为路径设置Backend以及未匹配的url默认访问的后端defaultBackend。
    

Ingress 中的每个路径都需要有对应的路径类型（Path Type），未明确设置 pathType 的路径无法通过合法性检查，当前支持的路径类型有三种：

-   Exact：精确匹配 URL 路径，且区分大小写。
    
-   Prefix：基于以`/`分隔的URL路径前缀匹配, 且区分大小写，并且对路径中的元素逐个完成。
    
-   ImplementationSpecific：此路径类型匹配方法取决于 IngressClass, 具体实现可以将其作为单独的 pathType 处理或者与 Prefix 、 Exact 类型作相同处理。
    

说明： 如果路径的最后一个元素是请求路径中最后一个元素的子字符串，则不会匹配 （例如：/foo/bar 匹配 /foo/bar/baz, 但不匹配 /foo/barbaz）。

温馨提示: defaultBackend 通常在 Ingress 控制器中配置，以服务与规范中的路径不匹配的任何请求。

```
tee > test.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  defaultBackend:
    service:
      name: test
      port:
        number: 80
EOF
kubectl apply -f
```

注意, 入口控制器和负载平衡器可能需要一两分钟才能分配 IP 地址。 在此之前，你通常会看到地址字段的值被设定为 `<pending>`。

温馨提示: ingress控制器资源清单在v1.19以及之后版本集群中写法如下

```
cat <<-EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foo.bar.com
  namespace: default
  annotations:
    #URL重定向。
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
    # 在Ingress Controller的版本≥0.22.0之后，path中需要使用正则表达式定义路径，并在rewrite-target中结合捕获组一起使用。
      - path: /svc(/|$)(.*)
        backend:
          service:
            name: web1-service
            port:
              number: 80
        pathType: ImplementationSpecific
EOF
```

**Q: Nginx Ingress Controller与K8S集群版本对应关系**  
参考地址: https://github.com/kubernetes/ingress-nginx#support-versions-table

```
Support Versions table
Ingress-NGINX version k8s supported version Alpine Version Nginx Version
v1.2.1 1.23, 1.22, 1.21, 1.20, 1.19 3.14.6 1.19.10†
```

**快速安装配置**  
环境依赖说明: `Chart version 4.x.x and above: Kubernetes v1.19+`

步骤 01.如果您有 Helm，则可以使用以下命令部署入口控制器：

```
helm3 repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm3 repo update
helm3 search repo ingress-nginx -l
  # NAME                            CHART VERSION   APP VERSION     DESCRIPTION
  # ingress-nginx/ingress-nginx     4.1.4           1.2.1           Ingress controller for Kubernetes using NGINX a...
  # ingress-nginx/ingress-nginx     4.1.3           1.2.1           Ingress controller for Kubernetes using NGINX a...

# ingress-nginx Chart 模板参数配置
helm3 show values --version 4.1.4 ingress-nginx/ingress-nginx > values.yaml

#  values.yaml 配置修改完毕后安装 ingress-nginx
helm3 install ingress-nginx -f values.yaml --version 4.1.4 ingress-nginx/ingress-nginx  --namespace ingress-nginx --create-namespace --debug
```

温馨提示: 上面在不需进行可用参数配置时可采用一条命令搞定(注意提前拉取相关镜像)

```
helm3 upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

温馨提示: 使用如下更新helm图表部署的应用以更新修改为国内镜像源。

```
$ crictl pull registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.2.1
$ crictl pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1

helm3 upgrade ingress-nginx --namespace ingress-nginx --version 4.1.4 -f values.yaml ingress-nginx/ingress-nginx --debug
```

步骤 02.查看ingress-nginx的部署情况，一些 pod 应该在 ingress-nginx 命名空间中启动：

```
helm3 list -n ingress-nginx
  # NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                  APP VERSION
  # ingress-nginx   ingress-nginx   1               2022-06-24 13:12:36.692931823 +0800 CST deployed        ingress-nginx-4.1.4    1.2.1
helm3 history ingress-nginx  -n ingress-nginx
  # REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
  # 1               Fri Jun 24 13:20:49 2022        deployed        ingress-nginx-4.1.4     1.2.1           Install complete

# 等待 Pod 应用部署为正常
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
  # pod/ingress-nginx-controller-7cfd586878-fkdb5 condition met

# 查看 ingress-nginx 的 Pod 与 SVC
kubectl get pod,svc -n ingress-nginx
  # NAME                                               READY   STATUS      RESTARTS   AGE
  # pod/ingress-ingress-nginx-admission-create-pjm6x   0/1     Completed   0          7m47s
  # pod/ingress-nginx-controller-7cfd586878-fkdb5      1/1     Running     0          18m

  # NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
  # service/ingress-nginx-controller             NodePort    10.107.81.40     <none>        80:30080/TCP,443:30443/TCP   18m
  # service/ingress-nginx-controller-admission   ClusterIP   10.106.154.244   <none>        443/TCP                      18m

# 查看 ingress 的 ingressclasses 名称后续才能使用。
kubectl get ingressclasses.networking.k8s.io
  NAME    CONTROLLER             PARAMETERS   AGE
  nginx   k8s.io/ingress-nginx   <none>       19m
```

步骤 03.针对搭建的 ingress-nginx 服务验证, 此处访问应该是NGINX标准404错误页面,因为

```
# Get the application URL by running these commands:
export HTTP_NODE_PORT=30080
export HTTPS_NODE_PORT=30443
export NODE_IP=$(kubectl --namespace ingress-nginx get nodes -o jsonpath="{.items[0].status.addresses[1].address}")
echo "Visit http://$NODE_IP:$HTTP_NODE_PORT to access your application via HTTP."
echo "Visit https://$NODE_IP:$HTTPS_NODE_PORT to access your application via HTTPS."

# 尝试访问 ingress - nginx
$ curl -I --insecure http://devtest-master-212:30080
$ curl -I --insecure https://devtest-master-212:30443
  # HTTP/2 404
  # date: Fri, 24 Jun 2022 07:07:12 GMT
  # content-type: text/html
  # content-length: 146
  # strict-transport-security: max-age=15724800; includeSubDomains
```

步骤 04.优化ingress Pod内核参数以及扩容Pod到每个节点之上, 更多优化可以参考 \[https://blog.weiyigeek.top/2020/5-28-588.html#0x07-Kubernetes中ingress-nginx优化配置\]

```
# 编辑 ingress-nginx-controller资源清单添加一个 initContainers 进行内核参数初始化
$ kubectl edit deployments.apps -n ingress-nginx ingress-nginx-controller
....
  initContainers:
  - command:
    - sh
    - -c
    - |
      mount -o remount rw /proc/sys
      sysctl -w net.core.somaxconn=65535
      sysctl -w net.ipv4.tcp_tw_reuse=1
      sysctl -w net.ipv4.ip_local_port_range="1024 65535"
      sysctl -w fs.file-max=1048576
      sysctl -w fs.inotify.max_user_instances=16384
      sysctl -w fs.inotify.max_user_watches=524288
      sysctl -w fs.inotify.max_queued_events=16384
    image: alpine:3.10
    imagePullPolicy: IfNotPresent
    name: sysctl
    resources: {}
    securityContext:
      privileged: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
....

# 扩容 ingress-nginx-controller 副本为节点数量 (注意此处已去除master节点污点)
kubectl scale deployment --replicas=4 -n ingress-nginx ingress-nginx-controller

# 查看 ingress-controller 的Pod
kubectl get pod -n ingress-nginx -l app.kubernetes.io/component=controller -o wide
  NAME                                        READY   STATUS    RESTARTS   AGE     IP              NODE                 NOMINATED NODE   READINESS GATES
  ingress-nginx-controller-7f8cd9c4fc-6lqgw   1/1     Running   0          5m33s   10.66.55.65     devtest-master-213   <none>           <none>
  ingress-nginx-controller-7f8cd9c4fc-8svfj   1/1     Running   0          3m16s   10.66.53.79     devtest-work-215     <none>           <none>
  ingress-nginx-controller-7f8cd9c4fc-g75fz   1/1     Running   0          6m40s   10.66.237.129   devtest-master-212   <none>           <none>
  ingress-nginx-controller-7f8cd9c4fc-jl5qg   1/1     Running   0          5m33s   10.66.35.67     devtest-master-214   <none>           <none>
```

![WeiyiGeek.ingress-initContainers](https://i0.hdslb.com/bfs/article/9b3be6d77a5cef9c063ecf216d7841399f008ff3.png@767w_480h_progressive.webp)

本文至此完毕，更多技术文章，尽情期待下一章节！

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

完整原文地址：https://blog.weiyigeek.top/2022/6-7-664.html

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

**已发布的相关历史文章（点击即可进入）**

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

[运维实践-我在B站学源码编译Nginx利用GeoIP2模块实现显示IP所属地及处理不同地区访问](https://www.bilibili.com/read/cv18082692)

[](https://www.bilibili.com/read/cv18072582)[运维实践-我在B站学在源码编译Nginx里使用lua-nginx模块解析Lua脚本访问Redis数据库](https://www.bilibili.com/read/cv18072582)

[工作效率-我在B站学Markdown轻量级标记语言十五分钟让你快速学习语法到精通排版实践](https://www.bilibili.com/read/cv18072190)

[我在B站学习图像处理之使用EasyOCR库对行程码图片进行批量OCR文字识别](https://www.bilibili.com/read/cv16911816)

[使用NextCloud搭建私有网络云盘并支持Office文档在线预](https://www.bilibili.com/read/cv16835328)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

[9.我在B站学云原生之Kubernetes入门实践Pod模板创建、静态Pod以及Pod生命周期介绍](https://www.bilibili.com/read/cv16634321)

[](https://www.bilibili.com/read/cv16625708)[8.我在B站学云原生之Kubernetes入门实践资源清单与Namespace名称空间介绍](https://www.bilibili.com/read/cv16625708)

[7.我在B站学云原生之Kubernetes手把手教你使用二进制](https://www.bilibili.com/read/cv16625496)[方式部署K8S集群v1.23.6实践(下)](https://www.bilibili.com/read/cv16625253)

[7.我在B站学云原生之Kubernetes手把手教你使用二进制方式部署K8S集群v1.23.6实践(上)](https://www.bilibili.com/read/cv16625253)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】或者 关注 UP主【WeiyiGeek】联系我。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源于【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 个人博客。

**个人主页:** 【 https://weiyigeek.top 】  

**博客地址:** 【 https://blog.weiyigeek.top 】

![](https://i0.hdslb.com/bfs/article/8b134e8e411630dce6289ee89a08b3fc6edaa032.png@942w_549h_progressive.webp)

https://weiyigeek.top - Always keep a beginner's mind, don't forget the beginner's mind

专栏书写不易，如果您觉得这个专栏还不错的，请给这篇专栏 **【****点个赞、投个币、收个藏、关个注，转个发，留个言】(人间六大情)**，这将对我的肯定，谢谢！。

-   echo  "【点个赞】，动动你那粗壮的拇指或者芊芊玉手，亲！"
    
-   printf("%s", "【投个币】，万水千山总是情，投个硬币行不行，亲！")
    
-   fmt.Printf("【收个藏】，阅后即焚不吃灰，亲！")
    
-   System.out.println("【关个注】，后续浏览查看不迷路哟，亲！")
    
-   console.info("【转个发】，让更多的志同道合的朋友一起学习交流，亲！")
    
-   cout << "【留个言】，文章写得好不好、有没有错误，一定要留言哟，亲! " << endl;
    

GIF

![](https://i0.hdslb.com/bfs/article/11a629d1bc4369dc810216c5dedac871136167d7.gif@1s.webp)

谢谢，各位帅哥、美女四连支持！！这就是我的动力！

更多网络安全、系统运维、应用开发、物联网实践、网络工程、全栈文章，尽在 【https://blog.weiyigeek.top】 之中，谢谢各位看友支持！