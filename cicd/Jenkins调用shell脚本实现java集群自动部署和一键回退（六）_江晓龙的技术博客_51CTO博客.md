Jenkins实现java集群自动部署

**架构图**

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java](https://s2.51cto.com/images/blog/202111/18144814_6195f72e424889278.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 1.集群准备

### 1.1.环境概述

| 服务 | ip |
| --- | --- |
| gitlab | 192.168.81.210 |
| Jenkins | 192.168.81.220 |
| nginx-lb | 192.168.81.230 |
| nginx+tomcat | 192.168.81.240 |
| nginx+tomcat | 192.168.81.250 |

### 1.2.在web集群部署tomcat+nginx反向代理

配置nginx反向代理主要是为了动静分离，动态资源tomcat处理，静态资源nginx处理

**web01配置**

**web02配置**

### 1.3.配置nginx负载均衡

### 1.4.验证负载均衡

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_02](https://s2.51cto.com/images/blog/202111/18144814_6195f72ea8eab97543.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 2.实现java项目手动上线

### 2.1.准备java代码

### 2.2.将java项目推送至gitlab服务器

#### 2.2.1新增一个helloword项目

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_03](https://s2.51cto.com/images/blog/202111/18144814_6195f72ed72de17806.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 2.2.2.将代码推送至giitlab

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_git_04](https://s2.51cto.com/images/blog/202111/18144814_6195f72ef1a0242247.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 2.3.手动拉取源代码

### 2.4.对源代码进行编译成war包

#### 2.4.1.编程成war包

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_maven_05](https://s2.51cto.com/images/blog/202111/18144815_6195f72f5862566600.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 2.4.2.maven打包报错解决

### 2.5.通过scp方式将war包推送至web集群

### 2.6.重启tomcat

### 2.7.访问应用

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_06](https://s2.51cto.com/images/blog/202111/18144815_6195f72f8ce9499758.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 3.Jenkins实现java项目自动编译

思路：

1.需要新建一个maven的项目并按照Maven intergration plugin插件

2.Jenkins抓取gitlab上的java代码

3.Jenkins调用maven进行编译构建

4.Jenkins调用shell进行推送

### 3.1.安装Maven Integration插件

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_git_07](https://s2.51cto.com/images/blog/202111/18144815_6195f72fa155492124.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_git_08](https://s2.51cto.com/images/blog/202111/18144815_6195f72fb266d79264.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.2.创建一个maven项目

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_09](https://s2.51cto.com/images/blog/202111/18144815_6195f72fd83d068763.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.3.配置项目中的git源码管理

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_10](https://s2.51cto.com/images/blog/202111/18144816_6195f7302466149945.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.4.配置maven参数

配置maven参数，使得Jenkins识别maven部署路径、jdk部署路径，以便以能够对java代码进行编译，最后打包成war包即可发布。

Build就是配置maven参数

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_11](https://s2.51cto.com/images/blog/202111/18144816_6195f730535175199.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 3.4.1.新增jdk安装路径

点击新增jdk----取消install automatically的对勾

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_12](https://s2.51cto.com/images/blog/202111/18144816_6195f7307571261387.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

填写别名和jdk路径

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_13](https://s2.51cto.com/images/blog/202111/18144816_6195f730c249a54060.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 3.4.2.新增maven安装路径

点击新增—取消install automatically的对勾

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_14](https://s2.51cto.com/images/blog/202111/18144817_6195f7310b74d35823.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

填写部署路径—点击保存

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_15](https://s2.51cto.com/images/blog/202111/18144817_6195f7314de7b52716.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 3.4.3.填写maven选项

配置完maven和jdk后再次刷新项目配置页面会发现不再有刚刚的提示信息

Goals and options这里填写的就是maven的目录选项，因此只填写package即可  
![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_16](https://s2.51cto.com/images/blog/202111/18144817_6195f731944cf95242.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.5.填写部署前操作

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_maven_17](https://s2.51cto.com/images/blog/202111/18144817_6195f731e795b73637.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.6.调用maven进行编译构建

先预先看一下效果，能否编译成功，如果能成功在配置脚本部署方面

部署前操作

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_git_18](https://s2.51cto.com/images/blog/202111/18144818_6195f7321566497938.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

编译成功，会看到war包的路径  
![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_19](https://s2.51cto.com/images/blog/202111/18144818_6195f7325851c53903.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4.改造maven 项目实现自动构建

### 4.1.增加参数化构建tag标签

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_20](https://s2.51cto.com/images/blog/202111/18144818_6195f732b621069871.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

源码管理—分支选择更改的${git\_version}

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_git_21](https://s2.51cto.com/images/blog/202111/18144819_6195f7330589730832.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 4.2.编写自动构建脚本

思路：首先把打好的war包通过scp的方式拷贝至web集群的/opt目录，并在/opt目录进行解压，解压后再将tomcat/webapps目录的所有文件删掉，再将/opt目录解压的目录链接到tomcat/webapps/ROOT，最后重启tomcat即可实现自动部署。

### 4.3.在项目中添加执行脚本

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_maven_22](https://s2.51cto.com/images/blog/202111/18144819_6195f7334574992561.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 4.4.在git上打标签并推送至gitlab

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_23](https://s2.51cto.com/images/blog/202111/18144819_6195f7339751248903.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 4.5.根据版本进行构建

**立即构建**

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_maven_24](https://s2.51cto.com/images/blog/202111/18144819_6195f733cb94e32964.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**控制台输出**

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_25](https://s2.51cto.com/images/blog/202111/18144820_6195f7341913284247.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

如果构建报错如下图，请将jdk删掉重新安装

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_26](https://s2.51cto.com/images/blog/202111/18144820_6195f7347b5c514629.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 4.6.访问页面查看效果

**自动部署成功**

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_maven_27](https://s2.51cto.com/images/blog/202111/18144820_6195f734d9bd937768.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 5.java项目自动回退

### 5.1.编写脚本

思路：增加一个rollback的函数，通过find目录查找到要回退的版本的目录，然后通过软连接挂到webapps下面即可，通过deplo\_env来判断是部署还是回退，本次增加了防止重复构建的功能

### 5.2.改造项目

1.**修改脚本路径**  
![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_28](https://s2.51cto.com/images/blog/202111/18144821_6195f735191ef31417.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**2.增加一个参数化构建choice parameters**

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_29](https://s2.51cto.com/images/blog/202111/18144821_6195f73546a8858791.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

扩展：项目中的框可以随意调至，比如把刚刚新增的choice参数拉倒上面

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_30](https://s2.51cto.com/images/blog/202111/18144821_6195f7358e12b84789.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

构建时就会看到choice参数在上面  
![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_31](https://s2.51cto.com/images/blog/202111/18144821_6195f735a2c8c11861.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.3.打一个1.2版本的标签并将项目升级到1.2版本

#### 5.3.1.打标签

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_32](https://s2.51cto.com/images/blog/202111/18144821_6195f735d499c2212.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 5.3.2.升级到1.2版本

开始构建1.2版本

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_nginx_33](https://s2.51cto.com/images/blog/202111/18144822_6195f73621fb561939.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

升级成功  
![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_tomcat_34](https://s2.51cto.com/images/blog/202111/18144822_6195f7365392135011.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.4.回退至1.1版本

一般来说还是把回退单独放新建一个项目好，省掉了git拉取代码的时间以及maven构建的时间

![Jenkins调用shell脚本实现java集群自动部署和一键回退（六）_java_35](https://s2.51cto.com/images/blog/202111/18144822_6195f736ac7b929995.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

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