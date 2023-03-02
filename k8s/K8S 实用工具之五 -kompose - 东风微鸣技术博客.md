## 开篇[](https://ewhisper.cn/posts/35291/#%E5%BC%80%E7%AF%87%20-6)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

-   第一篇：《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》
-   第二篇：《[K8S 实用工具之二 - 终端 UI K9S](https://ewhisper.cn/posts/48546/)》
-   第三篇：《[K8S 实用工具之三 - 图形化 UI Lens](https://ewhisper.cn/posts/33163)》
-   第四篇：《[K8S 实用工具之四 - kubectl 实用插件](https://ewhisper.cn/posts/60907)》

## Kubernetes + Compose = Kompose[](https://ewhisper.cn/posts/35291/#Kubernetes-Compose-Kompose)

**从 Docker Compose 到 Kubernetes 的转换工具**

Kompose 是 dockercompose 到 Kubernetes （或 OpenShift) 等容器编排器的转换工具。

为什么开发者喜欢它？

-   使用 Docker Compose 简化开发过程，然后将容器部署到生产集群
-   转换你的 `docker-compose.yaml` 需要一个简单的命令 `kompose convert`

## 易如反掌[](https://ewhisper.cn/posts/35291/#%E6%98%93%E5%A6%82%E5%8F%8D%E6%8E%8C)

1.  找一个 `docker-compose.yaml` 文件；
2.  执行：`kompose convert`
3.  执行 `kubectl apply` 并检查您的 k8s 集群为您新部署的容器！

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code>$ wget https://raw.githubusercontent.com/kubernetes/kompose/master/examples/docker-compose-v3.yaml -O docker-compose.yaml<p>$ kompose convert</p><p>$ kubectl apply -f .</p><p>$ kubectl get po<br>NAME                            READY     STATUS              RESTARTS   AGE<br>frontend-591253677-5t038        1/1       Running             0          10s<br>redis-master-2410703502-9hshf   1/1       Running             0          10s<br>redis-slave-4049176185-hr1lr    1/1       Running             0          10s</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 实战[](https://ewhisper.cn/posts/35291/#%E5%AE%9E%E6%88%98)

比如我要在 K8S 上安装 RssHub，这是官方提供的 `docker-compose.yml`:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br></pre></td><td><pre><code><span>version:</span> <span>'3'</span><br><span>services:</span><br>    <span>rsshub:</span><br>        <span>image:</span> <span>diygod/rsshub</span><br>        <span>restart:</span> <span>always</span><br>        <span>ports:</span><br>            <span>-</span> <span>'1200:1200'</span><br>        <span>environment:</span><br>            <span>NODE_ENV:</span> <span>production</span><br>            <span>CACHE_TYPE:</span> <span>redis</span><br>            <span>REDIS_URL:</span> <span>'redis://redis:6379/'</span><br>            <span>PUPPETEER_WS_ENDPOINT:</span> <span>'ws://browserless:3000'</span><br>        <span>depends_on:</span><br>            <span>-</span> <span>redis</span><br>            <span>-</span> <span>browserless</span><br>    <span>browserless:</span><br>        <span># See issue 6680</span><br>        <span>image:</span> <span>browserless/chrome:1.43-chrome-stable</span><br>        <span>restart:</span> <span>always</span><br>        <span>ulimits:</span><br>          <span>core:</span><br>            <span>hard:</span> <span>0</span><br>            <span>soft:</span> <span>0</span><br>    <span>redis:</span><br>        <span>image:</span> <span>redis:alpine</span><br>        <span>restart:</span> <span>always</span><br>        <span>volumes:</span><br>            <span>-</span> <span>redis-data:/data</span><br><span>volumes:</span><br>    <span>redis-data:</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

执行 `kompose convert` 后，从 `docker-compose.yml` 生成以下文件：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>$ ll<br>.rw-r--r--  711 casey  1 Dec 21:20 browserless-deployment.yaml<br>.rw-r--r--  715 casey  1 Dec 21:20 docker-compose.yml<br>.rw-r--r--  243 casey  1 Dec 21:20 redis-data-persistentvolumeclaim.yaml<br>.rw-r--r--  867 casey  1 Dec 21:20 redis-deployment.yaml<br>.rw-r--r-- 1.0k casey  1 Dec 21:20 rsshub-deployment.yaml<br>.rw-r--r--  352 casey  1 Dec 21:20 rsshub-service.yaml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

每个 docker-compose 容器，会生成为一个 deployment，并为你自动转换好 label 和 env 等字段，以 `rsshub-deployment.yaml` 为例：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>apps/v1</span><br><span>kind:</span> <span>Deployment</span><br><span>metadata:</span><br>  <span>annotations:</span><br>    <span>kompose.cmd:</span> <span>kompose</span> <span>convert</span><br>    <span>kompose.version:</span> <span>1.26</span><span>.0</span> <span>(40646f47)</span><br>  <span>creationTimestamp:</span> <span>null</span><br>  <span>labels:</span><br>    <span>io.kompose.service:</span> <span>rsshub</span><br>  <span>name:</span> <span>rsshub</span><br><span>spec:</span><br>  <span>replicas:</span> <span>1</span><br>  <span>selector:</span><br>    <span>matchLabels:</span><br>      <span>io.kompose.service:</span> <span>rsshub</span><br>  <span>strategy:</span> {}<br>  <span>template:</span><br>    <span>metadata:</span><br>      <span>annotations:</span><br>        <span>kompose.cmd:</span> <span>kompose</span> <span>convert</span><br>        <span>kompose.version:</span> <span>1.26</span><span>.0</span> <span>(40646f47)</span><br>      <span>creationTimestamp:</span> <span>null</span><br>      <span>labels:</span><br>        <span>io.kompose.service:</span> <span>rsshub</span><br>    <span>spec:</span><br>      <span>containers:</span><br>        <span>-</span> <span>env:</span><br>            <span>-</span> <span>name:</span> <span>CACHE_TYPE</span><br>              <span>value:</span> <span>redis</span><br>            <span>-</span> <span>name:</span> <span>NODE_ENV</span><br>              <span>value:</span> <span>production</span><br>            <span>-</span> <span>name:</span> <span>PUPPETEER_WS_ENDPOINT</span><br>              <span>value:</span> <span>ws://browserless:3000</span><br>            <span>-</span> <span>name:</span> <span>REDIS_URL</span><br>              <span>value:</span> <span>redis://redis:6379/</span><br>          <span>image:</span> <span>diygod/rsshub</span><br>          <span>name:</span> <span>rsshub</span><br>          <span>ports:</span><br>            <span>-</span> <span>containerPort:</span> <span>1200</span><br>          <span>resources:</span> {}<br>      <span>restartPolicy:</span> <span>Always</span><br><span>status:</span> {}<br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

Docker compose 的 `ports` 部分，会转换为 SVC，以 `rsshub-service.yaml` 为例：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Service</span><br><span>metadata:</span><br>  <span>annotations:</span><br>    <span>kompose.cmd:</span> <span>kompose</span> <span>convert</span><br>    <span>kompose.version:</span> <span>1.26</span><span>.0</span> <span>(40646f47)</span><br>  <span>creationTimestamp:</span> <span>null</span><br>  <span>labels:</span><br>    <span>io.kompose.service:</span> <span>rsshub</span><br>  <span>name:</span> <span>rsshub</span><br><span>spec:</span><br>  <span>ports:</span><br>    <span>-</span> <span>name:</span> <span>"1200"</span><br>      <span>port:</span> <span>1200</span><br>      <span>targetPort:</span> <span>1200</span><br>  <span>selector:</span><br>    <span>io.kompose.service:</span> <span>rsshub</span><br><span>status:</span><br>  <span>loadBalancer:</span> {}<br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

Docker compose 的 `volumes` 字段，会转换为 PVC，以 `redis-data-persistentvolumeclaim.yaml` 为例：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>PersistentVolumeClaim</span><br><span>metadata:</span><br>  <span>creationTimestamp:</span> <span>null</span><br>  <span>labels:</span><br>    <span>io.kompose.service:</span> <span>redis-data</span><br>  <span>name:</span> <span>redis-data</span><br><span>spec:</span><br>  <span>accessModes:</span><br>    <span>-</span> <span>ReadWriteOnce</span><br>  <span>resources:</span><br>    <span>requests:</span><br>      <span>storage:</span> <span>100Mi</span><br><span>status:</span> {}<br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

安逸！

## 安装[](https://ewhisper.cn/posts/35291/#%E5%AE%89%E8%A3%85)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span># Linux</span><br>curl -L https://github.com/kubernetes/kompose/releases/download/v1.25.0/kompose-linux-amd64 -o kompose<p><span># macOS</span><br>curl -L https://github.com/kubernetes/kompose/releases/download/v1.25.0/kompose-darwin-amd64 -o kompose</p><p>chmod +x kompose<br>sudo mv ./kompose /usr/<span>local</span>/bin/kompose</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## That’s All[](https://ewhisper.cn/posts/35291/#That%E2%80%99s-All)

🎉🎉🎉

## 参考链接[](https://ewhisper.cn/posts/35291/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Kubernetes + Compose = Kompose](https://kompose.io/)