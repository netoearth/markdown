## 前言

  之前写过一遍比较粗略的jenkins + docker部署文章，这次有时间，认真的写一遍比较详细完整的jenkins +docker部署文章，由于有时间所以这次就多写一点吧，记录下我自己对docker的看法，以及它的作用，若有不对之处还请指出。

### Docker的作用

  其实一般的小型项目是用不上也没有必要使用docker的，docker的作用是资源隔离以及快速部署，在项目比较小的时间我们完全可以手动上传jar/war 包到服务器上，并输入命令启动即可，但是，想像一下如果你用上了微服务，好几个团队开发，那么这些服务加起来可能有好几十个，这个时候再手动去部署，想想看你要做什么？  1、安装 JDK    2、配置环境变量   3、安装一些必要的软件如TOMCAT之类的   4、等等不止这些，有些服务甚至有可能要修改JDK配置。

   每部署一台机器都要做如此多的操作是不是得烦死？如果是这种情况下，那么docker就可以派上用场了，我们可以直接把安装好各种软件以及配置的系统环境打包成一个docker镜像并保存在某个镜像中心，然后其它服务器只需几行命令就可以轻松使用打包好的环境并完成部署了，而且由于各个docker容器之间是隔离了，一台机器可以部署多种镜像也不会有影响。

### 有感而发

  最近网上看到很多老程序员们在谈论中年危机，仿佛看到了几年后的自己，我自认为自己也不属于那无可替代的1%，而且自己也确实感觉到了行情不好（如工资下降，面试变少等），感觉自己有可能以后会不再干程序员，希望留下来的文章能够帮到其他人吧，好了，下面开始正题。

## 一、安装docker

  最好将jenkins和docker安装在同一台服务器上，当然不是同一台服务器也可以，只不过要多一个步骤，docker安装步骤如下，以centOs为例：

 卸载旧版本：

```
sudo yum remove docker \                  docker-client \                  docker-client-latest \                  docker-common \                  docker-latest \                  docker-latest-logrotate \                  docker-logrotate \                  docker-engine
```

设置docker存储库：

```
sudo yum install -y yum-utils \  device-mapper-persistent-data \  lvm2
```

```
sudo yum-config-manager \    --add-repo \    https://download.docker.com/linux/centos/docker-ce.repo
```

安装docker:

```
sudo yum install docker-ce docker-ce-cli containerd.io
```

启动docker:

```
sudo systemctl start docker
```

测试docker:

```
sudo docker run hello-world
```

看到打印hello world! 就表示成功了。

)

