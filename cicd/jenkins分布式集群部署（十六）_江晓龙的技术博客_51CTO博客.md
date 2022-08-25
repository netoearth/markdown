©著作权归作者所有：来自51CTO博客作者江晓龙的技术博客的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[jenkins分布式集群部署（十六）](https://blog.51cto.com/jiangxl/4637735)  
[https://blog.51cto.com/jiangxl/4637735](https://blog.51cto.com/jiangxl/4637735)

jenkins分布式部署

## 1.jenkins分布式概念

jenkins分布式就是有多个slave节点，当需要构建的项目非常多时，slave会承担master的工作量，在slave在上创建项目。

slave的环境要和master一致，master上安装了什么软件在slave上要准备相同的，并且路径最好保持一致，与master的区别在于不用安装jenkins

![jenkins分布式集群部署（十六）_linux](https://s2.51cto.com/images/blog/202111/18144759_6195f71f0cf3872343.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

## 2.jenkins分布式部署

### 2.1.增加slave节点

jenkins分布式其实就是在页面上点击manage nodes and clouds新增一个节点即可

![jenkins分布式集群部署（十六）_配置信息_02](https://s2.51cto.com/images/blog/202111/18144759_6195f71f4202f70798.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

点击新增节点—输入节点名称即可

![jenkins分布式集群部署（十六）_java_03](https://s2.51cto.com/images/blog/202111/18144759_6195f71f845b830396.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.2.填写节点配置信息

填写节点名称：jenkins-node1-192.168.81.230

执行器数量：5（执行器数量就是执行任务的并发数）

远程工作目录：/home/jenkins（任意目录即可，不存在就会创建）

标签：node1（一定要写清楚）

用法：use this node as mush as possible

启动方式：lauch agents via shh

主机写上slave节点的ip地址，然后添加一个凭据，类型就写password即可，填写上用户名密码即可

![jenkins分布式集群部署（十六）_linux_04](https://s2.51cto.com/images/blog/202111/18144759_6195f71fc132e81837.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

**配置信息截图**

![jenkins分布式集群部署（十六）_maven_05](https://s2.51cto.com/images/blog/202111/18144759_6195f71fef80f84392.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.3.配置节点属性添加软件路径

```
此步骤配置主要是为了master与slave上的环境保持一致在slave1上安装依赖软件[root@jenkins-node1 ~]# yum -y install git java maven curl-devel expat-devel gettext-devel openssl-devel zlib-devel[root@jenkins-node1 ~]# scp root@192.168.81.220:/usr/local/sonar-scanner-4.0.0.1744-linux/ /usr/local/查看git安装位置[root@jenkins-node1 ~]# rpm -ql git | less /usr/libexec/git-core/git查看jdk、maven路径[root@jenkins-node1 ~]# mvn --versionApache Maven 3.0.5 (Red Hat 3.0.5-17)   Maven home: /usr/share/maven    #maven路径Java version: 1.8.0_262, vendor: Oracle CorporationJava home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre     #jdk路径Default locale: zh_CN, platform encoding: UTF-8OS name: "linux", version: "3.10.0-957.el7.x86_64", arch: "amd64", family: "unix"将脚本目录拷贝至slave1[root@jenkins-node1 ~]# scp -r root@192.168.81.220:/script /1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.
```

将路径信息填写到下图中

![jenkins分布式集群部署（十六）_maven_06](https://s2.51cto.com/images/blog/202111/18144800_6195f72031c0035848.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

点击重启代理

![jenkins分布式集群部署（十六）_linux_07](https://s2.51cto.com/images/blog/202111/18144800_6195f7205816e15126.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

配置成功

![jenkins分布式集群部署（十六）_java_08](https://s2.51cto.com/images/blog/202111/18144800_6195f720a1b4986899.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

**总结**

如果遇到git拉取代码失败，如下图，请将git工具变量删除

![jenkins分布式集群部署（十六）_配置信息_09](https://s2.51cto.com/images/blog/202111/18144800_6195f720d9a2744135.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

解决方法：将git从下面移除

![jenkins分布式集群部署（十六）_maven_10](https://s2.51cto.com/images/blog/202111/18144801_6195f7211faa963604.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

## 3.多运行几个项目看下是否在slave1上有执行

```
配置slave1连接gitlab、web集群、jenkins[root@jenkins-node1 ~]# ssh-keygen [root@jenkins-node1 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.81.210[root@jenkins-node1 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.81.220[root@jenkins-node1 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.81.240[root@jenkins-node1 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.81.250配置hosts[root@jenkins-node1 ~]# vim /etc/hosts192.168.81.210  gitlab.jiangxl.com1.2.3.4.5.6.7.8.9.10.
```

**配置与gitlab进行连接**

```
[root@jenkins-node1 ~]# cat .ssh/id_rsa.pub ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDqngv/x8mgqLC7/p2Zekzpue3VzxIcQc8iEWAumrF+r1qaCOihdBRrT6njWzLZJXx1ngAaCjtcJd59tUjFzU+VrPNqKGO4tHfFXrRWEJ8NonHIc0Yj8yq0rbVcab5urujjIZEOsjLOkPNtdOfuwCJhh3BiCMnF+0HWSpN4PyH9ALFRzzGDgMREHGwKG8pwu/F0ccTk/IGBUnGgWCt0I/8ah93kRxNxsUy+P9CQRkP6P2gNWsUo9vqhN0HnejGiBVcSEZdsSkFDwo3rZcRbsO1xNy0zpvdbfhSQ/C9AlEG099Qgp0LOl3RWWklqAqBfzYEl15D3ycevl5QJ0z7xITyT root@jenkins-node11.2.
```

**在gitlab上添加私钥**

![jenkins分布式集群部署（十六）_linux_11](https://s2.51cto.com/images/blog/202111/18144801_6195f7213fc1f62631.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

运行多个项目查看分布式结果，查看是master和slave都会执行任务

![jenkins分布式集群部署（十六）_git_12](https://s2.51cto.com/images/blog/202111/18144801_6195f7217f15a44107.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

slave1上会把从gitlab拉取的代码保存到/home/jenkins/workspace中

这个/home/jenkins是我们配置的工作目录

![jenkins分布式集群部署（十六）_配置信息_13](https://s2.51cto.com/images/blog/202111/18144801_6195f721d365646062.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

## 4.配置pipeline项目指定在slvae1上运行

### 4.1.编写pipeline脚本

```
pipeline{    agent { label 'node1' }         //指定在node1上执行    parameters {        string(name: 'git_version', defaultValue: 'v1.0', description: '输入tag版本')    }    stages {    //一个大的任务合集  (部署代码)        stage('获取代码') {            steps {                checkout([$class: 'GitSCM', branches: [[name: '${git_version}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '9847113e-e313-4dc6-9302-dc4dec14804b', url: 'git@gitlab.jiangxl.com:root/monitor.git']]])            }        }        stage('质量扫描'){            steps {                withSonarQubeEnv('sonarqube') {                    sh '/usr/local/sonar-scanner-4.0.0.1744-linux/bin/sonar-scanner -Dsonar.projectKey=${JOB_NAME} -Dsonar.sources=.'                }            }        }        stage('编译代码'){            steps {                echo "build code ok"                //sh 'mvn package'            }        }        stage('部署代码'){            steps {                sh 'sh -x /script/monitor_deploy_tag.sh'            }        }    }}1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.
```

### 4.2.将代码集成到项目中

```
pipeline{    agent { label 'node1' }         //指定在node1上执行    parameters {        string(name: 'git_version', defaultValue: 'v1.0', description: '输入tag版本')    }    stages {    //一个大的任务合集  (部署代码)        stage('获取代码') {            steps {                checkout([$class: 'GitSCM', branches: [[name: '${git_version}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '9847113e-e313-4dc6-9302-dc4dec14804b', url: 'git@gitlab.jiangxl.com:root/monitor.git']]])            }        }        stage('质量扫描'){            steps {                withSonarQubeEnv('sonarqube') {                    sh '/usr/local/sonar-scanner-4.0.0.1744-linux/bin/sonar-scanner -Dsonar.projectKey=${JOB_NAME} -Dsonar.sources=.'                }            }        }        stage('编译代码'){            steps {                echo "build code ok"                //sh 'mvn package'            }        }        stage('部署代码'){            steps {                sh 'sh -x /script/monitor_deploy_tag.sh'            }        }    }}1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.
```

![jenkins分布式集群部署（十六）_linux_14](https://s2.51cto.com/images/blog/202111/18144802_6195f7222dbd283484.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 4.3.构建项目

会看到在slave1上在执行

![jenkins分布式集群部署（十六）_linux_15](https://s2.51cto.com/images/blog/202111/18144802_6195f7224762079358.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

-   **赞**
-   **收藏**
-   **评论**
-   **举报**

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持JPEG/PNG/JPG，图片不超过1.9M

已经收到您得举报信息，我们会尽快审核

## 相关文章