来自：[芋道源码](https://developer.aliyun.com/group/yudaoyuanma) 2022-09-08 112

**简介：** Jenkins+Docker 一键自动化部署 SpringBoot 项目

本文章实现最简单全面的`Jenkins+docker+springboot` 一键自动部署项目，步骤齐全，少走坑路。

> 环境：centos7+git(gitee)

简述实现步骤：在docker安装jenkins，配置jenkins基本信息，利用Dockerfile和shell脚本实现项目自动拉取打包并运行。

## 一、安装docker

docker安装社区版本CE

1.确保 yum 包更新到最新。

2.卸载旧版本(如果安装过旧版本的话)

```
yum remove docker  docker-common docker-selinux docker-engine
```

3.安装需要的软件包

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

4.设置yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

5.安装docker

```
yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
yum install <自己的版本>  # 例如：sudo yum install docker-ce-17.12.0.ce
```

6.启动和开机启动

```
systemctl start docker
systemctl enable docker
```

7.验证安装是否成功

> 基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
> 
> -   项目地址：[https://gitee.com/zhijiantianya/ruoyi-vue-pro](https://gitee.com/zhijiantianya/ruoyi-vue-pro)
> -   视频教程：[https://doc.iocoder.cn/video/](https://doc.iocoder.cn/video/)

## 二、安装Jenkins

Jenkins中文官网：

> [https://www.jenkins.io/zh/](https://www.jenkins.io/zh/)

#### 1.安装Jenkins

docker 安装一切都是那么简单，注意检查8080是否已经占用！如果占用修改端口

```
docker run --name jenkins -u root --rm -d -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

如果没改端口号的话

安装完成后访问地址-> `http://{部署Jenkins所在服务IP}:8080`

此处会有几分钟的等待时间。

#### 2.初始化Jenkins

##### 2.1 解锁Jenkins

-   进入Jenkins容器：`docker exec -it {Jenkins容器名} bash`
-   例如 `docker exec -it jenkins bash`
-   查看密码：`cat /var/lib/jenkins/secrets/initialAdminPassword`
-   复制密码到输入框里面

![微信图片_20220908132601.png](https://ucc.alicdn.com/pic/developer-ecology/b1d26cc62fdf4bf293fa62786b582716.png "微信图片_20220908132601.png")

##### 2.2 安装插件

选择第一个：安装推荐的插件

![微信图片_20220908132634.png](https://ucc.alicdn.com/pic/developer-ecology/de8fd4051c76476286af2188bd3524b3.png "微信图片_20220908132634.png")

##### 2.3 创建管理员用户

此账户一定要记住哦

> 基于 Spring Cloud Alibaba + Gateway + Nacos + RocketMQ + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
> 
> -   项目地址：[https://gitee.com/zhijiantianya/yudao-cloud](https://gitee.com/zhijiantianya/yudao-cloud)
> -   视频教程：[https://doc.iocoder.cn/video/](https://doc.iocoder.cn/video/)

## 三、系统配置

#### 1\. 安装需要插件

进入【首页】–【系统管理】–【插件管理】–【可选插件】

搜索以下需要安装的插件，点击安装即可。

![微信图片_20220908132652.png](https://ucc.alicdn.com/pic/developer-ecology/7c274f36e519472f86f1b15cb5e8ee83.png "微信图片_20220908132652.png")

-   安装`Maven Integration`
-   安装`Publish Over SSH`(如果不需要远程推送，不用安装)
-   如果使用Gitee 码云，安装插件Gitee（Git自带不用安装）

#### 2\. 配置Maven

进入【首页】–【系统管理】–【全局配置】，拉到最下面maven–maven安装

![微信图片_20220908132715.png](https://ucc.alicdn.com/pic/developer-ecology/8c4f39d69fe04fbebc834d86f0e21139.png "微信图片_20220908132715.png")

## 四、创建任务

#### 1\. 新建任务

点击【新建任务】，输入任务名称，点击构建一个自由风格的软件项目

![微信图片_20220908132734.png](https://ucc.alicdn.com/pic/developer-ecology/5403b8dd15bf400fbf1f6de5f170565c.png "微信图片_20220908132734.png")

#### 2\. 源码管理

点击【源码管理】–【Git】，输入仓库地址，添加凭证，选择好凭证即可。

![微信图片_20220908132758.png](https://ucc.alicdn.com/pic/developer-ecology/ed23854891524d36929d6648e55ac0aa.png "微信图片_20220908132758.png")

![微信图片_20220908132800.png](https://ucc.alicdn.com/pic/developer-ecology/2bfc6e164ae44124a439034a1065e439.png "微信图片_20220908132800.png")

#### 3.构建触发器

点击【构建触发器】–【构建】–【增加构建步骤】–【调用顶层Maven目标】–【填写配置】–【保存】

![微信图片_20220908132825.png](https://ucc.alicdn.com/pic/developer-ecology/7f59cc000fbe4c51853072f4b9ace24a.png "微信图片_20220908132825.png")

此处命令只是install，看是否能生成jar包

```
clean install -Dmaven.test.skip=true
```

![微信图片_20220908132846.png](https://ucc.alicdn.com/pic/developer-ecology/e3f4a97589fa4a13938188b9c4adcb88.png "微信图片_20220908132846.png")

#### 4\. 保存

点击【保存】按钮即可

## 五、测试

该功能测试是否能正常打包

#### 1\. 构建

点击构建按钮

![微信图片_20220908132908.png](https://ucc.alicdn.com/pic/developer-ecology/1ff65ca456c34d4cb07dd61bae0249d6.png "微信图片_20220908132908.png")

#### 2.查看日志

点击正在构建的任务，或者点击任务名称，进入详情页面，查看控制台输出，看是否能成功打成jar包。

该处日志第一次可能下载依赖jar包失败，再次点击构建即可成功。

![微信图片_20220908132932.png](https://ucc.alicdn.com/pic/developer-ecology/886213f112b64017a976588b3087c47b.png "微信图片_20220908132932.png")

#### ![微信图片_20220908132955.png](https://ucc.alicdn.com/pic/developer-ecology/8f9613335da34801aefd2f783e388263.png "微信图片_20220908132955.png")

#### 3\. 查看项目位置

1.  `cd /var/jenkins_home/workspace`
2.  `ll` 即可查看是否存在

## 六、运行项目

因为我们项目和jenkins在同一台服务器，所以我们用shell脚本运行项目，原理既是通过dockerfile 打包镜像，然后docker运行即可。

#### 1\. Dockerfile

在springboot项目根目录新建一个名为Dockerfile的文件，注意没有后缀名，其内容如下:（大致就是使用jdk8，把jar包添加到docker然后运行prd配置文件。详细可以查看其他教程）

```
FROM jdk:8
VOLUME /tmp
ADD target/zx-order-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8888
ENTRYPOINT ["Bash","-DBash.security.egd=file:/dev/./urandom","-jar","/app.jar","--spring.profiles.active=prd"]
```

#### 2\. 修改jenkins任务配置

![微信图片_20220908132955.png](https://ucc.alicdn.com/pic/developer-ecology/d32eb387f35a4fcc881137e896f26cf8.png "微信图片_20220908132955.png")

配置如下：

![微信图片_20220908133042.png](https://ucc.alicdn.com/pic/developer-ecology/dae8eb5ff419443baf9273060994edac.png "微信图片_20220908133042.png")

-   `-t`：指定新镜像名
-   `.`：表示Dockfile在当前路径

```
cd /var/jenkins_home/workspace/zx-order-api
docker stop zx-order || true
docker rm zx-order || true
docker rmi zx-order || true
docker build -t zx-order .
docker run -d -p 8888:8888 --name zx-order zx-order:latest
```

备注：

1.  我上图用了`docker logs -f` 是为了方便看日志，真实不要用，因为会一直等待日志，构建任务会失败
2.  加`|| true` 是如果命令执行失败也会继续实行，为了防止第一次没有该镜像报错

#### 3\. 保存

点击保存即可

#### 4\. 构建

查看jenkins控制台输出，输出如下，证明成功！

![微信图片_20220908133058.png](https://ucc.alicdn.com/pic/developer-ecology/be576e2d6c8a402898af47f108b3688a.png "微信图片_20220908133058.png")

#### 5\. 验证

1.  `docker ps` 查看是否有自己的容器
2.  `docker logs` 自己的容器名 查看日志是否正确
3.  浏览器访问项目试一试

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。

相关文章

[![](https://ucc.alicdn.com/pic/developer-ecology/28751db540a94df29b9baace3742e2bc.png?x-oss-process=image/resize,h_118)](https://developer.aliyun.com/article/924131)

![](https://ucc.alicdn.com/avatar/avatar3.jpg)

[springboot快速入门](https://developer.aliyun.com/article/924131)

正所谓，天下武功，唯快不破，在当今生活节奏越来越快的时代，我们也要讲求效率，也要追求一个快字(不过有些方面还是不能快的，不要当快男哦)。springboot就是能简化配置、敏捷开发的东西。做同一个项目，用spring你可能还在写xml，用springboot的话你可能已经做完在约妹子了！

[![](https://ucc.alicdn.com/pic/developer-ecology/893ed163aac44787ac8414331feb90d6.png?x-oss-process=image/resize,h_118)](https://developer.aliyun.com/article/805777)

![](https://ucc.alicdn.com/avatar/3a40058ae17c4ea29f588ae7f9b39e33.jpg)

[Spring Boot使用Swagger2自动化文档与测试](https://developer.aliyun.com/article/805777)

本文目录 1. 场景 2. Swagger2用了就是爽 3. 具体实现 3.1 导入依赖 3.2 添加Swagger2配置类 3.3 配置静态资源访问 3.4 启动项目并进行查看、测试 3.5 通过注解自动生成API文档 4. 总结

[![](https://ucc.alicdn.com/pic/developer-ecology/8f931f4535954ed59be0aaf741ba58c3.jpeg?x-oss-process=image/resize,h_118)](https://developer.aliyun.com/article/754241)

![](https://ucc.alicdn.com/avatar/avatar3.jpg)

[Docker 部署 Spring Boot](https://developer.aliyun.com/article/754241)

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口，用户可以方便地创建和使用容器，把自己的应用放入容器。本文将为大家介绍如何在 Docker 上部署 Spring Boot 项目。

![](https://ucc.alicdn.com/avatar/img_8f3f791d8c80fb80eb41b06867576cb9.jpg)

[springboot快速入门](https://developer.aliyun.com/article/636561)

前言： 正所谓，天下武功，唯快不破，在当今生活节奏越来越快的时代，我们也要讲求效率，也要追求一个快字(不过有些方面还是不能快的，不要当快男哦)。springboot就是能简化配置、敏捷开发的东西。

![](https://ucc.alicdn.com/avatar/img_88d978702bdc28c1ec2969ca2629ef2e.jpg)

[Spring Boot快速入门](https://developer.aliyun.com/article/417650)

Spring 简介 在您第1次接触和学习Spring框架的时候，是否因为其繁杂的配置而退却了？在你第n次使用Spring框架的时候，是否觉得一堆反复黏贴的配置有一些厌烦？那么您就不妨来试试使用Spring Boot来让你更易上手，更简单快捷地构建Spring应用！ Spring Boot让我们的Spring应用变的更轻量化。