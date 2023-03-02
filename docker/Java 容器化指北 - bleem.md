## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%80%E3%80%81%E7%B3%BB%E7%BB%9F%E9%80%89%E6%8B%A9 "一、系统选择")一、系统选择[](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%80%E3%80%81%E7%B3%BB%E7%BB%9F%E9%80%89%E6%8B%A9)

关于最基础的底层镜像, 通常大多数我们只有三种选择: Alpine、Debian、CentOS; 这三者中对于运维最熟悉的一般为 CentOS, 但是很不幸的是 CentOS 后续已经不存在稳定版, 关于它的稳定性问题一直是个谜一样的问题; 这是一个仁者见仁智者见智的问题, 我个人习惯是能不用我绝对不用 😆.

排除掉 CentOS 我们只讨论是 Alpine 还是 Debian; 从镜像体积角度考虑无疑 Alpine 完胜, 但是 Alpine 采用的是 musl 的 C 库, 在某些深度依赖 glibc 的情况下可能会有一定兼容问题. 当然关于**深度依赖 glibc** 究竟有多深度取决于具体应用, 就目前来说我个人也只是遇到过 Alpine 官方源中的 OpneJDK 一些字体相关的 BUG.

综合来说, 我个人的建议是**如果应用深度依赖 glibc, 比如包含一些 JNI 相关的代码, 那么选择 Debian 或者说基于 Debian 的基础镜像是一个比较稳的选择; 如果没有这些重度依赖问题, 那么在考虑镜像体积问题上可以选择使用 Alpine.** 事实上 OpneJDK 本身体积也不小, 即使使用 Alpine 版本, 再安装一些常用软件后也不会小太多, 所以我个人习惯是使用基于 Debian 的基础镜像.

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%BA%8C%E3%80%81JDK-OR-JRE "二、JDK OR JRE")二、JDK OR JRE[](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%BA%8C%E3%80%81JDK-OR-JRE)

大多数人似乎从不区分 JDK 与 JRE, 所以要确定这事情需要先弄明白 JDK 和 JRE 到底是什么:

-   JDK: Java Development Kit
-   JRE: Java Runtime Environment

JDK 是一个开发套件, 它会包含一些调试相关的工具链, 比如 `javac`、`jps`、`jstack`、`jmap` 等命令, 这些都是为了调试和编译 Java 程序所必须的工具, 同时 JDK 作为开发套件是包含 JRE 的; 而 JRE 仅为 Java 运行时环境, 它只包含 Java 程序运行时所必须的一些命令以及依赖类库, 所以 JRE 会比 JDK 体积更小、更轻量.

如果只需要运行 Java 程序比如一个 jar 包, 那么 JRE 足以; 但是如果期望在运行时捕获一些信息进行调试, 那么应该选择 JDK. **我个人的习惯是为了解决一些生产问题, 通常选择直接使用 JDK 作为基础镜像, 避免一些特殊情况还需要挂载 JDK 的工具链进行调试. 当然如果没有这方面需求, 且对镜像体积比较敏感, 那么可以考虑使用 JRE 作为基础镜像.**

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%89%E3%80%81JDK-%E9%80%89%E6%8B%A9 "三、JDK 选择")三、JDK 选择[](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%89%E3%80%81JDK-%E9%80%89%E6%8B%A9)

### [](https://mritd.com/2022/11/08/java-containerization-guide/#3-1%E3%80%81OracleJDK-%E8%BF%98%E6%98%AF-OpenJDK "3.1、OracleJDK 还是 OpenJDK")3.1、OracleJDK 还是 OpenJDK[](https://mritd.com/2022/11/08/java-containerization-guide/#3-1%E3%80%81OracleJDK-%E8%BF%98%E6%98%AF-OpenJDK)

针对于这两者的选择, 取决于一个最直接的问题: 应用代码中是否有使用 Oracle JDK 私有 API.

**通常 “使用这些私有 API” 指的是引入了一些 `com.sun.*` 包下的相关类、接口等,** 这些 API 很多是 Oracle JDK 私有的, 在 OpneJDK 中可能完全不包含或已经变更. 所以如果代码中包含相关调用则只能使用 Oracle JDK.

**值得说明的是很多时候使用这些 API 并不是真正的业务需求, 很可能是开发在导入包时 “手滑” 并且凑巧被导入的 Class 等也能实现对应功能; 对于这种导入是可以被平滑替换的, 比如换成 Apache Commons 相关的实现.** 还有一种情况是开发误导入后及时发现了, 但是没有进行代码格式化和包清理, 这是会在代码头部遗留相关的 `import` 引用, 而 Java 是允许存在这种无用的 `import` 的; 针对这种只需要重新格式化和优化导入即可.

**Tips: IDEA 按 `Option + Command + L`(格式化) 还有 `Control + Option + O`(自动优化包导入).**

