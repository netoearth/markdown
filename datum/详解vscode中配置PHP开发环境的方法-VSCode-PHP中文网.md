本篇文章给大家详细介绍一下如何在[vscode](https://www.php.cn/tool/vscode/)配置PHP开发环境。有一定的参考价值，有需要的朋友可以参考一下，希望对大家有所帮助。

![](https://img.php.cn/upload/article/000/000/024/606c3ffb8a7ab407.jpg)

**一、下载XAMPP**

___

XAMPP是一个易于安装的Apache发行版，其中包含MariaDB、PHP和Perl。仅仅需要下载并启动安装程序。  

**XAMPP下载地址**

官网下载：https://www.apachefriends.org/zh\_cn/download.html  
（可能需要科学上网，笔者没办法下，链接放这）

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/d21e43fdf7ad207d814b4664307447d3-0.png)

其他地址下载：http://xiazai.zol.com.cn/detail/38/372445.shtml  
（建议下载这个，选择本地下载-电信通道或者联通通道都可以）

**下载完后，PHP版本号是下面这个，后面需要用到**  
![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/e4b75cebbbbd89aeb5b11d0aabae1d9d-1.png)

**安装XAMPP**

一路NEXT，安装地址最好不要选C盘，笔者安装的是英语版。

**安装成功**

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/e4b75cebbbbd89aeb5b11d0aabae1d9d-2.png)  
根据需求开启，笔者写PHP的话选择开启Apache。

**添加系统变量**

把PHP.exe所在文件夹路径（笔者的是“**D:\\XAMPP\\php**”）添加进**环境变量-系统变量-Path**中（直接搜索框搜索系统变量便可找到）。

在cmd中输入php -v，检查是否配置成功

**配置成功**

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/d9805bb7a5d00efdfee654d8d8c86851-3.png)

**二、 下载xdebug插件**

___

下载地址：https://xdebug.org/download

**下载什么PHP版本，可以在XAMPP中的README看到**  

（笔者的是PHP 7.4.0，而且是Thread safe版本，对应的是带TS的版本，下载下来对应的不带nts的版本，文件名：php\_xdebug-2.9.7-7.4-vc15-x86\_64.dll）

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/d9805bb7a5d00efdfee654d8d8c86851-4.png)  

**添加配置**

把下载的x-debug文件（php\_xdebug-2.9.7-7.4-vc15-x86\_64.dll）复制到php\\ext文件夹下  
用记事本修改php.ini文件，在文件末尾添加几行配置信息，然后保存。

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p></td><td><div><p><code>[xdebug]</code></p><p><code>zend_extension=</code><code>"D:/xampp/php/ext/php_xdebug-2.9.7-7.4-vc15-x86_64"</code></p><p><code>xdebug.remote_enable = 1</code></p><p><code>xdebug.remote_autostart = 1</code></p></div></td></tr></tbody></table>

**三、下载并安装VSCode**

___

下载地址：https://code.visualstudio.com/  

**在VSCode中安装调试插件**

1、点击扩展栏，输入PHP，选择PHP Debug安装。

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/c6999b80ef1e766a23333c417b61dfed-5.png)

推荐学习：《[vscode教程](https://www.php.cn/tool/vscode/)》

2、点击VSCode的 文件-首选项-设置（不同版本可能显示不同，注意查找用户设置），在设置里面的扩展找到php，点击setting.json添加以下一行配置：

<table><tbody><tr><td><p>1</p><p>2</p></td><td><div><p><code>"php.validate.executablePath"</code><code>: </code><code>"D:/xampp/php/php.exe"</code><code>,</code></p></div></td></tr></tbody></table>

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/c6999b80ef1e766a23333c417b61dfed-6.png)

3、配置Debug

跳出的launch.json默认即可，不需要改动。

4、然后简单调试下，验证配置是否成功。

**注意，一定要以打开文件夹的形式才能成功设置断点调试，单个文件无效。可以选择D：/xampp/php/www**

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p></td><td><div><p><code>&lt;?php</code></p><p><code>$a</code> <code>= </code><code>'hello world'</code><code>;</code></p><p><code>echo</code> <code>$a</code><code>;</code></p><p><code>?&gt;</code></p></div></td></tr></tbody></table>

**设置断点，然后启动调试。**

5、在浏览器中打开要调试的php（不是文件路径而是服务器的地址(http://localhost:3000/hello.php)）,VSCode就会命中到打断点的地方。

6、最后推荐安装这个插件：PHP Server

可以选择右键 PHP Server：Serve project，直接跳转到浏览器

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/45c492e42c7e442b8f908b45105a475f-7.png)

7、运行结果

![在这里插入图片描述](https://img.php.cn/upload/article/000/000/024/45c492e42c7e442b8f908b45105a475f-8.png)

更多编程相关知识，请访问：[编程视频](https://www.php.cn/course.html)！！

以上就是详解vscode中配置PHP开发环境的方法的详细内容，更多请关注php中文网其它相关文章！