![](https://ask.qcloudimg.com/http-save/yehe-7754373/uzp967q07b.jpeg?imageView2/2/w/1620);

> 当我们执行docker pull xxx的时候，docker默认是从registry.docker.com这个地址上去查找我们所需要的镜像文件，然后执行下载操作。这类的[镜像仓库](https://cloud.tencent.com/product/tcr?from=10680)就是docker默认的公共仓库，所有人都可以直接查看或下载、使用，但是呢，基于网络原因，下载速度有限制比较慢。因此，我们在公司内部内网环境中使用dokcer，一般不会将镜像文件上传到公网公共库中。但内部共享使用就是个问题，所以，私有仓库就由此产生了。

## **什么是私有仓库？**

私有仓库，就是本地（内网环境）组建的一个与公网公共库功能相似的镜像仓库。组建好之后，我们就可以将打包好的镜像提交到私有仓库中，这样内网其它用户也可以使用这个镜像文件。 本文使用官方提供的registry镜像来组建企业内网的私有镜像仓库

## **环境介绍**

两台安装好docker环境的主机

-   服务端：192.168.3.82 私有仓库[服务器](https://cloud.tencent.com/product/cvm?from=10680)在，运行registry容器
-   客户端：192.168.3.83 测试客户端，用于上传、下载镜像文件

## **安装布署过程**

##### **下载官方registry镜像文件**

```
[root@master ~]# docker pull registry
Using default tag: latest
Trying to pull repository docker.io/library/registry ... 
latest: Pulling from docker.io/library/registry
81033e7c1d6a: Pull complete 
b235084c2315: Pull complete 
c692f3a6894b: Pull complete 
ba2177f3a70e: Pull complete 
a8d793620947: Pull complete 
Digest: sha256:672d519d7fd7bbc7a448d17956ebeefe225d5eb27509d8dc5ce67ecb4a0bce54
Status: Downloaded newer image for docker.io/registry:latest
[root@master ~]# docker images |grep registry
docker.io/registry   latest  d1fd7d86a825   5 months ago  33.3 MB
```

##### **运行registry容器**

```
[root@master ~]# mkdir /docker/registry -p
[root@master ~]# docker run -itd -v /docker/registry/:/docker/registry -p 5000:5000 --restart=always --name registry registry:latest
26d0b91a267f684f9da68f01d869b31dbc037ee6e7bf255d8fb435a22b857a0e
[root@master ~]# docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED        STATUS        PORTS                    NAMES
26d0b91a267f   registry:latest  "/entrypoint.sh /e..."   4 seconds ago  Up 3 seconds  0.0.0.0:5000->5000/tcp   registry
参数说明
1）-itd：在容器中打开一个伪终端进行交互操作，并在后台运行；
2）-v：把宿主机的/docker/registry目录绑定到容器/docker/registry目录(这个目录是registry容器中存放镜像文件的目录)，来实现数据的持久化；
3）-p：映射端口；访问宿主机的5000端口就访问到registry容器的服务了；
4）--restart=always：这是重启的策略，假如这个容器异常退出会自动重启容器；
5）--name registry：创建容器命名为registry，可自定义任何名称；
6）registry:latest：这个是刚才pull下来的镜像；
```

##### **查看远程仓库镜像文件**

```
[root@master ~]# curl http://localhost:5000/v2/_catalog
{"repositories":[]}
同样也可以使用浏览器访问http://server-ip:5000/v2/_catalog, 结果相同，都是空的没有任何文件。
```

## **客户端操作**

##### **修改下载的镜像源**

```
[root@slave1 ~]# vim /etc/docker/daemon.json
{
"registry-mirrors":["https://registry.docker-cn.com"]
}
[root@slave1 ~]# systemctl restart docker
```

##### **下载测试镜像**

```
[root@slave1 ~]# docker pull nginx
Using default tag: latest
Trying to pull repository docker.io/library/nginx ... 
latest: Pulling from docker.io/library/nginx
683abbb4ea60: Pull complete 
6ff57cbc007a: Pull complete 
162f7aebbf40: Pull complete 
Digest: sha256:636dd2749d9a363e5b57557672a9ebc7c6d041c88d9aef184308d7434296feea
Status: Downloaded newer image for docker.io/nginx:latest
```

##### **给镜像打TAG**

```
[root@slave1 ~]# docker tag nginx:latest 192.168.3.82:5000/nginx:v1
[root@slave1 ~]# docker images
REPOSITORY                TAG       IMAGE ID        CREATED       SIZE
192.168.3.82:5000/nginx   v1        649dcb69b782    8 hours ago   109 MB
docker.io/nginx           latest    649dcb69b782    8 hours ago   109 MB
```

##### **上传镜像**

```
[root@slave1 ~]# docker push 192.168.3.82:5000/nginx:v1
The push refers to a repository [192.168.3.82:5000/nginx]
Get https://192.168.3.82:5000/v1/_ping: http: server gave HTTP response to HTTPS client
#注意这里出现报错提示，从提示信息可以看出需要使用https的方式才能上传，解决方案如下：
[root@slave1 ~]# vim /etc/docker/daemon.json
{
"registry-mirrors":["https://registry.docker-cn.com"],
 "insecure-registries":["192.168.3.82:5000"]
}
#添加私有镜像服务器的地址，注意书写格式为json，有严格的书写要求，需要重启docker服务生效配置
[root@slave1 ~]# systemctl restart docker
[root@slave1 ~]# docker push 192.168.3.82:5000/nginx:v1
The push refers to a repository [192.168.3.82:5000/nginx]
6ee5b085558c: Pushed 
78f25536dafc: Pushed 
9c46f426bcb7: Pushed 
v1: digest: sha256:edad5e71815c79108ddbd1d42123ee13ba2d8050ad27cfa72c531986d03ee4e7 size: 948
```

##### **重新查看镜像仓库**

```
[root@master ~]# curl http://localhost:5000/v2/_catalog
{"repositories":["nginx"]}
[root@master ~]# curl http://localhost:5000/v2/nginx/tags/list
{"name":"nginx","tags":["v1"]}
#查看有哪些版本
```

##### **测试下载**

```
#首先删除客户端主机之前从公共库下载下来的镜像文件
[root@slave1 ~]# docker images
REPOSITORY                TAG      IMAGE ID        CREATED        SIZE
192.168.3.82:5000/nginx   v1       649dcb69b782    10 hours ago   109 MB
docker.io/nginx           latest   649dcb69b782    10 hours ago   109 MB
[root@slave1 ~]# docker image rmi -f 649dcb69b782
Untagged: 192.168.3.82:5000/nginx:v1
Untagged: 192.168.3.82:5000/nginx@sha256:edad5e71815c79108ddbd1d42123ee13ba2d8050ad27cfa72c531986d03ee4e7
Untagged: docker.io/nginx:latest
Untagged: docker.io/nginx@sha256:636dd2749d9a363e5b57557672a9ebc7c6d041c88d9aef184308d7434296feea
Deleted: sha256:649dcb69b782d4e281c92ed2918a21fa63322a6605017e295ea75907c84f4d1e
Deleted: sha256:bf7cb208a5a1da265666ad5ab3cf10f0bec1f4bcb0ba8d957e2e485e3ac2b463
Deleted: sha256:55d02c20aa07136ab07ab47f4b20b97be7a0f34e01a88b3e046a728863b5621c
Deleted: sha256:9c46f426bcb704beffafc951290ee7fe05efddbc7406500e7d0a3785538b8735
[root@slave1 ~]# docker images
REPOSITORY       TAG             IMAGE ID        CREATED         SIZE
#此时客户端所有的镜像文件全部删除
[root@slave1 ~]# docker pull 192.168.3.82:5000/nginx:v1
Trying to pull repository 192.168.3.82:5000/nginx ... 
v1: Pulling from 192.168.3.82:5000/nginx
683abbb4ea60: Pull complete 
6ff57cbc007a: Pull complete 
162f7aebbf40: Pull complete 
Digest: sha256:edad5e71815c79108ddbd1d42123ee13ba2d8050ad27cfa72c531986d03ee4e7
Status: Downloaded newer image for 192.168.3.82:5000/nginx:v1
[root@slave1 ~]# docker images
REPOSITORY                TAG     IMAGE ID       CREATED         SIZE
192.168.3.82:5000/nginx   v1      649dcb69b782   11 hours ago    109 MB
#可以看出，客户端已正常从远端服务器拉取到所需要的镜像文件，其它内网服务器也可以正常共享这台镜像服务器上的镜像文件，不用去公网拉取。
```

注：之前分享的几篇资源文章，由于一些原因（链接失效或版权原因）很多资源被取消了，故民工哥重新整理了一份（**包括JAVA开发、大数据、Hadoop、zookeeper、移动端开发、系统运维、**[**数据库**](https://cloud.tencent.com/solution/database?from=10680)**、Python、微服务、架构师、Docker容器、K8S**等众多视频资源），需要的小伙伴直接访问下面的链接：

链接: https://pan.baidu.com/s/1VdIPrFcWLNq-8htjniB1Tw 提取码: w4ry

或者**公众号后台回复：“干货”** 即可获得全部高达800GB的视频资源。

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_53f281f2ae80)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。