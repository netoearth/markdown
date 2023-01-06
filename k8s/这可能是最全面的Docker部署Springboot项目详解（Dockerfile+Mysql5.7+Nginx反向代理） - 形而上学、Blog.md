## 一、前言

参考了网上很多的Docker部署Springboot项目文章，发现很多都写得不够全面，这篇文章详细说明了Docker如何部署Springboot项目（附MySQL5.7 docker部署，以及通过Nginx的反向代理实现IP与域名间的映射），使项目可以通过域名访问。废话不多说，让我们开始吧。

## 二、工具

1.  IDEA（配置好Maven）
2.  一个完整的Springboot项目
3.  Navicat Premium（数据库连接工具，可选其它）
4.  服务器文件传输工具，这里我使用的是winSCP（可选其它）
5.  Xshell（服务器远程连接工具）
6.  Linux CentOS 7.x 服务器

## 三、数据库准备

### 一、安装Docker

可以参考我这篇文章继续Docker的安装  
[CentOS 7.x Docker安装](https://www.upstudy.top/index.php/archives/41/)

### 二、使用Docker部署Mysql（Redis之类的同理）

**1.服务器使用命令，获取mysql镜像**

```
docker pull mysql:5.7
```

[![](https://img-blog.csdnimg.cn/20201206124648775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206124648775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**2.运行MySQL 5.7**

```
docker run -d -p 3300:3306 --name crmMysql-ee MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

参数说明：  
\-d 后台执行  
\-p 3300:3306 将MySQL容器的默认3306端口 映射到 服务器的3300端口上  
\--name 给容器起别名  
\-e 配置环境，并且将密码设置为123456

**3.使用数据库连接工具连接该数据库**  
[![](https://img-blog.csdnimg.cn/20201206130250876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206130250876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**4.导入项目数据库文件**  
[![](https://img-blog.csdnimg.cn/2020120613044557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/2020120613044557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
自此数据库准备完成

**1.将数据库url设置为刚刚使用docker部署的地址、密码**  
[![](https://img-blog.csdnimg.cn/20201206131055905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206131055905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**2.使用Maven工具将项目打包**  
[![](https://img-blog.csdnimg.cn/20201206131345272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206131345272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**3.查看生成的jar包，并且测试能否成功运行**  
生成一个jar包，不同的项目的jar包，名字会不同。  
[![](https://img-blog.csdnimg.cn/20201206143817560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206143817560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
复制到任意文件夹  
[![](https://img-blog.csdnimg.cn/20201206144518497.png "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206144518497.png "点击放大图片")  
文件栏中输入cmd进入当前文件夹终端  
[![](https://img-blog.csdnimg.cn/20201206144553758.png "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206144553758.png "点击放大图片")  
执行命令

```
java -jar crm-0.0.1-SNAPSHOT.jar
```

[![](https://img-blog.csdnimg.cn/20201206144729940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206144729940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
项目成功运行，则说明jar没有问题  
注意：如果在IDEA中也运行着该项目，则会报出端口占用的错误。  
至此项目打包完毕

## 五、上传jar包到Linux服务器中

**1\. 使用Xshell工具连接服务器**  
**2\. 在home目录下，新建一个文件夹，作为jar的上传位置**

```
mkdir /home/springboot
```

**3\. 使用工具上传jar包到该目录下**  
[![](https://img-blog.csdnimg.cn/20201206150023236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206150023236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
至此项目上传完成。

## 六、配置Docker

**1.Linux进入刚刚创建的并且已经有jar包的文件夹**

```
cd /home/springboot
```

**2.新建Dockerfile文件，并且编辑**

```
vim Dockerfile
```

执行后显示  
[![](https://img-blog.csdnimg.cn/20201206150824909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206150824909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**3.键盘输入 i ，进入编辑模式，输入以下信息（根据实际情况修改jar的名字即可）**

```
# 基础镜像
FROM java:8

# 作者信息
MAINTAINER weder

# 将当前目录下的jar包复制到容器里（crm-0.0.1-SNAPSHOT.jar修改为你的jar包名字）
COPY crm-0.0.1-SNAPSHOT.jar  /springboot.jar

# 提示当前项目在容器运行的端口
CMD ["========server.port = 8080=========="]

# 暴露运行的端口
EXPOSE 8080

# 执行jar包
ENTRYPOINT ["java","-jar","/springboot.jar"]
```

ESC退出编辑模式，输入  
:wq  
回车即可

**4.使用命令，查看文件是否写入成功**

```
cat Dockerfile
```

[![](https://img-blog.csdnimg.cn/20201206151730983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206151730983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**5.根据Dockerfile构建容器（名字为springboot），注意最后有个点**

```
docker build -t springboot .
```

[![](https://img-blog.csdnimg.cn/20201206152159537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/20201206152159537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
**6.运行该镜像**

```
docker run -d -p 8081:8080 --name app springboot
```

参数说明  
\-d 后台运行  
\-p 将服务器的8081映射到容器里的8080端口  
\--name app为该容器名，springboot为构建时起的镜像名

至此，在浏览器输入  
**你的IP:8081**  
即可访问这个springboot项目，到这里已经算完成一个springboot项目的部署。

## 七、使用Nginx反向代理为springboot项目配置域名（已申请了域名）

需要Linux已经配置好Nginx。我这里使用的是二级域名，首先需要确认域名是否已经解析（可ping通）  
**1.找到nginx配置文件**

```
locate nginx.conf
```

**2.编辑该配置文件**

添加以下server信息(如果本来也存在server信息也没关系，直接再加一个就行)，作为反向代理

```
server {
            listen       80;
            # 这里填写你的域名
            server_name  xxx.xxx.xxx;
    
            #charset koi8-r;
    
            #access_log  logs/host.access.log  main;
    
            location / {
            # 这里是转发的地址
                proxy_pass   http://127.0.0.1:8081;
                index  index.html index.htm;
            }
        }
```

参考如下  
[![](https://img-blog.csdnimg.cn/2020120616455296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "这可能是最全面的Docker部署Springboot项目详解（Dockerfile+Mysql5.7+Nginx反向代理）")](https://img-blog.csdnimg.cn/2020120616455296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3ODMzMg==,size_16,color_FFFFFF,t_70 "点击放大图片")  
配置完成后,重新加载nginx配置即可。

```
nginx -s reload
```

这样我们就能通过域名访问springboot项目了