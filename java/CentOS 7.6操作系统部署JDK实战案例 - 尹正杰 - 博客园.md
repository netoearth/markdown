　　　　　　　　　　**CentOS 7.6操作系统部署JDK实战案例**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。** 

 　　**本篇笔记只介绍Linux部署JDK，关于Windows部署JDK可参考:https://www.cnblogs.com/yinzhengjie2020/p/12206579.html。**

**一.安装openjdk**

**1>.查看JDK版本**

```
[root@master200.yinzhengjie.org.cn ~]# yum list all | grep jdk
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215230104272-472326555.png)

**2>.安装openjdk**

```
[root@master200.yinzhengjie.org.cn ~]# yum -y install java-1.8.0-openjdk.x86_64
```

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215230332438-1335319966.png)

**3>.验证java是否安装成功**

```
[root@master200.yinzhengjie.org.cn ~]# java -version
```

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215230418799-504389030.png)

**二.基于rpm方式安装Oracle JDK**

**1>.下载Oracle JDK**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie2020/p/12206579.html
```

**2>.下载rpm包后直接使用rpm命令安装即可**

```
[root@node201.yinzhengjie.org.cn ~]# rpm -ivh jdk-8u211-linux-x64.rpm 
```

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215231403452-2042049059.png)

**3>.配置JDK\_HOME变量**

```
[root@node201.yinzhengjie.org.cn ~]# cat /etc/profile.d/jdk.sh
export JAVA_HOME=/usr/java/default
export PATH=$JAVA_HOME/bin:$PATH
[root@node201.yinzhengjie.org.cn ~]# 
```

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215232339266-1888086307.png)

**三.**基于二进制方式安装Oracle JDK****

**1>.****下载Oracle JDK的二进制安装包**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie2020/p/12206579.html
```

**2>.将下载的二进制JDK安装包上传到服务器并解压到指定的位置**

```
[root@node202.yinzhengjie.org.cn ~]# tar zxf jdk-8u211-linux-x64.tar.gz -C /yinzhengjie/softwares/
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215233334765-1082339421.png)

**3>.配置JAVA\_HOME及环境变量**

```
[root@node202.yinzhengjie.org.cn ~]# cat /etc/profile.d/jdk.sh 
export JAVA_HOME=/yinzhengjie/softwares/jdk1.8.0_211
export PATH=$JAVA_HOME/bin:$PATH
[root@node202.yinzhengjie.org.cn ~]# 
```

 **![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200215233730532-1654613023.png)**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186