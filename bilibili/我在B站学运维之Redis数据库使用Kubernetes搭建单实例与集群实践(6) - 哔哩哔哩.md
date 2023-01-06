**本章目录:**

-   0x00 前言简述
    
-   0x01 Redis 单实例实践
    

-   1.配置准备
    
-   2.流程步骤
    

-   0x02 Redis 集群主从实践
    

-   (1) 自定义Redis集群构建
    

-   1.配置准备
    
-   2.流程步骤
    
-   3.使用实践
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 我们知道在 Kubernetes 容器编排平台中, 我们可以非常方便的进行应用的扩容缩, 同时也能非常方便的进行业务的迭代，此处由于学习以及开发测试的需求，本章主要讲解在Kubernetes搭建了单实例和Redis集群主从同步的环境流程步骤, 如果是高频访问重要的线上业务我们最好是部署在物理机器上;

**K8S 环境说明:**

```
# K8S 高可用集群
~$ kubectl cluster-info
  Kubernetes master is running at https://WeiyiGeek-lb-vip.k8s:16443
  KubeDNS is running at https://WeiyiGeek-lb-vip.k8s:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# K8s 集群节点
~$ kubectl get node -o wide
  NAME       STATUS   ROLES    AGE    VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
  WeiyiGeek-107   Ready    master   238d   v1.19.6   192.168.12.107   <none>        Ubuntu 20.04.1 LTS   5.4.0-70-generic   docker://20.10.3
  WeiyiGeek-108   Ready    master   238d   v1.19.6   192.168.12.108   <none>        Ubuntu 20.04.1 LTS   5.4.0-60-generic   docker://19.3.14
  WeiyiGeek-109   Ready    master   238d   v1.19.6   192.168.12.109   <none>        Ubuntu 20.04.1 LTS   5.4.0-60-generic   docker://19.3.14
  WeiyiGeek-223   Ready    <none>   238d   v1.19.6   192.168.12.223   <none>        Ubuntu 20.04.1 LTS   5.4.0-42-generic   docker://19.3.14
  WeiyiGeek-224   Ready    <none>   238d   v1.19.6   192.168.12.224   <none>        Ubuntu 20.04.1 LTS   5.4.0-42-generic   docker://19.3.14
  WeiyiGeek-225   Ready    <none>   238d   v1.19.6   192.168.12.225   <none>        Ubuntu 20.04.1 LTS   5.4.0-42-generic   docker://19.3.14

# 挂载的NFS到各个节点
 mount -l | grep "nfsdisk-31"
192.168.12.31:/nask8sapp on /nfsdisk-31 type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.12.31,mountvers=3,mountport=1048,mountproto=udp,local_lock=none,addr=192.168.12.31)


# 动态卷存储(NFS)
$ kubectl get storageclasses.storage.k8s.io
  NAME                            PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
  managed-nfs-storage (default)   fuseim.pri/ifs   Delete          Immediate           false                  234d
```

**Redis 版本说明:**

Redis 6.2.5  

Redis-cli 6.2.5

描述: 首先我们先使用kubernetes进行安装部署Redis单示例，此处有两种方案进行`redis`配置文件的引入一种是通过`hostPath`，另外一种是通过`configMap`(推荐方式)，不管您选择哪种方案最终实现的效果都是一样，只不过configMap更方便Redis服务配置的管理与更新。

-   Step 1.Redis 服务的配置文件清单准备。
    

```
tee /nfsdisk-31/datastore/redis/demo1/redis.conf <<'EOF'
# 绑定任意接口、服务端口、后台运行。
bind 0.0.0.0
port 6379
daemonize no
supervised no

# redis服务pid进程文件名
pidfile "/var/run/redis.pid"

# 关闭保护模式，并配置使用密码访问
protected-mode no
requirepass 123456

# 数据文件保存路径，rdb/AOF文件也保存在这里
dir "/data"

# 日志文件记录文件(notice / verbose)
# /var/log/redis/redis.log
loglevel verbose  
logfile "/logs/redis.log"

# 最大客户端连接数
maxclients 10000

# 客户端连接空闲多久后断开连接，单位秒，0表示禁用
timeout 300
tcp-keepalive 60 


# 内存初始化
maxmemory 1gb
maxmemory-policy volatile-lru
slowlog-max-len 128
lua-time-limit 5000

# Redis 数据持久化(rdb/aof)配置
# 数据自动保存脚本条件例如300s中有10key发生变化
save 300 10
save 60 10000
# RDB 文件名
dbfilename "dump.rdb"
# 对RDB文件进行压缩，建议以（磁盘）空间换（CPU）时间。
rdbcompression yes
# 版本5的RDB有一个CRC64算法的校验和放在了文件的最后。这将使文件格式更加可靠。
rdbchecksum yes
# RDB自动触发策略是否启用，默认为yes
rdb-save-incremental-fsync yes

# AOF开启
appendonly yes
# AOF文件名
appendfilename "appendonly.aof"
# 可选值 always， everysec，no，建议设置为everysec
appendfsync everysec

# Redis风险命令重命名
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command FLUSHDB b840fc02d524045429941cc15f59e41cb7be6c53
rename-command FLUSHALL b840fc02d524045429941cc15f59e41cb7be6c54
rename-command EVAL b840fc02d524045429941cc15f59e41cb7be6c55
rename-command DEBUG b840fc02d524045429941cc15f59e41cb7be6c56
# rename-command SHUTDOWN SHUTDOWN
EOF
```

