　　　　　　　　　　　　　　**Docker容器操作基础命令**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.从镜像启动一个容器**

**1>.前台启动容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      100                                                                                                 127.0.0.1:25                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
LISTEN     0      100                                                                                                       ::1:25                                                                                                                     :::*                  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run nginx
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run nginx　　　　　　　　　　　　　　#前台启动容器，当前终端会阻塞，需要另外启动一个终端查看，如下图所示。

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114140017751-447783467.png)**

**2>.后台启动容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
99b6a121adae        nginx               "nginx -g 'daemon of…"   3 minutes ago       Exited (0) 17 seconds ago                       zealous_khorana
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -d nginx
fb899a454b58342af808dd4b446e53696d5e1f69b7dcaf414f4467a041ac88a7
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
fb899a454b58        nginx               "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds                80/tcp              inspiring_morse
99b6a121adae        nginx               "nginx -g 'daemon of…"   4 minutes ago       Exited (0) 42 seconds ago                       zealous_khorana
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
fb899a454b58        nginx               "nginx -g 'daemon of…"   8 seconds ago       Up 7 seconds        80/tcp              inspiring_morse
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -d nginx　　　　　　　　　　　　#后台启动容器，并生成随机的容器ID和名称。

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114140354551-2067583044.png)**

**3>.创建容器时并进入容器（依赖于"-i"(保持一个标准输入)和"-t"(分配一个tty终端)两个参数）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -it nginx bash
root@b348fb4b1c0a:/# 
root@b348fb4b1c0a:/# cat /etc/issue
Debian GNU/Linux 10 \n \l

root@b348fb4b1c0a:/# 
root@b348fb4b1c0a:/# uname -a
Linux b348fb4b1c0a 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 GNU/Linux
root@b348fb4b1c0a:/# 
root@b348fb4b1c0a:/# exit 
exit
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# uname -a
Linux docker201.yinzhengjie.org.cn 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]#  
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -it nginx bash　　　　　　　　　　　　#会打开一个bash并直接进入到容器，并随机生成容器ID和名称。

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114130440776-658814033.png)**

**4>.自定义容器名称(一般在单机上会有点作用，在k8s集群中咱们很少去自定义容器名称哟~)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run --name mynginx -d nginx
a5e8c6f610f935fdbadf26d6278d477d71b83ca1890e78790299e5490ed6773a
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a5e8c6f610f9        nginx               "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        80/tcp              mynginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run --name mynginx -d nginx

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114140904278-743239571.png)

**5>.单次运行容器(容器退出后会自动删除,主要用于临时验证镜像内容的是否符合咱们的标准，如配置文件目录组织结构等信息)**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114141804892-8690743.png)

**6>.启动容器时指定DNS服务器地址及主机名等**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a5e8c6f610f9        nginx               "nginx -g 'daemon of…"   18 minutes ago      Up 18 minutes       80/tcp              mynginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -it --rm --name mynginx-test --hostname yinzhengjieDocker --dns 219.141.136.10 --dns 219.141.140.10 nginx bash
root@yinzhengjieDocker:/# 
root@yinzhengjieDocker:/# cat /etc/resolv.conf 
search yinzhengjie.org.cn
nameserver 219.141.136.10
nameserver 219.141.140.10
root@yinzhengjieDocker:/# 
root@yinzhengjieDocker:/# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -it --rm --name mynginx-test --hostname yinzhengjieDocker --dns 219.141.136.10 --dns 219.141.140.10 nginx bash

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114142941138-22717220.png)

**7>.创建容器时传递运行参数(容器需要有一个前台运行的进程才能保容器的运行，通过传递运行参数是一种方式，另外也可以在构建镜像的时候指定容器启动时在前台的运行命令)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# which tail
/usr/bin/tail
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -d --name myNginx nginx /usr/bin/tail -f "/etc/hosts"
fa701fc927e743ff44556fe4a12700fe180b399c2e6903d76a68a04613944c78
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
fa701fc927e7        nginx               "/usr/bin/tail -f /e…"   4 seconds ago       Up 3 seconds        80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -d --name myNginx nginx /usr/bin/tail -f "/etc/hosts"　　　　　　#创建容器时传递运行参数

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
fa701fc927e7        nginx               "/usr/bin/tail -f /e…"   37 seconds ago      Up 36 seconds       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container logs -f myNginx
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    fa701fc927e7
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container logs -f myNginx　　　　　　　　　　　　　　　　　　　　　　　　　　#实时查看容器的日志信息

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114172727364-37691671.png)

