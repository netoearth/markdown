**Win10下 Java环境变量配置**

首先，你应该已经安装了 Java 的 JDK 了（如果没有安装JDK，请跳转到此网址：http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html）

笔者安装的是 jdk-8u91-windows-x64

接下来主要讲怎么配置 Java 的环境变量，也是为了以后哪天自己忘记了做个备份

（注：win10的Java环境变量配置和其他的windows版本稍有不同）

**在电脑桌面 右键点击 “此电脑”的“属性”选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512222932249-1178980125.png)

**选择“高级系统设置”选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512223219655-1251440251.png)

**点击下面的“环境变量”选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512223357046-1881703129.png)

接下来就是具体的配置过程：

**点击“系统变量”下面的”新建“选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512223611077-1852208980.png)

**在”变量名“处填上”Java\_Home“**

**”变量值“为JDK安装路径，笔者的路径是”D:\\Program Files\\Java\\jdk1.8.0\_91“**

**点击”确定“选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512223838546-1858485630.png)

**在”系统变量“中找到”Path“**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512224107702-1187773252.png)

**选中”Path“点击”编辑“选项**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512224230452-538115428.png)

**选择右边的“编辑文本”，将引号里面的全部复制“%Java\_Home%\\bin;%Java\_Home%\\jre\\bin;”，到“变量值”栏的最前面，“确定”**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512224443687-1268631944.png)

**在“系统变量”栏，“新建”，“变量名”为“CLASSPATH”，“变量值”为“.;%Java\_Home%\\bin;%Java\_Home%\\lib\\dt.jar;%Java\_Home%\\lib\\tools.jar”，“确定”**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512224843484-1356351538.png)

点击“环境变量”最下面的“确定”选项

**回到电脑桌面，按快捷键“Win+R”，输入“cmd”**

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512225105562-1578800234.png)

检查Java环境是否配置成功

输入"java"

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512225343452-1007903408.png)

输入"javac"

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512225205765-304825305.png)

输入"java -version"

![](https://images2015.cnblogs.com/blog/805379/201605/805379-20160512225445859-652093728.png)

如果上面的三幅图都看见了，恭喜，环境变量配置好了！