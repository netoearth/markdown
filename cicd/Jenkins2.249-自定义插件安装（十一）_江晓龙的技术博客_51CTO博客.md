自定义安装Jenkins2.249

由于之前一直使用的Jenkins2.176版本在做sonarqube集成的时候总是有问题，因此直接使用最新版本的2.249来集成

## 1.安装Jenkins

```
1)安装[root@rancher ~]# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repoJenkins公钥文件[root@rancher ~]# rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key[root@rancher ~]# yum install java-1.8.0-openjdk -y [root@rancher ~]# yum install jenkins -y这里安装的版本是2.2492）配置加速源[root@rancher ~]# vim /var/lib/jenkins/hudson.model.UpdateCenter.xml <?xml version='1.1' encoding='UTF-8'?><sites>  <site>    <id>default</id>    <url>https://updates.jenkins.io/update-center.json</url>  </site></sites>常用的加速源原插件源地址：https://updates.jenkins.io/update-center.json改为下列三个url地址，任一一个即可1 http://mirror.xmission.com/jenkins/updates/update-center.json   # 推荐2 http://mirrors.shu.edu.cn/jenkins/updates/current/update-center.json3 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json3）启动Jenkins[root@gitlab ~]# systemctl start jenkins1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.
```

## 2.页面安装Jenkins

### 2.1.解锁Jenkins

![Jenkins2.249-自定义插件安装（十一）_xml](https://s2.51cto.com/images/blog/202111/18144802_6195f722b849828714.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

![Jenkins2.249-自定义插件安装（十一）_用户名_02](https://s2.51cto.com/images/blog/202111/18144802_6195f722ef4f294127.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.2.选择自定义安装

选择自定义安装

![Jenkins2.249-自定义插件安装（十一）_json_03](https://s2.51cto.com/images/blog/202111/18144803_6195f723489cd15463.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

将所有的插件都选上

![Jenkins2.249-自定义插件安装（十一）_用户名_04](https://s2.51cto.com/images/blog/202111/18144803_6195f72377a5916510.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

安装完即可

![Jenkins2.249-自定义插件安装（十一）_xml_05](https://s2.51cto.com/images/blog/202111/18144803_6195f723af25327921.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.3.安装完成后设置用户名密码

![Jenkins2.249-自定义插件安装（十一）_用户名_06](https://s2.51cto.com/images/blog/202111/18144804_6195f72400ff071823.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.4.实例配置

![Jenkins2.249-自定义插件安装（十一）_用户名_07](https://s2.51cto.com/images/blog/202111/18144804_6195f724e98d639439.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.5.开始使用Jenkins

![Jenkins2.249-自定义插件安装（十一）_自定义_08](https://s2.51cto.com/images/blog/202111/18144805_6195f72518d913063.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.6.登录Jenkins

![Jenkins2.249-自定义插件安装（十一）_git_09](https://s2.51cto.com/images/blog/202111/18144805_6195f72571f7233772.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.7.将警告信息去掉

**配置警告信息**

![Jenkins2.249-自定义插件安装（十一）_用户名_10](https://s2.51cto.com/images/blog/202111/18144805_6195f725bfee572740.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

将对勾取消

![Jenkins2.249-自定义插件安装（十一）_xml_11](https://s2.51cto.com/images/blog/202111/18144806_6195f7260158846877.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

**干干净净**

![Jenkins2.249-自定义插件安装（十一）_git_12](https://s2.51cto.com/images/blog/202111/18144806_6195f726929f976052.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk= "在这里插入图片描述")

### 2.8.将所有插件打包备份

```
[root@gitlab jenkins]# tar cfzP jenkins-plugins-2-249.tar.gz plugins/下次使用直接解包放在这里即可1.2.
```

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