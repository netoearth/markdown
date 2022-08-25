## 1.老项目改造实现pipline项目自动上线

## 1.1.通过jenkins获取流水线语法

### 1.1.1.获取从gitlab上拉取项目的语法

拉取gitlab上的代码可以通过jenkins获取流水线语法最后粘贴到脚本中

点击配置—高级项目选项—流水线—流水线语法

![Jenkins集成pipline流水线实现自动化上线（十五）_linux](https://s2.51cto.com/images/blog/202111/18144815_6195f72fd498e64903.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

配置拉取代码的信息

![Jenkins集成pipline流水线实现自动化上线（十五）_服务端_02](https://s2.51cto.com/images/blog/202111/18144816_6195f7301cfbe15355.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**点击生成流水线脚本**

![Jenkins集成pipline流水线实现自动化上线（十五）_git_03](https://s2.51cto.com/images/blog/202111/18144816_6195f730623b152044.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 1.1.2.获取钉钉报警语法

![Jenkins集成pipline流水线实现自动化上线（十五）_3d_04](https://s2.51cto.com/images/blog/202111/18144816_6195f730a46a857628.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 1.2.使用git parameters参数构建

**参数化构建使用blue ocean会报错，会提示不支持，但是不影响构建**

![Jenkins集成pipline流水线实现自动化上线（十五）_git_05](https://s2.51cto.com/images/blog/202111/18144816_6195f730e47fc13835.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 2.编写pipeline脚本

**添加一个Git parameters参数化构建**

![Jenkins集成pipline流水线实现自动化上线（十五）_linux_06](https://s2.51cto.com/images/blog/202111/18144817_6195f731149cd62077.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 3.将代码填写到项目中

![Jenkins集成pipline流水线实现自动化上线（十五）_linux_07](https://s2.51cto.com/images/blog/202111/18144817_6195f7316969b87855.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4.构建选择标签即可

![Jenkins集成pipline流水线实现自动化上线（十五）_3d_08](https://s2.51cto.com/images/blog/202111/18144817_6195f7319fdf082296.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**每一步都可以看到日志**

![Jenkins集成pipline流水线实现自动化上线（十五）_git_09](https://s2.51cto.com/images/blog/202111/18144817_6195f731d3b4d21211.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 5.使用pipeline语法中的parameters语法构建

利用pipeline语法中的parameters语法构建的话，需要将之前添加的git parameters参数删除，并且使用blue ocean去构建

### 5.1.脚本编写

### 5.2.将代码粘贴到项目中

![Jenkins集成pipline流水线实现自动化上线（十五）_5e_10](https://s2.51cto.com/images/blog/202111/18144818_6195f732011c396414.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.3.利用blue ocean去构建

普通构建也可以的，建议使用blue ocean去构建

![Jenkins集成pipline流水线实现自动化上线（十五）_3d_11](https://s2.51cto.com/images/blog/202111/18144818_6195f7322976135297.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**打开blue ocean—点击运行—填写版本即可**

![Jenkins集成pipline流水线实现自动化上线（十五）_5e_12](https://s2.51cto.com/images/blog/202111/18144818_6195f732658b529592.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**运行中，每一个阶段都看得清清楚楚，还会有日志输出**

![Jenkins集成pipline流水线实现自动化上线（十五）_3d_13](https://s2.51cto.com/images/blog/202111/18144818_6195f7328f55649682.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**部署成功**  
![Jenkins集成pipline流水线实现自动化上线（十五）_服务端_14](https://s2.51cto.com/images/blog/202111/18144818_6195f732bcfc892873.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.4.项目构建后显示sonarqube图标

我们在构建过程中使用到了sonarqube，但是我们也希望能看到sonarqube的图标并且点击图标即可跳转至sonarqube

![Jenkins集成pipline流水线实现自动化上线（十五）_服务端_15](https://s2.51cto.com/images/blog/202111/18144819_6195f733018d962533.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**完整脚本**

**构建成功后查看是有sonarqube图标**  
![Jenkins集成pipline流水线实现自动化上线（十五）_3d_16](https://s2.51cto.com/images/blog/202111/18144819_6195f73336ea416242.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 5.5.支持钉钉短信报警

![Jenkins集成pipline流水线实现自动化上线（十五）_服务端_17](https://s2.51cto.com/images/blog/202111/18144819_6195f733a4db144108.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**完整脚本**

**将pipeline代码写入到项目中**

![Jenkins集成pipline流水线实现自动化上线（十五）_5e_18](https://s2.51cto.com/images/blog/202111/18144819_6195f733edb8e58269.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**钉钉报警**  
![Jenkins集成pipline流水线实现自动化上线（十五）_3d_19](https://s2.51cto.com/images/blog/202111/18144820_6195f73444c6192995.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

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