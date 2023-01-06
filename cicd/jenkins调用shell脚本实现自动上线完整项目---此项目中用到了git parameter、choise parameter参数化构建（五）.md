利用jenkins自动部署完整项目

## 1.环境规划

| 主机 | ip |
| --- | --- |
| Jenkins | 192.168.81.220 |
| gitlab | 192.168.81.210 |
| lb | 192.168.81.230 |
| web01 | 192.168.81.240 |
| web02 | 192.168.81.250 |

## 2.gitlab新建monitor项目

### 2.1.配置Jenkins服务器公钥

将公钥文件添加到Gitlab中

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim](https://s2.51cto.com/images/blog/202111/18144804_6195f72480d2653172.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 2.2.创建monitor项目

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_02](https://s2.51cto.com/images/blog/202111/18144804_6195f72498e7a37902.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 2.3.将monitor的代码提交至远程项目仓库

代码已上传

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_03](https://s2.51cto.com/images/blog/202111/18144804_6195f724ccbda2847.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 3.jenkins拉取gitlab上的代码到本地

### 3.1.新建项目

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_04](https://s2.51cto.com/images/blog/202111/18144805_6195f7252430e19405.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

源码管理关联私钥

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_05](https://s2.51cto.com/images/blog/202111/18144805_6195f7255ffa935677.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.2.立即构建查看项目代码

立即构建

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_06](https://s2.51cto.com/images/blog/202111/18144805_6195f725891b734942.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

查看构建日志

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_07](https://s2.51cto.com/images/blog/202111/18144805_6195f725b93d377531.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

代码已获取

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_08](https://s2.51cto.com/images/blog/202111/18144806_6195f726349e462436.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4.配置nginx web集群

### 4.1.负载均衡配置

### 4.2.web节点配置

## 5.手动实现项目部署

### 5.1.拉取代码

点击立即构建拉取代码

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_09](https://s2.51cto.com/images/blog/202111/18144806_6195f7269bf7834345.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.2.将代码上传至各个节点服务器

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_10](https://s2.51cto.com/images/blog/202111/18144807_6195f7270e8dc95798.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 6.利用Jenkins实现自动部署

### 6.1.实现思路

### 6.2.脚本内容

### 6.3.修改Jenkins用户为root

### 6.4.改造3中创建项目

增加执行shell命令，指向脚本名即可

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_11](https://s2.51cto.com/images/blog/202111/18144807_6195f7274216147342.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 6.5.立即构建项目

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_12](https://s2.51cto.com/images/blog/202111/18144807_6195f727997b776578.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 6.6.查看控制台输出\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_13](https://s2.51cto.com/images/blog/202111/18144807_6195f727d1d3a29040.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 6.7.查看页面是否能访问以及节点软链接

链接的都是最新的表示成功  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_14](https://s2.51cto.com/images/blog/202111/18144808_6195f7280a50282161.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 7.Jenkins实现tag标签自动构建

### 7.1.改造项目支持tag标签

**配置项目----参数化构建—填写信息**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_15](https://s2.51cto.com/images/blog/202111/18144808_6195f7286028d89878.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**点击保存后就能看到在构建时可以选择版本**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_16](https://s2.51cto.com/images/blog/202111/18144808_6195f728999ed38398.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**将git分支也改成变量**  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_17](https://s2.51cto.com/images/blog/202111/18144808_6195f728c27a521523.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 7.2.改造脚本

项目中指定新脚本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_18](https://s2.51cto.com/images/blog/202111/18144809_6195f7291d11467177.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 7.3.制造标签

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_19](https://s2.51cto.com/images/blog/202111/18144809_6195f7295f77a3412.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 7.4.根据标签去自动构建

#### 7.4.1.构建v.1.1

**点击自动构建—填写上标签—开始构建**  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_20](https://s2.51cto.com/images/blog/202111/18144809_6195f729865e938832.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**控制台输出**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_21](https://s2.51cto.com/images/blog/202111/18144809_6195f729eb67744804.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**构建成功**

**版本为1.1，服务器中的目录链接到1.1**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_22](https://s2.51cto.com/images/blog/202111/18144810_6195f72a5426294673.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7.4.2.构建v1.2

**点击自动构建—填写上标签—开始构建**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_23](https://s2.51cto.com/images/blog/202111/18144810_6195f72ab0aec70460.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**控制台输出**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_24](https://s2.51cto.com/images/blog/202111/18144810_6195f72aecf3657313.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**构建成功**

**版本为1.2，服务器中的目录链接到1.2**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_25](https://s2.51cto.com/images/blog/202111/18144811_6195f72b38b1a9018.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7.4.3.构建v1.3

**点击自动构建—填写上标签—开始构建**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_26](https://s2.51cto.com/images/blog/202111/18144811_6195f72b7160e47556.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**控制台输出**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_27](https://s2.51cto.com/images/blog/202111/18144811_6195f72bb96c987060.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**构建成功**

**版本为1.3，服务器中的目录链接到1.3**  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_28](https://s2.51cto.com/images/blog/202111/18144812_6195f72c17ff77421.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 7.5.列表选择标签构建

#### 7.4.1.安装git parameter插件

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_29](https://s2.51cto.com/images/blog/202111/18144812_6195f72c602d445812.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7.4.2.关闭之前的文本使用安装好的git parameter

general—添加参数—git parameter—填写信息  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_30](https://s2.51cto.com/images/blog/202111/18144812_6195f72c8ad7973640.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7.4.3.点击立即构建会看到有标签列表

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_31](https://s2.51cto.com/images/blog/202111/18144812_6195f72cab75b45811.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7.4.4.构建1.1版本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_32](https://s2.51cto.com/images/blog/202111/18144812_6195f72ce402964800.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**查看已经构建成功**  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_33](https://s2.51cto.com/images/blog/202111/18144813_6195f72d3fa6318303.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 8.jenkins实现tag标签自动回退

### 8.1.新增一个choise parameter选项

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_34](https://s2.51cto.com/images/blog/202111/18144813_6195f72d86e5f3927.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 8.2.改造脚本增加rollback函数

思路：我们希望每次回退是直接修改一下链接文件而不是在点一次构建，因此我们在写脚本的时候可以先用find找到对应的链接文件，然后再把之前的链接删除，重新挂一个软连接  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_35](https://s2.51cto.com/images/blog/202111/18144813_6195f72da6b1a18280.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 8.3.项目中指定新脚本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_36](https://s2.51cto.com/images/blog/202111/18144813_6195f72de48fa17298.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 8.4.构建一个1.3版本并回退到1.2版本

#### 8.4.1构建1.3版本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_37](https://s2.51cto.com/images/blog/202111/18144814_6195f72e33ec033236.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**构建成功**  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_38](https://s2.51cto.com/images/blog/202111/18144814_6195f72e6845e88896.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 8.4.2.回退到1.2版本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_vim_39](https://s2.51cto.com/images/blog/202111/18144814_6195f72e954eb94223.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

成功的挂到的1.2版本回退成功

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_40](https://s2.51cto.com/images/blog/202111/18144814_6195f72eea88480877.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 9.Jenkins解决tag重复构建

可以通过Jenkins的变量来解决重复构建问题

用到的Jenkins变量：

-   GIT\_COMMIT
    -   当前提交的git commit id号
-   GIT\_PREVIOUS\_SUCCESSFUL\_COMMIT
    -   已成功构建过的git commit id号

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_41](https://s2.51cto.com/images/blog/202111/18144815_6195f72f4a4e593991.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

每次成功构建后都会将git commit id号写到GIT\_PREVIOUS\_SUCCESSFUL\_COMMIT这个变量中，我们可以利用GIT\_COMMIT变量和GIT\_PREVIOUS\_SUCCESSFUL\_COMMIT这个变量进行比对，如果值相等表示已经部署过了，无需再进行部署，如果值不同则说明是第一次构建，则立即进行构建。

### 9.1.改造脚本

重点修改了最后的if判断，增加了commit变量进行比对再进行构建

修改脚本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_42](https://s2.51cto.com/images/blog/202111/18144815_6195f72f7b2e592933.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 9.2.构建一个已经构建成功的tag并查看效果

1.3版本我们已经部署过了，我们在查看查看效果，是否无需再构建  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_43](https://s2.51cto.com/images/blog/202111/18144815_6195f72fb65af63434.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

commit id号进行比对，已经存在了构建成功的commit id号，则已经成功防止了重复构建，应用也不会部署指定的版本，还会保持当前的版本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_44](https://s2.51cto.com/images/blog/202111/18144815_6195f72fd91ee12879.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 9.3.新增一个1.4tag版本

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_nginx_45](https://s2.51cto.com/images/blog/202111/18144816_6195f7302769098219.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 9.4.构建1.4版本

升级到1.4版本查看是否能成功

**立即构建**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_46](https://s2.51cto.com/images/blog/202111/18144816_6195f7304be0c95121.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**构建成功**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_服务器_47](https://s2.51cto.com/images/blog/202111/18144816_6195f7308422176866.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 9.5.回退到1.1版本

回退到1.1版本查看是否能成功

立即构建  
![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_html_48](https://s2.51cto.com/images/blog/202111/18144816_6195f730c64d652470.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

构建成功

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_49](https://s2.51cto.com/images/blog/202111/18144817_6195f7312073732804.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 10.美化git parameter标签列表

**1.点击高级**

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_50](https://s2.51cto.com/images/blog/202111/18144817_6195f7316813496129.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**2.选择排序方式**

none：不排序

ASC：升序

DES：降序

一般都选择降序，把最新的版本排在上面

![jenkins调用shell脚本实现自动上线完整项目---此项目中用到了git parameter、choise parameter参数化构建（五）_git_51](https://s2.51cto.com/images/blog/202111/18144817_6195f731992b625802.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

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