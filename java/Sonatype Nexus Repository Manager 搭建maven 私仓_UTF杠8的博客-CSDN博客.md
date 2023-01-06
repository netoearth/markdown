**前言**：

下载安装  Nexus Repository Manager      默认账号/密码  :   admin/admin123   视版本而定，这里安装不做深究，主要对如何配置[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020) 私仓  和  idea 中 deploy  jar 包 进行记录。

![](https://img-blog.csdnimg.cn/20200109154556447.png)

### 0.首先创建   四个Repository 对应的各个Repository 的 type  需要注意。

_group_(仓库组)、_hosted_(宿主仓库)、proxy(代理仓库）

![](https://img-blog.csdnimg.cn/20200109144235157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5Mjk4NTc3,size_16,color_FFFFFF,t_70)

### 1.maven-central：         type 为  proxy   意为 仓库中没有对应的jar 包是需要去远程   仓库下载。我这在创建的时候  proxy   填的是阿里的 [http://maven.aliyun.com/nexus/content/groups/public/](http://maven.aliyun.com/nexus/content/groups/public/)   国内仓库快些。

![](https://img-blog.csdnimg.cn/20200109152106223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5Mjk4NTc3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200109145221929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5Mjk4NTc3,size_16,color_FFFFFF,t_70)

### 2.maven-public：          type  为  group  其中包含一些其他的仓库配置时可选    选定想要的仓库组组成。

![](https://img-blog.csdnimg.cn/20200109145801298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5Mjk4NTc3,size_16,color_FFFFFF,t_70)

### 3.maven-releases   和 maven-snapshots   type   为  host     他们俩的区别在于   里边的权限分别为

![](https://img-blog.csdnimg.cn/20200109150400484.png)![](https://img-blog.csdnimg.cn/20200109150457701.png)

之后就需要配置maven 的settings.xml  

1.配置私仓地址 

> ```
> <mirror>  <id>nexus</id> <mirrorOf>*</mirrorOf>    <name>Nexus saic</name>  <url>http://localhost:8081/repository/*-public/</url>  </mirror>
> ```

2.配置私仓的用户名密码 。这里需要记住这个id   之后的maven  项目中的pom.xml  需要使用

> ```
> <server><id>nexus</id><username>admin</username><password>admin123</password></server>
> ```
> 
> 这里需要记住这个id

3.配置profile

> ```
> <profile><id>nexus</id><repositories><repository><id>snapshots</id><url>http://localhost:8081/repository/*-snapshots/</url><releases><enabled>true</enabled></releases><snapshots><enabled>true</enabled></snapshots></repository><repository><id>releases</id><url>http://localhost:8081/repository/*-releases/</url><releases><enabled>true</enabled></releases><snapshots><enabled>true</enabled></snapshots></repository></repositories><pluginRepositories><pluginRepository><id>snapshots</id><url>http://localhost:8081/repository/*-snapshots/</url><releases><enabled>true</enabled></releases><snapshots><enabled>true</enabled></snapshots></pluginRepository><pluginRepository><id>releases</id><url>http://localhost:8081/repository/*-releases/</url><releases><enabled>true</enabled></releases><snapshots><enabled>true</enabled></snapshots></pluginRepository></pluginRepositories></profile>
> ```

4.启用对应id的这个profile

> ```
> <activeProfiles><activeProfile>nexus</activeProfile></activeProfiles>
> ```

配置  maven  项目中的  pom.xml   添加

这其中的id   就需要和上边的settings.xml  中的server   节点的 id对应上。找到对应的用户名密码

> ```
> <distributionManagement><repository><id>nexus</id><name>releases</name><url>http://localhost:8081/repository/*-releases/</url></repository><snapshotRepository><id>nexus</id><name>snapshots</name><url>http://localhost:8081/repository/*-snapshots/</url></snapshotRepository></distributionManagement>
> ```
> 
> 注意：
> 
> deploy  snapshots  需要  <version>1.0.1-RELEASE</version>
> 
> deploy releases  需要   <version>1.0.1-SNAPSHOT</version>

如有问题请留言。