**二.查看容器**

**1>.查看正在运行的容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
d7b7e5fc7ca5        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:81->80/tcp   infallible_keller
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
d7b7e5fc7ca5        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:81->80/tcp   infallible_keller
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container ps

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114130828948-1454853461.png)**

**2>.查看所有容器(包括当前正在运行以及已经关闭的所有容器)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ps --help

Usage:    docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container ps --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  ps --help

Usage:    docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker ps --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls --help

Usage:    docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container ls --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  container list --help

Usage:    docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container list 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container list --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                NAMES
b348fb4b1c0a        nginx               "bash"                   7 minutes ago       Exited (0) 6 minutes ago                          hopeful_archimedes
5aa8329d672c        nginx               "bash"                   7 minutes ago       Exited (127) 7 minutes ago                        admiring_pare
d7b7e5fc7ca5        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:81->80/tcp   infallible_keller
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                NAMES
b348fb4b1c0a        nginx               "bash"                   7 minutes ago       Exited (0) 6 minutes ago                          hopeful_archimedes
5aa8329d672c        nginx               "bash"                   7 minutes ago       Exited (127) 7 minutes ago                        admiring_pare
d7b7e5fc7ca5        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:81->80/tcp   infallible_keller
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container ps -a

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114131026600-599054872.png)**

**三.删除运行中的容器**

**1>.删除单个容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container rm --help

Usage:    docker container rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove the volumes associated with the container
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container rm --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  rm --help

Usage:    docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove the volumes associated with the container
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker rm --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                NAMES
b348fb4b1c0a        nginx               "bash"                   9 minutes ago       Exited (0) 8 minutes ago                          hopeful_archimedes
5aa8329d672c        nginx               "bash"                   10 minutes ago      Exited (127) 9 minutes ago                        admiring_pare
d7b7e5fc7ca5        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:81->80/tcp   infallible_keller
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours                  0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container rm -f b348fb4b1c0a                        #基于容器的ID强制删除该容器
b348fb4b1c0a
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker rm -f 5aa8329d672c
5aa8329d672c
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container rm -f d7b7e5fc7ca5
d7b7e5fc7ca5
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
274a04fde642        nginx               "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp   zealous_tereshkova
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container rm -f b348fb4b1c0a 　　　　　　　　　　　　　　　　#基于容器的ID强制删除该容器

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114131408729-1889847036.png)**

**2>.删除多个容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                                            NAMES
ac3396d3ab7b        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:53->53/udp, 0.0.0.0:443->443/tcp, 0.0.0.0:8085->80/tcp   myNginx-test05
ebd2fdcc1327        nginx               "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        192.168.6.201:8082->80/tcp                                       myNginx-test04
725af16a867d        nginx               "nginx -g 'daemon of…"   3 minutes ago        Up 3 minutes        192.168.6.201:10001->80/tcp                                      myNginx-test03
c08f515bdba6        nginx               "nginx -g 'daemon of…"   5 minutes ago        Up 5 minutes        192.168.6.201:8081->80/tcp                                       myNginx-test02
c39343ae12c8        nginx               "nginx -g 'daemon of…"   6 minutes ago        Up 6 minutes        0.0.0.0:8080->80/tcp                                             myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container rm -f ac3396d3ab7b ebd2fdcc1327 725af16a867d c08f515bdba6 c39343ae12c8
ac3396d3ab7b
ebd2fdcc1327
725af16a867d
c08f515bdba6
c39343ae12c8
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container rm -f ac3396d3ab7b ebd2fdcc1327 725af16a867d c08f515bdba6 c39343ae12c8

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114144518253-254123922.png)

