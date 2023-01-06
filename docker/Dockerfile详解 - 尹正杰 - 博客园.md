　　　　　　　　　　　　　　　**Dockerfile详解**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Dockerfile概述**

**1>.什么是dockerfile**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191020060253711-916713316.png)

```
如上图所示，DockerFile只是构建Docker Imges的源代码(只是一些文本指令不涉及编程语言):　　1>.Docker可以通过读取DockerFile中的指令自动生成Images;　　2>.Dockerfile是一个文本文档，包含用户在命令行上可以调用的所有命令组装image;　　3>.使用Docker Build用户可以创建一个自动生成，它连续执行多个命令行指令。
```

**2>.docker 语法格式(format)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
Docker format由两部分组成：
    以"#"开头的注释信息(comment);
    指令及其参数(INSTRUCTION arguments);指令不区分大小写(The instruction is not case-sensitive)　　然而，惯例是它们是大写的，以便更容易地将它们与参数(arguments)区分开来Docker按顺序在DockerFile中运行指令(Docker runs instructions in a Dockerfile in order)第一条非注释行指令必须是“FROM”，以便指定要从中生成的基础图像
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.dockerfile 黑名单(".dockerignore file"）** 

```
　　在docker client将上下文发送给docker守护进程之前，它会在上下文的根目录中查找一个名为.dockerginore的文件;　　如果此文件存在，docker client用它修改上下文以排除匹配模式的文件和目录;　　docker client将.dockerignore文件解释为换行分隔的模式列表，类似于unix shell的文件globs;
```

**4>.环境变量(Environment replacement)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　环境变量（用env语句声明）也可以在某些指令中用作dockerfile要解释的变量。

　　环境变量在dockerfile中用$variable_name或${variable_name}表示。

　　${variable_name}语法还支持一些标准bash修饰符:
　　　　${variable:-word}表示如果设置了variable，则结果将是该值。如果未设置variable，则结果将是word；
　　　　${variable:+word}表示如果设置了variable，则返回word，否则返回空字符串。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.docker 指令(Instructions)**

**1>.FROM**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　FROM指令是最重的一个且必须为Dockerfile文件开篇的第一个非注释行，用于为映像文件构建过程指定基准镜像，后续的指令运行于此基准镜像所提供的运行环境;　　实践中，基准镜像可以是任何可用镜像文件，默认情况下，docker build会在docker主机上找指定的镜像文件，在其不存在时，则会从Docker Hub Registry上拉取所需的镜像文件　　　　如果找不到指定的镜像文件，docker build会返回一个错误信息　　语法(syntax)格式如下:　　　　FROM <repository>[:<tag>] 或者 FROM <repositroy>@<digest>　　　　　　<repositroy>: 指定作为base image的名称;　　　　　　<tag>　　　　 : base image的标签，为可选项，省略时默认为latest;　　　　　　<digest>　　 : 指定镜像的hash值;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.MAINTANIER(该指令已经被废弃(depreacted)，不推荐使用，其功能可被LABLE指令替代)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　用于让Dockefile制作者提供本人的详细信息
　　Docker并不限制MAINTAINER指令可在出现的位置，但推荐将其放置于FROM指令之后
　　语法格式如下:　　　　MAINTAINER <authtor's detail>　　　　　　<author's detail>可是任何文本信息，但约定俗成地使用作者名称及邮件地址　　　　　　MAINTAINER "yinzzhengjie <y1053419035@qq.com>"
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.LABLE**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　LABLE指令将元数据添加到image中;　　LABLE是键值对(key-value);　　要在标签值中包含空格，请使用引号和反斜杠，就像在命令行解析中一样;　　一个image可以有多个LABEL指令;　　　　可以在一行指定多个标签;　　语法格式如下:　　　　LABLE <key>=<value> <key>=<value> <key>=<value>
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.COPY**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　用于从Docker主机复制文件至创建地新映像文件
　　语法格式:　　　　COPY <src> ... <dest> 或者 COPY ["<src>",..."<dest>"]　　　　　　<src>  : 要复制的源文件或目录，支持使用通配符。　　　　　　<dest> : 目标路径，即正在创建的image的文件系统路径;建议为<dest>使用绝对路径，否则，COPY指定则以WORDKDIR为起始路径。　　　　注意:正在路径中有空白符时，通常使用第二种格式。
　　文件复制准则:　　　　1>.<src>必须是build上下文中的路径，不能是其父目录中的文件；　　　　2>.如果<src>是目录，则其内部文件或子目录会被递归复制，但<src>目录自身不会被复制；　　　　3>.如果指定了多个<src>，或在<src>中使用了通配符，则<dest>必须是一个目录，且必须以"/"结尾；　　　　4>.如果<dest>实现不存在，它将会被自动创建，这包括其父目录路径。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# mkdir image01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vim Dockerfile
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　#编辑Dockerfile配置文件
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzzhengjie <y1053419035@qq.com>"
LABEL maintainer="yinzzhengjie <y1053419035@qq.com>"
COPY index.html /data/web/html/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat index.html              #千万别忘记该文件要和Dockerfile在同一个目录中哟
<h1>Busybox httpd server.</h1>
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# ll
total 8
-rw-r--r-- 1 root root 423 Oct 20 08:26 Dockerfile
-rw-r--r-- 1 root root  31 Oct 20 08:24 index.html
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　　　　　#编辑Dockerfile配置文件（复制文件案例）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-1 ./　　　　#使用docker build命令制作镜像，制作成功后可用"docker images"相关命令查看已有镜像。
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/4 : MAINTAINER "yinzzhengjie <y1053419035@qq.com>"
 ---> Running in fb5de66d8e30
Removing intermediate container fb5de66d8e30
 ---> ff47a41734e2
Step 3/4 : LABEL maintainer="yinzzhengjie <y1053419035@qq.com>"
 ---> Running in b2aa71dba87f
Removing intermediate container b2aa71dba87f
 ---> 7e8b7eaae37a
Step 4/4 : COPY index.html /data/web/html/
 ---> cd66108f3562
Successfully built cd66108f3562
Successfully tagged tinyhttpd:v0.1-1
[root@node101.yinzhengjie.org.cn ~/image01]#
[root@node101.yinzhengjie.org.cn ~/image01]# docker images　　　　　　　　　　#我们发现上面命令执行成功后，我们可以看到有对应的镜像名称。
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-1              794f641ae85e        3 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-1 ./　　　　#使用docker build命令制作镜像，制作成功后可用"docker images"相关命令查看已有镜像。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-1              794f641ae85e        7 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-1 cat /data/web/html/index.html　　　#验证时无需进入容器，在启动时查看容器中"/data/web/html/index.html"文件是否存在即可。
<h1>Busybox httpd server.</h1>
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vi Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　#编辑Dockerfile配置文件（复制文件和目录案例）
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件
COPY index.html /data/web/html/
#复制目录
COPY yum.repos.d /etc/yum.repos.d/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cp -r /etc/yum.repos.d/ ./
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# ll
total 8
-rw-r--r-- 1 root root 484 Oct 20 08:50 Dockerfile
-rw-r--r-- 1 root root  31 Oct 20 08:24 index.html
drwxr-xr-x 2 root root 209 Oct 20 08:51 yum.repos.d
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　#编辑Dockerfile配置文件（复制文件和目录案例）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-1              794f641ae85e        17 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-2 ./　　　　　　#制作镜像
Sending build context to Docker daemon  24.58kB
Step 1/4 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/4 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Running in c515e38ceda5
Removing intermediate container c515e38ceda5
 ---> 84ed65a65c8b
Step 3/4 : COPY index.html /data/web/html/
 ---> e8bbb2978d9a
Step 4/4 : COPY yum.repos.d /etc/yum.repos.d/
 ---> de390df4c4c3
Successfully built de390df4c4c3
Successfully tagged tinyhttpd:v0.1-2
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-2              de390df4c4c3        5 seconds ago       1.23MB
tinyhttpd           v0.1-1              794f641ae85e        17 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-2 ./　　　　　　#制作镜像

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-2              de390df4c4c3        About a minute ago   1.23MB
tinyhttpd           v0.1-1              794f641ae85e        19 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-2 ls /etc/yum.repos.d/　　　　　　#同理，在启动容器时查看指定的路径容器是否存在咱们自定义拷贝的文件即可。
CentOS-Base.repo
CentOS-CR.repo
CentOS-Debuginfo.repo
CentOS-Media.repo
CentOS-Sources.repo
CentOS-Vault.repo
CentOS-fasttrack.repo
docker-ce.repo
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.ADD**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　ADD指令类似于COPY指令，ADD支持使用TAR文件和URL路径;　　语法格式如下:　　　　ADD <src> ... <dest> 或者 ADD ["<src>",..."<dest>"]　　操作准则:　　　　1>.同COPY指令;　　　　2>.如果<src>为URL且<dest>不以"/"结尾，则<src>指定的文件将被下载并直接创建为<dest>;如果<dest>以"/"结尾，则文件名URL指定的文件将被直接下载并保存为<dest>/<filename>;　　　　3>.如果<src>是一个本地系统上的压缩格式的tar文件，它将被展开为一个目录，其行为类似于"tar -x"命令；然而,通过URL获取到的tar文件将不会自动展开;　　　　4>.如果<src>有多个，或其间接或直接使用了通配符，则<dest>必须是一个以"/"结尾的目录路径;如果<dest>不以"/"结尾，则其被视作一个普通文件，<src>的内容将被直接写入到<dest>;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vi Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　　　#编辑Dockerfile配置文件(将网络上的nginx安装包添加到镜像中)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件到镜像的"/data/web/html/"目录中
COPY index.html /data/web/html/
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# ll
total 8
-rw-r--r-- 1 root root 731 Oct 20 09:10 Dockerfile
-rw-r--r-- 1 root root  31 Oct 20 08:24 index.html
drwxr-xr-x 2 root root 209 Oct 20 08:51 yum.repos.d
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　　　#编辑Dockerfile配置文件(将网络上的nginx安装包添加到镜像中)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-2              de390df4c4c3        15 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        33 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-3 ./　　　　　　#等待打包成功，由于需要去网络下载软件，因此可能需要等待一段时间
Sending build context to Docker daemon  25.09kB
Step 1/5 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/5 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/5 : COPY index.html /data/web/html/
 ---> Using cache
 ---> e8bbb2978d9a
Step 4/5 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> de390df4c4c3
Step 5/5 : ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
Downloading [==================================================>]  1.035MB/1.035MB
 ---> aa780396b2a3
Successfully built aa780396b2a3
Successfully tagged tinyhttpd:v0.1-3
[root@node101.yinzhengjie.org.cn ~/image01]#  
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-3              aa780396b2a3        About a minute ago   2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        17 minutes ago       1.23MB
tinyhttpd           v0.1-1              794f641ae85e        34 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-3 ./　　　　　　#等待打包成功，由于需要去网络下载软件，因此可能需要等待一段时间

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-3              aa780396b2a3        About a minute ago   2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        17 minutes ago       1.23MB
tinyhttpd           v0.1-1              794f641ae85e        34 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-3 ls /usr/local/soft　　　　#我们发现该容器的确有咱们的nginx安装包
nginx-1.17.4.tar.gz
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　#编辑Dockerfile配置文件(将tar包添加到镜像指定目录中)　
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件到镜像的"/data/web/html/"目录中
COPY index.html /data/web/html/
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
#ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
#由于上面的方式可能打包镜像较慢，因此我们可以下载到本地，然后将tar包解压到指定的"/usr/local/nginx/"目录中
ADD nginx-1.17.4.tar.gz /usr/local/nginx/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# wget http://nginx.org/download/nginx-1.17.4.tar.gz
--2019-10-20 09:19:10--  http://nginx.org/download/nginx-1.17.4.tar.gz
Resolving nginx.org (nginx.org)... 95.211.80.227, 62.210.92.35, 2001:1af8:4060:a004:21::e3
Connecting to nginx.org (nginx.org)|95.211.80.227|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://103.78.124.80:81/2Q2W429F9BA45D0BBC1E21179428883AFD1B8E41A0D0_unknown_81036E318E74AD2688DD5563203057F97EAE0951_12/ngin
x.org/download/nginx-1.17.4.tar.gz [following]--2019-10-20 09:19:10--  http://103.78.124.80:81/2Q2W429F9BA45D0BBC1E21179428883AFD1B8E41A0D0_unknown_81036E318E74AD2688DD5563203057F97
EAE0951_12/nginx.org/download/nginx-1.17.4.tar.gzConnecting to 103.78.124.80:81... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1034845 (1011K) [application/octet-stream]
Saving to: ‘nginx-1.17.4.tar.gz’

100%[=============================================================================================>] 1,034,845   2.35MB/s   in 0.4s   

2019-10-20 09:19:10 (2.35 MB/s) - ‘nginx-1.17.4.tar.gz’ saved [1034845/1034845]

[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# ll
total 1020
-rw-r--r-- 1 root root     921 Oct 20 09:18 Dockerfile
-rw-r--r-- 1 root root      31 Oct 20 08:24 index.html
-rw-r--r-- 1 root root 1034845 Sep 24 23:13 nginx-1.17.4.tar.gz
drwxr-xr-x 2 root root     209 Oct 20 08:51 yum.repos.d
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　#编辑Dockerfile配置文件(将tar包添加到镜像指定目录中)　

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-3              aa780396b2a3        7 minutes ago       2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        23 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        41 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-4 ./　　　　　　　　　　#编译镜像
Sending build context to Docker daemon  1.061MB
Step 1/5 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/5 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/5 : COPY index.html /data/web/html/
 ---> Using cache
 ---> e8bbb2978d9a
Step 4/5 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> de390df4c4c3
Step 5/5 : ADD nginx-1.17.4.tar.gz /usr/local/nginx/
 ---> 5086cc9cdc4a
Successfully built 5086cc9cdc4a
Successfully tagged tinyhttpd:v0.1-4
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-4              5086cc9cdc4a        4 seconds ago       7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        8 minutes ago       2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        23 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        41 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-4 ./　　　　　　　　　　#编译镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-4              5086cc9cdc4a        About a minute ago   7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        9 minutes ago        2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        25 minutes ago       1.23MB
tinyhttpd           v0.1-1              794f641ae85e        42 minutes ago       1.22MB
redis               latest              de25a81a5a0b        2 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-4 ls /usr/local/nginx　　　　　　　　#查看容器内是否有解压的nginx目录
nginx-1.17.4
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-4 ls /usr/local/nginx/nginx-1.17.4
CHANGES
CHANGES.ru
LICENSE
README
auto
conf
configure
contrib
html
man
src
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-4 ls /usr/local/nginx　　　　　　　　#查看容器内是否有解压的nginx目录

**6>.WORKDIR**

```
　　用于为Docker中所有的RUN，CMD，ENTRYPOINT，COPY和ADD指定设定工作目录。　　语法格式如下:　　　　WORKDIR <dirpath>　　　　　　在Dockerfile文件中，WORKDIR指令可出现多次，其路径也可以分为相对路径，不过，其是相对此前一个WORKDIR指令指定的路径;　　　　　　另外，WORKDIR也可调用由ENV指定定义的变量。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vim Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　#编辑Dockerfile配置文件(指定工作目录案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件到镜像的"/data/web/html/"目录中
COPY index.html /data/web/html/
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
#ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
#指定工作目录,此处我们指定了工作目录，下面直接就可以使用相对路径指定解压目录位置啦！
WORKDIR /usr/local/
#由于上面的方式可能打包镜像较慢，因此我们可以下载到本地，然后将tar包解压到指定的"/usr/local/nginx/"目录中
ADD nginx-1.17.4.tar.gz ./nginx
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-4              5086cc9cdc4a        9 minutes ago       7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        17 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        32 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        50 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-5 ./
Sending build context to Docker daemon  1.061MB
Step 1/6 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/6 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/6 : COPY index.html /data/web/html/
 ---> Using cache
 ---> e8bbb2978d9a
Step 4/6 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> de390df4c4c3
Step 5/6 : WORKDIR /usr/local/
 ---> Running in 1093de6bd9e7
Removing intermediate container 1093de6bd9e7
 ---> e22b16220d31
Step 6/6 : ADD nginx-1.17.4.tar.gz ./nginx
 ---> f6f58ddb1520
Successfully built f6f58ddb1520
Successfully tagged tinyhttpd:v0.1-5
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-5 ls /usr/local/nginx
nginx-1.17.4
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-5 ls /usr/local/nginx/nginx-1.17.4
CHANGES
CHANGES.ru
LICENSE
README
auto
conf
configure
contrib
html
man
src
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　#编辑Dockerfile配置文件(指定工作目录案例)

**7>.VOLUME**

```
　　用于在image中创建一个挂载点目录，以挂载Docker host上的卷或其它容器上的卷。

　　语法格式如下:　　　　VOLUME <mountpoint> 或者 VOLUME ["<mountpoint>"]　　如果挂载点目录下此前有文件存在，docker run命令会卷挂载完成后将此前的所有文件复制到新挂载的卷中。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vi Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　#编辑Dockerfile配置文件(指定挂在卷案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件到镜像的"/data/web/html/"目录中
COPY index.html /data/web/html/
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
#ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
#指定工作目录,此处我们指定了工作目录，下面直接就可以使用相对路径指定解压目录位置啦！
WORKDIR /usr/local/
#由于上面的方式可能打包镜像较慢，因此我们可以下载到本地，然后将tar包解压到指定的"/usr/local/nginx/"目录中
ADD nginx-1.17.4.tar.gz ./nginx
#指定挂在卷
VOLUME /data/mysql/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　#编辑Dockerfile配置文件(指定挂在卷案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-5              f6f58ddb1520        8 minutes ago       7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        17 minutes ago      7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        25 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        41 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        58 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-6 ./　　　　　　#编译镜像
Sending build context to Docker daemon  1.061MB
Step 1/7 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/7 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/7 : COPY index.html /data/web/html/
 ---> Using cache
 ---> e8bbb2978d9a
Step 4/7 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> de390df4c4c3
Step 5/7 : WORKDIR /usr/local/
 ---> Using cache
 ---> e22b16220d31
Step 6/7 : ADD nginx-1.17.4.tar.gz ./nginx
 ---> Using cache
 ---> f6f58ddb1520
Step 7/7 : VOLUME /data/mysql/
 ---> Running in aa3cf8db646b
Removing intermediate container aa3cf8db646b
 ---> 2368b7546664
Successfully built 2368b7546664
Successfully tagged tinyhttpd:v0.1-6
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-6              2368b7546664        3 seconds ago       7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        8 minutes ago       7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        17 minutes ago      7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        25 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        41 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        58 minutes ago      1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-6 ./　　　　　　#编译镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-6              2368b7546664        About a minute ago   7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        10 minutes ago       7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        19 minutes ago       7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        27 minutes ago       2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        43 minutes ago       1.23MB
tinyhttpd           v0.1-1              794f641ae85e        About an hour ago    1.22MB
redis               latest              de25a81a5a0b        2 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-6 mount | grep mysql　　　　　　#验证后发现容器的确挂载了mysql目录
/dev/mapper/centos-root on /data/mysql type xfs (rw,relatime,attr2,inode64,noquota)
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-6 sleep 60                                 #我们让该容器存活60秒时间，在这个时间段咱们可以使用"docker inspect"相关命令快速查看容器相关信息


另外启动一个连接宿主机的终端，可执行命令查看对应的存储卷挂载信息：
[root@node101.yinzhengjie.org.cn ~/image01]# docker inspect -f {{.Mounts}} tinyweb1 
[{volume 942ca3b9d448b1ac52e3b3fc0c3c591d4f86e5514b842fb1543cb50704def0f1 /var/lib/docker/volumes/942ca3b9d448b1ac52e3b3fc0c3c591d4f86e
5514b842fb1543cb50704def0f1/_data /data/mysql local  true }][root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-6 mount | grep mysql　　　　　　#验证后发现容器的确挂载了mysql目录

**8>.EXPOSE**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　用于为容器打开指定要监听的端口以实现与外部通信;　　该配置需要在容器启动时使用"-P"选项才能生效，因此我们可以说它是待暴露端口;　　由于docker容器不确定你将来在哪台主机上运行容器，因此无法指定具体IP，而端口那些已经被占用了还得运维和开发人员来判断，不能为其分配已经占用的端口;　　语法格式:　　　　EXPOSE <port>[/<protocol>][<port>[/<protocol>] ...]　　　　EXPOSE指令可一次性指定多个端口，例如:EXPOSE 11211/udp 11211/tcp
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vi Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　　　#编辑Dockerfile配置文件(暴露指定端口案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#复制文件到镜像的"/data/web/html/"目录中
COPY index.html /data/web/html/
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
#ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
#指定工作目录,此处我们指定了工作目录，下面直接就可以使用相对路径指定解压目录位置啦！
WORKDIR /usr/local/
#由于上面的方式可能打包镜像较慢，因此我们可以下载到本地，然后将tar包解压到指定的"/usr/local/nginx/"目录中
ADD nginx-1.17.4.tar.gz ./nginx
#指定挂在卷
VOLUME /data/mysql/
#此处咱们暴露80端口
EXPOSE 80/udp 80/tcp
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# ll
total 1020
-rw-r--r-- 1 root root    1143 Oct 20 09:54 Dockerfile
-rw-r--r-- 1 root root      31 Oct 20 08:24 index.html
-rw-r--r-- 1 root root 1034845 Sep 24 23:13 nginx-1.17.4.tar.gz
drwxr-xr-x 2 root root     209 Oct 20 08:51 yum.repos.d
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　　　#编辑Dockerfile配置文件(暴露指定端口案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-6              2368b7546664        17 minutes ago      7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        25 minutes ago      7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        35 minutes ago      7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        43 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        58 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        About an hour ago   1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-7 ./　　　　　　#编译镜像
Sending build context to Docker daemon  1.061MB
Step 1/8 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/8 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/8 : COPY index.html /data/web/html/
 ---> Using cache
 ---> e8bbb2978d9a
Step 4/8 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> de390df4c4c3
Step 5/8 : WORKDIR /usr/local/
 ---> Using cache
 ---> e22b16220d31
Step 6/8 : ADD nginx-1.17.4.tar.gz ./nginx
 ---> Using cache
 ---> f6f58ddb1520
Step 7/8 : VOLUME /data/mysql/
 ---> Using cache
 ---> 2368b7546664
Step 8/8 : EXPOSE 80/udp 80/tcp
 ---> Running in 205f8a64ea3e
Removing intermediate container 205f8a64ea3e
 ---> 9a5668f5f3a3
Successfully built 9a5668f5f3a3
Successfully tagged tinyhttpd:v0.1-7
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-7              9a5668f5f3a3        5 seconds ago       7.45MB
tinyhttpd           v0.1-6              2368b7546664        17 minutes ago      7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        26 minutes ago      7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        35 minutes ago      7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        43 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        59 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        About an hour ago   1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-7 ./　　　　　　#编译镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-7              9a5668f5f3a3        45 seconds ago      7.45MB
tinyhttpd           v0.1-6              2368b7546664        18 minutes ago      7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        26 minutes ago      7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        35 minutes ago      7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        43 minutes ago      2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        59 minutes ago      1.23MB
tinyhttpd           v0.1-1              794f641ae85e        About an hour ago   1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-7 /bin/httpd -h /data/web/html -f　　　　　　#启动容器时将http服务启动并放在前台运行，这会导致当前在终端处于阻塞柱状态。
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-7 /bin/httpd -h /data/web/html -f　　　　　　#启动容器时将http服务启动并放在前台运行，这会导致当前在终端处于阻塞柱状态。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
f8c23dd18de2        tinyhttpd:v0.1-7    "/bin/httpd -h /data…"   3 minutes ago       Up 3 minutes        80/tcp, 80/udp      tinyweb1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect tinyweb1 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "192.168.100.1",
                    "IPAddress": "192.168.100.1",
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 192.168.100.1:80 　　　　　　　　#发现"tinyweb1"容器的web服务时可以被正常访问的
<h1>Busybox httpd server.</h1>
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker port tinyweb1　　　　　　　　　　#虽然上面的服务可以正常访问，但并没有真正暴露端口，若想要真的暴露端口在启动容器时需要加"-P"
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker kill tinyweb1　　　　　　　　　　#杀死容器便于咱们重新启动容器
tinyweb1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name tinyweb1 --rm -P tinyhttpd:v0.1-7 /bin/httpd -h /data/web/html -f　　　　　　#注意添加"-P"选项不用指定具体端口，因为进行中有带暴露端口哟~启动时依旧处于阻塞状态，需要另外启动一个终端查看端口是否暴露

以下代码为新启动终端的验证代码指令:
[root@node101.yinzhengjie.org.cn ~/image01]# docker container ls
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS 
NAMES46bb8617d884 tinyhttpd:v0.1-7 "/bin/httpd -h /data…" 58 seconds ago Up 56 seconds 0.0.0.0:32768->80/tcp, 0.0.0.
0:32768->80/udp tinyweb1
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker port tinyweb1　　　　　　#此时发现端口暴露成功啦，咱们可以使用物理机访问，效果如下图所示。
80/udp -> 0.0.0.0:32768
80/tcp -> 0.0.0.0:32768
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]#
[root@node101.yinzhengjie.org.cn ~/image01]# hostname -i　　　　　　　　　　　　#查看本机的IP地址信息
172.30.1.101
[root@node101.yinzhengjie.org.cn ~/image01]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191020100902957-560776934.png)

**9>.ENV**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
用于为镜像定义所需的环境变量，并可被Dockerfile文件中位于其后的其它指令(如ENV,ADD,COPY等)所调用。

调用格式为$variable_name或${variable_name}语法格式:　　ENV <key> <value> 或 ENV <key>=<value> ...操作准则:　　　　1>.第一种格式中，<key>之后的所有内容均会被视作其<value>的组成部分，因此，一次只能设置一个变量;　　　　2>.第二种格式可用一次设置多个变量，每个变量为一个"<key>=<value>"的键值对，如果<value>中包含空格，可以以反斜线(\)进行转义，也可通过对<value>加引号进行标识；另外，反斜线也可用于续行；　　　　3>.定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vim Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　　　　　#编辑Dockerfile配置文件(添加环境变量案例案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
#添加作者信息，不推荐使用
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#建议使用LABEL,它可以代替MAINTAINER的功能
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#我们此处可以定义多个变量,其中"\"表示续行符
ENV DOC_ROOT=/data/web/html/ \
    WEB_SERVER_PACKAGE="nginx-1.17.4"
#使用上面定义的环境变量DOC_ROOT，如果该变量不存在或没有赋值时，则使用咱们自定义的值"/data/web/html/",这样写保险点
COPY index.html ${DOC_ROOT:-/data/web/html/}
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
#ADD http://nginx.org/download/nginx-1.17.4.tar.gz /usr/local/soft/
#指定工作目录,此处我们指定了工作目录，下面直接就可以使用相对路径指定解压目录位置啦！
WORKDIR /usr/local/
#调用上面定义的变量
ADD ${WEB_SERVER_PACKAGE}.tar.gz ./nginx
#指定挂在卷
VOLUME /data/mysql/
#此处咱们暴露80端口
EXPOSE 80/udp 80/tcp
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-7              9a5668f5f3a3        31 minutes ago      7.45MB
tinyhttpd           v0.1-6              2368b7546664        48 minutes ago      7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        57 minutes ago      7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        About an hour ago   7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        About an hour ago   2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        2 hours ago         1.23MB
tinyhttpd           v0.1-1              794f641ae85e        2 hours ago         1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-8 ./
Sending build context to Docker daemon  1.061MB
Step 1/9 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/9 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/9 : ENV DOC_ROOT=/data/web/html/     WEB_SERVER_PACKAGE="nginx-1.17.4"
 ---> Running in b7c50ec7bc67
Removing intermediate container b7c50ec7bc67
 ---> 8767193bffa3
Step 4/9 : COPY index.html ${DOC_ROOT:-/data/web/html/}
 ---> 6284c780a63e
Step 5/9 : COPY yum.repos.d /etc/yum.repos.d/
 ---> 84a1a8306dec
Step 6/9 : WORKDIR /usr/local/
 ---> Running in f48c8a59dafb
Removing intermediate container f48c8a59dafb
 ---> 738275e6f2f9
Step 7/9 : ADD ${WEB_SERVER_PACKAGE}.tar.gz ./nginx
 ---> 977975f635e4
Step 8/9 : VOLUME /data/mysql/
 ---> Running in f107b57cf296
Removing intermediate container f107b57cf296
 ---> d941c7d08cd0
Step 9/9 : EXPOSE 80/udp 80/tcp
 ---> Running in 00cbe64d58cd
Removing intermediate container 00cbe64d58cd
 ---> 42c71beb05d5
Successfully built 42c71beb05d5
Successfully tagged tinyhttpd:v0.1-8
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-8              42c71beb05d5        3 seconds ago       7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        31 minutes ago      7.45MB
tinyhttpd           v0.1-6              2368b7546664        49 minutes ago      7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        57 minutes ago      7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        About an hour ago   7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        About an hour ago   2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        2 hours ago         1.23MB
tinyhttpd           v0.1-1              794f641ae85e        2 hours ago         1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-8 ls /usr/local/nginx
nginx-1.17.4
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-8 ls /usr/local/nginx/nginx-1.17.4
CHANGES
CHANGES.ru
LICENSE
README
auto
conf
configure
contrib
html
man
src
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加环境变量案例案例)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 --rm tinyhttpd:v0.1-8 printenv　　　　#我们发现容器中的变量的确是咱们Dockerfile中定义的变量。
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=747c79b4907d
DOC_ROOT=/data/web/html/
WEB_SERVER_PACKAGE=nginx-1.17.4
HOME=/root
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 -e WEB_SERVER_PACKAGE=nginx-1.15.1 --rm tinyhttpd:v0.1-8 printenv　　　　　　　　#注意，我们可以在容器运行时修改环境变量所对应的值PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=12766146359e
WEB_SERVER_PACKAGE=nginx-1.15.1
DOC_ROOT=/data/web/html/
HOME=/root
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 -e WEB_SERVER_PACKAGE=nginx-1.15.1 --rm tinyhttpd:v0.1-8 ls /usr/local/nginx　　　　#不难发现，尽管我们在容器运行时修改了环境变量的值，但不会应影响编译时的变量，因为编译成镜像过程在前，而运行容器在后。nginx-1.17.4
[root@node101.yinzhengjie.org.cn ~/image01]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**10>.RUN**

 ![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191020110907761-844770847.png)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　RUN用于指定docker build过程中运行的程序，其可以时任何命令;　　如上图所示，RUN指令所执行的命令依赖于基础镜像(由FROM指令指定的镜像名称)，换句话说，如果该基础镜像不存在的命令，我们就不能执行;　　基础语法：　　　　RUN <command> 或者 RUN ["<executable>","<paraml>","<param2>"]　　操作准则:　　　　1>.第一种格式中，<command>通常是一个shell命令，且以"/bin/sh -c"来运行它，这意味着此进程在容器中的PID不为1，不能接收Unix信号，因此，当使用docker stop <container>命令停止容器时，此进程接收不到SIGTERM信号;　　　　2>.第二种语法格式中的参数是JSON格式的数组，其中<executable>为要运行的命令，后面的<parmN>为传递命令的选项或参数；然而，此种格式指定的命令不会以"/bin/sh -c"来发起(换句话数，此种格式指令的命令直接由内核创建)，因此常见的shell操作如变量替换以及通配符(?,*等)替换将不会进行;不过，如果运行的命令依赖于此shell特性的话，可以将其替换为:RUN ["/bin/bash","-c","<executable>","<param 1>"] 　　注意：json数组中，要使用双引号。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image01/
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# vim Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# cat Dockerfile 　　　　　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加RUN指令代码案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox:latest
#添加作者信息，不推荐使用
MAINTAINER "yinzhengjie <y1053419035@qq.com>"
#建议使用LABEL,它可以代替MAINTAINER的功能
#LABEL maintainer="yinzhengjie <y1053419035@qq.com>"
#我们此处可以定义多个变量,其中"\"表示续行符
ENV DOC_ROOT=/data/web/html/ \
    WEB_SERVER_PACKAGE="nginx-1.17.4.tar.gz"
