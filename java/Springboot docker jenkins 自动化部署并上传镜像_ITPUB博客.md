阿里云开源镜像站：  
[https://developer.aliyun.com/mirror/?utm\_content=g\_1000303593](http://blog.itpub.net/70003733/viewspace-2843747/%3D%3D.%2BqsTxp6ATst87ejFfiRA26gnqzaXPzww5t8%2FlfA9DV673EDE7Qex9vyQs6OTx4a%2FTR0LEYWWoYD%2BcxpJYAg62g%3D%3D)

springboot + docker + jenkins自动化部署项目，jenkins、mysql、redis都是docker运行的，并且没有使用虚拟机，就在阿里云服务器（centos7）运行

1、前期准备工作不说了

2、在项目根目录下新建Dockerfile  
![](http://img.blog.itpub.net/blog/2021/11/23/57e30e8cc9b5435e.png?x-oss-process=style/bb)

Dockerfile文件内容为：

```
#基础镜像
FROM openjdk:12
#作者
MAINTAINER demo <demo@qq.com>
VOLUME /tmp
#指定配置文件，以及jar包在服务器上的路径
ENTRYPOINT ["java","-Dspring.profiles.active=prod","-jar","/lcy/work/tools/tools.jar"]
#暴露端口
EXPOSE 8092
```

3、在服务器找个目录新建一个.sh 文件

```
#!/bin/sh
echo '================开始构建镜像=============='
#镜像名称
IMAGE_NAME='registry.cn-beijing.aliyuncs.com/???/tools'
#打包后在jenkins的地址
SOURCE_PATH='/lcy/jenkins/workspace/tools'
#Dockerfile执行jar包的地址
BASE_PATH='/lcy/work/tools'
echo IMAGE_NAME=$IMAGE_NAME
echo '================复制JAR包==================='
echo $SOURCE_PATH/target/tools-0.0.1-SNAPSHOT.jar
cp $SOURCE_PATH/target/tools-0.0.1-SNAPSHOT.jar $BASE_PATH/tools.jar
chmod -R 777 $BASE_PATH/tools.jar
echo '================复制完成===================='
echo '================当前docker版本=============='
docker -v
echo '================构建镜像开始================'
docker build -t $IMAGE_NAME -f $SOURCE_PATH/Dockerfile .
echo '================构建镜像结束================'
#输入要推送镜像的地址，根据镜像仓库提示的地址复制
echo '================推送镜像开始================'
docker login --username=??? --password=??? registry-vpc.cn-beijing.aliyuncs.com
docker push $IMAGE_NAME
echo '================推送镜像结束================'
echo '================获取容器id=================='
CID=$(docker ps | grep "$IMAGE_NAME" | awk '{print $1}')
echo 容器id=$CID
echo '================获取镜像id=================='
IID=$(docker images | grep "$IMAGE_NAME" | awk '{print $3}')
echo 镜像id=$IID
if [ -n "$CID" ]; then
    echo 存在$IMAGE_NAME容器,停止容器并删除
    docker stop tools
    docker rm tools
else
    echo 不存在$IMAGE_NAME容器,开始启动
    docker run -p 8092:8092 -d --name tools -v $BASE_PATH:$BASE_PATH $IMAGE_NAME
fi
```

4、配置jenkins，执行新建的.sh（记得给好权限）其它的配置就不说了，比如github的配置  
![](http://img.blog.itpub.net/blog/2021/11/23/f9462744ccc52d67.png?x-oss-process=style/bb)  
5、运行结果  
![](http://img.blog.itpub.net/blog/2021/11/23/ea4a3de87b38c0e9.png?x-oss-process=style/bb)

来自 “ ITPUB博客 ” ，链接：http://blog.itpub.net/70003733/viewspace-2843747/，如需转载，请注明出处，否则将追究法律责任。