> 作者：苏奕嘉｜SelectDB 生态研发工程师

Docker 容器化部署是当前最常见的部署方式之一，具有创建简单、快速部署、移植性强等特点，可极大节省应用开发、测试和部署时间，一次构建，随处运行。

本教程可指导有快速部署测试和 Docker 学习需求的同学，快速进行部署单节点 Apache Doris 集群或伪分布式 Apache Doris 集群。同时介绍如何自主的构建 Apache Doris 的 Docker 生态内容，如 Dockerfile 书写、Docker Images 构建、Docker-Compose 编排等内容。

本教程所有模块的构建背景都是以**快速部署和学习为目，利用 1FE 1BE 规格**来撰写的，如有 NFE MBE 需求的同学可自行在此基础上进行编排改写。

本教程不再赘述 Docker 安装、 Dockerfile 、Docker-Compose 等编写时脚本内部使用的相应命令作用，如有需要可参考 Docker 官方文档。

## Dockerfile

### 编写思路

镜像的制作应当注意以下几点：

1.  基础父镜像最好选用经过 Docker-Hub 认证的官方镜像，如有 `DOCKER OFFICIAL IMAGE` 标签的镜像提供者，一般是官方认证过没有问题的镜像源。
2.  基础父镜像选用时尽可能以能最小可用环境的原则进行选择，即选择时以满足基础必备环境的同时，镜像尽可能以最小的原则进行挑选，不要直接使用诸如完整的 CentOS 镜像、Ubuntu 镜像等提供完备功能的镜像，如果使用后者，则我们构建出的包会很大，不利于使用和传播，也会造成磁盘空间浪费。
3.  最好不要高度封装，若只是想提供一个软件的原生镜像，最好的构建方式是 `最小可用环境` + `软件本身` ，期间不夹杂其他的逻辑和功能处理，这样可以更原子化的进行镜像编排以及后续的维护和更新操作。
4.  单独用一个脚本制成的镜像完成后续对其他镜像的各种业务操作和运维操作。
5.  构建 Image 和使用 Docker-Compose 时，需要用官方完整的 Docker 程序，使用官方提供的脚本进行安装最佳。

根据 Apache Doris 的简洁架构（FE + BE），以及不依赖其他组件和环境的特性，制作时应将 FE 和 BE 分离制作为两个镜像，再制作 Register 镜像来操控 FE 和 BE 镜像，这样可以原子化的制作可用镜像，也便于后期的维护和更新。

### Docker 服务部署（ CentOS 系统和 Ubuntu 系统）

_Docker_ 是一个开源的应用容器引擎，该教程的环境基础为 _Docker_。

本小节只以 CentOS 系统和 Ubuntu 系统为例，Windows 系统较繁琐不再赘述，MacOS 系统官方下载安装即可，简易程度极高，不再赘述。

1.  卸载老旧的版本（未安装过可略）

```
# CentOS
yum remove docker docker-engine docker.io
# Ubuntu
sudo apt-get remove docker docker-engine docker.io
```

1.  安装最新的 Docker

```
curl -sSL https://get.docker.com/ | sh 
```

1.  启动并添加开机自启

```
sudo systemctl start docker #启动docker
sudo systemctl enable docker #加入开机自启动
```

1.  检查安装

```
docker version
```

1.  若出现 Client 和 Server 两部分内容，说明安装成功。

### FE Dockerfile

FE 是 Java 程序，所以我们选用 `Openjdk` 提供的 JDK8 最新的 `8u342-jdk` 镜像作为基础父镜像，提供两个版本：

**以下脚本需命名为 Dockerfile ！**

-   使用本地二进制包构建

