Kubernetes 使用有好几年了，但在今年 5 月才完成 CKA 的考试。虽说用了几年，还是提前刷了部分题熟悉下。

绝大部分题都是有在 minikube 的环境上操作过，只有部分比如升级集群受限于环境问题没有实地操作。

## 写在最前

1.  保存常用文档进书签，如果有 Alfred 启用浏览器书签 workflow。效果见下图
2.  kubectl 自动补全 `echo "source <(kubectl completion bash)" >> ~/.bashrc; source ~/.bashrc`
3.  每道题开始前要切换 context 和 namespace，直接复制题目里的命令即可
4.  必要的 alias
5.  善用 `--dry-run=client -o yaml` 避免手动敲太多
6.  善用 `kubectl explain [resource[.field]]`
7.  看懂题目最重要，输出正确的结果更重要（重要的事讲三遍）
8.  看懂题目最重要，输出正确的结果更重要（重要的事讲三遍）
9.  看懂题目最重要，输出正确的结果更重要（重要的事讲三遍）

书签地址：[K8s-CKA-CAKD-Bookmarks.html](https://gist.github.com/addozhang/3ca950ce9b38930abfe7c5fb067e74de)

![alfred-bookmarks-workflow](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/07/01/20210630162758.png)

## 安全：RBAC

> 在默认命名空间中创建一个名为 dev-sa 的服务帐户，dev-sa 可以在 dev 命名空间中创建以下组件： `Deployment`、`StatefulSet`、`DaemonSet`

### 知识点

-   role
-   sa
-   rolebinding
-   auth can-i

参考文档： [https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities)

### 解题思路

```
$ kubectl create sa dev-sa
$ kubectl create role dev-role --verb=create --resource=deployment,statefulset,daemonset
#检查
$ kubectl describe role dev-role
Name:         dev-role
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources          Non-Resource URLs  Resource Names  Verbs
  ---------          -----------------  --------------  -----
  daemonsets.apps    []                 []              [create]
  deployments.apps   []                 []              [create]
  statefulsets.apps  []                 []              [create]
  
$ kubectl create rolebinding dev --serviceaccount default:dev-sa --role dev-role

#检查
$ kubectl auth can-i create deployment --as system:serviceaccount:default:dev-sa
yes
$ kubectl auth can-i create statefulset --as system:serviceaccount:default:dev-sa
yes
$ kubectl auth can-i create daemonset --as system:serviceaccount:default:dev-sa
yes
$ kubectl auth can-i create pod --as system:serviceaccount:default:dev-sa
no
```

## 多容器 Pod

> 创建一个pod名称日志，容器名称 `log-pro` 使用image `busybox`，在 `/log/data/output.log` 输出重要信息。然后另一个容器名称 `log-cus` 使用 image `busybox`，在 `/log/data/output.log` 加载 `output.log` 并打印它。 请注意，此日志文件只能在 pod 内共享。

### 知识点

-   pod
-   volume: emptyDir

参考文档： [https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

### 解题思路

```
kubectl run log --image busybox --dry-run=client -o yaml > log.yaml
```

修改 log.yaml

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: log
  name: log
spec:
  containers:
  - command:
    - sh
    - -c
    - echo important information > /log/data/output.log; sleep 1d
    image: busybox
    name: log-pro
    resources: {}
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /log/data
      name: log
  - command:
    - sh
    - -c
    - tail -f /log/data/output.log
    image: busybox
    name: log-cus
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /log/data
      name: log
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: log
    emptyDir: {}
status: {}
```

执行创建 `kubectl apply -f log.yaml`

检查

```
$ kubectl logs log -c log-cus
important information
```

## 安全：网络策略 NetworkPolicy

> 只有命名空间 `mysql` 的 pod 只能被另一个命名空间 `internal` 的 pod 通过 8080 端口进行访问

### 知识点

-   NetworkPolicy
-   Ingress

参考文档： [https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource](https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource)

### 解题思路

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cka-network
  namespace: target #目的命名空间
spec:
  podSelector: {}
  policyTypes:
  - Ingress #策略影响入栈流量
  ingress:
  - from: #允许流量的来源
    - namespaceSelector:
        matchLabels:
          ns: source #源命名空间的 label
    ports:
    - protocol: TCP
      port: 8080 #允许访问的端口

```

## 节点状态及污点

> 统计这个集群中没有污染的就绪节点，并输出到文件 `/root/cka/readyNode.txt`。

### 知识点

-   Node
-   Taint（污点）

参考文档：https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

### 解题思路

```
# Ready 状态的数量
$ kubectl get node | grep -w Ready | wc -l
# 查看含有 Taint 的数量，需要排除掉这些
$ kubectl describe node | grep Taints | grep -i NoSchedule | wc -l
```

## 资源

> 将占用CPU资源最多的pod名称输出到文件 `/root/cka/name.txt`

### 知识点

-   kubectl top 命令
-   metrics

### 解题思路

如果是 minikube 环境，报错 `error: Metrics API not available`，可以执行 `minikube addons enable metrics-server` 命令开启 metrics server。

通过 `kubectl top` 命令找到 cpu 最高的 pod，将其名字写入 `/root/cka/name.txt`。

```
$ kubectl top pod
# 或者
$ kubectl top pod | sort -k 2 -n
```

## 网络：DNS

> 有 pod 名称 `pod-nginx`，创建服务名称 `service-nginx`，使用 `nodePort` 暴露pod。 然后创建一个 pod 使用 image `busybox` 来 `nslookup` pod `pod-nginx` 和 service `service-nginx`。

### 知识点

-   service with nodePort
-   kubectl expose
-   kubectl run

参考文档：https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

### 解题思路

使用 `kubectl expose` 创建 service。

```
# 创建 service
kubectl expose pod pod-nginx --name service-nginx --type NodePort --target-port 80
# 创建 pod
kubectl run busybox --image busybox:latest --command sleep 1h
```

获取 pod 的 ip 地址，pod 的 dns lookup 需要用用到 ip。

```
$ kubectl get po -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
busybox     1/1     Running   0          2m17s   172.17.0.5   cka    <none>           <none>
pod-nginx   1/1     Running   0          59m     172.17.0.4   cka    <none>           <none>
```

执行 nslookup

```
$ kubectl exec busybox -it -- nslookup 172.17.0.4
4.0.17.172.in-addr.arpaname = 172-17-0-4.service-nginx.default.svc.cluster.local.

$ kubectl exec busybox -it -- nslookup service-nginx
Server:10.96.0.10
Address:10.96.0.10#53

Name:service-nginx.default.svc.cluster.local
Address: 10.110.253.70
```

## 工作负载：扩容

> 将命名空间 `dev` 中的 Deployment `scale-deploy` 缩放到三个 pod 并记录下来。

参考文档：https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

### 知识点

-   deployment scale up
-   kubectl scale

### 解题思路

`kubectl scale` 的使用，需要参数 `--record` 进行记录（将操作命令记录到 deployment 的 `kubernetes.io/change-cause` annotation 中）。

```
$ kubectl scale deployment scale-deploy --replicas 3 --record
```

## 集群备份及恢复

> 备份 etcd 并将其保存在主节点上的 `/root/cka/etcd-backup.db`。

> 最后恢复备份。

### 知识点

-   etcd 的备份及恢复

参考文档：https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

### 解题思路

Kubernetes 的所有数据都记录在 etcd 中，对 etcd 进行备份就是对集群进行备份。

连接 etcd 需要证书，证书可以从 apiserver 获取，因为 apiserver 需要连接 etcd。新版本的 apiserver 都是以 static pod 的方式运行，证书是通过 volume 挂载到 pod 中的。

比如 minikube 环境，证书是从 node 节点的 `/var/lib/minikube/certs` 挂载进去的。

要先 ssh 到 master 节点上。命令的执行非常快，如果长时间没结束，那就说名有问题了。

```
#备份
$ ETCDCTL_API=3 etcdctl snapshot save /root/cka/etcd-backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/var/lib/minikube/certs/etcd/ca.crt \
--cert=/var/lib/minikube/certs/apiserver-etcd-client.crt \
--key=/var/lib/minikube/certs/apiserver-etcd-client.key
```

由于只说了 restore，所以就执行 restore 的命令，默认会恢复到当前目录的 `default.etcd` 下。

```
#恢复
$ ETCDCTL_API=3 etcdctl snapshot restore /root/cka/etcd-backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/var/lib/minikube/certs/etcd/ca.crt \
--cert=/var/lib/minikube/certs/apiserver-etcd-client.crt \
--key=/var/lib/minikube/certs/apiserver-etcd-client.key
```

## 集群节点升级

> 将master节点版本从 1.20.0 升级到 1.21.0，确保 master 节点上的 pod 重新调度到其他节点，升级完成后，使 master 节点可用。

### 知识点

-   drain
-   cordon

参考文档：https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrading-control-plane-nodes

### 解题思路

受限于环境，没有实地操作。

```
# 将节点设置为不可调度
$ kubectl cordon master
# 驱逐 master 节点上的 pod
$ kubectl drain master --ignore-daemonsets

# 进行升级
$ apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.21.0-00 kubectl=1.21.0-00 && \
apt-mark hold kubelet kubectl
# 重新启动kubelet
$ systemctl daemon-reload
$ systemctl restart kubelet

# 将节点设置为可调度
$ kubectl uncordon master
```

## 集群：节点故障排查

> 现在 node01 还没有准备好，请找出根本原因并使其准备好，然后创建一个确保它在 node01 上运行的 pod。

### 知识点

-   节点故障排查

### 解题思路

这种问题大概率问题出在 kubelet 上

```
ssh node01
systemctl status kubelet
systemctl restart kubelet
# 再检查node状态
```

插播一个故障，本地安装 2 个节点的 minikube 集群时，第二个节点持续 `NotReady`。使用 `systemctl status kubelet` 看到 `unable to update cni config: no networks found in /etc/cni/net.mk`。

检查该目录确实没有文件，从 master 节点复制到该节点后重启 kubelet 解决。

## 存储：持久化卷

> 集群中有一个持久卷名称 `dev-pv`，创建一个持久卷声明名称 `dev-pvc`，确保这个持久卷声明会绑定持久卷，然后创建一个 pod 名称 `test-pvc`，将这个 pvc 挂载到 path `/tmp/data`，使用 nginx 镜像。

### 知识点

-   PersistentVolume
-   PersistentVolumeClaim
-   Mount Volume

参考文档：https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume

### 解题思路

创建 pvc 前先获取 pv的信息

```
$ kubectl get pv dev-pv -o yaml
```

创建 pv

```
$ cat > pvc.yaml <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dev-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

$ kubectl apply -f pvc.yaml
```

创建 pod 的 manifest，记得使用 `kubectl run --dry-run=client -o yaml`

```
kubectl run test-pvc --image nginx --dry-run=client -o yaml > test-pvc.yaml
```

修改之后得到最终的 pod yaml

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test-pvc
  name: test-pvc
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: dev-pvc
  containers:
  - image: nginx
    name: test-pvc
    resources: {}
    volumeMounts:
      - name: data
        mountPath: /tmp/data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

理论上只要 pod 能运行，就说明成功。也可以进一步确认挂载是否成功，在 pod 的 `/tmp/data` 中 touch 个文件，然后到节点的目录中查看是有该文件。

## 工作负载：多容器的 Deployment

> 创建一个名为 `deploy-important` 的 Deployment，标签为 `id=very-important`（pod 也应该有这个标签）和命名空间 dev 中的 3 个副本。 它应该包含两个容器，第一个名为 `container1` 并带有镜像，第二个名为 container2 的图像为 `kubernetes/pause`。

> 在一个工作节点上应该只运行该部署的一个 Pod。 我们有两个工作节点：`cluster1-worker1` 和 `cluster1-worker2`。 因为 Deployment 有三个副本，所以结果应该是在两个节点上都有一个 Pod 正在运行。 不会调度第三个 Pod，除非添加新的工作节点。

### 知识点

-   deployment
-   pod label
-   replicas
-   multi container pod
-   pod anti affinity

官方文档参考：https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#never-co-located-in-the-same-node

### 解题思路

先创建模板

```
$ kubectl create deployment deploy-important --image nginx --replicas 3 --dry-run=client -o yaml > deploy-important.yaml
```

修改后的 yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploy-important
    id: very-important
  name: deploy-important
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-important
      id: very-important
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploy-important
        id: very-important
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: id
                operator: In
                values:
                - very-important
            topologyKey: kubernetes.io/hostname
      containers:
      - image: nginx
        name: container1
        resources: {}
      - image: kubernetes/pause
        name: container2
status: {}
```

minikube 上测试只能调度一个 pod，符合预期

```
$ kgpo
NAME                                READY   STATUS    RESTARTS   AGE
deploy-important-659d54fc47-6cp8r   0/2     Pending   0          3h10m
deploy-important-659d54fc47-92z4d   2/2     Running   0          3h10m
deploy-important-659d54fc47-c6llc   0/2     Pending   0          3h10m
```

## 存储：Secret的使用

> 在 `secret` 命名空间下，使用镜像 `busybox:1.31.1` 创建一个名为 `secret-pod` 的 pod，并保证 pod 运行一段时间

> 有个名为 `sercret1.yaml` 的 Secret 文件，在 `secret` 命名空间下创建 Secret，并以只读的方式挂在到 Pod 的 `/tmp/secret1` 目录 创建一个新的 Secret `secret2` 包含 `user=user1` 和 `pass=1234`，分别以缓解变量 `APP_USER` 和 `APP_PASS` 输入到 Pod 中

### 知识点

-   secret
-   toleration
-   taints

参考文档： [https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets) [https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

### 解题思路

创建 namespace

```
$ kubectl create ns secret
```

创建 pod 模板

```
$ kubectl run secret-pod --image busybox:1.31.1 --dry-run=client -o yaml --command -- sleep 1d > secret-pod.yaml
```

修改 secret1.yaml，使用 secret namespace

```
apiVersion: v1
data:
  halt: IyEvYmluL2Jhc2g=
kind: Secret
metadata:
  creationTimestamp: "2021-05-15T07:48:02Z"
  name: secret1
  namespace: secret
type: Opaque
```

创建 secret2

```
$ kubectl create secret generic secret2 --from-literal user=user1 --from-literal pass=1234
```

修改模板

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
spec:
  containers:
  - command:
    - sleep
    - 1d
    image: busybox:1.31.1
    name: secret-pod
    env:
    - name: APP_USER
      valueFrom:
        secretKeyRef:
          name: secret2
          key: user
    - name: APP_PASS
      valueFrom:
        secretKeyRef:
          name: secret2
          key: pass
    resources: {}
    volumeMounts:
    - mountPath: /tmp/secret1
      name: sec
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: sec
    secret:
      secretName: secret1
status: {}
```

检查结果：

```
$ kubectl exec secret-pod -- cat /tmp/secret1/halt
#!/bin/bash

$ kubectl exec secret-pod -- env | grep 'APP_'
APP_USER=user1
APP_PASS=1234
```

## 工作负载：静态 Pod

> 在 `cluster3-master1` 上的 `default` 命名空间中创建一个名为 `my-static-pod` 的静态 Pod。 使用镜像 `nginx:1.16-alpine` 并分配 10m CPU 和 20Mi 内存的资源。

> Then create a NodePort Service named static-pod-service which exposes that static Pod on port 80 and check if it has Endpoints and if its reachable through the cluster3-master1 internal IP address. You can connect to the internal node IPs from your main terminal. 然后创建一个名为 `static-pod-service` 的 NodePort Service，该服务在端口 80 上公开该静态 Pod，并检查它是否具有端点以及是否可以通过 `cluster3-master1` 内部 IP 地址访问它。

### 知识点

-   static pod
-   resource
-   nodeport service
-   endpoints

参考文档：

-   [https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
-   [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

### 解题思路

创建pod模板

```
$ kubectl run my-static-pod --image nginx:1.16-alpine --dry-run=client -o yaml > static-pod.yaml
```

修改模板，增加资源配置

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-static-pod
  name: my-static-pod
spec:
  containers:
  - image: nginx:1.16-alpine
    name: my-static-pod
    resources:
      requests:
        cpu: "10m"
        memory: "20Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

ssh 到主机，找到 kubelet 配置文件的位置 `ps -ef | grep kubelet`

查看配置文件（minikube：/var/lib/kubelet/config.yaml）中 `staticPodPath` 配置的就是静态 pod 的 manifest 的位置（minikube：/etc/kubernetes/manifests）

将 `static-pod.yaml` 放到正确的文件夹中，然后重启 kubelet

```
$ systemctl restart kubelet
```

检查pod是否正确运行

创建 node port

```
$ kubectl expose pod my-static-pod --name static-pod-service --type NodePort --port 80
```

检查是否创建成功

```
$ kubectl get svc
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
static-pod-service   NodePort    10.97.248.99   <none>        80:31938/TCP   68s
```

获取 node 的 ip

```
$ kubectl get node -o wide
NAME   STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
cka    Ready    master   10h   v1.18.8   192.168.64.3   <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.4
```

在 minikube 的环境下可直接通过 `minikube ip` 获取

测试

```
$ http 192.168.64.3:31938 --headers
HTTP/1.1 200 OK
Accept-Ranges: bytes
Connection: keep-alive
Content-Length: 612
Content-Type: text/html
Date: Sat, 15 May 2021 08:35:11 GMT
ETag: "5d52db33-264"
Last-Modified: Tue, 13 Aug 2019 15:45:55 GMT
Server: nginx/1.16.1
```

## 调度：污点和容忍度

> 在命名空间 `default` 中创建图像 `httpd:2.4.41-alpine` 的单个 Pod。Pod 应命名为 `pod1`，容器名为 `pod1-container`。在不给任何节点添加新标签的前提下，将该 pod 调度到主节点上。

### 知识点

-   Taint
-   Label
-   Tolerance

参考文档：https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

### 解题思路

```
#找出master节点（一般考试只有一个节点）
$ kubectl get node
#找到 master 节点的 taints，需要在 pod 的 .spec.tolerations 排除掉
$ kubectl describe node xxxx | grep -w Taints
#找到 master 节点的 labels
$ kubectl describe node xxxx | grep -w Labels -A10
```

创建pod模板

```
$ kubectl run pod1 --image httpd:2.4.41-alpine --dry-run=client -o yaml > pod1.yaml
```

修改模板： 这里假设主节点的 Taint 为 `node-role.kubernetes.io/master=:NoSchedule`

```
# minikube 集群名为 cka，主节点同名
$ kubectl describe node cka | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

最终的 pod 如下

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: httpd:2.4.41-alpine
    name: pod1
    resources: {}
  dnsPolicy: ClusterFirst
  tolerations:
  - key: node-role.kubernetes.io/master
    effect: NoSchedule
  nodeSelector:
    node-role.kubernetes.io/master: ""
  restartPolicy: Always
status: {}
```

最后检查下是否调度到主节点上：

```
$ kubectl get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP           NODE   NOMINATED NODE   READINESS GATES
pod1   1/1     Running   0          102s   10.244.0.3   cka    <none>           <none>
```

## kubectl 命令和排序

> 所有命名空间中都有各种 Pod。 将命令写入 /opt/course/5/find\_pods.sh，其中列出所有按 AGE 排序的 Pod（metadata.creationTimestamp）。

> 将第二个命令写入 /opt/course/5/find\_pods\_uid.sh，其中列出按字段 metadata.uid 排序的所有 Pod。对这两个命令都使用 kubectl 排序。

### 知识点

-   kubectl 命令的使用，主要是 `--all-namespaces` （缩写 `-A`） 和 `--sort-by`

### 解题思路

```
$ cat > /opt/course/5/find_pods.sh <<EOF
kubectl get pod -A --sort-by '.metadata.creationTimestamp'
EOF
```

```
$ cat > /opt/course/5/find_pods_uid.sh <<EOF
kubectl get pod -A --sort-by '.metadata.uid'
EOF 
```

## 存储：持久化卷和挂载

> 创建一个名为 `safari-pv` 的新 `PersistentVolume`。它应该具有 2Gi 的容量、`accessMode` `ReadWriteOnce`、`hostPath` `/Volumes/Data` 并且没有定义 `storageClassName`。

> 接下来在命名空间 `project-tiger` 中创建一个名为 `safari-pvc` 的新 `PersistentVolumeClaim`。 它应该请求 2Gi 存储，`accessMode` `ReadWriteOnce` 并且不应定义 `storageClassName`。 PVC 应该正确绑定到 PV。

> 最后在命名空间 `project-tiger` 中创建一个新的 Deployment `safari`，它将该卷挂载到 `/tmp/safari-data`。该 Deployment 的 Pod 应该是镜像 httpd:2.4.41-alpine。

### 知识点

-   pv
-   pvc
-   pod 使用 pvc
-   deployment
-   mount PVC volume

参考文档：

-   [https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)
-   [https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)
-   [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

### 解题思路

创建 pv

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/Volumes/Data"
```

创建 pvc

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: safari-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

检查是否绑定成功

```
$ kubectl get pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
safari-pvc   Bound    pvc-d4c15825-2de3-470f-8ed0-9519cacaad21   2Gi        RWO            standard       24s
```

创建 deployment 模板

```
$ kubectl create deployment safari --image httpd:2.4.41-alpine --dry-run=client -o yaml
```

最终的yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: safari
  name: safari
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: safari
    spec:
      containers:
      - image: httpd:2.4.41-alpine
        name: httpd
        resources: {}
        volumeMounts:
        - mountPath: /tmp/safari-data
          name: data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: safari-pvc
status: {}
```

## kubectl 命令和 context

> 可以通过 kubectl 上下文从主终端访问多个集群。将所有这些上下文名称写入 /opt/course/1/contexts。

> 接下来在 /opt/course/1/context\_default\_kubectl.sh 中写一个显示当前上下文的命令，该命令应该使用kubectl。

> 最后在 /opt/course/1/context\_default\_no\_kubectl.sh 中写入第二个执行相同操作的命令，但不使用 kubectl。

### 知识点

kubectl config 相关命令的使用

### 解题思路

```
kubectl config get-contexts -o name > /opt/course/1/contexts
```

```
cat > /opt/course/1/context_default_kubectl.sh <<EOF
kubectl config current-context
EOF

chmod +x /opt/course/1/context_default_kubectl.sh
```

```
cat ~/.kube/config | grep current-context | awk '{print $2}'
```

## 工作负载：缩容

> 命名空间 `project-c13` 中有两个名为 `o3db-*` 的 Pod。 C13 管理层要求将 Pod 缩减为一个副本以节省资源。 记录动作。

### 知识点

-   scale
-   deploy
-   statefulset

参考文档： [https://kubernetes.io/zh/docs/tasks/run-application/scale-stateful-set/](https://kubernetes.io/zh/docs/tasks/run-application/scale-stateful-set/) [https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)

### 解题思路

```
kubectl scale <resource> xxx --replicas=1
```

scale 命令需要确认资源类型：deployment/statefulset

## 应用就绪和探活

> 在命名空间 `default` 中执行以下操作。为 `nginx:1.16.1-alpine` 创建一个名为 `ready-if-service-ready` 的 Pod。配置一个 `LivenessProbe`，它只是运行 `true`。还要配置一个 `ReadinessProbe` 来检查 `url` `http://service-am-i-ready:80` 是否可达，可以使用 `wget -T2 -O- http://service-am-i-ready:80`。 启动 Pod 并确认它因为 `ReadinessProbe` 而没有准备好。

> 创建第二个名为 `am-i-ready` 的 Pod 镜像 `nginx:1.16.1-alpine`，标签 `id:cross-server-ready`。已经存在的服务 `service-am-i-ready` 现在应该有第二个 Pod 作为端点。

### 知识点

-   probe
-   pod

参考文档： [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### 解题思路

```
kubectl run ready-if-service-ready --image nginx:1.16.1-alpine --dry-run=client -o yaml > ready-if-service-ready.yaml

kubectl run am-i-ready --image nginx:1.16.1-alpine --labels id=cross-server-ready --dry-run=client -o yaml > am-i-ready.yaml
```

添加 probes

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ready-if-service-ready
  name: ready-if-service-ready
spec:
  containers:
  - image: nginx:1.16.1-alpine
    name: ready-if-service-ready
    resources: {}
    livenessProbe:
      exec:
        command:
          - echo
          - hi
    readinessProbe:
      exec:
        command:
          - wget
          - -T2
          - -O-
          - http://service-am-i-ready:80
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## 集群：控制平面

> 使用 `ssh cluster1-master1` ssh 进入主节点。检查 master 组件 kubelet、kube-apiserver、kube-scheduler、kube-controller-manager 和 etcd 如何在 master 节点上启动/安装。还要找出 DNS 应用的名称以及它是如何在主节点上启动/安装的。

> 将结果写入文件 /opt/course/8/master-components.txt。该文件的结构应如下所示：

> ```
> # /opt/course/8/master-components.txt
> kubelet: [TYPE]
> kube-apiserver: [TYPE]
> kube-scheduler: [TYPE]
> kube-controller-manager: [TYPE]
> etcd: [TYPE]
> dns: [TYPE] [NAME]
> ```

> Choices of \[TYPE\] are: not-installed, process, static-pod, pod

### 知识点

Kubernetes components 的安装方式

### 解题思路

当前比较的组件都是以static pod的形式运行的，而 static pod 都是由 Kubelet 管理的，所以从 kubelet 处入手。

以 minikube 为例：

```
$ ps -ef | grep -w kubelet
root      140597       1  2 May15 ?        00:36:00 /var/lib/minikube/binaries/v1.18.8/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=cka --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.64.3

$ systemctl is-active kubelet
active
#kubelet: process
```

根据前面进程中的信息，查看 `/var/lib/kubelet/config.yaml`中的内容。可以得到：

etcd: static-pod kube-apiserver: static-pod kube-controller-manager: static-pod

```
$ cat /var/lib/kubelet/config.yaml | grep -i staticpod
staticPodPath: /etc/kubernetes/manifests

ls /etc/kubernetes/manifests
etcd.yaml  kube-apiserver.yamlkube-controller-manager.yaml  kube-scheduler.yaml
```

最后上下dns，查看下pod，得知 dns: pod

```
$ kubectl get pod -A | grep dns
kube-system   coredns-66bff467f8-6k2br      1/1     Running   0          32h
```

最后将上面的结果写入到 `/opt/course/8/master-components.txt`，不能前功尽弃。

## 集群：Pod 调度

> 使用 `ssh cluster2-master1` ssh 进入主节点。暂时停止 `kube-scheduler`，这意味着可以在之后再次启动它。

> 为镜像 `httpd:2.4-alpine` 创建一个名为 `manual-schedule` 的 Pod，确认它已启动但未在任何节点上调度。

> 现在您是调度程序并拥有所有权力，在节点 `cluster2-master1` 上手动调度该 Pod。 确保它正在运行。

> 再次启动 `kube-scheduler` 并通过在镜像 `httpd:2.4-alpine` 创建第二个名为 `manual-schedule2` 的 Pod 并检查它是否在 `cluster2-worker1` 上运行来确认其运行正常。

### 知识点

-   kubernetes 组件的运行方式
-   创建 pod
-   pod 调度

### 解题思路

kube-scheduler 是以 static pod 的方式运行，因此我们需要 ssh 到节点上，将 scheduler 的 yaml 移出（记住不要删掉，还要还原回去），重启 kubelet

```
$ ps -ef | grep -w kubelet
root      140597       1  2 May15 ?        00:36:00 /var/lib/minikube/binaries/v1.18.8/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=cka --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.64.3

$ cat /var/lib/kubelet/config.yaml | grep -i staticpod
staticPodPath: /etc/kubernetes/manifests

$ ls /etc/kubernetes/manifests
etcd.yaml  kube-apiserver.yamlkube-controller-manager.yaml  kube-scheduler.yaml

$ mv /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes

$ systemctl restart kubelet
```

检查下 schedule pod 没有运行，然后尝试创建 pod，并查看 pod 处于 pending 状态，即没有 kube-scheduler 为其调度。

```
$ kubectl run manual-schedule --image httpd:2.4-alpine

$ kubectl get pod | grep manual-schedule
manual-schedule          0/1     Pending   0          16s
```

手动调度，即为 pod 指定一个 `nodeName`，我的 minikube 只有一个 node 名为 cka，修改pod：

```
$ kubectl get pod manual-schedule -o yaml > manual-schedule.yaml
```

添加 nodeName 之后

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-05-16T07:27:16Z"
  labels:
    run: manual-schedule
  name: manual-schedule
  namespace: dev
  resourceVersion: "84805"
  selfLink: /api/v1/namespaces/dev/pods/manual-schedule
  uid: c4b592f6-1e07-4911-a7fe-867d813c7a55
spec:
  containers:
  - image: httpd:2.4-alpine
    imagePullPolicy: IfNotPresent
    name: manual-schedule
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-v7f28
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: cka #node name here
  enableServiceLinks: true
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-v7f28
    secret:
      defaultMode: 420
      secretName: default-token-v7f28
status:
  phase: Pending
  qosClass: BestEffort
```

强制更新 pod（运行时只能修改部分内容）：

```
$ kubectl replace -f manual-schedule.yaml --force
pod "manual-schedule" deleted
pod/manual-schedule replaced
```

再次检查

```
$ kubectl get pod | grep manual-schedule
manual-schedule          1/1     Running   0          15s
```

恢复 kube-scheduler 的运行：

```
$ mv /etc/kubernetes/kube-scheduler.yaml /etc/kubernetes/manifests
$ systemctl restart kubelet
```

检查是否运行

```
$ kubectl get pod -A | grep kube-scheduler
kube-system   kube-scheduler-cka            1/1     Running   0          66s
```

创建第二个pod，并检查是否在运行（running）状态

```
$ kubectl run manual-schedule2 --image httpd:2.4-alpine
pod/manual-schedule2 created

kubectl get pod manual-schedule2
NAME               READY   STATUS    RESTARTS   AGE
manual-schedule2   1/1     Running   0          6s
```

## 集群：备份及恢复

> 对在 `cluster3-master1` 上运行的 etcd 进行备份，并将其保存在主节点上的 `/tmp/etcd-backup.db`。

> 然后在集群中创建一个你喜欢的 Pod。

> 最后恢复备份，确认集群仍在工作并且创建的 Pod 不再与我们在一起。

### 知识点

-   etc 的作用：存储集群的状态信息，包括 pod 信息
-   etc 的备份和恢复

参考文档： [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

### 解题思路

etcd的命令执行，记得设置API的版本 `ETCDCTL_API=3`

操作 etcd 需要 `endpoints`、`cacert`、`cert`、`key`。Kubernetes 的所有组件与 etcd 的数据交互都是通过 api-server 完成的，我只需要找到 api-server 的运行命令就行，两种方式：到 master 主机查看 api-server 的进程；或者去 api-server 的 pod 查看 `.spec.containers[].command`

```
#ssh to master
$ ps -ef | grep kube-apiserver

$ kubectl get pod -n kube-system kube-apiserver-cka -o jsonpath='{.spec.containers[].command}'
```

etcd 备份，命令直接从 Kubernetes 官方文档复制再修改

```
#ssh to master

$ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt --cert=/var/lib/minikube/certs/apiserver-etcd-client.crt --key=/var/lib/minikube/certs/apiserver-etcd-client.key \
  snapshot save /tmp/etcd-backup.db
Snapshot saved at /tmp/etcd-backup.db
```

创建 pod

```
$ kubectl run sleep1d --image busybox --command -- sleep 1d
#检查 pod 运行情况
$ kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
sleep1d   1/1     Running   0          10s
```

恢复 etcd 的备份，复制前面的命令并修改，恢复备份到 `/var/lib/etcd-backup`

```
$ ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/var/lib/minikube/certs/etcd/ca.crt --cert=/var/lib/minikube/certs/apiserver-etcd-client.crt --key=/var/lib/minikube/certs/apiserver-etcd-client.key snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup
2021-05-16 08:09:17.797061 I | mvcc: restore compact to 85347
2021-05-16 08:09:17.803208 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
```

修改 etcd 的配置， `/etc/kubernetes/manifests/etcd.yaml`

```
  volumes:
  - hostPath:
      path: /var/lib/minikube/certs/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup  #原来是/var/lib/minikube/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}
```

保存后重启kubelet

```
$ systemctl restart kubelet
```

检查pod是否存在：

```
$ kubectl get pod sleep1d
Error from server (NotFound): pods "sleep1d" not found
```

## 安全：网络策略

> 发生了一起安全事件，入侵者能够从一个被黑的后端 Pod 访问整个集群。

> 为了防止这种情况，在命名空间 `project-snake` 中创建一个名为 `np-backend` 的 `NetworkPolicy`。它应该只允许 `backend-*` Pods：

> 连接到端口 `1111` 上的 `db1-*` Pod 连接到端口 `2222` 上的 `db2-*` Pod 在策略中使用 Pod 的应用程序标签。

> 实施后，例如，端口 3333 上从 `backend-*` Pod 到 `vault-*` Pod 的连接应该不再有效。

### 知识点

-   NetworkPolicy

参考文档： [https://kubernetes.io/docs/concepts/services-networking/network-policies](https://kubernetes.io/docs/concepts/services-networking/network-policies)

### 解题思路

为 backend-\* pod 设置 egress 的 NetworkPolicy，只允许其访问 db1-\* 的 1111 端口和 db2-\* 的 2222 端口，策略中使用 app label 来进行匹配。

从 Kubernetes 官网文档中复制一段yaml配置进行修改。

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db1
    ports:
    - protocol: TCP
      port: 1111
  - to:                
    - podSelector:
        matchLabels:
          app: db2          
    ports:
    - protocol: TCP
      port: 2222      
```

假设 backend pod 的 app label 为 backend，db1 的 为 db1，db2 的为 db2。

创建环境：

```
$ kubectl run backend --image nginx --labels app=backend
$ kubectl run db1 --image nginx --labels app=db1
$ kubectl run db2 --image nginx --labels app=db2
$ kubectl run vault --image nginx --labels app=vault

$ kubectl get pod -L app
NAME      READY   STATUS              RESTARTS   AGE   APP
backend   1/1     Running   0          13s   backend
db1       1/1     Running             0          66s   db1
db2       1/1     Running             0          71s   db2
vault     1/1     Running             0          79s   vault
```

由于我们用的 nginx 镜像，将前面的 NetworkPolicy 端口修改一下：

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db1
    ports:
    - protocol: TCP
      port: 80
  - to:                
    - podSelector:
        matchLabels:
          app: db2          
    ports:
    - protocol: TCP
      port: 80
```

检查一下：

```
$ kubectl get networkpolicy
NAME         POD-SELECTOR   AGE
np-backend   app=backend    31s
```

测试下网络：

```
#获取pod ip
$ kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
backend   1/1     Running   0          3m15s   172.17.0.7   cka    <none>           <none>
db1       1/1     Running   0          4m8s    172.17.0.6   cka    <none>           <none>
db2       1/1     Running   0          4m13s   172.17.0.3   cka    <none>           <none>
vault     1/1     Running   0          4m21s   172.17.0.4   cka    <none>           <none>

```

## 集群：kubelet 启动方式

> 节点 `cluster2-worker1` 已使用 kubeadm 和 TLS 引导添加到集群中。 找到 `cluster2-worker1` 的 “Issuer” 和 “Extended Key Usage” 值： kubelet 客户端证书，用于向外连接到 kube-apiserver 的证书。 kubelet 服务器证书，用于来自 kube-apiserver 的传入连接。 将信息写入文件 `/opt/course/23/certificate-info.txt`。

## 知识点

-   kubelet 的功能：连接 api-server；接受来自 api-server 的响应。两种情况都需要 TLS

### 解题思路

kubelet 连接 apiserver 的方式在配置文件中，先找出配置文件的保存位置。

```
# ssh 到节点上，查看 kubelet 的启动命令
$ ps -ef | grep kubelet
root        3935       1  1 12:54 ?        00:00:23 /var/lib/minikube/binaries/v1.20.0/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --cni-conf-dir=/etc/cni/net.mk --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=cka-m02 --kubeconfig=/etc/kubernetes/kubelet.conf --network-plugin=cni --node-ip=192.168.64.9
docker     13653   13616  0 13:22 pts/0    00:00:00 grep kubelet
```

```
#kubelet 连接 api server 的信息， client cert 的配置所在 /var/lib/kubelet/pki/kubelet-client-current.pem
cat /var/lib/kubelet/config.yaml 
#kubelet 的启动信息， servert cert 的配置所在 /var/lib/minikube/certs/ca.crt
cat /etc/kubernetes/kubelet.conf 
```

```
$ openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep -i issuer
        Issuer: CN = minikubeCA
$ openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep -i -A1 extended
            X509v3 Extended Key Usage:
                TLS Web Client Authentication

$ openssl x509 -noout -text -in /var/lib/minikube/certs/ca.crt | grep -i issuer
        Issuer: CN = minikubeCA
$ openssl x509 -noout -text -in /var/lib/minikube/certs/ca.crt | grep -i -A1 extended
            X509v3 Extended Key Usage:
                TLS Web Client Authentication, TLS Web Server Authentication
```

最后记得将信息写入到 `/opt/course/23/certificate-info.txt`

## 集群：证书

> 检查 kube-apiserver 服务器证书在 `cluster2-master1` 上的有效期。使用 openssl 或 cfssl 执行此操作。将到期日期写入 `/opt/course/22/expiration`。

> 同时运行正确的 kubeadm 命令以列出到期日期并确认两种方法显示相同的日期。

> 将更新 apiserver 服务器证书的正确 kubeadm 命令写入 `/opt/course/22/kubeadm-renew-certs.sh`。

### 知识点

-   api-server
-   openssl
-   kubeadm

参考文档：https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#check-certificate-expiration

### 解题思路

通过 kube-apiserver pod 的启动命令，或者 ssh 到 master 来查看命令参数，`tls-cert-file=/var/lib/minikube/certs/apiserver.crt`

```
$ openssl x509 -noout -text -in /var/lib/minikube/certs/apiserver.crt | grep -i valid -A2
        Validity
            Not Before: May 13 22:33:43 2021 GMT
            Not After : May 14 22:33:43 2022 GMT
```

将 `May 14 22:33:43 2022 GMT` 写入 `/opt/course/22/expiration`

通过 kubeadm 来检查

```
$ kubeadm certs check-expiration | grep -i apiserver
#macos 无法安装 kubeadm
#minikube 无法使用 kubeadm 检查
```

将 `kubeadm certs renew apiserver` 写入 /opt/course/22/kubeadm-renew-certs.sh

## 集群：升级节点

> 你的同事说节点 `cluster3-worker2` 运行的是较旧的 Kubernetes 版本，甚至不属于集群的一部分。将 kubectl 和 kubeadm 更新为在 `cluster3-master1` 上运行的确切版本。然后将此节点添加到集群中，您可以为此使用kubeadm。

### 知识点

-   kubeadm 升级集群

参考文档：

### 解题思路

检查node

检查当前组件版本

```
$ ssh cluster3-worker2

$ kubeadm version
$ kubectl version --short
Client Version: vx.xx.x
Server Version: vx.xx.x

$ kubelet --version
```

```
#使用命令并升级各个组件，并重启 kubelet
#如果启动失败，一般是需要token连接到api-server，需要ssh到master上运行 kubeadm create token --print-join-command
#再ssh到 node上，执行打印的命令，重启kubelet并检查装填
#最后检查node是否成功加入集群
```

## Docker 命令

> 在命名空间 `project-tiger` 中创建一个名为 `Tigers-reunite` 的 Pod 镜像 `httpd:2.4.41-alpine`，标签为 `pod=container` 和 `container=pod`。找出 Pod 被安排在哪个节点上。ssh 进入该节点并找到属于该 Pod 的 docker 容器。

> 将容器的 docker ID 和这些正在运行的进程/命令写入 `/opt/course/17/pod-container.txt`。

> 最后，使用 docker 命令将主 Docker 容器（来自 yaml 中指定的那个）的日志写入 `/opt/course/17/pod-container.log`。

### 知识点

-   docker 命令：ps、logs、inspect

### 解题思路

创建 pod

```
$ kubectl run tigers-reunite --image httpd:2.4.41-alpine --labels pod=container,container=pod
```

检查 pod 的信息

```
$ kubectl get pods --show-labels
NAME             READY   STATUS    RESTARTS   AGE   LABELS
tigers-reunite   1/1     Running   0          34s   container=pod,pod=container
```

获取pod所在的节点

```
$ kubectl get pod tigers-reunite -o jsonpath='{.spec.nodeName}'
cka
```

ssh到节点上

```
docker ps | grep tigers-reunite
e6ff69b437bc   54b0995a6305           "httpd-foreground"       About a minute ago   Up About a minute             k8s_tigers-reunite_tigers-reunite_dev_53391212-911d-4275-a19d-e8f8b0f85a98_0
06d3ca65eb08   k8s.gcr.io/pause:3.2   "/pause"                 About a minute ago   Up About a minute             k8s_POD_tigers-reunite_dev_53391212-911d-4275-a19d-e8f8b0f85a98_0

#使用docker inspect 或者 进入容器直接查看进程
$ docker inspect e6ff69b437bc | grep -i 'cmd\|entrypoint' -A1
            "Cmd": [
                "httpd-foreground"
--
            "Entrypoint": null,
            "OnBuild": null,
                
$ docker inspect 06d3ca65eb08 | grep -i 'cmd\|entrypoint' -A1
            "Cmd": null,
            "Image": "k8s.gcr.io/pause:3.2",
--
            "Entrypoint": [
                "/pause"
```

结果写入文件

```
e6ff69b437bc httpd-foreground
06d3ca65eb08 pause
```

写日志到文件

```
docker logs e6ff69b437bc > /opt/course/17/pod-container.log
```