©著作权归作者所有：来自51CTO博客作者江晓龙的技术博客的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[jenkis插件安装（三）](https://blog.51cto.com/jiangxl/4637865)  
[https://blog.51cto.com/jiangxl/4637865](https://blog.51cto.com/jiangxl/4637865)

## 1.jenkins插件

### 1.1.对插件进行加速

对插件进行加速就是替换源这里使用清华源

#### 1.1.1.获取镜像源

![jenkis插件安装（三）_镜像源](https://s2.51cto.com/images/blog/202111/18144825_6195f73974eb13043.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 1.1.2.jenkins配置镜像源

jenkins—>Manage Jenkins—>Manage Plugins---->高级  
![jenkis插件安装（三）_批量导入_02](https://s2.51cto.com/images/blog/202111/18144825_6195f739a89d695397.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 1.2.jenkins安装一个插件

jenkins—>Manage Jenkins—>Manage Plugins---->可选插件

![jenkis插件安装（三）_其他_03](https://s2.51cto.com/images/blog/202111/18144825_6195f739d632b34729.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 1.3.导入一个.hpi的插件

#### 1.3.1.在清华园下载一个hpi插件

![jenkis插件安装（三）_其他_04](https://s2.51cto.com/images/blog/202111/18144826_6195f73a0aa1773906.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 1.3.2.导入hpi插件

jenkins—>Manage Jenkins—>Manage Plugins---->高级

![jenkis插件安装（三）_json_05](https://s2.51cto.com/images/blog/202111/18144826_6195f73a359099375.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

导入完成  
![jenkis插件安装（三）_批量导入_06](https://s2.51cto.com/images/blog/202111/18144826_6195f73a7cd146722.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 1.4.备份插件及批量导入插件

#### 1.4.1.备份所有插件

#### 1.4.2.批量导入所有插件

## 2.取消jenkins插件更新提示

将下图这些提示全部取消掉，让jenkins看起来干净一点

![jenkis插件安装（三）_镜像源_07](https://s2.51cto.com/images/blog/202111/18144826_6195f73abfe0b32699.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

具体配置：

点击图中的配置显示那些告警----隐藏的安全警告----将对勾都取消掉

![jenkis插件安装（三）_批量导入_08](https://s2.51cto.com/images/blog/202111/18144827_6195f73b1773356972.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

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