#使用上面定义的环境变量DOC_ROOT，如果该变量不存在或没有赋值时，则使用咱们自定义的值"/data/web/html/",这样写保险点
COPY index.html ${DOC_ROOT:-/data/web/html/}
#复制目录下的文件级子目录到镜像的"/etc/yum.repos.d/"目录中
COPY yum.repos.d /etc/yum.repos.d/
#指定工作目录,此处我们指定了工作目录，下面直接就可以使用相对路径指定解压目录位置啦！
WORKDIR /usr/local/
#将网络上的Nginx安装包下载到镜像的"/usr/local/soft/"目录中
ADD http://nginx.org/download/${WEB_SERVER_PACKAGE} ./download/
#调用上面定义的变量
#ADD ${WEB_SERVER_PACKAGE} ./nginx
#指定挂在卷
VOLUME /data/mysql/
#此处咱们暴露80端口
EXPOSE 80/udp 80/tcp
#编译时指定要运行的命令
RUN cd /usr/local/download && \
    tar xf ${WEB_SERVER_PACKAGE} 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# cat Dockerfile 　　　　　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加RUN指令代码案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-8              42c71beb05d5        24 minutes ago      7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        55 minutes ago      7.45MB
tinyhttpd           v0.1-6              2368b7546664        About an hour ago   7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        About an hour ago   7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        2 hours ago         7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        2 hours ago         2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        2 hours ago         1.23MB
tinyhttpd           v0.1-1              794f641ae85e        2 hours ago         1.22MB
redis               latest              de25a81a5a0b        2 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker build -t tinyhttpd:v0.1-9 ./　　　　　　#编译镜像
Sending build context to Docker daemon  1.061MB
Step 1/10 : FROM busybox:latest
 ---> 19485c79a9bb
