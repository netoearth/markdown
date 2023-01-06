   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月30日 08:52 ·  阅读 391

![《‘狂’人日记》---Docker从入门到进阶之进阶操作(五)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/105060c2057f4b00b740a11196f7fee3~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第30天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 上一章讲的是`数据管理`，那么这波玩个好玩的，实战演练【nginx负载均衡-轮询】

**在常规场景中，我们经常遇到的一个问题就是使用nginx进行反向代理和负载均衡，那么我们这波直接在docker上来进行操作，启动多个容器来模拟负载均衡场景,废话不多说，直接上手操作**

## 1.创建测试网页

```
# 创建web1文件夹 以及页面
mkdir /root/web1
echo "This is website 1 Page 11111111111111111" > /root/web1/index.html

# 创建web2文件夹 以及页面
mkdir /root/web2
echo "This is website 2 Page 22222222222222222" > /root/web2/index.html
复制代码
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db7b38944ecc45b5ae43b8a65d8ce33a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

## 2.创建测试容器 nginx1 nginx2

```
# 创建nginx1容器挂载 web1页面
docker run -dit --name nginx1 -v /root/web1/index.html:/usr/share/nginx/html/index.html -p 8091:80 nginx
# 创建nginx2容器挂载 web2页面
docker run -dit --name nginx2 -v /root/web2/index.html:/usr/share/nginx/html/index.html -p 8092:80 nginx
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75af5398e6d340e0838f753cac0f7e57~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

## 1.3 创建nginx.conf配置文件

配置文件中的 172.31.36.113 为宿主机的内网IP地址，请按实际情况修改

**配置文件中的两个server ip:port 分别代表当前正在运行的docker容器，待会会将流量分发到tomcat\_alb组中，组中的2个server分别处理流量**

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;




events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;


    include /etc/nginx/conf.d/*.conf;
# 定义Tomcat服务的负载均衡
upstream tomcat_alb {

    server 172.31.36.113:8091;

    server 172.31.36.113:8092;

}
server {
    listen       80;
    server_name  172.31.36.113;#宿主机的内网IP

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
# 这个是调用Tomcat服务的负载均衡
    location / {
        proxy_pass http://tomcat_alb;
    }
}
}
复制代码
```

## 1.4 创建nginx负载均衡容器

```
# -v 标记挂载本地的 /root/nginx.conf 文件
docker run -dit --name nginx-alb -v /root/nginx.conf:/etc/nginx/nginx.conf -p 80:80 nginx
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5426e1db960b4679ba0ceb770b10ab3e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 1.5 负载测试

这里使用一个shell的`for`循环语句来进行curl测试，看不同的请求得到什么样的返回值

```
# 循环测试
for i in `seq 1 6`;do curl 172.31.36.113;done
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64eda34869ca48818b2a964e92adec94~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 1.6 制作nginx负载均衡dockerfile

通过上面5个步骤的实验，可以很方面的得到实验结果，那么也可以采取打包容器的方式来制作一个nginx的负载均衡容器镜像

## 1.6.1 准备nginx配置文件

首先将1.3的nginx配置文件复制下来，命名为 nginx.conf

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;




events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;


    include /etc/nginx/conf.d/*.conf;
# 定义Tomcat服务的负载均衡
upstream tomcat_alb {

    server 172.31.36.113:8091;

    server 172.31.36.113:8092;

}
server {
    listen       80;
    server_name  172.31.36.113;#宿主机的内网IP

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
# 这个是调用Tomcat服务的负载均衡
    location / {
        proxy_pass http://tomcat_alb;
    }
}
}
复制代码
```

### 1.6.2 启动容器

```
docker run -dit --name nginx-alb -v /root/nginx.conf:/etc/nginx/nginx.conf -p 80:80 nginx
复制代码
```

### 1.6.3 导出容器

```
[root@base ~]# docker run -dit --name nginx-alb -v /root/nginx.conf:/etc/nginx/nginx.conf -p 80:80 nginx
6d1e6eff0388267cef089c7db7a6822b9c6ab8ceb4baab4d1338dc6bd3388e08
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                    NAMES
6d1e6eff0388        nginx               "/docker-entrypoin..."   4 seconds ago       Exited (1) 2 seconds ago                            nginx-alb
7dda8b117154        registry            "/entrypoint.sh /e..."   4 hours ago         Up 2 hours                 0.0.0.0:5000->5000/tcp   registry
53b462f31348        ssh-server          "/usr/sbin/sshd -D"      24 hours ago        Exited (0) 22 hours ago                             sshd
88958f0f4fc8        7e989e1e46e7        "/bin/bash"              2 weeks ago         Exited (137) 2 weeks ago                            centos7
[root@base ~]# docker export 6d1e6eff0388 > nginx_alb,tar
[root@base ~]# ls -l|grep nginx_alb
-rw-r--r--.  1 root root 144022528 Aug 25 21:44 nginx_alb,tar

复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbc1ce0998d0488da8b229d6f1d265f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**现在您已经成功打包了该容器**

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 2

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