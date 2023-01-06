![](https://img-blog.csdnimg.cn/img_convert/a2b7cce76a0fc21ad691f3e495a56cf3.png)

本文章实现最简单全面的 Jenkins + Docker + Spring Boot 一键自动部署项目。步骤齐全，少走坑路。

-   环境：CentOS7 + Git (Gitee)
    
-   实现步骤：在 Docker 安装 Jenkins，配置 Jenkins 基本信息，利用 Dockerfile 和 Shell 脚本实现项目自动拉取打包并运行。
    

**一、安装docker**

安装社区版本 Docker CE

### **1\. 确保 yum 包更新到最新**

```
yum update
```

### **2\. 卸载旧版本（如果安装过旧版本的话）**

```
yum remove docker  docker-common docker-selinux docker-engine
```

### **3\. 安装需要的软件包**

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### **4\. 设置 yum 源**

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### **5\. 安装 Docker**

```
yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
```

**6\. 启动并设置开机启动**

```
systemctl start docker
```

### **7\. 验证安装是否成功**

```
docker version
```

## **二、安装 Jenkins**

Jenkins 中文官网：https://www.jenkins.io/zh/

### **1\. 安装 Jenkins**

Docker 安装一切都是那么简单。注意检查 8080 是否已经占用，如果占用请修改端口。

```
docker run --name jenkins -u root --rm -d -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

如果没改端口号的话，安装完成后访问地址：

```
http://{部署Jenkins所在服务IP}:8080
```

此处会有几分钟的等待时间。

### **2\. 初始化 Jenkins**

详情见官网教程：https://www.jenkins.io/zh/doc

**2.1 解锁 Jenkins**

```
# 进入Jenkins容器
```

**2.2 安装插件**

选择第一项：**安装推荐的插件**。

![](https://img-blog.csdnimg.cn/1c00444717b14b1cb9b6f0a5da8088db.png)

![](https://img-blog.csdnimg.cn/b6d1713716d44b958c2b79a89167a969.png) 

**2.3 创建管理员用户**

此账户信息一定要记住哦。

## **三、系统配置**

### **1\. 安装需要插件**

进入【首页】–【系统管理】–【插件管理】–【可选插件】。搜索以下需要安装的插件，点击安装即可。

![](https://img-blog.csdnimg.cn/img_convert/a5d22c10f6fc8772e9ff0f1f572d13c8.png)

-   安装 Maven Integration
    
-   安装 Publish Over SSH（如果不需要远程推送，不用安装）
    
-   如果使用 Gitee 码云，安装插件Gitee（自带 Git 不用单独安装）
    

### **2\. 配置 Maven**

进入【首页】–【系统管理】–【全局配置】，拉到页面最下方 maven–maven 安装。

![](https://img-blog.csdnimg.cn/11d76c8086734018aa323d69207c9c59.png) 

## **四、创建任务**

### **1\. 新建任务**

点击【新建任务】，输入任务名称，点击构建一个自由风格的软件项目。

![](https://img-blog.csdnimg.cn/c62bfb3c17c84351849c0a920800d490.png)

### **2\. 源码管理**

点击【源码管理】–【Git】，输入仓库地址，添加凭证，选择好凭证即可。

![](https://img-blog.csdnimg.cn/img_convert/20e03d337e062256096371fadc011e1b.png)

![](https://img-blog.csdnimg.cn/img_convert/28dafc95c83109508435a278113d72e4.png)

### **3\. 构建触发器**

点击【构建触发器】–【构建】–【增加构建步骤】–【调用顶层 Maven 目标】–【「填写配置」】–【保存】。

![](https://img-blog.csdnimg.cn/91a8001b263f4aa9bb6dbaf6bf0c46ee.png)

此处命令只是 install，看是否能生成 jar 包。

```
clean install -Dmaven.test.skip=true
```

### **4\. 保存**

点击【保存】按钮即可。

## **五、测试**

该功能测试是否能正常打包。

### **1\. 构建**

点击构建按钮。

![](https://img-blog.csdnimg.cn/4d1e9efd5e3241d99d2704a131f182ca.png)

### **2\. 查看日志**

点击正在构建的任务，或者点击任务名称进入详情页面，查看控制台输出。看是否能成功打成 jar 包。

该处日志第一次可能下载依赖 jar 包失败，再次点击构建即可成功。

![](https://img-blog.csdnimg.cn/33ed84d1c15b41ac88d1a10de1b8fd22.png)

 ![](https://img-blog.csdnimg.cn/d053498c57de460bb5c7904c07094e91.png)

**3\. 配置JDK版本**

**4\. 查看项目位置**

```
cd /var/jenkins_home/workspace
```

## **六、运行项目**

因为项目和 Jenkins 在同一台服务器，所以我们用 Shell 脚本运行项目，原理既是通过 Dockerfile 打包镜像，然后 docker 运行即可。

### **1\. Dockerfile**

在 Spring Boot 项目根目录新建一个名为 Dockerfile 的文件，注意没有后缀名。

其内容如下：（大致就是使用 JDK 8，把 jar 包添加到 docker 然后运行 prd 配置文件。详细可以查看其他教程）

```
FROM jdk:8
```

### **2\. 修改 Jenkins 任务配置**

![](https://img-blog.csdnimg.cn/img_convert/8b3d909cace0411b79d07d371ecea935.png)

 配置如下：

![](https://img-blog.csdnimg.cn/img_convert/e71e4635dce965e9cab1b7538ebfb04c.png)

"-t" 指定新镜像名，"." 表示 Dockfile 在当前路径。

```
cd /var/jenkins_home/workspace/zx-order-api
```

备注：  
1\. 上图用了 docker logs -f 是为了方便看日志，真实环境不要用，因为会一直等待日志，构建任务会失败；  
2\. 加 "|| true" 是如果命令执行失败也会继续实行，为了防止第一次没有该镜像报错；

### 3\. 保存：点击保存即可；

### 4\. 构建：查看 Jenkins 控制台输出，输出如下，证明成功；

![](https://img-blog.csdnimg.cn/img_convert/d59e854701067b9d139cb25496b9f0cf.png)

### 5\. 验证

```
docker ps # 查看是否有自己的容器
```