Step 2/10 : MAINTAINER "yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 84ed65a65c8b
Step 3/10 : ENV DOC_ROOT=/data/web/html/     WEB_SERVER_PACKAGE="nginx-1.17.4.tar.gz"
 ---> Using cache
 ---> ca3884045a32
Step 4/10 : COPY index.html ${DOC_ROOT:-/data/web/html/}
 ---> Using cache
 ---> a09b2118c19f
Step 5/10 : COPY yum.repos.d /etc/yum.repos.d/
 ---> Using cache
 ---> a8e64ad1b8dc
Step 6/10 : WORKDIR /usr/local/
 ---> Using cache
 ---> e1c6bfdebbf2
Step 7/10 : ADD http://nginx.org/download/${WEB_SERVER_PACKAGE} ./download/
Downloading [==================================================>]  1.035MB/1.035MB
 ---> Using cache
 ---> ee365ae66291
Step 8/10 : VOLUME /data/mysql/
 ---> Using cache
 ---> e018a3bd068f
Step 9/10 : EXPOSE 80/udp 80/tcp
 ---> Using cache
 ---> 82f80dee6ec6
Step 10/10 : RUN cd /usr/local/download &&     tar xf ${WEB_SERVER_PACKAGE}
 ---> Running in 0a1416ea8380
