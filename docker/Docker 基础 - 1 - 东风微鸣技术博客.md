## 镜像[](https://ewhisper.cn/posts/20855/#%E9%95%9C%E5%83%8F)

### 获取镜像[](https://ewhisper.cn/posts/20855/#%E8%8E%B7%E5%8F%96%E9%95%9C%E5%83%8F)

`docker pull`

### 查看镜像信息[](https://ewhisper.cn/posts/20855/#%E6%9F%A5%E7%9C%8B%E9%95%9C%E5%83%8F%E4%BF%A1%E6%81%AF)

`docker images`

`docker inspect <images id> # 获取镜像的详细信息`

### 搜寻镜像[](https://ewhisper.cn/posts/20855/#%E6%90%9C%E5%AF%BB%E9%95%9C%E5%83%8F)

`docker search`

### 删除镜像[](https://ewhisper.cn/posts/20855/#%E5%88%A0%E9%99%A4%E9%95%9C%E5%83%8F)

`docker rmi`

> 当一个镜像拥有多个标签，`docker rmi` 只是删除该镜像指定的标签，并不影响镜像文件  
> 当镜像只剩下一个标签时，再使用会彻底删除该镜像  
> 先删除该镜像的所有容器，再删除镜像

### 创建镜像[](https://ewhisper.cn/posts/20855/#%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F)

2 种方法：

-   基于已有镜像的 **容器** 创建
-   基于 Dockerfile 创建（推荐）

#### 基于已有镜像的容器创建[](https://ewhisper.cn/posts/20855/#%E5%9F%BA%E4%BA%8E%E5%B7%B2%E6%9C%89%E9%95%9C%E5%83%8F%E7%9A%84%E5%AE%B9%E5%99%A8%E5%88%9B%E5%BB%BA)

`docker commit`

> \-a: 作者信息  
> \-m: 提交信息  
> \-p 提交时暂停容器运行  
> \-c changelist

### 存出和载入镜像[](https://ewhisper.cn/posts/20855/#%E5%AD%98%E5%87%BA%E5%92%8C%E8%BD%BD%E5%85%A5%E9%95%9C%E5%83%8F)

**存出**：`sudo docker save -o ubuntu_16.04.tar ubuntu:16.04`

**载入**：`sudo docker load --input ubuntu_16.04.tar` 或者 `sudo docker load < ubuntu_16.04.tar`

> 导入镜像，以及其相关的元数据信息（包括标签等）

## 容器[](https://ewhisper.cn/posts/20855/#%E5%AE%B9%E5%99%A8)

### 创建容器[](https://ewhisper.cn/posts/20855/#%E5%88%9B%E5%BB%BA%E5%AE%B9%E5%99%A8)

使用交互式来运行容器：`sudo docker run -it ubuntu:latest /bin/bash`

**`docker run`在后台执行的标准操作：**

1.  检查本地是否存在指定的镜像， 不存在就从公有仓库下载
2.  利用镜像 **创建** 并**启动** 一个容器
3.  分配一个文件系统，并在只读的镜像层外面挂载一层 **可读写层**
4.  从宿主主机配置的网桥接口中 **桥接** 一个虚拟接口到容器中去。
5.  从地址池配置一个 **IP 地址** 给容器
6.  执行用户指定的 **应用程序**
7.  执行完毕后 **容器被终止**

#### 守护态运行[](https://ewhisper.cn/posts/20855/#%E5%AE%88%E6%8A%A4%E6%80%81%E8%BF%90%E8%A1%8C)

`sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"`

通过 `docker logs` 命令获取容器的输出信息：`sudo docker -tf logs 2855b4d76ccb`

> \-t: 打印时间戳  
> \-f: 刷新日志到最后  
> 2855b4d76ccb: 容器 id

### 终止容器[](https://ewhisper.cn/posts/20855/#%E7%BB%88%E6%AD%A2%E5%AE%B9%E5%99%A8)

`sudo docker stop 2855b4d76ccb`

查看所有容器的信息：`sudo docker ps -a`

处于终止状态的容器，可以通过 `docker start` 命令来重新启动：`sudo docker start 2855b4d76ccb`

重启容器：`sudo docker restart 2855b4d76ccb`

### 进入容器[](https://ewhisper.cn/posts/20855/#%E8%BF%9B%E5%85%A5%E5%AE%B9%E5%99%A8)

#### attach 指令[](https://ewhisper.cn/posts/20855/#attach%20%E6%8C%87%E4%BB%A4)

`sudo docker attach 2855b4d76ccb`

