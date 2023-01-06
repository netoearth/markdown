## 转载自:[https://my.oschina.net/yimingkeji/blog/2878371](https://my.oschina.net/yimingkeji/blog/2878371)

## 安装[jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020)

## Jenkins介绍

```
Jenkins是一个广泛用于持续构建的可视化web工具，可用于自动化与构建、测试、交付或部署软件相关的各种任务。可以通过安装包、tomcat、java、docker方式进行安装使用
```

官方地址：[https://jenkins.io/](https://jenkins.io/)

## Jenkins下载安装

这里通过tomcat来启动Jenkins

### 下载

到官网[https://jenkins.io/download/](https://jenkins.io/download/) 下载war包 ![](https://oscimg.oschina.net/oscnet/02c37cec60bef11382872ca4b5d36a30a41.jpg)

### 部署到tomcat

复制war包到tomcat的webapps目录下，启动tomcat ![](https://oscimg.oschina.net/oscnet/4e469ac8e0210cecf9301b4b4aa98172bbf.jpg)

### 解锁

这里提示解锁Jenkins，复制 `/Users/Shared/Jenkins/Home/secrets/initialAdminPassword`文件内容到页面

```
$ cat /Users/Shared/Jenkins/Home/secrets/initialAdminPasswordxxxxxxxxx
```

### 设置用户

解锁成功后，跳转到设置用户页面 ![](https://oscimg.oschina.net/oscnet/e80c9a483b448e0f6fc8f9d6c0c0e1f535a.jpg)

这里设置 admin / admin

### 安装默认插件

然后提示安装插件，选择默认的安装，等待安装完成。有些插件可能安装失败，提示要求升级最新版本Jenkins，升级成功后可以安装成功。

### 全局配置

点击 >系统设置>全局工具配置 ![](https://oscimg.oschina.net/oscnet/31fd75e32047ec7fe9758622fd858cd49fb.jpg)

配置maven、JDK、[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)、git ![](https://oscimg.oschina.net/oscnet/7a31f1931226fa58f6bce6583c667a2776a.jpg) ![](https://oscimg.oschina.net/oscnet/16abf80eccc694662fd699248a9b8d55ff1.jpg)

## 安装docker

## springboot项目

## 代码

项目地址：[https://gitee.com/yimingkeji/docker-springboot](https://gitee.com/yimingkeji/docker-springboot)

## Dockerfile内容：

```
#FROM openjdk:8-jdk-alpineFROM hub.c.163.com/dwyane/openjdk:8VOLUME /tmpADD docker-springboot-1.0-SNAPSHOT.jar app.jarENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

然后上传到自己的git地址

## 使用Jenkins部署项目到docker

在jenkins页面添加`自由风格的软件项目` ![](https://oscimg.oschina.net/oscnet/bf6fd6bd3924a03d3312a70d45fd19c0a70.jpg)

在`源码管理`里添加git仓库和用户名、密码配置，并且选择代码分支（这里是master）

![](https://oscimg.oschina.net/oscnet/7b4fe92faa8adce33cced74cfa11d59d0b1.jpg)

在`构建`步骤中，添加2个步骤

-   1.`顶级maven` 选择maven版本（上面的全局配置中配的maven），添加maven打包命令

```
clean install -Dmaven.test.skip=true
```

-   2.`执行shell` 添加shell：

```
mvn docker:buildecho "当前docker 镜像："docker images | grep dockerspringbootecho "启动容器----->"docker run -p 8001:8001 -d dockerspringbootecho "启动服务成功！"
```

![](https://oscimg.oschina.net/oscnet/6b79805e477c4c58d1ce3f260b928e1c2d6.jpg)

## 执行构建并启动服务

上面配置完成后，到Jenkins主页，选择配置好的项目，菜单中点击`立即构建`

![](https://oscimg.oschina.net/oscnet/2ead8591a2b57a295402197783743af40f4.jpg)

看到左边菜单里有执行的进度条，点进去后会看到执行日志 ![](https://oscimg.oschina.net/oscnet/0a43ead275619b1022d4cd65c8d7af8c298.jpg)

```
构建中 在工作空间 /Users/yangzhenlong/.jenkins/workspace/docker-springboot 中 > /usr/bin/git rev-parse --is-inside-work-tree # timeout=10... > /usr/bin/git checkout -f de05bbef05fae5ade0c79ce1d49154723d84fa88Commit message: "修改dockerfile" #git中最后的提交记录 > /usr/bin/git rev-list --no-walk 330c170c5829017d00942d7301834c2a196a29ac # timeout=10[docker-springboot] $ /Users/xxxx/apache-maven-3.5.4/bin/mvn -s /Users/xxx/apache-maven-3.5.4/conf/settings.xml -gs /Users/xxx/apache-maven-3.5.4/conf/settings.xml clean install -Dmaven.test.skip=true # 执行maven脚本[INFO] Scanning for projects......[INFO] ------------------------------------------------------------------------[INFO] BUILD SUCCESS[INFO] ------------------------------------------------------------------------[INFO] Total time: 3.412 s[INFO] Finished at: 2018-11-19T18:01:54+08:00[INFO] ------------------------------------------------------------------------[docker-springboot] $ /bin/sh -xe /Users/yangzhenlong/tools/tomcat8-jenkins/temp/jenkins7206360448776968606.sh+ mvn docker:build #构建成功后，执行shell脚本开始...[INFO] Scanning for projects......[INFO] Copying /Users/yangzhenlong/.jenkins/workspace/docker-springboot/src/main/docker/Dockerfile -> /Users/yangzhenlong/.jenkins/workspace/docker-springboot/target/docker/Dockerfile[INFO] Building image dockerspringboot #执行Dockerfile脚本开始...Step 1/4 : FROM hub.c.163.com/dwyane/openjdk:8 ---> 96cddf5ae9f1Step 2/4 : VOLUME /tmp ---> Using cache ---> 2e62fad6a16aStep 3/4 : ADD docker-springboot-1.0-SNAPSHOT.jar app.jar ---> 2d1319472a12Step 4/4 : ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] ---> Running in f5d8c06b3b7dRemoving intermediate container f5d8c06b3b7d ---> 3f24ff705b46ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}Successfully built 3f24ff705b46Successfully tagged dockerspringboot:latest[INFO] Built dockerspringboot[INFO] ------------------------------------------------------------------------[INFO] BUILD SUCCESS[INFO] ------------------------------------------------------------------------[INFO] Total time: 4.841 s[INFO] Finished at: 2018-11-19T18:02:00+08:00[INFO] ------------------------------------------------------------------------+ echo '当前docker 镜像：' # dockerFile构建镜像成功后，shell脚本打印docker镜像当前docker 镜像：+ docker images + grep dockerspringbootREPOSITORY                     TAG                 IMAGE ID            CREATED             SIZEdockerspringboot               latest              3f24ff705b46        1 second ago        657MB+ echo '启动容器----->'启动容器-----> #这里的中文会乱码，后面建议使用英文+ docker run -p 8080:8080 -d dockerspringboote0aeafb47410a77f30bcb24b744b49dd57b251594a68da997defeef24c256f2b+ echo $'�\220��\212��\234\215�\212��\210\220�\212\237�\201'启动服务成功！Finished: SUCCESS
```

然后浏览器访问 localhost:8080 ![](https://oscimg.oschina.net/oscnet/5ebad3f2bd7681fdc8fa078f0fb6b2e9b15.jpg)

## 改造shell脚本

如果下次构建该项目的时候，docker镜像和服务已存在，需要先删除镜像和服务

```
echo "remobe old container"docker ps -a | grep dockerspringboot | awk '{print $1}'| xargs docker rm -fecho "romove old image"docker rmi dockerspringbootmvn docker:buildecho "current docker images"docker images | grep dockerspringbootecho "start container"docker run -p 8001:8001 -d dockerspringbootecho "current container"docker ps -a | grep dockerspringbootecho "star service success!"
```

输出日志：

```
+ echo 'current docker images'current docker images+ docker imagesREPOSITORY                     TAG                 IMAGE ID            CREATED                  SIZEdockerspringboot               latest              8c9f93a324c0        Less than a second ago   657MB<none>                         <none>              3f24ff705b46        2 hours ago              657MB<none>                         <none>              e276f4ef3cc5        2 days ago               657MB<none>                         <none>              02b677ad12e1        2 days ago               657MBnginx                          latest              dbfc48660aeb        4 weeks ago              109MBmongo                          latest              a41c82c0998a        2 months ago             380MBredis                          3.2                 e2e6164a20de        4 months ago             76MBhub.c.163.com/library/nginx    latest              46102226f2fd        19 months ago            109MBmysql                          5.7                 9e64176cd8a2        19 months ago            407MBhub.c.163.com/library/mysql    5.7                 9e64176cd8a2        19 months ago            407MBhub.c.163.com/dwyane/openjdk   8                   96cddf5ae9f1        2 years ago              641MB+ echo 'start container'start container+ docker run -p 8001:8001 -d dockerspringboot0b79b907e9ca49831bd9259eeea2c60ec40132342b2ed3c00d4cf183756d6610+ echo 'current container'current container+ docker ps -a+ grep dockerspringboot0b79b907e9ca        dockerspringboot              "java -Djava.securit…"   1 second ago        Up Less than a second      0.0.0.0:8001->8001/tcp     happy_johnson+ echo 'star service success!'star service success!Finished: SUCCESS
```