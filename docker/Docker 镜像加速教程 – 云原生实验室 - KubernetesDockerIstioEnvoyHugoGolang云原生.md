在使用 Docker 和 Kubernetes 时，我们经常需要访问 `gcr.io` 和 `quay.io` 镜像仓库，由于众所周知的原因，这些镜像仓库在中国都无法访问，唯一能访问的是 Docker Hub，但速度也是奇慢无比。`gcr.azk8s.cn` 是 `gcr.io` 镜像仓库的代理站点，原来可以通过 `gcr.azk8s.cn` 访问 gcr.io 仓库里的镜像，但是目前 `*.azk8s.cn` 已经仅限于 `Azure` 中国的 IP 使用，不再对外提供服务了。国内其他的镜像加速方案大多都是采用定时同步的方式来缓存，这种方法是有一定延迟的，不能保证及时更新，ustc 和七牛云等镜像加速器我都试过了，非常不靠谱，很多镜像都没有。

为了能够顺利访问 `gcr.io` 等镜像仓库，我们需要在墙外自己搭建一个类似于 `gcr.azk8s.cn` 的镜像仓库代理站点。利用 Docker 的开源项目 [registry](https://docs.docker.com/registry/) 就可以实现这个需求，registry 不仅可以作为本地私有镜像仓库，还可以作为上游镜像仓库的缓存，也就是 `pull through cache`。

先来感受下速度：

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210217224424.png)

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210217224444.png)

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210217224500.jpg)

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210217224515.jpg)

## 1\. 前提条件

___

-   一台能够施展魔法的服务器（你懂得，可以直接访问 gcr.io）
-   一个域名和域名相关的 SSL 证书（docker pull 镜像时需要验证域名证书），一般用 [Let’s Encrypt](https://letsencrypt.org/) 就够了。

## 2\. 核心思路

___

registry 可以通过设置参数 `remoteurl` 将其作为远端仓库的缓存仓库，这样当你通过这个私有仓库的地址拉取镜像时，regiistry 会先将镜像缓存到本地存储，然后再提供给拉取的客户端（有可能这两个步骤是同时的，我也不太清楚）。我们可以先部署一个私有 registry，然后将 `remoteurl` 设为需要加速的镜像仓库地址，基本上就可以了。

## 3\. 定制 registry

为了能够支持缓存 `docker.io`、`gcr.io`、`k8s.gcr.io`、`quay.io` 和 `ghcr.io` 等常见的公共镜像仓库，我们需要对 registry 的配置文件进行定制，Dockerfile 如下：

```
FROM registry:2.6
LABEL maintainer="registry-proxy Docker Maintainers https://icloudnative.io"
ENV PROXY_REMOTE_URL="" \
    DELETE_ENABLED=""
COPY entrypoint.sh /entrypoint.sh
```

其中 `entrypoint.sh` 用来将环境变量传入配置文件：

```
#!/bin/sh

set -e

CONFIG_YML=/etc/docker/registry/config.yml

if [ -n "$PROXY_REMOTE_URL" -a `grep -c "$PROXY_REMOTE_URL" $CONFIG_YML` -eq 0 ]; then
    echo "proxy:" >> $CONFIG_YML
    echo "  remoteurl: $PROXY_REMOTE_URL" >> $CONFIG_YML
    echo "  username: $PROXY_USERNAME" >> $CONFIG_YML
    echo "  password: $PROXY_PASSWORD" >> $CONFIG_YML
    echo "------ Enabled proxy to remote: $PROXY_REMOTE_URL ------"
elif [ $DELETE_ENABLED = true -a `grep -c "delete:" $CONFIG_YML` -eq 0 ]; then
    sed -i '/rootdirectory/a\  delete:' $CONFIG_YML
    sed -i '/delete/a\    enabled: true' $CONFIG_YML
    echo "------ Enabled local storage delete -----"
fi

sed -i "/headers/a\    Access-Control-Allow-Origin: ['*']" $CONFIG_YML
sed -i "/headers/a\    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']" $CONFIG_YML
sed -i "/headers/a\    Access-Control-Expose-Headers: ['Docker-Content-Digest']" $CONFIG_YML

case "$1" in
    *.yaml|*.yml) set -- registry serve "$@" ;;
    serve|garbage-collect|help|-*) set -- registry "$@" ;;
esac

exec "$@"
```

## 4\. 启动缓存服务

构建好 Docker 镜像之后，就可以启动服务了。如果你不想自己构建，可以直接用我的镜像：`yangchuansheng/registry-proxy`。

一般来说，即使你要同时缓存 `docker.io`、`gcr.io`、`k8s.gcr.io`、`quay.io` 和 `ghcr.io`，一台 `1C 2G` 的云主机也足够了（前提是你不在上面跑其他的服务）。我的博客、评论服务和其他一堆乱七八糟的服务都要跑在云主机上，所以一台是不满足我的需求的，我直接买了两台腾讯云香港轻量级服务器。

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210216182846.png)

