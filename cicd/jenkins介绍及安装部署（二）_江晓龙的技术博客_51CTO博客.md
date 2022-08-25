### jenkins介绍及安装部署（二）

©著作权归作者所有：来自51CTO博客作者江晓龙的技术博客的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[jenkins介绍及安装部署（二）](https://blog.51cto.com/jiangxl/4637881)  
[https://blog.51cto.com/jiangxl/4637881](https://blog.51cto.com/jiangxl/4637881)

jenkins介绍及安装部署

## 1.介绍jenkins

jenkins是一个开源的，提供友好操作界面的持续集成ci工具，起源于hudson，主要用于持续、自动的构建测试软件项目，监控外部任务的运行，jenkins用java语言编写，可在tomcat等流行的servlet容器中运行，也可独立运行，通常与版本管理工具（SCM）、构建工具结合使用，常用的版本控制工具有SVN、GIT，构建工具有maven、ant、gradle

**CI/CD介绍：**

-   CI是一种软件开发时间，持续集成强调开发人员提交了新代码之后，立刻进行构建、测试，根据测试结果，我们可以确定新代码和原有代码能否正确的集成在一起。
-   CD是在持续集成的的基础上，将集成后的代码部署到更贴近真实运行环境中，比如，我们完成单元测试后，可以把代码部署到连接数据库的环境中更多的测试，如果代码没问题，可以继续手动部署到生产环境

## 2.安装jenkins

```
1）安装jdk1.8[root@jenkins ~]# yum -y install java[root@jenkins ~]# java -versionopenjdk version "1.8.0_262"OpenJDK Runtime Environment (build 1.8.0_262-b10)OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)2）安装jenkins[root@jenkins ~]# yum -y localinstall jenkins-2.176.1-1.1.noarch.rpm 3）启动jenkins[root@jenkins ~]# systemctl start jenkins[root@jenkins ~]# systemctl enable jenkins1.2.3.4.5.6.7.8.9.10.11.12.13.
```

```
浏览器访问http://192.168.81.220:8080耐性等待jenkins打开页面1.2.
```

![jenkins介绍及安装部署（二）_镜像源](https://s2.51cto.com/images/blog/202111/18144825_6195f7396ae6453513.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "20200823210936317")

## 3.页面安装jenkins

### 3.1.解锁jenkins

出来页面后把密码填进去

```
[root@jenkins ~]# cat /var/lib/jenkins/secrets/initialAdminPassworddcc793bb6caa4d8c9a5824de822645a91.2.
```

![jenkins介绍及安装部署（二）_java_02](https://s2.51cto.com/images/blog/202111/18144825_6195f739b5d0a78457.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.2.选择推荐插件

![jenkins介绍及安装部署（二）_构建工具_03](https://s2.51cto.com/images/blog/202111/18144826_6195f73a0592a53604.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.3.安装插件

![jenkins介绍及安装部署（二）_持续集成_04](https://s2.51cto.com/images/blog/202111/18144826_6195f73a4aa9c68909.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.4.创建管理员账号

![jenkins介绍及安装部署（二）_批量导入_05](https://s2.51cto.com/images/blog/202111/18144826_6195f73a8921244666.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.5.实例配置

![jenkins介绍及安装部署（二）_批量导入_06](https://s2.51cto.com/images/blog/202111/18144826_6195f73ac90a191315.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.6.安装完成jenkins

![jenkins介绍及安装部署（二）_批量导入_07](https://s2.51cto.com/images/blog/202111/18144827_6195f73b4558076809.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 3.7.登录jenkins

![jenkins介绍及安装部署（二）_镜像源_08](https://s2.51cto.com/images/blog/202111/18144827_6195f73b85ed879730.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

## 4.jenkins插件

### 4.1.对插件进行加速

对插件进行加速就是替换源这里使用清华源

#### 4.1.1.获取镜像源

```
镜像源地址https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/dynamic-2.176/update-center.json1.2.
```

![jenkins介绍及安装部署（二）_镜像源_09](https://s2.51cto.com/images/blog/202111/18144827_6195f73ba1e3e21269.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

#### 4.1.2.jenkins配置镜像源

jenkins—>Manage Jenkins—>Manage Plugins---->高级

![jenkins介绍及安装部署（二）_构建工具_10](https://s2.51cto.com/images/blog/202111/18144827_6195f73bcad1f21563.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 4.2.jenkins安装一个插件

jenkins—>Manage Jenkins—>Manage Plugins---->可选插件

![jenkins介绍及安装部署（二）_java_11](https://s2.51cto.com/images/blog/202111/18144828_6195f73c04ddd80947.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 4.3.导入一个.hpi的插件

#### 4.3.1.在清华园下载一个hpi插件

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/随便下载一个插件1.2.
```

![jenkins介绍及安装部署（二）_持续集成_12](https://s2.51cto.com/images/blog/202111/18144828_6195f73c2adcf14874.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

#### 4.3.2.导入hpi插件

jenkins—>Manage Jenkins—>Manage Plugins---->高级

![jenkins介绍及安装部署（二）_java_13](https://s2.51cto.com/images/blog/202111/18144828_6195f73c4d79d16223.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

导入完成

![jenkins介绍及安装部署（二）_持续集成_14](https://s2.51cto.com/images/blog/202111/18144828_6195f73c98a9e90735.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 4.4.备份插件及批量导入插件

#### 4.4.1.备份所有插件

```
[root@jenkins ~]# cd /var/lib/jenkins/[root@jenkins jenkins]# tar cfzP jenkins_2.176_plugins.tar.gz plugins/1.2.
```

#### 4.4.2.批量导入所有插件

```
[root@jenkins ~]# cd /var/lib/jenkins/[root@jenkins jenkins]# rm -rf plugins[root@jenkins jenkins]# tar xf jenkins_2.176_plugins.tar.gz重载jenkins[root@jenkins jenkins]# systemctl restart jenkins1.2.3.4.5.6.
```

## 5.取消jenkins插件更新提示

将下图这些提示全部取消掉，让jenkins看起来干净一点

![jenkins介绍及安装部署（二）_持续集成_15](https://s2.51cto.com/images/blog/202111/18144828_6195f73cc56ce74626.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

具体配置：

点击图中的配置显示那些告警----隐藏的安全警告----将对勾都取消掉

![jenkins介绍及安装部署（二）_构建工具_16](https://s2.51cto.com/images/blog/202111/18144828_6195f73ce60f360806.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

-   **赞**
-   **收藏**
-   **评论**
-   **举报**

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持JPEG/PNG/JPG，图片不超过1.9M

已经收到您得举报信息，我们会尽快审核

## 相关文章