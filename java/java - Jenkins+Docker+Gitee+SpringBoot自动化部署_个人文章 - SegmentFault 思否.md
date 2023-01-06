当我们使用传统的开发方式开发后台系统时，每写完一个功能点就需要重新运行一下项目，然后进行测试，如果是项目比较小还可以，但是如果项目比较大的话，由于涉及的人员比较多，这种开发方式就比较麻烦。基于此，我们就需要使用Jenkins配合Gitee搭建一个自动化部署平台，并将代码托管到服务器上，这样减轻了本地的电脑压力，也解放了部署的流程。

## 1， 搭建Jenkins平台

首先，我们需要搭建一下Jenkins自动化构建平台。首先，我们需要安装Docker，然后在Docker中安装Jenkins，安装的命令如下：

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker

sudo systemctl enable docker
```

通过以上指令即可成功安装Docker并启动，接下来我们通过Docker运行Jenkins。

```
docker run \
  -d \
  --rm \
  -u root \
  -p 8080:8080 \
  -v /home/jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /opt/develop_resource/apache-maven-3.6.3:/usr/local/maven \
  -v "$HOME":/home \
  jenkinsci/blueocean
```

执行上述指令Docker会自动拉取Jenkins的镜像并启动，因为我们要部署的是SpringBoot，所以需要准备JDK和Maven环境，不过该Jenkins镜像自带了JDK环境，只需准备一下Maven即可，首先，下载Maven压缩包，命令如下：

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

然后，使用命令解压文件。

```
tar -zxvf apache-maven-3.6.3-bin.tar.gz
```

解压之后千万要注意Maven所在的目录，比如：

```
/opt/develop_resource/apache-maven-3.6.3
```

将它挂载到容器的目录里，`-v /opt/develop_resource/apache-maven-3.6.3:/usr/local/maven`这一条指令中的Maven目录千万记得换成自己的。现在，我们可以运行刚才的指令启动Jenkins了，通过`docker ps`指令可以查看容器是否启动。

```
[root@10 /]
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS         PORTS                                                  NAMES
dfa1b8b2c7a3   jenkinsci/blueocean   "/sbin/tini -- /usr/…"   15 seconds ago   Up 9 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 50000/tcp   condescending_meitner
```

接下来，我们使用服务器的ip加端口8080的方式访问Jenkins。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473619 "在这里插入图片描述")  
管理员密码可以在Jenkins的启动日志中查看，使用docker logs dfa1b8b2c7a3查看日志：  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473620 "在这里插入图片描述")  
密码就是红框中的字符串，注意红框下的一段提示：

```
This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```

意思是你可以在/var/jenkins\_home/secrets/initialAdminPassword这个文件中查看到管理员密码，不过这是Jenkins容器内的目录，我们在启动Jenkins的就挂载了Jenkins的关键目录/var/jenkins\_home，宿主机目录为/home/jenkins-data，所以可以使用如下指令查看管理员密码。

```
cat /home/jenkins-data/secrets/initialAdminPassword
```

得到密码后输入到Jenkins页面解锁Jenkins，然后点击安装推荐的插件。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473621 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473622 "在这里插入图片描述")  
接下来，直接点击【下一步】。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473623 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473624 "在这里插入图片描述")  
到这一步，Jenkins平台就可以正式使用了。

## 2， Jenkins平台配置

接下来，就是对Jenkins平台的配置，首先配置Maven。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473625 "在这里插入图片描述")  
按步骤点击，即可进入系统配置。首先，在全局属性中进行配置。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473627 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473628 "在这里插入图片描述")  
还记得我们在运行Jenkins容器时挂载的Maven目录吗？挂载到Jenkins容器中的目录就是/usr/local/maven，如果实在搞不懂的你就保持和我的配置一样即可。

以同样的方式，再新增一个配置：  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473629 "在这里插入图片描述")  
PATH+EXTRA的作用是让原来PATH变量中的环境不丢失，最后点击保存。Maven配置完成后，需要配置Gitee。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473630 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473631 "在这里插入图片描述")  
点击可选插件，选中Gitee，然后点击Install without restart进行安装，等待安装完成即可，Gitee的相关配置我们放到后面讲。

## 3，创建SpringBoot应用

首先，我们创建一个简单的SpringBoot应用进行测试，控制器代码如下。

```
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

