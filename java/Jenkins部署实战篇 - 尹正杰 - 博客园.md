　　　　　　　　　　　　　　　　　　**Jenkins部署实战篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.持续集成理论**

**1>.开发写代码的演变过程**

　　1.1>.一个开发单打独斗，撸代码，开发网站，自由自在；

　　1.2>.多个开发同时开发一个网站，同时改一份代码。但是同时给一个文件会导致冲突；

　　1.3>.分支结构，每天上班第一件事克隆代码，下班前最后一件事合并代码；

　　1.4>.好景不长，开发越来越多，代码文件越来越多。每天下班前合并代码时，发现很多合并失败的文件。为了解决这些合并失败的文件，最后每天加班3小时人工合并代码而且还没有工资；

　　**温馨提示：面临以上的问题的解决方法：将合并代码的周期缩短，以前一天，现在两小时，一小时，半小时......,随时随地将代码合并，这种方法专业术语叫做持续集成。**

**2>.持续集成特点**

　　持续集成的英文名称为：Continuous Integation，简称CI。持续集成指的是，频繁地（一天多次）将代码集成到主干。它的好处主要有以下两个：

　　2.1>.快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易；

　　2.2>.防止分支大幅度偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成；

**3>.运维测试代码的演变**

　　3.1>.初级运维很苦逼，刚开始开发每次合并一次代码，然后运维把代码pull下来测试就可以了；

　　3.2>.但是，后来开发引进了持续集成方法论，开发们都“弹冠相庆”；

　　3.3>.运维同学感觉好苦逼，一天到晚不听的测试代码；

　　3.4>.每天下班之后，运维拖着疲倦的身子回到宿舍，就想，有没有办法自动化？

　　3.5>.初级运维请教了传说中的大神，知道了一些方法；

　　3.6>.借助一个自动化的部署工具，叫做Jenkins；

　　3.7>.开发上传自己的代码到GitLab，GitLab发消息通知Jenkins，随后Jenkins从仓库拉取代码，最后全自动部署到测试服务进行相关测试，并将测试结果通知运维和开发；

　　3.8>.还有偷懒的方法，直接把这个工具交给开发使用，从此就可以高枕无忧了；

　　**温馨提示：这种自动测试的方法叫做持续交付。**

**4>.持续交付特点**

　　持续交付英文名称为：Continuous Delivery，简称CD。我们可以说持续交付是持续集成的下一步，它的好处主要有以下两个：

　　4.1>.持续交付指的是频繁的将软件的新版本，交付给质量团队或者用户，以供审批。如果评审通过，代码就进入生产阶段；

　　4.2>.持续交付可以看作持续集成的下一步。它强调的时，不管怎么更新，软件是随时随地可以交付的；

**5>.运维上线代码的演变**

　　5.1>.代码测试通过了，该到生产环境部署了，运维又该干活了；

　　5.2>.其实这是一锤子买卖了，要么成功，要么失败回滚；

　　5.3>.可以使用自动部署工具，但是很多公司还是相信人工上线；

　　5.4>.但是我们还有偷懒的方法，比如Ansible，Saltstack；

　　**温馨提示：这里也有个方法叫做持续部署。**

 **6>.持续部署特点**

　　持续部署，的英文名称为：Continuous Deployment，简称CD。把代码部署到服务器就是持续部署，它有以下两个特点：

　　6.1>.持续部署是持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境；

　　6.2>.持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段；

　　**温馨提示：你是否经常听一些做持续集成的公司喜欢说的行话：CI/CD ->持续集成/持续交付/持续部署。**

**二.Jenkins介绍**

**1>.Jenkins是一个用Java编写的开源的持续集成工具。在Oracle发生争执后，项目充Hudson项目独立出来；**

**2>.Jenkins提供了软件开发的持续集成服务；**

**3>.Jenkins运行在servlet容器中（例如Apache Tomcat）；**

**4>.Jenkins支持软件配置管理（SCM）工具（包括Accurev SCM，CVS，Subversion，Git，Perforce，Clearcase和RIC）；**

**5>.Jenkins可以执行基于Apache Ant和Apache Maven的项目，以及任意的shell脚本和Windows批处理命令；**

**6>.Jenkins的主要开发者是川口耕介；**

**7>.**Jenkins是MIT许可证下发布的自由软件；****

**三.安装Jenkins环境准备**

