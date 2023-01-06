**简介：** 本文为大家讲解Springboot docker jenkins 自动化部署并上传镜像的过程。

![Springboot docker jenkins 自动化部署并上传镜像_推送](https://s2.51cto.com/images/blog/202112/08223103_61b0c1a789c373175.jpg?x-oss-process=image/format,webp/resize,m_fixed,w_1184)

镜像下载、域名解析、时间同步请点击  [阿里巴巴开源镜像站](https://developer.aliyun.com/mirror)

springboot + docker + jenkins自动化部署项目，jenkins、mysql、redis都是docker运行的，并且没有使用虚拟机，就在阿里云服务器（centos7）运行

1、前期准备工作不说了

2、在项目根目录下新建Dockerfile

![Springboot docker jenkins 自动化部署并上传镜像_spring_02](https://s2.51cto.com/images/blog/202112/08223103_61b0c1a77e6a734947.png?x-oss-process=image/format,webp/resize,m_fixed,w_1184)

Dockerfile文件内容为：

```
#基础镜像FROM openjdk:12#作者MAINTAINER demo <demo@qq.com>VOLUME /tmp#指定配置文件，以及jar包在服务器上的路径ENTRYPOINT ["java","-Dspring.profiles.active=prod","-jar","/lcy/work/tools/tools.jar"]#暴露端口EXPOSE 80921.2.3.4.5.6.7.8.9.
```

3、在服务器找个目录新建一个.sh文件

```
#!/bin/shecho '================开始构建镜像=============='#镜像名称IMAGE_NAME='registry.cn-beijing.aliyuncs.com/???/tools'#打包后在jenkins的地址SOURCE_PATH='/lcy/jenkins/workspace/tools'#Dockerfile执行jar包的地址BASE_PATH='/lcy/work/tools'echo IMAGE_NAME=$IMAGE_NAMEecho '================复制JAR包==================='echo $SOURCE_PATH/target/tools-0.0.1-SNAPSHOT.jarcp $SOURCE_PATH/target/tools-0.0.1-SNAPSHOT.jar $BASE_PATH/tools.jarchmod -R 777 $BASE_PATH/tools.jarecho '================复制完成===================='echo '================当前docker版本=============='docker -vecho '================构建镜像开始================'docker build -t $IMAGE_NAME -f $SOURCE_PATH/Dockerfile .echo '================构建镜像结束================'#输入要推送镜像的地址，根据镜像仓库提示的地址复制echo '================推送镜像开始================'docker login --username=??? --password=??? registry-vpc.cn-beijing.aliyuncs.comdocker push $IMAGE_NAMEecho '================推送镜像结束================'echo '================获取容器id=================='CID=$(docker ps | grep "$IMAGE_NAME" | awk '{print $1}')echo 容器id=$CIDecho '================获取镜像id=================='IID=$(docker images | grep "$IMAGE_NAME" | awk '{print $3}')echo 镜像id=$IIDif [ -n "$CID" ]; then    echo 存在$IMAGE_NAME容器,停止容器并删除    docker stop tools    docker rm toolselse    echo 不存在$IMAGE_NAME容器,开始启动    docker run -p 8092:8092 -d --name tools -v $BASE_PATH:$BASE_PATH $IMAGE_NAMEfi1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.
```

4、配置jenkins，执行新建的.sh文件（记得给好权限）其它的配置就不说了，比如github的配置

![Springboot docker jenkins 自动化部署并上传镜像_spring_03](https://s2.51cto.com/images/blog/202112/08223103_61b0c1a79aad073739.png?x-oss-process=image/format,webp/resize,m_fixed,w_1184)

5、运行结果

![Springboot docker jenkins 自动化部署并上传镜像_jar包_04](https://s2.51cto.com/images/blog/202112/08223103_61b0c1a7e034b69268.png?x-oss-process=image/format,webp/resize,m_fixed,w_1184)

本文转自： [Springboot docker jenkins 自动化部署并上传镜像-阿里云开发者社区](https://developer.aliyun.com/article/768149?spm=a2c6h.12873581.0.0.34342784qJIXxv&groupCode=mirror)

![Springboot docker jenkins 自动化部署并上传镜像_jar包_05](https://s2.51cto.com/images/202112/79114bd7947de973f8255176f8a36a4ce0cf78.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![Springboot docker jenkins 自动化部署并上传镜像_jar包_06](https://s2.51cto.com/images/202112/26dfa6d06d5e5cf92ec4481b394882028f3288.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![Springboot docker jenkins 自动化部署并上传镜像_推送_07](https://s2.51cto.com/images/202112/65595a530ebcb32ddd1623bc78da4f7b629e29.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![Springboot docker jenkins 自动化部署并上传镜像_jar_08](https://s2.51cto.com/images/202112/52881e297884193a1168076fd18178c958939c.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![Springboot docker jenkins 自动化部署并上传镜像_spring_09](https://s2.51cto.com/images/202112/499968053d99c9a37ad345cc47187f5ae51fea.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![Springboot docker jenkins 自动化部署并上传镜像_jar包_10](https://s2.51cto.com/images/202112/5793be99662ec386e6d6294cb4ded35ed388be.gif?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

-   **赞**
-   ****1**收藏**
-   **评论**
-   **举报**

**相关文章**