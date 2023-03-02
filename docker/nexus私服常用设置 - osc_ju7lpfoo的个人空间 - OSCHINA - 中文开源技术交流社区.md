https://www.jianshu.com/p/7a09915675d9

或许用得上。

settings.xml

```
<server>  　　  <!-- 服务器密码 -->
     <id>user-release</id>  
     <username>admin</username>  
     <password>admin123</password>  
   </server>  
   <server>  
     <id>user-snapshots</id>  
     <username>admin</username>  
     <password>admin123</password>  
   </server>  
</servers>

<!-- 配置私服地址映射 -->
<mirror>
       <id>nexus</id>
       <mirrorOf>*</mirrorOf>
       <name>MyRepository</name>
       <!--私服仓库地址-->
       <url>http://localhost:8081/nexus/content/groups/public</url>
</mirror>

<profile>
        <id>nexus</id>
        <repositories>
            <repository>
                <id>central</id>
                <name>MyRepository</name>
                <url>http://central</url>
                <releases>
                     <enabled>true</enabled>     
                </releases>
                <snapshots>
                     <enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <url>http://central</url>
                <releases>
                     <enabled>true</enabled>
                </releases>
                <snapshots>
                     <enabled>true</enabled>
                </snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>


  <!-- 激活私服映射 -->
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
```

pom.xml

```
<!-- 镜像加速 -->
<repositories>
    <repository>
        <id>nexus</id>
        <name>nexus</name>
            <url>http://localhost:8081/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
</repositories>

<!-- 镜像发布 -->
<distributionManagement>  
      <repository> 
        <!-- 两个ID必须与 setting.xml中的<server><id>nexus-releases</id></server>保持一致-->
          <id>user-release</id>  
          <name>User Project Release</name>  
          <url>http://localhost:8081/nexus/content/repositories/releases/</url>  
        </repository>  
  
     <snapshotRepository>  
        <id>user-snapshots</id>  
         <name>User Project SNAPSHOTS</name>
         <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>  
       </snapshotRepository>  
</distributionManagement>
```

展开阅读全文

本文转载自：https://www.cnblogs.com/aguncn/p/10343038.html