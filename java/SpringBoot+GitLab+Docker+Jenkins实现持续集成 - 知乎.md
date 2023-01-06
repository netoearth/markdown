**1\. 概述**

本文主要介绍持续集成的搭建方式，采用Docker的方式去搭建Jenkins环境，另外会涉及到SpringBoot和Git等技术。

**2\. 什么是持续集成**

传统的软件开发流程如下：

-   项目经理分配模块给开发人员
-   每个模块的开发人员并行开发，并进行单元测试
-   开发完毕，将代码集成部署到测试服务器，测试人员进行测试
-   测试人员发现bug，提交bug、开发人员修改bug
-   bug修改完毕再次集成、测试

![](https://pic4.zhimg.com/v2-384d08518bad26592b0cd6de7d3ecb8b_b.jpg)

但是这样就出现了如下问题：

-   模块之间依赖关系复杂，在集成时发现大量bug
-   测试人员等待测试时间过长
-   软件交付无法保障

那我们又该如何解决上述问题呢？可以采用持续集成来解决上述问题，那持续集成又是什么呢？大师Martin Fowler对 **持续集成** 是这样定义的：

持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试)来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。

持续集成的好处：

-   自动化集成部署，提高了集成效率。
-   更快地修复问题。
-   更快地进行交付。
-   提高了产品质量。

本文通过下图的模式进行持续集成的方案：

-   开发人员开发代码，推送到Git服务器中
-   当Git服务器中的代码发生变化时，会触发配置在Git服务器中的钩子地址，通知到Jenkins
-   Jenkins把代码下载下来，通过Maven，build成Docker镜像
-   再把Docker镜像推送到Docker仓库中
-   再从Docker仓库中构建出可以运行的Docker容器