> 当多个窗口同时 attach 到同一个容器时， 所有窗口会同步显示。当某个窗口因命令阻塞时，其他窗口也无法执行操作。

#### exec 命令[](https://ewhisper.cn/posts/20855/#exec-%20%E5%91%BD%E4%BB%A4)

`sudo docker exec -ti 2855b4d76ccb /bin/bash`

### 删除容器[](https://ewhisper.cn/posts/20855/#%E5%88%A0%E9%99%A4%E5%AE%B9%E5%99%A8)

`sudo docker rm 2855b4d76ccb`

> `-f` 强制删除运行中的容器  
> `-l` 删除容器链接，但保留容器  
> `-v` 删除容器挂载的数据卷

### 导入 / 导出容器[](https://ewhisper.cn/posts/20855/#%E5%AF%BC%E5%85%A5%20-%20%E5%AF%BC%E5%87%BA%E5%AE%B9%E5%99%A8)

#### 导出[](https://ewhisper.cn/posts/20855/#%E5%AF%BC%E5%87%BA)

`sudo docker export 2855b4d76ccb > test.tar`

> 可以将这些文件传输到其他机器上， 在其他机器上通过导入命令实现容器迁移

#### 导入[](https://ewhisper.cn/posts/20855/#%E5%AF%BC%E5%85%A5)

`cat test.tar | sudo docker import - test/ubuntu:v1.0`

> `docker load` 导入镜像存储文件到本地的镜像库  
> `docker import` 导入一个容器 **快照** 到本地镜像库

> 区别：
> 
> **容器快照** 文件将丢弃所有的 **历史记录和元数据信息**（仅保存容器当时的快照状态)，导入时可以重新指定标签等元数据信息。  
> **镜像存储文件** 将保存完整记录， 体积也大。

## 仓库[](https://ewhisper.cn/posts/20855/#%E4%BB%93%E5%BA%93)

### Docker Hub[](https://ewhisper.cn/posts/20855/#Docker-Hub)

#### 登录[](https://ewhisper.cn/posts/20855/#%E7%99%BB%E5%BD%95)

`docker login`

#### 搜索[](https://ewhisper.cn/posts/20855/#%E6%90%9C%E7%B4%A2)

`docker search`

#### docker tag[](https://ewhisper.cn/posts/20855/#docker-tag)

`sudo docker tag ubuntu:14.04 10.0.2.2:5000/test`

#### 自动创建[](https://ewhisper.cn/posts/20855/#%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA)

步骤：

