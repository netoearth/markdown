　　　　　　　　　　**Docker镜像-基于DockerFile制作编译版nginx镜像**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.下载nginx的安装包**

**1>.打开nginx官网，找到最新稳定版下载即可(http://nginx.org/en/download.html)**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200121085749082-1904026692.png)

**2>.下载nginx安装包**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cd /yinzhengjie/softwares/dockerfile/web/nginx/
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 12
-rwxr-xr-x 1 root root  461 Jan 21 16:37 docker-build.sh
-rw-r--r-- 1 root root 1833 Jan 21 16:44 Dockerfile
-rw-r--r-- 1 root root 1979 Jan 21 16:31 nginx.conf
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# wget http://nginx.org/download/nginx-1.14.2.tar.gz
--2020-01-21 16:53:09--  http://nginx.org/download/nginx-1.14.2.tar.gz
Resolving nginx.org (nginx.org)... 95.211.80.227, 62.210.92.35, 2001:1af8:4060:a004:21::e3
Connecting to nginx.org (nginx.org)|95.211.80.227|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1015384 (992K) [application/octet-stream]
Saving to: ‘nginx-1.14.2.tar.gz’

100%[==================================================================================================================================>] 1,015,384    355KB/s   in 2.8s   

2020-01-21 16:53:13 (355 KB/s) - ‘nginx-1.14.2.tar.gz’ saved [1015384/1015384]

[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 1004
-rwxr-xr-x 1 root root     461 Jan 21 16:37 docker-build.sh
-rw-r--r-- 1 root root    1833 Jan 21 16:44 Dockerfile
-rw-r--r-- 1 root root 1015384 Dec  4  2018 nginx-1.14.2.tar.gz
-rw-r--r-- 1 root root    1979 Jan 21 16:31 nginx.conf
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.属性编译安装nginx的步骤(如果你不清楚nginx的编译步骤请估计写不出来Dockerfile的，因此你得了解nginx的编译安装步骤，不熟悉的没关系，看一下我之前的笔记即可，熟悉的同学跳过该步骤)**

```
博主推荐阅读:
　　https://www.cnblogs.com/yinzhengjie/p/12031651.html
```

**二.准备测试数据**

**1>.模拟打包生产环境代码**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll
total 0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "<h1>YinZhengjie's Nginx Web Server</h1>" > index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "<h1>YinZhengjie's Nginx Web Server 2020</h1>" > index2020.html 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll
total 8
-rw-r--r-- 1 root root 45 Jan 21 17:38 index2020.html
-rw-r--r-- 1 root root 40 Jan 21 17:38 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# tar zcvf code.tar.gz index*
index2020.html
index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll
total 12
-rw-r--r-- 1 root root 197 Jan 21 17:38 code.tar.gz
-rw-r--r-- 1 root root  45 Jan 21 17:38 index2020.html
-rw-r--r-- 1 root root  40 Jan 21 17:38 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/softwares/dockerfile/web/nginx/
total 1004
-rwxr-xr-x 1 root root     461 Jan 21 16:37 docker-build.sh
-rw-r--r-- 1 root root    2603 Jan 21 17:34 Dockerfile
-rw-r--r-- 1 root root 1015384 Dec  4  2018 nginx-1.14.2.tar.gz
-rw-r--r-- 1 root root    1979 Jan 21 16:31 nginx.conf
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cp code.tar.gz /yinzhengjie/softwares/dockerfile/web/nginx/
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cd /yinzhengjie/softwares/dockerfile/web/nginx/
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 1008
-rw-r--r-- 1 root root     197 Jan 21 17:39 code.tar.gz
-rwxr-xr-x 1 root root     461 Jan 21 16:37 docker-build.sh
-rw-r--r-- 1 root root    2603 Jan 21 17:34 Dockerfile
-rw-r--r-- 1 root root 1015384 Dec  4  2018 nginx-1.14.2.tar.gz
-rw-r--r-- 1 root root    1979 Jan 21 16:31 nginx.conf
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200121094016268-697268798.png)

**2>.编写nginx的配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/nginx/nginx.conf 
user nginx;

worker_processes auto;

#错误日志也应该指定到咱们编译安装的nginx目录中
error_log /yinzhengjie/softwares/nginx/logs/error.log;

pid /run/nginx.pid;

#Docker最终运行Nginx建议大家把后台进程关闭,默认是"on".
daemon off;

events {
    worker_connections 1024;
}
   
http {
    #自定义Nginx的日志格式
    log_format my_access_json '{"@timestamp":"$time_iso8601",' 
                               '"host":"$server_addr",' 
                   　　　　　　　'"clientip":"$remote_addr",' 
                   　　　　　　  '"size":$body_bytes_sent,' 
                   　　　　　　　'"responsetime":$request_time,' 
                   　　　　　　　'"upstreamtime":"$upstream_response_time",' 
                   　　　　　　　'"upstreamhost":"$upstream_addr",' 
                   　　　　　　　'"http_host":"$host",' 
                   　　　　　　　'"uri":"$uri",' 
                   　　　　　　　'"domain":"$host",' 
                   　　　　　　　'"xff":"$http_x_forwarded_for",' 
                   　　　　　　　'"referer":"$http_referer",' 
                   　　　　　　　'"tcp_xff":"$proxy_protocol_addr",' 
                   　　　　　　　'"http_user_agent":"$http_user_agent",' 
                   　　　　　　　'"status":"$status"}';
    #同理，我们将访问日志也放到咱们编译安装nginx的目录中             
    access_log /yinzhengjie/softwares/nginx/logs/access_json.log my_access_json;

    sendfile            on;
    keepalive_timeout   65;
    include       mime.types;
    default_type  text/html;
    charset utf-8;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;

        #别忘记修改站点代码的根目录也要指定到咱们编译安装nginx的目录中哟~
        root         /yinzhengjie/softwares/nginx/html;
        location / {
        }
        error_page 404 /404.html;             
        location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.编写快速编译docker的脚本(我这里就是抛砖引玉了,你可以自行优化)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat docker-build.sh 
#!/bin/bash
#
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2020-01-18
#FileName：        docker-build.sh
#URL:             http://www.cnblogs.com/yinzhengjie
#Description：        The test script
#Copyright (C):     2020 All rights reserved
#********************************************************************


TAG=$1

docker image build -t nginx:${TAG} ./
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200121094347455-1119399672.png)

**4>.编写Dockerfile**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/nginx/Dockerfile 
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2019-11-25
#Blog:             http://www.cnblogs.com/yinzhengjie
#Description：        YinZhengjie's Nginx Dockerfile
#Copyright notice:     original works, no reprint! Otherwise, legal liability will be investigated.
#********************************************************************

#第一行先定义基础镜像,表示当前镜像文件是基于哪个进行编辑的.
FROM centos:centos7.6.1810

#指定镜像维护者的信息.
MAINTAINER Jason.Yin y1053419035@qq.com

#除了安装编译nginx的依赖的安装包外,还可以将一些常用的命令工具也安装上
#类似于这样的安装命令(或者经常改动相对较小的命令)应该尽量往前写,这样在多次编译时就不会重复执行了(因为默认会使用缓存),从而提升编译效率.
RUN yum -y install epel-release && yum -y install vim net-tools bridge-utils firewalld bc iotop bc gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel zip unzip zlib-devel lrzsz tree ntpdate telnet lsof tcpdump wget libevent libevent-devel systemd-devel bash-completion traceroute psmisc 
#将"nginx-1.14.2.tar.gz"安装包文件解压到"/usr/local/src"目录中
#相比COPY指令,ADD指令可以解压"*.tar.gz"的文件,但如果你的安装包是"*.zip"文件的话,ADD指令也不好使,得咱们自己使用unzip相关命令自行解压,索性我上面已经安装了unzip相关的软件包ADD nginx-1.14.2.tar.gz /usr/local/src

#编译安装nginx
RUN cd /usr/local/src/nginx-1.14.2 && ./configure --prefix=/yinzhengjie/softwares/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make && make install && ln -sv /yinzhengjie/softwares/nginx/sbin/nginx /usr/sbin/nginx
#将nginx的配置文件放在镜像的指定编译安装的目录"/yinzhengjie/softwares/nginx/conf"
COPY nginx.conf /yinzhengjie/softwares/nginx/conf

#将测试代码解压到咱们编译安装的nginx网站默认根目录中
ADD code.tar.gz /yinzhengjie/softwares/nginx/html

#创建nginx用户,yum方式安装无需做此步骤，因为默认yum安装会自动创建nginx用户,咱们上面指令了以nginx用户运行,因此我们需要在镜像中创建"nginx用户"
RUN useradd nginx -s /sbin/nologin -u 2000

#定义向外暴露的端口号,多个端口用空格做间隔,启动容器的时候"-p"需要使用此端向外端映射.
EXPOSE 80/tcp 443/tcp

#定义前台运行的命令,每个Docker只能有一条，如果定义了多条"CMD"指令那么最后一条CMD指令会覆盖之前的(即只有最后一条CMD被执行).
CMD ["nginx"]
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.编译生成nginx镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                       IMAGE ID            CREATED             SIZE
nginx               make-v0.1-20200121-0846   5d81119e1e53        9 minutes ago       590MB
<none>              <none>                    fd9dcda01aa8        12 minutes ago      590MB
<none>              <none>                    896b954fd6d4        14 minutes ago      590MB
<none>              <none>                    88d854efd276        18 minutes ago      589MB
<none>              <none>                    252190d9a459        53 minutes ago      589MB
centos              centos7.6.1810            f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cd /yinzhengjie/softwares/dockerfile/web/nginx/
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 1008
-rw-r--r-- 1 root root     197 Jan 21 17:39 code.tar.gz
-rwxr-xr-x 1 root root     461 Jan 21 16:37 docker-build.sh
-rw-r--r-- 1 root root    2867 Jan 21 18:02 Dockerfile
-rw-r--r-- 1 root root 1015384 Dec  4  2018 nginx-1.14.2.tar.gz
-rw-r--r-- 1 root root    2198 Jan 21 18:12 nginx.conf
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ./docker-build.sh make-v0.1-20200121-0846
Sending build context to Docker daemon  1.026MB
Step 1/10 : FROM centos:centos7.6.1810
 ---> f1cb7c7d58b7
Step 2/10 : MAINTAINER Jason.Yin y1053419035@qq.com
 ---> Using cache
 ---> 44bbf438782f
Step 3/10 : RUN yum -y install epel-release && yum -y install vim net-tools bridge-utils firewalld bc iotop bc gcc gcc-c++ glibc glibc-devel pcre pcre-
devel openssl openssl-devel zip unzip zlib-devel lrzsz tree ntpdate telnet lsof tcpdump wget libevent libevent-devel systemd-devel bash-completion traceroute psmisc ---> Using cache
 ---> c8282c8fcd0c
Step 4/10 : ADD nginx-1.14.2.tar.gz /usr/local/src
 ---> Using cache
 ---> 117d35e96dc3
Step 5/10 : RUN cd /usr/local/src/nginx-1.14.2 && ./configure --prefix=/yinzhengjie/softwares/nginx --user=nginx --group=nginx --with-http_ssl_module -
-with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make && make install && ln -sv /yinzhengjie/softwares/nginx/sbin/nginx /usr/sbin/nginx ---> Using cache
 ---> 5b687e6a0dad
Step 6/10 : COPY nginx.conf /yinzhengjie/softwares/nginx/conf
 ---> 2891153689a3
Step 7/10 : ADD code.tar.gz /yinzhengjie/softwares/nginx/html
 ---> a4847d0e3b19
Step 8/10 : RUN useradd nginx -s /sbin/nologin -u 2000
 ---> Running in 7562ef0c366a
Removing intermediate container 7562ef0c366a
 ---> e700a97c15de
Step 9/10 : EXPOSE 80/tcp 443/tcp
 ---> Running in cdc6364a25e7
Removing intermediate container cdc6364a25e7
 ---> fdc19386ef1f
Step 10/10 : CMD ["nginx"]
 ---> Running in acc980fae2ad
Removing intermediate container acc980fae2ad
 ---> e8520d2c15ae
Successfully built e8520d2c15ae
Successfully tagged nginx:make-v0.1-20200121-0846
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# docker image ls
REPOSITORY          TAG                       IMAGE ID            CREATED             SIZE
nginx               make-v0.1-20200121-0846   e8520d2c15ae        3 seconds ago       590MB
<none>              <none>                    5d81119e1e53        9 minutes ago       590MB
<none>              <none>                    fd9dcda01aa8        13 minutes ago      590MB
<none>              <none>                    896b954fd6d4        14 minutes ago      590MB
<none>              <none>                    88d854efd276        18 minutes ago      589MB
<none>              <none>                    252190d9a459        54 minutes ago      589MB
centos              centos7.6.1810            f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.验证编译版本的nginx镜像**

**1>.运行容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                       IMAGE ID            CREATED             SIZE
nginx               make-v0.1-20200121-0846   e8520d2c15ae        About an hour ago   590MB
<none>              <none>                    5d81119e1e53        About an hour ago   590MB
<none>              <none>                    fd9dcda01aa8        About an hour ago   590MB
<none>              <none>                    896b954fd6d4        About an hour ago   590MB
<none>              <none>                    88d854efd276        About an hour ago   589MB
<none>              <none>                    252190d9a459        2 hours ago         589MB
centos              centos7.6.1810            f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm -p 172.200.3.101:8080:80 --name myNginx --hostname yinzhengjie nginx:make-v0.1-20200121-0846 　　　　　　#运行容器后会阻塞在当前终端，需要再开启一个终端连接该容器,如下图所示。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200121112517873-812937062.png)

**2>.客户端访问时最好先连接容器查看日志记录**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS              PORTS                                 NAMES
86cbd7aa982c        nginx:make-v0.1-20200121-0846   "nginx"             5 minutes ago       Up 5 minutes        443/tcp, 172.200.3.101:8080->80/tcp   myNginx
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container inspect -f "{{.NetworkSettings.IPAddress}}" myNginx
172.17.0.2
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                 Local Address:Port                                                                Peer Address:Port              
LISTEN     0      20480                                                  172.200.3.101:8080                                                                           *:*                  
LISTEN     0      128                                                                *:22                                                                             *:*                  
LISTEN     0      128                                                               :::22                                                                            :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it myNginx bash
[root@yinzhengjie /]# 
[root@yinzhengjie /]# tail -10f /yinzhengjie/softwares/nginx/logs/access_json.log 
{"@timestamp":"2020-01-21T11:29:57+00:00","host":"172.17.0.2",　　　　　　　'"clientip":"172.200.0.1",'　　　　　　"size":0,　　　　　　　'"responsetime":0.000,'　　　　　　　'"upstreamtim
e":"-",'　　　　　　　'"upstreamhost":"-",'　　　　　　　'"http_host":"docker101.yinzhengjie.org.cn",'　　　　　　　'"uri":"/index.html",'　　　　　　　'"domain":"docker101.yinzhengjie.org.cn",'　　　　　　　'"xff":"-",'　　　　　　　'"referer":"-",'　　　　　　　'"tcp_xff":"",'　　　　　　　'"http_user_agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36",'　　　　　　　'"status":"304"}'{"@timestamp":"2020-01-21T11:30:31+00:00","host":"172.17.0.2",　　　　　　　'"clientip":"172.200.0.1",'　　　　　　"size":45,　　　　　　　'"responsetime":0.000,'　　　　　　　'"upstreamti
me":"-",'　　　　　　　'"upstreamhost":"-",'　　　　　　　'"http_host":"docker101.yinzhengjie.org.cn",'　　　　　　　'"uri":"/index2020.html",'　　　　　　　'"domain":"docker101.yinzhengjie.org.cn",'　　　　　　　'"xff":"-",'　　　　　　　'"referer":"-",'　　　　　　　'"tcp_xff":"",'　　　　　　　'"http_user_agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36",'　　　　　　　'"status":"200"}'
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200121113321634-1626092053.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186