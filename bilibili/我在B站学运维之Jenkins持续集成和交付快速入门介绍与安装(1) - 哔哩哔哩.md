**本章目录:**

-   0x00 前言简述
    

-   Jenkins 介绍
    
-   Jenkins 特性
    
-   Jenkins 应用场景
    
-   Jenkins Job 类型
    

-   0x01 安装配置
    

-   Ubuntu
    
-   Docker 安装
    
-   Kubernetes 安装
    

-   0x02 基础知识
    

-   Jenkins 环境变量
    
-   Jenkins 参数构建类型
    
-   Jenkins 并发构建
    
-   Jenkins 自动化构建
    
-   Jenkins 上下游构建
    
-   Jenkins 调用API
    

-   0x03 基础配置
    

-   视图管理
    
-   插件管理
    
-   权限管理
    
-   构建工具管理
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

Q: 什么是Jenkins?

> 答: Jenkins 是一个开源持续集成的工具（`CI&CD`）由Java开发而成, 用于自动化各种任务，包括构建、测试和部署软件(`自动化服务器`); Jenkins 支持各种运行方式，可通过系统包、Docker 或者 通过一个独立的Java程序;  
> 官方介绍 : 全球领先的开源自动化服务器,Jenkins 提供了数以百计的插件来支持构建、部署和自动化任何项目  
> 官方标语 : “Build great things at any scale”-“建造伟大的事情以任何规模”

Tips ：个人理解 Jenkins 是一个调度平台，本身不需要处理任何事情，而是通过众多的插件来完成所有的工作;

**Q: 为什么要用Jenkins?**

> 答: Jenkins的前身是Hudson, 是基于Java开发的一种持续集成工具，用于监控秩序重复的工作, 它是可以将各个开源的软件进行集成的调度平台，例如( `Gitlab/SVN 、Maven、Sonarqube、Shell、钉钉通知、项目监控` )等；

**Jenkins 发行线版本说明:**

-   TLS 长期支持版本: 每12周从常规版本流中选择，作为该时间段的稳定版本。 每隔 4 周，我们会发布稳定版本，其中包括错误和安全修复反向移植。
    
-   每周更新版本: 每周都会发布一个新版本，为用户和插件开发人员提供错误修复和功能。
    

-   开源的java语言开发持续集成工具，支持CI，CD；
    
-   易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理；
    
-   消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告；
    
-   分布式构建：支持Jenkins能够让多台计算机一起构建/测试；
    
-   文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等；
    
-   丰富的插件支持:支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，docker
    

-   1.创建一个项目，手动构建，完成简单任务，比如拉取代码进行编译（持续集成）。
    
-   2.编译失败通知用户
    
-   3.参数化构建
    
-   4.代码改动自动触发构建或者定时触发构建
    
-   5.一个项目构建完成后自动调用另一个项目的构建，完成一连串任务
    
-   6.并发构建
    
-   7.集群化部署开发（CI/CD）
    

-   1) Freestyle project : 自由风格项目，主要的项目类型
    
-   2) Maven project : maven项目专有，类似freestyle，更简单
    
-   3) Multiconfigration project : 多配置项目，适合大量不同配置（环境、平台等）构建
    
-   4) Pipeline : 流水线项目，适合使用pipeline 插件功能构建流水线任务，或者使用freestyle project不容易实现的负责任务
    
-   5) Multibranch pipeline : 多分支流水线项目，根据SCM仓库中的分支创建多个pipeline项目
    

**安装方式**  
安装参考: https://www.jenkins.io/zh/doc/book/installing/

-   Windows（Jar 、War）、Linux（yum|rpm 、apt|dpkg）、Mac
    
-   Docker
    

PS : Jenkins通常作为一个独立的应用程序在其自己的流程中运行， 内置Java servlet 容器/应用程序服务器（Jetty）。

**系统要求**  
最低推荐配置:

-   256MB 可用内存
    
-   1GB 可用磁盘空间(作为一个Docker容器运行jenkins的话推荐10GB)
    

小团队推荐的硬件配置:

-   1 GB+ 可用内存
    
-   50 GB+ 可用磁盘空间
    

软件配置: Java 8—无论是Java运行时环境（JRE）还是Java开发工具包（JDK）都可以

Tips : 安装最低配置：不少于256M内存，不低于1G磁盘，JDK版本>=8（openjdk也可以）。

