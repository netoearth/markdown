   

[![](https://p3-passport.byteimg.com/img/user-avatar/d27416d57805e32a2c16b33c82189fb5~100x100.awebp)](https://juejin.cn/user/1679709497471037)

2021年08月10日 16:40 ·  阅读 3035

怎么把`Java`应用打包成Docker镜像？对熟悉`Docker`的同学这应该是一个很简单的问题，把项目打包成`JAR`包然后在`Dockerfile`里用`ADD`命令把`JAR`文件放到镜像里，启动命令设置执行这个`JAR`文件即可。

比如一个使用Maven构建的Spring应用就可以用下面这个`Dockerfile`构建镜像。

```
FROM openjdk:8-jre
ADD target/*.jar /application.jar
ENTRYPOINT ["java", "-jar","/application.jar"]
复制代码
```

上面这个Dockerfile很好理解，使用Maven构建的Java项目的目录结构统一是

```
project
│   pom.xml
└───src // 源文件目录
│   │
│   └───main
│       │   
│       └───java
│       
└───target // class和jar文件的目录
复制代码
```

用`mvn clean package`打包后会把`JAR`文件生成在`target`目录里，通过`java -jar`命令即可执行编译好的程序。

所以上面的Dockerfile里就进行了把`JAR`从`target`目录里添加到Docker镜像中以及将`jar -jar /application.jar` 设置成容器的启动命令这两部操作。

不过除了这种最原始的方法外我们还可以使用`Maven`的一些插件，或者`Docker`的多阶段打包功能来完成把`Java`应用打包成`Docker`镜像的动作。

### Maven插件构建镜像

Spotify公司的`dockerfile-maven-plugin`和Google公司出品的`jib-maven-plugin`是两款比较有名的插件，下面简单介绍一下`dockerfile-maven-plugin`的配置和使用。

其实使用方法很简单，我们在POM文件里引入这个plugin，并结合上面那个Dockerfile就能让插件帮助我们完成应用镜像的打包。

```
<groupId>com.example</groupId>
    <artifactId>hello-spring</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>helloworld</name>
<plugin>
<groupId>com.spotify</groupId>
<artifactId>dockerfile-maven-plugin</artifactId>
<version>1.4.10</version>
<executions>
<execution>
<id>default</id>
      <goals>
        <goal>build</goal>
        <goal>push</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <repository>${docker.registry.url}/${image.prefix}/${artifactId}</repository>
    <tag>${project.version}</tag>
    <buildArgs>
      <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
    </buildArgs>
  </configuration>
</plugin>
复制代码
```

插件里使用的`docker.registry.url`和`image.prefix`是我单独为`Docker`的镜像仓库设置的属性。

```
<properties>
<java.version>1.8</java.version>
  <image.prefix>kevinyan001</image.prefix>
  <docker.registry.url></private.registry.url>
</properties>
复制代码
```

这里可以随意设置成私有仓库的远程地址和镜像前缀，比如在阿里云的镜像服务上创建一个叫`docker-demo`的空间，上面的属性就需要这样配置：

```
<properties>
  <java.version>1.8</java.version>
  <image.prefix>docker-demo</image.prefix>
  <docker.registry.url>registry.cn-beijing.aliyuncs.com</docker.registry.url>
</properties>
复制代码
```

在POM文件里配置好插件后伴随着我们打包应用执行`mvc clean package`操作时`dockerfile-maven-plugin`就会自动根据我们的配置打包好一个叫做`kevinyan001/hello-spring:0.0.1-SNAPSHOT`的Docker镜像。

`dockerfile-maven-plugin`除了能帮助我们打包应用镜像外还可以让它帮助我们把镜像push到远端仓库，不过我觉得用处不大，感兴趣的同学可以去网上搜搜看这部分功能怎么配置。

### 用Docker的多阶段构建打包镜像

上面介绍了使用`Maven`插件帮助我们打包`Java`应用的镜像，其实我们还可以把`mvn clean package`这一步也交给`Docker`来完成。当然把Java应用的源码放在`Docker`镜像里再编译打包在发布出去肯定是有问题的，我们知道在`Dockerfile`里每个指令`ADD`、`RUN`这些都是在单独的层上进行，指令越多会造成镜像越大，而且包含`Java`项目的源码也是一种风险。

不过好在后来`Docker`支持了多阶段构建，允许我们在一个`Dockerfile`里定义多个构建阶段，先拉起一个容器完成用于的构建，比如说我们可以在这个阶段里完成`JAR`的打包，然后第二个阶段重新使用一个`jre`镜像把上阶段打包好的`JAR`文件拷贝到新的镜像里。

> 这一点在Go语言比较有优势，第一阶段编译好的二进制执行文件直接拷贝到一个最基础的`linux`镜像里就能让Go的应用在容器里运行。关于Go应用的多阶段打包，可以查看我以前的文章[线上Go项目的Docker镜像应该怎么构建？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FhoJgL-sP2OtMEFAhYNCItA "https://mp.weixin.qq.com/s/hoJgL-sP2OtMEFAhYNCItA") 进行了解。

使用下面的`Dockerfile`可以通过多阶段构建完成`Java`应用的`Docker`镜像打包。

```
# Dockerfile也可以不放在项目目录下，通过 -f 指定Dockerfile的位置，比如在项目根下执行以下命令 
docker build -t <some tag> -f <dirPath/Dockerfile> .

FROM kevinyan001/aliyun-mvn:0.0.1 AS MAVEN_BUILD

COPY pom.xml /build/
COPY src /build/src

WORKDIR /build/
# mount anonymous host directory as .m2 storage for contianer 
VOLUME /root/.m2

RUN mvn clean package -Dmaven.test.skip=true --quiet

FROM openjdk:8-jre


COPY --from=MAVEN_BUILD /build/target/*.jar /app/application.jar

ENTRYPOINT ["java", "-jar", "/app/application.jar"]

复制代码
```

上面我们用的这些Dockerfile也可以不用放在项目的根目录里，现在已经支持通过 `-f` 指定`Dockerfile`的位置，比如在项目根下执行以下命令完成镜像的打包。

```
docker build -t kevinyan001/hello-spring:0.0.1 -f <dirPath/Dockerfile> .
复制代码
```

上面第一个镜像是我自己做的，因为Maven官方的镜像的远程仓库慢的一批，只能自己包装一下走阿里云的镜像源了。试了试速度也不快，主要是随随便便一个`Spring`项目依赖就太多了。大家如果这块有什么加快`Docker` 构建速度的方法也可以留言一起讨论讨论。

不可否认用多阶段构建打出来的`Go`镜像基本上是10M左右，但是`Spring`的应用随随便便就是上百兆，这个对容器的构建速度、网络传输成本是有影响的，那么`Spring`应用的镜像怎么瘦身呢，这个就留到以后的文章进行探讨了。

> 今天的文章就到这里啦，如果喜欢我的文章就帮我点个赞吧，我会每周通过技术文章分享我的所学所见和第一手实践经验，感谢你的支持。**微信搜索关注公众号--网管叨bi叨**每周教会你一个进阶知识。

文章被收录于专栏：

![cover](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95414745836549ce9143753e2a30facd~tplv-k3u1fbpfcp-no-mark:160:160:160:120.awebp)

开发工程师的Kubernetes学习笔记

以一个后端开发工程师的视角，描述认识和使用Kubernetes的过程。结合通俗易懂的示例，带你一起走进Kubernetes世界的大门。 主要以Kubernetes应用和写代码时怎么利用Kubernetes来提升服务的鲁棒性。

相关课程

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64886f6ba6e94fb982ed744d4873f1a1~tplv-k3u1fbpfcp-zoom-mark-crop-v2:0:0:160:224.awebp?)

SkyWalking：应用监控和链路跟踪

[聆听的车辙 ![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")](https://juejin.cn/user/1134351732184008)   

652购买

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/509056a472bf458591c0389a2a781da1~tplv-k3u1fbpfcp-zoom-mark-crop-v2:0:0:160:224.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 4

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