### [](https://mritd.com/2022/11/08/java-containerization-guide/#3-2%E3%80%81OracleJDK-%E9%87%8D%E5%BB%BA%E9%97%AE%E9%A2%98 "3.2、OracleJDK 重建问题")3.2、OracleJDK 重建问题[](https://mritd.com/2022/11/08/java-containerization-guide/#3-2%E3%80%81OracleJDK-%E9%87%8D%E5%BB%BA%E9%97%AE%E9%A2%98)

当没有办法必须使用 Oracle JDK 时, 推荐自行下载 Oracle JDK 压缩包并编写 Dockerfile 创建基础镜像. 但是这会涉及到一个核心问题: **Oracle JDK 一般不提供历史版本,** 所以如果要考虑未来的重新构建问题, 建议保留好下载的 Oralce JDK 压缩包.

### [](https://mritd.com/2022/11/08/java-containerization-guide/#3-3%E3%80%81OpenJDK-%E5%8F%91%E8%A1%8C%E7%89%88 "3.3、OpenJDK 发行版")3.3、OpenJDK 发行版[](https://mritd.com/2022/11/08/java-containerization-guide/#3-3%E3%80%81OpenJDK-%E5%8F%91%E8%A1%8C%E7%89%88)

众所周知 OpenJDK 是一个开源发行版, 基于开源协议各大厂商都提供一些增值服务, 同时也预编译了一些 Docker 镜像供我们使用; 目前主流的一些发行版本如下:

-   AdoptOpenJDK
-   Amazon Corretto
-   IBM Semeru Runtime
-   Azul Zulu
-   Liberica JDK

这些发行版很多是大同小异的, 一些发行版可能提供的基础镜像选择更多, 比如 AdoptOpenJDK 提供基于 Alpine、Ubuntu、CentOS 的三种基础镜像发行版; 还有一些发行版提供其他的 JVM 实现, 比如 IBM Semeru Runtime 提供 OpenJ9 JVM 的预编译版本.

目前我个人比较喜欢 AdoptOpenJDK, 因为它是社区驱动的, 由 JUG 成员还有一些厂商等社区成员组成; 而 Amazon Corretto 和 IBM Semeru Runtime 看名字就可以知道是云高端玩家做的, 可用性也比较棒. 其他的类似 Azul Zulu、Liberica JDK 则是一些 JVM 提供厂商, 有些还有点算得上是黑料的东西, 不算特别推荐.

目前 AdoptOpenJDK 已经合并到 Eclipse Foundation, 现在叫做 Eclipse Adoptium; 所以如果想要使用 AdoptOpenJDK 镜像, Docker Hub 中应该使用 [eclipse-temurin](https://hub.docker.com/_/eclipse-temurin) 用户下的相关镜像.

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%9B%9B%E3%80%81JVM-%E9%80%89%E6%8B%A9 "四、JVM 选择")四、JVM 选择[](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%9B%9B%E3%80%81JVM-%E9%80%89%E6%8B%A9)

对于 JVM 实现来说, Oracle 有一个 JVM 实现规范, 这个实现规范定义了兼容 Java 代码运行时的这个 VM 应当具备哪些功能; 所以**只要满足这个 JVM 实现规范且经过了认证, 那么这个 JVM 实现理论上就可以应用于生产.** 目前市面上也有很多 JVM 实现:

-   Hotspot
-   OpenJ9
-   TaobaoVM
-   LiquidVM
-   Azul Zing

这些 JVM 实现可能具有不同的特性和性能, 比如 Hotspot 是最常用的 JVM 实现, 综合性能、兼容性等最佳; 由 IBM 创建目前属于 Eclipse 基金会的 OpneJ9 对容器化更友好, 提供更快启动和内存占用等特性.

**通常建议如果对 JVM 不是很熟悉的情况下, 请使用 “标准的” Hotspot; 如果有更高要求且期望自行调试一些 JVM 优化参数, 请考虑 Eclipse OpenJ9.** 我个人比较喜欢 OpenJ9, 原因是它的文档写的很不错, 只要细心看可以读到很多不错的细节等; 如果要使用 OpenJ9 镜像, 推荐直接使用 [ibm-semeru-runtimes](https://hub.docker.com/_/ibm-semeru-runtimes) 预编译的镜像.

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%BA%94%E3%80%81%E4%BF%A1%E5%8F%B7%E9%87%8F%E4%BC%A0%E9%80%92 "五、信号量传递")五、信号量传递[](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%BA%94%E3%80%81%E4%BF%A1%E5%8F%B7%E9%87%8F%E4%BC%A0%E9%80%92)

