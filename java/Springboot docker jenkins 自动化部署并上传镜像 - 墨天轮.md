本文来自于【阿里云官方镜像站：https://developer.aliyun.com/mirror/?utm\_content=g\_1000307095 】

https://developer.aliyun.com/article/768149?spm=a2c6h.12873581.0.0.70212784gaDzaG

springboot + docker + jenkins自动化部署项目，jenkins、mysql、redis都是docker运行的，并且没有使用虚拟机，就在阿里云服务器（centos7）运行

1、前期准备工作不说了

2、在项目根目录下新建Dockerfile

Dockerfile文件内容为：

#基础镜像FROM openjdk:12#作者MAINTAINER demo <demo@qq.com>

VOLUME /tmp#指定配置文件，以及jar包在服务器上的路径ENTRYPOINT \["java","-Dspring.profiles.active=prod","-jar","/lcy/work/tools/tools.jar"\]#暴露端口EXPOSE 8092

3、在服务器找个目录新建一个.sh文件

#!/bin/shecho '================开始构建镜像=============='#镜像名称

IMAGE\_NAME='registry.cn-beijing.aliyuncs.com/???/tools'#打包后在jenkins的地址

SOURCE\_PATH='/lcy/jenkins/workspace/tools'#Dockerfile执行jar包的地址

BASE\_PATH='/lcy/work/tools'echo IMAGE\_NAME=$IMAGE\_NAMEecho '================复制JAR包==================='echo $SOURCE\_PATH/target/tools-0.0.1-SNAPSHOT.jar

cp $SOURCE\_PATH/target/tools-0.0.1-SNAPSHOT.jar $BASE\_PATH/tools.jar

chmod -R 777 $BASE\_PATH/tools.jar

echo '================复制完成===================='echo '================当前docker版本=============='

docker -v

echo '================构建镜像开始================'

docker build -t $IMAGE\_NAME -f $SOURCE\_PATH/Dockerfile .

echo '================构建镜像结束================'#输入要推送镜像的地址，根据镜像仓库提示的地址复制echo '================推送镜像开始================'

docker login --username=??? --password=??? registry-vpc.cn-beijing.aliyuncs.com

docker push $IMAGE\_NAMEecho '================推送镜像结束================'echo '================获取容器id=================='

CID=$(docker ps | grep "$IMAGE\_NAME" | awk '{print $1}')

echo 容器id=$CIDecho '================获取镜像id=================='

IID=$(docker images | grep "$IMAGE\_NAME" | awk '{print $3}')

echo 镜像id=$IIDif \[ -n "$CID" \]; then

echo 存在$IMAGE\_NAME容器,停止容器并删除

docker stop tools

docker rm tools

else

echo 不存在$IMAGE\_NAME容器,开始启动

docker run -p 8092:8092 -d --name tools -v $BASE\_PATH:$BASE\_PATH $IMAGE\_NAMEfi

4、配置jenkins，执行新建的.sh文件（记得给好权限）其它的配置就不说了，比如github的配置

5、运行结果