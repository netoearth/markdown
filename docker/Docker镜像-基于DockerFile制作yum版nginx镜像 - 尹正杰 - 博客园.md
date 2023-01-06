　　　　　　　　　　**Docker镜像-基于DockerFile制作yum版nginx镜像**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

　　**DockerFile可以说是一种能被Docker程序解释的脚本，DockerFile是一条条指令组成的，每一条指令对应Linux下面的一条命令，Docker程序将这些DockerFile指令再翻译成真正的Linux命令，其有自己的书写方式和支持的命令。**

　　**Docker程序读取DockerFile并根据指令生成对应的Docker镜像，相比手动生成镜像的方式(可参考:https://www.cnblogs.com/yinzhengjie/p/12190111.html)，DockerFile更能直观的展示镜像是怎么产生的，有了DockerFile，当后期有额外的需求时，只要在之前的DockerFile添加或修改响应的命令即可重新生成新的Docker镜像，避免了重复手动执行镜像的麻烦。**

**一.下载centos镜像并初始化系统**

**1>.下载CentOS指定版本镜像(默认下载最新的版本，而我指定的是CentOS 7.6主流版本)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image pull centos:centos7.6.1810
centos7.6.1810: Pulling from library/centos
ac9208207ada: Pull complete 
Digest: sha256:62d9e1c2daa91166139b51577fe4f4f6b4cc41a3a2c7fc36bd895e2a17a3e4e6
Status: Downloaded newer image for centos:centos7.6.1810
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.在宿主机上创建存放DockerFile的存储目录(目录结构按照业务类型或者系统类型等方式划分，方便后期镜像比较多的时候进行分类)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# mkdir /yinzhengjie/softwares/dockerfile/{web/{apache,nginx,tomcat,jdk},system/{centos,ubantu,redhat,suse,debain}} -pv
mkdir: created directory ‘/yinzhengjie’
mkdir: created directory ‘/yinzhengjie/softwares’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/web’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/web/apache’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/web/nginx’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/web/tomcat’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/web/jdk’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system/centos’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system/ubantu’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system/redhat’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system/suse’
mkdir: created directory ‘/yinzhengjie/softwares/dockerfile/system/debain’
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.**准备源码包与配置文件****

**1>.模拟打包生产环境代码**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 20
-rwxr-xr-x. 1 root root  462 Jan 18 20:25 docker-build.sh
-rw-r--r--. 1 root root 1758 Jan 19 01:38 Dockerfile
-rw-r--r--. 1 root root   45 Jan 18 19:54 index2020.html
-rw-r--r--. 1 root root   40 Jan 18 17:05 index.html
-rw-r--r--. 1 root root 1711 Jan 19 01:55 nginx.conf
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat index.html 
<h1>YinZhengjie's Nginx Web Server</h1>
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat index2020.html 
<h1>YinZhengjie's Nginx Web Server 2020</h1>
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# tar zcvf code.tar.gz index.html index2020.html 
index.html
index2020.html
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ll
total 24
-rw-r--r--. 1 root root  199 Jan 19 02:06 code.tar.gz
-rwxr-xr-x. 1 root root  462 Jan 18 20:25 docker-build.sh
-rw-r--r--. 1 root root 1758 Jan 19 01:38 Dockerfile
-rw-r--r--. 1 root root   45 Jan 18 19:54 index2020.html
-rw-r--r--. 1 root root   40 Jan 18 17:05 index.html
-rw-r--r--. 1 root root 1711 Jan 19 01:55 nginx.conf
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.编写nginx的配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat nginx.conf 
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;

#Docker最终运行Nginx建议大家把后台进程关闭,默认是"on".
daemon off;

include /usr/share/nginx/modules/*.conf;

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
              
    access_log /var/log/nginx/access_json.log my_access_json;

    sendfile            on;
    keepalive_timeout   65;
    include       mime.types;
    default_type  text/html;
    charset utf-8;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;

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
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.**编写Dockerfile****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat Dockerfile 
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

#定义执行的命令，将安装nginx的步骤执行一遍。建议大家把多条RUN命令使用"&&"符号写成一行,这样可以减少镜像的层数(Layer),
#类似于这样的安装命令(或者经常改动相对较小的命令)应该尽量往前写,这样在多次编译时就不会重复执行了(因为默认会使用缓存),从而提升编译效率.
RUN yum -y install epel-release && yum -y install nginx && yum -y install net-tools vim wget pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop

#将nginx的配置文件放在镜像的指定目录"/etc/nginx/"
ADD nginx.conf /etc/nginx/

#如果"code.tar.gz"是压缩文件则自动解压压缩包,并将解压的结果放在镜像的指定目录"/usr/share/nginx/html".
ADD code.tar.gz /usr/share/nginx/html

#定义向外暴露的端口号,多个端口用空格做间隔,启动容器的时候"-p"需要使用此端向外端映射.
EXPOSE 80/tcp 443/tcp

#定义前台运行的命令,每个Docker只能有一条，如果定义了多条"CMD"指令那么最后一条CMD指令会覆盖之前的(即只有最后一条CMD被执行).
CMD ["nginx"]

[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**四.执行镜像构建**

**1>.自定义简单的构建镜像的脚本(我这里抛砖引玉，你可以自行扩充)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# cat docker-build.sh 
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

[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.使用自定义的脚本构建镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# docker image ls
REPOSITORY            TAG                  IMAGE ID            CREATED             SIZE
nginx                 v0.2-20200118-1750   ef094745c27f        16 minutes ago      448MB
jason/centos7-nginx   v0.1.20200114        7372d16c99bc        31 hours ago        448MB
jason/centos7-nginx   latest               c4e0980a825a        32 hours ago        448MB
mysql                 5.6.44               c30095c52827        6 months ago        256MB
centos                centos7.6.1810       f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# ./docker-build.sh v0.2-20200118-1750　　　　　　　　#不难发现，当前的镜像版本号已经存在啦~执行后之前的镜像文件的版本号将被重置为"<none>"
Sending build context to Docker daemon  10.24kB
Step 1/7 : FROM centos:centos7.6.1810
 ---> f1cb7c7d58b7
Step 2/7 : MAINTAINER Jason.Yin y1053419035@qq.com
 ---> Using cache
 ---> edd0bcfc5908
Step 3/7 : RUN yum -y install epel-release && yum -y install nginx && yum -y install net-tools vim wget pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop
 ---> Using cache
 ---> 779cda1b20a1
Step 4/7 : ADD nginx.conf /etc/nginx/
 ---> adaaeace9a08
Step 5/7 : ADD code.tar.gz /usr/share/nginx/html
 ---> 415004a0dc4d
Step 6/7 : EXPOSE 80/tcp 443/tcp
 ---> Running in 4e9071f1c399
Removing intermediate container 4e9071f1c399
 ---> 5c91af75787c
Step 7/7 : CMD ["nginx"]
 ---> Running in c0cbd8d86a50
Removing intermediate container c0cbd8d86a50
 ---> abb7704b09d7
Successfully built abb7704b09d7
Successfully tagged nginx:v0.2-20200118-1750
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# docker image ls
REPOSITORY            TAG                  IMAGE ID            CREATED             SIZE
nginx                 v0.2-20200118-1750   abb7704b09d7        2 seconds ago       448MB
<none>                <none>               ef094745c27f        17 minutes ago      448MB
jason/centos7-nginx   v0.1.20200114        7372d16c99bc        31 hours ago        448MB
jason/centos7-nginx   latest               c4e0980a825a        32 hours ago        448MB
mysql                 5.6.44               c30095c52827        6 months ago        256MB
centos                centos7.6.1810       f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
[root@docker201.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**五.使用咱们自己的镜像测试配置是否生效**

**1>.基于咱们自制的镜像启动容器需要做端口映射**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container  ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d1197bf0e557        mysql:5.6.44        "docker-entrypoint.s…"   4 hours ago         Up 4 hours          0.0.0.0:3306->3306/tcp   vibrant_rosalind
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY            TAG                  IMAGE ID            CREATED             SIZE
nginx                 v0.2-20200118-1750   abb7704b09d7        8 minutes ago       448MB
<none>                <none>               ef094745c27f        25 minutes ago      448MB
jason/centos7-nginx   v0.1.20200114        7372d16c99bc        32 hours ago        448MB
jason/centos7-nginx   latest               c4e0980a825a        32 hours ago        448MB
mysql                 5.6.44               c30095c52827        6 months ago        256MB
centos                centos7.6.1810       f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker run -it --rm -p 80:80 --name myNginx nginx:v0.2-20200118-1750 　　　　　　　　　　　　#启动成功后会在当前终端阻塞。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200118182520017-2031240388.png)

**2>.使用exec命令连接新生成的容器并**查看容器内部的数据是否正确****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container exec -it myNginx bash
[root@2902fadc453a /]# 
[root@2902fadc453a /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 7  bytes 586 (586.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@2902fadc453a /]# 
[root@2902fadc453a /]# cat /etc/nginx/nginx.conf
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;

#Docker最终运行Nginx建议大家把后台进程关闭,默认是"on".
daemon off;

include /usr/share/nginx/modules/*.conf;

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
              
    access_log /var/log/nginx/access_json.log my_access_json;

    sendfile            on;
    keepalive_timeout   65;
    include       mime.types;
    default_type  text/html;
    charset utf-8;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;

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
[root@2902fadc453a /]# 
[root@2902fadc453a /]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:80                                                                                                                      *:*                  
LISTEN     0      128                                                                                                      [::]:80                                                                                                                   [::]:*                  
[root@2902fadc453a /]# 
[root@2902fadc453a /]# cd /var/log/nginx/
[root@2902fadc453a nginx]# 
[root@2902fadc453a nginx]# ll
total 0
-rw-r--r--. 1 root root 0 Jan 18 18:23 access_json.log
-rw-r--r--. 1 root root 0 Jan 18 18:23 error.log
[root@2902fadc453a nginx]# 
[root@2902fadc453a nginx]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>****.使用客户端访问宿主机的映射端口，验证咱们自己nginx的镜像**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200118183903738-735204375.png)**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186