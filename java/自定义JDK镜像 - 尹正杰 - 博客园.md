　　　　　　　　　　　　　　**自定义JDK镜像**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.镜像分层**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　如下图所示,镜像分层就是将构建镜像的过程进行拆解，找到和其它服务的共同点并将其定制为一个基础镜像，这样可以很大的提示工作效率。有利于镜像的重复利用，就像开发喜欢编写函数来实现代码的复用性原理一样。　　镜像分层的优点:　　　　提升镜像的编译速度(比如基于CentOS制作Nginx镜像，需要安装一大堆依赖环境，从而导致编译速度下降，但如果基于已经安装好相关的依赖包的基础镜像制作Nginx镜像的话,那么就直接安装Nginx服务即可,从而无形中提升编译速度)。　　　　镜像的复用性较强,做好基础镜像公司的其它员工就在制作镜像了，技术支持可以直接使用你做好的镜像去做相应的试验岂不美哉。　　镜像分层的缺点:　　　　基础镜像如果少安装了某个服务，若改动该镜像将导致所有基于该镜像制作的子镜像都发送变动,因此在制作基础镜像是要提前考虑周全哟。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122065712453-1853932522.png)

**二.制作基于CentOS基础镜像**

**1>.**在宿主机上创建存放DockerFile的存储目录(目录结构按照业务类型或者系统类型等方式划分，方便后期镜像比较多的时候进行分类)****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/
total 0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# mkdir -pv /yinzhengjie/softwares/dockerfile/{web/{apache,nginx,tomcat,jdk},system/{centos,ubantu,redhat,suse,debain}}
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
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.不推荐写多个RUN指令，建议将多条RUN指令指定为一行，使用"&&"符号进行连接(以下是验证过程，知道这个技巧的小伙伴可直接跳过当前步骤)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# cat Dockerfile 
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2019-11-25
#Blog:             http://www.cnblogs.com/yinzhengjie
#Description：        YinZhengjie's CentOS Base Dockerfile
#Copyright notice:     original works, no reprint! Otherwise, legal liability will be investigated.
#********************************************************************

#第一行先定义基础镜像,表示当前镜像文件是基于哪个进行编辑的.
FROM centos:centos7.6.1810

#指定镜像维护者的信息.
MAINTAINER Jason.Yin y1053419035@qq.com

#安装常用的命令
RUN yum -y install epel-release && yum -y install vim net-tools bridge-utils firewalld bc iotop bc gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel zip unzip zlib-devel 
lrzsz tree ntpdate telnet lsof tcpdump wget libevent libevent-devel systemd-devel bash-completion traceroute psmisc
#创建基础用户
RUN useradd nginx -u 2019 && useradd tomcat -u 2020 && rm -rf /etc/localtime 

#指定时区,很明显我指令Linux相关命令竟然使用了3个"RUN"指令，那么这意味着该镜像关于RUN指令会多出来3个层次，因此生产环境中建议大家把同一个指令能写完的尽量使用"&&"连接写完即可.
RUN ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122074247295-723007516.png)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]#  docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx-base          v0.1.0              fec9b606a66d        41 seconds ago      551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image history nginx-base:v0.1.0 
IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
fec9b606a66d        About a minute ago   /bin/sh -c ln -sv /usr/share/zoneinfo/Asia/S…   33B                 
32a9f437aee4        About a minute ago   /bin/sh -c useradd nginx -u 2019 && useradd …   595kB               
e8c87f5e07b6        About a minute ago   /bin/sh -c yum -y install epel-release && yu…   348MB               
728084fa237b        2 minutes ago        /bin/sh -c #(nop)  MAINTAINER Jason.Yin y105…   0B                  
f1cb7c7d58b7        10 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           10 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           10 months ago        /bin/sh -c #(nop) ADD file:54b004357379717df…   202MB               
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# ll
total 4
-rw-r--r-- 1 root root 1360 Jan 22 07:51 Dockerfile
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image save nginx-base:v0.1.0 > nginx:v0.1
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# ll -h
total 541M
-rw-r--r-- 1 root root 1.4K Jan 22 07:51 Dockerfile
-rw-r--r-- 1 root root 541M Jan 22 08:05 nginx:v0.1
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122074905239-2072246837.png)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# cat Dockerfile 
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