```
# 选择基础镜像
FROM openjdk:8u342-jdk

# 根据自己需求进行替换，路径使用相对路径
ARG package_url=本地二进制包路径，与 package_name 拼接得到完整目录
# 例如 ARG package_url=./

ARG package_name=二进制包文件名
# 例如 ARG package_name=apache-doris-1.1.1-bin-x86.tar.gz

ARG package_path=解压后文件目录名
# 例如 ARG package_path=apache-doris-1.1.1-bin-x86

# 设置环境变量
ENV JAVA_HOME="/usr/local/openjdk-8/" \
    PATH="/usr/local/apache-doris/fe/bin:$PATH"

# 下载软件至镜像内，可根据需要替换
COPY $package_url/$package_name /usr/local/

# 部署软件
RUN tar -zxvf /usr/local/$package_name -C /usr/local/ && \
    mv /usr/local/$package_path /usr/local/apache-doris && \
    rm -rf /usr/local/$package_name /usr/local/apache-doris/be /usr/local/apache-doris/udf /usr/local/apache-doris/apache_hdfs_broker

CMD ["bash"]
```

-   使用 URL 下载构建

```
# 选择基础镜像
FROM openjdk:8u342-jdk

# 根据自己需求进行替换
ARG package_url=远端URL下载地址
# 例如 ARG package_url=https://mirrors.tuna.tsinghua.edu.cn/apache/doris/1.1/1.1.1-rc03/apache-doris-1.1.1-bin-x86.tar.gz

ARG package_name=二进制包文件名
# 例如 ARG package_name=apache-doris-1.1.1-bin-x86.tar.gz

ARG package_path=解压后文件目录名
# 例如 ARG package_path=apache-doris-1.1.1-bin-x86

# 设置环境变量
ENV JAVA_HOME="/usr/local/openjdk-8/" \
    PATH="/usr/local/apache-doris/fe/bin:$PATH"

# 部署软件
RUN curl -o /usr/local/apache-doris.tar.gz $package_url && \
    tar -zxvf /usr/local/apache-doris.tar.gz -C /usr/local/ && \
    mv /usr/local/$package_path /usr/local/apache-doris && \
    rm -rf /usr/local/$package_name /usr/local/apache-doris/be /usr/local/apache-doris/udf /usr/local/apache-doris/apache_hdfs_broker

CMD ["bash"]
```

构建 Apache-Doris-FE-Docker-Image：

```
# 镜像名为用户自定义的名称
docker build . -t 镜像名 
```

### BE Dockerfile

BE Docker Image 构建，与 FE 大同小异，只需要更改基础父镜像和删除内容即可，修改如下：

1.  基础父镜像修改为 bitnami/minideb:latest
2.  ENV PATH 中 fe 修改为 be，删除 Java 环境变量，添加 broker 环境变量
3.  最后清理目录中，将 be 修改为 fe

-   使用本地二进制包构建

```
# 选择基础镜像
FROM bitnami/minideb:latest

# 根据自己需求进行替换
ARG package_url=本地二进制包路径，与 package_name 拼接得到完整目录
# 例如 ARG package_url=./

ARG package_name=二进制包文件名
# 例如 ARG package_name=apache-doris-1.1.1-bin-x86.tar.gz

ARG package_path=解压后文件目录名
# 例如 ARG package_path=apache-doris-1.1.1-bin-x86

# 设置环境变量
ENV PATH="/usr/local/apache-doris/be/bin:/usr/local/apache-doris/apache_hdfs_broker/bin:$PATH"

# 下载软件至镜像内，可根据需要替换
COPY $package_url/$package_name /usr/local/

# 部署软件
RUN tar -zxvf /usr/local/$package_name -C /usr/local/ && \
    mv /usr/local/$package_path /usr/local/apache-doris && \
    rm -rf /usr/local/$package_name /usr/local/apache-doris/fe /usr/local/apache-doris/udf

CMD ["bash"]
```

-   使用 URL 下载构建

