©著作权归作者所有：来自51CTO博客作者码农小宋的原创作品，请联系作者获取转载授权，否则将追究法律责任

本文章实现最简单全面的`Jenkins+docker+springboot` 一键自动部署项目，步骤齐全，少走坑路。

> 环境：centos7+git(gitee)

简述实现步骤：在docker安装jenkins，配置jenkins基本信息，利用Dockerfile和shell脚本实现项目自动拉取打包并运行。

## 一、安装docker

docker安装社区版本CE

1.确保 yum 包更新到最新。

2.卸载旧版本(如果安装过旧版本的话)

```
yum remove docker  docker-common docker-selinux docker-engine1.
```

3.安装需要的软件包

```
yum install -y yum-utils device-mapper-persistent-data lvm21.
```

4.设置yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo1.
```

5.安装docker

```
yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0yum install <自己的版本>  # 例如：sudo yum install docker-ce-17.12.0.ce1.2.
```

6.启动和开机启动

```
systemctl start dockersystemctl enable docker1.2.
```

7.验证安装是否成功

## 二、安装Jenkins

Jenkins中文官网：

>  [https://www.jenkins.io/zh/](https://www.jenkins.io/zh/)

#### 1.安装Jenkins

docker 安装一切都是那么简单，注意检查8080是否已经占用！如果占用修改端口

```
docker run --name jenkins -u root --rm -d -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean1.
```

如果没改端口号的话

安装完成后访问地址-> `http://{部署Jenkins所在服务IP}:8080`

此处会有几分钟的等待时间。

#### 2.初始化Jenkins

##### 2.1 解锁Jenkins

-   进入Jenkins容器：`docker exec -it {Jenkins容器名} bash`
-   例如`docker exec -it jenkins bash`
-   查看密码：`cat /var/lib/jenkins/secrets/initialAdminPassword`
-   复制密码到输入框里面

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_git](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12047bc98692.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

##### 2.2 安装插件

选择第一个：安装推荐的插件

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_02](https://s2.51cto.com/images/blog/202208/14203530_62f8ec1262ede75369.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

##### 2.3 创建管理员用户

此账户一定要记住哦

## 三、系统配置

#### 1\. 安装需要插件

进入【首页】–【系统管理】–【插件管理】–【可选插件】

搜索以下需要安装的插件，点击安装即可。

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_03](https://s2.51cto.com/images/blog/202208/14203530_62f8ec127f52094071.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

-   安装`Maven Integration`
-   安装`Publish Over SSH`(如果不需要远程推送，不用安装)
-   如果使用Gitee 码云，安装插件Gitee（Git自带不用安装）

#### 2\. 配置Maven

进入【首页】–【系统管理】–【全局配置】，拉到最下面maven–maven安装

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_docker_04](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12e153e6804.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

## 四、创建任务

#### 1\. 新建任务

点击【新建任务】，输入任务名称，点击构建一个自由风格的软件项目

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_docker_05](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12bce4417582.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 2\. 源码管理

点击【源码管理】–【Git】，输入仓库地址，添加凭证，选择好凭证即可。

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_06](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12c717020200.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")![Jenkins+Docker 一键自动化部署 SpringBoot 项目_docker_07](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12cbf6a90489.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 3.构建触发器

点击【构建触发器】–【构建】–【增加构建步骤】–【调用顶层Maven目标】–【填写配置】–【保存】

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_docker_08](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12d098043502.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")此处命令只是install，看是否能生成jar包

```
clean install -Dmaven.test.skip=true1.
```

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_git_09](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12d6a5f37280.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 4\. 保存

点击【保存】按钮即可

## 五、测试

该功能测试是否能正常打包

#### 1\. 构建

点击构建按钮

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_10](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12f2b6130667.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 2.查看日志

点击正在构建的任务，或者点击任务名称，进入详情页面，查看控制台输出，看是否能成功打成jar包。

该处日志第一次可能下载依赖jar包失败，再次点击构建即可成功。

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_11](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12e47b74146.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")![Jenkins+Docker 一键自动化部署 SpringBoot 项目_git_12](https://s2.51cto.com/images/blog/202208/14203530_62f8ec12f125791279.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 3\. 查看项目位置

1.  `cd /var/jenkins_home/workspace`
2.  `ll` 即可查看是否存在

## 六、运行项目

因为我们项目和jenkins在同一台服务器，所以我们用shell脚本运行项目，原理既是通过dockerfile 打包镜像，然后docker运行即可。

#### 1\. Dockerfile

在springboot项目根目录新建一个名为Dockerfile的文件，注意没有后缀名，其内容如下:（大致就是使用jdk8，把jar包添加到docker然后运行prd配置文件。详细可以查看其他教程）

```
FROM jdk:8VOLUME /tmpADD target/zx-order-0.0.1-SNAPSHOT.jar app.jarEXPOSE 8888ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar","--spring.profiles.active=prd"]1.2.3.4.5.
```

#### 2\. 修改jenkins任务配置

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_13](https://s2.51cto.com/images/blog/202208/14203531_62f8ec1306f4252645.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")配置如下：

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_14](https://s2.51cto.com/images/blog/202208/14203531_62f8ec130a74828987.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

-   `-t`：指定新镜像名
-   `.`：表示Dockfile在当前路径

```
cd /var/jenkins_home/workspace/zx-order-apidocker stop zx-order || truedocker rm zx-order || truedocker rmi zx-order || truedocker build -t zx-order .docker run -d -p 8888:8888 --name zx-order zx-order:latest1.2.3.4.5.6.
```

备注：

1.  我上图用了`docker logs -f` 是为了方便看日志，真实不要用，因为会一直等待日志，构建任务会失败
2.  加`|| true` 是如果命令执行失败也会继续实行，为了防止第一次没有该镜像报错

#### 3\. 保存

点击保存即可

#### 4\. 构建

查看jenkins控制台输出，输出如下，证明成功！

![Jenkins+Docker 一键自动化部署 SpringBoot 项目_maven_15](https://s2.51cto.com/images/blog/202208/14203531_62f8ec130d98376523.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp "图片")

#### 5\. 验证

1.  `docker ps` 查看是否有自己的容器
2.  `docker logs` 自己的容器名 查看日志是否正确
3.  浏览器访问项目试一试

-   **赞**
-   **收藏**
-   **评论**
-   **举报**

**相关文章**