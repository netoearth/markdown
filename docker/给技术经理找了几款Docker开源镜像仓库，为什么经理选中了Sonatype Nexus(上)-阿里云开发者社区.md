docker使用教程相关系列 目录

目录

一、场景

二、分析报告结论

总体分析表

1、Docker Registry

2、VMware Harbor

3、Sonatype Nexus

4、SUSE Portus

三、技术经理总结

四、Nexus介绍

五、拉取镜像

1、查找镜像

2、拉取镜像

六、启动容器

启动容器报错

解决方案：

七、访问并配置nexus

一、场景

市民赵铁柱在A公司担任开发工程师。

技术经理让赵铁柱找适合公司的Docker开源镜像仓库，并形成分析报告提交给经理。赵铁柱通过身边的人脉咨询和网上查找资料，最终选定了四款仓库。

1、Docker Registry

2、VMware Harbor

3、Sonatype Nexus

4、SUSE Portus

二、分析报告结论

总体分析表

 ![image.png](https://ucc.alicdn.com/pic/developer-ecology/863d78bec1ec42a388f40ef2cb859427.png "image.png")1、Docker Registry

Docker Registry是最流行的开源私有镜像仓库，以镜像格式发布，在下载后运行一个Docker Registry容器即可启动一个私有镜像仓库服务。

Docker Registry的有点如下：

Docker Registry的最大优点就是简单，只需要运行一个容器就能集中管理一个集群范围内的镜像，其他机器就能从该镜像仓库下载镜像了。

在安全性方面，Docker Registry支持TLS和基于签名的身份验证。

Docker Registry也提供了Restful API，以提供外部系统调用和管理镜像库中的镜像

2、VMware Harbor

VMware Harbor（简称Harbor）项目是由VMware中国研发团队开发的开源容器镜像仓库系统，基于Docker Registry并对其进行了许多增强，主要特性包括：

基于角色的访问控制

镜像复制

Web UI管理界面

可以集成LDAP或AD用户认证系统

审计日志

提供RESTful API以提供外部客户端调用

镜像安全漏洞扫描（从v1.2版本开始集成了Clair景象扫描工具）

Harbor相对于Docker Registry，提供了更好的用户管理、角色权限管理、审计日志，以及多个Harbor镜像仓库之间的镜像复制功能，可以用作企业私有镜像库的服务器。不过由于Harbor的组件较多，所以与外界的集成较为复杂。

3、Sonatype Nexus

Sonatype Nexus是一个软件仓库管理器，主要有2.X和3.X两个大版本。2.X版本主要支持Maven、P2、OBR、Yum等仓库软件；3.X版本主要支持Docker、NuGet、npm、Bower、PyPI、Ruby Gems、Apt、Conam、R、CPAN、Raw、Helm等仓库软件，也支持构建工具Maven。

Sonatype Nexus的特点如下：

部署简单，通过启动一个容器即可完成

支持TLS安全认证

提供Web UI管理界面

支持代理仓库（Docker Proxy），可以将到Nexus镜像仓库的操作代理到另一个远程镜像库

支持仓库组（Docker Group），可以把多个仓库组合成一个地址提供服务

除了支持Docker镜像，还支持对其他软件仓库的管理，例如：Yum、Npm等。

4、SUSE Portus

SUSE Portus是另一个开源镜像库，其特点包括：

基于组（Team）和命名空间（Namespace）的细粒度访问权限控制

Web UI管理界面

可以集成LDAP用户认证系统，也支持OAuth

审计日志

提供RESTful API，以供外部客户端调用

镜像安全漏洞扫描（集成Clair镜像扫描工具）

三、技术经理总结

1、公司现在已经在使用Nexus当Maven私服，Nexus3又支持docker，到时用一套私服仓库就可以一举多得。

2、Harbor功能强大，但组件多，配置和运维的复杂度高，增加运维难度。

3、Docker Registry不满足公司需要，而且也没有图形界面管理；SUSE Portus跟Nexus有差不多的功能，最终公司还是选择Nexus3。

四、Nexus介绍

"Docker官方镜像仓库" 访问速度很慢，Sonatype Nexus允许搭建我们自己的镜像仓库，为实现镜像拉取、推送提供便利。

Sonatype Nexus是一个软件仓库管理器，主要有2.X和3.X两个大版本。2.X版本主要支持Maven、P2、OBR、Yum等仓库软件；3.X版本主要支持Docker、NuGet、npm、Bower、PyPI、Ruby Gems、Apt、Conam、R、CPAN、Raw、Helm等仓库软件，也支持构建工具Maven。

五、拉取镜像

1、查找镜像

 ![image.png](https://ucc.alicdn.com/pic/developer-ecology/e765cec9753a43159d54b554d2a49e71.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/cec944152a484e2e938196173f5e6601.png "image.png")![image.png](https://ucc.alicdn.com/pic/developer-ecology/213dce3e076441bab493db3192a4dbe8.png "image.png")

 ![image.png](https://ucc.alicdn.com/pic/developer-ecology/8130cc29980c48e2b087a9a6020f03d4.png "image.png")![image.png](https://ucc.alicdn.com/pic/developer-ecology/3e8a6be85ae34384a86c0449a9d71ccb.png "image.png")

```
mkdir: cannot create directory '../sonatype-work/nexus3/log': Permission denied
mkdir: cannot create directory '../sonatype-work/nexus3/tmp': Permission denied
OpenJDK 64-Bit Server VM warning: Cannot open file ../sonatype-work/nexus3/log/jvm.log due to No such file or directory
 
Warning:  Cannot open log file: ../sonatype-work/nexus3/log/jvm.log
Warning:  Forcing option -XX:LogFile=/tmp/jvm.log
java.io.FileNotFoundException: ../sonatype-work/nexus3/tmp/i4j_ZTDnGON8hezynsMX2ZCYAVDtQog=.lock (No such file or directory)
  at java.io.RandomAccessFile.open0(Native Method)
  at java.io.RandomAccessFile.open(RandomAccessFile.java:316)
  at java.io.RandomAccessFile.<init>(RandomAccessFile.java:243)
  at com.install4j.runtime.launcher.util.SingleInstance.check(SingleInstance.java:72)
  at com.install4j.runtime.launcher.util.SingleInstance.checkForCurrentLauncher(SingleInstance.java:31)
  at com.install4j.runtime.launcher.UnixLauncher.checkSingleInstance(UnixLauncher.java:88)
  at com.install4j.runtime.launcher.UnixLauncher.main(UnixLauncher.java:67)
java.io.FileNotFoundException: /nexus-data/karaf.pid (Permission denied)
  at java.io.FileOutputStream.open0(Native Method)
  at java.io.FileOutputStream.open(FileOutputStream.java:270)
  at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
  at java.io.FileOutputStream.<init>(FileOutputStream.java:101)
  at org.apache.karaf.main.InstanceHelper.writePid(InstanceHelper.java:127)
  at org.apache.karaf.main.Main.launch(Main.java:243)
  at org.sonatype.nexus.karaf.NexusMain.launch(NexusMain.java:113)
  at org.sonatype.nexus.karaf.NexusMain.main(NexusMain.java:52)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at com.exe4j.runtime.LauncherEngine.launch(LauncherEngine.java:85)
  at com.install4j.runtime.launcher.UnixLauncher.main(UnixLauncher.java:69)
java.lang.RuntimeException: /nexus-data/log/karaf.log (No such file or directory)
  at org.apache.karaf.main.util.BootstrapLogManager.getDefaultHandlerInternal(BootstrapLogManager.java:102)
  at org.apache.karaf.main.util.BootstrapLogManager.getDefaultHandlersInternal(BootstrapLogManager.java:137)
  at org.apache.karaf.main.util.BootstrapLogManager.getDefaultHandlers(BootstrapLogManager.java:70)
  at org.apache.karaf.main.util.BootstrapLogManager.configureLogger(BootstrapLogManager.java:75)
  at org.apache.karaf.main.Main.launch(Main.java:244)
  at org.sonatype.nexus.karaf.NexusMain.launch(NexusMain.java:113)
  at org.sonatype.nexus.karaf.NexusMain.main(NexusMain.java:52)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at com.exe4j.runtime.LauncherEngine.launch(LauncherEngine.java:85)
  at com.install4j.runtime.launcher.UnixLauncher.main(UnixLauncher.java:69)
Caused by: java.io.FileNotFoundException: /nexus-data/log/karaf.log (No such file or directory)
  at java.io.FileOutputStream.open0(Native Method)
  at java.io.FileOutputStream.open(FileOutputStream.java:270)
  at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
  at org.apache.karaf.main.util.BootstrapLogManager$SimpleFileHandler.open(BootstrapLogManager.java:193)
  at org.apache.karaf.main.util.BootstrapLogManager$SimpleFileHandler.<init>(BootstrapLogManager.java:182)
  at org.apache.karaf.main.util.BootstrapLogManager.getDefaultHandlerInternal(BootstrapLogManager.java:100)
  ... 12 more
Error creating bundle cache.
```

 ![image.png](https://ucc.alicdn.com/pic/developer-ecology/e4290e64886445e5b0f660ff3ef0077f.png "image.png")再运行启动新命令

在原先的命令基础上加了“--privileged=true”

```
docker run -p 8081:8081 --privileged=true --name nexus -v /usr/local/docker/nexus/nexus-data:/nexus-data 8716903d1912
```

注：\--privileged，该参数可以设置是否给docker容器特权，如果该参数为true，使得docker容器内的root权限为宿主机的root权限，而非只是容器内的root权限

看下服务是否启动正常

![image.png](https://ucc.alicdn.com/pic/developer-ecology/f7c8c6bfc9444e70b3aaedbb973e987f.png "image.png")

## 七、访问并配置nexus

打开浏览器，访问 http://:8081/

注：有时启动会比较慢，要等会。。等不及的话，可以看下日志

出现这个界面，就说明启动成功了  ![image.png](https://ucc.alicdn.com/pic/developer-ecology/49f2328c23f54d8f81cc9ee6305a036f.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/132a14a6f07f4322920fd6210c34b873.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/fa7bff48810d4fb7bc99dc2dc45d852a.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/286010d13cd84c2eb7bad406d9c3a8f5.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/a98423177f7a4886b6ced8cd65001a31.png "image.png") ![image.png](https://ucc.alicdn.com/pic/developer-ecology/3c70051235a549de9bd99104ce49d0fe.png "image.png")未完待续。。

[给技术经理找了几款Docker开源镜像仓库，为什么经理选中了Sonatype Nexus(下)](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw)

参考：常见的几种开源镜像仓库介绍

[https://blog.csdn.net/Andriy\_dangli/article/details/84381383](https://blog.csdn.net/Andriy_dangli/article/details/84381383)