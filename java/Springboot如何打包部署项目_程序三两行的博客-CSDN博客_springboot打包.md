## 前言

这里打包的是jar项目，也就是没有webapp目录，通过[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)打包插件打包发布到服务器

## 1\. pom文件引入插件

```
<build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
```

boot使用这个插件可以将项目打包成一个可运行的jar，无需在目标服务器安装tomcat等

## 2.idea中快速打包

![](https://img-blog.csdnimg.cn/20190612102022440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NDkxNTA4,size_16,color_FFFFFF,t_70)

当然打包之前记得选择自己生产环境配置文件

## 3.java –jar运行项目

我们打开刚才打包之后的所在的位置，确实有一个jar包

![](https://img-blog.csdnimg.cn/20190612102215606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NDkxNTA4,size_16,color_FFFFFF,t_70)

在当前文件路径直接使用cmd通过java –jar运行项目

![](https://img-blog.csdnimg.cn/20190612102336240.png)

## 4.访问项目

[http://localhost:8080/index](http://localhost:8080/hello)

## 5.发布到服务器

发布之前服务器需要已经安装了jdk环境，数据库（项目数据库服务器如果是本服务器）

通过类似xshell等上传文件将上面的jar文件上传到服务器某个文件夹下，在当前路径下运行下面命令即可在后台启动项目，其中hellword-0.0.1-SNAPSHOT.jar是我的项目

nohup java -jar hellword-0.0.1-SNAPSHOT.jar > hellword-0.0.1-SNAPSHOT.log  2>&1 &

补充：其他相关命令

nohup java -jar xxx.jar &

这样执行后，nohup会把执行结果中的日志输出到当前文件夹下面的nohup.out文件中，通常情况下我们使用以上命令即可。 也可手动指定一个参数来规定日志文件的输出地点

nohup java -jar xxx.jar > catalina.out  2>&1 &

如果不需要输出日志，可以使用如下命令

nohup java -jar xxx.jar >/dev/null &

为了方便管理，我们还可以通过Shell来编写一些用于启动应用的脚本，比如关闭应用的脚本：stop.sh

```
#!/bin/bashPID=$(ps -ef | grep yourapp.jar | grep -v grep | awk '{ print $2 }')if [ -z "$PID" ]thenecho Application is already stoppedelseecho kill $PIDkill $PIDfi
```

启动应用的脚本：start.sh

```
#!/bin/bashnohup java -jar yourapp.jar --server.port=8888 &
```

整合了关闭和启动的脚本：run.sh，由于会先执行关闭应用，然后再启动应用，这样不会引起端口冲突等问题，适合在持续集成系统中进行反复调用。

```
#!/bin/bashecho stop applicationsource stop.shecho start applicationsource start.sh
```

在Spring Boot的Maven插件中，还提供了构建完整可执行程序的功能，什么意思呢？就是说，我们可以不用java -jar，而是直接运行jar来执行程序。这样我们就可以方便的将其创建成系统服务在后台运行了。主要步骤如下：在pom.xml中添加Spring Boot的插件，并注意设置executable配置

```
<build>      <plugins>        <plugin>          <groupId>org.springframework.boot</groupId>          <artifactId>spring-boot-maven-plugin</artifactId>          <configuration>            <executable>true</executable>          </configuration>        </plugin>      </plugins>    </build>
```

在完成上述配置后，使用mvn install进行打包，构建一个可执行的jar包，创建软连接到/etc/init.d/目录下

sudo ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp

在完成软连接创建之后，我们就可以通过如下命令对yourapp.jar应用来控制启动、停止、重启操作了

/etc/init.d/yourapp start|stop|restart

如何发布war项目，参考文章

[https://blog.csdn.net/qq\_22638399/article/details/81506448](https://blog.csdn.net/qq_22638399/article/details/81506448)