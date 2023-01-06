本章目录：

0x01 安装部署

-   (2) 集群搭建Jenkins Master 节点
    

-   资源清单
    
-   创建查看
    
-   服务访问
    
-   NFS (Network File System) 环境
    
-   NFS Client Provisioner 环境
    
-   2.1) 基础环境
    
-   2.2) 搭建流程
    

-   (3) 集群动态创建 Agent 节点 - Slave 节点
    

-   内置 Jenkins Master 接入内部 K8s 集群
    
-   独立Jenkins Master节点接入外部K8s集群
    

0x03 补充说明

-   (1) K8s 集群中对搭建的Jenkins进行版本升级
    
-   (2) 移植其他Jenkins机器上的插件到Kubernetes安装的Jenkins中然后进行重新Jenkins(需要非常注意版本问题-)
    

0x04 入坑出坑

-   问题1.在K8s中安装Jenkins时报错从logs日志显示`Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?`错误
    
-   问题2.Jenkins调用节点执行任务时`java.lang.IllegalStateException: Agent is not connected after 100 seconds, status: Running`报错导致不断重启
    
-   问题3.基于 Kubernetes 部署 Jenkins 动态 slave 后，运行 Jenkins Job 会抛java.nio.channels.ClosedChannelException
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

**环境准备:**

```
- Kubernetes 集群 ："v1.19.6"
- NFS : 数据持久化最常用的共享存储
- Jenkins 镜像 : jenkins/jenkins:2.277-alpine
```

描述：它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。我们可以利用NFS共享Jenkins运行的配置文件、Maven的仓库依赖文件等。

```
# NFS 客户端安装: Master&node节点都执行
~$ ansible dtk8s -m shell -a "sudo apt-get install nfs-common rpcbind -y"

# 查看Windows 2008 R2搭建的NFS服务端
showmount -e 192.168.1.31
Export list for 192.168.1.31:
/nask8sapp  (everyone)

# 安全配置 (设置指定的作用域)
基于 Unix 软件的 Portmap 的入站规则，允许 Portmap 服务的流量。[TCP 111] - 设置其作用域
NFS 服务器(NFS-UDP-In) NFS 服务器允许 NFS 通信的入站规则。[UDP 2049] - 设置其作用域
NFS 服务器(NFS-TCP-In) NFS 服务器允许 NFS 通信的入站规则。[TCP 2049] - 设置其作用域

# 临时与永久挂载
ansible dtk8s -m shell -a "sudo mkdir /nfsdisk-31"  # 目录创建
sudo mount.nfs 192.168.12.31:/nask8sapp /nfsdisk-31/
ls /nfsdisk-31/
# /nfsdisk-31/1.txt

# 通过 /etc/fstab 挂载
ansible dtk8s -m shell -a 'echo "192.168.12.31:/nask8sapp /nfsdisk-31 nfs defaults 0 0"|sudo tee -a /etc/fstab'
# weiyigeek-* | CHANGED | rc=0 >>
# 192.168.12.31:/nask8sapp /nfsdisk-31 nfs defaults 0 0
# ...
ansible dtk8s -m shell -a 'sudo mount -a'
ansible dtk8s -m shell -a 'mount | grep "nfs"'
# weiyigeek-* | CHANGED | rc=0 >>
# 192.168.12.31:/nask8sapp on /nfsdisk-31 type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.12.31,mountvers=3,mountport=1048,mountproto=udp,local_lock=none,addr=192.168.12.31)

# 至此下一步我们需要在k8s集群中进行配置 NFS client provisioner
```

注意: 安装配置是一个Kubernetes的简易NFS的外部provisioner，本身不提供NFS，需要现有 的NFS服务器提供存储。  
nfs-client-provisioner 构建的yaml文件:

```
cat > nfs-client-provisioner.yaml  <<'EOF'
# NFS 驱动编排的资源清单
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:          # 策略
    type: Recreate   # 再生(循环使用)
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner   # 服务帐户名称
      containers:
      - name: nfs-client-provisioner
        image: registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner
        volumeMounts:
        - name: timezone
          mountPath: /etc/localtime                # 时区设置
        - name: nfs-client-root
          mountPath: /persistentvolumes
        env:
        - name: PROVISIONER_NAME                   # StorageClass 对象中定义的 provisioner 键需要保持一致
          value: fuseim.pri/ifs
        - name: NFS_SERVER
          value: 192.168.12.31
        - name: NFS_PATH
          value: /nask8sapp
      volumes:
      - name: timezone                            # 时区定义
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      - name: nfs-client-root                     # 存储卷
        nfs:
          server: 192.168.12.31
          path: /nask8sapp
---
# Storageclass 部署文件
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage    # 重要 StorageClass Name绑定
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"   #设置其为默认存储后端
provisioner: fuseim.pri/ifs   # 或选择另一个名称，必须与NFS 驱动编排匹配部署的 env PROVISIONER_NAME
parameters:
  archiveOnDelete: "false"    # 删除pvc后，后端存储上的pv也自动删除
EOF


# nfs-client-provisioner - rbac授权资源清单
cat > nfs-client-rbac.yaml<<'EOF'
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "update"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-client-provisioner
  namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-client-provisioner
  namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
EOF
```

执行 & 运行pod

```
~/k8s/nfs-client-provisioner$ ls
# nfs-client-provisioner.yaml  nfs-client-rbac.yaml

~/k8s/nfs-client-provisioner$ kubectl create -f .
# deployment.apps/nfs-client-provisioner created
# storageclass.storage.k8s.io/managed-nfs-storage created
# serviceaccount/nfs-client-provisioner created
# clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
# clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
# role.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
# rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created

$ kubectl get storageclasses.storage.k8s.io
# NAME                            PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
# managed-nfs-storage (default)   fuseim.pri/ifs   Delete          Immediate           false                  53s

$ kubectl get pod
# NAME                                     READY   STATUS    RESTARTS   AGE
# nfs-client-provisioner-57946d456-b4s7l   1/1     Running   0          22s
```