**1>.****Jenkins服务器配置**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908224755081-1386107870.png)

**2>.**GitLab服务器配置****

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908224741295-1190961290.png)

**3>.Jenkins的RPM安装包**下载地址****

　　本篇博客采用常规安装方法，即使用RPM包安装

　　Jenkins官网的RPM安装包：http://pkg.jenkins.io/redhat-stable/

　　清华源地址关于Jenkins的安装包：https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/

**4>****.linux中能发邮件的账号**

**四.安装Jenkins**

**1>.安装jdk**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@jenkins yinzhengjie]# yum search jdk
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.shu.edu.cn
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.shu.edu.cn
====================================================== N/S matched: jdk =======================================================
copy-jdk-configs.noarch : JDKs configuration files copier
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.7.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.7.0-openjdk-headless.x86_64 : The OpenJDK runtime environment without audio and video support
java-1.7.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-accessibility.i686 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.i686 : OpenJDK accessibility connector for packages with debug on
java-1.8.0-openjdk-accessibility-debug.x86_64 : OpenJDK accessibility connector for packages with debug on
java-1.8.0-openjdk-debug.i686 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-debug.x86_64 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-demo.i686 : OpenJDK Demos
java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.8.0-openjdk-demo-debug.i686 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-demo-debug.x86_64 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-devel.i686 : OpenJDK Development Environment
java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.8.0-openjdk-devel-debug.i686 : OpenJDK Development Environment with full debug on
java-1.8.0-openjdk-devel-debug.x86_64 : OpenJDK Development Environment with full debug on
java-1.8.0-openjdk-headless.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless-debug.i686 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-headless-debug.x86_64 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-javadoc-debug.noarch : OpenJDK API Documentation for packages with debug on
java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK API Documentation compressed in single archive
java-1.8.0-openjdk-javadoc-zip-debug.noarch : OpenJDK API Documentation compressed in single archive for packages with debug on
java-1.8.0-openjdk-src.i686 : OpenJDK Source Bundle
java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk-src-debug.i686 : OpenJDK Source Bundle for packages with debug on
java-1.8.0-openjdk-src-debug.x86_64 : OpenJDK Source Bundle for packages with debug on
ldapjdk-javadoc.noarch : Javadoc for ldapjdk
icedtea-web.x86_64 : Additional Java components for OpenJDK - Java browser plug-in and Web Start implementation
ldapjdk.noarch : The Mozilla LDAP Java SDK

  Name and summary matches only, use "search all" for everything.
