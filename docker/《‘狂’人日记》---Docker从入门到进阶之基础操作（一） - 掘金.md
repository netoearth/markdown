   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月23日 20:24 ·  阅读 119

![《‘狂’人日记》---Docker从入门到进阶之基础操作（一）](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/877b1e20437b4afb99e6b41f958a403c~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第23天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 1.安装并启用

## 1.1 CentOS安装Docker

```
# 更新源
[root@base ~]# yum -y update
# 安装docker
[root@base ~]# yum -y install docker
# 查看docker版本
[root@base ~]# docker info
# 启动docker、设置开机自启
[root@base ~]# systemctl start docker
[root@base ~]# systemctl enable docker

复制代码
```

![docker info](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94ef7cf623534c16b4ea6961d4638d20~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![start_docker](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5eb063426b744a5292f2fa43b4e6a73d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.2 Ubuntu安装Docker

```
# 更新源
f@baseu:~$ sudo apt-get -y update
# 安装Docker
f@baseu:~$ sudo apt-get -y install docker.io
# 查看docker版本信息
f@baseu:~$ sudo docker info
# 启动和设置docker开机自启
f@baseu:~$ sudo systemctl start docker
f@baseu:~$ sudo systemctl enable docker
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/319784a5cb244c47846c93b91a39dc40~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3d5367ec88a4fc6be30d82062a65dec~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.3 Windows安装Docker

**[Docker 官网下载安装](https://link.juejin.cn/?target=https%3A%2F%2Fwww.docker.com%2Fget-started%2F "https://www.docker.com/get-started/")**

[下载Windows版本](https://link.juejin.cn/?target=https%3A%2F%2Fdesktop.docker.com%2Fwin%2Fmain%2Famd64%2FDocker%2520Desktop%2520Installer.exe%3Futm_source%3Ddocker%26utm_medium%3Dwebreferral%26utm_campaign%3Ddd-smartbutton%26utm_location%3Dmodule "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module")

## 2.常用命令

## 2.1 容器命令

### 2.1.1 ps命令

#### 2.1.1.1 使用语法

```
Usage:  docker ps [OPTIONS]

List containers

Options:
  -a, --all             显示所有容器（默认显示正在运行）
  -f, --filter filter   根据提供的条件过滤输出
      --format string   使用Go模板的漂亮打印容器
      --help            打印使用
  -n, --last int        显示最后创建的n个容器（包括所有状态）（默认值为-1）
  -l, --latest          显示最新创建的容器（包括所有状态）
      --no-trunc        不要截断输出
  -q, --quiet           仅显示数字ID
  -s, --size            显示总文件大小

复制代码
```

#### 2.1.1.2 -a 显示所有容器

```
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
609b039cc1e6        nginx               "/docker-entrypoin..."   4 seconds ago       Up 2 seconds               80/tcp              nginx
88958f0f4fc8        centos-7            "/bin/bash"              2 weeks ago         Exited (137) 2 weeks ago                       centos7
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ef281a53c6842a4b495d5c25a3857b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 2.1.1.2 -f,--filter 过滤

```
[root@base ~]# docker ps -af name='centos7'
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
88958f0f4fc8        centos-7            "/bin/bash"         2 weeks ago         Exited (137) 2 weeks ago                       centos7
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fdc34a8750c49d28c7740e7b8ffbda1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 2.1.1.3 -q 只显示id

```
[root@base ~]# docker ps -aq
609b039cc1e6
88958f0f4fc8
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5dd3a8327c96412c90d33ca6d5b15307~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 2.1.1.4 组合使用

```
# 查询所有容器中name为 centos7 的 并且只显示id
[root@base ~]# docker ps -aqf name='centos7'
88958f0f4fc8
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a05e8e2da764b02be512959567a79cd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 2.1.2 run 运行容器

#### 2.1.2.1 使用语法(常用参数语法)

```
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      
  -d, --detach                                在后台运行容器并打印container ID
  -i, --interactive                           保持STDIN打开，即使未连接
  -p, --publish list                          将容器的端口发布到主机（默认值[]）
  -P, --publish-all                           将所有公开端口发布到随机端口
      --restart string                        容器退出时应用的重新启动策略（默认为“否”）
      --name string                           为容器指定一个名称
  -t, --tty                                   分配一个伪TTY


复制代码
```

#### 2.1.2.2 -dit 后台运行，打开STDIN并分配一个伪TTY

```
[root@base ~]# docker run -dit nginx
056121918e5bdf3072cf9dc0e5533569d86e4a8d0c4faf0526be9a431711ba1c
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
056121918e5b        nginx               "/docker-entrypoin..."   20 seconds ago      Up 18 seconds              80/tcp              unruffled_chandrasekhar
609b039cc1e6        nginx               "/docker-entrypoin..."   21 hours ago        Exited (0) 21 hours ago                        nginx
88958f0f4fc8        centos-7            "/bin/bash"              2 weeks ago         Exited (137) 2 weeks ago                       centos7
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a67593306184647aeea8b9ccbe45739~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 2.1.2.3 --name 设定名字

```
[root@base ~]# docker run -dit --name nginx_named nginx
8925665a0958b70246993cecfa24e3580c114ecfd5fa2a15eb8cbc754d765b1a
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
8925665a0958        nginx               "/docker-entrypoin..."   2 seconds ago       Up 2 seconds               80/tcp              nginx_named
056121918e5b        nginx               "/docker-entrypoin..."   6 minutes ago       Up 6 minutes               80/tcp              unruffled_chandrasekhar
609b039cc1e6        nginx               "/docker-entrypoin..."   21 hours ago        Exited (0) 21 hours ago                        nginx
88958f0f4fc8        centos-7            "/bin/bash"              2 weeks ago         Exited (137) 2 weeks ago                       centos7

复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb450a754d544cd68bfea1d4554558df~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 2.1.2.4 -p 指定端口/ -P 随机端口

```
# -p指定端口
[root@base ~]# docker run -dit --name nginx -p 8080:80 nginx
c484ad61ed99710970889a54c94af47fdb6c50096331ac75e022e2b8383ec14a
[root@base ~]# curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


# -P 随机端口（32768起）
[root@base ~]# docker run -dit --name nginx_P -P nginx
3bbda870da081d5bd79e1bcc5c1ef6364382264f1f59d57ba45441af2f1c5161
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                     PORTS                   NAMES
3bbda870da08        nginx               "/docker-entrypoin..."   2 seconds ago        Up 2 seconds               0.0.0.0:32768->80/tcp   nginx_P
c484ad61ed99        nginx               "/docker-entrypoin..."   About a minute ago   Up About a minute          0.0.0.0:8080->80/tcp    nginx
88958f0f4fc8        centos-7            "/bin/bash"              2 weeks ago          Exited (137) 2 weeks ago                           centos7
[root@base ~]# curl localhost:32768
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f94767341b164cb6974ce7f955c23640~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d8470459b3d429ca963d26c989a5e3f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 2.1.3 rm删除容器

#### 2.1.3.1 使用语法

```
Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
      --help      Print usage
  -l, --link      Remove the specified link
  -v, --volumes   Remove the volumes associated with the container


复制代码
```

#### 2.1.3.2 删除容器

```
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                   NAMES
3bbda870da08        nginx               "/docker-entrypoin..."   21 hours ago        Up 2 seconds               0.0.0.0:32768->80/tcp   nginx_P
c484ad61ed99        nginx               "/docker-entrypoin..."   21 hours ago        Exited (0) 21 hours ago                            nginx
88958f0f4fc8        centos-7            "/bin/bash"              2 weeks ago         Exited (137) 2 weeks ago                           centos7
[root@base ~]# docker rm nginx
nginx
[root@base ~]# docker rm nginx_P
Error response from daemon: You cannot remove a running container 3bbda870da081d5bd79e1bcc5c1ef6364382264f1f59d57ba45441af2f1c5161. Stop the container before attempting removal or use -f


复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ece20fd6881442c88419c8a78931e4ee~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 2.1.3.3 -f 强制删除容器

```

[root@base ~]# docker rm -f nginx_P
nginx_P
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
88958f0f4fc8        centos-7            "/bin/bash"         2 weeks ago         Exited (137) 2 weeks ago                       centos7

复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1675a502f8340ec811738cb77981c44~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 3

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