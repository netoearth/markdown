## [docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)部署jenkins

## 1、拉取[jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020)镜像

```
docker pull jenkins/jenkins
```

## 2、新建文件夹，用来[挂载](https://so.csdn.net/so/search?q=%E6%8C%82%E8%BD%BD&spm=1001.2101.3001.7020)docker目录

```
mkdir -pv /var/jenkins_home
```

添加修改权限(这一步非常重要！这一步非常重要！这一步非常重要！)

```
chown -R 1000 /var/jenkins_home
```

## 3、启动jenkins

```
docker run --name jenkins -p 8088:8080 -p 50000:50000 \
--restart=always -u root \
-v /var/run/docker.sock:/var/run/docker.sock  \
-v $(which docker):/bin/docker \
-v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
-v /var/lib/docker/tmp:/var/lib/docker/tmp \
-v /var/jenkins_home:/var/jenkins_home \
-d jenkins/jenkins
```

**启动命令解析**

```
--restart=always #Docker重启后该容器也为随之重启

-u root          
#以root的身份去运行镜像(避免在容器中调用Docker命令没有权限)
#最好使用docker用户去运行

-v $(which docker):/bin/docker
#将宿主机的docker命令挂载到容器中
#可以使用which docker命令查看具体位置
 
-v /var/run/docker.sock:/var/run/docker.sock
#容器中的进程可以通过它与Docker守护进程进行通信

-v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7
#libltdl.so.7是Docker命令执行所依赖的函数库
#容器中library的默认目录是 /usr/lib/x86_64-linux-gnu/
#把宿主机的libltdl.so.7 函数库挂载到该目录即可
#可以通过whereis libltdl.so.7命令查看具体位置
#centos7位置/usr/lib64/libltdl.so.7
#ubuntu位置/usr/lib/x86_64-linux-gnu/libltdl.so.7

-v /var/jenkins_home:/var/jenkins_home
#将运行的docker镜像目录挂载到本地的/var/jenkins_home

```

## 3、进入jenkins容器查看密码

```
#第一步：进入容器
docker exec -it jenkins /bin/bash
#第二步：查看能否使用宿主机的docker命令(如果命令运行成功切显示版本信息则成功)
docker -v
#第三步：查看密码
cat /var/jenkins_home/secrets/initialAdminPassword


```

## 4、安装插件

**Maven Integration**

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9354a3f5d1640fd970a1b2412c73a43.png)

**GitLab**

