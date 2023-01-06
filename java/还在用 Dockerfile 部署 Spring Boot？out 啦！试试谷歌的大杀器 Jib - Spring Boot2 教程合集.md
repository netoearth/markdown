之前松哥和大家分享过一篇将 Spring Boot 项目部署到远程 Docker 上的文章：

-   [一键部署 Spring Boot 到远程 Docker 容器](https://mp.weixin.qq.com/s/vSCQLvQBYMYoPhdlO2v3XA)

但是这种部署有一个问题，就是一个小小的 helloworld 构建成镜像之后，竟然都有 660 MB+，这就有点过分了；而且这种方式步骤繁琐，很多人看了头大。

因此松哥今天想再和大家聊一聊另外一种方案 **Jib**，这是谷歌开源的一个容器化运行方案，使用它我们将 Spring Boot 进行容器化部署只要两步：

-   第一步配置 Maven Plugin
-   第二步构建

我们一起来看看。

## Jib

在之前那篇文章中，我们将 Spring Boot 项目进行容器化部署，要求开发人员要有一定的 Docker 技能作为支撑，然而在实际开发中，并非每个人都是 Docker 专家，或者说会用 Docker。

有鉴于此，Google 搞出来一个 Jib，使 Spring Boot 容器化部署变得更加简便，开发人员可以不需要任何 Docker 相关的技能，就能将 Spring Boot 项目构建成 Docker 中的镜像，而且还可以“顺便”将镜像 push 到 register 上，极大的简化了部署过程。

Jib 使用 Java 开发，使用也非常简单，可以作为 Maven 或者 Gradle 的插件直接集成到我们的项目中。它利用镜像分层和注册表缓存来实现快速、增量的构建。Jib 会自动读取项目的构建配置，代码组织到不同的层（依赖项、资源、类）中，然后它只会重新构建和推送发生变更的层。在项目进行快速迭代时，Jib 只将发生变更的层推送到 registers 来缩短构建时间。

好了，大致了解了 Jib 之后，接下来我们来看看 Jib 要怎么使用。

## 准备工作

Jib 可以直接将构建好的镜像 push 到 registers 上，如果公司有自己的私有镜像站的话，可以直接推送到私有镜像站上，本文我就将构建好的镜像推送到官方的 Docker Hub 上，因此需要大家提前准备一个 Docker Hub 的账号，账号大家可以直接去 Docker Hub 上面注册（[https://hub.docker.com/](https://hub.docker.com/)），大家要是对 Docker Hub 这些东西不了解，可以在公众号后台回复 docker，获取松哥自制的 Docker 教程。

## 牛刀小试

首先我们来创建一个 Spring Boot 工程，创建时只需要添加一个 Web 依赖即可：

![](http://www.javaboy.org/images/boot/46-1.png)

项目创建成功后，添加一个 HelloController 用来做测试：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello jib";
    }
}
```

然后，在 pom.xml 中添加上 Jib 的插件，如下：

```
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>1.7.0</version>
    <configuration>
        <from>
            <image>openjdk:alpine</image>
        </from>
        <to>
            <image>docker.io/wongsung/dockerjib</image>
            <tags>
                <tag>v1</tag>
            </tags>
            <auth>
                <username>wongsung</username>
                <password>你的密码</password>
            </auth>
        </to>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

关于这段配置，我说如下几点：

1.  首先就是版本号的问题，我这里使用的是 `1.7.0` ，网上有的教程比较老，用的 0.x 的版本，老的版本在配置 Docker 认证的时候非常麻烦，所以版本这块建议大家使用当前最新版。
2.  from 中的配置表示本镜像构建所基于的根镜像为 `openjdk:alpine`
3.  to 中的配置表示本镜像构建完成后，要发布到哪里去，如果是发布到私有镜像站，就写自己私有镜像站的地址，如果是发布到 Docker Hub 上，就参考我这里的写法 `docker.io/wongsung/dockerjib`，其中 wongsung 表示你在 Docker Hub 上注册的用户名，dockerjib 表示你镜像的名字，可以随意取。
4.  tags 中配置的是自己镜像的版本。
5.  auth 中配置你在 Docker Hub 上的用户名/密码。
6.  executions 节点中的就是常规配置了，我就不再多说了。

配置完成后，在命令行执行如下命令将当前下项目构建成一个 Docker 镜像并 push 到 Docker Hub：

构建完成后，我们在 Docker Hub 上就能看到自己的镜像了：

![](http://www.javaboy.org/images/boot/46-2.png)

接下来，启动 Docker ，在 Docker 中执行如下命令拉取镜像下来并运行：

```
docker run -d --name mydockerjib -p 8080:8080 docker.io/wongsung/dockerjib:v1
```

启动成功后，我们在浏览器中就可以直接访问我们刚才的 Spring Boot 项目中的 hello 接口了：

![](http://www.javaboy.org/images/boot/46-3.png)

是不是很方便？比我第一次给大家介绍的方案要方便很多。

**注意**

这种方式是将项目构建成镜像后并 push 到 registers 上，这种构建方式不需要你本地安装 Docker，如果你需要在本地运行镜像，那当然需要 Docker，单纯的构建是不需要 Docker 环境的。

## 本地构建

如果你电脑本地刚好安装了 Docker ，有 Docker 环境，那么也可以将项目构建成本地 Docker 的镜像，

首先我们来查看一下本地镜像：

![](http://www.javaboy.org/images/boot/46-4.png)

可以看到只有 MySQL 镜像，然后我们执行如下命令构建本地镜像：

```
mvn compile jib:dockerBuild
```

看到如下构建日志信息表示构建成功：

![](http://www.javaboy.org/images/boot/46-5.png)

构建完成后，我们再来看本地镜像：

![](http://www.javaboy.org/images/boot/46-6.png)

可以都看到，已经构建成功了，接下来启动命令和上面一样，我就不重复展示了。

## 结语

容器的出现，让我们的 Java 程序比任何时候都接近“一次编写，到处运行”，Spring Boot 容器化部署也是越来越方便，后面有空松哥再和大家聊聊结合 jenkins 的用法，好了，本文的案例我已经上传到 GitHub：https://github.com/lenve/javaboy-code-samples，有问题欢迎留言讨论。