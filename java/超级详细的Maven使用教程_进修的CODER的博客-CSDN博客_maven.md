## 什么是[Maven](https://so.csdn.net/so/search?q=Maven&spm=1001.2101.3001.7020)？

如今我们构建一个项目需要用到很多第三方的类库，如写一个使用Spring的[Web项目](https://so.csdn.net/so/search?q=Web%E9%A1%B9%E7%9B%AE&spm=1001.2101.3001.7020)就需要引入大量的jar包。一个项目Jar包的数量之多往往让我们瞠目结舌，并且Jar包之间的关系错综复杂，一个Jar包往往又会引用其他Jar包，缺少任何一个Jar包都会导致项目编译失败。

以往开发项目时，程序员往往需要花较多的精力在引用Jar包搭建项目环境上，而这一项工作尤为艰难，少一个Jar包、多一个Jar包往往会报一些让人摸不着头脑的异常。

而Maven就是一款帮助程序员构建项目的工具，我们只需要告诉Maven需要哪些Jar 包，它会帮助我们下载所有的Jar，极大提升开发效率。

## 安装Maven 和 Maven的Eclipse插件

 [安装Maven和Maven的Eclipse插件](http://blog.csdn.net/qjyong/article/details/9098213%C2%A0%C2%A0 "安装Maven和Maven的Eclipse插件")，用Eclipse的同学可以参考，推荐使用IDEA开发；

## Maven规定的目录结构

若要使用Maven，那么项目的目录结构必须符合Maven的规范，其目录结构如下：

![](https://img-blog.csdn.net/201808131239225?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdmVxdWFucXVxbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## Maven基本命令

1.  **\-v:**查询Maven版本
    
    本命令用于检查maven是否安装成功。
    
    Maven安装完成之后，在命令行输入**mvn -v**，若出现maven信息，则说明安装成功。
    
2.  **compile**：编译
    
    将java源文件编译成class文件
    
3.  **test**:测试项目
    
    执行test目录下的测试用例
    
4.  **package**:打包
    
    将项目打成jar包
    
5.  **clean**:删除target文件夹
    
6.  **install**:安装
    
    将当前项目放到Maven的本地仓库中。供其他项目使用
    

## 什么是Maven仓库？

Maven仓库用来存放Maven管理的所有Jar包。分为：本地仓库 和 中央仓库。

-   **本地仓库**：Maven本地的Jar包仓库。
-   **中央仓库**： Maven官方提供的远程仓库。

当项目编译时，Maven首先从本地仓库中寻找项目所需的Jar包，若本地仓库没有，再到Maven的中央仓库下载所需Jar包。

## 什么是“坐标”？

在Maven中，坐标是Jar包的唯一标识，Maven通过坐标在仓库中找到项目所需的Jar包。

如下代码中，groupId和artifactId构成了一个Jar包的坐标。

```
<dependency><groupId>cn.missbe.web.search</groupId><artifactId>resource-search</artifactId><packaging>jar</packaging><version>1.0-SNAPSHOT</version></dependency>
```

-   **groupId**:所需Jar包的项目名
-   **artifactId**:所需Jar包的模块名
-   **version**:所需Jar包的版本号

## 传递依赖 与 排除依赖

-   传递依赖：如果我们的项目引用了一个Jar包，而该Jar包又引用了其他Jar包，那么在默认情况下项目编译时，Maven会把直接引用和间接引用的Jar包都下载到本地。
-   排除依赖：如果我们只想下载直接引用的Jar包，那么需要在pom.xml中做如下配置：(将需要排除的Jar包的坐标写在中)

```
<exclusions><exclusion><groupId>cn.missbe.web.search</groupId><artifactId>resource-search</artifactId><packaging>pom</packaging><version>1.0-SNAPSHOT</version></exclusion></exclusions>
```

## **依赖范围scope**

在项目发布过程中，帮助决定哪些构件被包括进来。欲知详情请参考依赖机制。      
\- **compile** ：默认范围，用于编译        
\- **provided**：类似于编译，但支持你期待jdk或者容器提供，类似于classpath        
\- **runtime**: 在执行时需要使用        
\- **test**:    用于test任务时使用        
\- **system**: 需要外在提供相应的元素。通过systemPath来取得        
\- **systemPath**: 仅用于范围为system。提供相应的路径        
\- **optional**:   当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用

## 依赖冲突

若项目中多个Jar同时引用了相同的Jar时，会产生依赖冲突，但Maven采用了两种避免冲突的策略，因此在Maven中是不存在依赖冲突的。

**1.短路优先**

```
本项目——>A.jar——>B.jar——>X.jar本项目——>C.jar——>X.jar
```

若本项目引用了A.jar，A.jar又引用了B.jar，B.jar又引用了X.jar，并且C.jar也引用了X.jar。

在此时，Maven只会引用引用路径最短的Jar。

**2.声明优先**

若引用路径长度相同时，在pom.xml中谁先被声明，就使用谁。

## 聚合

1.  什么是聚合？
    
    将多个项目同时运行就称为聚合。
    
2.  如何实现聚合？
    
    只需在pom中作如下配置即可实现聚合：
    

```
<modules><module>web-connection-pool</module><module>web-java-crawler</module></modules>
```

## 继承

1.  什么是继承？
    
    在聚合多个项目时，如果这些被聚合的项目中需要引入相同的Jar，那么可以将这些Jar写入父pom中，各个子项目继承该pom即可。
    
2.  如何实现继承？
    

-   父pom配置：将需要继承的Jar包的坐标放入标签即可。
    

```
<dependencyManagement><dependencies><dependency><groupId>cn.missbe.web.search</groupId><artifactId>resource-search</artifactId><packaging>pom</packaging><version>1.0-SNAPSHOT</version></dependency> </dependencies></dependencyManagement>
```

-   子pom配置：

```
<parent><groupId>父pom所在项目的groupId</groupId><artifactId>父pom所在项目的artifactId</artifactId><version>父pom所在项目的版本号</version></parent><parent><artifactId>resource-search</artifactId><groupId>cn.missbe.web.search</groupId><version>1.0-SNAPSHOT</version></parent>
```

## 使用Maven构建Web项目

1.  New Maven项目：选择WebApp：
    
2.  若使用JSP，需添加Servlet依赖：
    
    注：Servlet依赖只在编译和测试时使用！
    

```
<dependency><groupId>javax.servlet</groupId><artifactId>javax.servlet-api</artifactId><version>3.0.1</version><scope>provided</scope></dependency>
```

1.  在Bulid Path中设置resource输出目录：
    
2.  勾选：Dynamic Web Module
    
3.  删掉测试目录
    
4.  在pom中加入jetty的插件，并设置JDK版本：
    

```
<plugins> <plugin> <groupId>org.apache.maven.plugins</groupId>  <artifactId>maven-compiler-plugin</artifactId>  <configuration> <source>1.8</source>  <target>1.8</target> </configuration> </plugin>  <plugin> <groupId>org.eclipse.jetty</groupId>  <artifactId>jetty-maven-plugin</artifactId>  <version>9.3.10.v20160621</version>  <executions> <execution> <phase>package</phase> </execution> </executions> </plugin> </plugins>
```

1.  运行项目：
    
2.  输入：jetty:run
    
3.  访问127.0.0.1:8080
    
    若出现如下界面，表示成功！
    

## pom.xml详解

pom.xml是Maven的核心，你的项目需要什么Jar包就在pom.xml里面配置。当编译项目时Maven读取该文件，并从仓库中下载相应的Jar包。

```
<?xml version="1.0" encoding="utf-8"?><project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0http://maven.apache.org/maven-v4_0_0.xsd">  <parent> <artifactId/>  <groupId/>  <version/> </parent>  <modelVersion>4.0.0</modelVersion>  <groupId>cn.missbe.web</groupId>  <artifactId>search-resources</artifactId>  <packaging>war</packaging>  <version>1.0-SNAPSHOT</version>  <name>search-resources</name>  <url>http://www.missbe.cn</url>  <description>A maven project to study maven.</description>  <prerequisites> <maven/> </prerequisites>  <build> <sourceDirectory/>  <scriptSourceDirectory/>  <testSourceDirectory/>  <outputDirectory/>  <testOutputDirectory/>  <extensions> <extension> <groupId/>  <artifactId/>  <version/> </extension> </extensions>  <resources> <resource> <targetPath/>  <filtering/>  <directory/>  <includes/>  <excludes/> </resource> </resources>  <testResources> <testResource> <targetPath/><filtering/><directory/><includes/><excludes/> </testResource> </testResources>  <directory/>  <finalName/>  <filters/>  <pluginManagement> <plugins> <plugin> <groupId/>  <artifactId/>  <version/>  <extensions/>  <executions> <execution> <id/>  <phase/>  <goals/>  <inherited/>  <configuration/> </execution> </executions>  <dependencies> <dependency>......</dependency> </dependencies>  <inherited/>  <configuration/> </plugin> </plugins> </pluginManagement>  <plugins> <plugin> <groupId/><artifactId/><version/><extensions/>  <executions> <execution> <id/><phase/><goals/><inherited/><configuration/> </execution> </executions>  <dependencies> <dependency>......</dependency> </dependencies>  <goals/><inherited/><configuration/> </plugin> </plugins> </build>  <modules/>  <repositories> <repository> <releases> <enabled/>  <updatePolicy/>  <checksumPolicy/> </releases>  <snapshots> <enabled/><updatePolicy/><checksumPolicy/> </snapshots>  <id>banseon-repository-proxy</id>  <name>banseon-repository-proxy</name>  <url>http://192.168.1.169:9999/repository/</url>  <layout>default</layout> </repository> </repositories>  <pluginRepositories> <pluginRepository>......</pluginRepository> </pluginRepositories>  <dependencies> <dependency> <groupId>org.apache.maven</groupId>  <artifactId>maven-artifact</artifactId>  <version>3.8.1</version>  <type>jar</type>  <classifier/>  <scope>test</scope>  <systemPath/>  <exclusions> <exclusion> <artifactId>spring-core</artifactId>  <groupId>org.springframework</groupId> </exclusion> </exclusions>  <optional>true</optional> </dependency> </dependencies>  <dependencyManagement> <dependencies> <dependency>......</dependency> </dependencies> </dependencyManagement>  <distributionManagement> <repository> <uniqueVersion/>  <id>banseon-maven2</id>  <name>banseon maven2</name>  <url>file://${basedir}/target/deploy</url>  <layout/> </repository>  <snapshotRepository> <uniqueVersion/>  <id>banseon-maven2</id>  <name>Banseon-maven2 Snapshot Repository</name>  <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>  <layout/> </snapshotRepository>  <site> <id>banseon-site</id>  <name>business api website</name>  <url>scp://svn.baidu.com/banseon:/var/www/localhost/banseon-web</url> </site>  <downloadUrl/>  <status/> </distributionManagement>  <properties/> </project>
```

___

参考原文：[Maven使用详解](https://blog.csdn.net/u010425776/article/details/52027706 "Maven使用详解")