[root@jenkins yinzhengjie]# 
```

查找java的安装包（\[root@jenkins yinzhengjie\]# yum search jdk）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@jenkins yinzhengjie]# yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.shu.edu.cn
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.shu.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package java-1.8.0-openjdk.x86_64 1:1.8.0.181-3.b13.el7_5 will be installed
--> Processing Dependency: java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.181-3.b13.el7_5 for package: 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64
--> Processing Dependency: libjvm.so(SUNWprivate_1.1)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64
--> Processing Dependency: libjava.so(SUNWprivate_1.1)(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64
--> Processing Dependency: libjvm.so()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64
--> Processing Dependency: libjava.so()(64bit) for package: 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64
---> Package java-1.8.0-openjdk-devel.x86_64 1:1.8.0.181-3.b13.el7_5 will be installed
--> Running transaction check
---> Package java-1.8.0-openjdk-headless.x86_64 1:1.8.0.181-3.b13.el7_5 will be installed
--> Processing Dependency: jpackage-utils for package: 1:java-1.8.0-openjdk-headless-1.8.0.181-3.b13.el7_5.x86_64
--> Running transaction check
---> Package javapackages-tools.noarch 0:3.4.1-11.el7 will be installed
--> Processing Dependency: python-javapackages = 3.4.1-11.el7 for package: javapackages-tools-3.4.1-11.el7.noarch
--> Running transaction check
---> Package python-javapackages.noarch 0:3.4.1-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===============================================================================================================================
 Package                                  Arch                Version                               Repository            Size
===============================================================================================================================
Installing:
 java-1.8.0-openjdk                       x86_64              1:1.8.0.181-3.b13.el7_5               updates              250 k
 java-1.8.0-openjdk-devel                 x86_64              1:1.8.0.181-3.b13.el7_5               updates              9.8 M
Installing for dependencies:
 java-1.8.0-openjdk-headless              x86_64              1:1.8.0.181-3.b13.el7_5               updates               32 M
 javapackages-tools                       noarch              3.4.1-11.el7                          base                  73 k
 python-javapackages                      noarch              3.4.1-11.el7                          base                  31 k

Transaction Summary
===============================================================================================================================
Install  2 Packages (+3 Dependent packages)

Total download size: 42 M
Installed size: 144 M
Downloading packages:
(1/5): python-javapackages-3.4.1-11.el7.noarch.rpm                                                      |  31 kB  00:00:00     
(2/5): javapackages-tools-3.4.1-11.el7.noarch.rpm                                                       |  73 kB  00:00:00     
java-1.8.0-openjdk-1.8.0.181-3 FAILED                                                        ] 579 kB/s | 9.8 MB  00:00:56 ETA 
http://centos.ustc.edu.cn/centos/7.5.1804/updates/x86_64/Packages/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64.rpm: [Errno 14] curl#7 - "Failed to connect to 2001:da8:d800:95::110: Network is unreachable"
Trying other mirror.
(3/5): java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64.rpm                                              | 250 kB  00:00:00     
(4/5): java-1.8.0-openjdk-devel-1.8.0.181-3.b13.el7_5.x86_64.rpm                                        | 9.8 MB  00:00:47     
(5/5): java-1.8.0-openjdk-headless-1.8.0.181-3.b13.el7_5.x86_64.rpm                                     |  32 MB  00:01:02     
-------------------------------------------------------------------------------------------------------------------------------
Total                                                                                          687 kB/s |  42 MB  00:01:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python-javapackages-3.4.1-11.el7.noarch                                                                     1/5 
  Installing : javapackages-tools-3.4.1-11.el7.noarch                                                                      2/5 
  Installing : 1:java-1.8.0-openjdk-headless-1.8.0.181-3.b13.el7_5.x86_64                                                  3/5 
  Installing : 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64                                                           4/5 
  Installing : 1:java-1.8.0-openjdk-devel-1.8.0.181-3.b13.el7_5.x86_64                                                     5/5 
  Verifying  : 1:java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64                                                           1/5 
  Verifying  : 1:java-1.8.0-openjdk-headless-1.8.0.181-3.b13.el7_5.x86_64                                                  2/5 
  Verifying  : 1:java-1.8.0-openjdk-devel-1.8.0.181-3.b13.el7_5.x86_64                                                     3/5 
  Verifying  : python-javapackages-3.4.1-11.el7.noarch                                                                     4/5 
  Verifying  : javapackages-tools-3.4.1-11.el7.noarch                                                                      5/5 

Installed:
  java-1.8.0-openjdk.x86_64 1:1.8.0.181-3.b13.el7_5           java-1.8.0-openjdk-devel.x86_64 1:1.8.0.181-3.b13.el7_5          

Dependency Installed:
  java-1.8.0-openjdk-headless.x86_64 1:1.8.0.181-3.b13.el7_5              javapackages-tools.noarch 0:3.4.1-11.el7             
  python-javapackages.noarch 0:3.4.1-11.el7                              

Complete!
[root@jenkins yinzhengjie]#
```

安装java社区版本（即openjdk版本，它就类似linux的CentOS系统），不建议安装oracle的官方版本，因为它收费（\[root@jenkins yinzhengjie\]# yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel）

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins yinzhengjie]# java -version                                #查看java版本
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
[root@jenkins yinzhengjie]# 
[root@jenkins ~]# mkdir /usr/java                                        #切结，如果是源码安装的java的话，我们需要手动创建这个目录
[root@jenkins ~]# 
[root@jenkins ~]# cd /usr/java/
[root@jenkins java]# 
[root@jenkins java]# ln -s /soft/jdk1.8.0_181/ /usr/java/jdk1.8            #这两个链接必须得做，否则如果你rpm方式安装jdk的话，可能会出现Jenkins只启动进程，但是端口始终无法启动的状况
[root@jenkins java]# 
[root@jenkins java]# ln -s /soft/jdk1.8.0_181/ /usr/java/defalut
[root@jenkins java]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.安装Jenkins的rpm安装包**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@jenkins download]# wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.73.1-1.1.noarch.rpm
--2018-09-08 20:13:20--  https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.73.1-1.1.noarch.rpm
Resolving mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)... 101.6.8.193, 2402:f000:1:408:8100::1
Connecting to mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)|101.6.8.193|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 72756863 (69M) [application/x-redhat-package-manager]
Saving to: ‘jenkins-2.73.1-1.1.noarch.rpm’

