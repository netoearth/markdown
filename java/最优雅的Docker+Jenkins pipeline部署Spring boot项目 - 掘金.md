    

[![](https://p3-passport.byteimg.com/img/mosaic-legacy/3797/2889309425~100x100.awebp)](https://juejin.cn/user/2013961031530344)

2019年09月29日 16:39 ·  阅读 6712

使用Jenkins发布项目已经4年多，做过多次改进，尽量减少Jenkins的配置和发布脚本的书写。从最初的`构建一个自由风格的软件项目`到`流水线项目`到现在的`流水线+docker`，流水线可以让项目发布流程更加清晰，`docker`可以大大减少`Jenkins`配置。

**下面带大家使用最简单，最快速的方式使用Jenkins，无需配置git、maven、jdk、node等环境，一切用docker搞定，只需要写脚本，其他无需配置**

此教程仅为进阶教程，具体的jenkins安装，git凭据配置，邮件发送等需要自行查找教程进行配置。

### 本教程使用的环境

-   centos7
-   docker-ce 18.09.1(务必使用docker-ce，老版本的docker会导致Jenkins中无法使用docker daemon)

### 准备工作

#### 安装docker

`yum install docker-ce`

#### 配置docker加速器（可选）

`很多厂商都提供docker加速服务，比如阿里云，腾讯云 详情请自行百度。不配置国内加速器，下载镜像可能很缓慢。`

#### 使用docker安装Jenkins

`docker pull jenkins/jenkins，切勿docker pull jenkins已经废弃`

[Jenkins docker hub地址](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2Fr%2Fjenkins%2Fjenkins "https://hub.docker.com/r/jenkins/jenkins")

#### 启动Jenkins

```
docker run -u root -itd --name jenkins -p 6001:8080 -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -e TZ="Asia/Shanghai" -v /etc/localtime:/etc/localtime:ro -v /volume1/docker/jenkins:/var/jenkins_home jenkins/jenkins
复制代码
```

##### 启动命令含义

`-p 6001:8080`Jenkins默认网页访问端口为8080，将端口映射到外部主机

`-v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock`使Jenkins内部可以使用docker命令

`-e TZ="Asia/Shanghai" -v /etc/localtime:/etc/localtime:ro`配置Jenkins容器的时区

`-v /volume1/docker/jenkins:/var/jenkins_home` 将Jenkins的配置映射到外部主机卷，容器删除仍可保留配置

#### 进入Jenkins容器内部，判断docker命令是否正常执行

`docker exec -it jenkins bash`

`docker info`

可能需要安装libltdl7，如果需要执行以下命令

`apt-get update`

`apt-get install -y libltdl7`

### 配置Jenkins

`http://主机IP：6001`可访问Jenkins网页端，其他流程忽略，全部默认就可以。

无需配置git、maven、jdk、node等环境，我们使用docker解决

#### 新建一个流水线项目

![image](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/29/16d7c2ac9c78bd31~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

[关于流水线脚本书写，有两种写法，官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fjenkins.io%2Fdoc%2Fbook%2Fpipeline%2F "https://jenkins.io/doc/book/pipeline/")

[Using Docker with Pipeline，官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fjenkins.io%2Fdoc%2Fbook%2Fpipeline%2Fdocker%2F "https://jenkins.io/doc/book/pipeline/docker/")

#### 编写脚本

![image](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/29/16d7c2b217ee82e6~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

##### 分享我自己项目的发布脚本，适用于spring boot项目，细节需要自己修改。此脚本需要使用Dockfile，参考下一节

```
import java.text.SimpleDateFormat
node {
    try {
        //名字这么写是为了可以发布到腾讯docker仓库，可随意更改
        def dockerId='tengxun'
        def dockerUrl='ccr.ccs.tencentyun.com'
        def dockerNamespace='esms'
        def dockerName='esms-api'
        def env='test'
        
        def dateFormat = new SimpleDateFormat("yyyyMMddHHmm")
        def dockerTag = dateFormat.format(new Date())
        
        stage('git pull'){
            sh 'pwd'
            git credentialsId: '此处填写git凭证，需要在Jenkins凭据中配置', url: '此处填写git地址'
        }
        stage('mvn install') {
            sh 'pwd'
            docker.image('maven:3.6.0-jdk-8-alpine').inside('-v /volume1/docker/.m2:/root/.m2') {
                sh 'mvn --version'
                sh 'mvn clean install'
            }
        }
        stage('docker run') {
            dir("esms-main") {
                sh 'pwd'
                def imageUrl = "${dockerUrl}/${dockerNamespace}/${dockerName}:${dockerTag}"
                def customImage = docker.build(imageUrl)
                sh "docker rm -f ${dockerName} | true"
                //--network esms-net配置网络信息，需要先docker network create esms-net，用于多个服务交互，可选
                customImage.run("-it -d --name ${dockerName} -p 8090:8090 --network esms-net -e SPRING_PROFILES_ACTIVE=${env}")
                //only retain last 3 images，自动删除老的容器，只保留最近3个
                sh """docker rmi \$(docker images | grep ${dockerName} | sed -n  '4,\$p' | awk '{print \$3}') || true"""
            }
        }
        currentBuild.result="SUCCESS"
    } catch (e) {
        currentBuild.result="FAILURE"
        throw e
    } finally {
        //此处若想发布邮件，需要在系统管理-系统设置中配置邮件服务器
        mail to: 'xxxx@qq.com',subject: "Jenkins: ${currentBuild.fullDisplayName}，${currentBuild.result}",body:"${currentBuild.result}"
    }
}
复制代码
```

##### 最终发布以后docker images查看，名称格式类似如下，如使用上面的代码发布后名称为

`ccr.ccs.tencentyun.com/esms/esms-api`

![image](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/29/16d7c2c2e8d0ebb4~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

### Sprint Boot项目结构以及Dockerfile

![image](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/29/16d7c2bd6aed7971~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

#### Dockerfile书写，仅做参考，具体根据自己项目结构

```
FROM openjdk:8-jdk-alpine

MAINTAINER liunewshine@qq.com

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && mkdir -p /esms

WORKDIR /esms

EXPOSE 8090

ADD ./target/esms.jar ./

CMD java -Djava.security.egd=file:/dev/./urandom -jar esms.jar
复制代码
```

### 发布项目

直接构建即可，未做git的webhook，不能联动git push等，有需求的可以自己加上

![image](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/29/16d7c2bf9d107551~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

### 总结

总体过程很简单，主要就是安装Jenkins->编写pipeline脚本->编写Dockerfile脚本。

相关课程

![「DDD 案例实战课」封面](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8de725a2865f435382aa44985dd9217f~tplv-k3u1fbpfcp-no-mark:420:420:300:420.awebp?)

![「SkyWalking：应用监控和链路跟踪」封面](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64886f6ba6e94fb982ed744d4873f1a1~tplv-k3u1fbpfcp-no-mark:420:420:300:420.awebp?)

SkyWalking：应用监控和链路跟踪

[车辙cz ![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")](https://juejin.cn/user/1134351732184008)   

759购买

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 20

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