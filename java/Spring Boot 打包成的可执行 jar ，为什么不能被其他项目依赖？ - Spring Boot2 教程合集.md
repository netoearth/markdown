前两天被人问到这样一个问题:

“松哥，为什么我的 Spring Boot 项目打包成的 jar ，被其他项目依赖之后，总是报找不到类的错误？”

大伙有这样的疑问，就是因为还没搞清楚可执行 jar 和普通 jar 到底有什么区别？今天松哥就和大家来聊一聊这个问题。

## 多了一个插件

Spring Boot 中默认打包成的 jar 叫做 可执行 jar，这种 jar 不同于普通的 jar，普通的 jar 不可以通过 `java -jar xxx.jar` 命令执行，普通的 `jar` 主要是被其他应用依赖，`Spring Boot` 打成的 `jar` 可以执行，但是不可以被其他的应用所依赖，即使强制依赖，也无法获取里边的类。但是可执行 jar 并不是 Spring Boot 独有的，Java 工程本身就可以打包成可执行 jar 。

有的小伙伴可能就有疑问了，既然同样是执行 `mvn package` 命令进行项目打包，为什么 Spring Boot 项目就打成了可执行 jar ，而普通项目则打包成了不可执行 jar 呢？

这我们就不得不提 Spring Boot 项目中一个默认的插件配置 `spring-boot-maven-plugin` ，这个打包插件存在 5 个方面的功能，从插件命令就可以看出：

![](http://www.javaboy.org/images/boot/23-1.png)

五个功能分别是：

-   build-info：生成项目的构建信息文件 build-info.properties
-   repackage：这个是默认 goal，在 `mvn package` 执行之后，这个命令再次打包生成可执行的 jar，同时将 `mvn package` 生成的 jar 重命名为 `*.origin`
-   run：这个可以用来运行 Spring Boot 应用
-   start：这个在 `mvn integration-test` 阶段，进行 `Spring Boot` 应用生命周期的管理
-   stop：这个在 `mvn integration-test` 阶段，进行 `Spring Boot` 应用生命周期的管理

这里功能，默认情况下使用就是 repackage 功能，其他功能要使用，则需要开发者显式配置。

## 打包

repackage 功能的 作用，就是在打包的时候，多做一点额外的事情：

1.  首先 `mvn package` 命令 对项目进行打包，打成一个 `jar`，这个 `jar` 就是一个普通的 `jar`，可以被其他项目依赖，但是不可以被执行
2.  `repackage` 命令，对第一步 打包成的 `jar` 进行再次打包，将之打成一个 可执行 `jar` ，通过将第一步打成的 `jar` 重命名为 `*.original` 文件

举个例子：

对任意一个 Spring Boot 项目进行打包，可以执行 `mvn package` 命令，也可以直接在 `IDEA` 中点击 `package` ，如下 ：

![](http://www.javaboy.org/images/boot/23-2.png)

打包成功之后， `target` 中的文件如下：

![](http://www.javaboy.org/images/boot/23-3.png)

这里有两个文件，第一个 `restful-0.0.1-SNAPSHOT.jar` 表示打包成的可执行 `jar` ，第二个 `restful-0.0.1-SNAPSHOT.jar.original` 则是在打包过程中 ，被重命名的 `jar`，这是一个不可执行 `jar`，但是可以被其他项目依赖的 `jar`。通过对这两个文件的解压，我们可以看出这两者之间的差异。

## 两种 jar 的比较

可执行 `jar` 解压之后，目录如下：

![](http://www.javaboy.org/images/boot/23-4.png)

可以看到，可执行 jar 中，我们自己的代码是存在 于 `BOOT-INF/classes/` 目录下，另外，还有一个 `META-INF` 的目录，该目录下有一个 `MANIFEST.MF` 文件，打开该文件，内容如下：

```
Manifest-Version: 1.0
Implementation-Title: restful
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: org.javaboy.restful.RestfulApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.6.RELEASE
Created-By: Maven Archiver 3.4.0
Main-Class: org.springframework.boot.loader.JarLauncher
```

可以看到，这里定义了一个 `Start-Class`，这就是可执行 `jar` 的入口类，`Spring-Boot-Classes` 表示我们自己代码编译后的位置，`Spring-Boot-Lib` 则表示项目依赖的 `jar` 的位置。

换句话说，如果自己要打一个可执行 `jar` 包的话，除了添加相关依赖之外，还需要配置 `META-INF/MANIFEST.MF` 文件。

这是可执行 jar 的结构，那么不可执行 jar 的结构呢？

我们首先将默认的后缀 `.original` 除去，然后给文件重命名，重命名完成，进行解压：

![](http://www.javaboy.org/images/boot/23-5.png)

解压后可以看到，不可执行 `jar` 根目录就相当于我们的 `classpath`，解压之后，直接就能看到我们的代码，它也有 `META-INF/MANIFEST.MF` 文件，但是文件中没有定义启动类等。

```
Manifest-Version: 1.0
Implementation-Title: restful
Implementation-Version: 0.0.1-SNAPSHOT
Build-Jdk-Spec: 1.8
Created-By: Maven Archiver 3.4.0
```

**注意**

这个不可以执行 `jar` 也没有将项目的依赖打包进来。

从这里我们就可以看出，两个 `jar` ，虽然都是 `jar` 包，但是内部结构是完全不同的，因此一个可以直接执行，另一个则可以被其他项目依赖。

## 一次打包两个 jar

一般来说，Spring Boot 直接打包成可执行 `jar` 就可以了，不建议将 Spring Boot 作为普通的 `jar` 被其他的项目所依赖。如果有这种需求，建议将被依赖的部分，单独抽出来做一个普通的 `Maven` 项目，然后在 Spring Boot 中引用这个 `Maven` 项目。

如果非要将 Spring Boot 打包成一个普通 `jar` 被其他项目依赖，技术上来说，也是可以的，给 `spring-boot-maven-plugin` 插件添加如下配置：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build>
```

配置的 `classifier` 表示可执行 `jar` 的名字，配置了这个之后，在插件执行 `repackage` 命令时，就不会给 `mvn package` 所打成的 `jar` 重命名了，所以，打包后的 jar 如下：

![](http://www.javaboy.org/images/boot/23-6.png)

第一个 jar 表示可以被其他项目依赖的 jar ，第二个 jar 则表示一个可执行 jar。

好了，关于 Spring Boot 中 jar 的问题，我们就说这么多，有问题欢迎留言讨论。