![在这里插入图片描述](https://img-blog.csdnimg.cn/34bb13e33f2045359fb64c685a63cdbf.png)

**ssh**

![在这里插入图片描述](https://img-blog.csdnimg.cn/f7872cf42b8a4964b1478899246823e5.png)

## 5、配置全局变量

![在这里插入图片描述](https://img-blog.csdnimg.cn/bde35373791348ac919ae0b372a0535e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 6、配置全局工具

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ed6dc0164ba48f4aa941aff84cc3a61.png)

### 6.1、配置jdk

**这里采用自动安装,因为jenkins自带的jdk版本是11的，我用jdk8**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/9689a3dd9f534d23bbb02f8c80d2302d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

这里需要账号，可以在网上自己找一个，我找的这里就不贴上去了

### 6.2、配置git路径

**在容器内继续输入which git，并配置git路径**

```
which git
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/edeabcb500de49a3860d87de49e83b77.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 6.3、配置maven

![在这里插入图片描述](https://img-blog.csdnimg.cn/e366c538ff7b4fcd944d4905d577eb89.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 7、自动构建springBoot项目

## **注意注意！！！！这个一定要注意,必须在你创建的springBoot项目里面配置docker构建插件！不然后面脚本运行的docker build命令是没法把镜像上传到指定服务器的！！！**

如下图  
![在这里插入图片描述](https://img-blog.csdnimg.cn/41b66097425747edbff916bdf7b99176.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

现在可以在jenkins新建项目了  
![在这里插入图片描述](https://img-blog.csdnimg.cn/e950f47778c345bcb31127b61c8b4229.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 7.1、配置Git仓库拉代码构建

**1、选择maven**

![在这里插入图片描述](https://img-blog.csdnimg.cn/421f5c9c603e4853a96bbdb648f2518b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

**2、确认之后配置git仓库**

![在这里插入图片描述](https://img-blog.csdnimg.cn/10e39a82c7734a4385c3f8adebeeb5e2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

**3、配置打包命令**

![在这里插入图片描述](https://img-blog.csdnimg.cn/32ecdbed9f45471eac403885ae5be369.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

**4、脚本**

#### 1）部署到jenkins所在的服务器

**添加构建完成之后执行的shell命令**

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-YpiX5hUD-1634895185339)(docker部署jenkins.assets/image-20211015140838800.png)]](https://img-blog.csdnimg.cn/3f8c2b4229a34078b81c290218dc7043.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

##### **脚本1**

（这个是git里面没有配置dockerFile的，脚本自动配置dockerfile）

```
#!/bin/bash
#1.2.3.4需要根据不同项目修改不同值
#1.服务名称
SERVER_NAME=tool-test

#2.启动dev配置文件
CONFIG_NAME=dev

#3.源jar路径,mvn打包完成之后，target目录下的jar包或jar包名称(包含后缀)
JAR_NAME=tools.jar

#4.端口
PORT=9002

# 源jar路径  
#/usr/local/jenkins_home/workspace--->jenkins 工作目录
#demo 项目目录
#target 打包生成jar包的目录
JAR_PATH=/var/jenkins_home/workspace/$SERVER_NAME/target

#创建createImages和demo文件夹
mkdir -pv /var/jenkins_home/createImages/$SERVER_NAME

#在demo文件夹下创建Dockerfile文本文件
touch /var/jenkins_home/createImages/$SERVER_NAME/Dockerfile

#写入Dockerfile
cat >>/var/jenkins_home/createImages/$SERVER_NAME/Dockerfile<<EOF
FROM java:8
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ADD $JAR_NAME $SERVER_NAME
ENTRYPOINT ["java","-jar","$SERVER_NAME","--spring.profiles.active=$CONFIG_NAME"]
EOF

# 打包完成之后，把jar包移动到创建Docker镜像的目录（此目录包含Dockerfile文件）--->createImages,demo这两个目录需要自己提前创建(上面已经提前创建了)

JAR_WORK_PATH=/var/jenkins_home/createImages/$SERVER_NAME

echo "清理$SERVER_NAME的容器"

containerId=$(docker ps -a | grep -w  tools | awk '{print $1}')
if [ -n "$containerId" ];
then 
docker stop $containerId
docker rm $containerId
echo "成功停止、删除容器"
fi
imageld=$(docker images | grep -w tools | awk '{print $3}')
if [ -n "$imageld" ];
then
docker rmi -f  $imageld
echo "成功删除镜像"
fi

echo "停止并清除镜像完成,进入创建新镜像过程"

echo "复制jar包到/createImages/demo目录下"

#复制jar包到/createImages/demo目录下
cp $JAR_PATH/$JAR_NAME $JAR_WORK_PATH

#切换到demo创建镜像文件夹目录下
cd $JAR_WORK_PATH

#修改文件权限
chmod 755 $JAR_NAME

echo "执行命令创建新镜像"

#执行docker创建镜像命令
docker build -t $SERVER_NAME .

#删除Dockerfile文本文件
rm /var/jenkins_home/createImages/$SERVER_NAME/Dockerfile

#运行镜像
docker run -d -p $PORT:$PORT --restart=always -u root --name $SERVER_NAME $SERVER_NAME

echo "运行新镜像,流程完毕,端口号为：$PORT"


```

#### 2）部署到jenkins不同的服务器

##### ①配置用户凭证

**依次点击以下视图内容**

![在这里插入图片描述](https://img-blog.csdnimg.cn/54708a4f88d74bd1ade888bd8cfe98ec.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/65944623464a480d86e2e566b0943ea1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ecd1db2674c422194d34f286affad0f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

##### **②编辑脚本**

（这里一共有3块命令，都在脚本2里面）

![在这里插入图片描述](https://img-blog.csdnimg.cn/0ba27c98d2c344689dacd65bb36f3bd5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)

##### 脚本2

```
#远程服务器执行
containerId=$(docker ps -a | grep -w  tools | awk '{print $1}')
if [ -n "$containerId" ];
then 
docker stop $containerId
docker rm $containerId
echo "成功停止、删除容器"
fi
imageld=$(docker images | grep -w tools | awk '{print $3}')
if [ -n "$imageld" ];
then
docker rmi -f  $imageld
echo "成功删除镜像"
fi

```

```
#打包并构建docker镜像
mvn clean package docker:build
```

```
#  远程服务器执行
docker run -d -p 9002:9002 --restart=always -u root --name tools tools
```

## 8、启动

**构建的时候只需要点击项目名称->立即构建就可以自动运行之前配置的脚本了(也可以直接点击右边的绿色三角图标构建,后面再把前端VUE的构建脚本贴上)**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/223dd43445f84a13ab13295c67f38ad6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAeOe6oOe7k3g=,size_20,color_FFFFFF,t_70,g_se,x_16)