```
# 选择基础镜像
FROM bitnami/minideb:latest

# 根据自己需求进行替换
ARG package_url=远端URL下载地址
# 例如 ARG package_url=https://mirrors.tuna.tsinghua.edu.cn/apache/doris/1.1/1.1.1-rc03/apache-doris-1.1.1-bin-x86.tar.gz

ARG package_name=二进制包文件名
# 例如 ARG package_name=apache-doris-1.1.1-bin-x86.tar.gz

ARG package_path=解压后文件目录名
# 例如 ARG package_path=apache-doris-1.1.1-bin-x86

# 设置环境变量
ENV PATH="/usr/local/apache-doris/be/bin:/usr/local/apache-doris/apache_hdfs_broker/bin:$PATH"

# 部署软件
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    curl -o /usr/local/apache-doris.tar.gz $package_url && \
    tar -zxvf /usr/local/apache-doris.tar.gz -C /usr/local/ && \
    mv /usr/local/$package_path /usr/local/apache-doris && \
    rm -rf /usr/local/$package_name /usr/local/apache-doris/fe /usr/local/apache-doris/udf

CMD ["bash"]
```

构建 Apache-Doris-BE-Docker-Image：

```
docker build . -t 镜像名 
```

### Register Dockerfile

构建 Register Docker Image 时，主是是通过 Docker-Compose 来调度管控 Doris 的 FE 和 BE 启停及注册的脚本。

以下演示皆为最基础的脚本版本，脚本健壮性并不是很高，可根据自己需要进行定制化开发，比如增加流程判断，确定成功失败等。

-   初始化 FE 脚本
-   init\_fe.sh

```
#!/bin/bash
echo "Start initializing Apache-Doris FE!"
cd /usr/local/apache-doris/
perl -pi -e "s|# priority_networks = 10.10.10.0/24;192.168.0.0/16|priority_networks = 172.20.80.0/16|g" /usr/local/apache-doris/fe/conf/fe.conf
start_fe.sh --daemon
echo "Apache-Doris FE initialized successfully!"
```

-   初始化 BE 脚本
-   init\_be.sh

```
#!/bin/bash
echo "Start initializing Apache-Doris BE!"
cd /usr/local/apache-doris/
perl -pi -e "s|# priority_networks = 10.10.10.0/24;192.168.0.0/16|priority_networks = 172.20.80.0/16|g" /usr/local/apache-doris/be/conf/be.conf
start_be.sh --daemon
echo "Apache-Doris BE initialized successfully!"
```

-   注册 BE 至 FE 脚本
-   [register.sh](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fregister.sh)

```
#!/bin/sh
echo "Welcome to Apache-Doris Docker-Compose to quickly build test images!"
echo "Start modifying the configuration files of FE and BE and run FE and BE!"
docker cp /root/init_fe.sh doris-fe:/root/init_fe.sh
docker exec doris-fe bash -c "/root/init_fe.sh"
docker cp /root/init_be.sh doris-be:/root/init_be.sh
docker exec doris-be bash -c "/root/init_be.sh"
sleep 30
echo "Get started with the Apache Doris registration steps!"
mysql -h 172.20.80.2 -P 9030 -uroot -e "ALTER SYSTEM ADD BACKEND "172.20.80.3:9050";"
echo "The initialization task of Apache-Doris has been completed, please start to experience it!"
```

Register Dockerfile 内容如下：

```
# Base Images 基础镜像
FROM bash:latest

# 添加脚本文件
ADD *.sh /root/

# 设置工作目录
WORKDIR /root/

# 设置镜像源及基础环境
RUN /bin/sh -c echo 'http://mirrors.sjtug.sjtu.edu.cn/alpine/v3.15/main' > /etc/apk/repositories && \
    echo 'http://mirrors.sjtug.sjtu.edu.cn/alpine/v3.15/community' >> /etc/apk/repositories && \
    apk update && \
    apk add --no-cache mysql-client docker && \
    rm -rf /usr/bin/docker-proxy /usr/bin/docker-init /usr/bin/dockerd && \
    chmod 755 *.sh
```

## Docker-Compose