Tips: 非常大的巨坑在使用k8s中的container作为redis容器时其`daemonize no`一定要设置为no 【非常注意】。

-   Step 2.准备Redis相关目录的创建与创建configMap资源对象的redis配置文件
    

```
# 数据与日志存储目录
mkdir /nfsdisk-31/datastore/redis/demo1/{data,logs}
```

-   Step 3.方式1.k8s资源清单的部署准备(`此处采用hostpath卷的方式来传入配置文件以及存储持久化的redis{aof/rdb}数据`)
    

```
tee Redis-single.yaml <<'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: database
spec:
  serviceName: redisdemo1
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2.5-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
          name: server
        command: [ "redis-server", "/conf/redis.conf" ]
        # 单个文件与目录挂载
        volumeMounts:
        - name: conf
          mountPath: /conf/redis.conf
          readOnly: true
        - name: data
          mountPath: /data
        - name: logs
          mountPath: /logs
        # 时区设置
        - name: timezone
          mountPath: /etc/localtime
      volumes:
      - name: conf
        # 采用hostPath卷,只映射配置文件到pod中
        hostPath:
          type: FileOrCreate
          path: /nfsdisk-31/datastore/redis/demo1/redis.conf
      - name: data
        # 采用hostPath卷
        hostPath:
          type: DirectoryOrCreate     
          path: /nfsdisk-31/datastore/redis/demo1/data
      - name: logs
        # 采用hostPath卷
        hostPath:
          type: DirectoryOrCreate 
          path: /nfsdisk-31/datastore/redis/demo1/logs
        # 时区定义
      - name: timezone                             
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
---
apiVersion: v1
kind: Service
metadata:
  name: redisdemo1
  namespace: database
spec:
  type: ClusterIP
  ports:
  - port: 6379
    targetPort: 6379
    name: server
  selector:
    app: redis
EOF
```

-   Step 4.方式2.k8s资源清单的部署准备(`此处采用configMap的方式来传入配置文件以及采用动态存储卷来存储持久化的redis{aof/rdb}数据`)
    

```
tee redis-single-1.yaml <<'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cm
  namespace: database
spec:
  serviceName: redisdemo2
  replicas: 1
  selector:
    matchLabels:
      app: redis-cm
  template:
    metadata:
      labels:
        app: redis-cm
    spec:
      containers:
      - name: redis
        image: redis:6.2.5-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
          name: server
        command: [ "redis-server", "/conf/redis.conf"]
        volumeMounts:
        # 从configmap获取的配置文件，挂载到指定文件中
        - name: conf
          mountPath: /conf/redis.conf
          subPath: redis.conf
        - name: data
          mountPath: /data
        - name: logs
          mountPath: /logs
        # 时区设置
        - name: timezone
          mountPath: /etc/localtime              
      volumes:
      - name: conf
        # 配置文件采用configMap
        configMap:
          name: redis-single
          defaultMode: 0755
        # 日志采用hostPath卷
      - name: logs
        hostPath:
          type: DirectoryOrCreate 
          path: /nfsdisk-31/datastore/redis/demo1/logs
        # 时区定义
      - name: timezone                             
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "managed-nfs-storage"
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redisdemo2
  namespace: database
spec:
  type: ClusterIP
  ports:
  - port: 6379
    targetPort: 6379
    name: server
  selector:
    app: redis-cm
EOF
```

Step 1.部署redis单实例基础环境

```
# (1) 名称空间的创建 （PS:非常注意在删除名称空间时需要查看是否有存在Pod）
kubectl create namespace database

# (2) Redis 配置文件部署到configMap中 & 查看
kubectl create configmap -n database redis-single --from-file=/nfsdisk-31/datastore/redis/demo1/redis.conf
kubectl get cm -n database redis-single -o json
  # NAME           DATA   AGE
  # redis-single   1      12s
```