当我们需要关闭一个程序时, 通常系统会像该进程发送一个终止信号, 同样在容器停止时 Kubernetes 或者其他容器工具也会像容器内 PID 1 的进程发送终止信号; 如果容器内运行一个 Java 程序, 那么信号传递给 JVM 后 Java 相关的框架比如 Spring Boot 等就会检测到此信号, 然后开始执行一些关闭前的清理工作, **这被称之为 “优雅关闭(Graceful shutdown)”**.

如果在我们容器化 Java 应用时没有正确的让信号传递给 JVM, 那么调度程序比如 Kubernetes 在等待容器关闭超时以后就会进行强制关闭, **这很可能导致一些 Java 程序无法正常释放资源, 比如数据库连接没有关闭、注册中心没有反注册等.** 为了验证这个问题, 我创建了一个 Spring Boot 样例项目来进行测试, 其中项目中包含的核心文件如下(完整代码请看 [GitHub](https://github.com/mritd/SpringBootGracefulShutdownExample)):

-   **BeanTest.java:** 使用 `@PreDestroy` 注册 Hook 来监听关闭事件模拟优雅关闭
-   **Dockerfie.bad:** 错误示范的 Dockerfile
-   **Dockerfile.direct:** 直接运行命令来实现优雅关闭
-   **Dockerfile.exec:** 利用 exec 来实现优雅关闭
-   **Dockerfile.bash-c:** 利用 `bash -c` 来实现优雅关闭
-   **Dockerfile.tini:** 验证 tini 在某些情况下无法实现优雅关闭
-   **Dockerfile.dumb-init:** 验证 dumb-init 在某些情况下无法实现优雅关闭

由于 `BeanTest` 只做打印测试都是通用的, 所以这里直接贴代码:

```
package com.example.springbootgracefulshutdownexample;

import org.springframework.stereotype.Component;

import javax.annotation.PreDestroy;

@Component
public class BeanTest {
    @PreDestroy
    public void destroy() {
        System.out.println("==================================");
        System.out.println("接收到终止信号, 正在执行优雅关闭...");
        System.out.println("==================================");
    }
}
```

### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-1%E3%80%81%E9%94%99%E8%AF%AF%E7%9A%84%E4%BF%A1%E5%8F%B7%E4%BC%A0%E9%80%92 "5.1、错误的信号传递")5.1、错误的信号传递[](https://mritd.com/2022/11/08/java-containerization-guide/#5-1%E3%80%81%E9%94%99%E8%AF%AF%E7%9A%84%E4%BF%A1%E5%8F%B7%E4%BC%A0%E9%80%92)

在很多原始的 Java 项目中通常会存在一个启动运行脚本, 这些脚本可能是自行编写的, 也可能是一些比较老的 Tomcat 启动脚本等; 当我们使用脚本启动并且没有合理的调整 Dockerfile 时就会出现信号无法正确传递的问题; 例如下面的错误示范:

**entrypoint.bad.sh: 负责启动**

```
#!/usr/bin/env bash

java -jar /SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar
```

**Dockerfie.bad: 使用 bash 启动脚本, 这会导致终止信号无法传递**

```
FROM eclipse-temurin:11-jdk

COPY entrypoint.bad.sh /
COPY target/SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar /

# 下面几种种方式都无法转发信号
#CMD /entrypoint.bad.sh
#CMD ["/entrypoint.bad.sh"]
CMD ["bash", "/entrypoint.bad.sh"]
```

通过这个 Dockerfile 打包运行后, **在使用 `docker stop` 命令时明显卡顿一段时间(实际上是 docker 在等待容器内进程自己退出), 当到达预定的超时时间后容器内进程被强行终止, 故没有打印优雅关闭的日志:**

[![](https://cdn.oss.link/markdown/LKOTWO.png)](https://cdn.oss.link/markdown/LKOTWO.png)

### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-2%E3%80%81%E6%AD%A3%E7%A1%AE%E7%9A%84%E4%BF%A1%E5%8F%B7%E4%BC%A0%E9%80%92 "5.2、正确的信号传递")5.2、正确的信号传递[](https://mritd.com/2022/11/08/java-containerization-guide/#5-2%E3%80%81%E6%AD%A3%E7%A1%AE%E7%9A%84%E4%BF%A1%E5%8F%B7%E4%BC%A0%E9%80%92)

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-1%E3%80%81%E7%9B%B4%E6%8E%A5%E8%BF%90%E8%A1%8C%E6%96%B9%E5%BC%8F "5.2.1、直接运行方式")5.2.1、直接运行方式[](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-1%E3%80%81%E7%9B%B4%E6%8E%A5%E8%BF%90%E8%A1%8C%E6%96%B9%E5%BC%8F)

要解决信号传递这个问题其实很简单, 也有很多方法; 比如常见的直接使用 `CMD` 或 `ENTRYPOINT` 指令运行 java 程序:

**Dockerfile.direct: 直接运行 java 程序, 能够正常接受到终止信号**

```
FROM eclipse-temurin:11-jdk

COPY target/SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar /

CMD ["java", "-jar", "/SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar"]
```

可以看到, 在 Dockerfile 中直接运行 java 命令这种方式可以让 jvm 正确的通知应用完成优雅关闭:

[![](https://cdn.oss.link/markdown/zYFRSx.png)](https://cdn.oss.link/markdown/zYFRSx.png)

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-2%E3%80%81%E9%97%B4%E6%8E%A5-Exec-%E6%96%B9%E5%BC%8F "5.2.2、间接 Exec 方式")5.2.2、间接 Exec 方式[](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-2%E3%80%81%E9%97%B4%E6%8E%A5-Exec-%E6%96%B9%E5%BC%8F)

熟悉 Docker 的同学都应该清楚, 在 Dockerfile 里直接运行命令无法解析环境变量; 但是有些时候我们又依赖脚本进行变量解析, 这时候我们可以先在脚本内解析完成, 并采用 `exec` 的方式进行最终执行; 这种方式也可以保证信号传递(不上图了):

**entrypoint.exec.sh: exec 执行最终命令, 可以转发信号**

```
#!/usr/bin/env bash

# 假装进行一些变量处理等操作...
export VERSION="0.0.1"

exec java -jar /SpringBootGracefulShutdownExample-${VERSION}-SNAPSHOT.jar
```

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-3%E3%80%81Bash-c-%E6%96%B9%E5%BC%8F "5.2.3、Bash-c 方式")5.2.3、Bash-c 方式[](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-3%E3%80%81Bash-c-%E6%96%B9%E5%BC%8F)

除了直接执行和 exec 方式其实还有一个我称之为 “不稳定” 的解决方案, 就是使用 `bash -c` 来执行命令; 在使用 `bash -c` 执行一些简单命令时, 其行为会跟 exec 很相似, 也会把子进程命令替换到父进程从而让 `-c` 后的命令直接接受到系统信号; **但需要注意的是, 这种方式不一定百分百成功, 比如当 `-c` 后面的命令中含有管道、重定向等可能仍会触发 `fork`, 这时子命令仍然无法完成优雅关闭.**

**Dockerfile.bash-c: 采用 `bash -c` 执行, 在命令简单情况下可以做到优雅关闭**

```
FROM eclipse-temurin:11-jdk

COPY entrypoint.bad.sh /
COPY target/SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar /

CMD ["bash", "-c", "java -jar /SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar"]
```

关于 `bash -c` 的相关讨论, 可以参考 [StackExchange](https://unix.stackexchange.com/questions/466496/why-is-there-no-apparent-clone-or-fork-in-simple-bash-command-and-how-its-done).

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-4%E3%80%81tini-%E6%88%96-dump-init "5.2.4、tini 或 dump-init")5.2.4、tini 或 dump-init[](https://mritd.com/2022/11/08/java-containerization-guide/#5-2-4%E3%80%81tini-%E6%88%96-dump-init)

> 守护工具并不是万能的, tini 和 dump-init 都有一定问题.

这两个工具是大部分人都熟知的利器, 甚至连 Docker 本身都集成了; 不过似乎很多人都有一个误区(我以前也是这么觉得的), 那就是**认为加了 tini 或者 dump-init 信号就可以转发, 就可以优雅关闭了; 而事实上并不是这样, 很多时候你加了这两个东西也只能保证僵尸进程的回收, 但是子进程仍然可能无法优雅关闭.** 比如下面的例子:

**Dockerfile.tini: 加了 tini 也无法优雅关闭的情况**

```
FROM eclipse-temurin:11-jdk

RUN set -e \
    && apt update \
    && apt install tini psmisc -y

COPY entrypoint.bad.sh /
COPY target/SpringBootGracefulShutdownExample-0.0.1-SNAPSHOT.jar /

ENTRYPOINT ["tini", "-vvv", "--"]

CMD ["bash", "/entrypoint.bad.sh"]
```

对于 dump-init 也有同样的问题, 归根结底这个问题的根本还是在 bash 上: **当使用 bash 启动脚本后, bash 会 fork 一个新的子进程; 而不管是 tini 还是 dump-init 的转发逻辑都是将信号传递到进程组; 只要进程组中的父进程响应了信号, 那么就认为转发完成, 但此时进程组中的子进程可能还没有完成优雅关闭父进程就已经死了, 这会导致变为子进程最终还会被强制 kill 掉.**

### [](https://mritd.com/2022/11/08/java-containerization-guide/#5-3%E3%80%81%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5 "5.3、最佳实践")5.3、最佳实践[](https://mritd.com/2022/11/08/java-containerization-guide/#5-3%E3%80%81%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)

根据上面的测试和验证结果, 这里总结一下最佳实践:

-   1、容器内内置 tini 或者 dump-init 是比较好的做法可以防止僵尸进程
-   2、tini 或者 dump-init 并不能百分百实现优雅关闭
-   3、简单命令直接 CMD 执行可以接受信号转发实现优雅关闭
-   4、复杂命令在脚本内进行 exec 执行也可以接受信号转发实现优雅关闭
-   5、直接使用 `bash -c` 运行在简单命令执行时也可以优雅关闭, 但需要实际测试来确定准确性

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%85%AD%E3%80%81%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6 "六、内存限制")六、内存限制[](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%85%AD%E3%80%81%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6)

> Java 应用的容器化内存限制是一个老生常谈的问题, 国内也有很多资料, 不过这些文章很多都过于老旧或者直接翻译自国外的文章; 我发现很少有人去深究和测试这个问题, 随着这两年容器化的发展其实很多东西早已不适用, 为此在这里决定专门仔细的测试一下这个内存问题(**只想看结论的可直接观看 6.3 章节.**).

众所周知, Java 是有虚拟机的, Java 代码被编译成 Class 文件然后在 JVM 中运行; JVM 默认会根据操作系统环境来自动设置堆内存(HeapSize), 而**容器化 Java 应用面临的挑战其一就是如何让 JVM 获取到正确的可用内存避免被 kill.**

### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1%E3%80%81%E6%97%A0%E9%85%8D%E7%BD%AE%E4%B8%8B%E7%9A%84%E8%87%AA%E9%80%82%E5%BA%94 "6.1、无配置下的自适应")6.1、无配置下的自适应[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1%E3%80%81%E6%97%A0%E9%85%8D%E7%BD%AE%E4%B8%8B%E7%9A%84%E8%87%AA%E9%80%82%E5%BA%94)

在默认不配置时, 理想状态的 JVM 应当能识别到我们对容器施加的内存 limit, 从而自动调整堆内存大小; 为了验证这种理想状态下哪些版本的 OpenJDK 能做到, 我抽取一些特定版本进行了以下测试:

-   使用 `docker run -m 512m ...` 将容器内存限制为 512m, 实际宿主机为 16G
-   使用 `java -XX:+PrintFlagsFinal -version | grep MaxHeapSize` 命令查看 JVM 默认的最大堆内存(后来发现 `-XshowSettings:vm` 看起来更清晰)

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-1%E3%80%81OpenJDK-8u111 "6.1.1、OpenJDK 8u111")6.1.1、OpenJDK 8u111[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-1%E3%80%81OpenJDK-8u111)

这个版本的 OpenJDK 尚未对容器化做任何支持, 所以理论上它是不可能能获取到 limit 的内存限制:

[![](https://cdn.oss.link/markdown/3mReMw.png)](https://cdn.oss.link/markdown/3mReMw.png)

可以看到 JVM 并没有识别到 limit, 仍然按照大约宿主机 1/4 的体量去分配的堆内存, 所以如果里面的 java 应用内存占用高了可能会被直接 kill.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-2%E3%80%81OpenJDK-8u131 "6.1.2、OpenJDK 8u131")6.1.2、OpenJDK 8u131[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-2%E3%80%81OpenJDK-8u131)

选择 8u131 这个版本是因为在此版本添加了 `-XX:+UseCGroupMemoryLimitForHeap` 参数来支持内存自适应, 这里我们先不开启, 先直接进行测试:

[![](https://cdn.oss.link/markdown/9Vhiv0.png)](https://cdn.oss.link/markdown/9Vhiv0.png)

同样在默认情况下是无法识别内存限制的.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-3%E3%80%81OpenJDK-8u222 "6.1.3、OpenJDK 8u222")6.1.3、OpenJDK 8u222[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-3%E3%80%81OpenJDK-8u222)

8u191 版本从 OpneJDK 10 backport 回了 `XX:+UseContainerSupport` 参数来支持 JVM 容器化, 不过该版本暂时无法下载, 这里使用更高的 `8u222` 测试, 测试时同样暂不开启特定参数进行测试:

[![](https://cdn.oss.link/markdown/883BW2.png)](https://cdn.oss.link/markdown/883BW2.png)

同样的内存无法正确识别.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-4%E3%80%81OpenJDK-11-0-15 "6.1.4、OpenJDK 11.0.15")6.1.4、OpenJDK 11.0.15[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-4%E3%80%81OpenJDK-11-0-15)

OpenJDK 11 版本已经开始对容器化的全面支持, 例如 `XX:+UseContainerSupport` 已被默认开启, 所以这里我们仍然选择不去修改任何设置去测试:

[![](https://cdn.oss.link/markdown/kx2saM.png)](https://cdn.oss.link/markdown/kx2saM.png)

可以看到, 即使默认打开了 `UseContainerSupport` 开关, 仍然无法正常的自适应内存.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-5%E3%80%81OpenJDK-11-0-16 "6.1.5、OpenJDK 11.0.16")6.1.5、OpenJDK 11.0.16[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-5%E3%80%81OpenJDK-11-0-16)

可能很多人会好奇, 都测试了 11.0.15 为什么还要测试 11.0.16? 因为这两个版本在不设置的情况下有个奇怪的差异:

[![](https://cdn.oss.link/markdown/5rA9ch.png)](https://cdn.oss.link/markdown/5rA9ch.png)

**可以看到, `11.0.16` 版本在不做任何设置时自动适应了容器内存限制, 堆内存从接近 4G 变为了 120M.**

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-6%E3%80%81OpenJDK-17 "6.1.6、OpenJDK 17")6.1.6、OpenJDK 17[](https://mritd.com/2022/11/08/java-containerization-guide/#6-1-6%E3%80%81OpenJDK-17)

OPneJDK 17 是目前最新的 LTS 版本, 这里再专门测试一下 OpneJDK 17 不调整任何参数时的内存自适应情况:

[![](https://cdn.oss.link/markdown/yY49dp.png)](https://cdn.oss.link/markdown/yY49dp.png)

可以看到 OpneJDK 17 与 OpenJDK 11.0.16 版本一样, 都可以实现内存的自适应.

### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-2%E3%80%81%E6%9C%89%E9%85%8D%E7%BD%AE%E4%B8%8B%E7%9A%84%E8%87%AA%E9%80%82%E5%BA%94 "6.2、有配置下的自适应")6.2、有配置下的自适应[](https://mritd.com/2022/11/08/java-containerization-guide/#6-2%E3%80%81%E6%9C%89%E9%85%8D%E7%BD%AE%E4%B8%8B%E7%9A%84%E8%87%AA%E9%80%82%E5%BA%94)

在上面的无配置情况下我们进行了一些测试, 测试结果从 11.0.15 版本开始出现了一些 “令人费解” 的情况; 理论上 11+ 已经自动打开了容器支持参数, 但是某些版本内存自适应仍然无效, 这促使我对其他参数的实际效果产生了怀疑; 为此我开始按照各个参数的添加版本手动启用这些参数进行了一些测试.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-1%E3%80%81OpenJDK-8u131 "6.2.1、OpenJDK 8u131")6.2.1、OpenJDK 8u131[](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-1%E3%80%81OpenJDK-8u131)

8u131 正式开始进行容器化支持, 在这个版本增加了一个 JVM 选项来告诉 JVM 使用 cgroup 设置的内存限制; 我增加了 `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap` 参数进行测试, 测试结果是这个选项在我当前的环境中似乎完全不生效:

[![](https://cdn.oss.link/markdown/7vX0yq.png)](https://cdn.oss.link/markdown/7vX0yq.png)

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-2%E3%80%81OpenJDK-8u222 "6.2.2、OpenJDK 8u222")6.2.2、OpenJDK 8u222[](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-2%E3%80%81OpenJDK-8u222)

从 8u191 版本开始, 又增加了另一个开启容器化支持的参数 `-XX:+UseContainerSupport`, 该参数从 OpenJDK 10 反向合并而来; 我尝试使用这个参数来进行测试, 结果仍然是没什么卵用:

[![](https://cdn.oss.link/markdown/LuRkwx.png)](https://cdn.oss.link/markdown/LuRkwx.png)

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-3%E3%80%81OpenJDK-11 "6.2.3、OpenJDK 11+")6.2.3、OpenJDK 11+[](https://mritd.com/2022/11/08/java-containerization-guide/#6-2-3%E3%80%81OpenJDK-11)

从 11+ 版本开始 `-XX:+UseContainerSupport` 已经自动开启, 我们不需要再做什么特殊设置, 所以结果是跟无配置测试结果一致的: **从 `11.0.15` 以后的版本开始能够自适应, 之前的版本(包括 `11.0.15`)都不支持自适应.**

### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-3%E3%80%81%E5%88%86%E6%9E%90%E4%B8%8E%E6%80%BB%E7%BB%93 "6.3、分析与总结")6.3、分析与总结[](https://mritd.com/2022/11/08/java-containerization-guide/#6-3%E3%80%81%E5%88%86%E6%9E%90%E4%B8%8E%E6%80%BB%E7%BB%93)

经过上面的一些测试后会发现, 在很多文章或文档中描述的参数出现了莫名其妙不好使的情况; 这主要是因为容器化这两年一个很重要的更新: **Cgroups v2**; 限于篇幅问题这里不在一一罗列测试截图, 下面仅说一下结论.

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-3-1%E3%80%81Cgroups-V1 "6.3.1、Cgroups V1")6.3.1、Cgroups V1[](https://mritd.com/2022/11/08/java-containerization-guide/#6-3-1%E3%80%81Cgroups-V1)

对于使用 `Cgroups V1` 的容器化环境来说, “旧的” 一些规则仍然适用(新内核增加内核参数 `systemd.unified_cgroup_hierarchy=0` 回退到 Cgroups V1):

-   1、OpenJDK 8u131 以及之后版本增加 `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap` 参数支持内存自适应.
-   2、OpenJDK 8u191 以及之后版本增加 `-XX:+UseContainerSupport` 参数支持内存自适应.
-   3、OpenJDK 11 以及之后版本默认开启了 `-XX:+UseContainerSupport` 参数, 自动支持内存自适应

#### [](https://mritd.com/2022/11/08/java-containerization-guide/#6-3-2%E3%80%81Cgroups-V2 "6.3.2、Cgroups V2")6.3.2、Cgroups V2[](https://mritd.com/2022/11/08/java-containerization-guide/#6-3-2%E3%80%81Cgroups-V2)

在新版本系统(具体自行查询)配合较新的 containerd 等容器化工具时, 已经默认转换为 `Cgroups V2`, **需要注意的是针对于 `Cgroups V2` 的内存自适应只有在 OpneJDK 11.0.16 以及之后的版本才支持, 在这之前开启任何参数都没用.**

关于 Cgroups V2 的一些支持细节具体请查看 [JDK-8230305](https://bugs.openjdk.org/browse/JDK-8230305):

[![](https://cdn.oss.link/markdown/v5mjuB.png)](https://cdn.oss.link/markdown/v5mjuB.png)

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%83%E3%80%81DNS-%E7%BC%93%E5%AD%98 "七、DNS 缓存")七、DNS 缓存[](https://mritd.com/2022/11/08/java-containerization-guide/#%E4%B8%83%E3%80%81DNS-%E7%BC%93%E5%AD%98)

在大部分 Java 程序中我们都会使用域名去访问一些服务, 可能是访问某些 API 端点或者是访问一些数据库, 而不论哪样只要使用了域名就会涉及到 DNS 缓存问题; **Java 的 DNS 缓存是由 JVM 控制的, 不要理所当然的以为 JVM DNS 缓存非常友好, 某些时候 DNS 缓存可能超出预期.** 为了测试 DNS 缓存情况我从[某大佬](https://gist.github.com/andystanton/958a9a87f5b5a4eae537f96f896a19bc)这里抄来一个测试脚本, 该脚本会测试三个版本的 OpenJDK DNS 缓存情况:

**jvm-dns-ttl-policy.sh**

```
#!/usr/bin/env bash

set -e

for tag in 8-jdk 11-jdk 17-jdk; do

    tag_name="jvm-dns-ttl-policy"
    output_file="$(mktemp)"

    jvm_args=""
    if ! [ "${tag}" == "8-jdk" ]; then
        jvm_args="--add-exports java.base/sun.net=ALL-UNNAMED"
    fi

    ttl=""
    if ! [ "${1}" == "" ]; then
        ttl="-Dsun.net.inetaddr.ttl=${1}"
    fi

    dockerfile="
FROM        eclipse-temurin:${tag}
WORKDIR     /var/tmp
RUN         printf ' \\
              public class DNSTTLPolicy { \\
                public static void main(String args[]) { \\
                  System.out.printf(\"Implementation DNS TTL for JVM in Docker image based on 'eclipse-temurin:${tag}' is %%d seconds\\\\n\", sun.net.InetAddressCachePolicy.get()); \\
                } \\
              }' >DNSTTLPolicy.java
RUN         javac ${jvm_args} DNSTTLPolicy.java -XDignore.symbol.file
CMD         java ${jvm_args} ${ttl} DNSTTLPolicy
ENTRYPOINT  java ${jvm_args} ${ttl} DNSTTLPolicy
"

    dockerfile_security_manager="
FROM        eclipse-temurin:${tag}
WORKDIR     /var/tmp
RUN         printf ' \\
              public class DNSTTLPolicy { \\
                public static void main(String args[]) { \\
                  System.out.printf(\"Implementation DNS TTL for JVM in Docker image based on 'eclipse-temurin:${tag}' (with security manager enabled) is %%d seconds\\\\n\", sun.net.InetAddressCachePolicy.get()); \\
                } \\
              }' >DNSTTLPolicy.java
RUN         printf ' \\
              grant { \\
                permission java.security.AllPermission; \\
              };' >all-permissions.policy
RUN         javac ${jvm_args} DNSTTLPolicy.java -XDignore.symbol.file
CMD         java ${jvm_args} ${ttl} -Djava.security.manager -Djava.security.policy==all-permissions.policy DNSTTLPolicy
ENTRYPOINT  java ${jvm_args} ${ttl} -Djava.security.manager -Djava.security.policy==all-permissions.policy DNSTTLPolicy
"

    echo "Building Docker image based on eclipse-temurin:${tag} ..." >&2
    docker build -t "${tag_name}" - <<<"${dockerfile}" 2>&1 > /dev/null
    docker run --rm "${tag_name}" &>"${output_file}"
    cat "${output_file}"
    docker build -t "${tag_name}" - <<<"${dockerfile_security_manager}" 2>&1 > /dev/null
    docker run --rm "${tag_name}" &>"${output_file}"
    cat "${output_file}"
    echo ""

done
```

### [](https://mritd.com/2022/11/08/java-containerization-guide/#7-1%E3%80%81%E9%BB%98%E8%AE%A4-DNS-%E7%BC%93%E5%AD%98 "7.1、默认 DNS 缓存")7.1、默认 DNS 缓存[](https://mritd.com/2022/11/08/java-containerization-guide/#7-1%E3%80%81%E9%BB%98%E8%AE%A4-DNS-%E7%BC%93%E5%AD%98)

默认不做任何设置的 DNS 缓存结果如下(直接运行脚本即可):

[![](https://cdn.oss.link/markdown/4tB7jk.png)](https://cdn.oss.link/markdown/4tB7jk.png)

**可以看到, 默认情况下 DNS TTL 被设置为 30s, 如果开启了 `Security Manager` 则变为 -1s, 那么 -1s 什么意思呢(截取自 OpenJDK 11 源码):**

```
/* The Java-level namelookup cache policy for successful lookups:
 *
 * -1: caching forever
 * any positive value: the number of seconds to cache an address for
 *
 * default value is forever (FOREVER), as we let the platform do the
 * caching. For security reasons, this caching is made forever when
 * a security manager is set.
 */
private static volatile int cachePolicy = FOREVER;

/* The Java-level namelookup cache policy for negative lookups:
 *
 * -1: caching forever
 * any positive value: the number of seconds to cache an address for
 *
 * default value is 0. It can be set to some other value for
 * performance reasons.
 */
private static volatile int negativeCachePolicy = NEVER;
```

### [](https://mritd.com/2022/11/08/java-containerization-guide/#7-2%E3%80%81%E8%AE%BE%E7%BD%AE-DNS-%E7%BC%93%E5%AD%98 "7.2、设置 DNS 缓存")7.2、设置 DNS 缓存[](https://mritd.com/2022/11/08/java-containerization-guide/#7-2%E3%80%81%E8%AE%BE%E7%BD%AE-DNS-%E7%BC%93%E5%AD%98)

为了避免这种奇奇怪怪的 DNS 缓存策略问题, 最好我们在启动时通过增加 `-Dsun.net.inetaddr.ttl=xxx` 参数手动设置 DNS 缓存时间:

[![](https://cdn.oss.link/markdown/AWMvSc.png)](https://cdn.oss.link/markdown/AWMvSc.png)

**可以看到, 一但我们手动设置了 DNS 缓存, 那么不论是否开启 `Security Manager` 都会遵循我们的设置.** 如果需要更细致的调试 DNS 缓存推荐使用 Alibaba 开源的 [DCM](https://github.com/alibaba/java-dns-cache-manipulator) 工具.

## [](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%85%AB%E3%80%81Native-%E7%BC%96%E8%AF%91 "八、Native 编译")八、Native 编译[](https://mritd.com/2022/11/08/java-containerization-guide/#%E5%85%AB%E3%80%81Native-%E7%BC%96%E8%AF%91)

Native 编译优化是指通过 GraalVM 将 Java 代码编译为可以直接被平台执行的二进制文件, 编译后的可执行文件运行速度会有极大提升. **但是 GraalVM 需要应用的代码层调整、框架升级等操作, 总体来说比较苛刻; 但是如果是新项目, 最好让开发能支持一下 GraalVM 的 Native 编译, 这对启动速度等有巨大提升.**

上面介绍的用于测试优雅关闭的项目已经内置了 GraalVM 支持, 只需要下载 GraalVM 并设置 `JAVA_HOME` 和 `PATH` 变量, 并使用 `mvn clean package -Dmaven.test.skip=true -Pnative` 编译即可:

[![](https://cdn.oss.link/markdown/69Dkfu.png)](https://cdn.oss.link/markdown/69Dkfu.png)

编译成功后将在 `target` 目录下生成可以直接执行的二进制文件, 以下为启动速度对比测试:

[![](https://cdn.oss.link/markdown/ky6yv3.png)](https://cdn.oss.link/markdown/ky6yv3.png)

**可以看到 GraalVM 编译后启动速度具有碾压级的优势, 基本差出一个数量级; 但是综合来说这种方式目前还不是特别成熟, 迄今为止国内 Java 生态仍是 OpneJDK 8 横行, 老旧项目想要满足 GraalVM 需要调整的地方比较巨大; 所以总结就是新项目能支持尽量支持, 老项目不要作死.**