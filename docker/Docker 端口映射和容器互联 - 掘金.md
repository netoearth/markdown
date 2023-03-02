    



## 简介

在实践中，经常会碰到需要多个服务组件容器共同协作的情况，这往往需要多个容器之间能够互相访问到对方的服务。Docker除了通过网络访问外，还提供了两个很方便的功能来满足服务访问的基本需求：一个是允许映射容器内应用的服务端口到本地宿主主机；另一个是互联机制实现多个容器间通过容器名来快速访问。

## 端口映射与容器互联

### 端口映射实现容器访问

在启动容器的时候，如果不指定对应参数，在容器外部是无法通过网络来访问容器内的网络应用和服务的。

当容器中运行一些网络应用，要让外部访问这些应用时，可以通过`-P`或`-p`参数来指定端口映射。当使用`-P`（大写的）标记时，Docker会随机映射一个`49000～49900`的端口到内部容器开放的网络端口。`-p`（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有`IP:HostPort:ContainerPort`、`IP::ContainerPort`、`HostPort:ContainerPort`。

1.  映射所有接口地址 使用`HostPort:ContainerPort`格式本地的`5000`端口映射到容器的`5000`端口，可以执行如下命令：`docker run -p 5000:5000 training/webapp python app.py`。
2.  映射到指定地址的指定端口 可以使用`IP:HostPort:ContainerPort`格式指定映射使用一个特定地址，比如localhost地址`127.0.0.1`：`docker run -p 127.0.0.1:5000:5000 training/webapp python app.py`。
3.  映射到指定地址的任意端口 使用`IP::ContainerPort`绑定localhost的任意端口到容器的`5000`端口，本地主机会自动分配一个端口：`docker run -p 127.0.0.1::5000 training/webapp python app.py`。

> 使用`docker port`来查看当前映射的端口配置，也可以查看到绑定的地址。 容器有自己的内部网络和IP地址，使用`docker[container]inspect+容器ID`可以获取容器的具体信息。

### 互联机制实现便捷互访

容器的互联（linking）是一种让多个容器中的应用进行快速交互的方式。它会在源和接收容器之间创建连接关系，接收容器可以通过容器名快速访问到源容器，而不用指定具体的IP地址。

1.  自定义容器命名 连接系统依据容器的名称来执行。因此，首先需要自定义一个好记的容器命名。虽然当创建容器的时候，系统默认会分配一个名字，但自定义的命名，比较好记。使用`--name`标记可以为容器自定义命名：`docker run -d -P --name web training/webapp python app.py`。

> 容器的名称是唯一的。如果已经命名了一个叫`web`的容器，当你要再次使用`web`这个名称的时候，需要先用`docker rm`命令删除之前创建的同名容器。在执行`docker[container]run`的时候如果添加`--rm`标记，则容器在终止后会立刻删除。注意，`--rm`和`-d`参数不能同时使用。

2.  容器互联 使用`--link`参数可以让容器之间安全地进行交互。 下面先创建一个新的数据库容器： `docker run -d --name db -e MYSQL_ROOT_PASSWORD=Passw0rd! mysql` 然后创建一个新的`webapp`容器，并将它连接到`db`容器： `docker run -d -P --name webapp --link db:db debian:stretch` 此时，`db`容器和`web`容器建立互联关系。 `--link`参数的格式为`--link <name or id>:alias`，其中`name`是要链接的容器的名称，`alias`是源容器在`link`下的别名。

