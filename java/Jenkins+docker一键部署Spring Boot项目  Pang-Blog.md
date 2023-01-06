## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#%E4%B8%80%E3%80%81%E7%B3%BB%E7%BB%9F%E7%8E%AF%E5%A2%83 "一、系统环境")一、系统环境

以下环境皆为必要系统环境；以下环境皆为我测试机的环境，实际操作不一定要如此高的版本

-   Ubuntu 20.04LTS
-   Java 8
-   Maven 3.6.3
-   Docker 19.03.13
-   Jenkins 2.249.3
-   git 2.25.1

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#1%E3%80%81%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE "1、创建项目")1、创建项目

使用IDEA等创建一个简单的Spring Boot项目，注意下面标红的几个点

[![image-20201125105510326](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125105510326.png "image-20201125105510326")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125105510326.png))[![image-20201125105537719](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125105537719.png "image-20201125105537719")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125105537719.png)

然后创建一个类`Hello0Controller`，添加如下代码

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><span><span>@RestController</span></span><br><span><span>public</span> <span><span>class</span> <span>HelloController</span> </span>{</span><br><span></span><br><span>    <span>@RequestMapping</span>(<span>"/hello"</span>)</span><br><span>    <span><span>public</span> String <span>hello</span><span>(String str)</span></span>{</span><br><span>        <span>return</span> <span>"hello "</span>+str;</span><br><span>    }</span><br><span>}</span><br></pre></td></tr></tbody></table>

此时，运行这个项目，访问`http://localhost:port/hello?str=world`以得到一串字符串`hello world`。以此来测试项目是否创建成功，是否运行成功。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#2%E3%80%81%E6%89%93%E5%8C%85%E9%A1%B9%E7%9B%AE "2、打包项目")2、打包项目

在 项目根目录处运行指令`mvn package`，对这个项目进行打包，正常情况下，项目会生成target文件夹，文件夹下的`xxx-0.0.1-SNAPSHOT.jar`即为打包生成的可执行jar包（xxx为项目的`artifactId`）

为了测试打包的jar包是否正确，我们可以使用指令执行该jar包，然后进行访问查看效果。执行指令如下

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span><span>cd</span> targer/</span><br><span>java -jar xxx-0.0.1-SNAPSHOT.jar</span><br></pre></td></tr></tbody></table>

访问`http://localhost:port/hello?str=world`以得到一串字符串`hello world`。以此来测试项目是否创建成功，是否运行成功。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#3%E3%80%81%E5%B0%86%E8%AF%A5%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0GitHub "3、将该项目上传到GitHub")3、将该项目上传到GitHub

省略……

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Jenkins%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE "三、使用Jenkins自动部署Spring Boot项目")三、使用Jenkins自动部署Spring Boot项目

我们先不使用Docker，直接用Jenkins部署这个SpringBoot项目。

先来统计一下，我们用到的指令如下

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>git <span>clone</span> https://github.com/xxx.git</span><br><span><span>cd</span> xxx</span><br><span>mvn package</span><br><span><span>cd</span> target</span><br><span>java -jar xxx-0.0.1-SNAPSHOT.jar</span><br></pre></td></tr></tbody></table>

如果我们自己在服务器终端，直接运行以上几个指令即可在服务器运行起我们的SpringBoot项目。Jenkins其实也只是将这些指令集成一下，做到一键即可部署。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#1%E3%80%81%E5%AE%89%E8%A3%85%E3%80%81%E8%BF%90%E8%A1%8CJenkins "1、安装、运行Jenkins")1、安装、运行Jenkins

