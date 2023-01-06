　　　　　　　　　　　　**Docker自建仓库之Docker Registry部署实战**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

　　**本篇博客将介绍通过官方提供的docker registry 镜像来简单搭建一套本地私有仓库环境,生产环境中很少有人使用docker registry,因为它没有管理界面，这一点对于运维人员并不友好,而对于开发人员其实有没有管理界面都无所谓。**

**一.Docker Registry概述**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Docker Registry作为Docker的核心组件之一负责镜像内容的存储和分发，客户端的docker pull以及push命令都将直接与registry进行交互。　　最初版本的registry由python实现，由于设计初期在安全性，性能以及API的设计上有着诸多的缺陷,该版本在0.9之后停止了开发,由新的项目distribution(新docker register被称为Distribution)来重新设计并开发下一代registry,新的项目由Golang开发。　　所有的API,底层存储方式,系统架构都进行了全方面的重新设计以解决上一代registry中存在的问题日，2016年4月份registry 2.0正式发布，docker 1.6版本开始支持registry 2.0,而八月份随着docker 1.8发布,docker hub正式启用2.1版本registry全面替代之前版本registry，新版registry对镜像存储格式进行了重新设计并和旧版本不兼容,docker 1.5和之前的版本无法读取2.0的镜像。　　另外,Registry 2.4版本之后支持了回收站机制，也就是可以删除镜像了，在2.4版本之前是无法支持删除镜像的，所以如果你要使用最好是大于Registry 2.4版本的哟~
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.搭建单机仓库**

**1>.下载Docker Registry镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image pull registry
Using default tag: latest
latest: Pulling from library/registry
486039affc0a: Pull complete 
ba51a3b098e6: Pull complete 
8bb4c43d6c8e: Pull complete 
6f5f453e5f2d: Pull complete 
42bc10b72f42: Pull complete 
Digest: sha256:7d081088e4bfd632a88e3f3bcd9e007ef44a796fddfe3261407a3f9f04abe1e7
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127100639935-689914105.png)

**2>.创建授权使用目录**

```
[root@docker101.yinzhengjie.org.cn ~]# mkdir -pv /yinzhengjie/data/docker/auth
mkdir: created directory ‘/yinzhengjie/data’
mkdir: created directory ‘/yinzhengjie/data/docker’
mkdir: created directory ‘/yinzhengjie/data/docker/auth’
[root@docker101.yinzhengjie.org.cn ~]# 
```

**3>.创建创建用户名和密码**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# cd /yinzhengjie/data/docker/
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# ll
total 0
drwxr-xr-x 2 root root 6 Jan 27 18:21 auth
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# docker run --entrypoint htpasswd registry -Bbn jason 2020 > auth/htpasswd
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# ll auth/
total 4
-rw-r--r-- 1 root root 68 Jan 27 18:21 htpasswd
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# cat auth/htpasswd 
jason:$2y$05$Gzol9U5vYUMe2kEaUEj03OA2bAKnhK3CnZJFOzv2ljAqrawW/db4e

[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/data/docker]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127102315637-787036278.png)

**4>.启动docker registry**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -d -p 6000:5000 --restart=always --name myRegistry01 -v /yinzhengjie/data/docker/auth/:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
b06b6468313a577d5b33f92e70f7e5843b0a5cdd1d0793eaa5bf96be9ffdf14d
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::6000                                                                                                                   :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
b06b6468313a        registry            "/entrypoint.sh /etc…"   8 seconds ago       Up 7 seconds        0.0.0.0:6000->5000/tcp   myRegistry01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it myRegistry01 sh
/ # 
/ # netstat -untalp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 :::5000                 :::*                    LISTEN      1/registry
/ # 
/ # exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127105242024-53770782.png)

**5>.验证端口和容器**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127105439816-2146594447.png)

**6>.**测试登录仓库****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# vim /etc/docker/daemon.json 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "insecure-registries":["docker101.yinzhengjie.org.cn:6000"]
}
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# systemctl restart docker
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker login docker101.yinzhengjie.org.cn:6000
Username: jason
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127110349440-2107220689.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 27
 Server Version: 19.03.5
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-957.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.84GiB
 Name: docker101.yinzhengjie.org.cn
 ID: ZPMZ:2YLN:PQIW:2CN4:GYX6:LAV5:4WMX:U2PH:GIDV:R363:TQI3:QP2O
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Username: yinzhengjie2019
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  docker101.yinzhengjie.org.cn:6000
  127.0.0.0/8
 Registry Mirrors:
  https://tuv7rqqq.mirror.aliyuncs.com/
 Live Restore Enabled: false

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker info

**7>.使用yum方式安装Docker Registry服务**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie/p/11706627.html
```

**三.验证Docker Registry**

**1>.在"docker101.yinzhengjie.org.cn"登陆后上传镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                   IMAGE ID            CREATED             SIZE
centos-haproxy                                            v1.8.20               1858fe05d96f        3 days ago          606MB
registry                                                  latest                708bc6af7e5e        3 days ago          25.8MB
tomcat-app01                                              v0.1                  bf45c22f2d5b        4 days ago          983MB
tomcat-base                                               8.5.50                9ff79f369094        5 days ago          968MB
jdk-base                                                  1.8.0_231             0f63a97ddc85        5 days ago          953MB
centos-base                                               7.6.1810              b4931fd9ace2        5 days ago          551MB
centos                                                    centos7.6.1810        f1cb7c7d58b7        10 months ago       202MB
yinzhengjie2019/centos                                    v0.1_centos7.6.1810   f1cb7c7d58b7        10 months ago       202MB
registry.cn-beijing.aliyuncs.com/yinzhengjie2020/centos   v0.1_centos7.6.1810   f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image tag centos-base:7.6.1810 docker101.yinzhengjie.org.cn:6000/jason/centos-base:v7.6.1810
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                   IMAGE ID            CREATED             SIZE
centos-haproxy                                            v1.8.20               1858fe05d96f        3 days ago          606MB
registry                                                  latest                708bc6af7e5e        3 days ago          25.8MB
tomcat-app01                                              v0.1                  bf45c22f2d5b        4 days ago          983MB
tomcat-base                                               8.5.50                9ff79f369094        5 days ago          968MB
jdk-base                                                  1.8.0_231             0f63a97ddc85        5 days ago          953MB
centos-base                                               7.6.1810              b4931fd9ace2        5 days ago          551MB
docker101.yinzhengjie.org.cn:6000/jason/centos-base       v7.6.1810             b4931fd9ace2        5 days ago          551MB
centos                                                    centos7.6.1810        f1cb7c7d58b7        10 months ago       202MB
yinzhengjie2019/centos                                    v0.1_centos7.6.1810   f1cb7c7d58b7        10 months ago       202MB
registry.cn-beijing.aliyuncs.com/yinzhengjie2020/centos   v0.1_centos7.6.1810   f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image push docker101.yinzhengjie.org.cn:6000/jason/centos-base:v7.6.1810 
The push refers to repository [docker101.yinzhengjie.org.cn:6000/jason/centos-base]
0f448859d86e: Pushed 
89169d87dbe2: Pushed 
v7.6.1810: digest: sha256:62c5a70f2846bd7f8ecd65785e379d0e00acf33ae899f0ec96754a3731b2d425 size: 742
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127111040355-1308853364.png)

**2>.在"docker102.yinzhengjie.org.cn"登陆后下载镜像**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127111534542-555766817.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186