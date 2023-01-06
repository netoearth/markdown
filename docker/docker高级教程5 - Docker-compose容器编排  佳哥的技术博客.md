## 五、Docker-compose 容器编排

### 5.1 Docker-compose 是什么

Docker-Compose 是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

### 5.2 能干什么

docker 建议我们每一个容器中只运行一个服务,因为 docker 容器本身占用资源极少,所以最好是将每个服务单独的分割开来但是这样我们又面临了一个问题？

如果我需要同时部署好多个服务,难道要每个服务单独写 Dockerfile 然后在构建镜像,构建容器,这样累都累死了,所以 docker 官方给我们提供了 docker-compose 多服务部署的工具

例如要实现一个 Web 微服务项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库 mysql 服务容器，redis 服务器，注册中心 eureka，甚至还包括负载均衡容器等等。。。。。。

Compose 允许用户通过一个单独的 docker-compose.yml 模板文件 （YAML 格式）来定义 一组相关联的应用容器为一个项目（project）。

可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose 解决了容器与容器之间如何管理编排的问题。

### 5.3 去哪里

#### 5.3.1 官网

[https://docs.docker.com/compose/compose-file/compose-file-v3/open in new window](https://docs.docker.com/compose/compose-file/compose-file-v3/)

#### 5.3.2 官网下载

[https://docs.docker.com/compose/install/open in new window](https://docs.docker.com/compose/install/)

#### 5.3.3 安装步骤

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

![52](https://luojia.work/assets/52-80a497b9.png)

#### 5.3.4 卸载步骤

```
sudo rm /usr/local/bin/docker-compose
```

### 5.4 Compose 核心概念

#### 5.4.1 一文件

#### 5.4.2 两要素

服务(service):

> 一个个应用容器实例，比如订单微服务、库存微服务、mysql 容器、nginx 容器或者 redis 容器。

工程(project):

> 由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

### 5.5 Compose 使用的三个步骤

1.  编写 Dockerfile 定义各个微服务应用并构建出对应的镜像文件
2.  使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务。
3.  最后，执行 docker-compose up 命令 来启动并运行整个应用程序，完成一键部署上线

### 5.6 Compose 常用命令

```
Compose 常用命令
docker-compose -h                           #  查看帮助
docker-compose up                           #  启动所有 docker-compose服务
docker-compose up -d                        #  启动所有 docker-compose服务 并后台运行
docker-compose down                         #  停止并删除容器、网络、卷、镜像。
docker-compose exec  yml里面的服务id                 # 进入容器实例内部  docker-compose exec  docker-compose.yml文件中写的服务id  /bin/bash
docker-compose ps                      # 展示当前docker-compose编排过的运行的所有容器
docker-compose top                     # 展示当前docker-compose编排过的容器进程
 
docker-compose logs  yml里面的服务id     #  查看容器输出日志
docker-compose config     #  检查配置
docker-compose config -q  #  检查配置，有问题才有输出
docker-compose restart   #  重启服务
docker-compose start     #  启动服务
docker-compose stop      #  停止服务
```