Compose 是⽤于定义和运⾏多容器 Docker 应⽤程序的⼯具，我们使用它来完成对 Doris 服务的编排及初始化工作。

Docker-compose 的脚本文件命名应为 docker-compose.yml

这里有几个参数需要着重讲一下：

1.  Apache Doris 是 IP 敏感和存储敏感的有状态服务，所以在设计 Compose 时要将此作为准则之一。
2.  解决上述问题的方案是通过设置 networks 自定义一个独有的网卡，指定每个 Docker 容器节点的容器 IP 地址。
3.  由于该教程为 1FE 1BE 的快速部署服务制作教程，持久化挂载能力将在脚本内演示，如有需要可自行添加映射卷。
4.  Compose 的服务模块执行是可以配置依赖顺序的，这样可以有效的防止有些脚本关系因为依赖错误而执行失败。

这里演示两个脚本，一个 1FE 1BE（添加数据持久化映射关系），一个 1FE 3BE（未添加数据持久化映射关系），如有需要可依照该基础进行自行修改适配。

-   **1FE 1BE**

```
version: '3'
services:
  docker-fe:
    image: "FE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-fe"
    container_name: "doris-fe"
    tty: true
    hostname: "fe"
    restart: always
    ports:
      - 8030:8030
      - 9030:9030
    networks:
      doris_net:
        ipv4_address: 172.20.80.2
  docker-be:
    image: "BE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-be"
    container_name: "doris-be"
    tty: true
    hostname: "be"
    restart: always
    ports:
      - 8041:8040
      - 9001:9000
      - 9051:9050
    networks:
      doris_net:
        ipv4_address: 172.20.80.3
  register:
    image: "Register-Image" # 如 "apache/doris:register"
    container_name: "doris-register"
    hostname: "register"
    privileged: true
    command: ["sh","-c","/root/register.sh"]
    depends_on:
      - docker-fe
      - docker-be-01
    volumes: 
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      doris_net:
        ipv4_address: 172.20.80.4
networks:
  doris_net:
    ipam:
      config:
      - subnet: 172.20.80.0/16
```

-   **1FE 3BE**

```
version: '3'
services:
  docker-fe:
    image: "FE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-fe"
    container_name: "doris-fe"
    tty: true
    hostname: "fe"
    restart: always
    ports:
      - 8030:8030
      - 9030:9030
    networks:
      doris_net:
        ipv4_address: 172.20.80.2
  docker-be-01:
    image: "BE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-be"
    container_name: "doris-be-01"
    tty: true
    hostname: "be-01"
    restart: always
    ports:
      - 8041:8040
      - 9001:9000
      - 9051:9050
    networks:
      doris_net:
        ipv4_address: 172.20.80.3
  docker-be-02:
    image: "BE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-be"
    container_name: "doris-be-02"
    tty: true
    hostname: "be-02"
    restart: always
    ports:
      - 8042:8040
      - 9002:9000
      - 9052:9050
    networks:
      doris_net:
        ipv4_address: 172.20.80.4
  docker-be-03:
    image: "BE-Docker-Image" # 如 "apache/doris:1.1.2-rc06-be"
    container_name: "doris-be-03"
    tty: true
    hostname: "be-03"
    restart: always
    ports:
      - 8043:8040
      - 9003:9000
      - 9053:9050
    networks:
      doris_net:
        ipv4_address: 172.20.80.5
  # 请看注意事项！
  register:
    image: "Register-Image" # 如 "apache/doris:register"
    container_name: "doris-register"
    hostname: "register"
    privileged: true
    command: ["sh","-c","/root/register.sh"]
    depends_on:
      - docker-fe
      - docker-be-01
      - docker-be-02
      - docker-be-03
    volumes: 
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      doris_net:
        ipv4_address: 172.20.80.6
networks:
  doris_net:
    ipam:
      config:
      - subnet: 172.20.80.0/16
```

