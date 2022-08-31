![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

[Ubuntu](https://so.csdn.net/so/search?q=Ubuntu&spm=1001.2101.3001.7020)下安装NVIDIA驱动的三种方法：

1\. 使用标准Ubuntu仓库进行自动安装

2\. 使用PPA仓库进行自动化安装

3\. 使用官方的NVIDIA驱动进行手动安装

个人认为，第一种方法操作最为简单，方便，第三种方法是最稳定，最常用的。

很多博客进行驱动安装前都需要禁用Nouveau驱动，但是我安装驱动的时候搞了很久，最后安装好驱动就是根据第一种方式并且没禁用Nouveau驱动的情况下，我也找到了开始一直安装不成功的原因，就是我的GCC和G++降级后没有升级，所以如果可以先试试把GCC和G++升级到最高版本，然后运用第一种方式安装驱动，第一种方式下面会提到。如果有需求要禁用Nouveau驱动的，那我也给出禁用Nouveau驱动的方法。

Nouveau驱动禁用方法：

```
sudo gedit /etc/modprobe.d/blacklist.conf或者sudo vim /etc/modprobe.d/blacklist.conf在最后两行添加：blacklist nouveauoptions nouveau modeset=0     执行sudo update -initramfs -u   
```

接下来讲三种安装方法：

1\. 使用标准Ubuntu 仓库进行自动安装

```
sudo ubuntu-drivers devicessudo ubuntu-drivers autoinstall完成后重启 就可完成安装NVIDIA驱动
```

2.使用PPA仓库进行自动安装

使用图形驱动程序PPA存储库允许我们安装NVIDIA beta驱动程序，但是这种方法存在不稳定的风险

```
sudo add-apt-repository ppa:graphics-drivers/ppa      sudo apt update         sudo ubuntu-drivers devices sudo apt install nvidia-xxx        
```

3\. 使用官方的NVIDIA驱动进行手动安装

先要搞清楚你的nvidia显卡是什么什么型号

```
lshw -numeric -C display或者lspci -vnn | grep VGA  
```

然后到NVIDIA官网下载对应你显卡型号的最新版本驱动进行下载  保存到你自己的路径文件夹

NVIDIA官网驱动下载地址： [https://www.nvidia.com/zh-cn/](https://www.nvidia.com/zh-cn/)           进入后选择最上面的驱动程序 就可以自行选择自己的显卡

```
sudo telinit 3用cd 进入你放nvidia驱动的路径用 ./  或者  bash  进行安装安装的过程如下：(按照以下步骤)Accept LicenseThe distribution-provided pre-install script failed! Are you sure you want to continue?CONTINUE INSTALLATIONWould you like to run the nvidia-xconfig utility?YES之后执行sudo reboot 
```

上面三种方法结束后，需要检验是否安装好了nvidia驱动。

```
sudo reboot  sudo nvidia-smi  
```

![](https://img-blog.csdn.net/20180809215859530?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ld19kZWxldGVf/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果出现和上图相似的内容就说明是安装成功了。