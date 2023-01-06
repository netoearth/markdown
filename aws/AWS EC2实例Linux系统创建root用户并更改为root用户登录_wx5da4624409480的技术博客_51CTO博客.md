

对于刚创建AWS EC2实例，或者经常使用AWS 实例的小伙伴们来说，刚创建的EC2实例是没有ROOT权限的，因此不能直接使用root用户去登陆实例，也无法获取到root权限。一般情况下，EC2实例默认是以ec2-user为用户名去登陆的（除了Ubuntu系统实例，它的默认用户名是ubuntu）。对于如何去创建root及密码，以及更改用户登陆方式--改为root用户登陆实例，就显得很有必要。下文就是帮助大家如何去创建root密码，以及如何ROOT用户去登陆实例。

1、创建root及设置密码：sudo passwd root

这里设置密码为：tesunet@2020

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_用户名](https://s2.51cto.com/images/blog/202201/13000509_61defc35dffc086087.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

2、创建完成后，切换到root用户下：su root，然后输入刚刚设置的密码

3、编辑EC2实例的SSH登录方式，编辑SSH文件：vi /etc/ssh/sshd\_config，修改一下三处内容

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root权限_02](https://s2.51cto.com/images/blog/202201/13000509_61defc35f17fb48732.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root用户_03](https://s2.51cto.com/images/blog/202201/13000510_61defc3618c4060573.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root权限_04](https://s2.51cto.com/images/blog/202201/13000510_61defc3631cd757855.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

修改完成之后保存。

4、再编辑authorized\_keys文件，将ssh-rsa 前面的文字全部删除，确保ssh-rsa没有任何文字，包括空格。

输入命令：vi /root/.ssh/authorized\_keys`   `

修改之前：

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root用户_05](https://s2.51cto.com/images/blog/202201/13000510_61defc364682554567.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

修改之后

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root用户_06](https://s2.51cto.com/images/blog/202201/13000510_61defc3676fdf8413.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

保存。

5、在完成以上设置之后，重启实例，重新打开SecureCRT直接使用root登陆。

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root权限_07](https://s2.51cto.com/images/blog/202201/13000510_61defc3695eb037508.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root用户_08](https://s2.51cto.com/images/blog/202201/13000510_61defc369e90321473.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

![AWS EC2实例Linux系统创建root用户并更改为root用户登录_root权限_09](https://s2.51cto.com/images/blog/202201/13000510_61defc36ad99988627.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

修改完成。

