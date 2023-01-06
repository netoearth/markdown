在ESXI上安装了一个Ubuntu16.04,空闲时CPU占用1.2G左右的资源，使用top查看进程，发现是 compiz 这个进程在占用cpu  
网上搜索了一下，发现Compiz是用OpenGL来增强界面显示效果的，虚拟机中不需要这个耗费CPU资源的特性，要将他禁用掉。

需要先安装另外一个桌面管理器来替代

```
sudo apt-get install gnome-session-flashback

```

安装完后，注销账号重新登录，在登录输入用户名的右侧有一个圆按钮，点这个按钮选择 gnome flashback (Metacity), 输入密码登录系统，出来一个省CPU的简洁界面。  
再次查看空闲时ESXI对这个虚拟机监察到的资源占用情况，只有10MHZ不到的CPU资源消耗，解决问题

