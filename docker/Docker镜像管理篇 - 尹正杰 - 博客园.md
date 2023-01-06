　　　　　　　　　　　　　　**Docker镜像管理篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Docker镜像概述**

**1>.什么是docker镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Docker镜像含有启动容器所需要的文件系统及所需要的内容，因此镜像主要用于创建并启动docker容器。
　　Docker镜像含里面是一层层文件系统，叫做联合文件系统(Union FS),联合文件系统开源将几层目录挂载到一起，形成一个虚拟文件系统，虚拟文件系统的目录结构就像普通的Linux系统的目录结构一样，docker通过这些文件再加上宿主机的内核提供了一个Linux的虚拟环境，每一层文件系统我们叫做layer。

　　联合文件系统(Union FS)可以对每一层(layer)文件系统设置三种权限，即只读(readonly)，读写(readwrite)和写出(whiteout-able)。严格来说,docker镜像中每一层文件系统都是只读的或者说是禁止直接修改,因为大家都知道容器是基于镜像启动的,如果A,B用户都基于同一个镜像启动的容器进行开发，如果镜像内容不一致会导致两个人拿到的初始代码不一致！

　　构建镜像的时候，从一个最基础的操作系统开始,每个构建的操作都相当于做一层的修改，增加了一层文件系统，一层层网上叠加，上层的修改会覆盖底层该位置的可见性。这很容易理解，就像上层把底层遮住了一样，当使用镜像的时候，我们只会看到一个完全的整体，不知道里面有几层文件系统也不需要知道里面有几层，如下图所示。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114072955346-62588242.png)

**2>.容器，镜像和父镜像关系**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114073308880-2061516407.png)**

**3>.docker镜像(image)和Linux文件系统对比**

```
　　一个典型的Linux文件系统由bootfs(全称为"boot file system")和rootfs("root file system")两部分组成。bootfs主要包含bootloader和kernel，bootloader主要用于引导加载kernel，当kernel被加载到内存中后bootfs会被卸载(umount)掉。rootfs包含就是典型Linux系统中的"/bin","/dev","/etc","/home","/proc"等标准目录和文件。

　　docker image中最基础的两层结构，不同的Linux发行版(如Ubuntu和CentOS)在rootfs这一层会有所区别,但是对于docker镜像通常都比较小,官方提供的CentOS基础镜像在220MB左右,一些其它版本镜像甚至不到10MB，docker镜像直接调用宿主机的内核，镜像中只提供rootfs,也就是只需要包括最基本的命令,工具和程序库就可以了，比如alpine镜像仅有5.59M左右。

　　如下图所示,两个不同的docker image在同一个宿主机内核上实现不同的rootfs。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114075111130-358367670.png)

**二.Docker镜像的管理命令**

```
　　docker命令是常使用的docker客户端命令,其后面可以加不同的参数以实现响应的功能，常用的镜像管理相关命令如下。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image --help

Usage:    docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker image --help

**1>.搜索镜像**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker search --help