PS : Jenkins 依赖于 Java 环境, 如果是在不使用Docker安装Jenkins时就需要安装配置Java环境;

```
# Jenkins 版本与 Java版本依赖关系
2.164 (2019-02) and newer: Java 8 or Java 11
2.54 (2017-04) and newer: Java 8
1.612 (2015-05) and newer: Java 7
```

Jenkins的`Debian Package Repository`，以自动化安装和升级。要使用此存储库，请先将键添加到系统：  
Jenkins Debian Packages：https://pkg.jenkins.io/debian-stable/

官方安装:

```
# 添加  gpg key
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

# 在 /etc/apt/sources.list 中添加以下条目 deb https://pkg.jenkins.io/debian-stable binary/
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
# 更新您的本地包索引，然后最终安装Jenkins：
sudo apt-get update
sudo apt-get install jenkins
```

国内安装:

```
# Java 环境安装配置 jdk-8u211-linux-x64.tar.gz
# 下载地址: https://www.oracle.com/cn/java/technologies/javase-downloads.html
sudo mkdir /opt/app/ && cd $_
tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/

# 环境变量设置（临时生效） set oracle jdk environment
ln -s /usr/local/jdk1.8.0_211/bin/java /usr/bin/java
export JAVA_HOME=/usr/local/jdk1.8.0_211 ## 这里要注意目录要换成自己解压的jdk 目录
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

# 环境安装测试
root@jenkins:/usr/local/jdk1.8.0_211# java -version
  # java version "1.8.0_211"
  # Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
  # Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)


# Jenkins 下载& 安装
sudo wget https://mirrors.aliyun.com/jenkins/debian-stable/jenkins_2.263.1_all.deb
sudo dpkg -i jenkins_2.263.1_all.deb

# 更新站点配置
sed -i "s#updates.jenkins.io#mirrors.tuna.tsinghua.edu.cn/jenkins/updates/#g" /var/lib/jenkins/hudson.model.UpdateCenter.xml

# 启动 Jenkins 并查看状态
systemctl restart jenkins
systemctl status jenkins
  # ● jenkins.service - LSB: Start Jenkins at boot time
  #      Loaded: loaded (/etc/init.d/jenkins; generated)
  #      Active: active (exited) since Wed 2020-12-23 14:10:12 UTC; 6s ago
  #        Docs: man:systemd-sysv-generator(8)
  #     Process: 357567 ExecStart=/etc/init.d/jenkins start (code=exited, status=0/SUCCESS)
```

**安装后设置向导:**  
参考地址: https://www.jenkins.io/zh/doc/book/installing/#setup-wizard

-   (1) 解锁 Jenkins  
    当您第一次访问新的Jenkins实例时，系统会要求您使用自动生成的密码对其进行解锁。  
    浏览到 http://localhost:8080（或安装时为Jenkins配置的任何端口），并等待解锁 Jenkins 页面出现。
    

```
# Jenkins控制台日志显示可以获取密码的位置（在Jenkins主目录中）
cat /var/lib/jenkins/secrets/initialAdminPassword
# 23092528120a45488a73ff4e7565e06f
```

-   (2) 自定义jenkins插件  
    解锁 Jenkins之后，在 Customize Jenkins 页面内， 您可以安装任何数量的有用插件作为您初始步骤的一部分。  
    两个选项可以设置:
    

-   安装推荐的插件 - 安装推荐的一组插件，这些插件基于最常见的用例.
    
-   选择插件来安装 - 选择安装的插件集。当你第一次访问插件选择页面时，默认选择建议的插件。(`新手首次搭建建议选择此项`)
    

