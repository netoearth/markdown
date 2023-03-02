## 前言

上一篇主要讲了如何进行单机版本kubesphere，本篇主要讲如何基于单机镜像完成集群的配置与管理。

## 一、导出镜像

> 以下操作必须要在之前的单机上执行，不然没效果。

```
#创建配置文件
./kk create manifest
```

**查看配置文件**  
注意这里的harbor前面的注释一定要记得关闭  
尽量把配置写全一点，这样内部后续包就包含进来了。  
当然 mainfest.yaml，如果太大就会导致打包的文件太大。  
所以我们给出两份demo文件，mainfest\_mini.yaml，mainfest\_full.yaml

```
# 把文件拷贝一份
cp mainfest_simple.yaml mainfest.yaml
```

最小版本

```
vim mainfest_mini.yaml
文件开始>>>>>

apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Manifest
metadata:
  name: sample
spec:
  arches:
  - amd64
  operatingSystems:
  - arch: amd64
    type: linux
    id: centos
    version: "7"
    osImage: CentOS Linux 7 (Core)
    repository:
      iso:
        localPath:
        url: "https://github.com/kubesphere/kubekey/releases/download/v2.0.0/centos-7-amd64-rpms.iso"
  kubernetesDistributions:
  - type: kubernetes
    version: v1.21.5
  components:
    helm:
      version: v3.6.3
    cni:
      version: v0.9.1
    etcd:
      version: v3.4.13
    ## For now, if your cluster container runtime is containerd, KubeKey will add a docker 20.10.8 container runtime in the below list.
    ## The reason is KubeKey creates a cluster with containerd by installing a docker first and making kubelet connect the socket file of containerd which docker contained.
    containerRuntimes:
    - type: docker
      version: 20.10.8
    crictl:
      version: v1.22.0

    docker-registry:
      version: "2"
    harbor:
      version: v2.4.1
    docker-compose:
      version: v2.2.2
  images:
  - docker.io/calico/cni:v3.20.0
  - docker.io/calico/kube-controllers:v3.20.0
  - docker.io/calico/node:v3.20.0
  - docker.io/calico/pod2daemon-flexvol:v3.20.0
  - docker.io/coredns/coredns:1.8.0
  - docker.io/csiplugin/snapshot-controller:v4.0.0
  - docker.io/kubesphere/k8s-dns-node-cache:1.15.12
  - docker.io/kubesphere/ks-apiserver:v3.2.1
  - docker.io/kubesphere/ks-console:v3.2.1
  - docker.io/kubesphere/ks-controller-manager:v3.2.1
  - docker.io/kubesphere/ks-installer:v3.2.1
  - docker.io/kubesphere/kube-apiserver:v1.21.5
  - docker.io/kubesphere/kube-controller-manager:v1.21.5
  - docker.io/kubesphere/kube-proxy:v1.21.5
  - docker.io/kubesphere/kube-rbac-proxy:v0.8.0
  - docker.io/kubesphere/kube-scheduler:v1.21.5
  - docker.io/kubesphere/kube-state-metrics:v1.9.7
  - docker.io/kubesphere/kubectl:v1.21.0
  - docker.io/kubesphere/notification-manager-operator:v1.4.0
  - docker.io/kubesphere/notification-manager:v1.4.0
  - docker.io/kubesphere/notification-tenant-sidecar:v3.2.0
  - docker.io/kubesphere/pause:3.4.1
  - docker.io/kubesphere/prometheus-config-reloader:v0.43.2
  - docker.io/kubesphere/prometheus-operator:v0.43.2
  - docker.io/mirrorgooglecontainers/defaultbackend-amd64:1.4
  - docker.io/openebs/provisioner-localpv:2.10.1
  - docker.io/prom/alertmanager:v0.21.0
  - docker.io/prom/node-exporter:v0.18.1
  - docker.io/prom/prometheus:v2.26.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/coredns:1.8.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-apiserver:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controller-manager:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-proxy:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-scheduler:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pause:3.4.1
  registry:
    auths: {}
```

最大版本