#安装常用的命令,创建基础用户,指定时区
RUN yum -y install epel-release && yum -y install vim net-tools bridge-utils firewalld bc iotop bc gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel zip unzip zlib-devel 
lrzsz tree ntpdate telnet lsof tcpdump wget libevent libevent-devel systemd-devel bash-completion traceroute psmisc && useradd nginx -u 2019 && useradd tomcat -u 2020 && rm -rf /etc/localtime && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122075228755-1674421691.png)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# ll
total 553428
-rw-r--r-- 1 root root      1119 Jan 22 08:09 Dockerfile
-rw-r--r-- 1 root root 566703104 Jan 22 08:05 nginx:v0.1
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx-base          v0.1.1              29f44b369129        55 seconds ago      551MB
nginx-base          v0.1.0              fec9b606a66d        16 minutes ago      551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image history nginx-base:v0.1.1 
IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
29f44b369129        About a minute ago   /bin/sh -c yum -y install epel-release && yu…   349MB               
728084fa237b        18 minutes ago       /bin/sh -c #(nop)  MAINTAINER Jason.Yin y105…   0B                  
f1cb7c7d58b7        10 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           10 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           10 months ago        /bin/sh -c #(nop) ADD file:54b004357379717df…   202MB               
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image save nginx-base:v0.1.1 > nginx:v0.2
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# ll
total 1106808
-rw-r--r-- 1 root root      1119 Jan 22 08:09 Dockerfile
-rw-r--r-- 1 root root 566703104 Jan 22 08:05 nginx:v0.1
-rw-r--r-- 1 root root 566659072 Jan 22 08:19 nginx:v0.2
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122080215083-606148020.png)

**3>.编写系统基础镜像的Dockerfile文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# cat Dockerfile 
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

#安装常用的命令,创建基础用户,指定时区
RUN yum -y install epel-release && yum -y install vim net-tools bridge-utils firewalld bc iotop bc gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel zip unzip zlib-devel 
lrzsz tree ntpdate telnet lsof tcpdump wget libevent libevent-devel systemd-devel bash-completion traceroute psmisc && useradd nginx -u 2019 && useradd tomcat -u 2020 && rm -rf /etc/localtime && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.编译系统基础镜像(其实也就是安装一些基础命令,修改时区，添加普通用户的功能)**

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# docker image build -t centos-base:7.6.1810 .
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122080909616-582741668.png)

**5>.基础镜像编译成功并验证**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-base         7.6.1810            b4931fd9ace2        About an hour ago   551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm centos-base:7.6.1810 bash
[root@181f160da0ba /]# 
[root@181f160da0ba /]# id nginx
uid=2019(nginx) gid=2019(nginx) groups=2019(nginx)
[root@181f160da0ba /]# 
[root@181f160da0ba /]# id tomcat
uid=2020(tomcat) gid=2020(tomcat) groups=2020(tomcat)
[root@181f160da0ba /]# 
[root@181f160da0ba /]# date -R
Wed, 22 Jan 2020 09:55:23 +0800
[root@181f160da0ba /]# 
[root@181f160da0ba /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**6>.将编译脚本记录(以防止后期你忘记当时编译的tag版本)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# vim build-command.sh
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# cat build-command.sh
#!/bin/bash
#
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2020-01-18
#FileName：        docker-build.sh
#URL:             http://www.cnblogs.com/yinzhengjie
#Description：        Build CentOS base Script
#Copyright (C):     2020 All rights reserved
#********************************************************************

docker image build -t centos-base:7.6.1810 .
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# ll
total 8
-rw-r--r-- 1 root root  468 Jan 22 08:42 build-command.sh
-rw-r--r-- 1 root root 1119 Jan 22 08:09 Dockerfile
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/system/centos]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.基于咱们**自己的"centos-base:7.6.1810"镜像制作jdk基础镜像****

