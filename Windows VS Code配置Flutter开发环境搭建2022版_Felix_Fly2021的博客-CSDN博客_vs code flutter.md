![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

于 2022-03-15 11:14:33 首次发布

1.下载 [VS Code](https://code.visualstudio.com/)  
2.打开VSCode 安装插件

-   Dart
-   Flutter
-   Awesome Flutter Snippets (非必要)
-   Flutter Widget Snippets (非必要)

3.下载[Flutter SDK](https://flutter.cn/docs/get-started/install/windows)（我现在用的是 [flutter\_windows\_2.10.3-stable.zip](https://storage.flutter-io.cn/flutter_infra_release/releases/stable/windows/flutter_windows_2.10.3-stable.zip)）,解压缩放在任意位置

4.配置环境变量

编辑**系统变量**

-   变量名：Path  
    变量值：你的[flutter](https://so.csdn.net/so/search?q=flutter&spm=1001.2101.3001.7020) sdk 文件夹路径\\bin
    
    _记住是你的flutter sdk路径+bin目录_
    

新建**系统变量**

-   变量名：FLUTTER\_STORAGE\_BASE\_URL  
    变量值：https://storage.flutter-io.cn
    
-   变量名：PUB\_HOSTED\_URL  
    变量值：https://pub.flutter-io.cn
    

好了之后可以打开VS Code **Shift+Ctrl+P**调出命令框

输入**Flutter:New Project** Enter执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/d59d01013e7c456bb872298771ac5f39.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmVsaXhfRmx5MjAyMQ==,size_20,color_FFFFFF,t_70,g_se,x_16)  
执行完选择 **Application**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/1561b502db954f6ea21187d045696046.png)  
选择项目保存位置

输入项目名字

Flutter项目就新建好啦

这里可以选择运行的设备和平台  
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e5e7ed883fd4b46b7ac9edf5b874462.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmVsaXhfRmx5MjAyMQ==,size_20,color_FFFFFF,t_70,g_se,x_16)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ae58ea80df740aba3eb3352ededae4a.png)

按右边三角符号就能运行了  
![在这里插入图片描述](https://img-blog.csdnimg.cn/3bb8140fc8b74243accea68eb6f41bee.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmVsaXhfRmx5MjAyMQ==,size_20,color_FFFFFF,t_70,g_se,x_16)  
我选择的是Chrome, 运行成功！！！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/68d9e26288194a4c9fb7ac5daa930f21.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmVsaXhfRmx5MjAyMQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

如需Windows Desktop编译教程请看我 [另外一篇文章](https://blog.csdn.net/qq_39457683/article/details/123494789)