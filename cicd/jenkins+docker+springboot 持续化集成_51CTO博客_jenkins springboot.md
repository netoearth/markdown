©著作权归作者所有：来自51CTO博客作者小时候的阳光的原创作品，请联系作者获取转载授权，否则将追究法律责任

## 1\. 安装 Jenkins + maven + jdk + git

## 2\. spring boot 项目添加Dockerfile

![jenkins+docker+springboot 持续化集成_jenkins](https://s2.51cto.com/images/blog/202109/22/441decf05d7ed2e162c138db5c9cd9f9.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

这里没有使用dockermaven插件，直接原生 Dockerfile，保持代码侵入最低。

```
FROM 172.16.0.57:5000/openjdk:8-jdk-alpineMAINTAINER guzhongtao@middol.comVOLUME /var/log/tiannake/hibernateRUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtimeRUN echo 'Asia/Shanghai' >/etc/timezoneENV LANG zh_CN.UTF-8ADD target/hibernate.jar app.jarENTRYPOINT ["java","-Xms1024m","-Xmx1024m","-Dspring.profiles.active=test","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]1.2.3.4.5.6.7.8.9.10.
```

FROM 172.16.0.57:5000/openjdk:8-jdk-alpine 这个是我的docker私服里面的基础镜像，去掉 172.16.0.57:5000则从 dockerhuber上下载该基础镜像。

VOLUME /var/log/tiannake/hibernate 这个是我springboot 日志打印的路径。

ADD target/hibernate.jar app.jar 这个我项目编译后统一路径在 target下面，编译的finalName是hibernate.jar

## 3\. 配置 Jenkins 全局系统工具

![jenkins+docker+springboot 持续化集成_springboot_02](https://s2.51cto.com/images/blog/202109/22/09d6bb170d37a1270dc09873485c77e4.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

![jenkins+docker+springboot 持续化集成_springboot_03](https://s2.51cto.com/images/blog/202109/22/f8ec46c40840290d22fa013a509e395d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

## 4\. 配置 Jenkins 新建一个maven项目

![jenkins+docker+springboot 持续化集成_Dockerfile_04](https://s2.51cto.com/images/blog/202109/22/23009c707b6b9da395f28c8d3e517c85.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

![jenkins+docker+springboot 持续化集成_springboot_05](https://s2.51cto.com/images/blog/202109/22/334b04c4d30b9f5bb4bb437f719e4781.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

![jenkins+docker+springboot 持续化集成_springboot_06](https://s2.51cto.com/images/blog/202109/22/a9a8206d30be02f802cf618224eaa568.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_750 "在这里插入图片描述")

最重要的是编译后的shell脚本

先删除容器，镜像后创建镜像，启动容器。

```
#!/bin/bash export BUILD_ID=dontKillMeecho "动态参数配置 begin===============>"CDNAME=hibernate-2.0echo "image and container name is $CDNAME"BootPort=8101echo "the spring boot port is $BootPort"dockerLogVolume=/var/log/tiannake/hibernateecho "the dockerLogVolume is $dockerLogVolume"linuxLogPath=/log/docker/tannaikeecho "the linuxLogPath is $linuxLogPath"echo "动态参数配置 end"echo "begin create the container..."echo "stop and delete container"CID=$(docker ps -a | grep "$CDNAME" | awk '{print $1}')if [ -n "$CID" ]; thenecho "has container，CID=$CID"docker stop $CIDdocker rm $CIDfiecho "delete image"IMD=$(docker image ls | grep "$CDNAME" | awk '{print $1}')if [ -n "$IMD" ]; thenecho "has image，IMD=$IMD"docker rmi $IMDfiecho "pwd..."pwdecho "build docker image"docker build -f Dockerfile -t $CDNAME:latest .echo "current docker images："docker images | grep $CDNAMEecho "start container ===============> "docker run -d -p $BootPort:$BootPort -v $linuxLogPath:$dockerLogVolume --name $CDNAME $CDNAME:latestCID=$(docker ps | grep "$CDNAME" | awk '{print $1}')if [ -n "$CID" ]; thenecho "============ start container，CID=$CID ============"echo "============ start container SUCCESS！============"elseecho "============ start container FAIL ！！！！！============"fi1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.45.46.47.48.49.50.51.52.53.54.55.56.57.58.59.60.61.62.63.64.65.66.67.
```

-   **赞**
-   **收藏**
-   **评论**
-   **举报**