1.  登录 Docker Hub, 连接 GitHub 到 Docker
2.  在 Docker Hub 中配置自动创建：[https://hub.docker.com/add/automated-build/caseycui/](https://hub.docker.com/add/automated-build/caseycui/)
3.  选取一个目标网站项目（需要含 Dockerfile）和分支
4.  指定 Dockerfile 的位置，并提交创建
5.  之后，可以在 Docker Hub 的“自动创建”页面中跟踪每次创建的状态。

### 创建和使用私有仓库[](https://ewhisper.cn/posts/20855/#%E5%88%9B%E5%BB%BA%E5%92%8C%E4%BD%BF%E7%94%A8%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93)

#### 使用 registry 镜像创建私有仓库[](https://ewhisper.cn/posts/20855/#%E4%BD%BF%E7%94%A8%20-registry-%20%E9%95%9C%E5%83%8F%E5%88%9B%E5%BB%BA%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93)

`sudo docker -d -p 5000:5000 -v /opt/docker/registry/:/tmp/registry registry`

监听端口映射到 5000，docker 容器中的 `/tmp/registry` 被映射到本地的 `/opt/docker/registry/` 上。

#### 管理私有仓库镜像[](https://ewhisper.cn/posts/20855/#%E7%AE%A1%E7%90%86%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93%E9%95%9C%E5%83%8F)

-   修改 tag 的 REGISTRYHOST `sudo docker tag ubuntu:latest 172.17.0.1:5000/test`
-   使用 docker push 上传 `sudo docker push 172.17.0.1:5000/test`
-   使用 docker pull 下载 `sudo docker pull 172.17.0.1:5000/test`

## 数据管理[](https://ewhisper.cn/posts/20855/#%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86)

两种方式:

-   数据卷(Data Volumes)
-   数据卷容器(Data Volume Containers)

### 数据卷[](https://ewhisper.cn/posts/20855/#%E6%95%B0%E6%8D%AE%E5%8D%B7)

特性:

-   可以在容器间共享和重用
-   对数据卷的修改会立马生效
-   对数据卷的更新, 不会影响容器
-   卷会一直存在, 直到没有容器使用

> 类似 Linux 的 mount 操作

#### 挂载一个主机目录作为数据卷[](https://ewhisper.cn/posts/20855/#%E6%8C%82%E8%BD%BD%E4%B8%80%E4%B8%AA%E4%B8%BB%E6%9C%BA%E7%9B%AE%E5%BD%95%E4%BD%9C%E4%B8%BA%E6%95%B0%E6%8D%AE%E5%8D%B7)

`sudo docker run -v /src/webapp:/opt/webapp training/webapp python app.py`

加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录.

> 主机目录必须是绝对路径
> 
> 默认权限是 **读写** (rw), 可以设置为 **只读**(ro) `/src/webapp:/opt/webapp:ro`

### 数据卷容器[](https://ewhisper.cn/posts/20855/#%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AE%B9%E5%99%A8)

> **数据卷容器** 其实就是普通的 **容器**, 专门用它提供数据卷供其他容器挂载使用.

1.  创建一个数据卷容器: `sudo docker run -it -v /dbdata --name dbdata ubuntu`
    
2.  在其他容器中使用 `volumes-from` 来挂载 _dbdata_ 容器中的数据卷.
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code> <p> sudo docker run -it --volumes-from dbdata --name db1 ubuntu</p><p>  sudo docker run -it --volumes-from dbdata --name db2 ubuntu</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
    > 3 个容器任何一方在该目录写入, 其他容器都可以看到。  
    > 如果删除了容器, **数据卷** 并不会自动删除. 如果要删除数据卷, **必须在删除最后一个挂载着它的容器时显式使用 `docker rm -v` 命令来指定同时删除关联的容器**
    

### 利用数据卷容器迁移数据[](https://ewhisper.cn/posts/20855/#%E5%88%A9%E7%94%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AE%B9%E5%99%A8%E8%BF%81%E7%A7%BB%E6%95%B0%E6%8D%AE)

#### 备份[](https://ewhisper.cn/posts/20855/#%E5%A4%87%E4%BB%BD)

`sudo docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata`

#### 恢复[](https://ewhisper.cn/posts/20855/#%E6%81%A2%E5%A4%8D)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash<br>sudo docker run --volumes-from dbdata2 -v $(<span>pwd</span>):/backup busybox tar xvf /backup/backup.tar<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 网络基础配置[](https://ewhisper.cn/posts/20855/#%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80%E9%85%8D%E7%BD%AE)

### 端口映射实现访问容器[](https://ewhisper.cn/posts/20855/#%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84%E5%AE%9E%E7%8E%B0%E8%AE%BF%E9%97%AE%E5%AE%B9%E5%99%A8)

`sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py`

> \-P: 映射到随机端口

#### 查看端口映射配置[](https://ewhisper.cn/posts/20855/#%E6%9F%A5%E7%9C%8B%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84%E9%85%8D%E7%BD%AE)

`sudo docker port loving_montalcini`

### 容器互联实现容器间通信[](https://ewhisper.cn/posts/20855/#%E5%AE%B9%E5%99%A8%E4%BA%92%E8%81%94%E5%AE%9E%E7%8E%B0%E5%AE%B9%E5%99%A8%E9%97%B4%E9%80%9A%E4%BF%A1)

容器的连接 (linking) 系统, 它会在源和接收容器之间创建一个隧道, 接收容器可以看到源容器指定的信息.

连接系统依据容器的名称来执行.

> 在执行 `docker run` 的时候如果添加 `--rm` 标记, 则容器在终止后会立即删除. `--rm` 和 `-d` 无法同时使用.

#### 容器互联[](https://ewhisper.cn/posts/20855/#%E5%AE%B9%E5%99%A8%E4%BA%92%E8%81%94)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>sudo docker run -d --name db training/postgres  <span># 创建一个新的数据库容器</span><br>sudo docker run -d -P --name web --link db:db training/webapp python app.py  <span># --link name:alias alias 是连接的别名</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

Docker 通过 2 种方式为容器公开连接信息:

-   环境变量
-   更新 `/etc/hosts` 文件

使用 `env` 命令查看 web 容器的环境变量:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code>$ docker <span>exec</span> web env<br>PATH=/usr/<span>local</span>/sbin:/usr/<span>local</span>/bin:/usr/sbin:/usr/bin:/sbin:/bin<br>HOSTNAME=62b19d3b5add<br>DB_PORT=tcp://172.17.0.2:5432<br>DB_PORT_5432_TCP=tcp://172.17.0.2:5432<br>DB_PORT_5432_TCP_ADDR=172.17.0.2<br>DB_PORT_5432_TCP_PORT=5432<br>DB_PORT_5432_TCP_PROTO=tcp<br>DB_NAME=/web/db<br>DB_ENV_PG_VERSION=9.3<br>HOME=/root<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

`/etc/hosts` 文件:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code>$ docker <span>exec</span> web cat /etc/hosts<p>127.0.0.1       localhost<br>::1     localhost ip6-localhost ip6-loopback<br>fe00::0 ip6-localnet<br>ff00::0 ip6-mcastprefix<br>ff02::1 ip6-allnodes<br>ff02::2 ip6-allrouters</p><p>172.17.0.2      db 4288c4f9ad47</p><p> 172.17.0.3      62b19d3b5add  <span># 本容器</span></p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

也可以通过 ping 来测试:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code>$ docker <span>exec</span> web ping 172.17.0.2<br>PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.<br>64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.083 ms<br>64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.054 ms<br>64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.054 ms<br>64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.053 ms<br>64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.055 ms<br>64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.057 ms<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 使用 Dockerfile 创建镜像[](https://ewhisper.cn/posts/20855/#%E4%BD%BF%E7%94%A8%20-Dockerfile-%20%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F)

### 基本结构[](https://ewhisper.cn/posts/20855/#%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84)

-   基础镜像信息
-   维护者信息
-   镜像操作指令
-   容器启动时执行指令

### 指令[](https://ewhisper.cn/posts/20855/#%E6%8C%87%E4%BB%A4)

#### FROM[](https://ewhisper.cn/posts/20855/#FROM)

第一条指令必须为 FROM 指令

#### MAINTAINER(已弃用)[](https://ewhisper.cn/posts/20855/#MAINTAINER-%20%E5%B7%B2%E5%BC%83%E7%94%A8)

制定维护者信息. 以后可以用 `LABEL maintainer="CaseyCui cuikaidong@foxmail.com"`

#### RUN[](https://ewhisper.cn/posts/20855/#RUN)

两种格式:

-   `RUN <command>` _bash_格式, 命令在 bash 中运行, 默认在 Linux 是 `/bin/sh -c` 在 windows 上是 `cmd /S /C`
-   `RUN ["executable", "param1", "param2"]` (用 `docker exec` 执行)

每条 RUN 指令将在当前镜像基础上执行指定命令, 并提交为新的镜像.

##### 最佳实践:[](https://ewhisper.cn/posts/20855/#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)

**apt-get 安装:**

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>RUN</span><span> apt-get update &amp;&amp; apt-get install -y --no-install-recommends \</span><br><span>    aufs-tools \</span><br><span>    automake \</span><br><span>    build-essential \</span><br><span>    curl \</span><br><span>    dpkg-sig \</span><br><span>    libcap-dev \</span><br><span>    libsqlite3-dev \</span><br><span>    mercurial \</span><br><span>    reprepro \</span><br><span>    ruby1.9.1 \</span><br><span>    ruby1.9.1-dev \</span><br><span>    s3cmd=1.1.* \</span><br><span>&amp;&amp; rm -rf /var/lib/apt/lists/*</span><br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

> `apt-get update && apt-get install` 合用
> 
> `apt-get install -y --no-install-recommends` 不安装其他推荐的包, `-no-install-suggests` 也可以加上
> 
> `ruby1.9.1 s3cmd=1.1.*` 安装指定版本的包
> 
> `rm -rf /var/lib/apt/lists/*` 删除软件包残留

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br></pre></td><td><pre><code><span>## a few minor docker-specific tweaks</span><br><span>## see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap</span><br><span>RUN</span><span> <span>set</span> -xe \</span><br><span>    \</span><br><span><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L40-L48</span></span><br>    &amp;&amp; echo <span>'#!/bin/sh'</span> &gt; /usr/sbin/policy-rc.d \<br>    &amp;&amp; echo <span>'exit 101'</span> &gt;&gt; /usr/sbin/policy-rc.d \<br>    &amp;&amp; chmod +x /usr/sbin/policy-rc.d \<br>    \<br><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L54-L56</span><br>    &amp;&amp; dpkg-divert --local --rename --<span>add</span><span> /sbin/initctl \</span><br><span>    &amp;&amp; cp -a /usr/sbin/policy-rc.d /sbin/initctl \</span><br><span>    &amp;&amp; sed -i <span>'s/^exit.*/exit 0/'</span> /sbin/initctl \</span><br><span>    \</span><br><span><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L71-L78</span></span><br>    &amp;&amp; echo <span>'force-unsafe-io'</span> &gt; /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \<br>    \ <br><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L85-L105</span><br>    &amp;&amp; echo <span>'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };'</span> &gt; /etc/apt/apt.conf.d/docker-clean \<br>    &amp;&amp; echo <span>'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };'</span> &gt;&gt; /etc/apt/apt.conf.d/docker-clean \<br>    &amp;&amp; echo <span>'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";'</span> &gt;&gt; /etc/apt/apt.conf.d/docker-clean \<br>    \<br><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L109-L115</span><br>    &amp;&amp; echo <span>'Acquire::Languages "none";'</span> &gt; /etc/apt/apt.conf.d/docker-no-languages \<br>    \ <br><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L118-L130</span><br>    &amp;&amp; echo <span>'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";'</span> &gt; /etc/apt/apt.conf.d/docker-gzip-indexes \<br>    \ <br><span>## https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L134-L151</span><br>    &amp;&amp; echo <span>'Apt::AutoRemove::SuggestsImportant "false";'</span> &gt; /etc/apt/apt.conf.d/docker-autoremove-suggests<br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