![WeiyiGeek.选择安装的插件](https://i0.hdslb.com/bfs/article/6dc1f0dc0c79a20eae92d2f3ee2b2d2425fd1fbd.png@942w_749h_progressive.webp)

PS : 如果您不确定需要哪些插件，请选择`安装建议的插件` 。 您可以通过Jenkins中的`Manage Jenkins > Manage Plugins`页面在稍后的时间点安装（或删除）其他Jenkins插件 。

-   (3) 不安装任何插件进入`插件列表`点击右上角的`x`关闭窗口,显示如下页面则跳过了管理员用户密码重设;
    

PS : 一旦初始安装完成后，可通过插件管理器安装其他插件

-   (4) 进入 Jenkins 控制台安装成功，然后第一个管理员用户；  
    点击 Dashboard -> admin -> Configure -> Password 进行管理员密码设置;  
    从这时起，Jenkins用户界面只能通过提供有效的用户名和密码凭证来访问。
    

![WeiyiGeek.Jenkins](https://i0.hdslb.com/bfs/article/8fa64200064e17ba08b0b987649e2404ec7a0b65.png@942w_671h_progressive.webp)

Ubuntu 下 Jenkins 相关文件及其目录(目录结构)一览:

```
# 配置文件
/etc/default/jenkins
# /etc/sysconfig/jenkins # CentOS

# 家目录&插件目录&构建工作目录
/var/lib/jenkins/
/var/lib/jenkins/config.xml  # jenkins root configuration
/var/lib/jenkins/userContent  # 该目录下的文件将在http://server/userContent/下提供
/var/lib/jenkins/plugins   # 插件
/var/lib/jenkins/logs      # 存放jenkins相关的日志
/var/lib/jenkins/fingerprints  # 存储指纹记录
/var/lib/jenkins/secrets   # 密码秘钥所在目录
/var/lib/jenkins/workspace # 各任务的构建工作目录
/var/lib/jenkins/jobs      # 浏览器上面创建的任务都会存放在这里
          +- [JOBNAME]          # 每个作业Job的子目录
              +- config.xml     # (job configuration file)
              +- workspace      # (working directory for the version control system)
              +- latest         # (symbolic link to the last successful build)
              +- builds
                +- [BUILD_ID]      # (for each build)
                  +- build.xml     # (build result summary)
                  +- log           # (log file)
                  +- changelog.xml  # (change log)

# 启动文件 {start|stop|status|restart|force-reload}
/etc/init.d/jenkins

# systemd 管理的serice
/run/systemd/generator.late/jenkins.service
/run/systemd/generator.late/graphical.target.wants/jenkins.service
/run/systemd/generator.late/multi-user.target.wants/jenkins.service
```

**文件内容&目录结构详细说明**

-   1.config.xml 核心配置文件: 包含了Jenkins的版本信息、权限认证规则、workspace目录定义、builds目录定义、视图信息等等。其他的 xml 文件是 Jekins 服务扩展功能的配置信息文件。
    
-   2.plugins 插件目录: 已经安装的Jenkins插件都可以在里面找到对应的文件。每一个插件基本是由一个目录和一个与目录同名的文件配对组成。
    
-   3.jobs 执行任务存储目录: 该目录是 Jenkins 管理的所有`构建任务的配置细节、构建后的产物和数据`。Jenkins 服务所有的 Job 都会在这个目录下，创建一个以 Job 名称命名的文件夹。  
    job 任务的文件夹中存储的文件有：
    

```
config.xml 任务的XML格式声明信息。
nextBuildNumber 文件记录下次构建时的 buildNumber
builds 目录存储此 Job 构建的历史。

~ $ ls /var/lib/jenkins/jobs/
HelloWorld  Kubernetes-jenkins-slave  pineline-use-node
```

-   4.workspace 工作空间目录: 包含了这个构建作业的源代码, Jenkins存放项目的工作空间。进入这个workspace目录，里面就是你之前创建的项目的目录。在构建过程中Jenkins会根据项目中配置的远程代码仓库的地址去拉取源码到项目目录中并在这里完成打包。之前我们在打包的脚本中用到的`$WORKSPACE`表示的就是workspace下对应项目的目录。
    
-   5.tools 工具目录: Jenkins 服务设置安装 tools ，会安装在这个目录中。安装工具的方式是：【Manage Jenkins】 -> 【Global Tool Configuration】 页面。
    
-   6.updates 更新目录: 用来存放可用的插件更新。
    
-   7.users 用户信息目录: 存储用户的账号信息。
    
-   8.nodes 节点目录: Jenkins 在配置了主从或者工作节点之后会在这里有相应的信息。
    
-   9.userContent 用户生成的文件: 用于存储在 Jenkins 管理过程中生成的文件；比如使用`Convert To Pipeline 插件`可以将 JOB 转换成 Pipeline，生成的 Pipeline 的内容会以文件的形式存储在这个文件夹中。
    
-   10.fingerprints 文件指纹目录: `文件指纹（fingerprints）`是一个简单的MD5校验和。Jenkins维护了一个md5sum数据库，用于文件指纹校验。对于每个md5sum，Jenkins记录了哪些项目的哪些构建使用了他。在每次构建运行和文件被采集指纹时这个数据库会更新。为了避免过多的磁盘使用，Jenkins不存储实际的文件。相反它只存储md5sum和它的使用记录。
    
-   11.logs 日志目录: 用于存储 Jenkins 服务的日志，主要是事件日志和工作日志。
    
-   12.war 目录: 如果是以WAR包形式运行的Jenkins，该目录下存放的是解压后的WAR包。
    
-   13.Jenkins 服务另外的文件目录:
    

```
secrets
init.groovy.d
workflow-libs
scriptler
config-history
```

Tips: jenkins存放数据不依靠数据库所以在移植时只需要拷贝整个程序主目录即可，需要注意的是jobs和plugins目录比较重要

描述: 使用容器化的方式部署 Jenkins Master 节点，可以选择自行构建镜像，推荐使用 Jenkins 官方提供的镜像。

Jenkins 服务官方在 dockerhub 平台发布官方镜像https://hub.docker.com/u/jenkins

**1.Jenkins Master 镜像基础使用**

-   Jenkins Web 服务监听的端口为 8080
    
-   Jenkins 使用JNLP 连接 Agent 节点使用的端口为 5000
    
-   Jenkins 在 /var/jenkins\_home 目录中保存workspace中的数据，包括插件和配置信息
    
-   Jenkins 服务默认使用jenkins 用户运行，uid为1000；请注意文件权限问题
    

**2.Jenkins Master 使用和升级**  
使用 Jenkins 的镜像构建容器时，至少要将端口映射出去，将/var/jenkins\_home 目录做持久化。

```
# 示例：用 jenkins：2.107.3 创建容器并运行。
docker volume create jenkins-data
docker run --name jenkins-production \
           --detach \
           -p 50000:50000 \
           -p 8080:8080 \
           -v jenkins-data:/var/jenkins_home \
           jenkins/jenkins:2.275-alpine
# If run for the first time, just run the following to get the admin
# password once it has finished starting
docker exec jenkins-production bash -c 'cat $JENKINS_HOME/secrets/initialAdminPassword'
```

在需要对 Jenkins 服务升级时，只需要使用新的镜像替换即可；持久化存储和端口映射操作相同。  
示例：将 Jenkins 服务升级到 2.121.3

```
docker stop jenkins-production
docker rm jenkins-production
# just temporarily docker rename it instead if that makes you worried
docker run --name jenkins-production \
           --detach \
           -p 50000:50000 \
           -p 8080:8080 \
           -v jenkins-data:/var/jenkins_home \
           jenkins/jenkins:2.277-alpine
```

**3.Jenkins Master 节点配置扩展(了解即可)**

-   3.1 设置Master 节点的执行者 executors 数量  
    描述: 使用groovy脚本指定并设置Jenkins Master 实例的执行程序数。默认情况下，它设置为2个执行者，但是您可以扩展映像并将其更改为所需的执行者数:
    

```
executors.groovy
import jenkins.model.*
Jenkins.instance.setNumExecutors(5)

# 和 Dockerfile

FROM jenkins/jenkins:lts
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy
```

-   3.2 传递 JVM 的启动参数  
    描述: 您可能需要自定义运行Jenkins的 JVM，通常是为了传递系统属性（props列表）或调整堆内存设置
    

```
# 使用JAVA_OPTS环境变量：
docker run --name myjenkins -p 8080:8080 -p 50000:50000 --env JAVA_OPTS=-Dhudson.footerURL=http://mycompany.com jenkins/jenkins:lts
```

-   3.3 给 Jenkins 启动器传递参数  
    描述: 您传递给运行 Jenkins 镜像的 docker 的参数将传递给 jenkins 启动器 ，例如运行 `docker run jenkins/jenkins:lts --version` 将显示Jenkins版本
    

```
# 通过定义Jenkins参数 JENKINS_OPTS
示例1.Dockerfile使用此选项来强制将HTTPS与映像中包含的证书一起使用。
FROM jenkins/jenkins:lts
COPY https.pem /var/lib/jenkins/cert
COPY https.key /var/lib/jenkins/pk
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8083 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk
EXPOSE 8083

# 通过定义Jenkins参数 JENKINS_SLAVE_AGENT_PORT
示例2.Dockerfile中定义来更改jenkins的默认从属代理端口。
FROM jenkins/jenkins:lts
ENV JENKINS_SLAVE_AGENT_PORT 50001
# 或作为docker的参数
docker run --name myjenkins -p 8080:8080 -p 50001:50001 --env JENKINS_SLAVE_AGENT_PORT=50001 jenkins/jenkins:lts
```

Tips ：此环境变量将用于设置将系统属性添加`jenkins.model.Jenkins.slaveAgentPort`到JAVA\_OPTS的端口 。

-   3.4 配置日志记录  
    描述: 可以通过属性文件和`java.util.logging.config.fileJava`属性来配置Jenkins日志记录。例如：
    

```
mkdir data

cat > data/log.properties <<EOF
handlers=java.util.logging.ConsoleHandler
jenkins.level=FINEST
java.util.logging.ConsoleHandler.level=FINEST
EOF

docker run --name myjenkins -p 8080:8080 -p 50000:50000 \
--env JAVA_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties"\
 - v `pwd`/data:/var/jenkins_home jenkins/jenkins:lts
```

-   3.5 调整Jenkins 浏览器访问的路径path  
    描述: 如果您需要在浏览器中访问 Jenkins 服务时要添加一个 path 路径，则需要添加环境变量`JENKINS_OPTS="--prefix=/jenkins"`。
    

Tips : 容器中 Docker 客户端配置

-   1.构建 NodeJS 任务执行镜像的 Dockerfile 文件内容:
    

```
FROM docker.io/node:alpine
run apk add --no-cache git docker-cli
RUN yarn config set registry "https://registry.npm.taobao.org"
```

-   2.配置 Docker 客户端连接远程的 Docker 服务端:使用一个环境变量就可以实现这个配置。
    

```
env:
- name: DOCKER_HOST
  value: tcp://docker.example.com:2375
```

Tips : 配置镜像仓库的认证信息,认证信息一般存储在 `/root/.docker/config.json` 文件中。这里可以模仿前面 Maven 服务的 settings.xml 文件的配置方式配置, 然后就可以实现了连接 Docker Server 端构建镜像，对镜像打 tag 的操作, 然后将制作好的镜像推送到远程的镜像仓库中，例如Harbor。

请参考后续的 `<(2) 集群搭建Jenkins Master 节点>` 章节的文章。

描述: 环境变量可以被看作是pipeline与Jenkins交互的媒介, 环境变量可以分为`Jenkins内置变量`和`自定义变量`以及`自定义全局环境变量`。

比如，可以在 pipeline 中通过 BUILD\_NUMBER 变量知道构建任务的当前构建次数。

**1.内置变量**  
描述: 在pipeline执行时，Jenkins通过一个名为 env 的全局变量，将Jenkins内置环境变量暴露出来。

```
${env.BUILD_NUMBER} 方式一，推荐使用
$env.BUILD_NUMBER 方式二,
${BUILD_NUMBER} 方式三，不推荐使用
```

例如:在实际工作中经常用到的变量。

```
BUILD_URL：当前构建的页面URL。
BUILD_NUMBER：构建号，累加的数字。
BRANCH_NAME：多分支pipeline项目支持。
GIT_BRANCH：通过git拉取的源码构建的项目才会有此变量。
```

**2.自定义环境变量**  
描述: 当 pipeline 变得复杂时，我们就会有定义自己的环境变量的需求。声明式 pipeline 提供了environment 指令，方便自定义变量。

另外 environment 指令可以在pipeline中定义，代表变量作用域为整个 pipeline；也可以在 stage 中定义，代表变量只在该阶段有效。

Tips : 如果在environment中定义的变量与env中的变量重名，那么被重名的变量的值会被覆盖掉。

**3.自定义全局环境变量**  
描述: 如果我们需要定义一些全局的跨pipeline的自定义变量。

我们可以进入 `Manage Jenkins→Configure System→Global properties` 页，勾选`“Environment variables”复选框`，单击“Add”按钮，在输入框中输入变量名和变量值即可，

Tips : 自定义全局环境变量会被加入 env 属性列表中，所以，使用自定义全局环境变量与使用Jenkins内置变量的方法无异。

Tips : Jenkins 内置变量参考 请看补充说明中的内置环境变量

主要缺省参数类型如下几类:

-   Boolean 参数
    
-   Choice 参数 (常用)
    
-   String 参数 （常用）
    
-   File 参数
    
-   密码 参数
    
-   凭据 参数
    
-   其他 参数
    
-   运行时 参数
    

在参数化构建过程 -> 构建参数添加 -> 构建参数的变量 ->通过 `$paramater 或者 ${paramater}`的形式使用

![WeiyiGeek.构建参数](https://i0.hdslb.com/bfs/article/7832716d2cdd17eeb3ba7cf8ecc78fe3ad33ee6a.png@801w_617h_progressive.webp)

Tips ： 环境变量生效的顺序`全局环境变量 < slave配置环境变量 < JOB参数 < JOB injected环境变量`

官网地址: https://www.jenkins.io/  
官方文档: https://www.jenkins.io/zh/doc/

描述: 我们创建Job可以进行并发构建，每个并发构建使用单独的workspace，默认是`<workspace path>@<num>`

![WeiyiGeek.并发构建](https://i0.hdslb.com/bfs/article/40798b4520f549607473b33834fb59f3c701efc2.png@942w_236h_progressive.webp)

Tips : slave 也可以进行并发构建前提是需要`Node and Label parameter plugin`插件，并且需要配置node类型参数;

**常见的triggers几种:**

-   Build periodically : 设定类似cron周期性时间触发构建
    
-   poll SCM : 设定类似 cron 周期性时间触发检查代码变化，只有代码变动时才触发构建
    
-   Hooks : 用过SVN的都知道钩子 `Github hooks / Gitlab hooks`
    
-   Events : Gerrit event
    

![WeiyiGeek.triggers](https://i0.hdslb.com/bfs/article/61f72b7a962b54b20d8e9b135adc8b62c4353e74.png@942w_311h_progressive.webp)

**build periodically**  
描述: H 代表 jenkins 自己分配时间不去指定客观时间, 注意一般都是Jenkins根据监控资源利用率算法分配的。

```
# * * * * *
# 第一颗*表示分钟，取值0~59
# 第二颗*表示小时，取值0~23
# 第三颗*表示一个月的第几天，取值1~31
# 第四颗*表示第几月，取值1~12
# 第五颗*表示一周中的第几天，取值0~7，其中0和7代表的都是周日

1.每30分钟构建一次：H/30 * * * *

2.每2个小时构建一次 H H/2 * * *

3.每天早上8点构建一次: 0 8 * * *

4.每天的8点，12点，22点，一天构建3次 0 8,12,22 * * *

# 多个时间点，中间用逗号隔开
```

**Poll SCM**  
描述: 他会定时检查源码变更（根据SCM软件的版本号），如果有更新就checkout最新code下来然后执行构建动作；

比如: 你可以如果我想每隔30分钟检查一次源码变化有变化就执行;

H/30 \* \* \* \*  

触发下游构建：

-   (1) 其他项目构建后触发: "Build other projects" under "Post build actions" -> 输入项目名称
    
-   (2) 利用 Parameterized Trigger 插件 参数化构建 -> 在构建后操作步骤中 -> `Trigger Parameterized build on other peoject` -> `Restrict matrix execuson to a subset`
    

![WeiyiGeek.上下游构建](https://i0.hdslb.com/bfs/article/f9388804c796ac456db73722a0fe0a819bdacc3b.png@942w_243h_progressive.webp)

描述: Jenkins 服务提供了很多类型的API；因为工作需要这里只记录一下执行job的API使用CURL调用的方式。

**Jenkins API 介绍**

-   1.Jenkins API 从级别上分类
    

```
* 站点 API：创建Job、复制Job、Build 队列、重启Jenkins等
* Job API：修改Job、删除Job、获取 Build 信息、Build Job、禁用Job、启用Job
* Build Job: 根据 Build Number 获取Build 信息，获取Build 控制台的输出日志
```

-   2.传输数据格式: POST 传输数据支持的格式有`XML,JSON,PYTHON`
    
-   3.安全处理: 可以找到一些开发语言编写的API封装包，结合到自己的脚本中，提高开发效率。
    

```
* CSRF 跨域攻击保护：可以在全局安全中设置。
* 用户验证设置：可以使用用户的密码，也可以为用户创建token。
```

**CURL 调用 API 执行 Job 实例:**

```
# Curl Api 调用
CURL -X POST ${JOB_URI}
--user ${user_name}:${user_password}|${user_token}
--data-urlencode json="{\"parameter\":${JSON_DATA},\"statusCode\":\"303\",\"redirectTo\":\".\"}"
```

**参数解析:**

-   JOB\_URL: 就是Job 的访问URL地址。在Job 页面中点击`”Build with Parameters”`就可以获取到地址。一般以”build?delay=0sec“结尾; 示例: `http://192.168.1.1/job/HelloWorld/build?delay=0sec`
    
-   user\_name,user\_password,user\_token 用户验证信息
    
-   JSON\_DATA：Job 运行时，需要传递的参数，由Json 结构封装；示例：`[{\"name\":\"branchName\",\"value\":\"origin/dev\"},{\"name\":\"projectName\",\"value\":\"helloworld\"}]`
    

描述: 当Job创建数量达到一定时我们需要在Jenkins中建立视图（分类）,可以帮助我们快速找到某个所需Job;  
实际上Job的视图类似于我们电脑上的文件夹可以通过一些过滤规则，将已经创建好的Job过滤到视图之中,也可以在视图中直接创建我们的Job;

我们可以采用`View`或者`directory`文件两种方式进行管理:

-   views - 视图方式
    

1.views视图更加灵活，不改变job的路径

2.views有多种形式、层级、看板，流水线等多样化

-   directory - 文件夹方式 (名称空间隔离)
    

1.文件夹适合多个团队共用Jenkins 

2.性能更好，执行速度更快 

3.支持RBAC权限管理

两则异同说明: 文件夹创建一个可以嵌套存储的容器利用它可以进行分组，`而视图仅仅是一个过滤器，其次文件夹则是一个独立的命名空间`，因此你可以有多个相同名称的的内容，只要它们在不同的文件夹里即可。

**views - 视图方式**  
创建流程：

-   Step 1.我的视图 -> 点击所有旁边 + 号 -> 设置视图名称 -> 选择三种类型`列表视图/包括全局视图/我的视图`  
    列表视图 : 显示简单列表。你可以从中选择任务来显示在某个视图中  
    包括全局视图 : Shows the content of a global view.  
    我的视图 : 该视图自动显示当前用户有权限访问的任务
    
-   Step 2.此处设置列表视图为例 -> 选择任务列表 -> 增加或者删除指定显示列
    

![WeiyiGeek.views](https://i0.hdslb.com/bfs/article/35dece393d04a19d86dc11b573f3080ff7c1d7f5.png@942w_419h_progressive.webp)

**directory - 文件夹方式**  
创建流程:

-   Step 1.创建Job -> 选择`文件夹` -> 输入任务名称`directory-test`
    
-   Step 2.在`directory-test`文件夹下 -> 可以继续创建视图`(+ )` -> 选择(`列表视图/包括全局视图/我的视图`)
    

![WeiyiGeek.directory](https://i0.hdslb.com/bfs/article/1ed820caa1a71ca9bcd95b8183dc75adcd377698.png@942w_435h_progressive.webp)

**(1) 插件安装加速**  
Q: 在安装插件时如何进行配置安装加速?

> 答: 选择国内清华大学的Jenkins插件更新源 `https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`

配置位置: `Dashboard > Plugin Manager > Advanced > Update Site`

![WeiyiGeek.插件安装加速](https://i0.hdslb.com/bfs/article/2d3e8cf5d87a0a9314d4dda39d1c4c93659ed04b.png@942w_804h_progressive.webp)

**（2）插件安装的几种方式**

-   Step 1.采用插件更新源搜索进行在线安装，例如下面对Jenkins进行汉化插件的安装, 搜索 `Localization: Chinese (Simplified)` 插件安装即可;  
    

![WeiyiGeek.Jenkins 汉化插件安装](https://i0.hdslb.com/bfs/article/de2b578fca5dec9ee3f83da89a9ab9715776551a.png@942w_287h_progressive.webp)

当安装插件安装完成时候可以选择重启 Jenkins 或 跳转到首页, 此处我重启 Jenkins 如下图所示汉化成功;

![WeiyiGeek.Jenkins 汉化界面](https://i0.hdslb.com/bfs/article/71cdf934114b8fc8ea66ccf5dbd909346066f8c6.png@942w_645h_progressive.webp)

-   Step 2.通过Web页面上传.hpi文件进行插件安装  
    清华大学 jenkins 插件下载: https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/  
    例如下面进行 git 插件下载 进行上传其 `git.hpi` 文件进行插件安装，之后上传即可;
    

![WeiyiGeek.Jenkins git 插件安装](https://i0.hdslb.com/bfs/article/19379c327f8ba53b6083bfdf0b3e0baab06a1c3a.png@942w_837h_progressive.webp)

-   Step 3.导入之前服务器已安装的插件进行离线安装；
    

```
# (1) Jenkins 已安装的插件目录
/var/lib/jenkins/plugins$ ls
  # apache-httpcomponents-client-4-api      display-url-api      jdk-tool                  localization-zh-cn      script-security      trilead-api
  # apache-httpcomponents-client-4-api.jpi  display-url-api.jpi  jdk-tool.jpi              localization-zh-cn.jpi  script-security.jpi  trilead-api.jpi
  # command-launcher                        git                  jsch                      mailer                  ssh-credentials      workflow-scm-step
  # command-launcher.jpi                    git-client           jsch.jpi                  mailer.jpi              ssh-credentials.jpi  workflow-scm-step.jpi
  # credentials                             git-client.jpi       localization-support      scm-api                 structs              workflow-step-api
  # credentials.jpi                         git.jpi              localization-support.jpi  scm-api.jpi             structs.jpi          workflow-step-api.jpi


# (2) 打包插件目录并上传到另外一台同版本的Jenkins服务器(PS:不同版本间可能会出现插件不兼容的情况)
tar -zcvf jenkins_2.263.1_plugins.tar.gz /var/lib/jenkins/plugins

# (3) 解压插件目录并进行所属者组修改
tar -zxvf jenkins_2.263.1_plugins.tar.gz
chown -R jenkins:jenkins /var/lib/jenkins/plugins

# (4) 启动 Jenkins 服务
systemctl stop jenkins
systemctl start jenkins

# (5) 启动 日志 查看
tail -f /var/log/jenkins/jenkins.log
```

描述: 由于`jenkins`默认的权限管理体系不支持用户组或角色的配置，因此需要安装第三发插件来支持角色的配置，我们使用`Role-based Authorization Strategy`插件，安装请参考前面插件管理章节。

插件功能特点:

-   按角色分权限
    
-   按项目分权限
    

操作流程：

-   Step 1.插件安装后我们先启用该插件管理Jenkins权限 -> 面板 -> 全局安全配置（`Global Security`）-> 授权策略 -> 找到并选中 `Role-Based Strategy` -> 然后应用保存 。
    
-   Step 2.管理Jenkins -> 安全 -> `Manage and Assign Roles` -> 可以进行角色权限管理与分配 -> 此处添加一个test角色并设置其权限 -> 应用 -> 保存;  
    ·
    
-   Step 3.Assign Roles -> Global roles(按角色)、Item roles(按项目)、Node roles(按节点) -> 用户添加(同时您需要在Jenkins用户中新增一个该用户) -> 保存应用
    

-   添加用户和组。默认情况下用户/组名是区分大小写的，因此它需要与您在`Configure Global Security->Access Control->Security Realm`下的用户/组定义匹配。
    

![WeiyiGeek.jenkins权限管理](https://i0.hdslb.com/bfs/article/a8a2bff0efaea84b326dbad35ca15df7fff0b46a.png@942w_557h_progressive.webp)

在 Jenkins 服务中运行 Pipeline 等任务过程中，需要依赖一些工具（环境需求）；比如 JDK,MAVEN或者golang 或者 python 环境。

构建工具的安装方式有三种:

-   1.自行安装工具: 即自行在服务器上安装配置，然后在 Jenkins 服务中配置好这些工具的安装信息就可以使用了。配置路径：`Manage Jenkins→Global Tool Configuration`
    
-   2.Jenkins 自动安装工具: 利用 Jenkins 服务提供的【工具自动安装功能】，实现工具的快速配置。
    
-   3.tools 指令安装工具:帮助我们自动下载并安装所指定的构建工具，并将其加入 PATH 变量中。这样，我们就可以在sh步骤里直接使用了, 但在agent none的情况下不会生效。
    

Tips : tools指令默认支持3种工具：JDK、Maven、Gradle。通过安装插件，tools 指令还可以支持更多的工具。

原文地址: https://mp.weixin.qq.com/s/wNZPvV4okZBDuLT7sgZyww

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>               如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！     
> 
>               欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 

![](https://i0.hdslb.com/bfs/article/fd8645896d98c4d98503adf2e586e5f992a9a2c6.png@942w_483h_progressive.webp)

个人主页：https://weiyigeek.top