![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[夜拾柒](https://blog.csdn.net/yeweiyang16) ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCurrentTime2.png) 于 2017-05-03 11:25:49 发布 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes2.png) 5001 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect2.png) 收藏 3

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

![](https://img-blog.csdn.net/20170503112426863)  

首先，我们看到前面有VC14，VC9意思就是该版本PHP是用VisualStudio2008编译的，而VC11则是用VisualStudio2012编译的，后面的依次类推。  
如果我们下载的是VC11版本，就需要安装VisualC++RedistributableforVisualStudio2012.  
但是我们不需要专门安装这些运行库，因为电脑每次自动更新好最新运行库。  
而对于Non Thread Safe和Thread safe版本的选择，  

如果我们下载的是VC11版本，就需要安装VisualC++RedistributableforVisualStudio2012.  
但是我们不需要专门安装这些运行库，因为电脑每次自动更新好最新运行库。  
而对于Non Thread Safe和Thread safe版本的选择，  

PHP有2中运行方式：ISAPI和FastCGI。

ISAPI执行方式是以DLL动态库的形式使用，可以在被用户请求后执行，在处理完一个用户请求后不会马上消失，所以需要进行线程安全检查，这样来提高程序的执行效率，所以如果是以ISAPI来执行PHP，建议选择Thread Safe版本；而FastCGI执行方式是以单一线程来执行操作，所以不需要进行线程的安全检查，除去线程安全检查的防护反而可以提高执行效率，所以，如果是以FastCGI来执行PHP，建议选择Non Thread Safe版本。

对于apache服务器来说一般选择isapi方式，而对于nginx服务器则选择FastCGI方式。

所以我们这里选择Thread Safe版本