不知道各位小伙伴在生产环境都是怎么部署 Spring Boot 的，打成 jar 直接一键运行？打成 war 扔到 Tomcat 容器中运行？不过据松哥了解，容器化部署应该是目前的主流方案。

不同于传统的单体应用，微服务由于服务数量众多，在部署的时候出问题的可能性更大，这个时候，结合 Docker 来部署，就可以很好的解决这个问题，这也是目前使用较多的方案之一。

将 Spring Boot 项目打包到 Docker 容器中部署，有很多不同的方法，今天松哥主要来和大家聊一聊如何将 Spring Boot 项目一键打包到远程 Docker 容器，然后通过运行一个镜像的方式来启动一个 Spring Boot 项目。

至于其他的 Spring Boot 结合 Docker 的用法，大家不要着急，后续的文章，松哥会和大家慢慢的一一道来。

## 1.准备工作

### 1.1 准备 Docker

我这里以 CentOS7 为例来给大家演示。

首先需要在 CentOS7 上安装好 Docker，这个安装方式网上很多，我就不多说了，我自己去年写过一个 Docker 入门教程，大家可以在公众号后台回复 `Docker` 获取教程下载地址。

Docker 安装成功之后，我们首先需要修改 Docker 配置，开启允许远程访问 Docker 的功能，开启方式很简单，修改 `/usr/lib/systemd/system/docker.service` 文件，加入如下内容：

```
-H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock
```

如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlfDvCzoddAS5bYbeKqZhqzMdhhvqiaJbDSbQdichCibwXeiccw2FRDBcTDx4TnlJibYDpSPpaNF18KhZg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

配置完成后，保存退出，然后重启 Docker：

```
systemctl daemon-reloadservice docker restart
```

Docker 重启成功之后，Docker 的准备工作就算是 OK 了。

### 1.2 准备 IDEA

IDEA 上的准备工作，主要是安装一个 Docker 插件，点击 `File->Settings->Plugins->Browse Repositories` 如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlfDvCzoddAS5bYbeKqZhqzWeylxcSEFvqZbhEniaHkM4zsTgran2eR5icibV36KTQ80vkGw7kzaL2LA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

点击右边绿色的 Install 按钮，完成安装，安装完成之后需要重启一下 IDEA。

IDEA 重启成功之后，我们依次打开 `File->Settings->Build,Execution,Deployment->Docker` ，然后配置一下 Docker 的远程连接地址：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlfDvCzoddAS5bYbeKqZhqzQHicjOQxLywGu2XRZKftmDDFntW5lVhQ9KR5EKOMib1icTswF4XQecSKA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

配置一下 Docker 的地址，配置完成后，可以看到下面有一个 Connection successful 提示，这个表示 Docker 已经连接上了。

如此之后，我们的准备工作就算是 OK 了。

## 2.准备项目

接下来我们来创建一个简单的 Spring Boot 项目（只需要引入 `spring-boot-starter-web` 依赖即可），项目创建成功之后，我们再创建一个普通的 `HelloDockerController`，用来做测试，如下：

```
@RestControllerpublic class HelloDockerController {    @GetMapping("/hello")    public String hello() {        return "hello docker!";    }}
```

这是一个很简单的接口，无需多说。

## 3.配置 Dockerfile

接下来，在项目的根目录下，我创建一个 Dockerfile ，作为我镜像的构建文件，具体位置如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/GvtDGKK4uYlfDvCzoddAS5bYbeKqZhqzgdziaWpOyEp4wbzojib667B3jlq7jeMytiaZHb5mQK2T3GDtCico0jJMvQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

文件内容如下：

```
FROM hub.c.163.com/library/java:latestVOLUME /tmpADD target/docker-0.0.1-SNAPSHOT.jar app.jarENTRYPOINT ["java","-jar","/app.jar"]
```

这里只有简单的四行，我说一下：

1.  Spring Boot 项目的运行依赖 Java 环境，所以我自己的镜像基于 Java 镜像来构建。
    
2.  考虑到 Docker 官方镜像下载较慢，我这里使用了网易提供的 Docker 镜像。
    
3.  由于 Spring Boot 运行时需要 tmp 目录，这里数据卷配置一个 /tmp 目录出来。
    
4.  将本地 target 目录中打包好的 .jar 文件复制一份新的 到 /app.jar。
    
5.  最后就是配置一下启动命令，由于我打包的 jar 已经成为 app.jar 了，所以启动命令也是启动 app.jar。
    

这是我们配置的一个简单的 Dockerfile。

## 4.配置 Maven 插件

接下来在 pom.xml 文件中，添加如下插件：

```
<plugin>    <groupId>com.spotify</groupId>    <artifactId>docker-maven-plugin</artifactId>    <version>1.2.0</version>    <executions>        <execution>            <id>build-image</id>            <phase>package</phase>            <goals>                <goal>build</goal>            </goals>        </execution>    </executions>    <configuration>        <dockerHost>http://192.168.66.131:2375</dockerHost>        <imageName>javaboy/${project.artifactId}</imageName>        <imageTags>            <imageTag>${project.version}</imageTag>        </imageTags>        <forceTags>true</forceTags>        <dockerDirectory>${project.basedir}</dockerDirectory>        <resources>            <resource>                <targetPath>/</targetPath>                <directory>${project.build.directory}</directory>                <include>${project.build.finalName}.jar</include>            </resource>        </resources>    </configuration></plugin>
```

这个插件的配置不难理解：