建议直接看文档进行安装：[Jenkins官网](https://www.jenkins.io/zh)

安装完后运行时安装插件选择推荐插件即可，遇到的大部分Jenkins问题都可自行搜索解决。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#2%E3%80%81%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83 "2、配置环境")2、配置环境

访问服务器运行的Jenkins主页，选择Manage Jenkins -> Global Tool Configuration，进入到全局环境变量配置，将需要配置的环境（Java、Maven、Git）填入即可。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#3%E3%80%81%E5%9C%A8Jenkins%E4%B8%8A%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE "3、在Jenkins上新建项目")3、在Jenkins上新建项目

访问服务器运行的Jenkins主页，选择“新建Item（New Item）”

然后输入该项目名称，并选择“流水线”后，点击确定新建项目

[![image-20201125113416654](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125113416654.png "image-20201125113416654")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125113416654.png)

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#4%E3%80%81%E7%BC%96%E5%86%99%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B5%81%E6%B0%B4%E7%BA%BF "4、编写自定义流水线")4、编写自定义流水线

点击确定按钮以后，跳转到配置页面，在配置页面，我们只用关心流水线脚本，其余配置若感兴趣可自行查阅其作用。

[![image-20201125114310541](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125114310541.png "image-20201125114310541")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125114310541.png)

对于pipeline语法可以浏览官网进行学习，本篇文章所使用的都是简单的指令。本篇文章的脚本如下（注意根据自己的项目进行修改）

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br></pre></td><td><pre><span>pipeline {</span><br><span>    agent any</span><br><span>    stages {</span><br><span>        stage(<span>'使用git下载代码'</span>) {</span><br><span>            steps {</span><br><span>                </span><br><span>                echo <span>'git from github...'</span></span><br><span>                </span><br><span>                git <span>url:</span> <span>'https://github.com/xxx.git'</span>, <span>branch:</span> <span>'main'</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'使用Maven构建'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"maven build start..."</span></span><br><span>                </span><br><span>                sh <span>"mvn -Dmaven.test.skip=true clean package"</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'运行项目'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"Spring boot start..."</span></span><br><span>                sh <span>"java -jar target/xxx-0.0.1-SNAPSHOT.jar"</span></span><br><span>            }</span><br><span>        }</span><br><span>    }</span><br><span>}</span><br></pre></td></tr></tbody></table>

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#5%E3%80%81%E4%B8%80%E9%94%AE%E6%9E%84%E5%BB%BA "5、一键构建")5、一键构建

进入到项目页面，点击按钮“Build Now”即可构建

[![image-20201125150210237](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125150210237.png "image-20201125150210237")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125150210237.png)

下面是构建完成的截图

[![image-20201125154325447](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125154325447.png "image-20201125154325447")](https://pangyuworld.github.io/assets/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/image-20201125154325447.png)

注意这里由于我们的指令问题，是不会构建完成，而是卡在运行项目这里，点击左下角`#12`左边的圆圈按钮即可查看输出信息

___

到此，Jenkins配合Spring Boot实现自动化部署已经完成，每次上传完代码以后直接点击BuildNow按钮即可重新部署。但是，目前运行项目这个步骤一直无法完成，这个问题有两个解决办法：

1.  使用`nohup java -jar xxx.jar &`指令，但是如果使用这个指令需要用单引号包住这个指令且jar文件需要是绝对路径，感兴趣可以尝试一下。
2.  使用Docker构建镜像并运行镜像

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#%E5%9B%9B%E3%80%81%E9%85%8D%E5%90%88Docker "四、配合Docker")四、配合Docker

为了解决上述问题，我们引入Docker。引入Docker不光是为了解决上面那个问题，容器化已经是大势所趋，具体使用Docker有哪些好处可以自行搜索。

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#1%E3%80%81Docker%E5%B0%86Spring-Boot%E9%A1%B9%E7%9B%AE%E6%89%93%E5%8C%85%E6%88%90%E9%95%9C%E5%83%8F "1、Docker将Spring Boot项目打包成镜像")1、Docker将Spring Boot项目打包成镜像

Docker对Spring Boot项目打包的方式有很多种，这里我参考官方推荐的一种，参考网址：[https://spring.io/guides/gs/spring-boot-docker/](https://spring.io/guides/gs/spring-boot-docker/)

### [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#%E6%96%B0%E5%BB%BADockerfile "新建Dockerfile")新建Dockerfile

在项目根目录新建一个名为Dockerfile的文件，注意没有后缀名，其内容如下

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span><span>FROM</span> java:<span>8</span></span><br><span><span>VOLUME</span><span> /tmp</span></span><br><span><span>COPY</span><span> target/xxx-0.0.1-SNAPSHOT.jar app.jar</span></span><br><span><span>ENTRYPOINT</span><span> [<span>"java"</span>,<span>"-jar"</span>,<span>"/app.jar"</span>]</span></span><br></pre></td></tr></tbody></table>

> Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
> 
> [Dockerfile指令|菜鸟教程](https://www.runoob.com/docker/docker-dockerfile.html)

### [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#%E5%9C%A8%E6%9C%AC%E5%9C%B0%E6%9E%84%E5%BB%BA%E9%95%9C%E5%83%8F%EF%BC%88%E6%B5%8B%E8%AF%95%E7%94%A8%EF%BC%8C%E5%8F%AF%E8%B7%B3%E8%BF%87%EF%BC%89 "在本地构建镜像（测试用，可跳过）")在本地构建镜像（测试用，可跳过）

如果你本地有Docker环境的话，可以在本地进行构建镜像。

在项目根目录运行指令`docker build -t spring-demo .`即可

运行该镜像的话，只用执行指令`docker run -d -p 8080:8080 spring-demo`即可，`8080:8080`指**本地端口8080(第一个)映射到容器端口8080(第二个)**

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#2%E3%80%81%E5%B0%86Dockerfile%E4%B8%8A%E4%BC%A0%E5%88%B0GitHub "2、将Dockerfile上传到GitHub")2、将Dockerfile上传到GitHub

省略….

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#3%E3%80%81%E4%BF%AE%E6%94%B9pipeline%E8%84%9A%E6%9C%AC "3、修改pipeline脚本")3、修改pipeline脚本

修改后的脚本如下

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br></pre></td><td><pre><span>pipeline {</span><br><span>    agent any</span><br><span>    stages {</span><br><span>        stage(<span>'停止旧容器'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"stop if project is running..."</span></span><br><span>                sh <span>'docker ps -f name=spring-demo -q | xargs --no-run-if-empty docker container stop'</span></span><br><span>                sh <span>'docker ps -a -f name=spring-demo -q | xargs --no-run-if-empty docker container rm'</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'使用git下载代码'</span>) {</span><br><span>            steps {</span><br><span>                </span><br><span>                echo <span>'git from github...'</span></span><br><span>                </span><br><span>                git <span>url:</span> <span>'https://github.com/xxx.git'</span>, <span>branch:</span> <span>'main'</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'使用Maven构建'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"maven build start..."</span></span><br><span>                </span><br><span>                sh <span>"mvn -Dmaven.test.skip=true clean package"</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'Docker构建'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"docker build start..."</span></span><br><span>                sh <span>"docker build -t spring-demo ."</span></span><br><span>            }</span><br><span>        }</span><br><span>        stage(<span>'Docker运行'</span>) {</span><br><span>            steps {</span><br><span>                echo <span>"docker image start..."</span></span><br><span>                sh <span>"docker run -d -p 8080:8080 --name spring-demo spring-demo"</span></span><br><span>            }</span><br><span>        }</span><br><span>    }</span><br><span>}</span><br></pre></td></tr></tbody></table>

## [](https://pangyuworld.github.io/2020/11/21/Jenkins-docker%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2Spring-Boot%E9%A1%B9%E7%9B%AE/#5%E3%80%81%E4%B8%80%E9%94%AE%E6%9E%84%E5%BB%BA-1 "5、一键构建")5、一键构建

先将没有集成Docker的那个构建停止，然后重新点击BuildNow即可

___

以上，Jenkins+Docker+SpringBoot的自动部署就实现完成了，我的项目代码地址：[https://github.com/pangyuworld/spring-demo](https://github.com/pangyuworld/spring-demo) 仅供参考

___