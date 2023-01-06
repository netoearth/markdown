0%

[](https://github.com/bboysoulcn "Follow me on GitHub")

![](https://www.bboy.app/oss/20221012-1.webp)

### 简介

因为某些原因国外的

-   registry.k8s.io
-   quay.io
-   k8s.gcr.io
-   ghcr.io
-   gcr.io
-   dockerhub

这些镜像仓库在国内的访问体验不是很好，因为自己国外有几台服务器，索性搭建一个镜像代理服务

### 操作

其实很简单，我的想法就是cloudflare代理到服务器里面的nginx，然后nginx代理到registry容器就这样

`docker-compose.yaml`

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>version</span>: <span>"3"</span>
</span></span><span><span><span>services</span>:
</span></span><span><span>  <span>dockerhub</span>:
</span></span><span><span>    <span>image</span>: <span>"registry:2.8.1"</span>
</span></span><span><span>    <span>container_name</span>: <span>"dockerhub"</span>
</span></span><span><span>    <span>restart</span>: <span>"always"</span>
</span></span><span><span>    <span>volumes</span>:
</span></span><span><span>      - <span>"/etc/localtime:/etc/localtime"</span>
</span></span><span><span>      - <span>"./data:/var/lib/registry"</span>
</span></span><span><span>      - <span>"./config.yml:/etc/docker/registry/config.yml"</span>
</span></span><span><span>    <span>ports</span>:
</span></span><span><span>      - <span>"5000:5000"</span>
</span></span></code></pre></td></tr></tbody></table>

`config.yml`

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>version</span>: <span>0.1</span>
</span></span><span><span><span>storage</span>:
</span></span><span><span>  <span>filesystem</span>:
</span></span><span><span>    <span>rootdirectory</span>: <span>/var/lib/registry</span>
</span></span><span><span>  <span>delete</span>:
</span></span><span><span>    <span>enabled</span>: <span>true</span>
</span></span><span><span>  <span>maintenance</span>:
</span></span><span><span>    <span>uploadpurging</span>:
</span></span><span><span>      <span>enabled</span>: <span>true</span>
</span></span><span><span>      <span>age</span>: <span>168h</span>
</span></span><span><span>      <span>dryrun</span>: <span>false</span>
</span></span><span><span>      <span>interval</span>: <span>1m</span>
</span></span><span><span><span>http</span>:
</span></span><span><span>  <span>addr</span>: <span>0.0.0.0</span>:<span>5000</span>
</span></span><span><span><span>proxy</span>:
</span></span><span><span>  <span>remoteurl</span>: <span>https://registry-1.docker.io</span>
</span></span></code></pre></td></tr></tbody></table>

配置就是这么多配置，之后就可以直接pull镜像了

`docker pull xxxxx.com/prom/prometheus`

使用cloudflare作为cdn的话速度还是比较快的

其他的镜像仓库代理直接修改

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>proxy</span>:
</span></span><span><span>  <span>remoteurl</span>: <span>https://registry-1.docker.io</span>
</span></span></code></pre></td></tr></tbody></table>

这里就好了

欢迎关注我的博客 [www.bboy.app](https://www.bboy.app/ "www.bboy.app")

Have Fun