既然买了两台，肯定得 [组个 k3s 集群](https://icloudnative.io/posts/deploy-k3s-cross-public-cloud/)啦，看主机名就知道我是用来干啥的。其中 2C 4G 作为 master 节点，1C 2G 作为 node 节点。

以 `docker.io` 为例，创建资源清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dockerhub
  labels:
    app: dockerhub
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dockerhub
  template:
    metadata:
      labels:
        app: dockerhub
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - dockerhub
              topologyKey: kubernetes.io/hostname
            weight: 1
      dnsPolicy: None
      dnsConfig:
        nameservers:
          - 8.8.8.8
          - 8.8.4.4
      containers:
      - name: dockerhub
        image: yangchuansheng/registry-proxy:latest
        env:
        - name: PROXY_REMOTE_URL
          value: https://registry-1.docker.io
        - name: PROXY_USERNAME
          value: yangchuansheng
        - name: PROXY_PASSWORD
          value: ********
        ports:
        - containerPort: 5000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
        - mountPath: /var/lib/registry
          name: registry
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: registry
        hostPath:
          path: /var/lib/registry
---
apiVersion: v1
kind: Service
metadata:
  name: dockerhub
  labels:
    app: dockerhub
spec:
  selector:
    app: dockerhub
  ports:
    - protocol: TCP
      name: http
      port: 5000
      targetPort: 5000
```

使用资源清单创建对应的服务：

```
🐳  → kubectl apply -f dockerhub.yaml
```

如果你只有一台主机，可以使用 `docker-compose` 来编排容器，配置文件可以自己参考 k8s 的配置修改，本文就不赘述了。

## 5\. 代理选择

如果只缓存 `docker.io`，可以直接将 registry-proxy 的端口改成 `443`，并添加 SSL 证书配置。如果要缓存多个公共镜像仓库，就不太推荐这么做了，因为 443 端口只有一个，多个 registry-proxy 服务不能共用一个端口，合理的做法是使用边缘代理服务根据域名来转发请求到不同的 registry-proxy 服务。

对于 Kubernetes 集群来说，`Ingress Controller` 即边缘代理，常见的 `Ingress Controller` 基本上都是由 `Nginx` 或者 [Envoy](https://icloudnative.io/envoy-handbook/) 来实现。 [Envoy](https://icloudnative.io/envoy-handbook/) 虽为代理界新秀，但生而逢时，它的很多特性都是原生为云准备的，是真正意义上的 Cloud Native L7 代理和通信总线。比如它的服务发现和动态配置功能，与 `Nginx` 等代理的热加载不同， [Envoy](https://icloudnative.io/envoy-handbook/) 可以通过 `API` 来实现其控制平面，控制平面可以集中服务发现，并通过 `API` 接口动态更新数据平面的配置，不需要重启数据平面的代理。不仅如此，控制平面还可以通过 API 将配置进行分层，然后逐层更新。

目前使用 [Envoy](https://icloudnative.io/envoy-handbook/) 实现的 Ingress Controller 有 [Contour](https://icloudnative.io/posts/use-envoy-as-a-kubernetes-ingress/)、 [Ambassador](https://github.com/datawire/ambassador) 和 [Gloo](https://github.com/solo-io/gloo) 等，如果你对 [Envoy](https://icloudnative.io/envoy-handbook/) 比较感兴趣，并且想使用 Ingress Controller 作为边缘代理，可以试试 [Contour](https://icloudnative.io/posts/use-envoy-as-a-kubernetes-ingress/)。Ingress Controller 对底层做了抽象，屏蔽了很多细节，无法顾及到所有细节的配置，必然不会支持底层代理所有的配置项，所以我选择使用原生的 [Envoy](https://icloudnative.io/envoy-handbook/) 来作为边缘代理。如果你是单机跑的 registry-proxy 服务，也可以试试 [Envoy](https://icloudnative.io/envoy-handbook/)。

## 6\. 代理配置

首先创建 [Envoy](https://icloudnative.io/envoy-handbook/) 的资源清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: envoy
  namespace: kube-system
  labels:
    app: envoy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: envoy
  strategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: envoy
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: envoy
        image: envoyproxy/envoy:v1.17-latest
        imagePullPolicy: IfNotPresent
        command:
        - envoy
        - /etc/envoy/envoy.yaml
        ports:
        - containerPort: 443
          name: https
        - containerPort: 80
          name: http
        - containerPort: 15001
          name: http-metrics
        volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
        - mountPath: /etc/envoy
          name: envoy
        - mountPath: /root/.acme.sh/icloudnative.io
          name: ssl
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: ssl
        hostPath:
          path: /root/.acme.sh/icloudnative.io
      - name: envoy
        hostPath:
          path: /etc/envoy
```

使用资源清单创建对应的服务：

```
🐳  → kubectl apply -f envoy.yaml
```

这里选择使用 `hostPath` 将 envoy 的配置挂载到容器中，然后 [通过文件来动态更新配置](https://icloudnative.io/posts/file-based-dynamic-routing-configuration/)。来看下 [Envoy](https://icloudnative.io/envoy-handbook/) 的配置，先进入 `/etc/envoy` 目录。

`bootstrap` 配置：

```
node:
  id: node0
  cluster: cluster0
dynamic_resources:
  lds_config:
    path: /etc/envoy/lds.yaml
  cds_config:
    path: /etc/envoy/cds.yaml
admin:
  access_log_path: "/dev/stdout"
  address:
    socket_address:
      address: "0.0.0.0"
      port_value: 15001
```

`LDS` 的配置：

```
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.config.listener.v3.Listener
  name: listener_http
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 80
  filter_chains:
  - filters:
    - name: envoy.filters.network.http_connection_manager
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
        stat_prefix: ingress_http
        codec_type: AUTO
        access_log:
          name: envoy.access_loggers.file
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
            path: /dev/stdout
        route_config:
          name: http_route
          virtual_hosts:
          - name: default
            domains:
            - "*"
            routes:
            - match:
                prefix: "/"
              redirect:
                https_redirect: true
                port_redirect: 443
                response_code: "FOUND"
        http_filters:
        - name: envoy.filters.http.router
- "@type": type.googleapis.com/envoy.config.listener.v3.Listener
  name: listener_https
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 443
  listener_filters:
  - name: "envoy.filters.listener.tls_inspector"
    typed_config: {}
  filter_chains:
  - transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
        common_tls_context:
          alpn_protocols: h2,http/1.1
          tls_certificates:
          - certificate_chain:
              filename: "/root/.acme.sh/icloudnative.io/fullchain.cer"
            private_key:
              filename: "/root/.acme.sh/icloudnative.io/icloudnative.io.key"
    filters:
    - name: envoy.filters.network.http_connection_manager
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
        stat_prefix: ingress_https
        codec_type: AUTO
        use_remote_address: true
        access_log:
          name: envoy.access_loggers.file
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
            path: /dev/stdout
        route_config:
          name: https_route
          response_headers_to_add:
          - header:
              key: Strict-Transport-Security
              value: "max-age=15552000; includeSubdomains; preload"
          virtual_hosts:
          - name: docker
            domains:
            - docker.icloudnative.io
            routes:
            - match:
                prefix: "/"
              route:
                cluster: dockerhub
                timeout: 600s
        http_filters:
        - name: envoy.filters.http.router
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router 
```

`CDS` 的配置：

```
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.config.cluster.v3.Cluster
  name: dockerhub
  connect_timeout: 15s
  type: strict_dns
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: dockerhub
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: dockerhub.default
              port_value: 5000
```

这里的 `address` 使用的是 Kubernetes 集群内部域名，其他部署方式请自己斟酌。

配置好了 [Envoy](https://icloudnative.io/envoy-handbook/) 之后，就可以通过代理服务器拉取 `docker.io` 的镜像了。

## 7\. 验证加速效果

现在你就可以通过代理服务器来拉取公共镜像了。比如你想拉取 `nginx:alpine` 镜像，可以使用下面的命令：

```
🐳  → docker pull docker.icloudnative.io/library/nginx:alpine

alpine: Pulling from library/nginx
801bfaa63ef2: Pull complete
b1242e25d284: Pull complete
7453d3e6b909: Pull complete
07ce7418c4f8: Pull complete
e295e0624aa3: Pull complete
Digest: sha256:c2ce58e024275728b00a554ac25628af25c54782865b3487b11c21cafb7fabda
Status: Downloaded newer image for docker.icloudnative.io/library/nginx:alpine
docker.icloudnative.io/library/nginx:alpine
```

## 8\. 缓存所有镜像仓库

前面的示例只是缓存了 `docker.io`，如果要缓存所有的公共镜像仓库，可以参考 4-6 节的内容。以 `k8s.gcr.io` 为例，先准备一个资源清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gcr-k8s
  labels:
    app: gcr-k8s
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gcr-k8s
  template:
    metadata:
      labels:
        app: gcr-k8s
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - gcr-k8s
              topologyKey: kubernetes.io/hostname
            weight: 1
      dnsPolicy: None
      dnsConfig:
        nameservers:
          - 8.8.8.8
          - 8.8.4.4
      containers:
      - name: gcr-k8s
        image: yangchuansheng/registry-proxy:latest
        env:
        - name: PROXY_REMOTE_URL
          value: https://k8s.gcr.io
        ports:
        - containerPort: 5000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
        - mountPath: /var/lib/registry
          name: registry
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: registry
        hostPath:
          path: /var/lib/registry
---
apiVersion: v1
kind: Service
metadata:
  name: gcr-k8s
  labels:
    app: gcr-k8s
spec:
  selector:
    app: gcr-k8s
  ports:
    - protocol: TCP
      name: http
      port: 5000
      targetPort: 5000
```

将其部署到 Kubernetes 集群中：

```
🐳  → kubectl apply -f gcr-k8s.yaml
```

在 `lds.yaml` 中添加相关配置：

```
          virtual_hosts:
          - name: docker
            ...
            ...
          - name: k8s
            domains:
            - k8s.icloudnative.io
            routes:
            - match:
                prefix: "/"
              route:
                cluster: gcr-k8s
                timeout: 600s
```

在 `cds.yaml` 中添加相关配置：

```
- "@type": type.googleapis.com/envoy.config.cluster.v3.Cluster
  name: gcr-k8s
  connect_timeout: 1s
  type: strict_dns
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: gcr-k8s
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: gcr-k8s.default
              port_value: 5000
```

其他镜像仓库可照搬上述步骤，以下是我自己跑的所有缓存服务容器：

```
🐳  → kubectl get pod -o wide

gcr-8647ffb586-67c6g                     1/1     Running   0          21h     10.42.1.52    blog-k3s02
ghcr-7765f6788b-hxxvc                    1/1     Running   0          21h     10.42.1.55    blog-k3s01
dockerhub-94bbb7497-x4zwg                1/1     Running   0          21h     10.42.1.54    blog-k3s02
gcr-k8s-644db84879-7xssb                 1/1     Running   0          21h     10.42.1.53    blog-k3s01
quay-559b65848b-ljclb                    1/1     Running   0          21h     10.42.0.154   blog-k3s01
```

## 9\. 容器运行时配置

配置好所有的缓存服务后，就可以通过代理来拉取公共镜像了，只需按照下面的列表替换镜像地址中的字段就行了：

| 原 URL | 替换后的 URL |
| --- | --- |
| docker.io/xxx/xxx 或 xxx/xxx | docker.icloudnative.io/xxx/xxx |
| docker.io/library/xxx 或 xxx | docker.icloudnative.io/library/xxx |
| gcr.io/xxx/xxx | gcr.icloudnative.io/xxx/xxx |
| k8s.gcr.io/xxx/xxx | k8s.icloudnative.io/xxx/xxx |
| quay.io/xxx/xxx | quay.icloudnative.io/xxx/xxx |
| ghcr.io/xxx/xxx | ghcr.icloudnative.io/xxx/xxx |

当然，最好的方式还是直接配置 registry mirror，`Docker` 只支持配置 `docker.io` 的 registry mirror，`Containerd` 和 `Podman` 支持配置所有镜像仓库的 registry mirror。

### Docker

Docker 可以修改配置文件 `/etc/docker/daemon.json`，添加下面的内容：

```
{
    "registry-mirrors": [
        "https://docker.icloudnative.io"
    ]
}
```

然后重启 Docker 服务，就可以直接拉取 docker.io 的镜像了，不需要显示指定代理服务器的地址，Docker 服务本身会自动通过代理服务器去拉取镜像。比如：

```
🐳 → docker pull nginx:alpine
🐳 → docker pull docker.io/library/nginx:alpine
```

### Containerd

Containerd 就比较简单了，它支持任意 registry 的 mirror，只需要修改配置文件 `/etc/containerd/config.toml`，添加如下的配置：

```
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://docker.icloudnative.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
          endpoint = ["https://k8s.icloudnative.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
          endpoint = ["https://gcr.icloudnative.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
          endpoint = ["https://ghcr.icloudnative.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
          endpoint = ["https://quay.icloudnative.io"]
```

重启 `Containerd` 服务后，就可以直接拉取所有镜像了，不需要修改任何前缀，Containerd 会根据配置自动选择相应的代理 URL 拉取镜像。

### Podman

Podman 也支持任意 registry 的 mirror，只需要修改配置文件 `/etc/containers/registries.conf`，添加如下的配置：

```
unqualified-search-registries = ['docker.io', 'k8s.gcr.io', 'gcr.io', 'ghcr.io', 'quay.io']

[[registry]]
prefix = "docker.io"
insecure = true
location = "registry-1.docker.io"

[[registry.mirror]]
location = "docker.icloudnative.io"

[[registry]]
prefix = "k8s.gcr.io"
insecure = true
location = "k8s.gcr.io"

[[registry.mirror]]
location = "k8s.icloudnative.io"

[[registry]]
prefix = "gcr.io"
insecure = true
location = "gcr.io"

[[registry.mirror]]
location = "gcr.icloudnative.io"

[[registry]]
prefix = "ghcr.io"
insecure = true
location = "ghcr.io"

[[registry.mirror]]
location = "ghcr.icloudnative.io"

[[registry]]
prefix = "quay.io"
insecure = true
location = "quay.io"

[[registry.mirror]]
location = "quay.icloudnative.io"
```

然后就可以直接拉取所有镜像了，不需要修改任何前缀，Podman 会根据配置自动选择相应的代理 URL 拉取镜像。而且 Podman 还有 `fallback` 机制，上面的配置表示先尝试通过 `registry.mirror` 中 `location` 字段的 URL 来拉取镜像，如果失败就会尝试通过 `registry` 中 location 字段的 URL 来拉取。

## 10\. 清理缓存

缓存服务会将拉取的镜像缓存到本地，所以需要消耗磁盘容量。一般云主机的磁盘容量都不是很大，OSS 和 s3 存储都比较贵，不太划算。

为了解决这个问题，我推荐定期删除缓存到本地磁盘的部分镜像，或者删除所有镜像。方法也比较简单，单独再部署一个 registry，共用其他 registry 的存储，并启用 `delete` 功能，然后再通过 API 或者 Dashboard 进行删除。

先准备一个资源清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reg-local
  labels:
    app: reg-local
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reg-local
  template:
    metadata:
      labels:
        app: reg-local
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - reg-local
              topologyKey: kubernetes.io/hostname
            weight: 1
      containers:
      - name: reg-local
        image: yangchuansheng/registry-proxy:latest
        env:
        - name: DELETE_ENABLED
          value: "true"
        ports:
        - containerPort: 5000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
        - mountPath: /var/lib/registry
          name: registry
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: registry
        hostPath:
          path: /var/lib/registry
---
apiVersion: v1
kind: Service
metadata:
  name: reg-local
  labels:
    app: reg-local
spec:
  selector:
    app: reg-local
  ports:
    - protocol: TCP
      name: http
      port: 5000
      targetPort: 5000
```

将其部署到 Kubernetes 集群中：

```
🐳  → kubectl apply -f reg-local.yaml
```

再准备一个 Docker Registry UI 的资源清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry-ui
  labels:
    app: registry-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry-ui
  template:
    metadata:
      labels:
        app: registry-ui
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - registry-ui
              topologyKey: kubernetes.io/hostname
            weight: 1
      tolerations:
      - key: node-role.kubernetes.io/ingress
        operator: Exists
        effect: NoSchedule
      containers:
      - name: registry-ui
        image: joxit/docker-registry-ui:static
        env:
        - name: REGISTRY_TITLE
          value: My Private Docker Registry
        - name: REGISTRY_URL
          value: "http://reg-local:5000"
        - name: DELETE_IMAGES
          value: "true"
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
---
apiVersion: v1
kind: Service
metadata:
  name: registry-ui
  labels:
    app: registry-ui
spec:
  selector:
    app: registry-ui
  ports:
    - protocol: TCP
      name: http
      port: 80
      targetPort: 80
```

将其部署到 Kubernetes 集群中：

```
🐳  → kubectl apply -f registry-ui.yaml
```

这样就可以通过 Dashboard 来清理镜像释放空间了。

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@second/img/20210217172620.png)

或者直接简单粗暴，定时删除整个存储目录的内容。例如，执行命令 `crontab -e`，添加如下内容：

```
* * */2 * * /usr/bin/rm -rf /var/lib/registry/* &>/dev/null
```

表示每过两天清理一次 `/var/lib/registry/` 目录。

## 11\. 防白嫖认证

最后还有一个问题，我把缓存服务的域名全部公开了，如果大家都来白嫖，我的云主机肯定承受不住。为了防止白嫖，我得给 registry-proxy 加个认证，最简单的方法就是使用 `basic auth`，用 `htpasswd` 来存储密码。

1.  为用户 `admin` 创建一个密码文件，密码为 `admin`：
    
    ```
    🐳 → docker run \
      --entrypoint htpasswd \
      registry:2.6 -Bbn admin admin > htpasswd
    ```
    
2.  创建 Secret：
    
    ```
    🐳 → kubectl create secret generic registry-auth --from-file=htpasswd
    ```
    
3.  修改资源清单的配置，以 docker.io 为例：
    
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: dockerhub
      labels:
        app: dockerhub
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: dockerhub
      template:
        metadata:
          labels:
            app: dockerhub
        spec:
          affinity:
            podAntiAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
              - podAffinityTerm:
                  labelSelector:
                    matchExpressions:
                    - key: app
                      operator: In
                      values:
                      - dockerhub
                  topologyKey: kubernetes.io/hostname
                weight: 1
          dnsPolicy: None
          dnsConfig:
            nameservers:
              - 8.8.8.8
              - 8.8.4.4
          containers:
          - name: dockerhub
            image: yangchuansheng/registry-proxy:latest
            env:
            - name: PROXY_REMOTE_URL
              value: https://registry-1.docker.io
            - name: PROXY_USERNAME
              value: yangchuansheng
            - name: PROXY_PASSWORD
              value: ********
    +       - name: REGISTRY_AUTH_HTPASSWD_REALM
    +         value: Registry Realm
    +       - name: REGISTRY_AUTH_HTPASSWD_PATH
    +         value: /auth/htpasswd 
            ports:
            - containerPort: 5000
              protocol: TCP
            volumeMounts:
            - mountPath: /etc/localtime
              name: localtime
            - mountPath: /var/lib/registry
              name: registry
    +       - mountPath: /auth
    +         name: auth
          volumes:
          - name: localtime
            hostPath:
              path: /etc/localtime
          - name: registry
            hostPath:
              path: /var/lib/registry
    +     - name: auth
    +       secret:
    +         secretName: registry-auth
    ```
    
    apply 使其生效：
    
    ```
    🐳 → kubectl apply -f dockerhub.yaml
    ```
    
4.  尝试拉取镜像：
    
    ```
    🐳 → docker pull docker.icloudnative.io/library/nginx:latest
    
    Error response from daemon: Get https://docker.icloudnative.io/v2/library/nginx/manifests/latest: no basic auth credentials
    ```
    
5.  登录镜像仓库：
    
    ```
    🐳 → docker login docker.icloudnative.io
    Username: admin
    Password:
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    
    Login Succeeded
    ```
    
    现在就可以正常拉取镜像了。
    

如果你想更细粒度地控制权限，可以使用 `Token` 的方式来进行认证，具体可以参考 [docker\_auth](https://github.com/cesanta/docker_auth) 这个项目。

## 12\. 费用评估

好了，现在我们来评估一下这一切的费用。首先你得有一个会魔法的服务器，国内的肯定不用考虑了，必须选择国外的，而且到国内的速度还过得去的，最低最低不会低于 **30 人民币/月** 吧。除此之外，你还得拥有一个个人域名，这个价格不好说，总而言之，加起来肯定不会低于 30 吧，多数人肯定是下不去这个手的。没关系，我有一个更便宜的方案，我已经部署好了一切，你可以直接用我的服务，当然我也是自己买的服务器，每个月也是要花钱的，如果你真的想用，**只需要每月支付 3 元**，以此来保障我每个月的服务器费用。当然肯定不止你一个人，目前大概有十几个用户，后面如果人数特别多，再考虑加服务器。这个需要你自己考虑清楚，有意者扫描下方的二维码向我咨询：

![](https://jsdelivr.icloudnative.io/gh/yangchuansheng/imghosting@master/img/20200430221955.png)

\-------他日江湖相逢 再当杯酒言欢-------