```
mainfest_full.yaml

文件开始>>>>>
---
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Manifest
metadata:
  name: sample
spec:
  arches:
  - amd64
  operatingSystems:
  - arch: amd64
    type: linux
    id: centos
    version: "7"
    repository:
      iso:
        localPath: ""
        url: "https://github.com/kubesphere/kubekey/releases/download/v2.0.0/centos-7-amd64-rpms.iso"
  kubernetesDistributions:
  - type: kubernetes
    version: v1.21.5
  components:
    helm:
      version: v3.6.3
    cni:
      version: v0.9.1
    etcd:
      version: v3.4.13
    ## For now, if your cluster container runtime is containerd, KubeKey will add a docker 20.10.8 container runtime in the below list.
    ## The reason is KubeKey creates a cluster with containerd by installing a docker first and making kubelet connect the socket file of containerd which docker contained.
    containerRuntimes:
    - type: docker
      version: 20.10.8
    crictl:
      version: v1.22.0
    ##
    # docker-registry:
    #   version: "2"
    harbor:
      version: v2.4.1
    docker-compose:
      version: v2.2.2
  images:
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-apiserver:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controller-manager:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-proxy:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-scheduler:v1.21.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pause:3.5
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pause:3.4.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/coredns:1.8.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/cni:v3.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controllers:v3.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/node:v3.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pod2daemon-flexvol:v3.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/typha:v3.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/flannel:v0.12.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/provisioner-localpv:2.10.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/linux-utils:2.10.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/haproxy:2.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nfs-subdir-external-provisioner:v4.0.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/k8s-dns-node-cache:1.15.12
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-installer:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-apiserver:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-console:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-controller-manager:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kubectl:v1.21.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kubectl:v1.20.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kubefed:v0.8.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tower:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/minio:RELEASE.2019-08-07T01-59-21Z
  - registry.cn-beijing.aliyuncs.com/kubesphereio/mc:RELEASE.2019-08-07T23-14-43Z
  - registry.cn-beijing.aliyuncs.com/kubesphereio/snapshot-controller:v4.0.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nginx-ingress-controller:v0.48.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/defaultbackend-amd64:1.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/metrics-server:v0.4.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/redis:5.0.14-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/haproxy:2.0.25-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/alpine:3.14
  - registry.cn-beijing.aliyuncs.com/kubesphereio/openldap:1.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/netshoot:v1.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/cloudcore:v1.7.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/edge-watcher:v0.1.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/edge-watcher-agent:v0.1.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/gatekeeper:v3.5.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/openpitrix-jobs:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-apiserver:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-controller:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-tools:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-jenkins:v3.2.0-2.249.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jnlp-slave:3.27-1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-base:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-nodejs:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-maven:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-python:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-go:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-go:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-base:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-nodejs:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-maven:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-python:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-go:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-go:v3.2.0-podman
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2ioperator:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2irun:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2i-binary:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tomcat85-java11-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tomcat85-java11-runtime:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tomcat85-java8-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tomcat85-java8-runtime:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-11-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-8-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-8-runtime:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-11-runtime:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nodejs-8-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nodejs-6-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nodejs-4-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/python-36-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/python-35-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/python-34-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/python-27-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/configmap-reload:v0.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus:v2.26.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-config-reloader:v0.43.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-operator:v0.43.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-rbac-proxy:v0.8.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-state-metrics:v1.9.7
  - registry.cn-beijing.aliyuncs.com/kubesphereio/node-exporter:v0.18.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/k8s-prometheus-adapter-amd64:v0.6.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/alertmanager:v0.21.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/thanos:v0.18.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/grafana:7.4.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-rbac-proxy:v0.8.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager-operator:v1.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager:v1.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-tenant-sidecar:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/elasticsearch-curator:v5.7.6
  - registry.cn-beijing.aliyuncs.com/kubesphereio/elasticsearch-oss:6.7.0-1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/fluentbit-operator:v0.11.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/docker:19.03
  - registry.cn-beijing.aliyuncs.com/kubesphereio/fluent-bit:v1.8.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/log-sidecar-injector:1.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/filebeat:6.7.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-operator:v0.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-exporter:v0.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-ruler:v0.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-auditing-operator:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-auditing-webhook:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pilot:1.11.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/proxyv2:1.11.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-operator:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-agent:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-collector:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-query:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-es-index-cleaner:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kiali-operator:v1.38.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kiali:v1.38
  - registry.cn-beijing.aliyuncs.com/kubesphereio/busybox:1.31.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nginx:1.14-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/wget:1.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/hello:plain-text
  - registry.cn-beijing.aliyuncs.com/kubesphereio/wordpress:4.8-apache
  - registry.cn-beijing.aliyuncs.com/kubesphereio/hpa-example:latest
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java:openjdk-8-jre-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/fluentd:v1.4.2-2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/perl:latest
  - registry.cn-beijing.aliyuncs.com/kubesphereio/examples-bookinfo-productpage-v1:1.16.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/examples-bookinfo-reviews-v1:1.16.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/examples-bookinfo-reviews-v2:1.16.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/examples-bookinfo-details-v1:1.16.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/examples-bookinfo-ratings-v1:1.16.3
  registry:
    auths: {}
<<<<<文件结束

#导出镜像, 不知道为啥我操作了两次。
#导出文件大小及时间取决于我们配置的文件多少。
./kk artifact export -m manifest.yaml -o kubesphere.tar.gz
#最后成功执行如下
13:48:20 CST success: [LocalHost]
13:48:20 CST [ChownOutputModule] Chown output file
13:48:20 CST success: [LocalHost]
13:48:20 CST [ChownWorkerModule] Chown ./kubekey dir
13:48:20 CST success: [LocalHost]
13:48:20 CST Pipeline[ArtifactExportPipeline] execute successful
[root@localhost ~]# ll
# kubesphere.tar.gz 就是我们打的镜像文件了
```