-   **Step 1.创建PV、PVC，为Jenkins提供数据持久化**
    

```
cat > jenkins-PVC.yaml <<'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: devops
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: devops
  annotations:   # 空间标注
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
EOF
```

-   **Step 2.创建角色授权**
    

```
cat > jenkins-role.yaml <<'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-sa
  namespace: devops
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-cr
rules:
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get","list","watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-crd
roleRef:
  kind: ClusterRole
  name: jenkins-cr
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: jenkins-sa
  namespace: devops
EOF
```

-   **Step 3.在Kubernetes中Deployment部署Jenkins以及Service资源创建**
    

```
cat > jenkins-deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops
spec:
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      terminationGracePeriodSeconds: 10
      serviceAccount: jenkins-sa
      containers:
      - name: jenkins
        image: jenkins/jenkins:2.275-alpine
        imagePullPolicy: IfNotPresent
        env:
        - name: JAVA_OPTS
          value: -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85 -Duser.timezone=Asia/Shanghai
        ports:
        - containerPort: 8080
          name: web
          protocol: TCP
        - containerPort: 50000
          name: agent
          protocol: TCP
        resources:
          limits:
            cpu: 1000m
            memory: 1Gi
          requests:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 60
          timeoutSeconds: 5
          failureThreshold: 12
        readinessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 60
          timeoutSeconds: 5
          failureThreshold: 12
        volumeMounts:
        - name: jenkinshome
          mountPath: /var/jenkins_home
      securityContext:
        fsGroup: 1000
      volumes:
      - name: jenkinshome
        persistentVolumeClaim:
          claimName: jenkins-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: devops
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  type: NodePort
  ports:
  - name: web
    port: 8080
    targetPort: web
    nodePort: 30001
  - name: agent
    port: 50000
    targetPort: agent
EOF
```

-   **Step 4.采用kubectl命令创建上面的资源清单**
    

```
# (1) PVC 持久卷创建
$ kubectl create -f jenkins-PVC.yaml
  # namespace/devops created
  # persistentvolumeclaim/jenkins-pvc created

# (2) Jenkins 在集群中角色创建绑定
kubectl create -f jenkins-role.yaml
  # serviceaccount/jenkins-sa created
  # clusterrole.rbac.authorization.k8s.io/jenkins-cr created
  # clusterrolebinding.rbac.authorization.k8s.io/jenkins-crd created

# (3) 部署 Jenkins deployment 和 SVC
kubectl create -f jenkins-deployment.yaml
  # deployment.apps/jenkins created
  # service/jenkins created
```

-   **Step 5.查看创建的PVC、POD以及SVC**
    

```
~$ kubectl get pvc,pod,svc -n devops
  # NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
  # persistentvolumeclaim/jenkins-pvc   Bound    pvc-3cd916df-91cb-470d-b9ef-e9b4f115223d   5Gi        RWX            managed-nfs-storage   23m

  # NAME                          READY   STATUS    RESTARTS   AGE
  # pod/jenkins-689775956-nph9z   1/1     Running   0          15m

  # NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                          AGE
  # service/jenkins   NodePort   10.104.214.1   <none>        8080:30001/TCP,50000:30465/TCP   15m
```

-   **Step 6.我们进入Pod容器内部进行查看**
    

```
kubectl exec -n devops -it jenkins-7db7878f8f-78tmk bash
$ ps aux   # 可看到有两个进程正在运行
# PID   USER     TIME  COMMAND
# 1 jenkins   0:39 /sbin/tini -- /usr/local/bin/jenkins.sh  # 主运行文件
# 6 jenkins  57:00 java -Duser.home=/var/jenkins_home -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85 -Duser.timezone=Asia/Shanghai -Djenkins.model.Jenkins.slaveAgentPort=50000 -jar /usr/share/jenkins/jenkins.war   # 运行 jenkins.war 的命令
```

查看容器入口执行文件: cat `/usr/local/bin/jenkins.sh`

```
#! /bin/bash -e
: "${JENKINS_WAR:="/usr/share/jenkins/jenkins.war"}"
: "${JENKINS_HOME:="/var/jenkins_home"}"
: "${COPY_REFERENCE_FILE_LOG:="${JENKINS_HOME}/copy_reference_file.log"}"
: "${REF:="/usr/share/jenkins/ref"}"
touch "${COPY_REFERENCE_FILE_LOG}" || { echo "Can not write to ${COPY_REFERENCE_FILE_LOG}. Wrong volume permissions?"; exit 1; }
echo "--- Copying files at $(date)" >> "$COPY_REFERENCE_FILE_LOG"
find "${REF}" \( -type f -o -type l \) -exec bash -c '. /usr/local/bin/jenkins-support; for arg; do copy_reference_file "$arg"; done' _ {} +

# if `docker run` first argument start with `--` the user is passing jenkins launcher arguments
if [[ $# -lt 1 ]] || [[ "$1" == "--"* ]]; then

  # read JAVA_OPTS and JENKINS_OPTS into arrays to avoid need for eval (and associated vulnerabilities)
  java_opts_array=()
  while IFS= read -r -d '' item; do
    java_opts_array+=( "$item" )
  done < <([[ $JAVA_OPTS ]] && xargs printf '%s\0' <<<"$JAVA_OPTS")

  readonly agent_port_property='jenkins.model.Jenkins.slaveAgentPort'
  if [ -n "${JENKINS_SLAVE_AGENT_PORT:-}" ] && [[ "${JAVA_OPTS:-}" != *"${agent_port_property}"* ]]; then
    java_opts_array+=( "-D${agent_port_property}=${JENKINS_SLAVE_AGENT_PORT}" )
  fi

  if [[ "$DEBUG" ]] ; then
    java_opts_array+=( \
      '-Xdebug' \
      '-Xrunjdwp:server=y,transport=dt_socket,address=5005,suspend=y' \
    )
  fi

  jenkins_opts_array=( )
  while IFS= read -r -d '' item; do
    jenkins_opts_array+=( "$item" )
  done < <([[ $JENKINS_OPTS ]] && xargs printf '%s\0' <<<"$JENKINS_OPTS")

  exec java -Duser.home="$JENKINS_HOME" "${java_opts_array[@]}" -jar ${JENKINS_WAR} "${jenkins_opts_array[@]}" "$@"
fi

# As argument is not jenkins, assume user want to run his own process, for example a `bash` shell to explore this image
exec "$@"
```

