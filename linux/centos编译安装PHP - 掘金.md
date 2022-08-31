携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第31天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## **centos编译安装PHP7.4**

我的服务器是阿里云的centos7.8

### **1：下载PHP源码包**

一般我都是从官方网站下载：[php.net](https://link.juejin.cn/?target=https%3A%2F%2Fphp.net%2F "https://php.net/")

![1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7cd0dfd2a4b43068d1e46da12a9cd8e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954625310400.png")

当然，我是直接在服务器上下载的：

```
mkdir -p /usr/local/download
cd /usr/local/download
wget https://www.php.net/distributions/php-7.4.11.tar.gz
复制代码
```

### **2：为服务器安装编译环境**

我使用的服务器是纯净的环境，没有编译的环境，因此需要安装：

```
yum -y install gcc gcc-c++ autoconf automake build-essential zlib zlib-devel openssl openssl-devel pcre pcre-devel
复制代码
```

### **3：安装PHP7.4所需要的的编译环境**

```
yum install -y openssl-devel libxml2-devel bzip2-devel libcurl-devel libjpeg-devel libpng-devel freetype-devel libmcrypt-devel recode-devel libicu-devel libzip-devel sqlite-devel oniguruma-devel
复制代码
```

### **4：编译安装php7.4**

#### （1）：解压刚刚下载的源码包

```
tar -zxvf php-7.4.11.tar.gz
cd php-7.4.11
复制代码
```

#### （2）：编译（指定安装目录）并安装php-fpm

```
./configure --prefix=/usr/local/php --with-mysql --with-mysqli --with-pdo_mysql --with-iconv-dir --with-zlib --with-libxml-dir --enable-xml --with-curl
 --enable-fpm
 --enable-mbstring --with-gd --with-openssl --with-mhash --enable-sockets --with-xmlrpc --enable-zip --enable-soap --enable-bcmath
复制代码
```

![2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee0bae99832c4b859eead396e0052b15~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954632259963.png")

如上图所示，编译成功

#### （3）：安装

```
make&&make install
复制代码
```

### **5：简化PHP执行命令**

上一步操作中我们已经将PHP安装成功。

在服务器端执行PHP文件格式是这个样子的：

```
/usr/local/php/bin/php index.php
复制代码
```

但是我们使用yum源安装的PHP，在服务器上可以直接使用php命令来执行：

```
php index.php
复制代码
```

在当前登录用户（我是root）家目录下的.bash\_profile中添加如下内容：

```
vim /root/.bash_profile
复制代码
```

添加内容：

```
alias php=/usr/local/php/bin/php
alias phpfpm=/usr/local/php/sbin/php-fpm
复制代码
```

**修改之后文件内容：**

```
# .bash_profile
 
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
 
# User specific environment and startup programs
 
PATH=$PATH:$HOME/bin
 
export PATH
alias php=/usr/local/php/bin/php
alias phpfpm=/usr/local/php/sbin/php-fpm
复制代码
```

重载一下文件：

```
source /root/.bash_profile
复制代码
```

或者创建软连接

```
ln -s /usr/local/php/bin/php /usr/bin/php74
ln -s /usr/local/php/sbin/php-fpm /usr/bin/php-fpm74
复制代码
```

理论上就可以使用php命令来执行PHP文件了

![3.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7386d69eabe4512bafd0d3ed938599e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954639437915.png")

### **6：启动php-fpm报错解决方案**

这里只记录在我安装的时候遇到的错误。

#### **（1）：找不到php-fpm.conf文件**

```
[19-Apr-2018 16:02:08] ERROR: failed to open configuration file '/usr/local/php/etc/php-fpm.conf': No such file or directory (2)
 [19-Apr-2018 16:02:08] ERROR: failed to load configuration file '/usr/local/php/etc/php-fpm.conf'
 [19-Apr-2018 16:02:08] ERROR: FPM initialization failed
复制代码
```

错误信息说是找不到php-fpm.conf

解决方案：

到php的配置目录

```
cd /usr/local/php/etc
复制代码
```

有一个php-fpm.conf.default的文件，cp复制\\

```
cp php-fpm.conf.default php-fpm.conf
复制代码
```

编辑 php-fpm.conf 找到以下配置项， 配置如下

```
pid = /usr/local/php/var/run/php-fpm.pid
复制代码
```

再次运行

```
/usr/local/php/sbin/php-fpm
复制代码
```

进入下一个报错、

#### （2）：找不到[www.conf](https://link.juejin.cn/?target=http%3A%2F%2Fwww.conf%2F "http://www.conf/")配置文件

```
[root@iZuf60ynur81p6k0ysvtneZ etc]# /usr/local/php/sbin/php-fpm
[13-Oct-2020 18:03:57] WARNING: Nothing matches the include pattern '/usr/local/php/etc/php-fpm.d/*.conf' from /usr/local/php/etc/php-fpm.conf at line 145.
[13-Oct-2020 18:03:57] ERROR: No pool defined. at least one pool section must be specified in config file
[13-Oct-2020 18:03:57] ERROR: failed to post process the configuration
[13-Oct-2020 18:03:57] ERROR: FPM initialization failed
复制代码
```

解决方案：

进入php安装目录：

```
/usr/local/php/etc/php-fpm.d
复制代码
```

当前目录下有一个[www.conf.default](https://link.juejin.cn/?target=http%3A%2F%2Fwww.conf.default%2F "http://www.conf.default/")文件

```
cp 
www.conf.default
 
www.conf
复制代码
```

再次执行

```
/usr/local/php/sbin/php-fpm
复制代码
```

我的php-fpm便启动成功。

### **7：php.ini文件**

Php编译安装成功之后，是没有php.ini文件的，需要我们从源码包中复制过去。

#### **（1）：查询php.ini文件位置**

```
php -i | grep php.ini
[root@iZuf60ynur81p6k0ysvtneZ /]# php -i | grep php.ini
Configuration File (php.ini) Path => /usr/local/php/lib
复制代码
```

Php.ini文件存放目录在/usr/local/php/lib中

#### （2）：获取php.ini文件

找到了php.ini文件的存放位置，那么php.ini文件在哪获取呢？

进入我们的源码包，我们可以看到：

![4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca4a0b80481148cea556d3f9c08c8e51~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954646458770.png")

使用cp命令，将文件复制到指定位置并重命名php.ini

```
cp /usr/local/download/php-7.4.11/php.ini-development /usr/local/php/lib/php.ini
复制代码
```

### **8：安装php扩展**

**敲黑板，敲黑板，安装扩展这个很重要。**

我们使用php -m 命令可以查看PHP默认为我们安装了那些扩展：

```
[root@iZuf60ynur81p6k0ysvtneZ /]# php -m
[PHP Modules]
Core
ctype
date
dom
fileinfo
filter
hash
iconv
json
libxml
pcre
PDO
pdo_sqlite
Phar
posix
Reflection
session
SimpleXML
SPL
sqlite3
standard
tokenizer
xml
xmlreader
xmlwriter
 
[Zend Modules]
复制代码
```

其实，在PHP源码包中，为我们提供了部分的PHP的扩展源码包，这里没有的需要去PHP官网下载。在根目录下的ext目录中，如下图所示：

![5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/498439e16b5e4244b699777e08d26137~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954653794425.png")

所以，我们如果需要安装相关的拓展，不需要去别的地方找，源码包啥都给你提供了。

下面，记录一下，我安装gd库扩展的过程：

#### （1）：进入源码包的gd库目录：

```
cd /usr/local/download/php-7.4.11/ext/gd  #这里是我的目录，需要改成你自己的目录
复制代码
```

#### （2）： 生成configure

```
/usr/local/php/bin/phpize  
#这里是我的目录，需要改成你自己的目录
复制代码
```

如下图所示：注意文件的生成时间

![6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a32bc18845c44e139696d93f667b719a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954660110338.png")

#### （3）：执行编译

```
./configure --with-php-config=/usr/local/php/bin/php-config --with-jpeg-dir=/usr/local/jpeg/
复制代码
```

有的时候会出现如下警告：

```
creating libtool
appending configuration tag "CXX" to libtool
configure: patching config.h.in
configure: creating ./config.status
config.status: creating config.h
复制代码
```

不需要理会他

#### （4）：安装

```
make&&make install
复制代码
```

安装成功之后会返回文件安装位置：

```
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20190902/
复制代码
```

#### （5）：配置php.ini文件，使安装生效

在php.ini中添加

```
extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20190902/gd.so
复制代码
```

修改完成之后

查看php-fpm的进程

```
ps -aux | grep php-fpm
[root@iZuf60ynur81p6k0ysvtneZ lib]# ps -aux|grep php-fpm
root     14783  0.0  0.3 212116  6244 ?        Ss   Oct13   0:02 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
nobody   14784  0.0  0.3 212116  7448 ?        S    Oct13   0:00 php-fpm: pool www
nobody   14785  0.0  0.3 212116  7444 ?        S    Oct13   0:00 php-fpm: pool www
root     18338  0.0  0.0 112808   968 pts/0    R+   10:01   0:00 grep --color=auto php-fpm
复制代码
```

终止php-fpm进程：

```
Kill 14783
复制代码
```

重启php-fpm

```
/usr/local/php/sbin/php-fpm
复制代码
```

使用php -m查看安装模块：

```
[root@iZuf60ynur81p6k0ysvtneZ lib]# php -m
[PHP Modules]
Core
ctype
date
dom
fileinfo
filter
gd
hash
iconv
json
libxml
pcre
PDO
pdo_sqlite
Phar
posix
Reflection
session
SimpleXML
SPL
sqlite3
standard
tokenizer
xml
xmlreader
xmlwriter
 
[Zend Modules]
复制代码
```

我们发现gd库安装成功。

### **9：查看已安装的PHP扩展文件**

已安装的扩展目录在：/usr/local/php/include/php/ext

![7.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0824afd99fd479ea0a1a42043fdddf5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp "1620954670117405.png")

### **10：设置开机启动**

确保rc.local 文件有执行权限，否则，开机启动不生效

```
vim  /etc/rc.d/rc.local
复制代码
```

添加如下内容：

```
/usr/local/php/sbin/php-fpm
复制代码
```

至此 PHP 编译安装完成

有好的建议，请在下方输入你的评论。

欢迎访问个人博客 [guanchao.site](https://link.juejin.cn/?target=https%3A%2F%2Fguanchao.site "https://link.juejin.cn/?target=https%3A%2F%2Fguanchao.site")

欢迎访问我的小程序：打开微信->发现->小程序->搜索“时间里的”