**1>.去Oracle官网下载你业务需要的JDK环境**

```
　　Java官网下载地址:
　　　　https://www.oracle.com/technetwork/java/javase/archive-139210.html
```

**2>.编写profile文件用于覆盖镜像的profile文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/softwares/dockerfile/web/jdk/
total 189616
-rw-r--r-- 1 root root       463 Jan 22 08:58 build-command.sh
-rw-r--r-- 1 root root       130 Jan 22 08:56 Dockerfile
-rw-r--r-- 1 root root 194151339 Jan 19 02:08 jdk-8u231-linux-x64.tar.gz
-rw-r--r-- 1 root root      2109 Jan 22 09:52 profile
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/softwares/dockerfile/web/jdk/profile 
-rw-r--r-- 1 root root 2109 Jan 22 09:52 /yinzhengjie/softwares/dockerfile/web/jdk/profile
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/jdk/profile 
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}


if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`/usr/bin/id -u`
        UID=`/usr/bin/id -ru`
    fi
    USER="`/usr/bin/id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
fi

HOSTNAME=`/usr/bin/hostname 2>/dev/null`
HISTSIZE=1000
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi

for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then 
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

unset i
unset -f pathmunge

#Add ${JAVA_HOME} by yinzhengjie
export JAVA_HOME=/usr/local/jdk
export TOMCAT_HOME=/yinzhengjie/softwares/web/tomcat
export PATH=${JAVA_HOME}/bin:${JAVA_HOME}/jre/bin:${TOMCAT_HOME}/bin:$PATH
export CLASSPATH=.${CLASSPATH}:${JAVA_HOME}/lib:${JAVA_HOME}/jre/lib:${JAVA_HOME}/lib/tools.jar
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.编写Dockerfile**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/softwares/dockerfile/web/jdk/
total 189616
-rw-r--r-- 1 root root       463 Jan 22 08:58 build-command.sh
-rw-r--r-- 1 root root      1089 Jan 22 10:04 Dockerfile
-rw-r--r-- 1 root root 194151339 Jan 19 02:08 jdk-8u231-linux-x64.tar.gz
-rw-r--r-- 1 root root      2109 Jan 22 10:04 profile
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/jdk/Dockerfile 
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2019-11-25
#Blog:             http://www.cnblogs.com/yinzhengjie
#Description：        YinZhengjie's JDK base Dockerfile
#Copyright notice:     original works, no reprint! Otherwise, legal liability will be investigated.
#********************************************************************

#指定咱们自己制作的基础镜像
FROM centos-base:7.6.1810

#指定镜像维护者的信息.
MAINTAINER Jason.Yin y1053419035@qq.com

#安装JDK
ADD jdk-8u231-linux-x64.tar.gz /usr/local/src 

#创建软连接
RUN ln -sv /usr/local/src/jdk1.8.0_231 /usr/local/jdk

#创建环境变量
ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME  ${JAVA_HOME}/jre
ENV CLASSPATH ${JAVA_HOME}/lib/:${JRE_HOME}/lib:${JAVA_HOME}/lib/tools.jar
ENV PATH $PATH:${JAVA_HOME}/bin