然后在配置文件application.yml中添加：

```
server:
  port: 8000
```

并在main下新建docker文件夹，在docker文件夹下新建Dockefile文件，内容如下。

```
FROM java:8


MAINTAINER wwj


VOLUME /tmp


ADD /target/demo-0.0.1-SNAPSHOT.jar springboot.jar


RUN bash -c 'touch /springboot.jar'


EXPOSE 8000


ENTRYPOINT ["java", "-jar", "springboot.jar"]
```

需要注意的是ADD指令的编写，当SpringBoot应用打包完成后，其jar包会被放在target目录下。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473632 "在这里插入图片描述")  
所以需要指定该文件的位置，使用ADD指令将其放入待构建的容器中，接着在Gitee中新建一个仓库，并将代码推送到仓库中。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473633 "在这里插入图片描述")  
仓库名随便你叫什么，然后将刚才的应用推送上去即可。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473634 "在这里插入图片描述")

## 4，Gitee配置

推送完成后，回到Jenkins管理界面，我们来完成Gitee的配置，打开系统配置。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473635 "在这里插入图片描述")  
找到Gitee配置，填入对应的信息：  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473636 "在这里插入图片描述")  
点击添加按钮添加一个Jenkins凭证。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473637 "在这里插入图片描述")  
选择Gitee API 令牌：  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473638 "在这里插入图片描述")  
私人令牌的获取地址为：[https://gitee.com/profile/personal\_access\_tokens](https://link.segmentfault.com/?enc=yZIwy9lTD7WIzeMC9IoBMg%3D%3D.2UxTdiwjpV%2Bdi%2B6zrBvXikLv8dwEAeolDkILIGzUkwj5%2FA2N9D1D0MVwbnaxdlAy3cEEhILEOLF6vEK%2FUOuJ6w%3D%3D)。

![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473639 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473640 "在这里插入图片描述")

![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473641 "在这里插入图片描述")

## 5， 新建自动化部署任务

配置完成，接下来新建一个任务，点击新建Item。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473642 "在这里插入图片描述")  
随便输入一个任务名称，并选择【Freestyle project】。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473643 "在这里插入图片描述")  
在源码管理处勾选Git，并填入项目地址，然后在构建触发器位置勾选触发打包的时机。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473644 "在这里插入图片描述")  
在构建触发器最底部位置点击生成Gitee WebHook密码。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473645 "在这里插入图片描述")

然后打开Gitee项目中的WebHooks，添加webHook。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473646 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473647 "在这里插入图片描述")

此处的URL需要填入一个公网IP，所以如果你是用的虚拟机，就需要用内网穿透工具映射一下。

![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473648 "在这里插入图片描述")  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473649 "在这里插入图片描述")  
至于URL应该填什么，我们需要修改一下。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473650 "在这里插入图片描述")  
填写完成后点击添加，Gitee便会发送一个Post请求到Jenkins，如果请求结果如下所示，则配置成功了。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473651 "在这里插入图片描述")  
重新回到Jenkins管理界面，继续勾选构建触发器下的轮询SCM，输入轮询频率。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473652 "在这里插入图片描述")  
最后在构建位置下增加构建步骤，选择执行shell。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473653 "在这里插入图片描述")  
shell脚本代码如下。

```
#!/bin/bash -il
docker rm -f app_docker
sleep 1
docker rmi -f app_docker:1.0
sleep 1
mvn clean install -Dmaven.test.skip=true
sleep 1
docker build -t app_docker:1.0 -f ./src/main/docker/Dockerfile .
sleep 1
docker run -d -p 8000:8000 --name app_docker app_docker:1.0
```

该脚本表示删除正在运行的app\_docker容器，并删除app\_docker:1.0镜像，然后使用mvn命令打包从Gitee拉取来的项目代码，接着使用项目中的Dockerfile文件构建出一个镜像，名称为app\_docker:1.0，最后运行该镜像。

## 6，打包测试

最后点击保存，部署任务就创建完成了，我们来测试一下有没有问题。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473654 "在这里插入图片描述")  
点击立即构建，Jenkins会立马进行一次构建，查看控制台输出。  
![在这里插入图片描述](https://segmentfault.com/img/remote/1460000041473655 "在这里插入图片描述")  
最后，我们打开默认的地址即可。