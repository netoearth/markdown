文章首发于：[菠萝的博客](https://link.juejin.cn/?target=https%3A%2F%2Fwww.runnable.run%2F "https://www.runnable.run/")，欢迎大家关注获取最新更新

## 1\. 需求背景

现在越来越多的公司进行部署springboot项目都是采用Jenkins的方式进行部署，可以自动拉取git上的项目，然后进行打包成docker镜像，部署本地，再推送到公司的镜像仓库。

这样不仅可以对打包的镜像做一个备份，当下一次生产上线的时候，出现严重问题可以及时进行回滚，而且自动化的部署，可以让开发和测试的沟通更加畅快。再也不用让测试催开发说，你这个bug怎么还没修复，开发总是会说：修复了，还在打包部署。而传统方式的打包总是会花10来分钟，甚至更长。

从传统流程：修复bug->验证本地bug是否修复->打成jar/war/image->上传服务器/仓库->替换启动/pull image

到现在的自动化：修复bug->验证本地bug是否修复->Jenkins自动构建

不得不说还是非常的方便。

这篇博客主要讲述，一个springboot项目如何使用jenkins+docker进行一键部署。

## 2\. 开始前的准备

环境：

-   已经安装docker的linux系统，这里使用的是ubuntu
-   一个简单的springboot项目,且已经放到git仓库中，可以是github或者码云或者阿里云个人仓库
-   注册一个阿里云账号

## 3\. 通过docker安装Jenkins

官方文档说明：

[www.jenkins.io/doc/book/in…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fdoc%2Fbook%2Finstalling%2Fdocker%2F "https://www.jenkins.io/doc/book/installing/docker/")

这里将重点部分抽离出来，只需要较少的几步就可以完成Jenkins的镜像构建以及安装。

### 3.1 Jenkins的自定义镜像构建

在你的工作空间下，新建文件名为Dockerfile的文件，并且输入以下字符用来构建Jenkins镜像

```
FROM jenkins/jenkins:2.289.3-lts-jdk11
USER root
RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable"
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.24.7 docker-workflow:1.26"
复制代码
```

在当前文件夹下打开命令行

```
docker build -t myjenkins-blueocean:1.1 .
复制代码
```

不要忘了上面👆这个命令最后有个 “.”

这一个相当长时间的过程，可以先进行SpringBoot项目的对应[操作](https://juejin.cn/post/7053359079216922661#springboot_op "#springboot_op")

镜像构建完成之后，你可以通过 `docker images` 看到刚刚已经构建完成的镜像

### 3.2 创建Jenkins容器

```
sudo docker run -u root -d  -p 8080:8080  -p 50000:50000  -v /var/run/docker.sock:/var/run/docker.sock    myjenkins-blueocean:1.1
复制代码
```

-   `-u root` 使用root身份生成容器
-   `-p`映射端口
-   `-v /var/run/docker.sock:/var/run/docker.sock` 表示Docker守护程序通过其监听的基于Unix的套接字。 该映射允许`jenkinsci/blueocean` 容器与Docker守护进程通信， 如果`jenkinsci/blueocean` 容器需要实例化其他Docker容器，则该守护进程是必需的。说人话就是你在Jenkins容器中要调用宿主机的docker怎么办，就是通过这样的方式做到的

## 4\. 阿里云配置个人容器镜像服务

为什么不使用dockerhub的仓库是因为个人版只有公开仓库，这样并不是一个好的选择。

登录阿里云之后，进行搜索 `容器镜像服务`即可找到

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bddab6271d684130a2b21bee21fa8b07~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 4.1 设置密码

在个人版中，访问凭证中，设置 `固定密码`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bccd0680639e46f18ec3a9ede1bfc55c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 4.2 仓库管理中，创建命名空间

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afcfd8b2a6274b78b9f7f55c880b77fb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 4.3 仓库管理中，创建镜像仓库

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2abd7207b8b4997a055e4d895aa34b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

下一步，选择本地仓库

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8e7147cc14f4b46bcaab911572d9136~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

返回到刚刚创建的列表，点击进入刚刚创建的仓库，当中的公网地址，就是我们之后需要推送到阿里云仓库的参数

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12dd5c7de89047bbb2753448382806df~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

## 5\. SpringBoot项目添加Dockerfile，Jenkinsfile，以及sh脚本 {springboot\_op}

官方文档操作：[使用Maven构建Java应用程序](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Ftutorials%2Fbuild-a-java-app-with-maven%2F "https://www.jenkins.io/zh/doc/tutorials/build-a-java-app-with-maven/")

### 5.1 添加Dockerfile文件

为了完成这一步，我们需要编写Dockerfile，如下所示：

```
FROM openjdk:8-alpine

# PROJECT_NAME 填写你的项目名字
ENV PROJECT_NAME learn-1.4.1
# PROJECT_HOME 构建成镜像之后，存放的目录位置
ENV PROJECT_HOME /usr/local/${PROJECT_NAME}

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone
RUN mkdir $PROJECT_HOME && mkdir $PROJECT_HOME/logs

ARG JAR_FILE
COPY ${JAR_FILE} $PROJECT_HOME

ENTRYPOINT /usr/bin/java -jar -Xms1536m -Xmx1536m $PROJECT_HOME/$PROJECT_NAME.jar
复制代码
```

Dockerfile的详细用法可以参考：[Docker Dockerfile | 菜鸟教程](https://link.juejin.cn/?target=https%3A%2F%2Fwww.runoob.com%2Fdocker%2Fdocker-dockerfile.html "https://www.runoob.com/docker/docker-dockerfile.html")

将这个文件放置于项目的根目录下

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bec8402bae4349178d387d13309f19bc~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 5.2 引入maven docker 镜像编译插件

#### 5.2.1 maven的 pom.xml文件的 properties

```
<properties>
        <docker.registry.publish.url>registry.cn-hangzhou.aliyuncs.com</docker.registry.publish.url>
        <docker.namespace>namespace</docker.namespace>
        <docker.profile>dev</docker.profile>
    </properties>
复制代码
```

其中

-   `docker.registry.publish.url` docker推送到阿里云的公网地址
-   `docker.namespace` 之前创建仓库的命名空间
-   `docker.profile` 开发环境还是测试环境的配置

#### 5.2.2 maven的 pom.xml文件的 plugin

```
<!-- spring-boot-maven-plugin使得springboot项目能打成jar包进行启动 -->
          <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>run.runnable.learn.LearnApplication</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!--docker 镜像编译插件-->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.10</version>
                <dependencies>
                    <dependency>
                        <groupId>javax.activation</groupId>
                        <artifactId>activation</artifactId>
                        <version>1.1.1</version>
                    </dependency>
                </dependencies>
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
                    <repository>${docker.registry.publish.url}/${docker.namespace}/${project.build.finalName}-${docker.profile}</repository>
                    <tag>${project.version}</tag>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
复制代码
```

### 5.3 Jenkinsfile文件编写

参考资料：

[官方文档｜使用 Jenkinsfile](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Fbook%2Fpipeline%2Fjenkinsfile%2F "https://www.jenkins.io/zh/doc/book/pipeline/jenkinsfile/")

[官方文档｜流水线语法](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Fbook%2Fpipeline%2Fsyntax%2F "https://www.jenkins.io/zh/doc/book/pipeline/syntax/")

[Jenkins官方文档｜Maven 构建 Java 应用](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Ftutorials%2Fbuild-a-java-app-with-maven%2F "https://www.jenkins.io/zh/doc/tutorials/build-a-java-app-with-maven/")

在项目根目录下创建文件：`Jenkinsfile`

```
pipeline {
    agent none
    environment {
        deploy_tag = '1.4.1'
        profile='dev'
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean install -DskipTests=true -P${profile} \
                && mvn clean'
            }
        }
        stage('Deploy') {
            agent any
            steps {
                sh 'sh deploy_docker.sh restart ${deploy_tag} ${profile}'
            }
        }
        stage('Push') {
            agent any
            steps {
                sh 'sh deploy_docker.sh push ${deploy_tag} ${profile}'
            }
        }
    }
}
复制代码
```

在这段脚本中，

-   `environment` 是自定义的环境变量，定义两个参数，一个是deploy\_tag，部署镜像版本。profile是环境参数，测试环境或者生产环境

定义了三个阶段：`Build` `Deploy` `Push`

-   `Build`

这里的 `image` 参数（参考 [`agent`](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fdoc%2Fbook%2Fpipeline%2Fsyntax%23agent "https://www.jenkins.io/doc/book/pipeline/syntax#agent") 章节的 `docker` 参数） 是用来下载 [`maven:3-apline` Docker镜像](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2F_%2Fmaven%2F "https://hub.docker.com/_/maven/") （如果你的机器还没下载过它）并将该镜像作为单独的容器运行。这意味着：\* 你将在Docker中本地运行相互独立的Jenkins和Maven容器。

Maven容器成为了Jenkins用来运行你的流水线项目的[agent](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fdoc%2Fbook%2Fglossary%2F%23agent "https://www.jenkins.io/doc/book/glossary/#agent")。 这个容器寿命很短——它的寿命只是你的流水线的执行时间。

这里的 `args` 参数在暂时部署的Maven Docker容器的 `/root/.m2` （即Maven仓库）目录 和Docker主机文件系统的对应目录之间创建了一个相互映射。这背后的实现细节超出了本教程的范围，在此不做解释。 但是，这样做的主要原因是，在Maven容器的生命周期结束后，构建Java应用程序所需的工件 （Maven在流水线执行时进行下载）能够保留在Maven存储库中。这避免了在后续的流水线执行过程中， Maven反复下载相同的工件。请注意，不同于你为 [`jenkins-data`](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Ftutorials%2Fbuild-a-java-app-with-maven%2F%23download-and-run-jenkins-in-docker "https://www.jenkins.io/zh/doc/tutorials/build-a-java-app-with-maven/#download-and-run-jenkins-in-docker") 创建的Docker数据卷，Docker主机的文件系统在每次重启Docker时都会被清除。 这意味着每次Docker重新启动时，都会丢失下载的Maven仓库工件。

-   `Deploy`
    
    执行 `deploy_docker.sh` 中的restart方法
    
-   `Push`
    

执行 `deploy_docker.sh` 中的push方法

### 5.4 deploy\_docker.sh脚本编写

这个脚本是用来控制打包好的docker image一系列操作，包括push remove stop start等等

```
#!/bin/bash

#项目占用端口
APP_PORT=8081
# 项目名字
APP_NAME=learn-1.4.1
APP_HOME=/home/admin/${APP_NAME}
APP_OUT=${APP_HOME}/logs

PROG_NAME=$0
ACTION=$1
TAGNAME=$2
PROFILE=$3

# 阿里云仓库命名空间
APP_NAMESPACE=workspace
# 推送至阿里云仓库的地址
APP_REGISURL=registry.cn-hangzhou.aliyuncs.com
APP_RESP=${APP_REGISURL}/${APP_NAMESPACE}/${APP_NAME}-${PROFILE}
# 阿里云账号
APP_USERNAME=
# 阿里云配置个人容器镜像服务时，设置的密码
APP_PASSWORD=

mkdir -p ${APP_HOME}
mkdir -p ${APP_HOME}/logs

usage() {
    echo "Usage: $PROG_NAME {start|stop|restart|push}"
    exit 2
}

start_application() {
    echo "starting java process"
    docker run --log-opt max-size=1024m --log-opt max-file=3 --security-opt seccomp:unconfined -dit --name ${APP_NAME} -p ${APP_PORT}:${APP_PORT} --pid=host -v ${APP_OUT}:/usr/local/${APP_NAME}/logs ${APP_RESP}:${TAGNAME}
    echo "started java process"
}

stop_application() {
    echo "ending java process"
    docker stop ${APP_NAME} && docker rm ${APP_NAME}
    echo "ended java process"
}

push_application() {
    echo "starting image push"
    docker login --username=${APP_USERNAME} --password=${APP_PASSWORD} ${APP_REGISURL}
    echo "docker tag ${APP_RESP}:${TAGNAME} ${APP_RESP}:${TAGNAME}"
    echo "docker push ${APP_RESP}:${TAGNAME}"

    #docker tag 用于给镜像打标签
    docker tag ${APP_RESP}:${TAGNAME} ${APP_RESP}:${TAGNAME}
    # 推送到个人的阿里云仓库
    docker push ${APP_RESP}:${TAGNAME}
    docker images|grep ${APP_RESP}|grep none|awk '{print $3 }'|xargs docker rmi
    echo "ended image push"
}

rmi_application() {
    docker rmi ${APP_RESP}:${TAGNAME}
}

start() {
    start_application
}
stop() {
    stop_application
}
push() {
    push_application
}
rmi() {
    rmi_application
}

case "$ACTION" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    restart)
        stop
        start
    ;;
    slaveRestart)
        stop
        rmi
        start
    ;;
    push)
        push
    ;;
    *)
        usage
    ;;
esac
复制代码
```

## 6\. Jenkins配置

在安装完Jenkins之后，这里我们使用Jenkins的默认推荐安装，没有安装自定义的功能

我们要对Jenkins创建流水线，对以上的操作进行配置

参考资料：[Jenkins官方文档｜Maven 构建 Java 应用](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Ftutorials%2Fbuild-a-java-app-with-maven%2F "https://www.jenkins.io/zh/doc/tutorials/build-a-java-app-with-maven/")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d26f45c27d04c99bf5d928b31b3952b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 6.1 general

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6749e6244864bb7aadce93d25418f93~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 6.2 构建触发器

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de7401fd77304e6497dc09e10d066bb3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 6.3 流水线

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99954dc1493441afbbbd63d46e8b2bf0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

这里需要提一下，git的话，在截图中的Credentials中配置时需要注意

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/876062efaacd40bd95334ba625264222~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 6.4 立即构建

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1ffe102428446f499b2f7a79806890c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

成功后，可以看到linux上已经启动该服务，并且阿里云上镜像仓库已经存在你推送的镜像

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb5bf3f8040742b89b222f8c1d03cb72~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)