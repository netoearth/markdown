## 1. k8s基本命令记录
### 1.1. 任何命令如果不加-n xxx/--namespace xxx都会默认在default
### 1.2. 查看namespaces：
```bash
# kubectl get ns
NAME              STATUS   AGE
default           Active   4h11m
kube-node-lease   Active   4h11m
kube-public       Active   4h11m
kube-system       Active   4h11m
```

### 1.3. 升级deployment

升级deployment有两种方式，一种是直接修改，一种是修改yaml文件：

#### 1.3.1. 第一种
```bash
# kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
sa-frontend   2/2     2            2           3h34m
# kubectl edit deployment sa-frontend
Edit cancelled, no changes made.
# kubectl edit deployment/sa-frontend
Edit cancelled, no changes made.
# kubectl edit deployment.extensions/sa-frontend
Edit cancelled, no changes made.
```
#### 1.3.2. 第二种
```bash
# cat sa-frontend-deployment-green.yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sa-frontend
spec:
  replicas: 2
  minReadySeconds: 15
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: sa-frontend
    spec:
      containers:
        - image: rinormaloku/sentiment-analysis-frontend:green
          imagePullPolicy: Always
          name: sa-frontend
          ports:
            - containerPort: 80# 
# kubectl apply -f sa-frontend-deployment-green.yaml --record //升级要用apply，用create会因为重名而报错
deployment.extensions/sa-frontend configured
# 
# kubectl rollout status deployment sa-frontend //查看升级过程
Waiting for deployment "sa-frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "sa-frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "sa-frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "sa-frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "sa-frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "sa-frontend" rollout to finish: 1 of 2 updated replicas are available...
deployment "sa-frontend" successfully rolled out
# 

```

### 1.4. 回退上一个版本

```bash
# kubectl rollout history deployment sa-frontend //查看deployment版本
deployment.extensions/sa-frontend 
REVISION  CHANGE-CAUSE
1         <none>         <------1是版本
2         kubectl apply --filename=sa-frontend-deployment-green.yaml --record=true

# kubectl rollout undo deployment sa-frontend --to-revision=1 //回退到上一个版本
deployment.extensions/sa-frontend rolled back
```

### 1.5. 查看所有pod
```bash
# kubectl get pods -o wide --all-namespaces
NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
sa-frontend-66bd785c7f-mhjtc   1/1     Running   0          21m   10.100.0.13   localhost   <none>           <none>
sa-frontend-66bd785c7f-qwhzf   1/1     Running   0          21m   10.100.0.14   localhost   <none>           <none>
sa-logic-798b555684-c6r8b      1/1     Running   0          15m   10.100.0.15   localhost   <none>           <none>
sa-logic-798b555684-wb26v      1/1     Running   0          15m   10.100.0.16   localhost   <none>           <none>
```

### 1.6. 查看服务

```bash
# kubectl get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP        4h37m
sa-frontend-nodeport   NodePort    10.111.36.213    <none>        80:31068/TCP   78m
sa-logic               ClusterIP   10.110.167.111   <none>        80/TCP         3m27s
```

### 1.7. 查看服务详细信息

```bash
# kubectl describe service sa-frontend-nodeport
Name:                     sa-frontend-nodeport
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=sa-frontend
Type:                     NodePort //Service 通过 Cluster 节点的静态端口对外提供服务。Cluster 外部可以通过 <NodeIP>:<NodePort> 访问 Service
IP:                       10.111.36.213
Port:                     <unset>  80/TCP //ClusterIP 上监听的端口，只能Cluster 内部访问
TargetPort:               80/TCP //pod监听的端口，也就是pod里会有个程序监听此端口，用于提供某种服务
NodePort:                 <unset>  31068/TCP //节点上监听的端口，也就是对外提供服务的端口。默认随机分配，可指定
Endpoints:                10.100.0.13:80,10.100.0.14:80 // 也可以通过 Cluster 内部的 IP 访问
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
也就是访问本机有效IP:NodePort或者访问pod_ip:Port都会转发到TargetPort。
![](_v_images/20200520170333259_1765.png)

### 1.8. 查看pod详情

当pod没起来的时候，查看详情：

```bash
[root@sec-infoanalysis2-test ops]# kubectl describe pod billowing-kudu-hello-helm-5c6689fcf9-qjhbp
Name:           billowing-kudu-hello-helm-5c6689fcf9-qjhbp
Namespace:      default
Priority:       0
Node:           sec-k8s-node1/192.168.47.144
Start Time:     Mon, 21 Oct 2019 15:55:16 +0800
Labels:         app.kubernetes.io/instance=billowing-kudu
                app.kubernetes.io/name=hello-helm
                pod-template-hash=5c6689fcf9
Annotations:    <none>
Status:         Running
IP:             10.244.2.27
Controlled By:  ReplicaSet/billowing-kudu-hello-helm-5c6689fcf9
Containers:
  hello-helm:
    Container ID:   docker://68fdb85c3f78da1c034b17b8f1fd1203c7883b785cb98bc5165f37d157544af2
    Image:          nginx:stable
    Image ID:       docker-pullable://nginx@sha256:dd87b3bba63ff0cf6545fca46c9cdecb3e2ab09cacdbc1a08c1000ab97c76b75
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 21 Oct 2019 16:03:04 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-m4s9k (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-m4s9k:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-m4s9k
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                    Message
  ----    ------     ----   ----                    -------
  Normal  Scheduled  16m    default-scheduler       Successfully assigned default/billowing-kudu-hello-helm-5c6689fcf9-qjhbp to sec-k8s-node1
  Normal  Pulling    16m    kubelet, sec-k8s-node1  Pulling image "nginx:stable"
  Normal  Pulled     8m42s  kubelet, sec-k8s-node1  Successfully pulled image "nginx:stable"
  Normal  Created    8m41s  kubelet, sec-k8s-node1  Created container hello-helm
  Normal  Started    8m41s  kubelet, sec-k8s-node1  Started container hello-helm

```

### 1.9. 设置node的role标签

```
[root@infoanalysis2-test ops]# kubectl get nodes
NAME                     STATUS   ROLES    AGE   VERSION
infoanalysis1-test   Ready    node     78d   v1.15.1
infoanalysis2-test   Ready    master   78d   v1.15.1
k8s-node1            Ready    <none>   20s   v1.15.1
[root@infoanalysis2-test ops]# kubectl label nodes k8s-node1 node-role.kubernetes.io/node=
node/k8s-node1 labeled
[root@infoanalysis2-test ops]# kubectl get nodes
NAME                     STATUS   ROLES    AGE   VERSION
infoanalysis1-test   Ready    node     78d   v1.15.1
infoanalysis2-test   Ready    master   78d   v1.15.1
k8s-node1            Ready    node     2m    v1.15.1

```

### 查看错误
```bash
# journalctl -f -u kubelet
```