#其实除了使用上面的"ENV"指令添加环境变量的情况，咱们还可以使用简单粗暴的方式,即直接将镜像中"/etc/proflie"文件替换
COPY profile /etc/profile
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.编译镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-base         7.6.1810            b4931fd9ace2        2 hours ago         551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cd /yinzhengjie/softwares/dockerfile/web/jdk
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# ll
total 189616
-rw-r--r-- 1 root root       463 Jan 22 08:58 build-command.sh
-rw-r--r-- 1 root root      1089 Jan 22 10:04 Dockerfile
-rw-r--r-- 1 root root 194151339 Jan 19 02:08 jdk-8u231-linux-x64.tar.gz
-rw-r--r-- 1 root root      2109 Jan 22 10:04 profile
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# cat build-command.sh 
#!/bin/bash
#
#********************************************************************
#Author:        yinzhengjie
#QQ:             1053419035
#Date:             2020-01-18
#FileName：        docker-build.sh
#URL:             http://www.cnblogs.com/yinzhengjie
#Description：        Build jdk base Script
#Copyright (C):     2020 All rights reserved
#********************************************************************

docker image build -t jdk-base:1.8.0_231 .
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# bash build-command.sh 
Sending build context to Docker daemon  194.2MB
Step 1/9 : FROM centos-base:7.6.1810
 ---> b4931fd9ace2
Step 2/9 : MAINTAINER Jason.Yin y1053419035@qq.com
 ---> Running in 131aea9f65c6
Removing intermediate container 131aea9f65c6
 ---> 79db0c6b4f1e
Step 3/9 : ADD jdk-8u231-linux-x64.tar.gz /usr/local/src
 ---> d177d749896f
Step 4/9 : RUN ln -sv /usr/local/src/jdk1.8.0_231/bin /usr/local/jdk
 ---> Running in b2609be3353b
'/usr/local/jdk' -> '/usr/local/src/jdk1.8.0_231/bin'
Removing intermediate container b2609be3353b
 ---> 3c7c2d462bd6
Step 5/9 : ENV JAVA_HOME /usr/local/jdk
 ---> Running in 201883f5daa1
Removing intermediate container 201883f5daa1
 ---> adbecca86764
Step 6/9 : ENV JRE_HOME  ${JAVA_HOME}/jre
 ---> Running in 35ae25761426
Removing intermediate container 35ae25761426
 ---> 75e27503d1b1
Step 7/9 : ENV CLASSPATH ${JAVA_HOME}/lib/:${JRE_HOME}/lib:${JAVA_HOME}/lib/tools.jar
 ---> Running in 5c195e73319e
Removing intermediate container 5c195e73319e
 ---> d97a545c2015
Step 8/9 : ENV PATH $PATH:${JAVA_HOME}/bin
 ---> Running in 350f0d825b19
Removing intermediate container 350f0d825b19
 ---> 07947a6eb77f
Step 9/9 : COPY profile /etc/profile
 ---> 6166e8b6bb7c
Successfully built 6166e8b6bb7c
Successfully tagged jdk-base:1.8.0_231
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jdk-base            1.8.0_231           6166e8b6bb7c        43 seconds ago      953MB
centos-base         7.6.1810            b4931fd9ace2        2 hours ago         551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
[root@docker101.yinzhengjie.org.cn /yinzhengjie/softwares/dockerfile/web/jdk]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122094942244-187389091.png)**

**5>.验证JDK镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jdk-base            1.8.0_231           0f63a97ddc85        27 seconds ago      953MB
centos-base         7.6.1810            b4931fd9ace2        2 hours ago         551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm jdk-base:1.8.0_231 bash
[root@f5034f2a8433 /]# 
[root@f5034f2a8433 /]# tail -5 /etc/profile
#Add ${JAVA_HOME} by yinzhengjie
export JAVA_HOME=/usr/local/jdk
export TOMCAT_HOME=/yinzhengjie/softwares/web/tomcat
export PATH=${JAVA_HOME}/bin:${JAVA_HOME}/jre/bin:${TOMCAT_HOME}/bin:$PATH
export CLASSPATH=.${CLASSPATH}:${JAVA_HOME}/lib:${JAVA_HOME}/jre/lib:${JAVA_HOME}/lib/tools.jar
[root@f5034f2a8433 /]# 
[root@f5034f2a8433 /]# java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
[root@f5034f2a8433 /]# 
[root@f5034f2a8433 /]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200122095823849-508883646.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186