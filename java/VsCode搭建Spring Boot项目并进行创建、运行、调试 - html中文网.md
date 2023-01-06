源码地址：[](https://github.com/YANGKANG01/Spring-Boot-Demo)

**安装扩展**

安装如下两个主要扩展即可，这两个扩展已关联java项目开发主要使用的maven、springboot等所需要的扩展。

![](https://ss.html.cn/article/39/b9/9a/39b99ad149c01d6fbf427544e8754a65.jpg-600)

开始步骤：

1.  在 Visual Studio Code 中打开扩展视图(Ctrl+Shift+X)。
2.  输入“java”搜索商店扩展插件。
3.  找到并安装[Java Extension Pack (Java 扩展包)](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)，如果你已经安装了[Language Support for Java(TM) by Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.java)，也可以单独找到并安装[Java Debugger for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug)扩展。
4.  输入“Spring Boot Extension”搜索商店扩展插件。
5.  找到并安装 “Spring Boot Extension Pack”。安装过程中可能会比较慢，耐心等待即可。

**配置Maven：**

点左下角的设置图标->设置，打开设置内容筛选框，输入maven，然后点击右侧的打开json格式setting：

![](https://ss.html.cn/article/d4/59/8c/d4598c39aa875e4cf385186f50f6bb93.jpg-600)

然后把maven的可执行文件路径配置、maven的setting路径配置、java.home的路径配置，拷贝到右侧的用户设置区域并且设置为自己电脑的实际路径

![](https://ss.html.cn/article/17/b8/b8/17b8b8f9f8d899f8ce62733589a1d46c.jpg-600)

设置内容如下：

```
 { "workbench.iconTheme": "vscode-icons", "workbench.startupEditor": "newUntitledFile", "java.errors.incompleteClasspath.severity": "ignore", "workbench.colorTheme": "Atom One Dark", "java.home":"D:\\software\\Java\\jdk1.8.0_60", "java.configuration.maven.userSettings": "D:\\software\\apache-maven-3.3.3-bin\\apache-maven-3.3.3\\conf\\settings.xml", "maven.executable.path": "D:\\software\\apache-maven-3.3.3-bin\\apache-maven-3.3.3\\bin\\mvn.cmd", "maven.terminal.useJavaHome": true, "maven.terminal.customEnv": [ { "environmentVariable": "JAVA_HOME", "value": "D:\\software\\Java\\jdk1.8.0_60" } ], }
```

如果你的mvn更新包速度很慢，建议使用阿里云的镜像速度会快点（修改maven的setting配置如下）：

```
  alimavencentralaliyun mavenhttp://maven.aliyun.com/nexus/content/repositories/central/ nexus-aliyun*Nexus aliyunhttp://maven.aliyun.com/nexus/content/groups/public repo1centralHuman Readable Name for this Mirror.http://repo1.maven.org/maven2/ repo2centralHuman Readable Name for this Mirror.http://repo2.maven.org/maven2/
```

配置完成重启VSCode。

**创建Spring Boot项目**

使用快捷键(Ctrl+Shift+P)命令窗口，输入 Spring 选择创建 Maven 项目。 效果如下：

![](https://ss.html.cn/article/84/c2/5b/84c25b304033ae520b52327d64eb2e99.jpg-600)

选择需要使用的语言、Group Id、项目名称等，这里选择Java：

![](https://ss.html.cn/article/f5/0b/b8/f50bb8eb0aa01966c0d3546c8c961dc5.jpg-600)

![](https://ss.html.cn/article/e8/14/63/e81463fc2b9dd93c9caa7a48ad875a58.jpg-600)

![](https://ss.html.cn/article/f4/ad/cb/f4adcb6e2f5de65e5047dd9f21b3f9cb.jpg-600)

选择Spring Boot版本：

![](https://ss.html.cn/article/20/9e/4f/209e4f62bfbcd8c1d61640deead94d46.jpg-600)

选择需要引入的包，引入如下几个包即可满足web开发：

DevTools（代码修改热更新，无需重启）、Web（集成tomcat、SpringMVC）、Lombok（智能生成setter、getter、toString等接口，无需手动生成，代码更简介）、Thymeleaf（模板引擎）。

选择好要引入的包后直接回车，在新弹出的窗口中选择项目路径，至此Spring Boot项目创建完成。

![](https://ss.html.cn/article/c2/19/5e/c2195e5b37d8626b2061ba2a279887d6.jpg-600)

创建好后vscode右下角会有如下提示，点击Open it 即可打开刚才创建的Spring Boot项目。

![](https://ss.html.cn/article/76/a5/24/76a5244a9686161c5188cba7c0bf4c99.jpg-600)

**项目运行跟调试**

项目创建后会自动创建DemoApplication.java文件，在DemoApplication 文件目录下新建文件夹 Controller，新建文件HomeController.java。效果如下：

![](https://ss.html.cn/article/4b/0f/14/4b0f148b8c626e3166cea60d2e9c1b5f.jpg-600)

Ps:SpringBoot项目的Bean装配默认规则是根据DemoApplication类所在的包位置从上往下扫描。所以必须放在同一目录下否则会无法访问报如下所示错误：

![](https://ss.html.cn/article/f1/4b/6c/f14b6c256da6b888dd7ff2f876c28040.jpg-600)

启动工程之前还需要配置下运行环境，如下图，点左边的小虫子图标，然后点上面的下拉箭头，选择添加配置，第一次设置时VS Code会提示选择需要运行的语言环境，选择对应环境后自动创建 launch.json 文件。

![](https://ss.html.cn/article/ae/b7/da/aeb7da2cec27285f690be46e28ef3f6a.jpg-600)

launch.json 调试配置文件如下，默认不修改配置也可使用：

![](https://ss.html.cn/article/4d/28/cd/4d28cdd67763b45e6664ca758ab1d952.jpg-600)

选择对应的配置环境调式项目如下，默认端口为8080。

![](https://ss.html.cn/article/98/6a/c4/986ac4c1b72150ee44572fe333818cf0.jpg-600)

启动后可在控制台输出面板查看启动信息，显示如下后，访问：http://localhost:8080即可。

![](https://ss.html.cn/article/58/e4/1e/58e41e766653f05f0b717349f4c637c3.jpg-600)

最终效果如下：

![](https://ss.html.cn/article/29/6a/10/296a107e404f3b640f2c52bd44277a72.jpg-600)

**访问HTML页面**

在spring boot 中访问html需要引入Thymeleaf（模板引擎）包，在创建项目时已引用该包这里不需在重复引用。在resources-->templates目录下创建Index.html文件，效果如下：

![](https://ss.html.cn/article/e4/cc/08/e4cc08a562fab72b6503e8f2d311dcb6.jpg-600)

html内容：

```
   第一个HTML页面 Hello Spring Boot!!!
```

在controller目录下新建TestController.java文件，代码如下：

```
 @Controller public class TestController { /** * 本地访问内容地址 ：http://localhost:8080/hello * @param map * @return */ @RequestMapping("/hello") public String helloHtml(HashMap map) { map.put("hello", "欢迎进入HTML页面"); return "/index"; } }
```

Ps:如果要访问html页面注解必须为Controller不能为RestController。否则无法访问。

**RestController和Controller的区别：**

@RestController is a stereotype annotation that combines @ResponseBody and @Controller.  
意思是：  
@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。  
1)如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。

例如：本来应该到success.html页面的，则其显示success.

2)如果需要返回到指定页面，则需要用 @Controller配合视图解析器InternalResourceViewResolver才行。

3)如果需要返回json或者xml或者自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBody注解

效果展示如下：

![](https://ss.html.cn/article/b5/97/17/b597170c974086620cb8486d55ba13be.jpg-600)

到处基础配置结束，可以愉快的玩耍Spring Boot!