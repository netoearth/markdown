### 下载和安装

下载地址：[https://help.sonatype.com/repomanager3/download](https://help.sonatype.com/repomanager3/download)

[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190605232351200-907950787.png)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190605232351200-907950787.png)

注意：Nexus Repository Manager 3是一个Java服务器应用程序，安装需要 **jdk1.8**以上的版本。

下载解压后，用命令行到解压目录的bin目录下运行 **nexus.exe /run**（Linux运行./nexus run），启动完成后会显示“Started Sonatype Nexus”：

```
Copy-------------------------------------------------

Started Sonatype Nexus OSS 3.16.2-01

-------------------------------------------------

```

### 访问Nexus管理后台

Nexus管理后台地址：[http://localhost:8081/](http://localhost:8081/)  
点击右上角Sign in登录，默认账号和密码为：admin/admin123  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610213015661-702492136.jpg)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610213015661-702492136.jpg)

在Repositories 仓库管理界面中有多种默认的仓库，也可以添加新的仓库，本实例直接使用默认的仓库：  
**maven-central**，Type为proxy，表示代理仓库。代理仓库用来代理远程仓库（maven-central代理的是超级POM中配置的Maven中央仓库），当在下载组件时，如果代理仓库搜索不到，则会把请求转发到远程仓库从远程仓库下载。从远程仓库下载后会缓存到代理仓库，下次还有该组件的请求则会直接到代理仓库下载，不会再次请求远程仓库。

**maven-releases/maven-snapshots**，Type为hosted，表示为宿主仓库。宿主仓库主要用来部署团队内部使用的内部组件，默认的maven-releases和maven-snapshots分别用来部署团队内部的发布版本组件和快照版本组件。  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610231600331-1232886071.png)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610231600331-1232886071.png)

### 配置代理仓库

配置settings.xml：

```
Copy<settings>
   <!-- 配置镜像，此处拦截所有远程仓库的请求到代理仓库-->
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://localhost:8081/repository/maven-central/</url>
    </mirror>
  </mirrors>
  <!-- 配置远程库和远程插件库-->
  <profiles>
    <profile>
      <id>nexus</id>
      <!-- Maven用于填充构建系统本地存储库的远程仓库集合-->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <!-- 类似于repositories元素，指定Maven可以在哪里找到Maven插件的远程仓库位置-->
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <!-- 激活profiles配置 -->
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>

```

创建Maven项目，pom.xml如下：

```
Copy<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.nocoffee</groupId>
  <artifactId>coffee-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>coffee-api</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>

```

执行mvn clean，执行mvn clean需要下载maven-clean-plugin插件，通过Browse界面可以看到因为执行mvn clean而下载的maven-clean-plugin.jar：  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610225332639-1450157051.jpg)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610225332639-1450157051.jpg)

**注意：如果界面为空，表示没有下载，原因是之前下载过该插件到本地仓库，需要把本地仓库的maven-clean-plugin插件删除，我的本地仓库路径为D:\\Reporsitory，所以需要删掉文件夹：D:\\Reporsitory\\org\\apache\\maven\\plugins\\maven-clean-plugin，然后重新构建即可。**

### 配置宿主仓库

settings.xml增加如下配置：

```
Copy<servers>
     <server>
        <id>nexus</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>

```

配置pom.xml：

```
Copy<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.nocoffee</groupId>
  <artifactId>coffee-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>coffee-api</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  
  <distributionManagement>
<repository>
<id>nexus</id>
<name>maven-releases</name>
<url>http://localhost:8081/repository/maven-releases/</url>
</repository>
<snapshotRepository>
<id>nexus</id>
<name>maven-snapshots</name>
<url>http://localhost:8081/repository/maven-snapshots/</url>
</snapshotRepository>
   </distributionManagement>
</project>

```

执行mvn clean deploy将项目打包并发布到宿主仓库，构建成功后到Browse中maven-snapshots库查看（因为项目版本为0.0.1-SNAPSHOT，是带SNAPSHOT的快照版本）：  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610230350460-1227751448.jpg)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610230350460-1227751448.jpg)

### maven-releases库

需要将项目版本改成发布版本，在pom.xml中0.0.1-SNAPSHOT去掉-SNAPSHOT，改为0.0.1。重新执行mvn clean deploy：  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610230653502-956794239.jpg)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610230653502-956794239.jpg)

**注意：maven-releases库默认不能重新发布，需要可重新发布则需要修改该仓库配置。**  
测试重新发布到maven-releases库，执行mvn clean deploy将会构建失败：

```
Copy[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.112 s
[INFO] Finished at: 2019-06-10T16:34:29+08:00
[INFO] Final Memory: 18M/164M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy (default-deploy) on project coffee-api: Failed to deploy artifacts: Could not transfer artifact com.nocoffee:coffee-api:jar:0.0.1 from/to nexus (http://localhost:8081/repository/maven-releases/): Failed to transfer file: http://localhost:8081/repository/maven-releases/com/nocoffee/coffee-api/0.0.1/coffee-api-0.0.1.jar. Return code is: 400, ReasonPhrase: Repository does not allow updating assets: maven-releases. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException

```

将maven-releases库中Deployment pollcy改为Allow redeploy既可：  
[![](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610231242324-1035193821.jpg)](https://img2018.cnblogs.com/blog/945558/201906/945558-20190610231242324-1035193821.jpg)