## 1、在Windows上安装[Maven](https://so.csdn.net/so/search?q=Maven&spm=1001.2101.3001.7020)

## 1.1 检查JDK安装

在安装Maven之前，首先要确认已经正确安装了JDK。Maven可以运行在JDK1.4及以上的版本上。打开Windows的[命令行](https://so.csdn.net/so/search?q=%E5%91%BD%E4%BB%A4%E8%A1%8C&spm=1001.2101.3001.7020)，运行如下的命令来检查Java安装情况：

```
C:\Users\panjunbiao>echo %Java_Home%C:\Users\panjunbiao>java -version
```

**执行结果：**

![](https://img-blog.csdnimg.cn/20200211160737466.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

上述命令首先检查环境变量Java\_Home是否指向了正确的JDK目录，接着运行Java命令。如果Windows无法执行Java命令，或者无法找到Java\_Home环境变量，就需要检查Java是否安装了，或者环境变量是否设置正确。

## 1.2 下载Maven

Maven的下载页面：[Maven – Download Apache Maven](http://maven.apache.org/download.cgi "Maven – Download Apache Maven")

如下图，点击并下载Maven的压缩文件。

![](https://img-blog.csdnimg.cn/20200211161758925.jpg)

下载完成后，将安装文件解压到指定的目录中。解压后的目录结构如下：

![](https://img-blog.csdnimg.cn/20200211162120466.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

## 1.3 设置环境变量

接着需要设置环境变量，将Maven安装配置到操作系统环境中。打开系统属性面板（在桌面上右键“我的电脑” → “属性”），单击高级系统设置。

### 1.3.1 设置Maven\_Home环境变量

**（1）新建系统变量**

变量名：Maven\_Home

变量值：D:\\apache-maven-3.6.3

![](https://img-blog.csdnimg.cn/2020021116375967.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

**（2）修改Path变量值**

在Path变量值后面加上：;%Maven\_Home%\\bin;

![](https://img-blog.csdnimg.cn/20200211163814782.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

### 1.3.2 设置MAVEN\_OPTS环境变量

变量名：MAVEN\_OPTS

变量值：-Xms128m -Xmx512m

设置MAVEN\_OPTS环境变量不是必须的，但建议设置。因为Java默认的最大可用内存往往不能够满足Maven运行的需要，比如在项目较大时，使用Maven生成项目站点需要占用大量的内存，如果没有该配置，则很容易得到java.lang.OutOfMemeoryError。因此，一开始就配置该变量是推荐的做法。

![](https://img-blog.csdnimg.cn/20200211165008922.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

## 1.4 检查Maven安装

配置完成环境变量后，新增打开一个新的cmd窗口（这里强调新的窗口是因为新的环境变量配置需要新的cmd窗口才能生效），运行如下命令检查Maven的安装情况：

```
C:\Users\panjunbiao>echo %Maven_Home%C:\Users\panjunbiao>mvn -v
```

**执行结果：**

![](https://img-blog.csdnimg.cn/2020021116574764.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

第一条命令echo %Maven\_Home%用来检查环境变量Maven\_Home是否指向了正确的Maven安装目录；而mvn -v是Maven的执行命令，以检查Windows系统是否能够找到正确的mvn执行脚本。

## 2、Maven的配置

## 2.1 生成.m2文件夹

在cmd窗口中执行如下命令：

```
mvn help:system
```

先运行一条简单的命令：mvn help:system。该命令会打印出所有的Java系统属性和环境变量，这些信息对我们日常的变成工作很有帮助。该命令的目的是让Maven执行一个正在的任务。我们可以从命令行输出看到Maven会下载\\maven-help-plugin，包括pom文件和jar文件。这些文件都被下载到了Maven本地仓库中。

现在.m2文件夹就生成了，在用户目录下可以找到.m2文件夹，如：C:\\Users\\panjunbiao\\.m2

默认情况下，该文件夹下放置了Maven本地仓库（C:\\Users\\panjunbiao\\.m2\\repository），所有的Maven构件都被存储到该仓库中，可以方便重用。可以到C:\\Users\\panjunbiao\\.m2\\repository\\org\\apache\\maven\\plugins\\maven-help-plugin目录下找到刚刚下载的maven-help-plugin的pom文件和jar文件。

## 2.2 配置用户范围的settings.xml文件

默认情况下，.m2文件夹下除了repository仓库之外就没有其他目录和文件了，不过大多数Maven用户需要复制安装目录下的D:\\apache-maven-3.6.3\\conf\\settings.xml文件到.m2文件夹下C:\\Users\\panjunbiao\\.m2\\settings.xml。这是一条最佳实践。

Maven用户可以选择配置安装目录下的\\conf\\settings.xml文件或者.m2文件夹下的settings.xml文件。前者是全局范围的，整台机器上的所有用户都会直接受到该配置的影响，而后者是用户范围的，只有当前用户才会受到该配置的影响（推荐）。

## 2.3 配置Maven本地仓库

打开.m2文件夹下的settings.xml文件，添加如下配置：

```
<localRepository>D:\maven-local-repository</localRepository>
```

这样Maven本地仓库就不用在C盘下了。

## 2.4 配置中央仓库的镜像（改用：阿里云中央仓库镜像）

有时我们通过Maven去下载相关的依赖包时，会发现下载的速度非常慢，而有时又下载不了，没有响应。为什么会这么慢呢，原因是Maven默认连接的远程仓库是国外的（http://repo1.maven.org/maven2/）。为了提升下载速度，只要把Maven默认的镜像改换成国内的就行了，如阿里云的中央仓库镜像。

在settings.xml文件下的<mirrors>节点中，添加如下配置：

```
<mirror>        <id>alimaven</id><name>aliyun-maven</name><mirrorOf>central</mirrorOf><url>http://maven.aliyun.com/nexus/content/groups/public</url></mirror>
```

## 3、IntelliJ IDEA中使用Maven

## 3.1 设置IDEA属性

设置IDEA属性，在Build,Execution,Deployment → Build Tools → Maven，如下图：

![](https://img-blog.csdnimg.cn/20200212101947986.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

## 3.2 使用Maven创建项目

![](https://img-blog.csdnimg.cn/2020021210355046.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200212110103813.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

 最终的项目结构如下：

![](https://img-blog.csdnimg.cn/20200212105345430.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhbl9qdW5iaWFv,size_16,color_FFFFFF,t_70)

## 3.3 编写POM.xml配置文件

使用仓库搜索服务可以根据关键字得到Maven坐标，如：[mvnrepository](https://mvnrepository.com/ "mvnrepository")

这里以引入JUnit5的jar包为例，配置信息如下：

```
<dependencies><dependency><groupId>org.junit.platform</groupId><artifactId>junit-platform-launcher</artifactId><scope>test</scope></dependency><dependency><groupId>org.junit.jupiter</groupId><artifactId>junit-jupiter-engine</artifactId><scope>test</scope></dependency><dependency><groupId>org.junit.vintage</groupId><artifactId>junit-vintage-engine</artifactId><scope>test</scope></dependency></dependencies>
```

完整的POM.xml配置文件，如下:

```
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"><modelVersion>4.0.0</modelVersion><groupId>com.pjb.mvnbook</groupId><artifactId>hello-maven</artifactId><version>1.0-SNAPSHOT</version><dependencies><dependency><groupId>org.junit.platform</groupId><artifactId>junit-platform-launcher</artifactId><scope>test</scope></dependency><dependency><groupId>org.junit.jupiter</groupId><artifactId>junit-jupiter-engine</artifactId><scope>test</scope></dependency><dependency><groupId>org.junit.vintage</groupId><artifactId>junit-vintage-engine</artifactId><scope>test</scope></dependency></dependencies></project>
```

## 3.4 编写代码

（1）创建HelloMaven.java类。

```
public class HelloMaven{public String sayHello()    {return "Hello Maven";    }public static void main(String[] args)    {        System.out.println(new HelloMaven().sayHello());    }}
```

（2）创建HelloMavenTest.java测试类。

```
import org.junit.jupiter.api.AfterEach;import org.junit.jupiter.api.BeforeEach;import org.junit.jupiter.api.Test;import static org.junit.jupiter.api.Assertions.*;class HelloMavenTest{@BeforeEachvoid setUp()    {    }@AfterEachvoid tearDown()    {    }@Testvoid sayHello()    {HelloMaven helloMaven = new HelloMaven();String result = helloMaven.sayHello();        System.out.println(result);        assertEquals("Hello Maven",result);    }}
```