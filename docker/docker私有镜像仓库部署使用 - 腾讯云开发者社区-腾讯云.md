`nexus` 不光可以做为私人的`maven`仓库，还可以作为`docker`的[镜像仓库](https://cloud.tencent.com/product/tcr?from=10680) 如何使用`nexus` 做`maven`仓库，可以参考： [部署maven私服](https://www.lzmvlog.top/archives/%E9%83%A8%E7%BD%B2maven%E7%A7%81%E6%9C%8D)

下面将介绍`nexus`作为`docker`镜像仓库的使用

查找镜像：

拉取镜像：

```
$ docker pull sonatype/nexus3
```

运行启动：

```
$ docker run -d -p 8081:8081 -p 8082:8082 --name nexus --restart=always --privileged=true -v /d/mongo/nexus-data:/nexus-data sonatype/nexus3
```

> 8081端口用于访问nexus 8082端口用于[docker](https://cloud.tencent.com/product/tke?from=10680)访问私有镜像厂库

登录： 默认账号：admin

密码存放在 /nexus-data/admin.password 文件中

```
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                                           NAMES
f97b03c02535   sonatype/nexus3   "sh -c ${SONATYPE_DI…"   6 minutes ago   Up 6 minutes   0.0.0.0:8081-8082->8081-8082/tcp, :::8081-8082->8081-8082/tcp   nexus

$ docker exec -it f97b03c02535 /bin/bash
bash-4.4$ cat /nexus-data/admin.password
```

> 登录成功后需要修改密码

-   docker-compose

```
version: '3.1'
services: 
    nexus:
        restart: always
        image: sonatype/nexus3
        container_name: nexus
        ports:
            - 8081:8081
            - 8082:8082
        valumes:
            - nexus-data:/nexus-data
```

-   `nexus`创建`docker`镜像仓库

|  |  |
| --- | --- |
| 
hosted



 | 

私有仓库（替代harbor）



 |
| 

proxy



 | 

访问不能直接到达的网络，如另一个私有仓库，或者国外的公共仓库



 |
| 

group



 | 

聚合类型的仓库。它可以将前面我们创建的3个仓库聚合成一个URL对外提供服务，可以屏蔽后端的差异性，实现类似透明代理的功能



 |

参考：[https://segmentfault.com/a/1190000015629878](https://segmentfault.com/a/1190000015629878)

> 以下为 hosted 类型私有仓库 操作

修改 `daemon.json`

```
{
  "registry-mirrors": [
    "https://hub.docker.com/"
  ],
  "insecure-registries": [
    "127.0.0.1:8082"
  ]
}
```

> 下面以 nginx 镜像为例

```
# 登录docker
$ docker login 127.0.0.1:8082

# 拉取镜像
$ docker pull nginx

# 修改标签
# 注意 标签名称 应该是 repository 的 hostip:port/name
$ docker tag nginx 127.0.0.1:8082/nginx

# 推送镜像
# 如果标签不对无法 push
$ docke push 127.0.0.1:8082/nginx

# 拉取镜像 （由于配置了仓库地址可以直接拉取）
$ docker pull 127.0.0.1:8082/nginx
```

喜欢编程的，请关注我的博客[https://www.lzmvlog.top/](https://www.lzmvlog.top/)

原创声明，本文系作者授权腾讯云开发者社区发表，未经许可，不得转载。

如有侵权，请联系 cloudcommunity@tencent.com 删除。