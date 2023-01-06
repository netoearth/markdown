今天介绍Spring Boot项目是如何打包、发布的。

Spring Boot使用了内嵌[容器](https://cloud.tencent.com/product/tke?from=10680)，因此它的部署方式也变得非常简单灵活，一方面可以将Spring Boot项目打包成独立的jar或者war包来运行，也可以单独打包成war包部署到Tomcat容器中运行，如果涉及到大规模的部署Jinkins成为最佳选择之一。

## 一、项目打包

现在Maven、Gradle已经成了我们日常开发必不可少的构建工具，使用这些工具很容易地将项目打包成jar或者war包。下面就以Maven项目为例演示Spring Boot项目如何打包发布。

1\. 生成jar包

Maven默认会将项目打成jar包，也可以在pom.xml文件中指定打包方式。配置示例如下：

```
<groupId>com.weiz</groupId>

<artifactId>spring-boot-package</artifactId>

<version>1.0.0</version>

<name>spring-boot-package</name>

<!--指定打包方式-->

<packaging>jar</packaging>
```

上面的示例中，使用packaging标签知道打包方式，版本号为1.0.0。Maven打包会根据pom包中的packaging配置来决定是生成jar包或者war包。

然后，在项目根目录下，在控制台执行如下命令：

```
mvn clean package -Dmaven.test.skip=true
```

（1）mvn clean package其实是两条命令，mvn clean是清除项目target目录下的文件，mvn package打包命令。两个命令可以一起执行。

（2）-Dmaven.test.skip=true：排除测试代码后进行打包。

命令执行完成后，jar包会生成到target目录下，命名一般是“项目名+版本号.jar”的形式。如下图所示。

![](https://ask.qcloudimg.com/http-save/yehe-2935166/f41aa022af92b2097a5d056e68e18804.png?imageView2/2/w/1620)

2\. 生成war包

Spring Boot项目既可以生成war发布，也可以生成jar包发布。那么他们有什么区别呢？

l jar包：通过内置tomcat运行，不需要额外安装tomcat。如需修改内置tomcat的配置，只需要在spring boot的配置文件中配置。内置tomcat没有自己的日志输出，全靠jar包应用输出日志。但是部署简单方便，适合快速部署。

l war包：传统的应用交付方式，需要安装tomcat，然后将war包放到waeapps目录下运行，这样可以灵活选择tomcat版本，也可以直接修改tomcat的配置，同时有自己的tomcat日志输出，可以灵活配置安全策略。相对jar包来说没那么快速方便。

Spring Boot生成war包的方式和生成jar包的方式基本一样。只需要添加一些额外的配置，下面演示生成war包的方式：

步骤1：修改项目中的pom.xml文件

将<packaging>jar</packaging>改为<packaging>war</packaging>。示例代码如下：

```
<groupId>com.weiz</groupId>

<artifactId>spring-boot-package</artifactId>

<version>1.0.0</version>

<name>spring-boot-package</name>

<!--指定打包方式-->

<packaging>war</packaging>
```

上面的示例中，修改packaging标签，将jar包的形式改成了war包的形式，版本号为1.0.0。

步骤2：排除Tomcat

部署war包在Tomcat中运行，并不需要Spring Boot自带的Tomcat组件，所以需要在pom.xml文件中排除自带的Tomcat。示例代码如下：

```
<dependency>

<groupId>org.springframework.boot</groupId>

<artifactId>spring-boot-starter-web</artifactId>

</dependency>

<dependency>

<groupId>org.springframework.boot</groupId>

<artifactId>spring-boot-starter-tomcat</artifactId>

<scope>provided</scope>

</dependency>
```

上面的示例中，将Tomcat组件的scope属性设置为provided，这样在打包产生的war中就不会包含Tomcat相关的jar。

步骤3：注册启动类

在项目的启动类中继承 SpringBootServletInitializer并重写configure( )方法：

```
@SpringBootApplication

public class PackageApplication extends SpringBootServletInitializer {

@Override

protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {

return application.sources(PackageApplication.class);

}

public static void main(String[] args) {

SpringApplication.run(PackageApplication.class, args);

}

}
```

步骤4：生成war包

生成war包的命令与jar包的方式是一样的，具体命令如下：

```
mvn clean package -Dmaven.test.skip=true
```

执行完成后，会在target目录下生成：项目名+版本号.war文件，将打包好的war包复制到Tomcat[服务器](https://cloud.tencent.com/product/cvm?from=10680)中webapps目录下启动即可。

二、运行部署

以往我们开发部Web项目时非常繁琐，而使用Spring Boot开发部署一个命令就能解决，不需要再关注容器的环境问题，专心写业务代码即可。

Spring Boot内嵌的内置Tomcat、Jetty等容器对项目部署带来了更多的改变，在服务器上仅仅需要几条命令即可部署项目。一般开发环境直接java -jar命令启动，正式环境需要将程序部署成服务。下面开始演示Spring Boot项目是如何运行、部署的。

1\. 启动运行

简单就是直接启动jar包。启动jar包命令如下：

```
java -jar spring-boot-package-1.0.0.jar
```

这种方式是前台运行的，只要将控制台关闭，服务就会停止。实际生产中我们肯定不会前台运行，一般使用后台运行的方式来启动。

```
nohup java -jar spring-boot-package-1.0.0.jar &
```

上面的示例中，使用nohup java –jar xxx.jar &命令让程序以后台运行的方式执行。日志会被重定向到nohup.out文件中。也可以用“>filename 2>&1”来更改缺省的重定向文件名。命令如下：

```
nohup java -jar spring-boot-package-1.0.0.jar >spring.log 2>&1 &
```

上面的示例中，使用“>spring.log 2>&1”参数将系统的运行日志保存到spring.log中。

以上就是简单的启动jar包的方式，使用简单。

Spring Boot支持在启动时添加定制，比如设置应用的堆内存、垃圾回收机制、日志路径等。

（1）设置jvm参数

通过设置jvm的参数，优化程序的性能。

```
java -Xms10m -Xmx80m -jar spring-boot-package-1.0.0.jar
```

（2）选择运行环境

在配置多运行环境一节中，介绍了如何配置多运行环境。那么在启动项目时，选择对应的启动环境即可：

```
java -jar spring-boot-package-1.0.0.jar --spring.profiles.active=dev
```

一般项目打包时指定默认的运行环境，在启动运行时也可以再次设置运行的环境。

2\. 生产环境部署

上一节介绍的运行方式比较传统和简单的，实际生产环境中考虑到后期的[运维](https://cloud.tencent.com/solution/operation?from=10680)，建议大家使用服务的方式来部署。

下面通过示例演示Spring Boot项目配置成系统服务。

步骤1：将之前的jar包 spring-boot-package-1.0.0.jar复制到/usr/local/目录下。

步骤2：进入服务文件目录，命令如下：

步骤3：使用vim springbootpackage.service创建服务文件，示例代码如下：

```
[Unit]

Description=springbootpackage

After=syslog.target

[Service]

ExecStart=/usr/java/jdk1.8.0_221-amd64/bin/java -Xmx4096m -Xms4096m -Xmn1536m -jar /usr/local/spring-boot-package-1.0.0.jar

[Install]

WantedBy=multi-user.target
```

上面的示例中，主要是定义服务的名字，以及启动的命令和参数。使用只要修改Description和ExecStart即可。

步骤4：启动服务

```
// 启动服务

systemctl start springbootpackage

// 停止服务

systemctl stop springbootpackage

// 查看服务状态

systemctl status springbootpackage

// 查看服务日志

journalctl -u springbootpackage
```

上面的命令，通过systemctl start|stop|status springbootpackage命令启动、停止创建的springbootpackage服务。

如上图所示就是使用systemctl status springbootpackage命令查看服务状态，同时还可以通过journalctl -u springbootpackage命令查看服务完整日志。

此外，还需要设置服务开机启动，使用如下命令：

```
// 开机启动

systemctl enable springbootpackage
```

以上是打包成独立的jar包部署到服务器。如果是部署到Tomcat中，就按照Tomcat的相关命令来重新启动。

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_265a13d9fb1e)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。