> `set -xe` 调试模式, 返回非 0 (即不成功) 就退出

#### CMD[](https://ewhisper.cn/posts/20855/#CMD)

支持三种格式:

-   `CMD ["executable","param1","param2"]` 使用 `docker exec` 执行, **推荐方式**
-   `CMD ["param1","param2"]` 作为 _ENTRYPOINT_ 的默认参数
-   `CMD command param1 param2` _bash_

指定启动容器时执行的命令, 每个 Dockerfile 只能有一条 CMD 命令.

#### EXPOSE[](https://ewhisper.cn/posts/20855/#EXPOSE)

`EXPOSE 80 8443`

暴露的端口号, 供互联系统使用. 在启动时可以通过 `-P` 或 `-p` 来指定映射

#### ENV[](https://ewhisper.cn/posts/20855/#ENV)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>ENV</span> PG_MAJOR <span>9.3</span><br><span>ENV</span> PG_VERSION <span>9.3</span>.<span>4</span><br><span>RUN</span><span> curl -SL http://example.com/postgres-<span>$PG_VERSION</span>.tar.xz | tar -xJC /usr/src/postgress &amp;&amp; …</span><br><span>ENV</span> PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH<br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

#### ADD[](https://ewhisper.cn/posts/20855/#ADD)

**推荐** 只在 `src` 为 tar 文件时 (会自动解压) 使用. 其他时候使用 COPY. 不推荐使用 URL 的方式.

