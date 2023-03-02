## [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E4%B8%80%E3%80%81%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87 "一、环境准备")一、环境准备[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E4%B8%80%E3%80%81%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)

-   Ubuntu 20.04 x5
-   Etcd 3.4.16
-   Kubernetes 1.21.1
-   Containerd 1.3.3

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#1-1%E3%80%81%E5%A4%84%E7%90%86-IPVS "1.1、处理 IPVS")1.1、处理 IPVS[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#1-1%E3%80%81%E5%A4%84%E7%90%86-IPVS)

由于 Kubernetes 新版本 Service 实现切换到 IPVS，所以需要确保内核加载了 IPVS modules；以下命令将设置系统启动自动加载 IPVS 相关模块，执行完成后需要重启。

```
# Kernel modules
cat > /etc/modules-load.d/50-kubernetes.conf <<EOF
# Load some kernel modules needed by kubernetes at boot
nf_conntrack
br_netfilter
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
EOF

# sysctl
cat > /etc/sysctl.d/50-kubernetes.conf <<EOF
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
fs.inotify.max_user_watches=525000
EOF
```

重启完成后务必检查相关 module 加载以及内核参数设置:

```
# check ipvs modules
➜ ~ lsmod | grep ip_vs
ip_vs_sed              16384  0
ip_vs_nq               16384  0
ip_vs_fo               16384  0
ip_vs_sh               16384  0
ip_vs_dh               16384  0
ip_vs_lblcr            16384  0
ip_vs_lblc             16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs_wlc              16384  0
ip_vs_lc               16384  0
ip_vs                 155648  22 ip_vs_wlc,ip_vs_rr,ip_vs_dh,ip_vs_lblcr,ip_vs_sh,ip_vs_fo,ip_vs_nq,ip_vs_lblc,ip_vs_wrr,ip_vs_lc,ip_vs_sed
nf_conntrack          139264  1 ip_vs
nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
libcrc32c              16384  5 nf_conntrack,btrfs,xfs,raid456,ip_vs

# check sysctl
➜ ~ sysctl -a | grep ip_forward
net.ipv4.ip_forward = 1
net.ipv4.ip_forward_update_priority = 1
net.ipv4.ip_forward_use_pmtu = 0

➜ ~ sysctl -a | grep bridge-nf-call
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#1-2%E3%80%81%E5%AE%89%E8%A3%85-Containerd "1.2、安装 Containerd")1.2、安装 Containerd[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#1-2%E3%80%81%E5%AE%89%E8%A3%85-Containerd)

Containerd 在 Ubuntu 20 中已经在默认官方仓库中包含，所以只需要 apt 安装即可:

```
# 其他软件包后面可能会用到，所以顺手装了
apt install containerd bridge-utils nfs-common tree -y
```

安装成功后可以通过执行 `ctr images ls` 命令验证，**本章节不会对 Containerd 配置做说明，Containerd 配置文件将在 Kubernetes 安装时进行配置。**

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-1%E3%80%81%E5%AE%89%E8%A3%85-Etcd-%E9%9B%86%E7%BE%A4 "2.1、安装 Etcd 集群")2.1、安装 Etcd 集群[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-1%E3%80%81%E5%AE%89%E8%A3%85-Etcd-%E9%9B%86%E7%BE%A4)

Etcd 对于 Kubernetes 来说是核心中的核心，所以个人还是比较喜欢在宿主机安装；宿主机安装情况下为了方便我打包了一些 **`*-pack`** 的工具包，用于快速处理:

安装 CFSSL 和 ETCD

```
# 下载安装包
wget https://github.com/mritd/etcd-pack/releases/download/v3.4.16/etcd_v3.4.16.run
wget https://github.com/mritd/cfssl-pack/releases/download/v1.5.0/cfssl_v1.5.0.run