**3>.批量删除状态为已退出的容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                                                                          NAMES
864a9f0bb65a        nginx               "nginx -g 'daemon of…"   56 seconds ago      Exited (0) 55 seconds ago                                                                                  quirky_saha
38f133d58c66        nginx               "nginx -g 'daemon of…"   2 minutes ago       Exited (0) 2 minutes ago                                                                                   unruffled_allen
8c45174343fe        nginx               "nginx -g 'daemon of…"   2 minutes ago       Exited (0) 2 minutes ago                                                                                   gracious_montalcini
9bedca4e03be        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes                80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
d834dfa8cc11        nginx               "nginx -g 'daemon of…"   6 minutes ago       Up 6 minutes                80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
665b167a24d7        nginx               "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes                192.168.6.201:10002->80/tcp                                                    myNginx-test03
9060a616cfd9        nginx               "nginx -g 'daemon of…"   9 minutes ago       Up 9 minutes                192.168.6.201:8082->80/tcp                                                     myNginx-test02
33960733ae46        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes               0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -aq -f status=exited
864a9f0bb65a
38f133d58c66
8c45174343fe
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container rm -f `docker container ps -aq -f status=exited`
864a9f0bb65a
38f133d58c66
8c45174343fe
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                          NAMES
9bedca4e03be        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
d834dfa8cc11        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes        80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
665b167a24d7        nginx               "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes        192.168.6.201:10002->80/tcp                                                    myNginx-test03
9060a616cfd9        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes       192.168.6.201:8082->80/tcp                                                     myNginx-test02
33960733ae46        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes       0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container rm -f \`docker container ps -aq -f status=exited\`

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114150426359-311224647.png)

**4>.批量关闭正在运行的容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                          NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 3 minutes        80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes        192.168.6.201:10003->80/tcp                                                    myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes        192.168.6.201:8082->80/tcp                                                     myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes        0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a -q
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker stop $(docker container ps -a -q)
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   4 minutes ago       Exited (0) 4 seconds ago                       myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   4 minutes ago       Exited (0) 4 seconds ago                       myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   4 minutes ago       Exited (0) 4 seconds ago                       myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   5 minutes ago       Exited (0) 4 seconds ago                       myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   5 minutes ago       Exited (0) 4 seconds ago                       myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker stop $(docker container ps -a -q)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114151822553-227997711.png)**

**5>.**批量开启正在运行的容器****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   6 minutes ago       Exited (0) 2 minutes ago                       myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   6 minutes ago       Exited (0) 2 minutes ago                       myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   6 minutes ago       Exited (0) 2 minutes ago                       myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   7 minutes ago       Exited (0) 2 minutes ago                       myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   7 minutes ago       Exited (0) 2 minutes ago                       myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a -q -f status=exited
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker start $(docker container ps -a -q -f status=exited)
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                          NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   6 minutes ago       Up 3 seconds        80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   6 minutes ago       Up 3 seconds        80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 3 seconds        192.168.6.201:10004->80/tcp                                                    myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 3 seconds        192.168.6.201:8082->80/tcp                                                     myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 2 seconds        0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker start $(docker container ps -a -q -f status=exited)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114152036458-956557709.png)**

**6>.****批量强制关闭正在运行的容器******

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                          NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   9 minutes ago       Up 2 minutes        80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   9 minutes ago       Up 2 minutes        80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 2 minutes        192.168.6.201:10004->80/tcp                                                    myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 2 minutes        192.168.6.201:8082->80/tcp                                                     myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 2 minutes        0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a -q 
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker kill $(docker container ps -a -q)
8b429992e32f
68aa2fd7bef6
287ed27bcfd7
343aa3e9acf3
414b20722c93
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
8b429992e32f        nginx               "nginx -g 'daemon of…"   9 minutes ago       Exited (137) 3 seconds ago                       myNginx-test05
68aa2fd7bef6        nginx               "nginx -g 'daemon of…"   9 minutes ago       Exited (137) 3 seconds ago                       myNginx-test04
287ed27bcfd7        nginx               "nginx -g 'daemon of…"   10 minutes ago      Exited (137) 3 seconds ago                       myNginx-test03
343aa3e9acf3        nginx               "nginx -g 'daemon of…"   10 minutes ago      Exited (137) 3 seconds ago                       myNginx-test02
414b20722c93        nginx               "nginx -g 'daemon of…"   10 minutes ago      Exited (137) 3 seconds ago                       myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker kill $(docker container ps -a -q)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114152408812-243305308.png)

**7>.批量删除所有容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                                                                          NAMES
a1991e43206d        nginx               "nginx -g 'daemon of…"   3 seconds ago       Exited (0) 2 seconds ago                                                                                  happy_spence
9adf29a44026        nginx               "nginx -g 'daemon of…"   4 seconds ago       Exited (0) 3 seconds ago                                                                                  goofy_mendeleev
da17f02e11fb        nginx               "nginx -g 'daemon of…"   5 seconds ago       Exited (0) 5 seconds ago                                                                                  awesome_spence
6311f1a11a71        nginx               "nginx -g 'daemon of…"   7 seconds ago       Exited (0) 6 seconds ago                                                                                  tender_bassi
9bedca4e03be        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes               80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
d834dfa8cc11        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes              80/tcp, 192.168.6.201:8084->80/udp                                             myNginx-test04
665b167a24d7        nginx               "nginx -g 'daemon of…"   12 minutes ago      Up 12 minutes              192.168.6.201:10002->80/tcp                                                    myNginx-test03
9060a616cfd9        nginx               "nginx -g 'daemon of…"   13 minutes ago      Up 13 minutes              192.168.6.201:8082->80/tcp                                                     myNginx-test02
33960733ae46        nginx               "nginx -g 'daemon of…"   13 minutes ago      Up 13 minutes              0.0.0.0:8081->80/tcp                                                           myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ps -a -q
a1991e43206d
9adf29a44026
da17f02e11fb
6311f1a11a71
9bedca4e03be
d834dfa8cc11
665b167a24d7
9060a616cfd9
33960733ae46
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container rm -f $(docker container ps -a -q)
a1991e43206d
9adf29a44026
da17f02e11fb
6311f1a11a71
9bedca4e03be
d834dfa8cc11
665b167a24d7
9060a616cfd9
33960733ae46
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container rm -f $(docker container ps -a -q)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114150740834-333616491.png)

**四.容器的端口映射**

**1>.随机端口映射**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                     PORTS               NAMES
a177d1d09dad        nginx               "nginx -g 'daemon of…"   About a minute ago   Exited (0) 2 seconds ago                       blissful_chatterjee
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container rm -f a177d1d09dad
a177d1d09dad
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      100                                                                                                 127.0.0.1:25                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
LISTEN     0      100                                                                                                       ::1:25                                                                                                                     :::*                  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -P nginx                      #前台启动并随机映射本地端口到容器的80端口
172.17.0.1 - - [14/Jan/2020:13:32:14 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.29.0" "-"
192.168.0.1 - - [14/Jan/2020:13:36:25 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-"
2020/01/14 13:36:25 [error] 6#6: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "docker201.yinzhengjie.org.cn:10004", referrer: "http://docker201.yi
nzhengjie.org.cn:10004/"192.168.0.1 - - [14/Jan/2020:13:36:25 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://docker201.yinzhengjie.org.cn:10004/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-"
  
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -P nginx 　　　　　　　　　　　　　　#前台启动并随机映射本地端口到容器的80端口

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114134131593-1153707118.png)**

**2>.指定端口映射**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# hostname -i
192.168.6.201
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ss -ntlu
Netid  State      Recv-Q Send-Q                                                                                       Local Address:Port                                                                                                      Peer Address:Port              
tcp    LISTEN     0      128                                                                                                      *:22                                                                                                                   *:*                  
tcp    LISTEN     0      100                                                                                              127.0.0.1:25                                                                                                                   *:*                  
tcp    LISTEN     0      128                                                                                                     :::22                                                                                                                  :::*                  
tcp    LISTEN     0      100                                                                                                    ::1:25                                                                                                                  :::*                  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -p 10001:80 --name myNginx-test01 -it -d nginx                                               #将本地所有地址的10001端口映射到容器的80端口
2e51b900b725c2eddc90b47272f0eb875bfbd367e836c975868f767f70dcade0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -p 192.168.6.201:20002:80 --name myNginx-test02 -it -d nginx                                 #将本地192.168.6.201对应的IP地址的20002端口映射到容器的80端口
5025a1b50aee770c4cd0507d7d93925b0e6a5a56895438c7aee522a5d9d041b0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -p 192.168.6.201::80 --name myNginx-test03 -it -d nginx                                      #将本地192.168.6.201对应的IP地址的随机端口映射到容器的80端口
5f951096e1224514c5b9f813700013041b34ae949ff5a7f697cfd07103df1e19
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -p 192.168.6.201:40004:80/udp --name myNginx-test04 -it -d nginx                              #将本地192.168.6.201对应的IP地址的40004端口映射到容器的80UDP端口,如果不指定协议默认为UDP端口
5fc11671b64e941eb50b77d1b6917e36628b75789f8fd81115ff987de7334099
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -p 50001:80/udp -p 50002:443/tcp -p50003:53/udp --name myNginx-test05 -it -d nginx            #咱们还可以一次性映射多个端口/协议哟~
3df151d899dc264a4ab14de17b9d867334fc5ea3415445129e3a9c05c27b9e0b
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ss -ntlu
Netid  State      Recv-Q Send-Q                                                                                       Local Address:Port                                                                                                      Peer Address:Port              
udp    UNCONN     0      0                                                                                            192.168.6.201:40004                                                                                                                *:*                  
udp    UNCONN     0      0                                                                                                       :::50001                                                                                                               :::*                  
udp    UNCONN     0      0                                                                                                       :::50003                                                                                                               :::*                  
tcp    LISTEN     0      20480                                                                                        192.168.6.201:10006                                                                                                                *:*                  
tcp    LISTEN     0      128                                                                                                      *:22                                                                                                                   *:*                  
tcp    LISTEN     0      100                                                                                              127.0.0.1:25                                                                                                                   *:*                  
tcp    LISTEN     0      20480                                                                                        192.168.6.201:20002                                                                                                                *:*                  
tcp    LISTEN     0      20480                                                                                                   :::10001                                                                                                               :::*                  
tcp    LISTEN     0      20480                                                                                                   :::50002                                                                                                               :::*                  
tcp    LISTEN     0      128                                                                                                     :::22                                                                                                                  :::*                  
tcp    LISTEN     0      100                                                                                                    ::1:25                                                                                                                  :::*                  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker  container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                          NAMES
3df151d899dc        nginx               "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes        80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
5fc11671b64e        nginx               "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes        80/tcp, 192.168.6.201:40004->80/udp                                            myNginx-test04
5f951096e122        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        192.168.6.201:10006->80/tcp                                                    myNginx-test03
5025a1b50aee        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        192.168.6.201:20002->80/tcp                                                    myNginx-test02
2e51b900b725        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        0.0.0.0:10001->80/tcp                                                          myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114153609113-1287574009.png)

**3>.查看容器已经映射的端口**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                                                          NAMES
52e04f0425c5        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:50003->53/udp, 0.0.0.0:50001->80/udp, 0.0.0.0:50002->443/tcp   myNginx-test05
b4f9179d6229        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp, 192.168.6.201:40004->80/udp                                            myNginx-test04
b0b4bc54e0d7        nginx               "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        192.168.6.201:10005->80/tcp                                                    myNginx-test03
988b6c0f23dc        nginx               "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        192.168.6.201:20002->80/tcp                                                    myNginx-test02
c4a0488ca019        nginx               "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        0.0.0.0:10001->80/tcp                                                          myNginx-test01
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container port myNginx-test05
443/tcp -> 0.0.0.0:50002
53/udp -> 0.0.0.0:50003
80/udp -> 0.0.0.0:50001
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container port myNginx-test04
80/udp -> 192.168.6.201:40004
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container port myNginx-test03
80/tcp -> 192.168.6.201:10005
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container port myNginx-test02
80/tcp -> 192.168.6.201:20002
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container port myNginx-test01
80/tcp -> 0.0.0.0:10001
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container port myNginx-test05

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114153121725-1228234528.png)

**五.使用inspect命令获取容器的信息**

**1>.获取容器的详细信息**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              14 minutes ago      Up 14 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect  myNginx
[
    {
        "Id": "73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c",
        "Created": "2020-01-14T16:02:43.057451405Z",
        "Path": "bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 36347,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-01-14T16:02:43.272590229Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7",
        "ResolvConfPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/hostname",
        "HostsPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/hosts",
        "LogPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c-json.log",
        "Name": "/myNginx",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "shareable",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/asound",
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a-init/diff:/var/lib/docker/overlay2/df7fdc067554069fd6b56e73a90f5930e214e02d06b09f6476338606a8afc229/diff:/var/lib/docker/overlay2/97b082db97c394355eff2
e2d66872cd9f15d29a1453ba59675e822778ebe6beb/diff:/var/lib/docker/overlay2/ec680257b5ca772d2d7deacfd2b200359ea9bf4f02748ae3e143765475db8d7f/diff",                "MergedDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/merged",
                "UpperDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/diff",
                "WorkDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "73fe91c5c64d",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.17.7",
                "NJS_VERSION=0.3.7",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "bash"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "4d67f70a6813afad37ecf89da579c3c70b9f622b805392ae915bd0ab6c000645",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/4d67f70a6813",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "807d0935eb0323e5433a417290100b76629ab95242dbb5fc245c209892a45ba7",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "6b5b315db88a606c34602848b033adbd202fbd04e7de53c5daf9a06e982e1e4c",
                    "EndpointID": "807d0935eb0323e5433a417290100b76629ab95242dbb5fc245c209892a45ba7",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container inspect myNginx　　　　　　　　　　　　　　　　　　#查看容器名称为"myNginx"的容器详细信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              15 minutes ago      Up 15 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect  73fe91c5c64d
[
    {
        "Id": "73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c",
        "Created": "2020-01-14T16:02:43.057451405Z",
        "Path": "bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 36347,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-01-14T16:02:43.272590229Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7",
        "ResolvConfPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/hostname",
        "HostsPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/hosts",
        "LogPath": "/var/lib/docker/containers/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c/73fe91c5c64dc77d29ab091380c0c8cbbd99d4c4f408eae20ca8186476f8e89c-json.log",
        "Name": "/myNginx",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "shareable",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/asound",
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a-init/diff:/var/lib/docker/overlay2/df7fdc067554069fd6b56e73a90f5930e214e02d06b09f6476338606a8afc229/diff:/var/lib/docker/overlay2/97b082db97c394355eff2
e2d66872cd9f15d29a1453ba59675e822778ebe6beb/diff:/var/lib/docker/overlay2/ec680257b5ca772d2d7deacfd2b200359ea9bf4f02748ae3e143765475db8d7f/diff",                "MergedDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/merged",
                "UpperDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/diff",
                "WorkDir": "/var/lib/docker/overlay2/491db204831591ddd26d085f082c1c7e1018ec7ed3421366c0ad7151398b144a/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "73fe91c5c64d",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.17.7",
                "NJS_VERSION=0.3.7",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "bash"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "4d67f70a6813afad37ecf89da579c3c70b9f622b805392ae915bd0ab6c000645",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/4d67f70a6813",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "807d0935eb0323e5433a417290100b76629ab95242dbb5fc245c209892a45ba7",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "6b5b315db88a606c34602848b033adbd202fbd04e7de53c5daf9a06e982e1e4c",
                    "EndpointID": "807d0935eb0323e5433a417290100b76629ab95242dbb5fc245c209892a45ba7",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container inspect 73fe91c5c64d　　　　　　　　　　　　　　　#当然，也可以基于容器ID查看该容器的详细信息哟~

**2>.获取容器的IP地址**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              8 minutes ago       Up 8 minutes        80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect -f "{{.NetworkSettings.IPAddress}}" 73fe91c5c64d　　　　　　　　　　#可以指定容器的ID
172.17.0.2
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect -f "{{.NetworkSettings.IPAddress}}" myNginx　　　　　　　　　　　　　 #也可以指定容器的名称
172.17.0.2
[root@docker201.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.获取容器的PID信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              8 minutes ago       Up 8 minutes        80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect -f "{{.State.Pid}}" 73fe91c5c64d
36347
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect -f "{{.State.Pid}}" myNginx
36347
[root@docker201.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**六.进入到正在运行的容器**

**1>.使用attach命令连接正在运行的容器(生产环境中不推荐使用，因为使用exit退出后容器会跟着自动关闭)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c212441f89c4        nginx               "bash"              3 seconds ago       Up 2 seconds        80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container attach c212441f89c4
root@c212441f89c4:/# 
root@c212441f89c4:/# ls /
bin  boot  dev    etc  home  lib    lib64  media  mnt  opt    proc  root  run  sbin  srv  sys  tmp  usr  var
root@c212441f89c4:/# 
root@c212441f89c4:/# date
Tue Jan 14 15:53:02 UTC 2020
root@c212441f89c4:/# 
root@c212441f89c4:/# date +%Y-%m-%d-%H:%M:%S
2020-01-14-15:55:05
root@c212441f89c4:/# 
root@c212441f89c4:/# exit 
exit
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
c212441f89c4        nginx               "bash"              3 minutes ago       Exited (0) 3 seconds ago                       myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container attach c212441f89c4　　　　　　　　　　#指定需要连接容器的容器ID

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114160015314-1233758767.png)

**2>.使用exec命令(生产环境推荐使用这种方式)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              4 seconds ago       Up 4 seconds        80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container exec -it myNginx bash
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    73fe91c5c64d
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# exit 
exit
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              43 seconds ago      Up 42 seconds       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker container exec -it myNginx bash

  ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200117075835037-1463071699.png)

**3>.使用nsenter命令(生产环境也推荐使用这种方式，但需要单独安装nsenter命令)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# yum -y install util-linux                                            #安装nsenter命令
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
base                                                                                                                                                                                                                                                   | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                                                                       | 3.5 kB  00:00:00     
extras                                                                                                                                                                                                                                                 | 2.9 kB  00:00:00     
pouch-stable                                                                                                                                                                                                                                           | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                                                                | 2.9 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package util-linux.x86_64 0:2.23.2-59.el7 will be updated
---> Package util-linux.x86_64 0:2.23.2-61.el7_7.1 will be an update
--> Processing Dependency: libuuid = 2.23.2-61.el7_7.1 for package: util-linux-2.23.2-61.el7_7.1.x86_64
--> Processing Dependency: libsmartcols = 2.23.2-61.el7_7.1 for package: util-linux-2.23.2-61.el7_7.1.x86_64
--> Processing Dependency: libmount = 2.23.2-61.el7_7.1 for package: util-linux-2.23.2-61.el7_7.1.x86_64
--> Processing Dependency: libblkid = 2.23.2-61.el7_7.1 for package: util-linux-2.23.2-61.el7_7.1.x86_64
--> Running transaction check
---> Package libblkid.x86_64 0:2.23.2-59.el7 will be updated
---> Package libblkid.x86_64 0:2.23.2-61.el7_7.1 will be an update
---> Package libmount.x86_64 0:2.23.2-59.el7 will be updated
---> Package libmount.x86_64 0:2.23.2-61.el7_7.1 will be an update
---> Package libsmartcols.x86_64 0:2.23.2-59.el7 will be updated
---> Package libsmartcols.x86_64 0:2.23.2-61.el7_7.1 will be an update
---> Package libuuid.x86_64 0:2.23.2-59.el7 will be updated
---> Package libuuid.x86_64 0:2.23.2-61.el7_7.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================================================
 Package                                                            Arch                                                         Version                                                                  Repository                                                     Size
==============================================================================================================================================================================================================================================================================
Updating:
 util-linux                                                         x86_64                                                       2.23.2-61.el7_7.1                                                        updates                                                       2.0 M
Updating for dependencies:
 libblkid                                                           x86_64                                                       2.23.2-61.el7_7.1                                                        updates                                                       181 k
 libmount                                                           x86_64                                                       2.23.2-61.el7_7.1                                                        updates                                                       183 k
 libsmartcols                                                       x86_64                                                       2.23.2-61.el7_7.1                                                        updates                                                       141 k
 libuuid                                                            x86_64                                                       2.23.2-61.el7_7.1                                                        updates                                                        83 k

Transaction Summary
==============================================================================================================================================================================================================================================================================
Upgrade  1 Package (+4 Dependent packages)

Total download size: 2.6 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/5): libblkid-2.23.2-61.el7_7.1.x86_64.rpm                                                                                                                                                                                                           | 181 kB  00:00:00     
(2/5): libmount-2.23.2-61.el7_7.1.x86_64.rpm                                                                                                                                                                                                           | 183 kB  00:00:00     
(3/5): libuuid-2.23.2-61.el7_7.1.x86_64.rpm                                                                                                                                                                                                            |  83 kB  00:00:00     
(4/5): util-linux-2.23.2-61.el7_7.1.x86_64.rpm                                                                                                                                                                                                         | 2.0 MB  00:00:00     
(5/5): libsmartcols-2.23.2-61.el7_7.1.x86_64.rpm                                                                                                                                                                                                       | 141 kB  00:00:02     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                         899 kB/s | 2.6 MB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : libuuid-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                          1/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Updating   : libblkid-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                         2/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Updating   : libmount-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                         3/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Updating   : libsmartcols-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                     4/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Updating   : util-linux-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                       5/10 
  Cleanup    : util-linux-2.23.2-59.el7.x86_64                                                                                                                                                                                                                           6/10 
  Cleanup    : libmount-2.23.2-59.el7.x86_64                                                                                                                                                                                                                             7/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Cleanup    : libblkid-2.23.2-59.el7.x86_64                                                                                                                                                                                                                             8/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Cleanup    : libuuid-2.23.2-59.el7.x86_64                                                                                                                                                                                                                              9/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Cleanup    : libsmartcols-2.23.2-59.el7.x86_64                                                                                                                                                                                                                        10/10 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Verifying  : libblkid-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                         1/10 
  Verifying  : util-linux-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                       2/10 
  Verifying  : libsmartcols-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                     3/10 
  Verifying  : libmount-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                         4/10 
  Verifying  : libuuid-2.23.2-61.el7_7.1.x86_64                                                                                                                                                                                                                          5/10 
  Verifying  : libuuid-2.23.2-59.el7.x86_64                                                                                                                                                                                                                              6/10 
  Verifying  : libblkid-2.23.2-59.el7.x86_64                                                                                                                                                                                                                             7/10 
  Verifying  : libmount-2.23.2-59.el7.x86_64                                                                                                                                                                                                                             8/10 
  Verifying  : util-linux-2.23.2-59.el7.x86_64                                                                                                                                                                                                                           9/10 
  Verifying  : libsmartcols-2.23.2-59.el7.x86_64                                                                                                                                                                                                                        10/10 

Updated:
  util-linux.x86_64 0:2.23.2-61.el7_7.1                                                                                                                                                                                                                                       

Dependency Updated:
  libblkid.x86_64 0:2.23.2-61.el7_7.1                               libmount.x86_64 0:2.23.2-61.el7_7.1                               libsmartcols.x86_64 0:2.23.2-61.el7_7.1                               libuuid.x86_64 0:2.23.2-61.el7_7.1                              

Complete!
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# yum -y install util-linux 　　　　　　　　　　　　　　　　#安装nsenter命令

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              20 minutes ago      Up 20 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container inspect -f "{{.State.Pid}}" myNginx                        #获取PID
36347
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# nsenter -t 36347 -m -u -i -n -p                                            #通过容器的PID连接运行中的容器
mesg: ttyname failed: No such device
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# cat /etc/issue
Debian GNU/Linux 10 \n \l

root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# cat /etc/resolv.conf 
# Generated by NetworkManager
search yinzhengjie.org.cn
nameserver 192.168.7.254
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# exit 
logout
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              22 minutes ago      Up 22 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# nsenter -t 36347 -m -u -i -n -p 　　　　　　　　　　　　#通过容器的PID连接运行中的容器

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114162831149-168282213.png)

**4>.脚本方式连接容器**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# cat docker_connect.sh 
#!/bin/bash
#
#********************************************************************
#Author:             yinzhengjie
#QQ:                   1053419035
#Date:                 2019-11-25
#FileName：           docker_connect.sh 
#URL:                 http://www.cnblogs.com/yinzhengjie
#Description：          The test script
#Copyright notice:         original works, no reprint! Otherwise, legal liability will be investigated.
#********************************************************************

function docker_connect(){
    CONTAINER_NAME=$1
    CONTAINER_PID=$(docker container inspect -f "{{.State.Pid}}" ${CONTAINER_NAME})
    nsenter -t ${CONTAINER_PID} -m -u -i -n -p
}

docker_connect $1
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# cat docker\_connect.sh

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# ll docker_connect.sh 
-rw-r--r--. 1 root root 683 Jan 15 00:38 docker_connect.sh
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# chmod +x docker_connect.sh 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll docker_connect.sh 
-rwxr-xr-x. 1 root root 683 Jan 15 00:38 docker_connect.sh
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# chmod +x docker\_connect.sh

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              36 minutes ago      Up 36 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll
total 4
-rwxr-xr-x. 1 root root 683 Jan 15 00:38 docker_connect.sh
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ./docker_connect.sh myNginx
mesg: ttyname failed: No such device
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# cat /etc/issue
Debian GNU/Linux 10 \n \l

root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# date +%F
2020-01-14
root@73fe91c5c64d:/# 
root@73fe91c5c64d:/# exit 
logout
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
73fe91c5c64d        nginx               "bash"              37 minutes ago      Up 37 minutes       80/tcp              myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# ./docker\_connect.sh myNginx

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114164144929-293986317.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186