Usage:    docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker search --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker search alpine
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
alpine                                 A minimal Docker image based on Alpine Linux…   5995                [OK]                
mhart/alpine-node                      Minimal Node.js built on Alpine Linux           452                                     
anapsix/alpine-java                    Oracle Java 8 (and 7) with GLIBC 2.28 over A…   439                                     [OK]
frolvlad/alpine-glibc                  Alpine Docker image with glibc (~12MB)          228                                     [OK]
gliderlabs/alpine                      Image based on Alpine Linux will help you wi…   181                                     
alpine/git                             A  simple git container running in alpine li…   112                                     [OK]
mvertes/alpine-mongo                   light MongoDB container                         108                                     [OK]
yobasystems/alpine-mariadb             MariaDB running on Alpine Linux [docker] [am…   57                                      [OK]
kiasaki/alpine-postgres                PostgreSQL docker image based on Alpine Linux   45                                      [OK]
alpine/socat                           Run socat command in alpine container           44                                      [OK]
davidcaste/alpine-tomcat               Apache Tomcat 7/8 using Oracle Java 7/8 with…   40                                      [OK]
zzrot/alpine-caddy                     Caddy Server Docker Container running on Alp…   35                                      [OK]
easypi/alpine-arm                      AlpineLinux for RaspberryPi                     32                                      
jfloff/alpine-python                   A small, more complete, Python Docker image …   32                                      [OK]
byrnedo/alpine-curl                    Alpine linux with curl installed and set as …   28                                      [OK]
hermsi/alpine-sshd                     Dockerize your OpenSSH-server with rsync and…   26                                      [OK]
etopian/alpine-php-wordpress           Alpine WordPress Nginx PHP-FPM WP-CLI           23                                      [OK]
hermsi/alpine-fpm-php                  Dockerize your FPM PHP 7.4 upon a lightweigh…   22                                      [OK]
zenika/alpine-chrome                   Chrome running in headless mode in a tiny Al…   16                                      [OK]
bashell/alpine-bash                    Alpine Linux with /bin/bash as a default she…   14                                      [OK]
davidcaste/alpine-java-unlimited-jce   Oracle Java 8 (and 7) with GLIBC 2.21 over A…   13                                      [OK]
spotify/alpine                         Alpine image with `bash` and `curl`.            9                                       [OK]
tenstartups/alpine                     Alpine linux base docker image with useful p…   9                                       [OK]
roribio16/alpine-sqs                   Dockerized ElasticMQ server + web UI over Al…   7                                       [OK]
cfmanteiga/alpine-bash-curl-jq         Docker Alpine image with Bash, curl and jq p…   5                                       [OK]
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker search alpine

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114084510095-849386052.png)

**2>.下载镜像**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker pull --help

Usage:    docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker pull --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
e6b0cf9c0882: Pull complete 
Digest: sha256:2171658620155679240babee0a7714f6509fae66898db422ad803b951257db78
Status: Downloaded newer image for alpine:latest
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker pull alpine

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker run -it --rm alpine                #如果想要单次运行容器，当容器结束时该容器的数据默认也会一并删除掉。
/ # 
/ # cat /etc/issue 
Welcome to Alpine Linux 3.11
Kernel \r on an \m (\l)

/ # 
/ # apk add vim                    #安装vim工具
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
(1/6) Installing xxd (8.2.0-r0)
(2/6) Installing lua5.3-libs (5.3.5-r2)
(3/6) Installing ncurses-terminfo-base (6.1_p20191130-r0)
(4/6) Installing ncurses-terminfo (6.1_p20191130-r0)
(5/6) Installing ncurses-libs (6.1_p20191130-r0)
(6/6) Installing vim (8.2.0-r0)
Executing busybox-1.31.1-r8.trigger
OK: 42 MiB in 20 packages
/ # 
/ # apk add nginx               #安装nginx工具
(1/3) Installing pcre (8.43-r0)
(2/3) Installing nginx (1.16.1-r4)
Executing nginx-1.16.1-r4.pre-install
(3/3) Installing nginx-vim (1.16.1-r4)
Executing busybox-1.31.1-r8.trigger
OK: 43 MiB in 23 packages
/ # 
/ # netstat -natlp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 172.17.0.4:43840        151.101.228.249:80      TIME_WAIT   -
/ # 
/ # mkdir /run/nginx
/ # 
/ # nginx                         #启动nginx
/ # 
/ # netstat -natlp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      22/nginx: master pr
tcp        0      0 172.17.0.4:43840        151.101.228.249:80      TIME_WAIT   -
tcp        0      0 :::80                   :::*                    LISTEN      22/nginx: master pr
/ # 
/ # exit
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker run -it --rm alpine #如果想要单次运行容器，当容器结束时该容器的数据默认也会一并删除掉。

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114085426106-1666452970.png)

**3>.查看本地镜像**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image --help