```
➜  ~ docker exec -it webapp bash  # 在运行中的container中执行命令 区别 docker start 唤醒暂停运行的container
root@24fb992918ba:/# ping db
PING db (172.17.0.2) 56(84) bytes of data.
64 bytes from db (172.17.0.2): icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from db (172.17.0.2): icmp_seq=2 ttl=64 time=0.112 ms
64 bytes from db (172.17.0.2): icmp_seq=3 ttl=64 time=0.113 ms
64 bytes from db (172.17.0.2): icmp_seq=4 ttl=64 time=0.111 ms
64 bytes from db (172.17.0.2): icmp_seq=5 ttl=64 time=0.117 ms
^C
--- db ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4132ms
rtt min/avg/max/mdev = 0.083/0.107/0.117/0.013 ms
root@24fb992918ba:/# env
DB_PORT=tcp://172.17.0.2:3306
DB_PORT_3306_TCP_ADDR=172.17.0.2
DB_ENV_MYSQL_MAJOR=8.0
HOSTNAME=24fb992918ba
DB_PORT_33060_TCP_PORT=33060
DB_PORT_3306_TCP=tcp://172.17.0.2:3306
DB_PORT_3306_TCP_PORT=3306
PWD=/
HOME=/root
DB_ENV_MYSQL_ROOT_PASSWORD=Passw0rd!
DB_PORT_33060_TCP_PROTO=tcp
DB_PORT_33060_TCP_ADDR=172.17.0.2
DB_PORT_33060_TCP=tcp://172.17.0.2:33060
TERM=xterm
SHLVL=1
DB_ENV_MYSQL_VERSION=8.0.16-1debian9
DB_NAME=/webapp/db
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
DB_ENV_GOSU_VERSION=1.7
DB_PORT_3306_TCP_PROTO=tcp
_=/usr/bin/env
root@24fb992918ba:/# 
复制代码
```

> `-e`, `--env list` Set environment variables 数据库Container shell access？`docker exec -it db bash`。详见：[hub.docker.com/\_/mysql?tab…](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2F_%2Fmysql%3Ftab%3Ddescription "https://hub.docker.com/_/mysql?tab=description") 使用`docker ps`来查看容器的连接。 `docker`官方已不推荐使用`docker run --link`来链接2个容器互相通信，随后的版本中会删除`--link`。

4.  另一些方法

详见：[dockone.io/article/115…](https://link.juejin.cn/?target=http%3A%2F%2Fdockone.io%2Farticle%2F1156 "http://dockone.io/article/1156")

方法一：基于bridge接口的使用案例

```
docker network create backend  # 创建一个网络，默认bridge
docker network ls  # 显示所有网络
docker run -itd --net=backend --name=server server_img /bin/bash  # 为了让两个容器间可以相互通信，那么只需要将他们放在同一网络或者子网中
复制代码
```

方法二：overlay技术来将不同宿主机上的Docker容器连接起来

```
docker network create -d overlay myapp  # 创建一个overlay网络
docker run --name ng1 --net=myapp -d nginx  # 加入网络
复制代码
```

### 端口映射和容器互联 小结

在生产环境中，网络方面的需求更加复杂和多变，包括跨主机甚至跨数据中心的通信，这时候往往就需要引入额外的机制，例如SDN（软件定义网络）或NFV（网络功能虚拟化）的相关技术。

### Cheat Sheet

```
# 端口映射和容器互联 关键命令回顾
docker run -p 5000:5000 training/webapp python app.py  # 本地所有地址的5000端口映射到容器的5000端口
docker run -p 127.0.0.1:5000:5000 training/webapp python app.py  # 指定映射使用一个特定地址
docker run -p 127.0.0.1::5000 training/webapp python app.py  # 绑定localhost的任意端口（容器启动时随机指定）到容器的5000端口
docker run -d -P --name web training/webapp python app.py  # 为容器自定义命名，容器的名称是唯一的
docker run -d --name db -e MYSQL_ROOT_PASSWORD=Passw0rd! mysql  # 创建一个新的数据库容器，-e, --env list Set environment variables
docker run -d -P --name webapp --link db:db training/webapp python app.py  # 创建一个新的webapp容器，并将它连接到db容器，--link参数的格式为--link <name or id>:alias
docker exec -it webapp bash  # 在运行中的container中执行命令
docker start -i webapp  # 以交互方式启动一个已经停止的容器
docker network create backend  # 创建一个网络
docker network ls  # 显示所有网络
docker run -itd --net=backend --name=server server_img /bin/bash  # 为了让两个容器间可以相互通信，那么只需要将他们放在同一网络或者子网中
docker network create -d overlay myapp  # 创建一个overlay网络
docker run --name ng1 --net=myapp -d nginx  # 加入网络
复制代码
```

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 赞

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