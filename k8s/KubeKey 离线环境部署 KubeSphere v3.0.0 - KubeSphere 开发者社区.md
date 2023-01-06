### 环境准备

以三台centos 7.7 64bit 为例：

| name | ip | role |
| --- | --- | --- |
| node1 | 192.168.6.17 | etcd, master, worker |
| node2 | 192.168.6.18 | worker |
| node3 | 192.168.6.19 | worker |

确保机器已经安装所需依赖软件（sudo curl openssl ebtables socat ipset conntrack docker），离线环境下可使用私有源或者rpm包(centos类OS)或deb包(debian类OS)安装。

具体环境要求参见：[https://github.com/kubesphere/kubekey/tree/release-1.0#requirements-and-recommendations](https://github.com/kubesphere/kubekey/tree/release-1.0#requirements-and-recommendations)

> 建议：可将安装了所有依赖软件的操作系统制作成系统镜像使用，避免每台机器都安装依赖软件，即可提升交付部署效率，又可避免依赖问题的发生。

### 镜像仓库

可使用harbor或其他第三方镜像仓库。

如需快速部署自签名镜像仓库可参考：[https://kubesphere.com.cn/forum/d/2240-docker-registry](https://kubesphere.com.cn/forum/d/2240-docker-registry)

### 安装包下载：

> 提示：由于包含所有组件镜像，该压缩包较大，如果网络不佳，可能会导致下载耗时较长。也可根据文档中的镜像列表将相关镜像导入私有镜像仓库中后使用kubekey自行安装。

```
# md5: 65e9a1158a682412faa1166c0cf06772
curl -Ok https://kubesphere-installer.pek3b.qingstor.com/offline/v3.0.0/kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz
```

### 安装步骤：

#### 1\. 创建集群配置文件

安装包解压后进入`kubesphere-all-v3.0.0-offline-linux-amd64`  
[![](https://kubesphere.com.cn/forum/assets/files/2020-09-21/1600699818-393809-20200921224932.png)](https://kubesphere.com.cn/forum/assets/files/2020-09-21/1600699818-393809-20200921224932.png)

```
./kk create config --with-kubesphere v3.0.0
```

修改生成的配置文件`config-sample.yaml`，也可使用-f参数自定义配置文件路径。kk详细用法可参考：[https://github.com/kubesphere/kubekey](https://github.com/kubesphere/kubekey)

> 注意填写正确的私有仓库地址`privateRegistry`（如已准备好私有仓库可设置为已有仓库地址，若计划使用kubekey创建私有仓库，则该参数设置为：dockerhub.kubekey.local）

```
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: node1, address: 192.168.6.17, internalAddress: 192.168.6.17, password: Qcloud@123}
  - {name: node2, address: 192.168.6.18, internalAddress: 192.168.6.18, password: Qcloud@123}
  - {name: node3, address: 192.168.6.19, internalAddress: 192.168.6.19, password: Qcloud@123}
  roleGroups:
    etcd:
    - node1
    master:
    - node1
    worker:
    - node1
    - node2
    - node3
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: ""
    port: "6443"
  kubernetes:
    version: v1.17.9
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
    privateRegistry: dockerhub.kubekey.local
  addons: []

---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.0.0
spec:
  local_registry: ""
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  etcd:
    monitoring: true
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    es:
      elasticsearchDataVolumeSize: 20Gi
      elasticsearchMasterVolumeSize: 4Gi
      elkPrefix: logstash
      logMaxAge: 7
    mysqlVolumeSize: 20Gi
    minioVolumeSize: 20Gi
    etcdVolumeSize: 20Gi
    openldapVolumeSize: 2Gi
    redisVolumSize: 2Gi
  console:
    enableMultiLogin: false  # enable/disable multi login
    port: 30880
  alerting:
    enabled: false
  auditing:
    enabled: false
  devops:
    enabled: false
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsJavaOpts_MaxRAM: 2g
  events:
    enabled: false
    ruler:
      enabled: true
      replicas: 2
  logging:
    enabled: false
    logsidecarReplicas: 2
  metrics_server:
    enabled: true
  monitoring:
    prometheusMemoryRequest: 400Mi
    prometheusVolumeSize: 20Gi
  multicluster:
    clusterRole: none  # host | member | none
  networkpolicy:
    enabled: false
  notification:
    enabled: false
  openpitrix:
    enabled: false
  servicemesh:
    enabled: false
```

#### 2\. 环境初始化 （可选）

> 若已安装相关依赖，并且已经准备好镜像仓库，可略过该步骤。 (为避免依赖问题的产生，建议提前安装相关依赖或使用已安装相关依赖的系统镜像执行安装)

> 注意：如需使用kk创建自签名镜像仓库，则会在当前机器启动docker registry服务，请确保当前机器存在`registry:2`，如没有，可从kubesphere-images-v3.0.0/registry.tar中导入，导入命令：`docker load < registry.tar`

> 注意：由kk启动的镜像仓库端口为443，请确保所有机器均可访问当前机器443端口。镜像数据存储到本地/mnt/registry (建议单独挂盘)。

> `dependencies`目录中仅提供了ubuntu16.04 (ubuntu-16.04-amd64-debs.tar.gz)、ubuntu18.04 (ubuntu-18.04-amd64-debs.tar.gz)以及centos7 (centos-7-amd64-rpms.tar.gz)的相关依赖包，其它操作系统可自行制作rpm或deb依赖包。打包规则为 ${releaseID}-${versionID}-${osArch}-${debs or rpms}.tar.gz

```
# 执行如下命令会对配置文件中所有节点安装依赖：
./kk init os -f config-sample.yaml -s ./dependencies/

# 如需使用kk创建自签名镜像仓库，可执行如下命令：
./kk init os -f config-sample.yaml -s ./dependencies/ --add-images-repo
```

#### 3\. 导入镜像

进入`kubesphere-all-v3.0.0-offline-linux-amd64/kubesphere-images-v3.0.0`

[![](https://kubesphere.com.cn/forum/assets/files/2020-09-21/1600699932-276321-20200921225125.png)](https://kubesphere.com.cn/forum/assets/files/2020-09-21/1600699932-276321-20200921225125.png)

使用push-images.sh将镜像导入之前准备的仓库中：

```
# 脚本后镜像仓库地址请填写真实仓库地址
./push-images.sh  dockerhub.kubekey.local
```

如需自行到入镜像，可参考如下方法：  
以kubesphere/kube-apiserver:v1.17.9为例  
`docker tag kubesphere/kube-apiserver:v1.17.9 dockerhub.kubesphere.local/kubesphere/kube-apiserver:v1.17.9`

> 注意: retag镜像时需要保留原始镜像的namespace

### 部署

以上准备工作完成且再次检查配置文件无误后，执行安装。

#### 执行安装

```
./kk create cluster -f config-sample.yaml
```

### 附： 镜像列表

```
k8s：
kubesphere/kube-apiserver:v1.17.9
kubesphere/kube-scheduler:v1.17.9
kubesphere/kube-proxy:v1.17.9
kubesphere/kube-controller-manager:v1.17.9
kubesphere/pause:3.1
kubesphere/pause:3.2  (k8s版本大于v1.18)
kubesphere/etcd:v3.3.12
calico/kube-controllers:v3.15.1
calico/node:v3.15.1
calico/cni:v3.15.1
calico/pod2daemon-flexvol:v3.15.1
coredns/coredns:1.6.9
kubesphere/k8s-dns-node-cache:1.15.12

localVolume：
kubesphere/node-disk-manager:0.5.0
kubesphere/node-disk-operator:0.5.0
kubesphere/provisioner-localpv:1.10.0
kubesphere/linux-utils:1.10.0

kubesphere：
kubesphere/ks-apiserver:v3.0.0
kubesphere/ks-console:v3.0.0
kubesphere/ks-controller-manager:v3.0.0
kubesphere/ks-installer:v3.0.0
kubesphere/etcd:v3.2.18
kubesphere/kubectl:v1.0.0
kubesphere/ks-upgrade:v3.0.0
kubesphere/ks-devops:flyway-v3.0.0
redis:5.0.5-alpine
alpine:3.10.4
haproxy:2.0.4
mysql:8.0.11
nginx:1.14-alpine
minio/minio:RELEASE.2019-08-07T01-59-21Z
minio/mc:RELEASE.2019-08-07T23-14-43Z
mirrorgooglecontainers/defaultbackend-amd64:1.4
kubesphere/nginx-ingress-controller:0.24.1
osixia/openldap:1.3.0
csiplugin/snapshot-controller:v2.0.1
kubesphere/ks-upgrade:v3.0.0                      
kubesphere/ks-devops:flyway-v3.0.0

monitoring：
kubesphere/prometheus-config-reloader:v0.38.3
kubesphere/prometheus-operator:v0.38.3
prom/alertmanager:v0.21.0
prom/prometheus:v2.20.1
kubesphere/node-exporter:ks-v0.18.1
jimmidyson/configmap-reload:v0.3.0
kubesphere/notification-manager-operator:v0.1.0
kubesphere/notification-manager:v0.1.0
kubesphere/metrics-server:v0.3.7
kubesphere/kube-rbac-proxy:v0.4.1
kubesphere/kube-state-metrics:v1.9.6

logging:
kubesphere/elasticsearch-oss:6.7.0-1
kubesphere/elasticsearch-curator:v5.7.6
kubesphere/fluentbit-operator:v0.2.0
kubesphere/fluentbit-operator:migrator
kubesphere/fluent-bit:v1.4.6    
kubesphere/kube-auditing-operator:v0.1.0
kubesphere/kube-auditing-webhook:v0.1.0
kubesphere/kube-events-exporter:v0.1.0
kubesphere/kube-events-operator:v0.1.0
kubesphere/kube-events-ruler:v0.1.0
kubesphere/log-sidecar-injector:1.1
docker:19.03
service-mesh：
istio/citadel:1.4.8
istio/galley:1.4.8
istio/kubectl:1.4.8
istio/mixer:1.4.8
istio/pilot:1.4.8
istio/proxyv2:1.4.8
istio/sidecar_injector:1.4.8
jaegertracing/jaeger-agent:1.17
jaegertracing/jaeger-collector:1.17
jaegertracing/jaeger-operator:1.17.1
jaegertracing/jaeger-query:1.17
jaegertracing/jaeger-es-index-cleaner:1.17.1

devops：
jenkins/jenkins:2.176.2
jenkins/jnlp-slave:3.27-1
kubesphere/jenkins-uc:v3.0.0
kubesphere/s2ioperator:v2.1.1
kubesphere/s2irun:v2.1.1
kubesphere/builder-base:v2.1.0
kubesphere/builder-nodejs:v2.1.0
kubesphere/builder-maven:v2.1.0
kubesphere/builder-go:v2.1.0
kubesphere/s2i-binary:v2.1.0
kubesphere/tomcat85-java11-centos7:v2.1.0
kubesphere/tomcat85-java11-runtime:v2.1.0
kubesphere/tomcat85-java8-centos7:v2.1.0
kubesphere/tomcat85-java8-runtime:v2.1.0
kubesphere/java-11-centos7:v2.1.0
kubesphere/java-8-centos7:v2.1.0
kubesphere/java-8-runtime:v2.1.0
kubesphere/java-11-runtime:v2.1.0
kubesphere/nodejs-8-centos7:v2.1.0
kubesphere/nodejs-6-centos7:v2.1.0
kubesphere/nodejs-4-centos7:v2.1.0
kubesphere/python-36-centos7:v2.1.0
kubesphere/python-35-centos7:v2.1.0
kubesphere/python-34-centos7:v2.1.0
kubesphere/python-27-centos7:v2.1.0

notification&&alerting:
kubesphere/notification:flyway_v2.1.2
kubesphere/notification:v2.1.2   
kubesphere/alert-adapter:v3.0.0
kubesphere/alerting-dbinit:v3.0.0
kubesphere/alerting:v2.1.2

multicluster:
kubesphere/kubefed:v0.3.0
kubesphere/tower:v0.1.0

openpitrix(app store)：
openpitrix/generate-kubeconfig:v0.5.0
openpitrix/openpitrix:flyway-v0.5.0
openpitrix/openpitrix:v0.5.0
openpitrix/release-app:v0.5.0
```