# 安装 cfssl 和 etcd
chmod +x *.run
./etcd_v3.4.16.run install
./cfssl_v1.5.0.run install
```

安装完成后，**自行调整 `/etc/cfssl/etcd/etcd-csr.json` 相关 IP**，然后执行同目录下 `create.sh` 生成证书。

```
➜ ~ cat /etc/cfssl/etcd/etcd-csr.json
{
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "O": "etcd",
            "OU": "etcd Security",
            "L": "Beijing",
            "ST": "Beijing",
            "C": "CN"
        }
    ],
    "CN": "etcd",
    "hosts": [
        "127.0.0.1",
        "localhost",
        "*.etcd.node",
        "*.kubernetes.node",
        "10.0.0.11",
        "10.0.0.12",
        "10.0.0.13"
    ]
}

# 复制到 3 台 master
➜ ~ for ip in `seq 1 3`; do scp /etc/cfssl/etcd/*.pem root@10.0.0.1$ip:/etc/etcd/ssl; done
```

证书生成完成后调整每台机器的 Etcd 配置文件，然后修复权限启动。

```
# 复制配置
for ip in `seq 1 3`; do scp /etc/etcd/etcd.cluster.yaml root@10.0.0.1$ip:/etc/etcd/etcd.yaml; done

# 修复权限
for ip in `seq 1 3`; do ssh root@10.0.0.1$ip chown -R etcd:etcd /etc/etcd; done

# 每台机器启动
systemctl start etcd
```

启动完成后通过 `etcdctl` 验证集群状态:

```
# 稳妥点应该执行 etcdctl endpoint health
➜ ~ etcdctl member list
55fcbe0adaa45350, started, etcd3, https://10.0.0.13:2380, https://10.0.0.13:2379, false
cebdf10928a06f3c, started, etcd1, https://10.0.0.11:2380, https://10.0.0.11:2379, false
f7a9c20602b8532e, started, etcd2, https://10.0.0.12:2380, https://10.0.0.12:2379, false
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-2%E3%80%81%E5%AE%89%E8%A3%85-kubeadm "2.2、安装 kubeadm")2.2、安装 kubeadm[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-2%E3%80%81%E5%AE%89%E8%A3%85-kubeadm)

kubeadm 国内用户建议使用 aliyun 的安装源:

```
# kubeadm
apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt update

# ebtables、ethtool kubelet 可能会用，具体忘了，反正从官方文档上看到的
apt install kubelet kubeadm kubectl ebtables ethtool -y
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-3%E3%80%81%E5%AE%89%E8%A3%85-kube-apiserver-proxy "2.3、安装 kube-apiserver-proxy")2.3、安装 kube-apiserver-proxy[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-3%E3%80%81%E5%AE%89%E8%A3%85-kube-apiserver-proxy)

kube-apiserver-proxy 是我自己编译的一个仅开启四层代理的 Nginx，其主要负责监听 `127.0.0.1:6443` 并负载到所有的 Api Server 地址(`0.0.0.0:5443`):

```
wget https://github.com/mritd/kube-apiserver-proxy-pack/releases/download/v1.20.0/kube-apiserver-proxy_v1.20.0.run
chmod +x *.run
./kube-apiserver-proxy_v1.20.0.run install
```

安装完成后根据 IP 地址不同自行调整 Nginx 配置文件，然后启动:

```
➜ ~ cat /etc/kubernetes/apiserver-proxy.conf
error_log syslog:server=unix:/dev/log notice;

worker_processes auto;
events {
        multi_accept on;
        use epoll;
        worker_connections 1024;
}

stream {
    upstream kube_apiserver {
        least_conn;
        server 10.0.0.11:5443;
        server 10.0.0.12:5443;
        server 10.0.0.13:5443;
    }

    server {
        listen        0.0.0.0:6443;
        proxy_pass    kube_apiserver;
        proxy_timeout 10m;
        proxy_connect_timeout 1s;
    }
}

systemctl start kube-apiserver-proxy
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4%E3%80%81%E5%AE%89%E8%A3%85-kubeadm-config "2.4、安装 kubeadm-config")2.4、安装 kubeadm-config[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4%E3%80%81%E5%AE%89%E8%A3%85-kubeadm-config)

kubeadm-config 是一系列配置文件的组合以及 kubeadm 安装所需的必要镜像文件的打包，安装完成后将会自动配置 Containerd、ctrictl 等:

```
wget https://github.com/mritd/kubeadm-config-pack/releases/download/v1.21.1/kubeadm-config_v1.21.1.run
chmod +x *.run

# --load 选项用于将 kubeadm 所需镜像 load 到 containerd 中
./kubeadm-config_v1.21.1.run install --load
```

#### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-1%E3%80%81containerd-%E9%85%8D%E7%BD%AE "2.4.1、containerd 配置")2.4.1、containerd 配置[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-1%E3%80%81containerd-%E9%85%8D%E7%BD%AE)

Containerd 配置位于 `/etc/containerd/config.toml`，其配置如下:

```
version = 2
# 指定存储根目录
root = "/data/containerd"
state = "/run/containerd"
# OOM 评分
oom_score = -999

[grpc]
  address = "/run/containerd/containerd.sock"

[metrics]
  address = "127.0.0.1:1234"

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    # sandbox 镜像
    sandbox_image = "k8s.gcr.io/pause:3.4.1"
    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          # 开启 systemd cgroup
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
```

#### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-2%E3%80%81crictl-%E9%85%8D%E7%BD%AE "2.4.2、crictl 配置")2.4.2、crictl 配置[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-2%E3%80%81crictl-%E9%85%8D%E7%BD%AE)

在切换到 Containerd 以后意味着以前的 `docker` 命令将不再可用，containerd 默认自带了一个 `ctr` 命令，同时 CRI 规范会自带一个 `crictl` 命令；`crictl` 命令配置文件存放在 `/etc/crictl.yaml` 中:

```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
pull-image-on-create: true
```

#### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-3%E3%80%81kubeadm-%E9%85%8D%E7%BD%AE "2.4.3、kubeadm 配置")2.4.3、kubeadm 配置[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-4-3%E3%80%81kubeadm-%E9%85%8D%E7%BD%AE)

kubeadm 配置目前分为 2 个，一个是用于首次引导启动的 init 配置，另一个是用于其他节点 join 到 master 的配置；其中比较重要的 init 配置如下:

```
# /etc/kubernetes/kubeadm.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
# kubeadm token create
bootstrapTokens:
- token: "c2t0rj.cofbfnwwrb387890"
nodeRegistration:
  # CRI 地址(Containerd)
  criSocket: unix:///run/containerd/containerd.sock
  kubeletExtraArgs:
    runtime-cgroups: "/system.slice/containerd.service"
    rotate-server-certificates: "true"
localAPIEndpoint:
  advertiseAddress: "10.0.0.11"
  bindPort: 5443
# kubeadm certs certificate-key
certificateKey: 31f1e534733a1607e5ba67b2834edd3a7debba41babb1fac1bee47072a98d88b
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
clusterName: "kuberentes"
kubernetesVersion: "v1.21.1"
certificatesDir: "/etc/kubernetes/pki"
# Other components of the current control plane only connect to the apiserver on the current host.
# This is the expected behavior, see: https://github.com/kubernetes/kubeadm/issues/2271
controlPlaneEndpoint: "127.0.0.1:6443"
etcd:
  external:
    endpoints:
    - "https://10.0.0.11:2379"
    - "https://10.0.0.12:2379"
    - "https://10.0.0.13:2379"
    caFile: "/etc/etcd/ssl/etcd-ca.pem"
    certFile: "/etc/etcd/ssl/etcd.pem"
    keyFile: "/etc/etcd/ssl/etcd-key.pem"
networking:
  serviceSubnet: "10.66.0.0/16"
  podSubnet: "10.88.0.1/16"
  dnsDomain: "cluster.local"
apiServer:
  extraArgs:
    v: "4"
    alsologtostderr: "true"
#    audit-log-maxage: "21"
#    audit-log-maxbackup: "10"
#    audit-log-maxsize: "100"
#    audit-log-path: "/var/log/kube-audit/audit.log"
#    audit-policy-file: "/etc/kubernetes/audit-policy.yaml"
    authorization-mode: "Node,RBAC"
    event-ttl: "720h"
    runtime-config: "api/all=true"
    service-node-port-range: "30000-50000"
    service-cluster-ip-range: "10.66.0.0/16"
#    insecure-bind-address: "0.0.0.0"
#    insecure-port: "8080"
    # The fraction of requests that will be closed gracefully(GOAWAY) to prevent
    # HTTP/2 clients from getting stuck on a single apiserver.
    goaway-chance: "0.001"
#  extraVolumes:
#  - name: "audit-config"
#    hostPath: "/etc/kubernetes/audit-policy.yaml"
#    mountPath: "/etc/kubernetes/audit-policy.yaml"
#    readOnly: true
#    pathType: "File"
#  - name: "audit-log"
#    hostPath: "/var/log/kube-audit"
#    mountPath: "/var/log/kube-audit"
#    pathType: "DirectoryOrCreate"
  certSANs:
  - "*.kubernetes.node"
  - "10.0.0.11"
  - "10.0.0.12"
  - "10.0.0.13"
  timeoutForControlPlane: 1m
controllerManager:
  extraArgs:
    v: "4"
    node-cidr-mask-size: "19"
    deployment-controller-sync-period: "10s"
    experimental-cluster-signing-duration: "8670h"
    node-monitor-grace-period: "20s"
    pod-eviction-timeout: "2m"
    terminated-pod-gc-threshold: "30"
scheduler:
  extraArgs:
    v: "4"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
failSwapOn: false
oomScoreAdj: -900
cgroupDriver: "systemd"
kubeletCgroups: "/system.slice/kubelet.service"
nodeStatusUpdateFrequency: 5s
rotateCertificates: true
evictionSoft:
  "imagefs.available": "15%"
  "memory.available": "512Mi"
  "nodefs.available": "15%"
  "nodefs.inodesFree": "10%"
evictionSoftGracePeriod:
  "imagefs.available": "3m"
  "memory.available": "1m"
  "nodefs.available": "3m"
  "nodefs.inodesFree": "1m"
evictionHard:
  "imagefs.available": "10%"
  "memory.available": "256Mi"
  "nodefs.available": "10%"
  "nodefs.inodesFree": "5%"
evictionMaxPodGracePeriod: 30
imageGCLowThresholdPercent: 70
imageGCHighThresholdPercent: 80
kubeReserved:
  "cpu": "500m"
  "memory": "512Mi"
  "ephemeral-storage": "1Gi"
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
# kube-proxy specific options here
clusterCIDR: "10.88.0.1/16"
mode: "ipvs"
oomScoreAdj: -900
ipvs:
  minSyncPeriod: 5s
  syncPeriod: 5s
  scheduler: "wrr"
```

init 配置具体含义请自行参考官方文档，相对于 init 配置，join 配置比较简单，**不过需要注意的是如果需要 join 为 master 则需要 `controlPlane` 这部分，否则请注释掉 `controlPlane`。**

```
# /etc/kubernetes/kubeadm-join.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
controlPlane:
  localAPIEndpoint:
    advertiseAddress: "10.0.0.12"
    bindPort: 5443
  certificateKey: 31f1e534733a1607e5ba67b2834edd3a7debba41babb1fac1bee47072a98d88b
discovery:
  bootstrapToken:
    apiServerEndpoint: "127.0.0.1:6443"
    token: "c2t0rj.cofbfnwwrb387890"
    # Please replace with the "--discovery-token-ca-cert-hash" value printed
    # after the kubeadm init command is executed successfully
    caCertHashes:
    - "sha256:97590810ae34a82501717e33acfca76f16044f1a365c5ad9a1c66433c386c75c"
nodeRegistration:
  criSocket: unix:///run/containerd/containerd.sock
  kubeletExtraArgs:
    runtime-cgroups: "/system.slice/containerd.service"
    rotate-server-certificates: "true"
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-5%E3%80%81%E6%8B%89%E8%B5%B7-master "2.5、拉起 master")2.5、拉起 master[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-5%E3%80%81%E6%8B%89%E8%B5%B7-master)

在调整好配置后，拉起 master 节点只需要一条命令:

```
kubeadm init --config /etc/kubernetes/kubeadm.yaml --upload-certs --ignore-preflight-errors=Swap
```

拉起完成后记得保存相关 Token 以便于后续使用。

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-6%E3%80%81%E6%8B%89%E8%B5%B7%E5%85%B6%E4%BB%96-master "2.6、拉起其他 master")2.6、拉起其他 master[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-6%E3%80%81%E6%8B%89%E8%B5%B7%E5%85%B6%E4%BB%96-master)

在第一个 master 启动完成后，使用 `join` 命令让其他 master 加入即可；**需要注意的是 `kubeadm-join.yaml` 配置中需要替换 `caCertHashes` 为第一个 master 拉起后的 `discovery-token-ca-cert-hash` 的值。**

```
kubeadm join 127.0.0.1:6443 --config /etc/kubernetes/kubeadm-join.yaml --ignore-preflight-errors=Swap
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-7%E3%80%81%E6%8B%89%E8%B5%B7%E5%85%B6%E4%BB%96-node "2.7、拉起其他 node")2.7、拉起其他 node[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-7%E3%80%81%E6%8B%89%E8%B5%B7%E5%85%B6%E4%BB%96-node)

node 节点拉起与拉起其他 master 节点一样，唯一不同的是需要注释掉配置中的 `controlPlane` 部分。

```
# /etc/kubernetes/kubeadm-join.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
#controlPlane:
#  localAPIEndpoint:
#    advertiseAddress: "10.0.0.12"
#    bindPort: 5443
#  certificateKey: 31f1e534733a1607e5ba67b2834edd3a7debba41babb1fac1bee47072a98d88b
discovery:
  bootstrapToken:
    apiServerEndpoint: "127.0.0.1:6443"
    token: "c2t0rj.cofbfnwwrb387890"
    # Please replace with the "--discovery-token-ca-cert-hash" value printed
    # after the kubeadm init command is executed successfully
    caCertHashes:
    - "sha256:97590810ae34a82501717e33acfca76f16044f1a365c5ad9a1c66433c386c75c"
nodeRegistration:
  criSocket: unix:///run/containerd/containerd.sock
  kubeletExtraArgs:
    runtime-cgroups: "/system.slice/containerd.service"
    rotate-server-certificates: "true"
```

```
kubeadm join 127.0.0.1:6443 --config /etc/kubernetes/kubeadm-join.yaml --ignore-preflight-errors=Swap
```

### [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-8%E3%80%81%E5%85%B6%E4%BB%96%E5%A4%84%E7%90%86 "2.8、其他处理")2.8、其他处理[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#2-8%E3%80%81%E5%85%B6%E4%BB%96%E5%A4%84%E7%90%86)

由于 kubelet 开启了证书轮转，所以新集群会有大量 csr 请求，批量允许即可:

```
kubectl get csr | grep Pending | awk '{print $1}' | xargs kubectl certificate approve
```

同时为了 master 节点也能负载 pod，需要调整污点:

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

后续 CNI 等不在本文内容范围内。

## [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E4%B8%89%E3%80%81Containerd-%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C "三、Containerd 常用操作")三、Containerd 常用操作[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E4%B8%89%E3%80%81Containerd-%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)

```
# 列出镜像
ctr images ls

# 列出 k8s 镜像
ctr -n k8s.io images ls

# 导入镜像
ctr -n k8s.io images import xxxx.tar

# 导出镜像
ctr -n k8s.io images export kube-scheduler.tar k8s.gcr.io/kube-scheduler:v1.21.1
```

## [](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E5%9B%9B%E3%80%81%E8%B5%84%E6%BA%90%E4%BB%93%E5%BA%93 "四、资源仓库")四、资源仓库[](https://mritd.com/2021/05/29/use-containerd-with-kubernetes/#%E5%9B%9B%E3%80%81%E8%B5%84%E6%BA%90%E4%BB%93%E5%BA%93)

本文中所有 `*-pack` 仓库地址如下:

-   [https://github.com/mritd/cfssl-pack](https://github.com/mritd/cfssl-pack)
-   [https://github.com/mritd/etcd-pack](https://github.com/mritd/etcd-pack)
-   [https://github.com/mritd/kube-apiserver-proxy-pack](https://github.com/mritd/kube-apiserver-proxy-pack)
-   [https://github.com/mritd/kubeadm-config-pack](https://github.com/mritd/kubeadm-config-pack)