![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

卸载gnome-shell主程序

```
$ sudo apt-get remove gnome-shell
```

卸载gnome

```
$ sudo apt-get remove gnome
```

卸载不需要的[依赖关系](https://so.csdn.net/so/search?q=%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB&spm=1001.2101.3001.7020)

```
$ sudo apt-get autoremove
```

彻底卸载删除gnome的相关配置文件

```
$ sudo apt-get purge gnome
```

清理安装gnome时候留下的缓存程序软件包

```
$ sudo apt-get autoclean
$ sudo apt-get clean
```