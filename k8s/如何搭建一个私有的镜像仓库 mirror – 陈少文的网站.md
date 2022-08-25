## 1\. 为什么需要一个私有的镜像仓库 mirror

-   公网限速
-   dockerhub 拉取限制频率
-   减少拉取镜像时间

## 2\. 创建一个 Registry 镜像加速服务

-   生成一个配置文件

```
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

但这样启动的服务只能作为 Registry 而不是 Mirror。Registry 是用来存储镜像，直接对外提供服务；Mirror 使从 Registry 拿到镜像数据再转给 Client。

-   添加 proxy 字段到 config.yaml 文件中

在 config.yaml 可以配置很多特征，比如验证秘钥、存储后端等。这里仅添加 proxy 字段，最终的 config.yaml 内容如下:

```
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
proxy:
  remoteurl: https://registry-1.docker.io
```

-   启动镜像服务

创建存储目录

启动服务

```
docker run -d -p 5000:5000 --restart=always --name mirror \
             -v `pwd`/config.yml:/etc/docker/registry/config.yml \
             -v `pwd`/data:/var/lib/registry \
             registry:2
```

查看服务

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker ps
</span></span><span><span>
</span></span><span><span>CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                    NAMES
</span></span><span><span>fc10fdef7e3f   registry:2   <span>"/entrypoint.sh /etc…"</span>   <span>1</span> minutes ago   Up <span>1</span> minutes   0.0.0.0:5000-&gt;5000/tcp   mirror
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

## 3\. Docker Daemon 配置镜像加速源

-   修改 Docker 的配置文件 daemon.json

在 /etc/docker/daemon.json 文件中，增加镜像源

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="json"><span><span><span>{</span> 
</span></span><span><span>    <span>"registry-mirrors"</span><span>:</span> <span>[</span><span>"http://127.0.0.1:5000"</span><span>],</span>
</span></span><span><span>    <span>"live-restore"</span><span>:</span> <span>true</span>
</span></span><span><span><span>}</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

live-restore 为 true 时，重启 Docker 不会影响正在运行的容器。

-   重新加载配置

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>systemctl daemon-reload
</span></span><span><span>systemctl restart docker
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   查看 Docker 配置

-   测试拉取镜像

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker pull centos
</span></span><span><span>docker pull ubuntu
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   查看缓存数据大小