1.  首先在 execution 节点中配置当执行 mvn package 的时候，顺便也执行一下 docker:build
    
2.  然后在 configuration 中分别配置 Docker 的主机地址，镜像的名称，镜像的 tags，其中 dockerDirectory 表示指定 Dockerfile 的位置。
    
3.  最后 resource 节点中再配置一下 jar 的位置和名称即可。
    

OK，做完这些我们就算大功告成了。

## 5.打包运行

接下来对项目进行打包，打包完成后，项目会自动构建成一个镜像，并且上传到 Docker 容器中，打包方式如下：

打包过程会稍微有一点旧，因为还包含了镜像的构建，特别是第一次打包，需要下载基础镜像，会更慢一些。

部分打包日志如下（项目构建过程）:

项目打包成功之后，我们就可以在 Docker 容器中看到我们刚刚打包成的镜像了，如下：

### 5.1 运行方式一

此时，我们可以直接在 Linux 上像创建普通容器一样创建这个镜像的容器，然后启动，执行如下命令即可：

```
docker run -d --name javaboy -p 8080:8080 javaboy/docker:0.0.1
```

启动成功之后，我们就可以访问容器中的接口了。

但是这种操作显然还是有点麻烦，结合我们一开始安装的 Docker 插件，这个运行步骤还可以做进一步的简化。

### 5.2 运行方式二

大家注意，此时我们的 IDEA 中多了一个选项，就是 docker，如下：

点击左边的绿色启动按钮，连接上 Docker 容器，连接成功之后，我们就可以看到目前 Docker 中的所有容器和镜像了，当然也包括我们刚刚创建的 Docker 镜像，如下：

此时，我们选中这个镜像，右键单击，即可基于此镜像创建出一个容器，如下图：

我们选择 Create container，然后填入容器的一些必要信息，配置一下容器名称，镜像 ID 会自动填上，暴露的端口使用 Specify 即可，然后写上端口的映射关系：

配置完成后，点击下方的 `run` 按钮，就可以开始运行了。运行日志如下：

注意，这个日志是在 Docker 的那个窗口里打印出来的。

项目运行成功之后，在浏览器输入远程服务器的地址，就可以访问了：

如此之后，我们的 Spring Boot 项目就算顺利发布到远程 Docker 容器中了。

好玩吗？试试！

本文案例我已经上传到 GitHub，小伙伴们可以参考：https://github.com/lenve/javaboy-code-samples

1、[完结撒花！129 集 21 个小时，松哥自制的 Spring Boot2 系列视频教程杀青啦！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486306&idx=1&sn=a702902990f4cb7fa84c1c23fcba6e92&chksm=e9c35902deb4d0141c7bc42f378542e49b54f2283e2cfe6225542495f3dd65d13c0f1f07fc92&scene=21#wechat_redirect)

2、[40 篇原创干货，带你进入 Spring Boot 殿堂！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486246&idx=1&sn=7aa23f40fa044775e98fccffaa8d43d1&chksm=e9c35946deb4d05063a14258429cc7c21b232018a195d7b4281e2a3cea6500fd9f19ab401780&scene=21#wechat_redirect)

3、[IDEA 神器入坑指南！17 个常用快捷键奉上！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486239&idx=1&sn=d25591a1e7d31653863563f78d209177&chksm=e9c3597fdeb4d0691fc284c372d486365bd5923a88069f1b7250c857aca48a75de3a1def9fe0&scene=21#wechat_redirect)

4、[Spring Boot 中的同一个 Bug，竟然把我坑了两次！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486166&idx=1&sn=ca0528531239410aa4b4a113f10ebc3b&chksm=e9c358b6deb4d1a09f6a0572e829e1fd78f06e7385d08017c0e864ffa32c500f281a6bd9ef04&scene=21#wechat_redirect)

5、[Spring Boot 支持 Https 有那么难吗？](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486160&idx=1&sn=2dd53840370ab6a69aa1e910fbce01d1&chksm=e9c358b0deb4d1a6a6d9b2d47b58da3a4a7483147ef84ba059a47d10d7efbc61bcdeff1e503a&scene=21#wechat_redirect)

6、[一个野生程序员的自我修养](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486150&idx=1&sn=0ffc7a2191058f8aa558c3cf9b062a74&chksm=e9c358a6deb4d1b0a4b16cfeeddd9ae70f483c6494eee55addba648eec757fee371dccb295c8&scene=21#wechat_redirect)

7、[干货 | 鸟瞰 MySQL，唬住面试官！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486126&idx=1&sn=cdfbfc882bdf78ce45147abec317530c&chksm=e9c358cedeb4d1d8390f8481108ca480e0e2e912f7ceceee492733c179a40fc59296f920381a&scene=21#wechat_redirect)

8、[Spring Boot 修改静态资源一定要重启项目才会生效吗？未必！](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486119&idx=1&sn=c2cbf2e7f737fba902f6ebb25dabaefe&chksm=e9c358c7deb4d1d1305abd486554f77af7b35dae40b4f7fb93c109a373f250c57183e5899529&scene=21#wechat_redirect)

9、[Spring Boot 跨域，JSONP 太 low，试试 CORS 怎么样？  
](http://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247486095&idx=1&sn=f446acdf5b26549ccead854b8e0a873f&chksm=e9c358efdeb4d1f9d7b306c25b731fa6baa064bd5dc3051bb386621e0bf74fc969b694588227&scene=21#wechat_redirect)

喜欢就点个"在看"呗^\_^