#### COPY[](https://ewhisper.cn/posts/20855/#COPY)

`COPY <src> <dest>` 当使用本例目录为 src 时, 推荐使用 COPY

#### ENTRYPOINT[](https://ewhisper.cn/posts/20855/#ENTRYPOINT)

两种格式:

-   `ENTRYPOINT ["executable", "param1", "param2"]` _exec_ 格式
-   `ENTRYPOINT command param1 param2` _bash_ 格式

#### VOLUME[](https://ewhisper.cn/posts/20855/#VOLUME)

创建一个可以从本地主机或其他容器挂载的挂载点, 一般用来存放数据库和需要保持的数据等.

#### USER[](https://ewhisper.cn/posts/20855/#USER)

指定运行容器时的用户名或 UID, 后续的 `RUN` 也会使用指定用户.

当服务不需要管理员权限时, 可以通过该命令指定运行用户.

要临时获取管理员权限推荐使用 `gosu` , 不推荐`sudo`

#### WORKDIR[](https://ewhisper.cn/posts/20855/#WORKDIR)

为后续的 `RUN CMD ENTRYPOINT` 指令配置工作目录.

#### ONBUILD[](https://ewhisper.cn/posts/20855/#ONBUILD)

`ONBUILD [Dockerilfe 的指令]`

配置当所创建的镜像作为其他新创建镜像的基础镜像时, 所执行的操作指令.

使用 `ONBUILD` 指令的镜像, 推荐在 tag 中注明, 如 _ruby:1.9-onbuild_

### 创建镜像[](https://ewhisper.cn/posts/20855/#%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F%20-2)

`docker build [选项] 路径`

实现:

1.  读取指定路径下 (包括子目录) 的 Dockerfile
2.  将该路径下所有内容发送给 Docker 服务端
3.  由服务端来创建镜像

> 一般建议放置 Dockerfile 的目录为空目录

可以通过 _.dockerignore_ 文件来让 Docker 忽略路径下的目录和文件.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code>## comment<br>*/temp*<br>*/*/temp*<br>temp?<br></code><p><i></i>.DOCKERIGNORE</p></pre></td></tr></tbody></table>

指令示例: `sudo docker build -t build_repo/first_image /tmp/docker_builder/`