## 二、导入镜像

目标：我们期望在部署集群是无[外网](https://so.csdn.net/so/search?q=%E5%A4%96%E7%BD%91&spm=1001.2101.3001.7020)的情况下依然可以执行集群的安装。

## 环境准备

三台测试机器

| role | ip | hostname |
| --- | --- | --- |
| master | 192.168.3.65 | kube\_master01 |
| master | 192.168.3.66 | kube\_master02 |
| master | 192.168.3.67 | kube\_master03 |
| node | 192.168.3.68 | kube\_node1 |
| node | 192.168.3.69 | kube\_node2 |

master上需要安装 etcd 高可用集群，这样我用任意一台就可以管理我们的机器了，实际管理时我们可以在任意一台机器中操作kubectl即可，因为它们都是向高可用集群etcd数据库中写入数据，然后再完成schedule调度任务。

拷贝文件到kube\_master中

```
scp kk root@192.168.3.65:/root
scp kubesphere.tar.gz root@192.168.3.65:/root
```

## 执行操作

```
ssh root@192.168.3.65
##创建配置文件
./kk create config --with-kubesphere v3.2.1 --with-kubernetes v1.21.5 -f config-sample.yaml
##修改配置文件设置registry，注意设置hosts/registry
cp config-sample.yaml  config.yaml
cat config.yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: kube_master01, address: 192.168.3.65, internalAddress: 192.168.3.65, user: root, password: "123456"}
  - {name: kube_master02, address: 192.168.3.66, internalAddress: 192.168.3.66, user: root, password: "123456"}
  - {name: kube_master03, address: 192.168.3.67, internalAddress: 192.168.3.67, user: root, password: "123456"}
  - {name: kube_node01, address: 192.168.3.68, internalAddress: 192.168.3.68, user: root, password: "123456"}
  - {name: kube_node02, address: 192.168.3.69, internalAddress: 192.168.3.69, user: root, password: "123456"}
  roleGroups:
    etcd:
    - kube_master01
    - kube_master02
    - kube_master03
    control-plane:
    - kube_master01
    - kube_master02
    - kube_master03
    worker:
    - kube_node01
    - kube_node02
    registry:
    - kube_master02
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers
    internalLoadbalancer: haproxy
    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.21.5
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    type: harbor
    auths:
      "dockerhub.kubekey.local":
        username: admin
        password: Harbor12345
    plainHTTP: false
    privateRegistry: "dockerhub.kubekey.local"
    namespaceOverride: "kubesphereio"
    registryMirrors: []
    insecureRegistries: []
  addons: []



---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.2.1
spec:
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  local_registry: ""
  namespace_override: ""
  # dev_tag: ""
  etcd:
    monitoring: false
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    core:
      console:
        enableMultiLogin: true
        port: 30880
        type: NodePort
    # apiserver:
    #  resources: {}
    # controllerManager:
    #  resources: {}
    redis:
      enabled: false
      volumeSize: 2Gi
    openldap:
      enabled: false
      volumeSize: 2Gi
    minio:
      volumeSize: 20Gi
    monitoring:
      # type: external
      endpoint: http://prometheus-operated.kubesphere-monitoring-system.svc:9090
      GPUMonitoring:
        enabled: false
    gpu:
      kinds:
      - resourceName: "nvidia.com/gpu"
        resourceType: "GPU"
        default: true
    es:
      # master:
      #   volumeSize: 4Gi
      #   replicas: 1
      #   resources: {}
      # data:
      #   volumeSize: 20Gi
      #   replicas: 1
      #   resources: {}
      logMaxAge: 7
      elkPrefix: logstash
      basicAuth:
        enabled: false
        username: ""
        password: ""
      externalElasticsearchHost: ""
      externalElasticsearchPort: ""
  alerting:
    enabled: false
    # thanosruler:
    #   replicas: 1
    #   resources: {}
  auditing:
    enabled: false
    # operator:
    #   resources: {}
    # webhook:
    #   resources: {}
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
    # operator:
    #   resources: {}
    # exporter:
    #   resources: {}
    # ruler:
    #   enabled: true
    #   replicas: 2
    #   resources: {}
  logging:
    enabled: false
    containerruntime: docker
    logsidecar:
      enabled: true
      replicas: 2
      # resources: {}
  metrics_server:
    enabled: false
  monitoring:
    storageClass: ""
    # kube_rbac_proxy:
    #   resources: {}
    # kube_state_metrics:
    #   resources: {}
    # prometheus:
    #   replicas: 1
    #   volumeSize: 20Gi
    #   resources: {}
    #   operator:
    #     resources: {}
    #   adapter:
    #     resources: {}
    # node_exporter:
    #   resources: {}
    # alertmanager:
    #   replicas: 1
    #   resources: {}
    # notification_manager:
    #   resources: {}
    #   operator:
    #     resources: {}
    #   proxy:
    #     resources: {}
    gpu:
      nvidia_dcgm_exporter:
        enabled: false
        # resources: {}
  multicluster:
    clusterRole: none
  network:
    networkpolicy:
      enabled: false
    ippool:
      type: none
    topology:
      type: none
  openpitrix:
    store:
      enabled: false
  servicemesh:
    enabled: false
  kubeedge:
    enabled: false
    cloudCore:
      nodeSelector: {"node-role.kubernetes.io/worker": ""}
      tolerations: []
      cloudhubPort: "10000"
      cloudhubQuicPort: "10001"
      cloudhubHttpsPort: "10002"
      cloudstreamPort: "10003"
      tunnelPort: "10004"
      cloudHub:
        advertiseAddress:
          - ""
        nodeLimit: "100"
      service:
        cloudhubNodePort: "30000"
        cloudhubQuicNodePort: "30001"
        cloudhubHttpsNodePort: "30002"
        cloudstreamNodePort: "30003"
        tunnelNodePort: "30004"
    edgeWatcher:
      nodeSelector: {"node-role.kubernetes.io/worker": ""}
      tolerations: []
      edgeWatcherAgent:
        nodeSelector: {"node-role.kubernetes.io/worker": ""}
        tolerations: []
<<<文件结束
# 必须要设置好 registry ,不然后续我们更新将会很麻烦的。
# 执行 registry 安装
./kk init registry -f config.yaml -a kubesphere.tar.gz
.....
22:26:09 CST skipped: [kube_master02]
22:26:09 CST [InstallRegistryModule] Enable docker
22:26:11 CST skipped: [kube_master02]
22:26:11 CST [InstallRegistryModule] Install docker compose
22:26:15 CST success: [kube_master02]
22:26:15 CST [InstallRegistryModule] Sync harbor package
22:27:44 CST success: [kube_master02]
22:27:44 CST [InstallRegistryModule] Generate harbor config
22:27:47 CST success: [kube_master02]
22:27:47 CST [InstallRegistryModule] start harbor

Local image registry created successfully. Address: dockerhub.kubekey.local

22:28:24 CST success: [kube_master02]
22:28:24 CST Pipeline[InitRegistryPipeline] execute successful
```

好像安装成功了。。。

```
ssh root@192.168.3.66
[root@kube_node1 ~]# docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED         STATUS                   PORTS                                                                                                                       NAMES
0b6262ef1994   goharbor/nginx-photon:v2.4.1           "nginx -g 'daemon of…"   3 minutes ago   Up 3 minutes (healthy)   0.0.0.0:4443->4443/tcp, :::4443->4443/tcp, 0.0.0.0:80->8080/tcp, :::80->8080/tcp, 0.0.0.0:443->8443/tcp, :::443->8443/tcp   nginx
7577ff03905d   goharbor/harbor-jobservice:v2.4.1      "/harbor/entrypoint.…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               harbor-jobservice
3ba916375569   goharbor/notary-server-photon:v2.4.1   "/bin/sh -c 'migrate…"   3 minutes ago   Up 3 minutes                                                                                                                                         notary-server
6c6ab9420de0   goharbor/harbor-core:v2.4.1            "/harbor/entrypoint.…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               harbor-core
4e66ca5bff32   goharbor/trivy-adapter-photon:v2.4.1   "/home/scanner/entry…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               trivy-adapter
554c004cf1b2   goharbor/notary-signer-photon:v2.4.1   "/bin/sh -c 'migrate…"   3 minutes ago   Up 3 minutes                                                                                                                                         notary-signer
1368d5c294c3   goharbor/redis-photon:v2.4.1           "redis-server /etc/r…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               redis
955b1e91535f   goharbor/harbor-registryctl:v2.4.1     "/home/harbor/start.…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               registryctl
ae71922e7b43   goharbor/chartmuseum-photon:v2.4.1     "./docker-entrypoint…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               chartmuseum
636a15e66450   goharbor/harbor-db:v2.4.1              "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               harbor-db
f041fefe6684   goharbor/harbor-portal:v2.4.1          "nginx -g 'daemon of…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               harbor-portal
f030bb92b6ba   goharbor/registry-photon:v2.4.1        "/home/harbor/entryp…"   3 minutes ago   Up 3 minutes (healthy)                                                                                                                               registry
fc4fb2a3474f   goharbor/harbor-log:v2.4.1             "/bin/sh -c /usr/loc…"   3 minutes ago   Up 3 minutes (healthy)   127.0.0.1:1514->10514/tcp                                                                                                   harbor-log
```

## 创建 Harbor 项目

```
vim /create_project_harbor.sh
#!/usr/bin/env bash
   
# Copyright 2018 The KubeSphere Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
   
url="https://dockerhub.kubekey.local"  #修改url的值为https://dockerhub.kubekey.local
user="admin"
passwd="Harbor12345"
   
harbor_projects=(library
    kubesphereio
    kubesphere
    calico
    coredns
    openebs
    csiplugin
    minio
    mirrorgooglecontainers
    osixia
    prom
    thanosio
    jimmidyson
    grafana
    elastic
    istio
    jaegertracing
    jenkins
    weaveworks
    openpitrix
    joosthofman
    nginxdemos
    fluent
    kubeedge
)
   
for project in "${harbor_projects[@]}"; do
    echo "creating $project"
    curl -u "${user}:${passwd}" -X POST -H "Content-Type: application/json" "${url}/api/v2.0/projects" -d "{ \"project_name\": \"${project}\", \"public\": true}" -k #curl命令末尾加上 -k
done
<<<文件结束
#再次执行以下命令修改集群配置文件
vim config.yaml
  ...
  registry:
    type: harbor
    auths:
      "dockerhub.kubekey.local":
        username: admin
        password: Harbor12345
    plainHTTP: false
    privateRegistry: "dockerhub.kubekey.local"
    namespaceOverride: "kubesphereio"
    registryMirrors: []
    insecureRegistries: []
  addons: []
```

## 安装 KubeSphere 集群

```
./kk create cluster -f config.yaml -a kubesphere.tar.gz --with-packages
22:32:58 CST [ConfirmModule] Display confirmation form
+---------------+------+------+---------+----------+-------+-------+-----------+--------+---------+------------+-------------+------------------+--------------+
| name          | sudo | curl | openssl | ebtables | socat | ipset | conntrack | chrony | docker  | nfs client | ceph client | glusterfs client | time         |
+---------------+------+------+---------+----------+-------+-------+-----------+--------+---------+------------+-------------+------------------+--------------+
| kube_master01 | y    | y    | y       | y        |       | y     |           | y      |         | y          |             | y                | PDT 07:32:57 |
| kube_master02 | y    | y    | y       | y        |       | y     |           | y      | 20.10.8 | y          |             | y                | PDT 07:32:57 |
| kube_master03 | y    | y    | y       | y        |       | y     |           | y      |         | y          |             | y                | PDT 07:32:57 |
| kube_node01   | y    | y    | y       | y        |       | y     |           | y      |         | y          |             | y                | PDT 07:32:58 |
| kube_node02   | y    | y    | y       | y        |       | y     |           | y      |         | y          |             | y                | PDT 07:32:57 |
+---------------+------+------+---------+----------+-------+-------+-----------+--------+---------+------------+-------------+------------------+--------------+

.....
# 执行过程中你会可能会发现磁盘不够用的现象。。。。
failure: repodata/repomd.xml from base-local: [Errno 256] No more mirrors to try.
file:///tmp/kubekey/iso/repodata/repomd.xml: [Errno -1] Error importing repomd.xml for base-local: Damaged repomd.xml file: Process exited with status 1
failed: [kube_node1] [AddLocalRepository] exec failed after 1 retires: update local repository failed: Failed to exec command: sudo -E /bin/bash -c "yum clean all && yum makecache"
Loaded plugins: fastestmirror, langpacks
Cleaning repos: base-local

#没关系我们忽略错误试试
./kk create cluster -f config.yaml -a kubesphere.tar.gz --with-packages --ignore-err

.....结果还是不行
failure: repodata/repomd.xml from base-local: [Errno 256] No more mirrors to try.
file:///tmp/kubekey/iso/repodata/repomd.xml: [Errno -1] Error importing repomd.xml for base-local: Damaged repomd.xml file: Process exited with status 1

或者发生如下错误：

error: Pipeline[CreateClusterPipeline] execute failed: Module[CopyImagesToRegistryModule] exec failed:
failed: [LocalHost] [CopyImagesToRegistry] exec failed after 1 retires: read index.json failed: open /root/kubekey/images/index.json: no such file or directory

反正我是想近任何方法想把它跑通，但是确实没有办法搞出来。。。
```

花了大半天研究的离线部署，最后成了这样。。。

___

## kubesphere离线打包问题(安装问题整理)

1、下载文件要依赖已存在的集群

```
[root@kube_master ~]# ./kk create manifest
/root/manifest-sample.yaml already exists. Are you sure you want to overwrite this file? [yes/no]: yes
error: get kubernetes client failed: open /root/.kube/config: no such file or directory
```

如果我的集群里没有安装过kubernates或者kubesphere，竟然无法执行create manifest，这里没想通。

2、下载文件时经常性卡死退出需要关闭docker服务才行。

```
downloading amd64 harbor v2.4.1 ...
已杀死
```

3、下载文件的配置manifest.yaml没有给充足的说明如何使用导致下载文件时间巨长，其实很多文件根本没啥用。所以建议使用manifest-sample.yaml文件，方便简单。  
为了下载这些文件，我的整个磁盘最大占用差不多50G了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/55b1c26df17f409686d13c537df7da6d.png)

