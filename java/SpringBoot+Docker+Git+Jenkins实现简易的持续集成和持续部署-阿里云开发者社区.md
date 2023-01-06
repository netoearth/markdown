前言

本篇文章引导你使用Jenkins部署\[SpringBoot项目\],同时使用Docker和Git实现简单的持续集成和持续部署。（项目地址：sso-merryyou）

流程图如下：  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker02.png)

push代码到Github触发WebHook。（因网络原因，本篇使用gitee）  
Jenkins从仓库拉去代码  
mavem构建项目  
代码静态分析  
单元测试  
build镜像  
push镜像到镜像仓库（本篇使用的镜像仓库为网易镜像仓库）  
更新服务  
Jenkins安装

下载jenkins

从[https://jenkins.io/download/](https://jenkins.io/download/)下载对应的jenkins  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker07.png)

初始化密码

访问本地：[http://localhost:8080](http://localhost:8080/)输入密码  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker04.png)

选择插件

进入用户自定义插件界面，选择第二个（因为我们本次构建使用的为Pipelines）

勾选与Pipelines相关的插件  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker05.png)

等待插件安装完成  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker06.png)

配置用户名和密码  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker08.png)

全局配置

系统管理-》全局工具配置 配置Git,JDK和Maven  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker09.png)

安全配置

系统管理-》全局安全配置

勾选Allow anonymous read access  
取消防止跨站点请求伪造  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker10.png)

新建任务

新建任务-》流水线  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker11.png)

构建脚本

勾选触发远程构建 (WebHooks触发地址)，填写简单的Pipeline script  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker12.png)

```
#!groovy
pipeline{
    agent any

    stages {

        stage('test'){
            steps {
                echo "hello world"
            
            }
        }            
    }
}
```

测试脚本

立即构建  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker13.png)

控制台输出  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker14.png)

gitee集成WebHooks

添加SSH公匙  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker15.png)

配置WebHooks  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker16.png)

> 使用natapp实现内网穿透

![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker17.png)

修改脚本

修改Pipeline script  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker18.png)

```
#!groovy
pipeline{
    agent any
    //定义仓库地址
    environment {
        REPOSITORY="https://gitee.com/merryyou/sso-merryyou.git"
    }

    stages {

        stage('获取代码'){
            steps {
                echo "start fetch code from git:${REPOSITORY}"
                //清空当前目录
                deleteDir()
                //拉去代码    
                git "${REPOSITORY}"
            }
        }

        stage('代码静态检查'){
            steps {
                //伪代码检查
                echo "start code check"
            }
        }        

        stage('编译+单元测试'){
            steps {
                echo "start compile"
                //切换目录
                dir('sso-client1') {
                    //重新打包
                    bat 'mvn -Dmaven.test.skip=true -U clean install'
                }
            }
        }

        stage('构建镜像'){
            steps {
                echo "start build image"
                dir('sso-client1') {
                    //build镜像
                    bat 'docker build -t hub.c.163.com/longfeizheng/sso-client1:1.0 .'
                    //登录163云仓库
                    bat 'docker login -u longfei_zheng@163.com -p password hub.c.163.com'
                    //推送镜像到163仓库
                    bat 'docker push hub.c.163.com/longfeizheng/sso-client1:1.0'
                }
            }
        }

        stage('启动服务'){
            steps {
                echo "start sso-merryyou"
                //重启服务
                bat 'docker-compose up -d --build'
            }
        }                

    }
}
```

Pipeline的几个基本概念：

Stage: 阶段，一个Pipeline可以划分为若干个Stage，每个Stage代表一组操作。注意，Stage是一个逻辑分组的概念，可以跨多个Node。

Node: 节点，一个Node就是一个Jenkins节点，或者是Master，或者是Agent，是执行Step的具体运行期环境。

Step: 步骤，Step是最基本的操作单元，小到创建一个目录，大到构建一个Docker镜像，由各类Jenkins Plugin提供。

更多Pipeline语法参考：pipeline 语法详解

测试

docker-compose up -d 启动服务  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker19.png)

访问[http://sso-taobao:8083/client1](http://sso-taobao:8083/client1)登录  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker20.png)

修改内容效果如下：  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker03.gif)

更多效果图  
![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker21.png)

![](https://raw.githubusercontent.com/longfeizheng/longfeizheng.github.io/master/images/docker/docker22.png)

代码下载

github:[https://github.com/longfeizheng/sso-merryyou](https://github.com/longfeizheng/sso-merryyou)  
gitee:[https://gitee.com/merryyou/sso-merryyou](https://gitee.com/merryyou/sso-merryyou)  
文章来源：[https://my.oschina.net/merryyou/blog/1799317](https://my.oschina.net/merryyou/blog/1799317)