如果你的服务器是其它系统，可以参考官方教程：[https://docs.docker.com/install/](https://docs.docker.com/install/)  ，找到 SERVER 一栏点击对应的系统查看教程。

## 二、安装jenkins

1、直接打开 [http://mirrors.jenkins.io/war-stable/latest/jenkins.war](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)  下载。

2、或者去官网选择想要的版本：[https://jenkins.io/zh/download/](https://jenkins.io/zh/download/)  ，以war包为例，下载后上传到服务器上，使用java -jar jenkins.war --httpPort=你想要的端口  来启动

3、注意第一次启动控制台会有个初始密码，记下来（如果没有，使用cat /var/lib/jenkins/secrets/initialAdminPassword 命令查看），然后打开jenkins操作页面: 服务ip:启动时设置的端口  .

4、输入初始密码然后设置admin密码，然后安装默认的初始插件即可。

## 三、配置jenkins

1、进入jenkins主页面后点系统管理 -> 插件管理 通过搜索功能分别安装 [Maven Integration](https://wiki.jenkins.io/display/JENKINS/Maven+Project+Plugin) 、[Publish Over SSH](https://wiki.jenkins-ci.org/display/JENKINS/Publish+Over+SSH+Plugin)  、[GitLab](https://wiki.jenkins-ci.org/display/JENKINS/GitLab+Plugin) 、[Git](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin)

2、配置jenkins的git，点击系统管理 -> 全局工具配置 找到git 勾上自动安装输入名字，建议手动安装git不要用jenkins的，gitlab的安装自行搜索。

3、配置远程服务器（jenkins和docker在同一台服务器的可忽略），点击系统管理 -> 系统设置 ，找到 Publish over SSH ，分别 配置name （名字随意起）、 hostname（服务器IP）、Remote Directory（传送文件时将要放置的目录 ），记得勾上Use password authentication, or use a different key。

4、回到jenkins主界面，点击新建任务，

  1）在generel选项卡下勾选丢弃旧的构建，要保留的个数自己填

  2）在源码管理下选择GIt,repository Url 为git的url地址，如：[http://xxx/project/bss.git](http://101.37.150.146/xiahui/sx-global-master.git)   ,Credentials 是gitlab的账号密码，点击旁边的添加按钮，在弹出来的框中选择类型为username with password ，分别输入gitlab的账号和密码点击确定，然后在Credentials选中刚才添加的账号密码，  Branch Specifier 是gitlab的分支，填 \*/master就好

  3）在build选项卡下填写Root POM 和  Goals and options ，root pom是你的项目pom.xml文件的位置，Goals and options 是maven执行命令，可以填 clean install -DskipTests 

  4) 找到 Post Steps 点击 Add post-build step 选择send files or execute commands over ssh ,在name选项中选择第3步添加的服务器然后填写Transfers 里的内容，需要填的内容分别是：

      Source files(要发送的文件，可以填： \*\*/ \*.jar ，以逗号隔开) ，建议如下填写： 

```
**/system-service-*.jar,docker-build.sh,Dockerfile
```

   将jar包、docker构建命令、以及Dockerfile文件都传输过去。

  Remote directory (会新建一个文件夹并将文件放到该文件夹下)

      Exec command （传输完毕后要执行的sheel命令）,输入如下命令： 

```
cd /配置jenkins ssh服务器时配置的目录/上一步发送文件配置的远程目录bash docker-build.sh "bss-*.jar"   #后面的参数是指定只打包匹配的jar
```

   上面的docker-build.sh在后面的步骤中完成，先这么配置好，填完后点击保存.

PS:这一步如果docker和jenkins装在同一台服务器就可以省略，如果不是同一台服务器就需要这一步把jar包发送到远程服务器上再通过docker命令打包，如果在本机打包docker镜像则直接选择“执行shell”，然后输入bash ${WORKSPACE}/docker-build.sh 即可(${WORKSPACE}是jenkins的变量，表示项目构造目录，后面我们需要把docker-build.sh放在项目目录中).

## 四、编写docker构建shell命令

回到代码编辑器，在项目根目录新建docker-build.sh，输入如下代码：

```
#!/bin/bash -ilexexport BUILD_ID=dontKillMesource /etc/profile#此变量用来对应每个子项目的映射对应端口，避免随机映射端口导致服务无法访问declare -A map=(["system-service"]="8014:8014" ["server-2"]="8006:8006" ["sys-service"]="8206:8206")function buildAndRun(){#遍历文件夹获取jarfor file in `find  * -name "${1}"`do#Jenkins中编译好的jar名称     jar_name=${file##*/}#dockerfile位置    dockerfile="./"echo "dockerfile: ==> $dockerfile"    catName="${jar_name%-*.*.*-SNAPSHOT.jar}"#截取jar包的名称作为镜像名    IMAGE_NAME="你的dockerhub账号/$catName"    CONTAINER_NAME="$catName"# 构建Docker镜像    docker build -t $IMAGE_NAME "$dockerfile"# 推送Docker镜像#docker push $IMAGE_NAME   #docker login 再push# 判断镜像存在才启动容器    cid=$(docker images | grep $IMAGE_NAME |awk '{print $3}')if [ x"$cid" != x ]then#获取map中的值     --entrypoint=["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar","--spring.profiles.active=test","--server.port=${START_PORT}"] --expose=${START_PORT}    PORT_VAL=${map[$catName]}#截取：前的端口号作为启动端口    START_PORT=${PORT_VAL%%:*}# 启动Docker容器 大写P表示随机端口, --link分别是eureka、gateway的IP和端口，如果有多个eureka配置host并使用hostname.-link 可以让两个容器之间互相通信             -v xx:xx 设置映射目录，这样容器被删除该文件夹下日志也不会被删    RUN_SHELL="docker run -d -p ${PORT_VAL} -v /docker/logs:/logs --name $CONTAINER_NAME $IMAGE_NAME"echo $RUN_SHELL$RUN_SHELLfidone}#此sh用于在构建前删除已经存在的镜像function clearImage(){#遍历文件夹获取jarfor file in `find  * -name "${1}"`do#Jenkins中编译好的jar名称     jar_name=${file##*/}    catName="${jar_name%-*.*.*-SNAPSHOT.jar}"#截取jar包的名称作为镜像名    IMAGE_NAME="你的dockerhub账号/$catName"    CONTAINER_NAME="$catName"echo "container name : $CONTAINER_NAME"    cid=$(docker ps -a | grep $CONTAINER_NAME |awk '{print $1}')if [ x"$cid" != x ]thenecho "REMOVE CONTAINER ==> $CONTAINER_NAME"        docker rm -f $cidfi# 删除Docker镜像    tid=$(docker images | grep $IMAGE_NAME |awk '{print $3}')if [ x"$tid" != x ]thenecho "REMOVE IMAGE ==> $IMAGE_NAME"        docker rmi -f $tidfidone}#判断是否使用了参数，如有参数则只启动符合参数条件的jarif [ $# -gt 0 ] ;thenfor regx in $*doecho $regx    clearImage $regx    buildAndRun $regxdoneelse    clearImage "*.jar"    buildAndRun "*.jar"fi
```

这段代码有三处地方要修改下：

  1）  declare -A map=(\["bss"\]="8004:8004" \["server-2"\]="8006:8006" \["sys-service"\]="8206:8206")  ，由于docker启动后端口是随机产生的，如果不指定暴露端口与服务端口绑定会导致服务虽然能注册上注册中心但是其它服务无法访问。

  2）IMAGE\_NAME="你的github账号/$catName"   替换为你的dockerhub账号，这是docker镜像命名的规范要求，即：账号名/镜像名 ，别忘了上面的代码有两处 “IMAGE\_NAME="你的github账号/$catName” 要修改。

  3） dockerfile="./"  ，此变是你当前项目Dockerfile的位置，Dockerfile是构建docker镜像必不可少的文件，如果它在你的项目根目录，那么直接./即可。

**这段代码的大致作用是先判断有没有参数，如果有参数则根据参数的匹配符查找当前目录以及子目录的文件，并截取文件名作为镜像、容器名，然后在启动之前判断是否已经存在同名的容器或镜像，如果存在则删除并重新构建并启动新的容器，如果不传参数则默认该目录及子目录下的所有jar文件，所以如果想只打包某个服务的话，在jenkins执行shell命令那应该这样填写： bash docker-build.sh "system-service-\*.jar" 。** 

## 五、编写Dockerfile文件

在项目根目录添加Dockerfile文件（没有后缀），输入如下代码：

```
FROM java:8-jre-alpine#FROM 使用java环境镜像#设置挂载目录,使用项目名作为日志文件夹，所有项目的日志统一都是spring.log，因此使用文件夹区分#VOLUME /docker/logs/system-serviceVOLUME logs#声明使用maven传过来的变量#ARG JAR_FILE   #现使用sh命令构建已不再需要#将该项目的JAR添加到镜像中#ADD ${JAR_FILE} app.jar  #原本使用maven插件构建并传jar包位置作为变量，现改为写死jar包位置ADD target/system-service-*.jar app.jar#RUN bash -c 'touch app.jar'EXPOSE 8014#jar运行命令ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
```

有三处需要修改：

  1）  FROM java:8-jre-alpine    表示使用该镜像建构容器，可以改为你需要的镜像，这里没有其它需求所以直接使用了jdk8环境镜像。

  2） ADD target/system-service-\*.jar app.jar    改为你要打包的项目的路径，由于我使用的是maven + spring boot，所以打包好后在target文件夹下

  3） EXPOSE 8014    当前容器要暴露出去的端口，修改为当前服务的启动端口

## 六、测试是否成功

    回到jenkins操作页面，点击刚才配置的任务，点击立刻构建，左边的Build History会出现灰点，点击灰点进去看打印的日志，如无异常最后结束时会打印：Finished: SUCCESS   。

### 测试docker

输入docker images 看看是否成功构建了镜像，再输入docker ps，看看是否成功创建了容器，什么？没有？空的？

输入docker ps -a 查看所有容器，如果此时有了表示容器创建成功但启动失败了，检查Dockerfile 或者是不是因为环境问题导致项目启动失败了。

### 推送docker镜像

假设一切ok，此时想要在其它服务器上也部署一套，可以先输入docker login ,登陆docker hub，再输入docker push 你的镜像名  

将镜像推送到docker hub上，这样其它服务器就可以通过docker pull 命令直接拿到这个镜像了。

### 查看服务日志

构建docker镜像代码中其中一段：RUN\_SHELL="docker run -d -p ${PORT\_VAL} -v /docker/logs:/logs --name $CONTAINER\_NAME $IMAGE\_NAME"     -v /docker/logs:/logs  是表示将宿主机的目录 映射到该镜像的logs目录，众所周知spring boot日志默认目录是logs，所以我将 /docker/logs目录映射到了该镜像的logs目录，因此只要进入/docker/logs目录即可看到该服务的启动日志了