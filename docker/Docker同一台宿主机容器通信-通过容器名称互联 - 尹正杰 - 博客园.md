　　　　　　　　　　　**Docker同一台宿主机容器通信-通过容器名称互联**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.通过容器名称互联概述**

```
　　即在同一个宿主机上的容器之间可以通过自定义的容器名称相互访问,比如一个业务前端静态页面使用nginx,动态页面使用的是tomcat,由于容器在启动的时候其内部IP地址是DHCP随机分配的，所以如果通过内部访问的话,自定义名称是相对比较固定的，因此比较适合于此场景。
```

**二.通过容器名称互联实战案例**

**1>.创建一个tomcat容器(镜像制作可参考我之前的笔记:[https://www.cnblogs.com/yinzhengjie/p/12230043.html](https://www.cnblogs.com/yinzhengjie/p/12230043.html))**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy           v1.8.20             1858fe05d96f        8 days ago          606MB
registry                 latest              708bc6af7e5e        8 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        8 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        10 days ago         968MB
jdk-base                 1.8.0_231           0f63a97ddc85        10 days ago         953MB
centos-base              7.6.1810            b4931fd9ace2        10 days ago         551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it -d --name tomcat-app01 tomcat-app01:v0.1 
36315d1b0cb52beecd474121ae697c551ff7a51468f9600c228be18c02fc336d
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 36315d1b0cb52beecd474121ae697c551ff7a51468f9600c228be18c02fc336d bash
[root@36315d1b0cb5 /]# 
[root@36315d1b0cb5 /]# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    36315d1b0cb5
[root@36315d1b0cb5 /]# 
[root@36315d1b0cb5 /]# ping 36315d1b0cb5 -c 2
PING 36315d1b0cb5 (172.17.0.2) 56(84) bytes of data.
64 bytes from 36315d1b0cb5 (172.17.0.2): icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 36315d1b0cb5 (172.17.0.2): icmp_seq=2 ttl=64 time=0.032 ms

--- 36315d1b0cb5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.032/0.058/0.085/0.027 ms
[root@36315d1b0cb5 /]# 
[root@36315d1b0cb5 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
36315d1b0cb5        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   About a minute ago   Up About a minute   8080/tcp, 8443/tcp   tomcat-app01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.创建一个nginx容器(镜像制作可参考我之前的笔记:[https://www.cnblogs.com/yinzhengjie/p/12198878.html](https://www.cnblogs.com/yinzhengjie/p/12198878.html))时使用"--link"选项和上面创建的tomcat容器互联**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
nginx                    v0.1-20200201       1a8b4f68e96a        22 minutes ago      449MB
centos-haproxy           v1.8.20             1858fe05d96f        8 days ago          606MB
registry                 latest              708bc6af7e5e        8 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        8 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        10 days ago         968MB
jdk-base                 1.8.0_231           0f63a97ddc85        10 days ago         953MB
centos-base              7.6.1810            b4931fd9ace2        10 days ago         551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it -d --name nginx-web01 --link tomcat-app01 -p 80:80 nginx:v0.1-20200201 
8e05fc399edd16fe5bde32bebba269c0a07a0a1a928b46421693dd7ee4943ebf
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                         NAMES
8e05fc399edd        nginx:v0.1-20200201   "nginx"                  23 seconds ago      Up 22 seconds       0.0.0.0:80->80/tcp, 443/tcp   nginx-web01
36315d1b0cb5        tomcat-app01:v0.1     "/yinzhengjie/softwa…"   34 minutes ago      Up 34 minutes       8080/tcp, 8443/tcp            tomcat-app01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-web01 bash
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    tomcat-app01 36315d1b0cb5
172.17.0.3    8e05fc399edd
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# ping tomcat-app01 -c 1
PING tomcat-app01 (172.17.0.2) 56(84) bytes of data.
64 bytes from tomcat-app01 (172.17.0.2): icmp_seq=1 ttl=64 time=0.051 ms

--- tomcat-app01 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.051/0.051/0.051/0.000 ms
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.进入nginx容器编辑nginx的配置文件反向代理tomcat服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS              PORTS                         NAMES
8e05fc399edd        nginx:v0.1-20200201   "nginx"                  About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, 443/tcp   nginx-web01
36315d1b0cb5        tomcat-app01:v0.1     "/yinzhengjie/softwa…"   35 minutes ago       Up 35 minutes       8080/tcp, 8443/tcp            tomcat-app01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-web01 bash
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# vim /etc/nginx/nginx.conf
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# cat /etc/nginx/nginx.conf
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;

daemon off;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}
   
http {
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

    upstream tomcat {
　　　　server 172.17.0.2:8080;
    }

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;

        location / {   
    }

    location /app01 {
        proxy_pass http://tomcat;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;    
    }

        error_page 404 /404.html;             
        location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@8e05fc399edd /]# 
[root@8e05fc399edd /]# nginx -s reload
[root@8e05fc399edd /]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.浏览器访问Nginx的web页面(http://docker101.yinzhengjie.org.cn/)**

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201082325494-21122676.png)

**5>.浏览器访问Tomcat的web页面(http://docker101.yinzhengjie.org.cn/app01/)**

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201082422814-638059179.png)**

**三.通过容器名称互联的注意事项**

**1>.思考第二步存在的问题**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　先不要往下看，请闭眼10秒钟思考上面的配置可能会存在哪些故障问题,经过激烈的思考后再往下看咱们是否想到一起去了?　　问题一:　　　　nginx的配置文件中在upstream中写死了后端容器的IP地址.　　　　问题剖析:　　　　　　这样有一个致命的缺陷就是可移植性差,我们将该容器提交为镜像后再其它节点上运行很可能分配的容器IP地址不同会导致后端服务无法正常访问.　　　　解决方案:　　　　　　在配置文件中写后端服务器的主机名称,这样不管IP地址怎么变动都不用修改nginx的配置文件，只需要修改"/etc/hosts"文件将后端服务器主机的IP地址改成对应的IP地址即可(在新启动的容器这个操作是自动完成的).　　问题二:　　　　代码升级时可能后端容器的名称发生变动.　　　　问题刨析:　　　　　　这是一个让人和头疼的问题，尽管我们上面解决了写死IP地址的问题，但是后端的主机名一直频繁的改动那意味这我们依旧得重新nginx的配置文件.　　　　解决方案:　　　　　　在使用"--link"通过容器名称互联时我们可以自定义容器别名,使用容器别名在"/etc/hosts"文件中解析的后端服务器主机名是咱们自定义的后端主机名.
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.通过自定义容器别名互联案例**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it -d --name nginx-web02 --link tomcat-app01:www.yinzhengjie.org.cn -p 81:80 nginx:v0.1-20200201 
d5d2b409cc0c20486a1d454d6b26fdc398dae93df9c128577c95fdbbc617d868
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                         NAMES
d5d2b409cc0c        nginx:v0.1-20200201   "nginx"                  8 seconds ago       Up 7 seconds        443/tcp, 0.0.0.0:81->80/tcp   nginx-web02
8e05fc399edd        nginx:v0.1-20200201   "nginx"                  51 minutes ago      Up 51 minutes       0.0.0.0:80->80/tcp, 443/tcp   nginx-web01
36315d1b0cb5        tomcat-app01:v0.1     "/yinzhengjie/softwa…"   About an hour ago   Up About an hour    8080/tcp, 8443/tcp            tomcat-app01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-web02 bash
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    www.yinzhengjie.org.cn 36315d1b0cb5 tomcat-app01
172.17.0.4    d5d2b409cc0c
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# ping www.yinzhengjie.org.cn -c 3
PING www.yinzhengjie.org.cn (172.17.0.2) 56(84) bytes of data.
64 bytes from www.yinzhengjie.org.cn (172.17.0.2): icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from www.yinzhengjie.org.cn (172.17.0.2): icmp_seq=2 ttl=64 time=0.100 ms
64 bytes from www.yinzhengjie.org.cn (172.17.0.2): icmp_seq=3 ttl=64 time=0.040 ms

--- www.yinzhengjie.org.cn ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.040/0.066/0.100/0.025 ms
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.修改nginx的配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-web02 bash
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# vim /etc/nginx/nginx.conf
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# cat /etc/nginx/nginx.conf
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;

daemon off;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}
   
http {
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

    upstream tomcat {
　　　　server www.yinzhengjie.org.cn:8080;
    }

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;

        location / {   
    }

    location /app01 {
        proxy_pass http://tomcat;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;    
    }

        error_page 404 /404.html;             
        location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
[root@d5d2b409cc0c /]# 
[root@d5d2b409cc0c /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186