Removing intermediate container 0a1416ea8380
 ---> 24c887b252ad
Successfully built 24c887b252ad
Successfully tagged tinyhttpd:v0.1-9
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.1-9              24c887b252ad        About a minute ago   8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        30 minutes ago       7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        About an hour ago    7.45MB
tinyhttpd           v0.1-6              2368b7546664        About an hour ago    7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        About an hour ago    7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        2 hours ago          7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        2 hours ago          2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        2 hours ago          1.23MB
tinyhttpd           v0.1-1              794f641ae85e        2 hours ago          1.22MB
redis               latest              de25a81a5a0b        3 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# 
```

\[root@node101.yinzhengjie.org.cn ~/image01\]# docker build -t tinyhttpd:v0.1-9 ./　　　　　　#编译镜像

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1  -it --rm tinyhttpd:v0.1-9 ls /usr/local/download　　　　#发现文件下载后被解压了
nginx-1.17.4         nginx-1.17.4.tar.gz
[root@node101.yinzhengjie.org.cn ~/image01]# 
[root@node101.yinzhengjie.org.cn ~/image01]# docker run --name tinyweb1 -it --rm tinyhttpd:v0.1-9 ls /usr/local/download/nginx-1.17.4
CHANGES LICENSE auto configure html src
CHANGES.ru README conf contrib man
[root@node101.yinzhengjie.org.cn ~/image01]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**11>.CMD**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　类似于RUN指令，CMD指令也可用于任何命令或应用程序，不过，二者的运行时间点不同。　　　　RUN指令运行于映像文件构建过程中，而CMD指令运行于基于Dockerfile构建出的新映像文件启动一个容器时;　　　　CMD指令的首要目的在于为启动的容器指定默认要运行的程序，且其运行结束后，容器也将终止，不过CMD指定的命令其可用被docker run的命令行选项所覆盖;　　　　在Dockerfile中可以存在多个CMD指令，但仅最后一个会生效，而如果有多个RUN指令，则每个RUN指令都会从上到下依次执行。　　语法格式如下:　　　　CMD <command>　　　　CMD ["executable","<param1>","<param2>"]　　　　CMD ["<param1>","<param2>"]　　注意:前两种语法格式的意义同RUN指令，第三种则用于为ENTRYPOINT指令提供默认参数。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# mkdir image02 && cd image02
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# vim Dockerfile
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# cat Dockerfile　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加CMD指令代码案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox

LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"

ENV WEB_DOC_ROOT="/data/web/html"

RUN mkdir -p $WEB_DOC_ROOT && \
    echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.html

CMD /bin/httpd -f -h ${WEB_DOC_ROOT}
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# cat Dockerfile　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加CMD指令代码案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image02]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.1-9              24c887b252ad        20 hours ago        8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        21 hours ago        7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        21 hours ago        7.45MB
tinyhttpd           v0.1-6              2368b7546664        21 hours ago        7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        22 hours ago        7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        22 hours ago        7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        22 hours ago        2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        22 hours ago        1.23MB
tinyhttpd           v0.1-1              794f641ae85e        22 hours ago        1.22MB
redis               latest              de25a81a5a0b        3 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# docker build -t tinyhttpd:v0.2-1 ./
Sending build context to Docker daemon   2.56kB
Step 1/5 : FROM busybox
 ---> 19485c79a9bb
Step 2/5 : LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"
 ---> Running in e637b3c7b043
Removing intermediate container e637b3c7b043
 ---> 5b8318b6ff1a
Step 3/5 : ENV WEB_DOC_ROOT="/data/web/html"
 ---> Running in cd16357fa8c3
Removing intermediate container cd16357fa8c3
 ---> 097b9a975359
Step 4/5 : RUN mkdir -p $WEB_DOC_ROOT &&     echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.h
tml ---> Running in c91ae06fa275
Removing intermediate container c91ae06fa275
 ---> e474d58cb924
Step 5/5 : CMD /bin/httpd -f -h ${WEB_DOC_ROOT}
 ---> Running in 22e67c261cf0
Removing intermediate container 22e67c261cf0
 ---> 807230671730
Successfully built 807230671730
Successfully tagged tinyhttpd:v0.2-1
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tinyhttpd           v0.2-1              807230671730        About a minute ago   1.22MB
tinyhttpd           v0.1-9              24c887b252ad        20 hours ago         8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        21 hours ago         7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        21 hours ago         7.45MB
tinyhttpd           v0.1-6              2368b7546664        21 hours ago         7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        22 hours ago         7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        22 hours ago         7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        22 hours ago         2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        22 hours ago         1.23MB
tinyhttpd           v0.1-1              794f641ae85e        22 hours ago         1.22MB
redis               latest              de25a81a5a0b        3 days ago           98.2MB
busybox             latest              19485c79a9bb        6 weeks ago          1.22MB
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# docker build -t tinyhttpd:v0.2-1 ./　　　　　　#编译镜像

```
[root@node101.yinzhengjie.org.cn ~/image02]# docker image inspect tinyhttpd:v0.2-1 -f {{.ContainerConfig.Cmd}}　　　　　　#查看镜像的CMD配置信息是否生效
[/bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin/httpd -f -h ${WEB_DOC_ROOT}"]]
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# docker run --name tinyweb02 -it --rm tinyhttpd:v0.2-1　　　　　　　　　　　　　　#不难发现当我们运行容器时尽管加了"-it"选项已经无法进入到交互式界面了，而是依旧阻塞在当前终端，起源于时因为启动容器时默认是运行httpd服务的，而httpd服务是没有类似"/bin/sh"一样的功能,此时httpd是作为"/bin/sh"的一个子进程运行的。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
12f228a0646c        tinyhttpd:v0.2-1    "/bin/sh -c '/bin/ht…"   2 minutes ago       Up 2 minutes                            tinyweb02
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker exec -it tinyweb02 /bin/sh　　　　　　　　#进入"tinyweb02"容器的shell，查看相关信息
/ # 
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/httpd -f -h /data/web/html
    6 root      0:00 /bin/sh
   11 root      0:00 ps
/ # 
/ # printenv 
WEB_DOC_ROOT=/data/web/html
HOSTNAME=12f228a0646c
SHLVL=1
HOME=/root
TERM=xterm
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
/ # 
/ # netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 :::80                   :::*                    LISTEN      
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container inspect tinyweb02 -f {{.NetworkSettings.IPAddress}}
192.168.100.1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 192.168.100.1
<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker exec -it tinyweb02 /bin/sh　　　　　　　　#进入"tinyweb02"容器的shell，查看相关信息

**12>.ENTRYPOINT**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　类似CMD指令的功能，用于容器指定默认运行程序，从而使得容器像一个单独的可执行程序;
　　与CMD不同的是，由ENTRYPOINT启动的程序不会被docker run命令行指定的参数所覆盖，而且，这些命令行参数会被当做参数传递给ENTRYPOINT指令指定的程序(不过，docker run 命令的"--entrypoint"选项参数可覆盖ENTRYPOINT指令指定的程序)　　语法格式如下:　　　　ENTR YPOINT <command>　　　　ENTR YPOINT ["<executable>","<param1>","<param2>"]　　docker run命令传入的命令参数会覆盖CMD指令的内容并且附加到ENTRYPOINT命令最后做为其参数使用;　　Dockerfile文件中也可以存在多个ENTRYPOINT指令，但仅有最后一个会生效。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image02/
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# vim Dockerfile 
[root@node101.yinzhengjie.org.cn ~/image02]#
[root@node101.yinzhengjie.org.cn ~/image02]# cat Dockerfile 　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加ENTRYPOINT指令代码案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox
#此处我们可以定义多个标签
LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"

ENV WEB_DOC_ROOT="/data/web/html"
#制作镜像时会运行RUN指令
RUN mkdir -p $WEB_DOC_ROOT && \
    echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.html
#制作镜像时不会运行CMD指令，只有将该镜像当作容器运行时才会执行CMD指令
#CMD /bin/httpd -f -h ${WEB_DOC_ROOT}
#定义ENTRYPOINT指令
ENTRYPOINT /bin/httpd -f -h ${WEB_DOC_ROOT}
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# cat Dockerfile 　　　　　　　　　　　　#编辑Dockerfile配置文件并编译(添加ENTRYPOINT指令代码案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image02]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.2-1              807230671730        41 minutes ago      1.22MB
tinyhttpd           v0.1-9              24c887b252ad        21 hours ago        8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        21 hours ago        7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        22 hours ago        7.45MB
tinyhttpd           v0.1-6              2368b7546664        22 hours ago        7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        22 hours ago        7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        22 hours ago        7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        23 hours ago        2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        23 hours ago        1.23MB
tinyhttpd           v0.1-1              794f641ae85e        23 hours ago        1.22MB
redis               latest              de25a81a5a0b        3 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# docker build -t tinyhttpd:v0.2-2 ./　　　　　　　　#制作镜像
Sending build context to Docker daemon   2.56kB
Step 1/5 : FROM busybox
 ---> 19485c79a9bb
Step 2/5 : LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"
 ---> Using cache
 ---> 5b8318b6ff1a
Step 3/5 : ENV WEB_DOC_ROOT="/data/web/html"
 ---> Using cache
 ---> 097b9a975359
Step 4/5 : RUN mkdir -p $WEB_DOC_ROOT &&     echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.h
tml ---> Using cache
 ---> e474d58cb924
Step 5/5 : ENTRYPOINT /bin/httpd -f -h ${WEB_DOC_ROOT}
 ---> Running in cffeabca2487
Removing intermediate container cffeabca2487
 ---> c7447ee49e2b
Successfully built c7447ee49e2b
Successfully tagged tinyhttpd:v0.2-2
[root@node101.yinzhengjie.org.cn ~/image02]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tinyhttpd           v0.2-2              c7447ee49e2b        5 seconds ago       1.22MB
tinyhttpd           v0.2-1              807230671730        41 minutes ago      1.22MB
tinyhttpd           v0.1-9              24c887b252ad        21 hours ago        8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        21 hours ago        7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        22 hours ago        7.45MB
tinyhttpd           v0.1-6              2368b7546664        22 hours ago        7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        22 hours ago        7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        22 hours ago        7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        23 hours ago        2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        23 hours ago        1.23MB
tinyhttpd           v0.1-1              794f641ae85e        23 hours ago        1.22MB
redis               latest              de25a81a5a0b        3 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# docker build -t tinyhttpd:v0.2-2 ./　　　　　　　　#制作镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image02]# docker run --name tinyweb02  --rm tinyhttpd:v0.2-2 ls /data/web/html　　　　　　#运行该指令时发现会阻塞在当前终端，且"ls /data/web/html"指令无法执行，因为默认情况下启动容器时无法覆盖"ENTRYPOINT"指令，可通过"docker exec"命令进入tinyweb02容器查看究竟

另外启动终端，查看容器运行情况：
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
2e2c52bed064        tinyhttpd:v0.2-2    "/bin/sh -c '/bin/ht…"   21 seconds ago      Up 20 seconds                           tinyweb02
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker exec -it tinyweb02 /bin/sh
/ # 
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/httpd -f -h /data/web/html
    6 root      0:00 /bin/sh
   11 root      0:00 ps
/ # 
/ # netstat -ntl 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 :::80                   :::*                    LISTEN      
/ # 
/ # hostname -i
192.168.100.1
/ # 
/ # wget -O - -q 192.168.100.1
<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 192.168.100.1
<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# docker run --name tinyweb02 --rm tinyhttpd:v0.2-2 ls /data/web/html　　　　　　#运行该指令时发现会阻塞在当前终端，且"ls /data/web/html"指令无法执行，因为默认情况下启动容器时无法覆盖"ENTRYPOINT"指令，可通过"docker exec"命令进入tinyweb02容器查看究竟(详情请戳我)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image02]# cat Dockerfile 　　　　　　　　　　　　　　#CMD,ENTRYPOINT以及手动传参案例
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM busybox
#此处我们可以定义多个标签
LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"

ENV WEB_DOC_ROOT="/data/web/html"
#制作镜像时会运行RUN指令
RUN mkdir -p $WEB_DOC_ROOT && \
    echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.html
#CMD这种格式定义会以默认参数传递给ENTRYPOINT指令
CMD ["/bin/httpd", "-f", "-h ${WEB_DOC_ROOT"}
#定义ENTRYPOINT指令为一个shell,来接收默认的CMD指令
ENTRYPOINT ["/bin/sh", "-c"]
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# docker build -t tinyhttpd:v0.2-3 ./
Sending build context to Docker daemon   2.56kB
Step 1/6 : FROM busybox
 ---> 19485c79a9bb
Step 2/6 : LABEL  maintainer="yinzhengjie <y1053419035@qq.com>" app="httpd"
 ---> Using cache
 ---> 5b8318b6ff1a
Step 3/6 : ENV WEB_DOC_ROOT="/data/web/html"
 ---> Using cache
 ---> 097b9a975359
Step 4/6 : RUN mkdir -p $WEB_DOC_ROOT &&     echo "<h1>Busybox httpd server.[YinZhengjie dao ci yi you]</h1>" > ${WEB_DOC_ROOT}/index.h
tml ---> Using cache
 ---> e474d58cb924
Step 5/6 : CMD ["/bin/httpd", "-f", "-h ${WEB_DOC_ROOT"}
 ---> Running in a98eff300ce5
Removing intermediate container a98eff300ce5
 ---> 1c01ad3c2d71
Step 6/6 : ENTRYPOINT ["/bin/sh", "-c"]
 ---> Running in b2326c3d454f
Removing intermediate container b2326c3d454f
 ---> c06eb08ea6d6
Successfully built c06eb08ea6d6
Successfully tagged tinyhttpd:v0.2-3
[root@node101.yinzhengjie.org.cn ~/image02]# 
[root@node101.yinzhengjie.org.cn ~/image02]# docker run --name tinyweb02  --rm tinyhttpd:v0.2-3  "ls /data/web/html"　　　　#此处传参覆盖了CMD指令，而CMD原来的指令是传递给ENTRYPOINT指令的，从配置文件可以看出来ENTRYPOINT为一个shell，因此，命令行启动容器时的指令代替配置文件中CMD指令并代替它传递给了ENTRYPOINT指令，因此我们会看到有内容输出。
index.html
[root@node101.yinzhengjie.org.cn ~/image02]# 
```

\[root@node101.yinzhengjie.org.cn ~/image02\]# cat Dockerfile 　　　　　　　　　　　　　　#CMD,ENTRYPOINT以及手动传参案例

　　**综述案例可编写一套手动传参修改nginx的案例**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image03/
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# ll
total 12
-rw-r--r-- 1 root root 1003 Oct 22 11:24 Dockerfile
-rwxr-xr-x 1 root root  446 Oct 22 11:23 entrypoint.sh
-rw-r--r-- 1 root root   35 Oct 22 11:22 index.html
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# cat Dockerfile 
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

#指定基础镜像
FROM nginx:1.14-alpine

#添加一个标签
LABEL  maintainer="yinzhengjie <y1053419035@qq.com>"

#添加nginx的根目录
ENV NGINX_DOC_ROOT='/data/web/html/'

#自定义首页文件
ADD index.html ${NGINX_DOC_ROOT}

#将我们编写号的"entrypoint.sh"脚本(用于初始化配置文件的脚本)复制到容器的/bin目录中
ADD entrypoint.sh /bin/

#下面的"-g"表示让nginx的配置全局生效,使用"daemon off"让nginx不要在后台运行而是得在前台运行
CMD ["/usr/sbin/nginx","-g","daemon off;"]

#还记得我们上面讲"entrypoint.sh"放在容器的"/bin"目录吗?现在我们让容器运行时就调用该脚本(需要注意得有执行权限哟)
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat Dockerfile　　　　　　　　　　　　　　#编写Dockerfile文件,使用ENTRYPOINT调用自定义脚本初始化nginx的配置文件

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# cat entrypoint.sh 　　　　　　　　　　　　#用于自定义nginx的初始化配置文件
#!/bin/sh
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

cat > /etc/nginx/conf.d/www.conf << EOF
server {
    server_name ${HOSTNAME};
    listen ${IP:-0.0.0.0}:${PORT:-80};
    root ${NGINX_DOC_ROOT:-/usr/share/nginx/html};
}
EOF

#注意，在本案例中,这里的exec指令可以让调用该脚本的进程pid为1
exec "$@"
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat entrypoint.sh 　　　　　　　　　　　　#用于自定义nginx的初始化配置文件

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# cat index.html 　　　　　　　　　　　　　　#自定义用于测试的首页文字
<h1>YinZhengjie dao ci yi you</h1>
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# ll
total 12
-rw-r--r-- 1 root root 1003 Oct 22 11:24 Dockerfile
-rwxr-xr-x 1 root root  532 Oct 22 11:34 entrypoint.sh
-rw-r--r-- 1 root root   35 Oct 22 11:22 index.html
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat index.html 　　　　　　　　　　　　　　#自定义用于测试的首页文字

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker build -t myweb:v0.3-6 ./　　　　　　#编译镜像
Sending build context to Docker daemon  4.608kB
Step 1/7 : FROM nginx:1.14-alpine
 ---> 8a2fb25a19f5
Step 2/7 : LABEL  maintainer="yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 1a323f83eb7e
Step 3/7 : ENV NGINX_DOC_ROOT='/data/web/html/'
 ---> Using cache
 ---> 685d6ec7485e
Step 4/7 : ADD index.html ${NGINX_DOC_ROOT}
 ---> 7504f50d1c25
Step 5/7 : ADD entrypoint.sh /bin/
 ---> 4328b56b48c1
Step 6/7 : CMD ["/usr/sbin/nginx","-g","daemon off;"]
 ---> Running in 3f15cbd355f2
Removing intermediate container 3f15cbd355f2
 ---> 11f24894e2a5
Step 7/7 : ENTRYPOINT ["/bin/entrypoint.sh"]
 ---> Running in 7266ce370568
Removing intermediate container 7266ce370568
 ---> 0082820cf7f0
Successfully built 0082820cf7f0
Successfully tagged myweb:v0.3-6
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker build -t myweb:v0.3-6 ./　　　　　　#编译镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker run --name myweb01 -P --rm  myweb:v0.3-6　　　　　　#运行容器，如果有人访问会有响应的日志信息在终端显示
127.0.0.1 - - [22/Oct/2019:03:27:13 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
192.168.100.1 - - [22/Oct/2019:03:27:32 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker run --name myweb01 -P --rm myweb:v0.3-6　　　　　　#运行容器，如果有人访问会有响应的日志信息在终端显示

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker exec -it myweb01 /bin/sh　　　　　　　　　　　　#登录web01容器查看相关配置信息及启动端口并验证服务nginx是否正常
/ # 
/ # cat /etc/nginx/conf.d/www.conf 
server {
    server_name 848a98c91243;
    listen 0.0.0.0:80;
    root /data/web/html/;
}
/ # 
/ # netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      
/ # 
/ # wget -O - -q 848a98c91243
<h1>YinZhengjie dao ci yi you</h1>
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
848a98c91243        myweb:v0.3-6        "/bin/entrypoint.sh …"   2 minutes ago       Up 2 minutes        0.0.0.0:32775->80/tcp   myweb01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect myweb01 -f {{.NetworkSettings.IPAddress}}
192.168.100.1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 192.168.100.1
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker exec -it myweb01 /bin/sh　　　　　　　　　　　　#登录web01容器查看相关配置信息及启动端口并验证服务nginx是否正常

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker run --name myweb01 -P -e "PORT=8080" --rm  myweb:v0.3-6　　　　#启动时指定监听8080端口，

#另外启动一个终端使用docker exec相关命令进入容器验证指定端口是否启动成功，具体操作如下:
[root@node101.yinzhengjie.org.cn ~]# docker exec -it myweb01 /bin/sh
/ # 
/ # netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address State 
tcp 0 0 0.0.0.0:8080 0.0.0.0:* LISTEN 
tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN 
/ # 
/ # hostname 
d4010c281f5b
/ # 
/ # wget -O - -q http://d4010c281f5b:8080
<h1>YinZhengjie dao ci yi you</h1>
/ #
/ # ps
PID USER TIME COMMAND
1 root 0:00 nginx: master process /usr/sbin/nginx -g daemon off;
8 nginx 0:00 nginx: worker process
24 root 0:00 /bin/sh
33 root 0:00 ps
/ #
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**13>.USER**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　用于指定运行image时的或运行Dockerfile中任何RUN，CMD或ENTRYPOINT指令指定的程序时的用户名或UID
　　默认情况下，container的运行身份为root用户

　　语法格式如下:　　　　USER <UID>|<UserName>　　　　需要注意的是,<UID>可以写任意数字，但实践中必须为/etc/passwd中某用户的有效UID,否则docker run命令将运行失败。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**14>.HEALTHCHECK**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　HEALTHCHECK指令告诉docker如何测试容器以检查它是否仍在工作;　　这可以检测到一些情况，例如web服务器陷入无限循环，无法处理新连接，即使服务器进程仍在运行;　　健康检查指令有两种形式:　　　　HELTHCHECK [OPTIONS] CMD command(通过在容器内运行命令检查容器运行状况)　　　　HELTHCHECK NONE(禁用从基本映像继承的任何运行状况检查)　　　　操作准则:　　　　1>.可以在CMD之前显示的OPTIONS有:　　　　　　--interval=DURATION(表示间隔时间。default:30s)　　　　　　--timeout=DURATION(表示超时时长。default:30s)　　　　　　--start-period=DURATION(表示当容器启动成功后，间隔多少秒开始检测,容器的启动是非常快的，如果容器内部运行的服务启动时间较长，可以适当调大该值。default:0s)　　　　　　--retries=N(表示重试的次数。defaluts:3)　　　　2>.command的退出状态指示容器的健康状态。可能的值是：　　　　　　0:success(容器是健康的，可以使用)　　　　　　1:unhealthy(容器工作不正常)　　　　　　2:reserved(预留的代码表示，目前没有什么意义，该值咱们可以用来自定义状态，不过一般情况下，上面两种表示应该通用。)　　　　举个例子:HEALTHCHCK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image03
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# ll
total 12
-rw-r--r-- 1 root root 1146 Oct 22 15:43 Dockerfile
-rwxr-xr-x 1 root root  532 Oct 22 11:34 entrypoint.sh
-rw-r--r-- 1 root root   35 Oct 22 11:22 index.html
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# cat Dockerfile 　　　　　　　　　　　　　　　　#编写Dockerfile文件(使用HEALTHCHCK指令案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

#指定基础镜像
FROM nginx:1.14-alpine

#添加一个标签
LABEL  maintainer="yinzhengjie <y1053419035@qq.com>"

#添加nginx的根目录
ENV NGINX_DOC_ROOT='/data/web/html/'

#自定义首页文件
ADD index.html ${NGINX_DOC_ROOT}

#将我们编写号的"entrypoint.sh"脚本(用于初始化配置文件的脚本)复制到容器的/bin目录中
ADD entrypoint.sh /bin/

#暴露80端口
EXPOSE 80/tcp

#指定健康检查的方式
HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/

#下面的"-g"表示让nginx的配置全局生效,使用"daemon off"让nginx不要在后台运行而是得在前台运行
CMD ["/usr/sbin/nginx","-g","daemon off;"]

#还记得我们上面讲"entrypoint.sh"放在容器的"/bin"目录吗?现在我们让容器运行时就调用该脚本(需要注意得有执行权限哟)
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat Dockerfile 　　　　　　　　　　　　　　　　#编写Dockerfile文件(使用HEALTHCHCK指令案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker build -t myweb:v0.3-7 ./                                               #编译镜象文件
Sending build context to Docker daemon  5.632kB
Step 1/9 : FROM nginx:1.14-alpine
 ---> 8a2fb25a19f5
Step 2/9 : LABEL  maintainer="yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 1a323f83eb7e
Step 3/9 : ENV NGINX_DOC_ROOT='/data/web/html/'
 ---> Using cache
 ---> 685d6ec7485e
Step 4/9 : ADD index.html ${NGINX_DOC_ROOT}
 ---> Using cache
 ---> 7504f50d1c25
Step 5/9 : ADD entrypoint.sh /bin/
 ---> 9420fd8828c3
Step 6/9 : EXPOSE 80/tcp
 ---> Running in 6f3630f2e812
Removing intermediate container 6f3630f2e812
 ---> 44b493c15e03
Step 7/9 : HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/
 ---> Running in 55c2aa83b071
Removing intermediate container 55c2aa83b071
 ---> 104925b8c803
Step 8/9 : CMD ["/usr/sbin/nginx","-g","daemon off;"]
 ---> Running in de5058be298e
Removing intermediate container de5058be298e
 ---> a8e403e34336
Step 9/9 : ENTRYPOINT ["/bin/entrypoint.sh"]
 ---> Running in 88e7b12a9170
Removing intermediate container 88e7b12a9170
 ---> cb08b09cd959
Successfully built cb08b09cd959
Successfully tagged myweb:v0.3-7
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker build -t myweb:v0.3-7 ./ 　　　　　　#编译镜象文件

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker run --name myweb01 -P -e "PORT=8080" --rm  myweb:v0.3-7　　　　　　#运行容器发现的确是30秒检测一次，而且服务是正常的
127.0.0.1 - - [22/Oct/2019:07:46:11 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:46:41 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:47:12 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:47:43 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:48:14 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:48:44 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:49:15 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:49:45 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:50:16 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:50:46 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:51:16 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:51:47 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:52:17 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:52:47 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:07:53:19 +0000] "GET / HTTP/1.1" 200 35 "-" "Wget" "-"
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker run --name myweb01 -P -e "PORT=8080" --rm myweb:v0.3-7　　　　　　#运行容器发现的确是30秒检测一次，而且服务是正常的

**15>.SHELL**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　SHELL指令允许重写用于命令的SHELL形式的默认SHELL;

　　Linux上的默认shell为[“/bin/sh”、“-c”]，Windows上的默认shell为[“cmd”、“/s”、“/c”];

　　SHELL指令必须用json从dockerfile中写入（注意：json数组中，要使用双引号)
　　　　语法格式为:SHELL ["executable","parameters"]

　　SHELL指令可以出现多次,每一个SHELL指令都覆盖所有以前的SHELL指令，并影响所有后续的shell指令;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**16>.STOPSIGNAL**

```
　　停止信号指令设置将被发送到容器退出的系统调用信号;　　信号可以是与内核syscall表中的位置（例如9）匹配的有效无符号数，也可以是signame格式的信号名（例如sigkill）。　　语法格式:STOPSIGNAL signal
```

**17>.ARG（在build阶段进行传参）**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　ARG指令定义了一个变量，用户可以在构建时使用docker build命令使用--build arg<varname>=<value>标志将该变量传递给构建器;　　如果用户指定的生成参数未在dockerfile中定义，则生成将输出警告;　　语法格式:ARG <name>[=<default value>]　　dockerfile可以包含一个或多个ARG指令,ARG指令可以选择包含默认值:　　　　ARG version=1.14　　　　ARG user=yinzhengjie
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# cat Dockerfile 　　　　　　　　　　#编写Dockerfile文件(使用ARG指令案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

#指定基础镜像
FROM nginx:1.14-alpine

#定义一个"nginx_tag"的变量
ARG author="yinzhengjie <y1053419035@qq.com>"

#添加一个标签
LABEL  maintainer="${author}"

#添加nginx的根目录
ENV NGINX_DOC_ROOT='/data/web/html/'

#自定义首页文件
ADD index.html ${NGINX_DOC_ROOT}

#将我们编写号的"entrypoint.sh"脚本(用于初始化配置文件的脚本)复制到容器的/bin目录中
ADD entrypoint.sh /bin/

#暴露80端口
EXPOSE 80/tcp

#指定健康检查的方式
HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/

#下面的"-g"表示让nginx的配置全局生效,使用"daemon off"让nginx不要在后台运行而是得在前台运行
CMD ["/usr/sbin/nginx","-g","daemon off;"]

#还记得我们上面讲"entrypoint.sh"放在容器的"/bin"目录吗?现在我们让容器运行时就调用该脚本(需要注意得有执行权限哟)
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat Dockerfile 　　　　　　　　　　#编写Dockerfile文件(使用ARG指令案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker build -t myweb:v0.3-8 ./　　　　　　#编译时未传参(ARG)案例
Sending build context to Docker daemon  5.632kB
Step 1/10 : FROM nginx:1.14-alpine
 ---> 8a2fb25a19f5
Step 2/10 : ARG author="yinzhengjie <y1053419035@qq.com>"
 ---> Running in 7c23e18fdf27
Removing intermediate container 7c23e18fdf27
 ---> 2df7c2da984d
Step 3/10 : LABEL  maintainer="${author}"
 ---> Running in f653ba0babef
Removing intermediate container f653ba0babef
 ---> 307a9c4e074f
Step 4/10 : ENV NGINX_DOC_ROOT='/data/web/html/'
 ---> Running in 702cb41bd15b
Removing intermediate container 702cb41bd15b
 ---> e821637668df
Step 5/10 : ADD index.html ${NGINX_DOC_ROOT}
 ---> e6fe7e71117b
Step 6/10 : ADD entrypoint.sh /bin/
 ---> 9b4a8cf43e9d
Step 7/10 : EXPOSE 80/tcp
 ---> Running in 54dea44e7ea6
Removing intermediate container 54dea44e7ea6
 ---> ea211541c616
Step 8/10 : HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/
 ---> Running in 7c3cfc17a43c
Removing intermediate container 7c3cfc17a43c
 ---> 32c54a017574
Step 9/10 : CMD ["/usr/sbin/nginx","-g","daemon off;"]
 ---> Running in ff4426e3e60f
Removing intermediate container ff4426e3e60f
 ---> c42e3634b898
Step 10/10 : ENTRYPOINT ["/bin/entrypoint.sh"]
 ---> Running in 440ca41ef8e4
Removing intermediate container 440ca41ef8e4
 ---> 0110cfd06ce1
Successfully built 0110cfd06ce1
Successfully tagged myweb:v0.3-8
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker build -t myweb:v0.3-8 ./　　　　　　#编译时未给author传参案例

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker inspect myweb:v0.3-8 -f {{.Config.Labels}}　　　　　　#发现其值为Dockerfile文件默认的值
map[maintainer:yinzhengjie <y1053419035@qq.com>]
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker build --build-arg author="jason <jason@qq.com>" -t myweb:v0.3-9 ./    　　　　#编译时为author传参案例
Sending build context to Docker daemon  5.632kB
Step 1/10 : FROM nginx:1.14-alpine
 ---> 8a2fb25a19f5
Step 2/10 : ARG author="yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 2df7c2da984d
Step 3/10 : LABEL  maintainer="${author}"
 ---> Running in b3b17ac03da9
Removing intermediate container b3b17ac03da9
 ---> 76d57d18c1b5
Step 4/10 : ENV NGINX_DOC_ROOT='/data/web/html/'
 ---> Running in aff465ebab34
Removing intermediate container aff465ebab34
 ---> 628e53c5bc97
Step 5/10 : ADD index.html ${NGINX_DOC_ROOT}
 ---> d865631b87ba
Step 6/10 : ADD entrypoint.sh /bin/
 ---> 023a344ac94f
Step 7/10 : EXPOSE 80/tcp
 ---> Running in 5905a76c64b9
Removing intermediate container 5905a76c64b9
 ---> 18cb994bd91b
Step 8/10 : HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/
 ---> Running in f30f2aeac3f8
Removing intermediate container f30f2aeac3f8
 ---> 8387fe1c588d
Step 9/10 : CMD ["/usr/sbin/nginx","-g","daemon off;"]
 ---> Running in a0f4084ecd26
Removing intermediate container a0f4084ecd26
 ---> 68239d61c039
Step 10/10 : ENTRYPOINT ["/bin/entrypoint.sh"]
 ---> Running in 5295ef7926a2
Removing intermediate container 5295ef7926a2
 ---> 6f9999d46b69
Successfully built 6f9999d46b69
Successfully tagged myweb:v0.3-9
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker build --build-arg author="jason <jason@qq.com>" -t myweb:v0.3-9 ./ 　　　　#编译时为author传参案例

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker inspect myweb:v0.3-9 -f {{.Config.Labels}}　　　　　　#发现默认值已经被传入的参数给修改啦
map[maintainer:jason <jason@qq.com>]
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

**18>.ONBUILD**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　用于在Dockerfile中定义一个触发器;　　Dockerfile用于build映像文件，此映像文件亦可作为base image被另一个Dockerfile用作FROM指令的参数，并以之构建新的的映像文件;　　在后面的这个Dockerfile中的FROM指令在build过程中被执行时,将会"触发"创建其base image的Dockerfile文件中的ONBUILD指令的触发器；　　语法格式为:ONBUOLD <INSTRUCTION>　　尽管任何指令都可注册成为触发器指令，但ONBUILD不难自我嵌套，且不会触发FROM和MAINTAINER指令;　　使用包含ONBUILD指令的Dockerfile构建的镜像应该使用特殊的标签，例如ruby:2.0-onbuild。　　在ONBUILD指令中使用ADD或COPY指令应该格外小心，因为构建过程的上下文在缺少指定的源文件时会失败。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# ll
total 12
-rw-r--r-- 1 root root 1440 Oct 22 16:51 Dockerfile
-rwxr-xr-x 1 root root  532 Oct 22 11:34 entrypoint.sh
-rw-r--r-- 1 root root   35 Oct 22 11:22 index.html
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# cat Dockerfile 　　　　　　　　　　　　　　　　#查看Dockerfile文件(定义ONBUILD语句案例)
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

#指定基础镜像
FROM nginx:1.14-alpine

#定义一个"nginx_tag"的变量
ARG author="yinzhengjie <y1053419035@qq.com>"

#添加一个标签
LABEL  maintainer="${author}"

#添加nginx的根目录
ENV NGINX_DOC_ROOT='/data/web/html/'

#自定义首页文件
ADD index.html ${NGINX_DOC_ROOT}

#将我们编写号的"entrypoint.sh"脚本(用于初始化配置文件的脚本)复制到容器的/bin目录中
ADD entrypoint.sh /bin/

#暴露80端口
EXPOSE 80/tcp

#指定健康检查的方式
HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/

#定义一个ONBUILD指令,如果谁基于该当前dockerfile创建的镜像来指针镜像的话就会调用该语句,千万别忘记ADD指令后面需要指定两个参数啊，我第二个参数是调用上面定义的变量名称
ONBUILD ADD https://img2018.cnblogs.com/blog/795254/201910/795254-20191020060253711-916713316.png ${NGINX_DOC_ROOT}

#下面的"-g"表示让nginx的配置全局生效,使用"daemon off"让nginx不要在后台运行而是得在前台运行
CMD ["/usr/sbin/nginx","-g","daemon off;"]

#还记得我们上面将"entrypoint.sh"放在容器的"/bin"目录吗?现在我们让容器运行时就调用该脚本(需要注意得有执行权限哟)
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node101.yinzhengjie.org.cn ~/image03]# 
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# cat Dockerfile 　　　　　　　　　　　　　　　　#查看Dockerfile文件(定义ONBUILD语句案例)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image03]# docker build  -t myweb:v0.3-11 ./　　　　　　#编译镜像，用生成的镜像文件做其它镜像的基础镜像
Sending build context to Docker daemon  5.632kB
Step 1/11 : FROM nginx:1.14-alpine
 ---> 8a2fb25a19f5
Step 2/11 : ARG author="yinzhengjie <y1053419035@qq.com>"
 ---> Using cache
 ---> 2df7c2da984d
Step 3/11 : LABEL  maintainer="${author}"
 ---> Using cache
 ---> 307a9c4e074f
Step 4/11 : ENV NGINX_DOC_ROOT='/data/web/html/'
 ---> Using cache
 ---> e821637668df
Step 5/11 : ADD index.html ${NGINX_DOC_ROOT}
 ---> Using cache
 ---> e6fe7e71117b
Step 6/11 : ADD entrypoint.sh /bin/
 ---> Using cache
 ---> 9b4a8cf43e9d
Step 7/11 : EXPOSE 80/tcp
 ---> Using cache
 ---> ea211541c616
Step 8/11 : HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80}/
 ---> Using cache
 ---> 32c54a017574
Step 9/11 : ONBUILD ADD https://img2018.cnblogs.com/blog/795254/201910/795254-20191020060253711-916713316.png ${NGINX_DOC_ROOT}
 ---> Running in 0416d569fc16
Removing intermediate container 0416d569fc16
 ---> 984a937d8d06
Step 10/11 : CMD ["/usr/sbin/nginx","-g","daemon off;"]
 ---> Running in fcd1c04ad076
Removing intermediate container fcd1c04ad076
 ---> c56ee7ec4231
Step 11/11 : ENTRYPOINT ["/bin/entrypoint.sh"]
 ---> Running in a1370294a920
Removing intermediate container a1370294a920
 ---> c0a33ce27a6b
Successfully built c0a33ce27a6b
Successfully tagged myweb:v0.3-11
[root@node101.yinzhengjie.org.cn ~/image03]# 
```

\[root@node101.yinzhengjie.org.cn ~/image03\]# docker build -t myweb:v0.3-11 ./　　　　　　#编译镜像，用生成的镜像文件做其它镜像的基础镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd image04
[root@node101.yinzhengjie.org.cn ~/image04]# 
[root@node101.yinzhengjie.org.cn ~/image04]# ll
total 4
-rw-r--r-- 1 root root 315 Oct 22 16:52 Dockerfile
[root@node101.yinzhengjie.org.cn ~/image04]# 
[root@node101.yinzhengjie.org.cn ~/image04]# cat Dockerfile 　　　　　　　　　　　　　　　　#基于上面创建的镜像作为基础镜像案例
#!/bin/bash
#********************************************************************
#Author: YinZhengjie
#Email: y1053419035@qq.com
#Blog： https://www.cnblogs.com/yinzhengjie/
#Description: test image
#********************************************************************

FROM myweb:v0.3-11

RUN mkdir /yinzhengjie
[root@node101.yinzhengjie.org.cn ~/image04]# 
```

\[root@node101.yinzhengjie.org.cn ~/image04\]# cat Dockerfile 　　　　　　　　　　　　　　　　#基于上面创建的镜像作为基础镜像案例

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~/image04]# docker build -t test:v0.1-1 ./　　　　　　　　#不难发现我们在编译时，首先执行的是下载指令，而我们正在Dockerfile文件中并没有指定要下载文件，但却在基础镜像中使用ONBUILD指令啦！
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM myweb:v0.3-11
# Executing 1 build trigger
Downloading [==================================================>]   67.6kB/67.6kB
 ---> 05317026d392
Step 2/2 : RUN mkdir /yinzhengjie
 ---> Running in fcba6ed1f59a
Removing intermediate container fcba6ed1f59a
 ---> d2003d6841b7
Successfully built d2003d6841b7
Successfully tagged test:v0.1-1
[root@node101.yinzhengjie.org.cn ~/image04]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~/image04]# docker run --name test -it --rm test:v0.1-1　　　　　　#基于上面编译的镜像来启动一个容器，需要在启动一个终端来查看内部容器的运行状况
127.0.0.1 - - [22/Oct/2019:08:54:20 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:54:51 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:55:22 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:55:52 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:56:22 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:56:52 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:57:24 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:57:54 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:58:25 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:58:55 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:59:25 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:08:59:56 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:09:00:26 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [22/Oct/2019:09:00:56 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```

\[root@node101.yinzhengjie.org.cn ~/image04\]# docker run --name test -it --rm test:v0.1-1　　　　　　#基于上面编译的镜像来启动一个容器，需要在启动一个终端来查看内部容器的运行状况

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAME
S2b40d69e370e        test:v0.1-1         "/bin/entrypoint.sh …"   39 seconds ago      Up 38 seconds (healthy)   80/tcp              test[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker exec -it test /bin/sh
/ # 
/ # ls /　　　　　　　　　　　　#当前容器的确存在"/yinzhengjie"目录
bin          dev          home         media        opt          root         sbin         sys          usr          yinzhengjie
data         etc          lib          mnt          proc         run          srv          tmp          var
/ # 
/ # ls /data/web/html/　　　　#我们基于myweb:v0.3-11中ONBUILD指令下载的文件的确存在指定目录
795254-20191020060253711-916713316.png  index.html
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186