![WeiyiGeek.configMap资源](https://i0.hdslb.com/bfs/article/a023290222cc7f976828e3540ccd1a52cf839fc4.png@942w_378h_progressive.webp)

-   Step 2.按照方式1资源清单构建项目（hostpath的方式）
    

```
# (1) 资源清单的部署
kubectl create --save-config -f Redis-single.yaml
  # statefulset.apps/redis created
  # service/redisdemo1 created

# (2) 部署的redis单实例pod，sts,svc 查看
kubectl get sts -n database
  # NAME            READY   AGE
  # redis           1/1     9m16s

kubectl get pod -n database -l app=redis -o wide --show-labels
  # NAME      READY   STATUS    RESTARTS   AGE     IP               NODE       LABELS
  # redis-0   1/1     Running   0          4m12s   172.16.183.73   WeiyiGeek-224   app=redis,controller-revision-hash=redis-6d69c85769,statefulset.kubernetes.io/pod-name=redis-0

kubectl get svc -n database | grep "redisdemo1"
  # redisdemo1            ClusterIP   10.100.14.177   <none>        6379/TCP                         8m47s


# (3) pod启动日志查看
kubectl logs -n database -l app=redis
  # 1:C 07 Sep 2021 13:15:51.149 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=1, just started
  # 1:C 07 Sep 2021 13:15:51.149 # Configuration loaded
  # 1:M 07 Sep 2021 13:15:51.150 * monotonic clock: POSIX clock_gettime
  # 1:M 07 Sep 2021 13:15:51.151 * Running mode=standalone, port=6379.
  # 1:M 07 Sep 2021 13:15:51.151 # Server initialized
  # 1:M 07 Sep 2021 13:15:51.153 * Loading RDB produced by version 6.2.5
  # 1:M 07 Sep 2021 13:15:51.153 * RDB age 247 seconds
  # 1:M 07 Sep 2021 13:15:51.153 * RDB memory usage when created 0.77 Mb
  # 1:M 07 Sep 2021 13:15:51.153 * DB loaded from disk: 0.002 seconds
  # 1:M 07 Sep 2021 13:15:51.153 * Ready to accept connections


# (4) 服务访问验证测试
redis-cli -h 10.100.14.177
10.100.14.177:6379> auth 123456 # OK
10.100.14.177:6379> ping        # PONG
10.100.14.177:6379> role        # 1) "master"
10.100.14.177:6379> set name weiyigeek # OK
10.100.14.177:6379> hset bashdemo name "weiyigeek" age 18      # (integer) 2
10.100.14.177:6379> hset bashdemo addr "ChongQing" pro "北京"  # (integer) 2
10.100.14.177:6379> get name    # "weiyigeek"
10.100.14.177:6379> HGETALL bashdemo
1) "name"
2) "weiyigeek"
3) "age"
4) "18"
5) "addr"
6) "ChongQing"
7) "pro"
8) "\xe5\x8c\x97\xe4\xba\xac"
10.100.14.177:6379> HVALS bashdemo
1) "weiyigeek"
2) "18"
3) "ChongQing"
4) "\xe5\x8c\x97\xe4\xba\xac"
10.100.14.177:6379> save # OK
```

-   Step 3.按照方式2资源清单构建项目（动态卷的方式）
    

```
# (1) 部署redis的资源清单
kubectl create --save-config -f redis-single-1.yaml

# (2) 查看不是的sts，pod，svc，pv等资源
kubectl get sts -n database -o wide --show-labels redis-cm
  # NAME       READY   AGE     CONTAINERS   IMAGES               LABELS
  # redis-cm   1/1     3m26s   redis        redis:6.2.5-alpine   <none>

kubectl get pod -n database -o wide --show-labels -l app=redis-cm
  # NAME         READY   STATUS    RESTARTS   AGE     IP               NODE      LABELS
  # redis-cm-0   1/1     Running   0          2m54s   172.16.100.106   WeiyiGeek-224  app=redis-cm,controller-revision-hash=redis-cm-5988f6b7f8,statefulset.kubernetes.io/pod-name=redis-cm-0

kubectl get svc -n database -o wide --show-labels redisdemo2
  # NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR       LABELS
  # redisdemo2   ClusterIP   10.102.39.181   <none>        6379/TCP   6m12s   app=redis-cm   <none>

kubectl get pvc -n database -o wide --show-labels -l app=redis-cm
  # NAME                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE    VOLUMEMODE   LABELS
  # persistentvolumeclaim/data-redis-cm-0   Bound    pvc-00ee48e8-b1ca-4640-96c5-16f7265a2c61   1Gi        RWO            managed-nfs-storage   4m4s   Filesystem   app=redis-cm


# (3) pod启动日志查看
$ kubectl logs -n database redis-cm-0
  # 1:C 07 Sep 2021 14:48:10.357 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  # 1:C 07 Sep 2021 14:48:10.357 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=1, just started
  # 1:C 07 Sep 2021 14:48:10.357 # Configuration loaded
  # 1:M 07 Sep 2021 14:48:10.357 * monotonic clock: POSIX clock_gettime
  # 1:M 07 Sep 2021 14:48:10.358 * Running mode=standalone, port=6379.
  # 1:M 07 Sep 2021 14:48:10.358 # Server initialized
  # 1:M 07 Sep 2021 14:48:10.360 * Ready to accept connections


# (4) 服务访问验证测试
$ redis-cli -h 10.102.39.181
10.102.39.181:6379> auth 123456  # OK
10.102.39.181:6379> keys *       # (empty list or set)
10.102.39.181:6379> set name weiyigeek # OK
10.102.39.181:6379> hset 10086 name "China Mobile" age "15"  # (integer) 2
10.102.39.181:6379> get name        # "weiyigeek"
10.102.39.181:6379> HGET 10086 name # "China Mobile"
10.102.39.181:6379> HGETALL 10086
1) "name"
2) "China Mobile"
3) "age"
4) "15"
10.102.39.181:6379> save # OK

# (5) 查看其持久化rdb与aof文件
/nfsdisk-31/data/database-data-redis-cm-0-pvc-00ee48e8-b1ca-4640-96c5-16f7265a2c61# ls
  # appendonly.aof  dump.rdb

# (6) 访问日志查看
/nfsdisk-31/datastore/redis/demo1/logs# cat redis.log
  # 1:M 07 Sep 2021 15:08:58.044 - Accepted 172.16.0.192:59435
  # 1:M 07 Sep 2021 15:09:17.382 - DB 0: 1 keys (0 volatile) in 4 slots HT.
  # 1:M 07 Sep 2021 15:12:32.897 - DB 0: 2 keys (0 volatile) in 4 slots HT.
  # 1:M 07 Sep 2021 15:12:37.910 - DB 0: 2 keys (0 volatile) in 4 slots HT.
  # 1:M 07 Sep 2021 15:12:38.027 * DB saved on disk
```

描述: 在Kubernetes中部署Redis集群很有挑战，因为每个Redis实例都依赖于一个配置文件，该文件跟踪其他集群实例及其角色。为此，我们需要结合使用Kubernetes状态集（StatefulSets）和持久卷（PersistentVolumes）。

**Step 1.文件及其目录说明:**

-   Redis 配置文件: /conf/redis.conf
    
-   集群配置更新文件: /conf/update-node.sh
    
-   集群节点配置文件: /data/nodes.conf
    
-   数据存储目录：/data
    

**Step 2.配置文件准备**  
redis 配置文件的使用方式

-   (1) 方式1.创建 Configmap 来存储该配置文件 `kubectl create configmap redis-conf --from-file=redis.conf`，此处演示采用的方法
    
-   (2) 方式2.利用 Hostpath Volumes 卷挂载该配置文件
    

Redis配置文件示例:

```
# 监听端口
port 6379
# 启用外部连接关闭安全模式
protected-mode no
requirepass weiyigeek.top
# 开启Redis的AOF持久化 && 日志文件
appendonly yes 
appendfilename appendonly.aof 
# AOF持久化文件存在的位置以及其文件名称
dir /data
dbfilename dump.rdb
# 每秒钟同步一次折中的方案
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
# 主从认证及其从节点只读
masterauth weiyigeek.top
slave-read-only yes
# 集群模式打开
cluster-enabled yes 
cluster-config-file  /data/nodes.conf
cluster-node-timeout 5000
# 当负责一个插槽的主库下线且没有相应的从库进行故障恢复时集群仍然可用
cluster-require-full-coverage no
# 只有当一个主节点至少拥有其他给定数量个处于正常工作中的从节点的时候，才会分配从节点给集群中孤立的主节点
cluster-migration-barrier 1
```

**Step 3.Redis-cluster 部署的资源清单**  
描述: 部署清单包括三个部分configMap/Statefulset/Service资源对象清单.

```
cat > Redis-cluster-5.0.10.yaml <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-cluster
  namespace: database
data:
  # 外部命令参数传递执行精妙之处值得学习
  update-node.sh: |
    #!/bin/sh
    REDIS_NODES="/data/nodes.conf"
    if [ ! -f /data/nodes.conf ];then touch /data/nodes.conf;fi
    sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" ${REDIS_NODES}
    exec "$@"
  redis.conf: |+
    # 监听端口
    port 6379
    # 启用外部连接关闭安全模式
    protected-mode no
    masterauth weiyigeek.top
    requirepass weiyigeek.top
    # 开启Redis的AOF持久化 && 日志文件
    appendonly yes 
    appendfilename appendonly.aof 
    # AOF持久化文件存在的位置以及其文件名称
    dir /data
    dbfilename dump.rdb
    slave-read-only yes
    # 每秒钟同步一次折中的方案
    appendfsync everysec
    no-appendfsync-on-rewrite no
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    # 集群模式打开
    cluster-enabled yes 
    cluster-config-file /data/nodes.conf
    cluster-node-timeout 5000
    # 当负责一个插槽的主库下线且没有相应的从库进行故障恢复时集群仍然可用
    cluster-require-full-coverage no
    # 只有当一个主节点至少拥有其他给定数量个处于正常工作中的从节点的时候，才会分配从节点给集群中孤立的主节点
    cluster-migration-barrier 1
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: database
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:5.0.10-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
        - name: timezone
          mountPath: /etc/localtime                # 在Pod中时区设置(挂载主机的时区)
      volumes:
      - name: conf
        configMap:
          name: redis-cluster
          defaultMode: 0755
      - name: timezone 
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "managed-nfs-storage"
      resources:
        requests:
          storage: 5Gi
---
# headless Service
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster
  namespace: database
spec:
  clusterIP: "None"
  ports:
  - port: 6379
    targetPort: 6379
    name: client
  - port: 16379
    targetPort: 16379
    name: gossip
  selector:
    app: redis-cluster
EOF
```

**Step 4.资源清单部署流程与结果查看:**

```
# (1) 名称空间创建
$ kubectl create namespace database

# (2) 按照资源清单StatefulSet控制器创建Pod
$ kubectl create -f Redis-cluster-5.0.10.yaml
  # configmap/redis-cluster created
  # statefulset.apps/redis-cluster created
  # service/redis-cluster created

# (3) 查看应用的 statefulsets,pod,svc 等信息
$ kubectl get statefulsets,pod,svc -n database -o wide --show-labels | grep -v "redis-single"
  # NAME                             READY   AGE     CONTAINERS   IMAGES                LABELS
  # statefulset.apps/redis-cluster   6/6     27m     redis        redis:5.0.10-alpine   <none>

  # NAME                  READY   STATUS    RESTARTS   AGE     IP               NODE      LABELS
  # pod/redis-cluster-0   1/1     Running   0          27m     172.16.183.81    weiyigeek-225  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-0
  # pod/redis-cluster-1   1/1     Running   0          26m     172.16.182.226   weiyigeek-226  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-1
  # pod/redis-cluster-2   1/1     Running   0          26m     172.16.24.204    weiyigeek-223  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-2
  # pod/redis-cluster-3   1/1     Running   0          26m     172.16.100.67    weiyigeek-224  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-3
  # pod/redis-cluster-4   1/1     Running   0          26m     172.16.243.67    weiyigeek-109  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-4
  # pod/redis-cluster-5   1/1     Running   0          26m     172.16.135.195   weiyigeek-108  app=redis-cluster,controller-revision-hash=redis-cluster-6bf876ccf9,statefulset.kubernetes.io/pod-name=redis-cluster-5

  # NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE     SELECTOR            LABELS
  # service/redis-cluster   ClusterIP   None           <none>        6379/TCP,16379/TCP   25m     app=redis-cluster   <none>

# $ 通过svc服务名称访问集群 
ping redis-cluster.database.svc.cluster.local
```

**Step 5.手动进行 Redis 集群配置**

```
# (1) 获取各个pod的IP地址
kubectl get pod -n database -l app=redis-cluster -o jsonpath='{ range.items [*]}{.status.podIP}:6379 '| sed "s# :6379 ##g"
# 172.16.182.219:6379 172.16.183.72:6379 172.16.24.202:6379 172.16.100.65:6379 172.16.135.193:6379 172.16.243.65:6379

# (2) 进入redis-cluster-0 pod中的shell
kubectl exec -n database -it redis-cluster-0 sh 


# (3) 为此让我们运行以下命令并输入'yes'接受配置。我们将看到前三个节点将被选择为主节点，后三个节点将被选择为从节点。
/data $ redis-cli --cluster create --cluster-replicas 1 172.16.182.219:6379 172.16.183.72:6379 172.16.24.202:6379 172.16.100.65:6379 172.16.135.193:6379 172.16.243.65:6379
  # >>> Performing hash slots allocation on 6 nodes...
  # Master[0] -> Slots 0 - 5460
  # Master[1] -> Slots 5461 - 10922
  # Master[2] -> Slots 10923 - 16383
  # Adding replica 172.16.100.65:6379 to 172.16.182.219:6379
  # Adding replica 172.16.135.193:6379 to 172.16.183.72:6379
  # Adding replica 172.16.243.65:6379 to 172.16.24.202:6379
  # M: be20868bb9de5688d50c54d2654ffe1f11c28794 172.16.182.219:6379
  #    slots:[0-5460] (5461 slots) master
  # M: ba71016f951653985ff1ad42528f4c09bcf51ddb 172.16.183.72:6379
  #    slots:[5461-10922] (5462 slots) master
  # M: fb2f078d21d1efbcf93dc43cbead6aff9ea9eb76 172.16.24.202:6379
  #    slots:[10923-16383] (5461 slots) master
  # S: 17244c7a73416380f9ed254ee9d8b1b56836e0a0 172.16.100.65:6379
  #    replicates be20868bb9de5688d50c54d2654ffe1f11c28794
  # S: 677b964d754148db24e953bdd4e08de3a89c4eed 172.16.135.193:6379
  #    replicates ba71016f951653985ff1ad42528f4c09bcf51ddb
  # S: 200f775a8e0f83b9e39cd1bf597d65aba53d5f26 172.16.243.65:6379
  #    replicates fb2f078d21d1efbcf93dc43cbead6aff9ea9eb76
  # Can I set the above configuration? (type 'yes' to accept): yes
  # >>> Nodes configuration updated
  # >>> Assign a different config epoch to each node
  # >>> Sending CLUSTER MEET messages to join the cluster
  # Waiting for the cluster to join
  # ....
  # >>> Performing Cluster Check (using node 172.16.182.219:6379)
  # M: be20868bb9de5688d50c54d2654ffe1f11c28794 172.16.182.219:6379
  #    slots:[0-5460] (5461 slots) master
  #    1 additional replica(s)
  # M: fb2f078d21d1efbcf93dc43cbead6aff9ea9eb76 172.16.24.202:6379
  #    slots:[10923-16383] (5461 slots) master
  #    1 additional replica(s)
  # S: 677b964d754148db24e953bdd4e08de3a89c4eed 172.16.135.193:6379
  #    slots: (0 slots) slave
  #    replicates ba71016f951653985ff1ad42528f4c09bcf51ddb
  # M: ba71016f951653985ff1ad42528f4c09bcf51ddb 172.16.183.72:6379
  #    slots:[5461-10922] (5462 slots) master
  #    1 additional replica(s)
  # S: 17244c7a73416380f9ed254ee9d8b1b56836e0a0 172.16.100.65:6379
  #    slots: (0 slots) slave
  #    replicates be20868bb9de5688d50c54d2654ffe1f11c28794
  # S: 200f775a8e0f83b9e39cd1bf597d65aba53d5f26 172.16.243.65:6379
  #    slots: (0 slots) slave
  #    replicates fb2f078d21d1efbcf93dc43cbead6aff9ea9eb76
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [OK] All 16384 slots covered.
```

Tips: 也可以利用下面两条命令一步到位的方式:

```
# 方式1
export REDIS_POD_IP=$(kubectl get pod -n database -l app=redis-cluster -o jsonpath='{ range.items [*]}{.status.podIP}:6379 '| sed "s# :6379 ##g")
kubectl exec -it -n database redis-cluster-0 -- sh -c "/usr/local/bin/redis-cli -a weiyigeek.top --cluster create --cluster-replicas 1 ${REDIS_POD_IP}"

# 方式2
kubectl -n database exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -n database -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '| sed "s# :6379 ##g")
```

Tips: 在集群中我们可以利用SVC服务发现的机制采用 `redis-cluster-1.redis-cluster.database.svc.cluster.local` 域名的方式访问Pod;

**Step 6.Redis 集群访问验证**

```
# (1) 先以单台为例查看集群状态
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -a weiyigeek.top ping"
  # PONG

# (2) 集群信息 
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -a weiyigeek.top cluster info"
  # cluster_state:ok
  # cluster_slots_assigned:16384
  # cluster_slots_ok:16384
  # cluster_slots_pfail:0
  # cluster_slots_fail:0
  # cluster_known_nodes:6
  # cluster_size:3
  # cluster_current_epoch:6
  # cluster_my_epoch:1
  # cluster_stats_messages_ping_sent:1735
  # cluster_stats_messages_pong_sent:1797
  # cluster_stats_messages_sent:3532
  # cluster_stats_messages_ping_received:1792
  # cluster_stats_messages_pong_received:1735
  # cluster_stats_messages_meet_received:5
  # cluster_stats_messages_received:3532

# (3) 当前节点角色
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -a weiyigeek.top role"
  # 1) "master"  # 当前主机角色
  # 2) (integer) 1288
  # 3) 1) 1) "172.16.243.67"  # Slave IP
  #       2) "6379"
  #       3) "1288"

# (4) 从节点信息
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -a weiyigeek.top info replication"
# Replication
  # role:master
  # connected_slaves:1
  # slave0:ip=172.16.243.67,port=6379,state=online,offset=1372,lag=0
  # master_replid:222f4d18c5d0913cb367d8bff49edc717563a96d
  # master_replid2:0000000000000000000000000000000000000000
  # master_repl_offset:1372
  # second_repl_offset:-1
  # repl_backlog_active:1
  # repl_backlog_size:1048576
  # repl_backlog_first_byte_offset:1
  # repl_backlog_histlen:1372

# (5) 列举出集群 nodes 相关信息以及卡槽信息
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -a weiyigeek.top cluster nodes"
  # ee0aa0c0742e6a9362a96f182d54483ec065fdba 172.16.182.226:6379@16379 master - 0 1611220585017 2 connected 5461-10922
  # 3324007374edeaf4bed02423148970ee8171a7cb 172.16.135.195:6379@16379 slave ee0aa0c0742e6a9362a96f182d54483ec065fdba 0 1611220583514 6 connected
  # f5a740fcbfdcd398da6e380f496fb5cb92236748 172.16.183.81:6379@16379 myself,master - 0 1611220583000 1 connected 0-5460
  # 7697cf77b6da8b58663d23e9ad8150f33a6ee9cd 172.16.24.204:6379@16379 master - 0 1611220583515 3 connected 10923-16383
  # 99509917de27b2f9ba02fb1b09d137014a0ae5f9 172.16.100.67:6379@16379 slave 7697cf77b6da8b58663d23e9ad8150f33a6ee9cd 0 1611220585016 4 connected
  # b5aa6a5f06f1ad2b9debccc6842bcc5f0d9ff732 172.16.243.67:6379@16379 slave f5a740fcbfdcd398da6e380f496fb5cb92236748 0 1611220584015 5 connected
```

**Step 7.查看Redis 集群各工作节点状态**

```
# (1) 一个脚本全部搞定,对于/conf/update-node.sh的脚本此处也显化出其功能.
for pod_name in $(kubectl get pod -n database -l app=redis-cluster -o jsonpath='{ range.items [*]}{.spec.hostname} ');do
  echo ${pod_name}
  kubectl exec -it -n database ${pod_name} -- sh -c "redis-cli -a WeiyiGeek.com.cn cluster nodes" | grep "myself";
  kubectl exec -it -n database ${pod_name} -- sh -c "redis-cli -a WeiyiGeek.com.cn info replication" | egrep "role|slave"
  echo .
done

# (2) 当Pod重启后会又一个新的IP地址,此时通过update-node.sh脚本, 进行将原IP地址替换为新POD的IP地址
# 前三个为主-Master
# f5a740fcbfdcd398da6e380f496fb5cb92236748 172.16.183.81:6379@16379 myself,master - 0 1611221210000 1 connected 0-5460  # 注意槽的分配端
role:master
connected_slaves:1
slave0:ip=172.16.243.67,port=6379,state=online,offset=2436,lag=1

# ee0aa0c0742e6a9362a96f182d54483ec065fdba 172.16.182.226:6379@16379 myself,master - 0 1611221211000 2 connected 5461-10922
role:master
connected_slaves:1
slave0:ip=172.16.135.195,port=6379,state=online,offset=2436,lag=0

# 7697cf77b6da8b58663d23e9ad8150f33a6ee9cd 172.16.24.204:6379@16379 myself,master - 0 1611221209000 3 connected 10923-16383
role:master
connected_slaves:1
slave0:ip=172.16.100.67,port=6379,state=online,offset=2436,lag=0

# 后三个为从-Slave
# 99509917de27b2f9ba02fb1b09d137014a0ae5f9 172.16.100.67:6379@16379 myself,slave 7697cf77b6da8b58663d23e9ad8150f33a6ee9cd 0 1611221211000 4 connected
role:slave
slave_repl_offset:2436
slave_priority:100
slave_read_only:1
connected_slaves:0

# b5aa6a5f06f1ad2b9debccc6842bcc5f0d9ff732 172.16.243.67:6379@16379 myself,slave f5a740fcbfdcd398da6e380f496fb5cb92236748 0 1611221211000 5 connected
role:slave
slave_repl_offset:2436
slave_priority:100
slave_read_only:1
connected_slaves:0

# 3324007374edeaf4bed02423148970ee8171a7cb 172.16.135.195:6379@16379 myself,slave ee0aa0c0742e6a9362a96f182d54483ec065fdba 0 1611221209000 6 connected
role:slave
slave_repl_offset:2436
slave_priority:100
slave_read_only:1
connected_slaves:0
```

**Step 8.Redis 集群之数据写入和查询实践**

```
~/k8s/redis-master-slave$ kubectl exec -it -n database redis-cluster-0 -- sh -c "redis-cli -h redis-cluster-0 -c -a weiyigeek.top"
redis-cluster-0:6379> set name weiyigeek
-> Redirected to slot [5798] located at 172.16.182.226:6379   # 存储到 172.16.182.226 Master 节点中
OK
172.16.182.226:6379> set demo Redis-Cluster                   # 存储到 172.16.183.81 Master 节点中
-> Redirected to slot [903] located at 172.16.183.81:6379
OK
172.16.183.81:6379> set web www.weiyigeek.top
-> Redirected to slot [9635] located at 172.16.182.226:6379   # 存储到 172.16.182.226 Master 节点中
OK
```

Tips: 可以看到写入的数据根据分配的数据槽,分别存储在redis集群中各个对应的Master节点上.

**Step 9.利用redis-cli客户端工具连接查看集群**  
描述: 有时你的主机里面没有redis-tools相关工具时,并且在k8s中搭建的redis集群,如果想通过一些可视化的工具访问时,必须要进行代理访问,所以说非常的不方便,为了方便测试与调试,我们可以在k8s中部署带又redis-cli相关工具的容器.

```
tee redis-cli.yml<<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: redis-cli
  labels:
    app: redis-cli
spec:
  type: ClusterIP
  ports:
    - name: redis-cli
      port: 8080
  selector:
    app: redis-cli
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cli
  namespace: monitor
  labels:
    app: redis-cli
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-cli
  template:
    metadata:
      labels:
        app: redis-cli
    spec:
      containers:
        - name: redis-cli
          image: redis:6.2.4-alpine
          command:
            - "top"
            - "-d"
            - "360"
          resources:
            limits:
              cpu: 1000m
              memory: 1024Mi
EOF

# 部署
kubectl create --save-config -f redis-cli.yml

# 查看Pod状态
kubectl get pod -n monitor -l app=redis-cli -o wide
  NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
  redis-cli-5f786d69f-nckbt   1/1     Running   0          75d   172.19.6.14   aiserver   <none>           <none>

# 使用其连接redis集群
kubectl  -n monitor exec -it redis-cli-5f786d69f-nckbt -- sh -c "redis-cli -h redis-cluster.database.svc.cluster.local -c -a weiyigeek.top"
```

**总结:**  
Redis是一个强大的数据存储和缓存工具。因为Redis存储数据的方式，Redis集群更是能通过提供分片、相关性能优势、线性扩展和高可用性，来进一步扩展其功能。数据在多个节点之间自动分割，即使节点的子集出现故障或者不能和集群其他部分通信，操作仍然能够继续。

**3.1应用连接写入测试**  
描述: 我编写了一个java应用来测试数据的写入,以下是资源清单以及实践结果.

```
# - 配置清单
cat > redis-cluster-connect.yaml <<'END'
apiVersion: v1 
kind: Pod 
metadata:
 name: redis-test
 namespace: default 
 labels:
  app: redis-test
spec:
  containers:
  - name: redis-test
    image: java:8
    imagePullPolicy: IfNotPresent
    command:
    - "java"
    - "-jar"
    - "/app/redis.jar"
    volumeMounts:
      - name: app
        mountPath: /app
      - name: timezone
        mountPath: /etc/localtime       # 在Pod中时区设置(挂载主机的时区)
  volumes:
    - name: app
      hostPath:
        path: /nfsdisk-31/test
    - name: timezone 
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai 
END

# - 部署配置
kubectl create -f redis-cluster-connect.yaml
# - 运行状态
kubectl get pod -l app=redis-test -o wide
  # NAME         READY   STATUS    RESTARTS   AGE     IP              NODE   
  # redis-test   1/1     Running   0          5m57s   172.16.183.76   WeiyiGeek-225
# - 运行日志
kubectl logs redis-test  | more
# 测试输出
  # host:redis-test
  # ip:172.16.183.76
  # # 写入了10W的数据
  # # Set Keys start ------ Wed Sep 08 07:59:24 UTC 2021  
  # # Set Keys end ------ Wed Sep 08 08:05:27 UTC 2021
  # # Get Keys start ------
  # Wed Sep 08 08:05:27 UTC 2021
  # 0 - clusterredis-test1631087965024 : Ip:172.16.183.76 - Time:Wed Sep 08 07:59:25 UTC 2021
  # 1 - clusterredis-test1631087965033 : Ip:172.16.183.76 - Time:Wed Sep 08 07:59:25 UTC 2021
  # 2 - clusterredis-test1631087965035 : Ip:172.16.183.76 - Time:Wed Sep 08 07:59:25 UTC 2021


# 库中Keys总量
redis-cluster.database.svc.cluster.local:6379> info keyspace
  # # Keyspace
  # db0:keys=200000,expires=0,avg_ttl=0

# 通过通配符匹配keys名称
redis-cluster.database.svc.cluster.local:6379> keys clusterredis-test16310880576*
  # 1) "clusterredis-test1631088057679"
  # 2) "clusterredis-test1631088057605"
  # 3) "clusterredis-test1631088057622"
  # 4) "clusterredis-test1631088057644"

# 获取键值
redis-cli -h redis-cluster.database.svc.cluster.local -c -a weiyigeek.top
redis-cluster.database.svc.cluster.local:6379> get "clusterredis-test1631088057644"
"Ip:172.16.183.76 - Time:Wed Sep 08 08:00:57 UTC 2021"
redis-cluster.database.svc.cluster.local:6379> get "clusterredis-test1631088057645"
-> Redirected to slot [8995] located at 172.16.24.194:6379
"Ip:172.16.183.76 - Time:Wed Sep 08 08:00:57 UTC 2021"
172.16.24.194:6379>
```

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>         如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！            

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。】