Usage:    docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker image ls

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images --help

Usage:    docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker images

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114091716151-2137076885.png)

**4>.查看镜像的历史**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker history --help

Usage:    docker history [OPTIONS] IMAGE

Show the history of an image

Options:
      --format string   Pretty-print images using a Go template
  -H, --human           Print sizes and dates in human readable format (default true)
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker history --help

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114082610143-1477837917.png)

**5>.镜像导出**(镜像的导出使用命令临时导出可以，在生产环境不推荐使用，生产环境应该用私有仓库管理批量的镜像导出)****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker save --help

Usage:    docker save [OPTIONS] IMAGE [IMAGE...]

Save one or more images to a tar archive (streamed to STDOUT by default)

Options:
  -o, --output string   Write to a file, instead of STDOUT
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker save --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker save nginx -o /mnt/nginx.tar.gz                        #使用"-o"选项显式指定导出镜像到指定位置
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 127124
-rw-------. 1 root root 130174464 Jan 14 17:21 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker save nginx -o /mnt/nginx.tar.gz 　　　　　　　　　　　　#使用"-o"选项显式指定导出镜像到指定位置

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 127124
-rw-------. 1 root root 130174464 Jan 14 17:21 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker save nginx > /mnt/nginx-2020.tar.gz                    #也可以使用系统的重定向功能导出镜像文件。
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 254248
-rw-r--r--. 1 root root 130174464 Jan 14 17:23 nginx-2020.tar.gz
-rw-------. 1 root root 130174464 Jan 14 17:21 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# docker save nginx > /mnt/nginx-2020.tar.gz 　　　　　　　　　　#也可以使用系统的重定向功能导出镜像文件。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker save nginx -o /mnt/nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 127124
-rw-------. 1 root root 130174464 Jan 14 17:21 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# tar -xf /mnt/nginx-2020.tar.gz -C /mnt/                #解压导出的配置的镜像文件，我们可以查看该镜像的组织结构。
tar: manifest.json: implausibly old time stamp 1970-01-01 08:00:00
tar: repositories: implausibly old time stamp 1970-01-01 08:00:00
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 254264
drwxr-xr-x. 2 root root        50 Jan 10 06:20 bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc
-rw-r--r--. 1 root root      6670 Jan 10 06:20 c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7.json
drwxr-xr-x. 2 root root        50 Jan 10 06:20 da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e
drwxr-xr-x. 2 root root        50 Jan 10 06:20 fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d
-rw-r--r--. 1 root root       355 Jan  1  1970 manifest.json
-rw-r--r--. 1 root root 130174464 Jan 14 17:23 nginx-2020.tar.gz
-rw-------. 1 root root 130174464 Jan 14 17:21 nginx.tar.gz
-rw-r--r--. 1 root root        88 Jan  1  1970 repositories
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# cat /mnt/repositories                                 #记录了nginx镜像的版本号
{"nginx":{"latest":"da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e"}}
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# cat /mnt/manifest.json                                 #包含了镜像的相关配置，如配置文件(使用"Config"指定),镜像的版本号(使用"RepoTags"指定),分层信息(使用"Layers"指定,很明显该数组中有三个元素，说明有存储了三个分层的信息,继续往下看可探索究竟)。
[{"Config":"c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7.json","RepoTags":["nginx:latest"],"Layers":["bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer.tar","fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer.
tar","da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer.tar"]}]
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/
total 70792
-rw-r--r--. 1 root root      406 Jan 10 06:20 json
-rw-r--r--. 1 root root 72479232 Jan 10 06:20 layer.tar
-rw-r--r--. 1 root root        3 Jan 10 06:20 VERSION
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/
total 12
-rw-r--r--. 1 root root 1680 Jan 10 06:20 json
-rw-r--r--. 1 root root 3584 Jan 10 06:20 layer.tar
-rw-r--r--. 1 root root    3 Jan 10 06:20 VERSION
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/
total 56328
-rw-r--r--. 1 root root      482 Jan 10 06:20 json
-rw-r--r--. 1 root root 57670144 Jan 10 06:20 layer.tar
-rw-r--r--. 1 root root        3 Jan 10 06:20 VERSION
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# mkdir /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# tar -xf /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer.tar -C /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer
total 4
drwxr-xr-x. 16 root root 4096 Jan 10 06:20 etc
drwxr-xr-x.  5 root root   56 Jan 10 06:20 lib
drwxrwxrwt.  2 root root    6 Jan 10 06:20 tmp
drwxr-xr-x.  7 root root   66 Dec 24 08:00 usr
drwxr-xr-x.  5 root root   41 Dec 24 08:00 var
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# mkdir /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# tar -xf /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer.tar -C /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer
total 0
drwxr-xr-x. 3 root root 17 Dec 24 08:00 var
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# mkdir /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# tar -xf /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer.tar -C /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer
total 12
drwxr-xr-x.  2 root root 4096 Dec 24 08:00 bin
drwxr-xr-x.  2 root root    6 Nov 10 20:17 boot
drwxr-xr-x.  2 root root    6 Dec 24 08:00 dev
drwxr-xr-x. 28 root root 4096 Dec 24 08:00 etc
drwxr-xr-x.  2 root root    6 Nov 10 20:17 home
drwxr-xr-x.  7 root root   85 Dec 24 08:00 lib
drwxr-xr-x.  2 root root   34 Dec 24 08:00 lib64
drwxr-xr-x.  2 root root    6 Dec 24 08:00 media
drwxr-xr-x.  2 root root    6 Dec 24 08:00 mnt
drwxr-xr-x.  2 root root    6 Dec 24 08:00 opt
drwxr-xr-x.  2 root root    6 Nov 10 20:17 proc
drwx------.  2 root root   37 Dec 24 08:00 root
drwxr-xr-x.  3 root root   30 Dec 24 08:00 run
drwxr-xr-x.  2 root root 4096 Dec 24 08:00 sbin
drwxr-xr-x.  2 root root    6 Dec 24 08:00 srv
drwxr-xr-x.  2 root root    6 Nov 10 20:17 sys
drwxrwxrwt.  2 root root    6 Dec 24 08:00 tmp
drwxr-xr-x. 10 root root  105 Dec 24 08:00 usr
drwxr-xr-x. 11 root root  139 Dec 24 08:00 var
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/da4b42b1c7856aeb6c959188b9f628c0f928bec556dac7f1c87853e2519f3d5e/layer/var/
total 0
drwxr-xr-x. 3 root root 19 Jan 10 06:20 log
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/bc0c1f0bf5d3d116df7db18438f547181e0695612a45744f7964713c1126cebc/layer/var/
total 0
drwxr-xr-x. 2 root root  6 Nov 10 20:17 backups
drwxr-xr-x. 5 root root 48 Dec 24 08:00 cache
drwxr-xr-x. 7 root root 67 Dec 24 08:00 lib
drwxrwsr-x. 2 root ftp   6 Nov 10 20:17 local
lrwxrwxrwx. 1 root root  9 Dec 24 08:00 lock -> /run/lock
drwxr-xr-x. 3 root root 71 Dec 24 08:00 log
drwxrwsr-x. 2 root mem   6 Dec 24 08:00 mail
drwxr-xr-x. 2 root root  6 Dec 24 08:00 opt
lrwxrwxrwx. 1 root root  4 Dec 24 08:00 run -> /run
drwxr-xr-x. 2 root root 18 Dec 24 08:00 spool
drwxrwxrwt. 2 root root  6 Nov 10 20:17 tmp
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/fb21d4be2316838d12beecf48b8d7e832e099e2ac44d8ecd5b3e61180fbb7c9d/layer/var/
total 0
drwxr-xr-x. 6 root root 61 Jan 10 06:20 cache
drwxr-xr-x. 6 root root 55 Jan 10 06:20 lib
drwxr-xr-x. 4 root root 76 Jan 10 06:20 log
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -it --rm nginx bash
root@18e69ea2b98f:/# 
root@18e69ea2b98f:/# ls -l /
total 8
drwxr-xr-x.   2 root root 4096 Dec 24 00:00 bin
drwxr-xr-x.   2 root root    6 Nov 10 12:17 boot
drwxr-xr-x.   5 root root  360 Jan 14 09:48 dev
drwxr-xr-x.   1 root root   66 Jan 14 09:48 etc
drwxr-xr-x.   2 root root    6 Nov 10 12:17 home
drwxr-xr-x.   1 root root   56 Jan  9 22:20 lib
drwxr-xr-x.   2 root root   34 Dec 24 00:00 lib64
drwxr-xr-x.   2 root root    6 Dec 24 00:00 media
drwxr-xr-x.   2 root root    6 Dec 24 00:00 mnt
drwxr-xr-x.   2 root root    6 Dec 24 00:00 opt
dr-xr-xr-x. 136 root root    0 Jan 14 09:48 proc
drwx------.   2 root root   37 Dec 24 00:00 root
drwxr-xr-x.   3 root root   30 Dec 24 00:00 run
drwxr-xr-x.   2 root root 4096 Dec 24 00:00 sbin
drwxr-xr-x.   2 root root    6 Dec 24 00:00 srv
dr-xr-xr-x.  13 root root    0 Jan 13 08:35 sys
drwxrwxrwt.   1 root root    6 Jan  9 22:20 tmp
drwxr-xr-x.   1 root root   66 Dec 24 00:00 usr
drwxr-xr-x.   1 root root   17 Dec 24 00:00 var
root@18e69ea2b98f:/# 
root@18e69ea2b98f:/# 
root@18e69ea2b98f:/# ls -l /var/                                    #不难发现，容器中我们能看到的目录或文件，是上面三个分层文件对应的"/var"目录总和。因此我们在实际生活中压根就不关心有多少个分层，而是关心的镜像的使用。
total 0
drwxr-xr-x. 2 root root   6 Nov 10 12:17 backups
drwxr-xr-x. 1 root root  61 Jan  9 22:20 cache
drwxr-xr-x. 1 root root  55 Jan  9 22:20 lib
drwxrwsr-x. 2 root staff  6 Nov 10 12:17 local
lrwxrwxrwx. 1 root root   9 Dec 24 00:00 lock -> /run/lock
drwxr-xr-x. 1 root root  19 Jan  9 22:20 log
drwxrwsr-x. 2 root mail   6 Dec 24 00:00 mail
drwxr-xr-x. 2 root root   6 Dec 24 00:00 opt
lrwxrwxrwx. 1 root root   4 Dec 24 00:00 run -> /run
drwxr-xr-x. 2 root root  18 Dec 24 00:00 spool
drwxrwxrwt. 2 root root   6 Nov 10 12:17 tmp
root@18e69ea2b98f:/# 
root@18e69ea2b98f:/# exit 
exit
[root@docker201.yinzhengjie.org.cn ~]# 
```

查看导出镜像的分层信息详解(详情请戳这里)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114092807350-1502254751.png)

**6>.镜像导入(镜像的导入使用命令临时导入可以，但在生产环境不推荐使用，生产环境应该用私有仓库管理批量的镜像导入)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker load --help

Usage:    docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker load --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images                                                #查看当前宿主机现有的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker save nginx -o /mnt/nginx.tar.gz                        #我选择将nginx的镜像导出
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 127124
-rw-------. 1 root root 130174464 Jan 14 19:38 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# hostname -i
192.168.6.201
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# scp /mnt/nginx.tar.gz docker202.yinzhengjie.org.cn:~            #将导出的镜像拷贝到其它宿主机上
The authenticity of host 'docker202.yinzhengjie.org.cn (192.168.6.202)' can't be established.
ECDSA key fingerprint is SHA256:pCZRyla5kyGXvY13DnVliTHvaDFovhdVv9KflRDHuNw.
ECDSA key fingerprint is MD5:30:3c:4f:ca:9d:f8:40:6e:30:4d:ce:40:32:64:46:e8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'docker202.yinzhengjie.org.cn,192.168.6.202' (ECDSA) to the list of known hosts.
root@docker202.yinzhengjie.org.cn's password: 
nginx.tar.gz                                                                                                                                                                                                                                100%  124MB 124.1MB/s   00:01    
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ssh docker202.yinzhengjie.org.cn                                #远程到docker202.yinzhengjie.org.cn便于演示docker的
root@docker202.yinzhengjie.org.cn's password: 
Last login: Mon Jan 13 17:01:25 2020 from 192.168.0.1
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# hostname -i
192.168.6.202
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker --version                                                #查看当前docker的版本
Docker version 19.03.5, build 633a0ea
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# ll
total 127124
-rw-------. 1 root root 130174464 Jan 14 19:39 nginx.tar.gz
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images                                                #很明显当前宿主机是没有docker镜像的
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker load -i nginx.tar.gz                                     #我们将nginx镜像导入到当前宿主机上
556c5fb0d91b: Loading layer [==================================================>]  72.48MB/72.48MB
17fde96446df: Loading layer [==================================================>]  57.67MB/57.67MB
c26e88311e71: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: nginx:latest
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images                                                #镜像导入成功啦
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker load -i nginx.tar.gz 　　　　　　　　#我们将nginx镜像导入到当前宿主机上

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker images                                                #查看当前宿主机现有的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
alpine              latest              cc0abc535e36        2 weeks ago         5.59MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 0
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker save nginx -o /mnt/nginx.tar.gz                        #我选择将nginx的镜像导出
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ll /mnt/
total 127124
-rw-------. 1 root root 130174464 Jan 14 19:38 nginx.tar.gz
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# hostname -i
192.168.6.201
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# scp /mnt/nginx.tar.gz docker202.yinzhengjie.org.cn:~            #将导出的镜像拷贝到其它宿主机上
The authenticity of host 'docker202.yinzhengjie.org.cn (192.168.6.202)' can't be established.
ECDSA key fingerprint is SHA256:pCZRyla5kyGXvY13DnVliTHvaDFovhdVv9KflRDHuNw.
ECDSA key fingerprint is MD5:30:3c:4f:ca:9d:f8:40:6e:30:4d:ce:40:32:64:46:e8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'docker202.yinzhengjie.org.cn,192.168.6.202' (ECDSA) to the list of known hosts.
root@docker202.yinzhengjie.org.cn's password: 
nginx.tar.gz                                                                                                                                                                                                                                100%  124MB 124.1MB/s   00:01    
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ssh docker202.yinzhengjie.org.cn                                #远程到docker202.yinzhengjie.org.cn便于演示docker的
root@docker202.yinzhengjie.org.cn's password: 
Last login: Mon Jan 13 17:01:25 2020 from 192.168.0.1
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# hostname -i
192.168.6.202
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker --version                                                #查看当前docker的版本
Docker version 19.03.5, build 633a0ea
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# ll
total 127124
-rw-------. 1 root root 130174464 Jan 14 19:39 nginx.tar.gz
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images                                                #很明显当前宿主机是没有docker镜像的
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker load < nginx.tar.gz                                     #我们也可以使用系统的标准输入来导入镜像哟~
556c5fb0d91b: Loading layer [==================================================>]  72.48MB/72.48MB
17fde96446df: Loading layer [==================================================>]  57.67MB/57.67MB
c26e88311e71: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: nginx:latest
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images                                                #镜像导入成功啦
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker load < nginx.tar.gz 　　　　　　　　#我们也可以使用系统的标准输入来导入镜像哟~

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114115900270-1918359926.png)

**7>.删除镜像(温馨提示:通过镜像启动容器时镜像不能被删除，除非将容器全部关闭)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker image rm --help

Usage:    docker image rm [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Aliases:
  rm, rmi, remove

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker image rm --help

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker image rm -f nginx:latest 
Untagged: nginx:latest
Deleted: sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7
Deleted: sha256:3e51598e49c550f8b212a07c6ff2ed47a09eeb637f67d1b3c5468e9a8ee646e3
Deleted: sha256:a8b9a5643b3cc8082997d3d2fbaf4b53213ff80aa4169226be8b3768ae6e3605
Deleted: sha256:556c5fb0d91b726083a8ce42e2faaed99f11bc68d3f70e2c7bbce87e7e0b3e10
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker image rm -f nginx:latest

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker image rmi nginx                                    #使用rmr别名删除镜像。
Untagged: nginx:latest
Deleted: sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7
Deleted: sha256:3e51598e49c550f8b212a07c6ff2ed47a09eeb637f67d1b3c5468e9a8ee646e3
Deleted: sha256:a8b9a5643b3cc8082997d3d2fbaf4b53213ff80aa4169226be8b3768ae6e3605
Deleted: sha256:556c5fb0d91b726083a8ce42e2faaed99f11bc68d3f70e2c7bbce87e7e0b3e10
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker image rmi nginx 　　　　　　　　　　　　#使用rmr别名删除镜像。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker image remove nginx                            #使用remove别名来删除镜像
Untagged: nginx:latest
Deleted: sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7
Deleted: sha256:3e51598e49c550f8b212a07c6ff2ed47a09eeb637f67d1b3c5468e9a8ee646e3
Deleted: sha256:a8b9a5643b3cc8082997d3d2fbaf4b53213ff80aa4169226be8b3768ae6e3605
Deleted: sha256:556c5fb0d91b726083a8ce42e2faaed99f11bc68d3f70e2c7bbce87e7e0b3e10
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker image remove nginx 　　　　　　　　　　#使用remove别名来删除镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        4 days ago          126MB
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker image rm nginx                        #也可以不是使用选项和版本号就删除镜像啦~
Untagged: nginx:latest
Deleted: sha256:c7460dfcab502275e9c842588df406444069c00a48d9a995619c243079a4c2f7
Deleted: sha256:3e51598e49c550f8b212a07c6ff2ed47a09eeb637f67d1b3c5468e9a8ee646e3
Deleted: sha256:a8b9a5643b3cc8082997d3d2fbaf4b53213ff80aa4169226be8b3768ae6e3605
Deleted: sha256:556c5fb0d91b726083a8ce42e2faaed99f11bc68d3f70e2c7bbce87e7e0b3e10
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# docker image rm nginx 　　　　　　　　　　　　#也可以不是使用选项和版本号就删除镜像啦~

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                       IMAGE ID            CREATED             SIZE
nginx               make-v0.1-20200121-0846   e8520d2c15ae        13 hours ago        590MB
<none>              <none>                    5d81119e1e53        13 hours ago        590MB
<none>              <none>                    fd9dcda01aa8        14 hours ago        590MB
<none>              <none>                    896b954fd6d4        14 hours ago        590MB
<none>              <none>                    88d854efd276        14 hours ago        589MB
<none>              <none>                    252190d9a459        14 hours ago        589MB
centos              centos7.6.1810            f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls | grep "<none>" | awk '{print $3}'
5d81119e1e53
fd9dcda01aa8
896b954fd6d4
88d854efd276
252190d9a459
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls | grep "<none>" | awk '{print $3}' | xargs docker image rm -f
Deleted: sha256:5d81119e1e53c745e211ef4b3125b87ec5a531232974730684b370c1ceebc194
Deleted: sha256:d19549b10923d929766ecfcfb62658427899f59e2375d13b41ec45ad604dc8a2
Deleted: sha256:1ff90831f4bcebe11d7b00bd4902432ea4432771aa808983762834e0b870997b
Deleted: sha256:339606b2fb902370f8dd7598168ef199f9f8ab9e93acd76001590f4f6e260ad6
Deleted: sha256:6bac141bd71ce160b62d5a2174d5f9c13aea3e6af0eae5486531b7bc1c8d5c9e
Deleted: sha256:9b1c1c9906915b4471649692d56d9b5baf56b8266ce08f82788abd5a71625af8
Deleted: sha256:1d7a07a2065472071dbd0f62103ee492b721a1ceeb7d3659bad333bf51e6b064
Deleted: sha256:55ee57c23b5eb0b887ef7e8a6f686039a6bbb2d50a31727f9a67f0b21a079000
Deleted: sha256:fd9dcda01aa8c7f959789ee7cfac20792b49fd21ee55bcda21ed73f74d1054ad
Deleted: sha256:d4d8385c5bebebf9ae66126b2f99ed98f149f65fd5aa86ea9c75ce6e220737b9
Deleted: sha256:9c27648ace6d7b372b86373e3123f407df07e72b82bf8c2de0df4a8a245a6aec
Deleted: sha256:809ea28213aa3f6dea3d49c99573932f141d1bce773d3845896f08395d301e5c
Deleted: sha256:2ba35149b2615fc2b907781aba15eae846f6df44b2234731c5de455a61a9d5a3
Deleted: sha256:91a10b67eb5bbd0e60f13ee73ed08adad96c4981972d6683b4f76e759219718c
Deleted: sha256:1bc21eecb9d6ade378a372ed22270620d76e6f6859c390ff44770b839f940be3
Deleted: sha256:8e7ea65e50acb85d5dee68a294d1fd1cf9b6f410a4a61a59da064238e26c3032
Deleted: sha256:896b954fd6d4615b14eb41fccf0da17cb7676e45a25acb6ef7a5b8752f99e2d7
Deleted: sha256:0ce070ab31b2dab386e5e72f455bfb66bd8d0f2f6893824dcbded8335bc7df45
Deleted: sha256:ef4b2beb283b8941bb4a838f1c5908d11723d4571e815c740bd4820f0a3d04fb
Deleted: sha256:c7264fecc0528961f3ec98518b2020692e73217bba38bf689d77d541ef8fc0be
Deleted: sha256:88d854efd2763db9ae91271093004d9115c2e9cfa5ecafb0ac8b3307509bf8a4
Deleted: sha256:57cf06b029befe6668b3e23e71a1ebf815bd2a2e498f7706255792172f8995c3
Deleted: sha256:9ba534d5aba7f51d73154d514976736abe1b8ef96a80fa41befcbf7bd748c4b0
Deleted: sha256:471ec31934abb57e770dceb97c9afa199dbdae9e7d2249c95d82425fc141398c
Deleted: sha256:89a716428892aaf8130a7740305b4348a8f87ee2fbf6e3f5f0bb092fc34f080c
Deleted: sha256:b267e08815cf1221a8308df0ff8ff44b7a9acb9a26db4ea7dfd6fce47c2d2ec5
Deleted: sha256:252190d9a459cbf96f3f48d94b1ba48ef33a301f2102fcf198700ce2e2ad4d28
Deleted: sha256:306e3efc6b46c51052b767c8f721678215f5b6425d2be547161a0d14685452f8
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                       IMAGE ID            CREATED             SIZE
nginx               make-v0.1-20200121-0846   e8520d2c15ae        13 hours ago        590MB
centos              centos7.6.1810            f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image ls | grep "" | awk '{print $3}' | xargs docker image rm -f　　　　　　　　#批量删除镜像

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200114121555784-773629603.png)

**三.制作Docker镜像**

```
博主推荐阅读:
　　https://www.cnblogs.com/yinzhengjie/p/12190111.html
```

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186