4、无法下载ISO文件，需要手动指定安装的路径。

```
错误如下：
failed: [LocalHost] [DownloadISOFile] exec failed after 1 retires: Failed to download centos-7-amd64.iso iso file: curl -L -o /root/kubekey/centos-7-amd64.iso https://github.com/kubesphere/kubekey/releases/download/v2.0.0/centos-7-amd64-rpms.iso error: exit status 35
```

修改manifest.yaml如下:  
需要提前从github上手动下载下来才行。

```
  operatingSystems:
  - arch: amd64
    type: linux
    id: centos
    version: "7"
    osImage: CentOS Linux 7 (Core)
    repository:
      iso:
        localPath: /root/centos-7-amd64-rpms.iso
        url: ""
```

5、我的实际打包文件其实跟我的配置文件一点关系都没有，比如我使用manifest\_mini.yaml进行打包实际它依然会把无关的包给我打进来了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/0704464acc3c41639207c2908c07bffb.png)  
以至于我整个包差不多15个G了，哈哈。

## 总结

KubeSphere单机还是挺好用的，可能对支持联网状态的安装比较友好吧。对于tob场景下的离线安装我基本就放弃了。

其实之前腾讯云出一版离线安装教程，地址如下：  
https://cloud.tencent.com/developer/article/1802614  
下载离线包大约10G左右，并且版本是固定的，这对于我们生产环境来说不一定是好的，我感觉KubeSphere是应该思考一下如何支持离线部署了。