100%[=====================================================================================>] 72,756,863   496KB/s   in 2m 0s  

2018-09-08 20:15:20 (593 KB/s) - ‘jenkins-2.73.1-1.1.noarch.rpm’ saved [72756863/72756863]

[root@jenkins download]# ll
total 71052
drwxrwxr-x 3 yinzhengjie yinzhengjie      144 Sep  6 13:26 cdh
-rw-r--r-- 1 root        root        72756863 Sep 14  2017 jenkins-2.73.1-1.1.noarch.rpm
[root@jenkins download]# 
```

下载Jenkins安装包（\[root@jenkins download\]# wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.73.1-1.1.noarch.rpm）

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins download]# ll
total 71052
drwxrwxr-x 3 yinzhengjie yinzhengjie      144 Sep  6 13:26 cdh
-rw-r--r-- 1 root        root        72756863 Sep 14  2017 jenkins-2.73.1-1.1.noarch.rpm
[root@jenkins download]# 
[root@jenkins download]# rpm -ivh jenkins-2.73.1-1.1.noarch.rpm 
warning: jenkins-2.73.1-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:jenkins-2.73.1-1.1               ################################# [100%]
[root@jenkins download]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.启动Jenkins服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins download]# /etc/init.d/jenkins start
Starting jenkins (via systemctl):                          [  OK  ]
[root@jenkins download]# 
[root@jenkins download]# 
[root@jenkins download]# 
[root@jenkins download]# 
[root@jenkins download]# ps -ef | grep jenkins
jenkins    5627      1  0 20:18 ?        00:00:00 /etc/alternatives/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
root       5647   5501  0 20:20 pts/0    00:00:00 grep --color=auto jenkins
[root@jenkins download]# 
[root@jenkins ~]# netstat -untalp | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      6106/java           
[root@jenkins ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.访问Jenkins的webUI界面（根据界面的提示信息，去服务器端将密码写进去即可）**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908234448423-1946795782.png)**

**5>.上个步骤执行成功后，会弹出下面的对话框，我们点击关闭即可**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908234650021-1619561553.png)

**6>.开始使用Jenkins**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908234834413-1535515442.png)**

**7>.如果出现下面的界面，那么恭喜你，Jenkins部署成功啦！**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908234919386-2046021956.png)**

 **8>.Jenkins的目录介绍**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins ~]# rpm -ql jenkins
/etc/init.d/jenkins　　　　　　　　　　　　　　　　#启动文件存放位置
/etc/logrotate.d/jenkins　　　　　　　　　　　　　#日志分割配置文件
/etc/sysconfig/jenkins　　　　　　　　　　　　　　#Jenkins主配置文件
/usr/lib/jenkins　　　　　　　　　　　　　　　　　　
/usr/lib/jenkins/jenkins.war　　　　　　　　　　　　#war包存放目录，war包就是把网站站点打个包
/usr/sbin/rcjenkins　　　　　　　　　　　　　　　　#Jenkins的命令
/var/cache/jenkins　　　　　　　　　　　　　　　　#war包解压目录，Jenkins网页代码目录
/var/lib/jenkins　　　　　　　　　　　　　　　　　　#Jenkins的工作目录，Jenkins的配置就在这个目录
/var/log/jenkins　　　　　　　　　　　　　　　　　　#Jenkins的日志存放目录
[root@jenkins ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**五.修改Jenkins密码（我们在安装Jenkins的时候就发现Jenkins存放初始密码存放位置是：/var/lib/jenkins/secrets/initialAdminPassword）**

**1>.点击当前用户，进入用户配置界面**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908235504245-1593675718.png)**

**2>.点击设置**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908235542784-663104491.png)**

**3>.修改密码**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908235657534-1184652049.png)**

**4>.点击注销**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908235753759-905926889.png)**

**5>.使用新密码登录Jenkins**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908235856750-522789302.png)**

**6>.经过我反复验证，修改密码后，原始存放Jenkins密码文件并没有发生变化**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909000359679-158165001.png)**

**六.Jenkins的其它操作**

**1>.Jenkins的插件安装**

 　　**详情请参考：https://www.cnblogs.com/yinzhengjie/p/9589319.html**

**2>.**

**3>.**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186