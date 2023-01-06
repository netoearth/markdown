## sealos简介

sealos是一个纯golang开发的极简[kubernetes](https://so.csdn.net/so/search?q=kubernetes&spm=1001.2101.3001.7020)高可用集群部署工具，一个二进制工具加一个资源包，不依赖haproxy keepalived ansible等重量级工具，一条命令就可实现kubernetes高可用集群构建，无论是单节点还是集群，单master还是多master，生产还是测试都能很好支持！简单不意味着阉割功能，照样能全量支持kubeadm所有配置。

支持99年证书，不用担心生产集群证书过期，ipvs负载多master可用性与稳定性更高，已经有上千用户在生产环境中使用sealos, 有超过上千台服务器生产环境中使用sealos。

底层架构：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421133826368.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70)

官方网站：[https://sealyun.com/](https://sealyun.com/)

项目地址：[https://github.com/fanux/sealos](https://github.com/fanux/sealos)

## 部署kubernetes集群

准备节点清单，部署3个master节点1个worker节点：

| 节点名称 | IP地址 | OS版本 | 角色 |
| --- | --- | --- | --- |
| node01 | 192.168.72.50 | ubuntu 22.04 LTS | master |
| node02 | 192.168.72.51 | ubuntu 22.04 LTS | master |
| node03 | 192.168.72.52 | ubuntu 22.04 LTS | master |
| node04 | 192.168.72.53 | ubuntu 22.04 LTS | worker |

所有节点必须配置不同[主机名](https://so.csdn.net/so/search?q=%E4%B8%BB%E6%9C%BA%E5%90%8D&spm=1001.2101.3001.7020)

```
hostnamectl set-hostname xxx
```

所有节点[时间同步](https://so.csdn.net/so/search?q=%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5&spm=1001.2101.3001.7020)

```
yum install -y chrony
systemctl enable --now chronyd  
timedatectl set-timezone Asia/Shanghai
```

第一个node节点安装sealos工具：

```
wget -c https://sealyun-home.oss-cn-beijing.aliyuncs.com/sealos-4.0/latest/sealos-amd64 -O sealos && \
    chmod +x sealos && mv sealos /usr/bin
```

第一个node节点执行以下命令部署kubernetes集群：

```
sealos run labring/kubernetes:v1.24.2 labring/calico:v3.22.1 \
     --masters 192.168.72.50,192.168.72.51,192.168.72.52 \
     --nodes 192.168.72.53 -p 123456
```

参数说明：

-   `labring/kubernetes:v1.24.2`: dockerhub中的镜像，为kubernetes核心组件
-   `labring/calico:v3.22.1`: dockerhub中的镜像，为kubernetes网络插件
-   `-p`：为所有节点root密码

查看集群节点信息

```
root@node01:~# kubectl get nodes -o wide
NAME     STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
node01   Ready    control-plane   4h27m   v1.24.2   192.168.72.50   <none>        Ubuntu 22.04 LTS   5.15.0-27-generic   containerd://1.6.2
node02   Ready    control-plane   4h26m   v1.24.2   192.168.72.51   <none>        Ubuntu 22.04 LTS   5.15.0-27-generic   containerd://1.6.2
node03   Ready    control-plane   4h25m   v1.24.2   192.168.72.52   <none>        Ubuntu 22.04 LTS   5.15.0-27-generic   containerd://1.6.2
node04   Ready    <none>          4h24m   v1.24.2   192.168.72.53   <none>        Ubuntu 22.04 LTS   5.15.0-27-generic   containerd://1.6.2
```

查看集群pods信息

```
root@node01:~# kubectl get pods -A
NAMESPACE         NAME                                       READY   STATUS    RESTARTS        AGE
calico-system     calico-kube-controllers-6b44b54755-9x2hb   1/1     Running   0               3m20s
calico-system     calico-node-bz6ff                          1/1     Running   0               3m20s
calico-system     calico-node-kwz5f                          1/1     Running   0               3m20s
calico-system     calico-node-sm52x                          1/1     Running   0               3m20s
calico-system     calico-node-svqkp                          1/1     Running   0               3m20s
calico-system     calico-typha-845987b4fc-7tv9p              1/1     Running   0               3m21s
calico-system     calico-typha-845987b4fc-xg5mx              1/1     Running   0               3m10s
kube-system       coredns-6d4b75cb6d-rx9fd                   1/1     Running   0               6m11s
kube-system       coredns-6d4b75cb6d-vsdcf                   1/1     Running   0               6m11s
kube-system       etcd-node01                                1/1     Running   0               6m25s
kube-system       etcd-node02                                1/1     Running   0               5m1s
kube-system       etcd-node03                                1/1     Running   0               4m
kube-system       kube-apiserver-node01                      1/1     Running   0               6m22s
kube-system       kube-apiserver-node02                      1/1     Running   0               4m59s
kube-system       kube-apiserver-node03                      1/1     Running   0               3m54s
kube-system       kube-controller-manager-node01             1/1     Running   1 (2m46s ago)   6m22s
kube-system       kube-controller-manager-node02             1/1     Running   0               4m5s
kube-system       kube-controller-manager-node03             1/1     Running   0               3m59s
kube-system       kube-proxy-6m2nw                           1/1     Running   0               3m39s
kube-system       kube-proxy-7ngqv                           1/1     Running   0               6m11s
kube-system       kube-proxy-dkqkz                           1/1     Running   0               5m12s
kube-system       kube-proxy-xvzh2                           1/1     Running   0               4m1s
kube-system       kube-scheduler-node01                      1/1     Running   1 (5m1s ago)    6m22s
kube-system       kube-scheduler-node02                      1/1     Running   1 (2m37s ago)   3m46s
kube-system       kube-scheduler-node03                      1/1     Running   0               3m56s
kube-system       kube-sealyun-lvscare-node04                1/1     Running   0               3m32s
tigera-operator   tigera-operator-d7957f5cc-wsndk            1/1     Running   1 (2m45s ago)   3m32s
```

其他常用命令

-   sealos reset：清理集群
-   sealos add：增加节点
-   sealos delete：删除节点
-   sealos save：保存离线镜像包
-   sealos load：加载离线镜像包

## sealos设计原理

本地负载就是在每个node节点上都启动一个负载均衡，上游就是三个master，负载方式有很多 ipvs envoy nginx等，我们最终使用内核ipvs。

使用ipvs可以在join之前先把ipvs规则建立好，再去join就可以join进去了，然后对规则进行守护即可。一旦apiserver不可访问了，会自动清理掉所有node上对应的ipvs规则， master恢复正常时添加回来。

有个细节是所有对apiserver进行访问都是通过域名，因为master上连接自己就行，node需要通过虚拟ip链接多个master，这个每个节点的kubelet与kube-proxy访问apiserver的地址是不一样的，而kubeadm又只能在配置文件中指定一个地址，所以使用一个域名但是每个节点解析不同。

在 worker 上起了一个lvscare的static pod去守护这个 ipvs, 一旦apiserver不可访问了，会自动清理掉所有node上对应的ipvs规则， master恢复正常时添加回来。所以在node上加了三个东西，可以直观的看到：

```
cat /etc/kubernetes/manifests   # 这下面增加了lvscare的static pod
ipvsadm -Ln                     # 可以看到创建的ipvs规则
cat /etc/hosts                  # 增加了虚拟IP的地址解析
```

在/etc/kubernetes/manifests/下面增加了lvscare的static pod

```
root@node04:~# cat /etc/kubernetes/manifests/kube-sealyun-lvscare.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-sealyun-lvscare
  namespace: kube-system
spec:
  containers:
  - args:
    - care
    - --vs
    - 10.103.97.2:6443
    - --health-path
    - /healthz
    - --health-schem
    - https
    - --rs
    - 192.168.72.50:6443
    - --rs
    - 192.168.72.51:6443
    - --rs
    - 192.168.72.52:6443
    command:
    - /usr/bin/lvscare
    env:
    - name: LVSCARE_NODE_IP
      value: 192.168.72.53
    image: ghcr.io/labring/lvscare:v1.1.3-beta.7
    imagePullPolicy: IfNotPresent
    name: kube-sealyun-lvscare
    resources: {}
    securityContext:
      privileged: true
    volumeMounts:
    - mountPath: /lib/modules
      name: lib-modules
      readOnly: true
  hostNetwork: true
  volumes:
  - hostPath:
      path: /lib/modules
      type: ""
    name: lib-modules
status: {}
```

查看kube-system多创建出的负载均衡pod

```
# kubectl -n kube-system get pods -o wide | grep lvscare
kube-sealyun-lvscare-izj6ccgq3cou36gpcrkcyuz      1/1     Running   0          9m3s    192.168.0.174    izj6ccgq3cou36gpcrkcyuz   <none>           <none>
```

查看ipvs规则

```
yum install -y ipvsadm
ipvsadm -Ln
```

master示例

```
[root@iZj6ccgq3cou36gpcrkcyuZ ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.0.171:6443           Masq    1      1          0         
  -> 192.168.0.172:6443           Masq    1      1          0         
  -> 192.168.0.173:6443           Masq    1      0          0         
TCP  10.96.0.10:53 rr
  -> 100.69.190.1:53              Masq    1      0          0         
  -> 100.84.118.130:53            Masq    1      0          0         
TCP  10.96.0.10:9153 rr
  -> 100.69.190.1:9153            Masq    1      0          0         
  -> 100.84.118.130:9153          Masq    1      0          0         
TCP  10.103.97.2:6443 rr
  -> 192.168.0.171:6443           Masq    1      1          0         
  -> 192.168.0.172:6443           Masq    1      1          0         
  -> 192.168.0.173:6443           Masq    1      0          0         
UDP  10.96.0.10:53 rr
  -> 100.69.190.1:53              Masq    1      0          0         
  -> 100.84.118.130:53            Masq    1      0          0
```

node示例

```
[root@iZj6ccgq3cou36gpcrkcytZ ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.0.171:6443           Masq    1      2          0         
  -> 192.168.0.172:6443           Masq    1      2          0         
  -> 192.168.0.173:6443           Masq    1      0          0         
TCP  10.96.0.10:53 rr
  -> 100.69.190.1:53              Masq    1      0          0         
  -> 100.84.118.130:53            Masq    1      0          0         
TCP  10.96.0.10:9153 rr
  -> 100.69.190.1:9153            Masq    1      0          0         
  -> 100.84.118.130:9153          Masq    1      0          0         
UDP  10.96.0.10:53 rr
  -> 100.69.190.1:53              Masq    1      0          0         
  -> 100.84.118.130:53            Masq    1      0          0
```

查看/etc/hosts虚拟IP地址解析

每个master节点

```
# cat /etc/hosts
192.168.0.171   iZj6ccgq3cou36gpcrkcytZ iZj6ccgq3cou36gpcrkcytZ

192.168.0.171 apiserver.cluster.local
```

每个node节点

```
# cat /etc/hosts
192.168.0.172   iZj6ccgq3cou36gpcrkcyuZ iZj6ccgq3cou36gpcrkcyuZ

10.103.97.2 apiserver.cluster.local
```