![](https://pic3.zhimg.com/v2-4e5ede34ce40cee26c6037d6ace777c6_b.jpg)

接下来，我们来看搭建环境

本文中我们使用Centos7.x进行Docker的安装，所以我们需要在VmWare中先安装Centos7，这一步骤由读者自行安装。

3.1. Docker安装步骤

（1）yum 包更新到最新

（2）安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

（3）设置yum源为阿里云

```
sudo yum-config-manager --add-repo [url=mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo]mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo[/url]
```

（4）安装docker

```
sudo yum install docker-ce
```

（5）安装后查看docker版本

3.2. 设置ustc的镜像

ustc是老牌的linux镜像服务提供者了，还在遥远的ubuntu 5.04版本的时候就在用。ustc的docker镜像加速器速度很快。ustc docker mirror的优势之一就是不需要注册，是真正的公共服务。

编辑该文件：

```
vi /etc/docker/daemon.json
```

在该文件中输入如下内容：

```
{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

3.3. Docker的启动与停止

systemctl命令是系统服务管理器指令

启动docker：

停止docker：

重启docker：

查看docker状态：

开机启动：

好了，到此为止，我们的Docker的基础环境已经装好，接下来我们准备下GitLab环境。

```
docker start docker-registry
```

## 4\. GitLab

GitLab是一个利用 Ruby on Rails 开发的开源应用程序，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目安装。类似GitHub，能够浏览源代码，管理缺陷和注释，可以管理团队对仓库的访问。

4.1. GitLab安装部署

官方支持的方式：

包含一切的RPM包： [Download and install GitLab](https://link.zhihu.com/?target=https%3A//about.gitlab.com/downloads/)

手动安装： [doc/install/installation.md · master · GitLab.org / GitLab FOSS](https://link.zhihu.com/?target=https%3A//gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md)

第三方docker镜像： [sameersbn/docker-gitlab](https://link.zhihu.com/?target=https%3A//github.com/sameersbn/docker-gitlab)

镜像可以快速实现部署并使用，适合于熟悉Docker的人使用，入门很快。

4.2. 下载GitLab镜像

如果我们直接使用Docker的镜像方式去安装GitLab，我们还必须手动安装一些相关软件，例如：Redis，PostgreSql。我们这次选用docker-compose的方式去安装gitlab。

4.2.1 安装docker的docker-compose

docker-compose 是一个用来把 docker 自动化的东西。有了 docker-compose 你可以把所有繁复的 docker 操作全都一条命令，自动化的完成。

运行下边两条命令，即可安装docker-compose

```
curl -L "github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

![](https://pic4.zhimg.com/v2-647eaebe66ab9ea389521a09cd803bb3_b.jpg)

4.2.2 安装wget

4.2.3 下载docker-compose.yml

```
wget [url=/raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml]raw.githubusercontent.co ... /docker-compose.yml[/url]
```

![](https://pic3.zhimg.com/v2-413df5b0e73209213f2b97f5f7cf050e_b.jpg)

4.3. 运行docker-compose.yml文件

Compose 是一个用户定义和运行多个容器的 Docker 应用程序。在 Compose 中你可以使用 YAML 文件来配置你的应用服务。然后，只需要一个简单的命令，就可以创建并启动你配置的所有服务。

使用 Compose 基本会有如下三步流程：

在 Dockfile 中定义你的应用环境，使其可以在任何地方复制。

在 docker-compose.yml 中定义组成应用程序的服务，以便它们可以在隔离的环境中一起运行。

最后运行docker-compose up，Compose 将启动并运行整个应用程序。

![](https://pic4.zhimg.com/v2-982236cf23fa467d18106ba78e1496cf_b.jpg)

在浏览器中输入192.168.25.130:10080/，可以观察到下面的页面，此时GitLab已经搭建成功。

![](https://pic2.zhimg.com/v2-913698a22b1f22a0c6664323bc2e8f0d_b.jpg)

4.4. 初始化密码

初次访问，会弹出下列页面，我们需要为管理员root设置密码，例如12345678。

![](https://pic3.zhimg.com/v2-a5d21d57dd7ef8838541b3ca95bf1122_b.jpg)

4.5. 新建普通用户

![](https://pic3.zhimg.com/v2-4ba4a1b6433003a9bd63789d61a0d5b6_b.jpg)

我们可以为gitlab添加普通用户，切换到register选项卡中，注册新用户。

4.6. 新建项目

登陆之后，我们就可以新建项目了，我们输入项目名，新建即可。

![](https://pic1.zhimg.com/v2-578caba18dd6dcb732ad87647daf7bf8_b.jpg)

![](https://pic4.zhimg.com/v2-a43371d56e19b7d0363faf1a068522af_b.jpg)

**5\. 编写SpringBoot项目**

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。

接下来我们来编写一个最简单的SpringBoot入门项目。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 [url=http://maven.apache.org/xsd/maven-4.0.0.xsd]http://maven.apache.org/xsd/maven-4.0.0.xsd[/url]">
        <modelVersion>4.0.0</modelVersion>
 
        <groupId>com.itheima</groupId>
        <artifactId>springboot_jenkins01</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>jar</packaging>
 
        <!--所有的springboot工程都必须继承spring-boot-starter-parent-->
        <parent>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>2.0.6.RELEASE</version>
                <relativePath/> <!-- lookup parent from repository -->
        </parent>
 
        <properties>
                <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
                <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
                <java.version>1.8</java.version>
        </properties>
 
        <dependencies>
                <!--web功能的起步依赖-->
                <dependency>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-web</artifactId>
                </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        </dependencies>
 
        <build>
                <plugins>
                        <plugin>
                                <groupId>org.springframework.boot</groupId>
                                <artifactId>spring-boot-maven-plugin</artifactId>
                        </plugin>
                </plugins>
        </build>
</project>
```

**5.2. 核心配置文件application.properties**

由于我们是基础入门项目，所以我们只需要新建一个application.properties文件放在resources目录下（当然不放置也是可以的），内容为空即可。

**5.3. Controller类**

```
package com.itheima.controller;
 
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
public class QuickController {
 
    @GetMapping("/quick")
    public String quick(){
        return "欢迎使用 springboot ！Jenkins";
    }
}
```

**5.4. 引导类**

```
package com.itheima;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
 
@SpringBootApplication
public class SpringbootQuickApplication {
 
        public static void main(String[] args) {
                SpringApplication.run(SpringbootQuickApplication.class, args);
        }
}
```

**5.5. 运行项目**

 我们运行引导类的main方法即可运行项目，此时控制台会输出如下日志，注意观察标红的部分，由于我们并没有进行任何的配置，项目自动运行在了端口号为8080的tomcat中，项目名也默认为“”。

![](https://pic3.zhimg.com/v2-aca20ba44da12673e26d3dc922be1f8a_b.jpg)

我们在浏览器中输入"[localhost:8080/quick](https://link.zhihu.com/?target=http%3A//localhost%3A8080/quick)"，可以观察到下图的输出，这说明我们的项目已正常运行。

![](https://pic2.zhimg.com/v2-a09882c91bd55fa48726bca2825dfa95_b.jpg)

**5.6. pom\_docker\_registry.xml**

由于我们接下来会使用Jenkins把项目打包成Docker镜像，我们需要添加一个专门用于Docker的pom文件。内容为如下代码块：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 [url=http://maven.apache.org/xsd/maven-4.0.0.xsd]http://maven.apache.org/xsd/maven-4.0.0.xsd[/url]">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.itheima</groupId>
    <artifactId>springboot_jenkins01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <!--web功能的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>      
    </dependencies>
 
    <build>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <!--docker镜像相关的配置信息-->
                <configuration>
                    <!--镜像名，这里用工程名-->
                    <imageName>${project.artifactId}-${project.version}</imageName>
                    <!--Dockerfile文件所在目录-->
                    <dockerDirectory>
                        ${project.basedir}/src/main/resources
                    </dockerDirectory>
                    <!--TAG,这里用工程版本号-->
                    <imageTags>
                        <imageTag>${project.version}</imageTag>
                    </imageTags>
                    <registryUrl>192.168.25.130:5000</registryUrl>
                    <pushImage>true</pushImage>
                    <imageName>
                        192.168.25.130:5000/${project.artifactId}:${project.version}
                    </imageName>
                    <!--构建镜像的配置信息-->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>
                                ${project.artifactId}-${project.version}.jar
                            </include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**5.7. 配置Dockerfile**

在resources目录下创建Dockerfile文件，该文件用于构建docker镜像，文件名称必须是Dockerfile。

```
FROM java:8
ENV ARTIFACTID springboot_jenkins01
ENV ARTIFACTVERSION 1.0-SNAPSHOT
ENV HOME_PATH /home
WORKDIR $HOME_PATH
ADD /$ARTIFACTID-$ARTIFACTVERSION.jar $HOME_PATH/$ARTIFACTID.jar
ENTRYPOINT ["java", "-jar", "springboot_jenkins01.jar"]
```

这一切都准备完毕之后，我们把项目push到上一节搭建好的GitLab中。

**6\. Jenkins**

Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

**6.1. 安装Jenkins6.1.1. 下载Docker镜像**

```
docker pull jenkinsci/blueocean
```

**6.1.2. 创建Docker容器**

```
docker create --name jenkins -u root -p 8889:8080 --privileged=true -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /home/jenkins:/home docker.io/jenkinsci/blueocean
```

**6.1.3. 启动Docker容器**

**6.1.4. 访问Jenkins**

在浏览器中输入[192.168.25.130:8889](https://link.zhihu.com/?target=http%3A//192.168.25.130%3A8889/)，可以观察到下面的页面，第一次启动需要耐心的等待，时间比较久，另外还需要解锁密码，密码在下图标红的位置。

![](https://pic1.zhimg.com/v2-4b180418734acb385ef29323e0704228_b.jpg)

**6.1.5. 进入容器查看密码**

```
docker exec -it jenkins /bin/bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

![](https://pic4.zhimg.com/v2-9fae6f2a582ff77a02a3632cfcaca23b_b.jpg)

**6.2. 配置Jenkins6.2.1. 插件安装**

进入Jenkins中，我们可以进行插件的安装，先选择下图第一个选项即可。

![](https://pic4.zhimg.com/v2-46453d9f56990434a463794dd62e24bb_b.jpg)

**6.2.2. 创建用户**

![](https://pic4.zhimg.com/v2-89fb66947e9b7c5841cd8e7334a80f13_b.jpg)

**6.2.3. 保存进入**

![](https://pic3.zhimg.com/v2-13bb517fde47a7ebe020fa453a85086e_b.jpg)

**6.2.4. 设置JDK、Maven、Git**

进入【系统管理】，点击【全局工具配置】，设置JDK、Git和Maven，我们可以选择自动安装。

![](https://pic3.zhimg.com/v2-d20b1a440a2272ed3091e12e18015976_b.jpg)

![](https://pic4.zhimg.com/v2-e4cac3745924f701ebe447735dcb7e7b_b.jpg)

![](https://pic1.zhimg.com/v2-a502060a03eae911d0dea1193640339c_b.jpg)

**6.2.5. 设置全局凭据**

根据下图我们设置下全局的凭据，注意此处要设置的是当前CentOS7的用户名和密码

![](https://pic4.zhimg.com/v2-3958427ccd7a0725c9f2e35ce11f1427_b.jpg)

![](https://pic1.zhimg.com/v2-bb190213fd05208c360835c71b371228_b.jpg)

![](https://pic2.zhimg.com/v2-9867621af2c8f5cfaef72b5d63d5208d_b.jpg)

**6.3. 创建Jenkins任务6.3.1. 输入任务名称，选择【构建一个自由风格的软件项目】**

![](https://pic4.zhimg.com/v2-98c1a6591804e80d66f66bf71906a3a3_b.jpg)

**6.3.2. 设置GitLab中项目的地址**

![](https://pic2.zhimg.com/v2-2711cf9177b3295264905f93c564b3a5_b.jpg)

**6.3.3. 添加构建脚本**

![](https://pic2.zhimg.com/v2-47dd1a2a99aafbb6e229103809944f85_b.jpg)

![](https://pic1.zhimg.com/v2-04a3ef27a2db6701105a262965207c3c_b.jpg)

脚本内容：

```
#!/bin/bash
result=$(docker ps | grep "192.168.25.130:5000/springboot_jenkins01")
if [[ "$result" != "" ]]
then
echo "stop springboot_jenkins01"
docker stop springboot_jenkins01
fi
result1=$(docker ps -a | grep "192.168.25.130:5000/springboot_jenkins01")
if [[ "$result1" != "" ]]
then
echo "rm springboot_jenkins01"
docker rm springboot_jenkins01
fi
result2=$(docker images | grep "192.168.25.130:5000/springboot_jenkins01")
if [[ "$result2" != "" ]]
then
echo "192.168.25.130:5000/springboot_jenkins01:1.0-SNAPSHOT"
docker rmi 192.168.25.130:5000/springboot_jenkins01:1.0-SNAPSHOT
fi
```

**6.3.4. 添加顶层Maven目标**

![](https://pic1.zhimg.com/v2-e11c72efaff932c3acbc3748ac1d58b8_b.jpg)

![](https://pic4.zhimg.com/v2-6d298ec7674c9f2ac093a9d5e9ffe7db_b.jpg)

```
clean package -f pom_docker_registry.xml  -DskipTests docker:build
```

**6.3.5. 添加Shell执行脚本**

![](https://pic4.zhimg.com/v2-152f8caadbdebb58a611c2fd53999a07_b.jpg)

![](https://pic3.zhimg.com/v2-c05165bc98e2aec8f5953d751bdfa9be_b.jpg)

脚本内容

```
docker run --name springboot_jenkins01 -p 50101:50101 -idt 192.168.25.130:5000/springboot_jenkins01:1.0-SNAPSHOT 
docker logs -f springboot_jenkins01
```

**6.4. 执行任务**

点击项目的左侧菜单中的【立即构建】，可以观察到下面的日志内容，日志太长，我分段给大家截取：

-   从gitlab中拉取源码

![](https://pic3.zhimg.com/v2-2137a3c6d09354cc59897f3b700ac9da_b.jpg)

2.停止并删除之前运行的容器

![](https://pic3.zhimg.com/v2-59e4d48888fb35a97b1c92c56c0e2d3e_b.jpg)

3.通过Maven构建Docker镜像

![](https://pic3.zhimg.com/v2-d361ea3f3467f9f87f1c12f0e10a2522_b.jpg)

4.创建并运行Docker容器

![](https://pic1.zhimg.com/v2-63f6c718005d1b9b102b6761565024d4_b.jpg)

此时在浏览器中访问[192.168.25.130:28080/quick](https://link.zhihu.com/?target=http%3A//192.168.25.130%3A28080/quick)，可以看到以下页面，这也正是我们最初在本机运行时看到的内容

![](https://pic1.zhimg.com/v2-d0c877eab6c6091395c3ce139c977544_b.jpg)

**6.5. 配置钩子**

到此为止，我们已经实现了运行Jenkins任务，自动执行GitLab的代码，但是我们还无法实现当GitLab中的代码发生变化时，Jenkins自动构建任务的功能。接下来，我们配置下WebHook，来实现该功能。

**6.5.1. 安装GitLab插件**

![](https://pic2.zhimg.com/v2-d1b3ffe08aabda3822ee9e312b613d49_b.jpg)

**6.5.2. 安装GitLab WebHook插件**

![](https://pic3.zhimg.com/v2-f9bbd1816a957808d4baefe3ab50f72e_b.jpg)

**6.5.3. Jenkins中配置Build Triggers**

打开我们在Jenkins的任务进行设置，勾选Build Triggers，选择下面标红的复选框，并且复制其中的超链接。

![](https://pic4.zhimg.com/v2-a6ec09861daeda89310790db36df035f_b.jpg)

**6.5.4. GitLab配置webhook**

打开GitLab中我们的项目，点击设置中的Integrations，在URL中填入我们刚刚在Jenkins中粘贴的超链接。

![](https://pic3.zhimg.com/v2-e7c789426998e8d3c8d4b2e32791638e_b.jpg)

**6.5.5. 测试webhook**

添加完webHook之后，我们点击test按钮，进行推送测试。

![](https://pic4.zhimg.com/v2-f1be8b017ac314146157e265dadb6b03_b.jpg)

观察Jenkins中，会发现Jenkins已经自动执行了该任务。这说明我们的webhook也已经正确配置了。

![](https://pic4.zhimg.com/v2-cd2a852d8e6cc82d9a0ed9afd8475d1b_b.jpg)

**7\. 总结**

经过我们多款软件的安装配置，我们逐步掌握了如何上传SpringBoot项目到GitLab中，并使用Jenkins自动构建任务，另外依托于Docker，让这一切变得更加方便，希望大家都多多思考，让机器自动干活，减少我们IT从业人员的重复、繁琐的工作量。

来源：[【郑州校区】SpringBoot+GitLab+Docker+Jenkins实现持续集成 四-黑马程序员技术交流社区](https://link.zhihu.com/?target=http%3A//bbs.itheima.com/forum.php%3Fmod%3Dviewthread%26tid%3D505140)