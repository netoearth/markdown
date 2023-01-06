## Docker安装Jenkins自动部署SpringBoot项目

根据之前文章《[使用Docker安装好Jenkins](https://www.yanxizhu.com/index.php/archives/138/)》为前提搭建好Jenkins，不明白请看[https://www.yanxizhu.com/index.php/archives/138/](https://www.yanxizhu.com/index.php/archives/138/)。

环境说明：jenkins为docker部署，Docker+Jenkins+Gitee+JDK11+Maven3.8.5。

**以后每次改动代码，push提交到giee码云后会自动部署，不用手动点击部署。**

## 一、全局工具配置

【首页】-【系统管理】-【全局工具配置】

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092255415.png)

我之前启动jenkins容器映射参数如下，根据自己映射路径自行修改。

```
docker run -p 10240:8080 -p 10241:50000 --name jenkins \
-u root \
-v /mydata/jenkins_home:/var/jenkins_home \
-v /mydata/maven/apache-maven-3.8.5:/maven/apache-maven-3.8.5 \
-v /mydata/jdk/jdk-11.0.10/:/jdk/jdk-11.0.10 \
-v /mydata/maven/repo:/mydata/maven/repo \
-v /usr/bin/docker:/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
-d jenkins/jenkins:lts
```

**上面很重要，注意。**

1.  jdk配置

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092253058.png)

```
jdk11
```

路径

```
/jdk/jdk-11.0.10
```

1.  maven配置

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092254719.png)

```
maven3.8.5
```

路径

```
/maven/apache-maven-3.8.5
```

1.  git配置

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092254156.png)

```
Default
```

路径

```
/usr/bin/git
```

1.  docker配置

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092255382.png)

```
docker
```

路径

```
/usr/bin
```

注意点：

1、jenkins容器里面自带git，可通过命令查看路径。

2、注意自己jdk、mavn、docker安装路径。

查看jenkins自带git路径命令：

```
which git
```

## 二、插件安装

【首页】-【系统管理】-【插件管理】

插件1：

```
Publish Over SSH
```

插件2：

```
Gitee Plugin
```

[如果插件安装慢，可以修改源，请参考修改方案](https://www.yanxizhu.com/index.php/archives/138/)，[https://www.yanxizhu.com/index.php/archives/138/](https://www.yanxizhu.com/index.php/archives/138/)

注意：如果在【全局工具配置】没有对应的选项，就是缺少相应插件。

## 三、系统配置

1、SSH remote hosts配置

新增加配置ssh登陆凭证，此步骤的主要作用是jenkins 打包镜像后，能够远程去登陆和执行脚本文件。

Hostname：`xxx.xxx.xxx.x..`(需要登陆的服务器ip)  
Port：`22`（ssh登陆端口）  
Credentials：登陆账号和密码（此处点击`[添加]`按钮增加一个）

如果是本机可以不用配置

2、Gitee 配置

链接名：

```
gitee
```

Gitee 域名 URL：

```
https://gitee.com
```

添加凭证

Gitee API V5 的私人令牌（获取地址 [https://gitee.com/profile/personal\_access\_tokens](https://gitee.com/profile/personal_access_tokens)）

通过上面连接创建一个令牌，然后添加到这里。

## 四、准备项目

1、本地新建一个SpringBoot项目，新建HellocerConller控制层

```
package com.yanxizhu.jenkins.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @description: Jenkins自动部署测试
 * @author: <a href="mailto:batis@foxmail.com">清风</a>
 * @date: 2022/5/8 17:30
 * @version: 1.0
 */
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

本地启动项目确保通过127.0.0.1:8080/hello能够访问。

2、编写Dockerfile

```
# 指定是基于哪个基础镜像
FROM openjdk:11

# 作者信息
MAINTAINER batis

# 挂载点声明
VOLUME /tmp

# 将本地的一个文件或目录，拷贝到容器的文件或目录里
ADD /target/jenkins-demo-0.0.1-SNAPSHOT.jar springboot.jar

#shell脚本
RUN bash -c 'touch /springboot.jar'

# 将容器的8000端口暴露，给外部访问。
EXPOSE 8000

# 当容器运行起来时执行使用运行jar的指令
ENTRYPOINT ["java", "-jar", "springboot.jar"]
```

注意：修改jdk版本、打包后名称、端口信息

## 五、代码上传

登录码云新建仓库，名字随意将代码关联并提交到码云。新建仓库、代码push自行google。

## 六、WebHooks 管理配置

1、打开仓库 -> 管理 -> 右侧的webhooks

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092316803.png)

URL：填入服务器公网IP地址

WebHook密码通过以下生成。

## 七、部署SpringBoot项目

## 1、新建部署任务

任务名字随意、构建一个自由风格的软件项目。

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092318382.png)

## 2、General

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092325615.png)

描述随意填写，丢弃旧的构建策略,保持构建的天数1，保持构建的最大个数3，根据自己需要自行修改。

## 3、源码管理

选择git

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092349200.png)

Repository URL为自己gitee码云仓库地址。

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092323296.png)

Credentials点击“添加”，Credentials凭证，选择通过用户名密码添加，id、备注可以为空。

## 4、构建触发器

其它默认：

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092332649.png)

找到Gitee WebHook 密码，点击“生成按钮”生成，然后将该密码填入上面 “六、WebHooks 管理配置”中。

![](https://cdn.jsdelivr.net/gh/willxwu/CDN@main/images/202205092333956.png)

轮询 SCM策略：

```
* * * * *
```

注意：\*中间有空格，**当您输入 "\* \* \* \* \*" 时，意思为"每分钟"？也许您希望 "H \* \* \* \*" 每小时轮询**。

## 5、构建

选择执行shell脚本

```
#!/bin/bash -lex
docker rm -f app_docker
sleep 1
docker rmi -f app_docker:1.0
sleep 1
mvn clean install -Dmaven.test.skip=true
sleep 1
docker build -t app_docker:1.0 -f ./src/main/Dockerfile .
sleep 1
docker run -d -p 8000:8000 --name app_docker app_docker:1.0 
```

注意自己端口名称。

## 6、访问测试

通过自己xxx.xxx.xx.xx:8000/hello即可自己写的helloword了。

#### **以后每次改动代码，push提交到giee码云后会自动部署，不用手动点击部署。**

## 7、问题记录及解决方案

比如：

1、查不到mvn、docker、jdk命令，可能是jenkins容器中环境配置问题，可以参考[《Jenkins容器docker部署springboot项目-问题记录》](https://www.yanxizhu.com/index.php/archives/139/)

2、如果开启了防火墙注意开发相应端口或关闭防火墙

3、部署遇到问题，查看部署日志，以及google、baidu

相关参考：

[Docker开启Remote API访问](https://www.yanxizhu.com/index.php/archives/135/)

[docker启动Jenkins报错](https://www.yanxizhu.com/index.php/archives/136/)

[Docker安装Jenkins](https://www.yanxizhu.com/index.php/archives/138/)

[Nginx配置Jenkins二级域名，以及443 SSL证书访问](https://www.yanxizhu.com/index.php/archives/137/)

[Jenkins容器docker部署springboot项目-问题记录](https://www.yanxizhu.com/index.php/archives/139/)