-   **注意，由于我们 Register 镜像里的注册及初始化脚本都只写了 1FE 1BE 的情况，所以不适用 NFE MBE 的 Compose，如有需要需自行制作 Register 镜像**

## 快速使用

制作好镜像以后，就可以快速使用了，步骤如下：

### 1\. Docker-Compose 服务部署

_Compose_ 是用于定义和运行多容器 _Docker_ 应用程序的工具。

1.  下载最新版的 docker-compose 文件
    
    1.  GitHub 拉取
    2.  ```
        sudo curl -L https://github.com/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
        ```
        
    3.  国内 DaoCloud 拉取
    4.  ```
        sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
        ```
        

1.  添加执行权限

```
sudo chmod +x /usr/local/bin/docker-compose
```

1.  检查安装结果

```
docker-compose --version
> docker-compose version 1.25.1
```

### 2\. 拉取或者制作 Compose 脚本

以下为 1 FE 1 BE 单节点部署 Apache Doris 1.1.2 版本的脚本，如果有需要，可根据自身调整。

> 如需切换版本号，将 docker-fe 和 docker-be 的镜像进行切换即可

> 当前支持的镜像全部在 [https://hub.docker.com/repository/docker/apache/doris/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fhub.docker.com%2Frepository%2Fdocker%2Ffreeoneplus%2Fapache-doris%2F) 可查看，会随着版本发布不断更新

**注意！一定需要将该文件保存至目标目录，命名为** **`docker-compose.yml`** **。**

### 3\. 启动测试

1.  切换路径到 `docker-compose.yml` 目录下

```
cd docker-compose.yml_path
```

1.  启动项目

```
docker-compose up
```

1.  输出尾部信息应为如下

```
Creating doris-fe ... done
Creating doris-register ... done
Creating doris-fe ... 
Creating doris-register ... 
Attaching to doris-be, doris-fe, doris-register
doris-register | Welcome to Apache-Doris Docker-Compose to quickly build test images!
doris-register | Start modifying the configuration files of FE and BE and run FE and BE!
doris-register | Start initializing Apache-Doris FE!
doris-register | Apache-Doris FE initialized successfully!
doris-register | Start initializing Apache-Doris BE!
doris-register | Apache-Doris BE initialized successfully!
doris-register | Get started with the Apache Doris registration steps!
doris-register | The initialization task of Apache-Doris has been completed, please start to experience it!
doris-register exited with code 0
```

1.  **注意！若没有报错，看到** **`doris-register exited with code 0`** **后请 关闭 当前窗口！重新打开 Shell 链接进行操作，一定不能 Ctrl + C 切断退出！**
2.  检查运行
3.  如果是虚拟机，需要关闭防火墙，宿主机能 Ping 通虚拟机，如果是云服务器，需要开放 8030,9030 端口

-   命令行检查
-   ```
    # FE 健康检查
    curl http://fe_host:fe_http_port/api/bootstrap
    > {"status":"OK","msg":"Success"}
    # BE 健康检查
    curl http://be_host:be_http_port/api/health
    > {"status": "OK","msg": "To Be Added"}
    ```
    
-   访问 FE-WEB-UI 界面
-   ```
    http://虚拟机IP或者云服务器公网IP:8030
    ```
    
-   登录账号为 `root` 或者 `admin`
-   密码为空

## 结语

使用 Docker 镜像及 Compose 工具，可以快速的部署一套测试或者学习使用的集群，但有特殊需求的情况，比如更简单的命令完成指定规模的 NFE MBE 环境搭建，甚至多 FE 高可用及 Observer 混部的各种情况，将在《Apache Doris Docker 高阶制作教程》里讲解，以及提供相应的社区生态工具。

如需继续学习，请继续以 Apache Doris 官⽹⽂档为主。

如有⽂档谬误和困难，可移步⾄ Apache Doris 官⽅社区微信群进⾏指正和提问。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9050c6ecf894cf2ae809c07e1c54f41~tplv-k3u1fbpfcp-watermark.image?)