Tips : `/usr/local/bin/jenkins.sh` 以及 `usr/share/jenkins/`可用帮助我们进行Jenkins手动升级;

-   **Step 7.因为我们Service是采用NodePort类型其端口为30001，我们直接在浏览器用这个端口访问Jenkins UI, 下面是精简操作如果不理解的童鞋，需要按照第一篇文章的操作进行初始化；**
    

```
# Jenkins 初始化密码获取的两种方式;
# 方式1.认证
$ kubectl logs -n devops pod/jenkins-689775956-nph9z | grep -A 2 "following password"
Please use the following password to proceed to installation:

c45f558fa237472f9f8f954ceb3a323e

# 方式2.动态卷Jenkins持久化目录
# 容器中的目录 : /var/jenkins_home/secrets/initialAdminPassword - pvc-3cd916df-91cb-470d-b9ef-e9b4f115223d
$ cat /nfsdisk-31/devops-jenkins-pvc-pvc-3cd916df-91cb-470d-b9ef-e9b4f115223d/secrets/initialAdminPassword
c45f558fa237472f9f8f954ceb3a323e
```

![WeiyiGeek.Jenkins Init](https://i0.hdslb.com/bfs/article/dc109cdbed758bc694328768b1db72e1392abe32.png@942w_492h_progressive.webp)

-   **Step 7.安装 `Jenkins入门学习之持续化集成与部署` 文章的操作进行初始化, 当然您也可以选择自定义插件安装 -> Languages (两项插件) 先进行汉化**
    

![WeiyiGeek.Languages-Chinese](https://i0.hdslb.com/bfs/article/57010b5e7d219ae48e52df77bf545986b0c49ad7.png@942w_585h_progressive.webp)

-   **Step 8.Create First Admin User -> Instance Configuration (Jenkins URL) -> Save -> Restart 安装成功**
    

```
Jenkins URL: http://jenkins.weiyigeek.top:30001/  # 注意需要添加解析
Jenkins URL: http://192.168.10.10:30001
```

![WeiyiGeek.](https://i0.hdslb.com/bfs/article/50b72ad37d808c1f1da444bb0618869c702f7369.png@942w_647h_progressive.webp)

-   **Step 9.插件镜像仓库设置加快拉取进度 `Dashboard -> 插件管理 -> 高级 -> 升级站点`;**
    

```
# 升级站点
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

# 用户定义的时区 -> Time Zone -> Asia/Beijing
```

-   **Step 10.最后最新进行常用插件安装，可以参考最后一章**
    

描述: 前面我们说过Jenkins的分布式架构(Master-Slave)，其中 master 主要负责负载地调度而 slave 执行构建任务，并且当Job构建完成后Pod就会自动销毁, 所以其可以很好地解决性能与资源占用问题。

**步骤说明:**

-   Step 1.所以在 Jenkins 服务安装好 Kubernetes 插件 并配置好连接 Kubernetes 的信息，就可以在 Kubernetes 集群中动态创建 Agent 节点了。其中 Jenkins Master节点可以直接安装在宿主机中，也可以部署在 Kubernetes 集群中。  
    该插件为每个要启动的 Jenkins Agent 节点创建一个 Kubernetes Pod 对象，并在构建完成后销毁 Pod 。
    

-   1.JENKINS\_URL：Jenkins Web界面URL
    
-   2.JENKINS\_SECRET：用于认证的密钥
    
-   3.JENKINS\_AGENT\_NAME：Jenkins代理的名称
    
-   4.JENKINS\_NAME：Jenkins代理的名称（不建议使用。仅在此处是为了向后兼容）
    
-   Agent 节点是使用JNLP启动的，是通过 Agent 节点镜像自动连接 Jenkins Master 节点。使用这种连接方式，需要对 Agent 节点设置一些环境变量：
    

Tips : 这些环境变量会在 Pod 创建配置中设定好，用于 Agent 节点启动时连接 Master 节点。

-   Step 2.Kubernetes 插件使用时，最先要配置的是`连接 Kubernetes 集群的连接信息`和 `Jenkins 服务 Master 节点连接地址`(其他连接信息自动生成不需要配置)。
    

-   1.Jenkins 服务使用 Kubernetes 插件连接 Kubernetes 集群，并动态创建 Agent 节点。连接 Kubernetes 集群需要配置的 Kubernetes 连接信息包括：
    
-   2.Kubernetes 集群名称
    
-   3.Kubernetes 集群Api-server 的连接地址
    
-   4.Kubernetes 集群服务证书：Kubernetes 集群节点间通信都是使用证书双向认证加密的，一般所有的证书都使用同一个 CA 证书做证书申请签发；这里的服务器证书就是这个CA 证书。因为此时 Jenkins Master 节点也就是一个 Kubernetes Agent 节点，也需要信任 CA 证书，信任CA 证书签发的其他节点上的证书。
    
-   5.Kubernetes 命名空间：应该是创建的 Agent 节点在哪个命名空间中运行。
    
-   6.凭证（Kubernetes 认证）：支持的凭证包括：用户名密码、秘密文件（kubeconfig文件）、秘密文本（基于令牌的身份验证）（OpenShift）、来自私钥的Google服务帐户（GKE身份验证）、X.509客户端证书。
    

将会具体使用配置将会在下面进行详述的配置讲解。

**配置流程:**

-   Step 1.安装kubernetes插件: 点击 Manage Jenkins -> Manage Plugins -> Available -> Kubernetes 勾选安装即可。
    

```
# 安装 kubernetes plugins
Kubernetes Client API Plugin - Kubernetes客户端API插件，供其他Jenkins插件使用 - 4.11.1
Kubernetes Credentials Plugin - Kubernetes证书的公共类 - 0.8.0
Kubernetes plugin - 这个插件集成了Jenkins与Kubernetes - 1.28.7
```

-   Step 2.在Jenkins中配置k8s信息：安装完毕后点击 Manage Jenkins —> Configure System —> (拖到最下方) Add a new cloud —> 选择 Kubernetes，然后填写 Kubernetes 和 Jenkins 配置信息。
    

```
# kubernetes 集群名称 Name:
kubernetes
# Kubernetes 地址
https://kubernetes.default.svc.cluster.local
# Kubernetes 服务证书 key 内容
ca.crt
# Kubernetes 命名空间
devops
# 添加凭据可选
# Jenkins 地址 : The URL of the Jenkins Master server.
# 格式 : 服务名.namespace.svc.cluster.local:8080
http://jenkins.devops.svc.cluster.local:8080
# 注意: Jenkins通道不需要添加slave它会自动识别(网上博客说加上之后可能导致Pod反复重启-应该在新版本中不会存在)
# Jenkins 通道
jenkins.devops.svc.cluster.local:50000
# Pod Labels
Key: app
Value: k8s
# 其它值默认即可(如需配置相应即可)
```

Tips: 设置JNLP访问协议，打开`Jenkins/Configure Global Security`找到 Agents, 设置 Port 为 指定端口50000（`对于集群搭建Jenkins-Master想接入其它VM物理机上的Agent节点`）, Agent protocols 选Inbound TCP Agent Protocol/4 (TLS encryption)保存；

-   Step 3.点击Test Connection，如果出现 `Connected to Kubernetes v1.19.6` 的提示信息证明 Jenkins 已经可以和 Kubernetes 系统正常通信了(CSDN - 滞后性太严重了，还是得看官网)  
    Tips : 注意如果这里 Connection 失败的话，很有可能是权限问题，这里就需要把我们创建的 jenkins 的 serviceAccount 对应的 secret 添加到这里的 Credentials 里面。
    

![WeiyiGeek.JenkinsConnectionK8s](https://i0.hdslb.com/bfs/article/0b04624165d317b78021794f72c1acdcc2d05b39.png@942w_699h_progressive.webp)

-   Step 1.创建`Kubernetes Namespace`与`Service Account`(可参考前面搭建)
    

```
# 在Kubenates的上创建devops命名空间，用于Jenkins使用
kubectl create namespace devops

# 在Kubernetes上为Jenkins构建创建有Cluster Admin权限的Service Account jenkins：
kubectl create clusterrolebinding jenkins --clusterrole cluster-admin --serviceaccount=devops:jenkins
```

-   Step 2.生成调度凭证即`Kubernetes的 server certificate key和Client P12 Certificate File`, 其作用是采用`P12 Certificate File`提供给`jenkins Master`去调用Kubenetes,
    

```
# ~/.kube/config 运行以下命令分别生成生成 ca.crt, client.crt, client.key
$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ********ORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://weiyigeek-lb-vip.k8s:16443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR........VRFMLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNS........BQUklWQVRFIEtFWS0tLS0tCg==


# (1) 复制 certificate-authority-data 的内容运行以下命令生成 `ca.crt` - ca证书
echo "<certificate-authority-data>" | base64 -d > ca.crt
echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1ERXhNVEEwTlRRMU0xb1hEVE14TURFd09UQTBOVFExTTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTDdTCmNReTNnZDRoK0dHNitKcC8wcmloVGhkd096ZUsxZEFkK1oyK016ZkhvdFVMd2xva2dtd0h0S2JyTm9RdVNFOTMKdXFHbnpBeDArWmduRkx4cVR1bHk4U1A0eXhJTW9Fb0oxKytXNDl0ZklQa0NHOE50aVlmaG5sZ1VIU2haalR2TwpVSzF0Z2ZmSzR1UGlyazk1QWFYWnArMDloQmZDVnR4aFBjbzBxSDZlUVJUZk5xdVU1cHg3TnQzVlo4Wm5MQVBICkxKb0JCS081ZjJXd1l0aXNtZ0FxSkZCWWltQk84R1Z4azdEcjQwVGRRVWZ0NWNmZmFERW1tMlByb0l5WWpobXQKbDIrYVNJc3pGRFBRbDZpMVFnNVJKbzJ2aTgzTDltQ1F5Z3RWVEQ3QUwvSFkralNHbDgvbTVISlBvSUJWY0ZoVApUTi9DQmYwMWIzd1lqNGhVeFVzQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZIZHpBb2lGNytHeVpGN1dDZUFQbndNRjZQNk5NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBK2pjL1VGTkZNUUp5a1pEYUV0NDQ2MWg4cXQyOFVjYVYzelU0NUlyY2psaXU1T0FJZAoya2RYdXFtSCtncUJmMktDY1d0VGM4VU83ak9Kbm8yTWRpZVpJNG5od0crWG5mOWpjcFhabE5Ma1RjazdqdS9CCmk5UUUvWVdJS2JWOW9nQmlwTXR1VmdLbW4rYmM5b0RlTVk4dWRnN2czVUE1TVlla1ZqOG1FYXB3UmEyZk9udmcKVVI4Znd4Q2xyRXdtS2tqVlhKL0hSaWU1NFcvWkkzT1l6dE1zaGFRT0VNMTY2K2lQWUxSU3lscG96RTZ6WFZMZQpvVVQ0RCttdlJHRVJTcytpaVYzNzJLSENQUDByTWJHSHBncG1BNFh4NVpOdnlKL29TVTNIUjRGWVBzMEUvaG5KCktsalRFQThuM0FsTUxkWlhHTHd6ckoydEV5bENmYUFVaUFxVAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==" | base64 -d > ca.crt

# (2) 复制 client-certificate-data的内容，运行以下命令生成 `client.crt` -  客户端证书
echo "<client-certificate-data>" | base64 -d > client.crt
echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJV0JlN0JvSnhTWEF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBeE1URXdORFUwTlROYUZ3MHlNakF4TVRFd05EVTBOVFphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTVxL1lOYTh0K2JQZk1rUGwKZnU1NGVCVGtCSHVlcURKWTFvZ0pDVFMyWFNydkVrNTB4ZDVPVVZCWXloNlo4ZFdPZXhRVmVNR3RYZzlObWMzbwprVGRlMzh2eVRVNEE5VXNDakpibDlVenNBR3NCNGRUYVN2azJPUEJWN0pZYUNYMnZQQUtvdEI2aVl5WnJmQW5FClY2UlYzYTlKS1RaSEw3bFNUc3VtZ3A2WGtTVHBaSnYrSEpHTHF0ZFYxNk8vdjhoZ0llL2s1a0lZSjR1d01VOFIKUEpTbGdhdDJhck9GSU9HYkVqTEhjd3lXSWxFcGJSbklxQW41MzhvR01WZHF3WlhoaHRvbjZDWFJya1poRzRTbQpZUnBUdy9WZDRhTnB5ZElqbU9NYmlqeFE5aVRMckp1QjhNc3h6UnZjc2dkeG9iNjNCTzNCV0lWTE9aUExaVlU4Cks5M3BXUUlEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVkM01DaUlYdjRiSmtYdFlKNEErZkF3WG8vbzB3RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFMdWw4c3Fhc0VCRGVLclBPNUNZTk5UQjVoNXlBZ25SWi9TMDQ4S3NEOUd3UWhjeFFhdDFSdHRPClRuQ0VFbzZlU0t6ZXpMMW9ER3JiU2w3NDJkRFhiSDJzd0VHeFJjTEJ4ZTVhRVUxK2N6QklGRlBPN0xaWFlvYXUKNVRJeDVSODVVMXU3WXFUVERkUXR3LzFoUFJGajI0UnFSWkVIekl4NWpaZFY1ZTk0b0NYVENnVHhCeTNsdUZnMApSeXRML3I3eEdHdzF5cFdTZ0RUYVNQU2c3eGxNVGxOWVRFMVJ1RGh2THNpelpFb09PMjJnWDM4L0lCU1dwQ1U5CjJFUlFBbUhoYTZqOWQ4eDBub2xieTVIYXZqS3VadVVhYWMrVVNWNFNaMGw0QjJWeC9pNzZVSGRhSmgxL2lZenAKdHJEQ3NZeThTY2hFZFBxemdvS3krU2QxS3pNakZ0Zz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="  | base64 -d > client.crt

# (3) 复制 client-key-data的内容，运行以下命令生成 `client.key` - 密钥
echo "<client-key-data>" | base64 -d > client.key
echo "LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBNXEvWU5hOHQrYlBmTWtQbGZ1NTRlQlRrQkh1ZXFESlkxb2dKQ1RTMlhTcnZFazUwCnhkNU9VVkJZeWg2WjhkV09leFFWZU1HdFhnOU5tYzNva1RkZTM4dnlUVTRBOVVzQ2pKYmw5VXpzQUdzQjRkVGEKU3ZrMk9QQlY3SllhQ1gydlBBS290QjZpWXlacmZBbkVWNlJWM2E5SktUWkhMN2xTVHN1bWdwNlhrU1RwWkp2KwpISkdMcXRkVjE2Ty92OGhnSWUvazVrSVlKNHV3TVU4UlBKU2xnYXQyYXJPRklPR2JFakxIY3d5V0lsRXBiUm5JCnFBbjUzOG9HTVZkcXdaWGhodG9uNkNYUnJrWmhHNFNtWVJwVHcvVmQ0YU5weWRJam1PTWJpanhROWlUTHJKdUIKOE1zeHpSdmNzZ2R4b2I2M0JPM0JXSVZMT1pQTFpWVThLOTNwV1FJREFRQUJBb0lCQUR2UWdvNUE4dm5aQXRtRQpzMS83TTI5bmMwd2FSYVExRWNYbWxmazJHc2NEbCtPMlJoNzhLbkI1RmR5cW5KNFJFcFdsT29BS01BckFpdzJEClQzYy8xVERRTCs2TmVFQWlCL0l1T2tnbGZ0Z0k1djhJY3VXWHd0QjJ1TURVbHNHNVBoT2dXT0FEUlhYU0EzS3gKRWFEcjhudTl0SW1rRWtjMGxUdnJJQ3lrTklha2ZudkJMbVVsWm9RdFRBWVBQZVk0bmYzK3YrLzJodHJkWittWgpxRzdhQ0tBOHUwcUYxMFpoYlNXUWNVbWpOYnJRODMwa0JON04vYlFwSVNUcEh3NVNzMjdZM2VMY0lCTjRzTHZUCndISGRPbXVLOXI4YXpsWTYza1d3MlJNL1NGQlhxM0FpOUo2T1hnUzNaUndTVzk1aEFheDhtQUFINnFYZ1liaWIKS0MyTzZkRUNnWUVBOEVEZ1Z1ZFJVYis5R0ZJZTNNYnl5VWFyWU01cEZmdU80N2lNeDJuWnBXUWlCN0RVbGdrbQpPTERyOHpzL2NFbUdaZzdFT2RYcUxBQkcvUzFwbi80aHRZcXFlaXNUWGJBcDY0Ky9QMHQ5SjRSWUc0TWk3c1ROCjdtbzdZRnkwbU5xU3dEZTFiWndGeTZTczJteWtpU2ZtNFIwY041QmtMZHpHQk9BdXI3eldzeTBDZ1lFQTljNTAKZC9sbWpvZXBQQW94V2dZaEJFTzdnQ2FEbDYreWRNY3hwZ2RNNWVYdEs0d1o3b0IzOGY2Nk9yUkMyR21Hek51cQo1cTdFeWVpZzh5UHNTVHBLVndTRFA1WlRIUGZSU0JaRmlxeCt3SlVLMHFWWUtPNENVU29tSVZqeE1pOXRLenRRCmRRNVR4UnlPWUhzT1ZYMk5EVTZwMWFQTE5DVGFFeGlJbUZNRldsMENnWUJpbnQ3NERXQXVKSHprdk9ENlU1aFoKMHU2S2dIQldtN3FkODZXbVBlY2ZveWpzNjBONGl5enJYSVNlaFpXVzdEZUZNVTZQUnlZbkJiNGVNMFFHYnZVNwpaajV3ZzdvaFhTejRDenZBS2Fhb1VBVXkxZlBDKzNwbEFhcDU5ZFFVWXJTV3ZzZDB4UFVFRVFiN2FsbG9DNzhVCmJUU21BbGw5RWdFZkF6OW0yQ2R4eVFLQmdIL2p4UEZQRDY4RW9tYWNud1RKdjQvcWRibTlVQ1l4d2RYRWRlNSsKU2VJcmVQUjVWbHlpOXNVdjFWRUp6T1d3TWZTUUxpRUx1Vk9iOTNISnRQeDhtWVVnMGZEWms3QzB0MnljT2Q1bQoxU1A1NThHbFNYTXlNbjVzUVo2RUdpb1VSdWFCVytFcmJTWlhMelMva2J1bE1TaEZUMVBhZnJWSW56WGtROTJOCkJISDVBb0dBSWdrVFg1UnA2bk10UHF1aytOMy9JQVNXS2lURkxwQ0d6VXNsaGNoejBQM25oMDQxR3dsMlNveGYKNkRlYmRRK2ovbTJPcG9sai9sQm04NVB0WGx5WTdUTUh2MVFKSCs4TEFUb0NJQTRuaHdOMjl6V1hrRXhySTVTZApRU1NobU93R25nbWhZVEhEWWlTeDdPWFFhODE1dGpSc1hQRWtNVS9DOWs2UXdWK1RvRU09Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==" | base64 -d > client.key

# (4) 之后再根据前面步骤生成的 ca.crt, client.crt 和 client.key 来生成 PKCS12 格式的 cert.pfx
openssl pkcs12 -export -out cert.pfx -inkey client.key -in client.crt -certfile ca.crt
  # Enter Export Password: weiyigeek
  # Verifying - Enter Export Password: weiyigeek
```

Step 3.在Jenkins上集成 Kubernetes 首先需要将 cert.pfx 导入到`Jenkins Global Credential` 凭据中(注意输入密码)

![WeiyiGeek.cert.pfx](https://i0.hdslb.com/bfs/article/4d23d35e570cb16be274068abb461155eaf5dd4d.png@942w_950h_progressive.webp)

Step 4.然后按照上面的方式在`Jenkins`上配置Kubernetes Cloud, 不同的是需要填入以下信息，最后点击“Test Connection"按钮测试Jenkins是否可以成功连接Kubernetes。

```
# Kubernetes 名称 : kubernetes-external
# Kubernetes 地址 : https://192.168.12.110:16443    # 此处由于做了高可用所以不是master节点地址:6443
# Kubernetes 服务证书 key ：ca.crt 的内容
# Kubernetes 命名空间 : devops
# Jenkins 地址 ：http://jenkins.devops.svc.cluster.local:8080
# Jenkins 通道 ：jenkins.devops.svc.cluster.local:50000
# Tips : 注意勾选从 master 传递给 agent 的环境变量
```

![WeiyiGeek.](https://i0.hdslb.com/bfs/article/ad8a132b99c9a9655071bc593371deec8563e668.png@942w_716h_progressive.webp)

Tips ： 在进行第五步是建议在各个Work节点拉取`jenkins/inbound-agent:4.3-4`镜像

-   Step 5.下面设置 Pod Templates 此处需添加一个Pod 模板名称等相关信息
    

```
# 添加 Pod 模板
Pod 模板名称: jenkins-agent-jnlp
命名空间：devops                 # 一定不要写错误了否则不能创建Pod
标签列表: jenkins-agent-jnlp     # 后续Job可以指定该标签进行运行

# 添加容器列表 (jenkins/inbound-agent:4.3-4)  # 差点把我坑哭了 (测试时候可以先不加容器，默认是jnlp容器)
# 名称 ：       maven
# Docker 镜像 : maven:3.6-jdk-8-alpine
# 工作目录 : /home/jenkins/
# 运行的命令 : sleep 9000

# 挂载到 Pod 代理中的卷列表
# 选择 Host Path Volume 将maven进行持久化存储（此处路径与您setting配置有关默认是运行用户家目录中）
# Maven 持久化目录 : /home/jenkins/.m2  # 此处应该是您各个K8s的Work节点上NFS目录;
/nfsdisk-31/appstorage/mavenRepo # 主机目录（root@weiyigeek-107 ）NFS目录
/home/jenkins/.m2                # Pod挂载目录
# docker socket 目录:
/var/run/docker.sock    # 主机目录(由于各个节点都安装docker)
/var/run/docker.sock    # Pod挂载目录

# Service Account 账户权限
Service Account ：jenkins-sa   # 前面创建的ServiceAccount
```

![WeiyiGeek.Slave Pod 模板](https://i0.hdslb.com/bfs/article/e481b960da9afaf65ad208f0477428df531514ad.png@942w_495h_progressive.webp)

Tips : 警告如果要为JNLP代理提供自己的Docker映像则必须将容器命名为jnlp，以便它覆盖默认容器否则，将导致两个代理尝试同时连接到主服务器

Tips : Kubernetes 默认的jnlp容器是`"jenkins/inbound-agent:4.3-4", name: "jnlp"`，我们可以自定义容器进行覆盖只需将容器名称更改为jnlp即可(一般情况下不建议更改);

Tips ：镜像的选择就是一个坑开始使用的是`jenkins/inbound-agent:alpine`然后容器名称并未设置为jnlp覆盖默认的`"jenkins/inbound-agent:4.3-4"`容器，根本都没有执行节点加入命令, 此处凸显出官网的重要性，发现官网采用的是`jenkins/jnlp-slave:latest`(https://hub.docker.com/r/jenkins/jnlp-slave) 或者不加镜像容器信息 最后完满解决;

-   Step 6.创建一个Job Pipeline 验证测试环境;
    

```
// # scripted Pipeline
def podTemplates = 'jenkins-slave'
podTemplate(label: podTemplates, cloud: 'kubernetes')
{
    node (podTemplates) {
        stage('init') {
            echo "Hello world, Kubernetes Jenkins Slave"
            sh "hostname && ls /home/jenkins/.m2 && sleep 300"
        }
    }
}
```

输出结果:

```
// # scripted Pipeline
podTemplate(name: 'jenkins-slave', label: 'jenkins-slave', cloud: 'kubernetes',containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6-jdk-8-alpine', ttyEnabled: true, command: 'sleep', args: '30')
]){
    node (podTemplates) {
        stage('init') {
            container ('maven') {
                echo "Hello world, Kubernetes Jenkins Slave"
                sh 'mvn -version'
                sh "hostname && ls /home/jenkins/.m2 && pwd"
                sh "env"
                sh "pwd && ls"
            }
        }
    }
}
```

![WeiyiGeek.result](https://i0.hdslb.com/bfs/article/58bfd60e8fc3225f5c6ae849c68d498da4349f41.png@942w_782h_progressive.webp)

描述: 在 K8s 中对 Jenkins 升级是非常的简单只需要把image键中版本值进行改变(`只需要使用新的版本镜像替换即可`)，从而拉取新的镜像运行即可。

Tips : 注意此处做了PVC持久化如果未作持久化的童鞋需要注意数据的保存, 其次是拉取的版本的Jenkins镜像必须存在

```
$ grep "jenkins:2.277-alpine" jenkins-deployment.yaml
  # image: jenkins/jenkins:2.277-alpine
  # https://updates.jenkins.io/download/war/2.277/jenkins.war
$ kubectl apply -f jenkins-deployment.yaml
$ kubectl get pod -n devops
  # NAME                       READY   STATUS              RESTARTS   AGE
  # jenkins-5df679b7ff-5r8d4   0/1     ContainerCreating   0          24s
  # jenkins-7db7878f8f-78tmk   1/1     Running             2          18d  # 原版本等待新版本的Pod启动完毕后自动销毁;
```

Tips: 对于Jenkins版本在生产环境中不建议冒冒失失进行升级其中原因你我心知，但是针对于出现严重漏洞不得不修复时，建议构建新的环境再依次迁移（同时注意备份），确定无任何问题后再关闭下线并进行切换

Tips : 此处为测试学习环境，看到这个提示我的强迫症就上来了;

![WeiyiGeek.K8s集群中对Jenkins进行升级](https://i0.hdslb.com/bfs/article/20fe98e37b1312c7e07cd03606cb0d01ec7c73bf.png@672w_719h_progressive.webp)

简单操作流程:

```
$ tar -zxvf jenkins_2.272.x_plugins.tar.gz -C ./jenkins/

~/jenkins$ cd var/lib/jenkins/plugins/

~/jenkins/var/lib/jenkins/plugins$ cp -a . /nfsdisk-31/devops-jenkins-pvc-pvc-3cd916df-91cb-470d-b9ef-e9b4f115223d/plugins/

$ chown -R jenkins:jenkins /nfsdisk-31/devops-jenkins-pvc-pvc-3cd916df-91cb-470d-b9ef-e9b4f115223d/plugins
```

Tips : 注意此处是我的PVC持久化的目录路径,与你实践的环境是不一致的。

-   错误信息: 没有权限在 jenkins 的 home 目录下面创建文件;
    

```
kubectl -n kube-ops logs jenkins2-59764f8f65-rcvh5
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
```

-   错误原因: 因为默认的镜像使用的是 jenkins 这个用户，而我们通过 PVC 挂载到 nfs 服务器的共享数据目录下面却是 root 用户的，所以没有权限访问该目录
    
-   问题解决: 只需要在 nfs 共享数据目录下面把我们的目录权限重新分配下即可：chown -R 1000 /data/k8s/jenkins2  
    

**问题描述: Agent 不能 通过 jnlp 与 Jenkins 的 Master 相连接**

```
2021-02-04 05:37:46.886+0000 [id=1985]  WARNING o.c.j.p.k.KubernetesLauncher#launch: Error in provisioning; agent=KubernetesSlave name: jenkins-agent-jnlp-b63kv, template=PodTemplate{id='83f07044-1633-468a-91c8-e05ffb924303', name='jenkins-agent-jnlp', namespace='devops', label='jenkins-agent-jnlp', serviceAccount='jenkins-sa', volumes=[HostPathVolume [mountPath=/home/jenkins/.m2, hostPath=/tmp/.m2]], containers=[ContainerTemplate{name='jnlp', image='jenkins/inbound-agent:alpine', workingDir='/home/jenkins', command='sleep', args='9999999', resourceRequestCpu='', resourceRequestMemory='', resourceRequestEphemeralStorage='', resourceLimitCpu='', resourceLimitMemory='', resourceLimitEphemeralStorage='', livenessProbe=ContainerLivenessProbe{execArgs='', timeoutSeconds=0, initialDelaySeconds=0, failureThreshold=0, periodSeconds=0, successThreshold=0}}]}
java.lang.IllegalStateException: Agent is not connected after 100 seconds, status: Running
        at org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher.launch(KubernetesLauncher.java:244)
        at hudson.slaves.SlaveComputer.lambda$_connect$0(SlaveComputer.java:294)
        at jenkins.util.ContextResettingExecutorService$2.call(ContextResettingExecutorService.java:46)
        at jenkins.security.ImpersonatingExecutorService$2.call(ImpersonatingExecutorService.java:80)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
2021-02-04 05:37:46.886+0000 [id=1985]  INFO    o.c.j.p.k.KubernetesSlave#_terminate: Terminating Kubernetes instance for agent jenkins-agent-jnlp-b63kv
2021-02-04 05:37:46.900+0000 [id=1985]  INFO    o.c.j.p.k.KubernetesSlave#deleteSlavePod: Terminated Kubernetes instance for agent devops/jenkins-agent-jnlp-b63kv
2021-02-04 05:37:46.901+0000 [id=1985]  INFO    o.c.j.p.k.KubernetesSlave#_terminate: Disconnected computer jenkins-agent-jnlp-b63kv
Terminated Kubernetes instance for agent devops/jenkins-agent-jnlp-b63kv
Disconnected computer jenkins-agent-jnlp-b63kv
```

**问题原因:**

> 答: 这个问题困扰了我好久，总结可能出现该问题的情况，  
> 1.指定的 Jenkins-jnlp 容器镜像的Agent不能正常连接到Master  
> 2.指定的 Jenkins-jnlp 镜像启动参数问题。

**解决办法:**

> 答: 换镜像,在后面的章节中我会将自定义Jenkins Slave Jnlp 容器镜像的DockerFile文件进行分享。

异常问题:

```
FATAL: java.nio.channels.ClosedChannelException
java.nio.channels.ClosedChannelException
Also:   hudson.remoting.Channel$CallSiteStackTrace: Remote call to JNLP4-connect connection from 10.244.8.1/10.244.8.1:55340
at hudson.remoting.Channel.attachCallSiteStackTrace(Channel.java:1741)
at hudson.remoting.Request.call(Request.java:202)
at hudson.remoting.Channel.call(Channel.java:954)
at hudson.FilePath.act(FilePath.java:1071)
at hudson.FilePath.act(FilePath.java:1060)
at hudson.FilePath.mkdirs(FilePath.java:1245)
at hudson.model.AbstractProject.checkout(AbstractProject.java:1202)
at hudson.model.AbstractBuild$AbstractBuildExecution.defaultCheckout(AbstractBuild.java:574)
at jenkins.scm.SCMCheckoutStrategy.checkout(SCMCheckoutStrategy.java:86)
at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:499)
at hudson.model.Run.execute(Run.java:1819)
at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
at hudson.model.ResourceController.execute(ResourceController.java:97)
at hudson.model.Executor.run(Executor.java:429)
Caused: hudson.remoting.RequestAbortedException
at hudson.remoting.Request.abort(Request.java:340)
at hudson.remoting.Channel.terminate(Channel.java:1038)
at org.jenkinsci.remoting.protocol.impl.ChannelApplicationLayer.onReadClosed(ChannelApplicationLayer.java:209)
at org.jenkinsci.remoting.protocol.ApplicationLayer.onRecvClosed(ApplicationLayer.java:222)
at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:832)
at org.jenkinsci.remoting.protocol.FilterLayer.onRecvClosed(FilterLayer.java:287)
at org.jenkinsci.remoting.protocol.impl.SSLEngineFilterLayer.onRecvClosed(SSLEngineFilterLayer.java:172)
at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:832)
at org.jenkinsci.remoting.protocol.NetworkLayer.onRecvClosed(NetworkLayer.java:154)
at org.jenkinsci.remoting.protocol.impl.NIONetworkLayer.ready(NIONetworkLayer.java:142)
at org.jenkinsci.remoting.protocol.IOHub$OnReady.run(IOHub.java:795)
at jenkins.util.ContextResettingExecutorService$1.run(ContextResettingExecutorService.java:28)
at jenkins.security.ImpersonatingExecutorService$1.run(ImpersonatingExecutorService.java:59)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
Finished: FAILURE
```

问题原因: 抛 `java.nio.channels.ClosedChannelException` 异常的原因是 `Jenkins Slave Pod 在 Jenkins Job` 运行时突然挂掉，然后 Master Pod 无法和 Slave Pod 进行通信。那么解决方法就是找到 Slave Pod 经常挂掉的原因，经排查是 Slave Pod 的资源限制不合理，配置的 CPU 和内存太小，导致 Pod 在运行是很容易超出资源限制，然后被 k8s Kill 掉。

解决办法:打开 Jenkins 设置 Slave Pod 模版的资源限制：`Jenkins->系统管理->系统设置->云->镜像->Kubernetes Pod Template->Container Template->高级`，然后根据实际情况调整 CPU 和内存需求。

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

>         欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/04ec2a1812ca192ea0870ed12a235e1a3ba7b019.png@942w_483h_progressive.webp)

千磨万击还坚韧，任尔东西南北风。—— 郑板桥《竹石》     

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！