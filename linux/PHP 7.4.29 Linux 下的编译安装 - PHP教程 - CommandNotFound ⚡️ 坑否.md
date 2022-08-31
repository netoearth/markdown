[![](https://commandnotfound.cn/img/php.png)](https://commandnotfound.cn/php/2/17/seduction.aspx)

### 1\. 下载 PHP 7.4.29 源码

___

从[PHP官网](https://www.php.net/downloads.php)获取所需版本的 PHP 的下载地址。

```
#更新内容大同小异，基本都是修复安全问题以及一些小错误。团队建议使用对应分支的开发者能升级到最新版本。 wget https://www.php.net/distributions/php-7.4.29.tar.gz
```

### 2.解压 PHP 7.4.29

___

```
tar -xvf php-7.4.29.tar.gz
```

### 3\. 进入 PHP 7.4.29 目录

___

```
cd php-7.4.29
```

### 4\. 安装 PHP 7.4.29 依赖包

___

```
yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel \libcurl libcurl-devel libjpeg libjpeg-devel libpng \libpng-devel freetype freetype-devel gmp gmp-devel \libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel #PHP 7.4 新的改动:yum install libsqlite3x-devel -y yum install oniguruma-devel -y
```

### 5\. PHP 7.4.29 的编译配置

___

如果出现错误，基本都是上一步的依赖文件没有安装所致

可以从这一步起参考[PHP官方安装说明](http://php.net/manual/zh/install.unix.nginx.php)

**提示**：`prefix` 是安装目录，可以安装到你想安装的目录（类似在 Windows 下运行 setup.exe 安装包时候，让你选的安装目录），比如 `/Data/apps/php-7/`，在Linux下编译安装大部分应用一般先 `./configure`，然后 `make && make install`（make 和 make install 也可以分开执行），最后可选 `make clean` 的参考步骤。

_注意_：PHP 7.4.29 开始，已经不支持 `--with-gd`，得改成 `--enable-gd`，一些可能的选项变化如下：

```
configure: WARNING: unrecognized options: --with-libxml-dir, --with-pcre-regex, --with-pcre-dir, --with-gd, --with-jpeg-dir, --with-png-dir, --with-freetype-dir, --enable-mbregex-backtrack, --with-onig, --enable-wddx, --enable-zip 上面这些已不再被 PHP 7.4.X 支持： #Old 老的 PHP 编译选项...--with-png-dir=/usr/include/--with-jpeg-dir=/usr/include/--with-freetype-dir=/usr/include/ #New - PHP 7.4.X 之后新的 PHP 编译选项--with-png=/usr/include/--with-jpeg=/usr/include/--with-freetype=/usr/include/ -------------------------------------------# PCRE 高版本依赖（如果需要安装扩展 --with-external-pcre），官网新的 PCRE2 改为在 Github 中：wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.40/pcre2-10.40.tar.gztar xvf pcre2-10.40.tar.gz cd pcre2-10.40 ./configure --prefix=/usr/local/pcre2 \--enable-pcre2-16 \--enable-pcre2-32 \--enable-jit \--enable-jit-sealloc make && make install export PKG_CONFIG_PATH=/usr/local/pcre2/lib/pkgconfig/ -------------------------------------------# GD 库要求大于  gdlib >= 2.1.0：$ wget https://github.com/libgd/libgd/releases/download/gd-2.3.1/libgd-2.3.1.tar.gz $ tar xvfz libgd-2.3.1.tar.gz$ cd libgd-2.3.1 $ ./configure --prefix=/usr/local/libgd/2_3_1...** Configuration summary for libgd 2.3.1:    Support for gd/gd2 images:        yes   Support for Zlib:                 yes   Support for PNG library:          yes   Support for JPEG library:         yes   Support for WebP library:         no   Support for TIFF library:         no   Support for Freetype 2.x library: yes   Support for Fontconfig library:   no   Support for Xpm library:          no   Support for liq library:          no   Support for complex text:         no   Support for pthreads:             yes    ....  $ make && make install export PKG_CONFIG_PATH=/usr/local/libgd/2_3_1/lib/pkgconfig ------------------------------------------- 注意如上，如果有多个的话，按下面去 export，之后可以 echo $PKG_CONFIG_PATH 来验证： export PKG_CONFIG_PATH=/usr/local/pcre2/lib/pkgconfig/:/usr/local/libgd/2_3_1/lib/pkgconfig:/usr/local/lib64/pkgconfig/
```

更多请看 PHP 官网说明：[PHP:Migration to pkg-config](https://www.php.net/manual/en/migration74.other-changes.php#migration74.other-changes.pkg-config)，也可以用 `./configure --help` 命令可以列出当前可用的所有参数。

```
./configure  \--prefix=/Data/apps/php7.4.29  \--with-config-file-path=/Data/apps/php7.4.29/etc  \--enable-fpm  \--enable-gd  \--with-external-gd  \--with-fpm-user=nginx  \--with-fpm-group=nginx  \--enable-inline-optimization  \--disable-debug  \--disable-rpath  \--enable-shared  \--enable-soap  \--with-libxml \--with-xmlrpc  \--with-openssl  \--with-external-pcre \--with-sqlite3  \--with-zlib  \--enable-bcmath  \--with-iconv  \--with-bz2  \--enable-calendar  \--with-curl  \--with-cdb  \--enable-dom  \--enable-exif  \--enable-fileinfo  \--enable-filter  \--enable-ftp \--with-openssl-dir \--with-jpeg \--with-zlib-dir \--with-freetype \--enable-gd-jis-conv \--with-gettext \--with-gmp \--with-mhash \--enable-json \--enable-mbstring \--enable-mbregex \--enable-pdo \--with-mysqli=mysqlnd \--with-pdo-mysql=mysqlnd \--with-pdo-sqlite \--with-readline \--enable-session \--enable-shmop \--enable-simplexml \--enable-sockets \--enable-sysvmsg \--enable-sysvsem \--enable-sysvshm \--with-xsl \--with-zip \--enable-mysqlnd-compression-support \--with-pear \--enable-opcache
```

看到以下提示，证明已经正确安装

```
+--------------------------------------------------------------------+| License:                                                           || This software is subject to the PHP License, available in this     || distribution in the file LICENSE. By continuing this installation  || process, you are bound by the terms of this license agreement.     || If you do not agree with the terms of this license, you must abort || the installation process at this point.                            |+--------------------------------------------------------------------+ Thank you for using PHP.
```

这里可能会报很多 PHP 7.X 编译错误，一个个解决即可。这里有大部分的解决方法，`列出几个常见的`：

```
checking for png_write_image in -lpng… yes If configure fails try –with-xpm-dir= configure: error: freetype.h not found.解决: Reconfigure your PHP with the following option. --with-xpm-dir=/usr checking for png_write_image in -lpng… yes configure: error: libXpm.(a|so) not found.解决: yum install libXpm-devel checking for bind_textdomain_codeset in -lc… yes checking for GNU MP support… yes configure: error: Unable to locate gmp.h 解决: yum install gmp-devel configure: error: Please reinstall the libzip distribution解决: 安装最新libzip，如果是1.5以上版本的 libzip，需要CMAKE编译安装 configure: error: off_t undefined; check your library configuration解决: echo '/usr/local/lib64/usr/local/lib/usr/lib/usr/lib64'>>/etc/ld.so.conf&&ldconfig -v #如果提示以下错误，此时把这两个编译项从配置中删除即可(上述命令中已删除)configure: WARNING: unrecognized options: --with-mcrypt, --enable-gd-native-ttf undefined reference to `libiconv_open' #见下方解决方案
```

### 6\. 编译 PHP 7 并正式安装

___

```
#多核CPU机器make的时候，可以考虑用 make -j8make && make install
```

如果没有报错就表示安装完成。

### 7\. 配置 PHP 7 环境变量

___

```
vim /etc/profile
```

在末尾追加

```
#注意这个路径，是你编译安装的时候prefix的路径PATH=$PATH:/Data/apps/php7.4.29/bin export PATH
```

Linux 下执行 `source` 命令使得改动立即生效 \*\[ _注意_：这里可能有个坑，请看到文章最后 \]。

```
source /etc/profile
```

### 8.配置 `php-fpm`

___

把所需的几个配置文件放到需要的位置

```
#php.ini 配置文件cp php.ini-production /Data/apps/php7.4.29/etc/php.ini #conf 配置文件mv /Data/apps/php7.4.29/etc/php-fpm.conf.default /Data/apps/php7.4.29/etc/php-fpm.conf #www.conf 文件mv /Data/apps/php7.4.29/etc/php-fpm.d/www.conf.default /Data/apps/php7.4.29/etc/php-fpm.d/www.conf #php-fpm 文件，用于配置 service 服务启动等cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm #添加 php-fpm 执行权限chmod +x /etc/init.d/php-fpm
```

### 9.启动 `php-fpm`

___

```
#提示可以vim 这个php-fpm文件，看看里面写的是什么？/etc/init.d/php-fpm start #其他参数还有 stop, restart,status, force-quit, reload, configtest..#具体可以 VIM /etc/init.d/php-fpm 这个文件查阅. [root@Dev_Test ~]$ /etc/init.d/php-fpm restartGracefully shutting down php-fpm . doneStarting php-fpm  done #查看 php-fpm 状态[root@Dev_Test ~]$ /etc/init.d/php-fpm statusphp-fpm (pid 19214) is running...
```

### 编译安装时碰到 ``undefined reference to `libiconv_open'`` 问题解决方法

___

如果在官网 [http://www.gnu.org/software/libiconv/](http://www.gnu.org/software/libiconv/) 重新编译安装还不行，则需要 vim Makefile，添加 `-liconv` 到最后，修改位置大概在 `118` 行。

```
EXTRA_LIBS = -lcrypt -lzip -lzip -lz -lexslt -lresolv -lcrypt -lreadline -lncurses -lrt -lgmp -lpng -lz -ljpeg -lbz2 -lz -lrt -lm -ldl -lnsl -lxml2 -lz -lm -ldl -lssl -lcrypto -lcurl -lxml2      -lz -lm -ldl -lssl -lcrypto -lfreetype -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lcrypt -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -     lxslt -lxml2 -lz -ldl -lm -lssl -lcrypto -lcrypt -liconv
```

再次 `make && make install` 通过。

### PHP 7 编译安装的坑：

___

Nginx Error! The page you are looking for is temporarily unavailable. Please try again later.

参考[这里](https://www.scalescale.com/tips/nginx/php-fpm-connect-to-unixtmpphp5-fpm-sock-failed-2-no-such-file-or-directory/)的解决方案

```
cp /etc/php-fpm.d/www.conf.rpmsave /etc/php-fpm.d/www.confvi /etc/php-fpm.d/www.conf #listen 改为listen = 127.0.0.1:9000; #nginx 配置文件也改vi /etc/nginx/conf.d/yoursite.com.conf fastcgi_pass 127.0.0.1:9000; #重启php-fpm 和 nginx sudo /etc/init.d/php-fpm restartsudo systemctl restart nginx
```

Nginx 请求转发到 PHP 处理模块，我一般采用文件`PID`的方式，至于 PHP 在 nginx 配置里，是采用**端口监听方式**还是**PID方式**，后面我们继续阐述

编译安装 PHP 7.x 的时候，如果提示 Libzip 版本过低或提示 reinstall 解决方法

```
checking for libzip... configure: error: system libzip must be upgraded to version >= 0.11 #先删除 libzip 和 libzip-develyum remove libzip -y
```

到 [libzip 官网](https://libzip.org/ "libzip 官网")，下载最新版的编译安装（截至目前为止 Current version is `libzip-1.7.1.tar.gz`）。

```
wget https://libzip.org/download/libzip-1.8.0.tar.gztar -zxvf libzip-*cd libzip*mkdir build && cd build && cmake .. && make && make install #到这里你以为就没坑了吗？编译PHP7.4可能依然提示没有 libzip …#这时候看一下 libzip.so 的路径，比如我的在 /usr/local/lib64 下，那么： export PKG_CONFIG_PATH="/usr/local/lib64/pkgconfig/"  #让 configure 过程找到 libzip，这样就可以完成 PHP7.4 的升级编译安装了
```

注意：如果多个，请搜索文章开头的 `PKG_CONFIG_PATH` !!!

接下来的坑，如果提示 cmake: command not found，需要先 yum install cmake 或手动编译更高版本……由于这里需要高版本的 CMake，于是还得去 [CMake 官网](https://cmake.org/download/ "CMake 官网")下载最新版的 CMake。

```
#2022年04月12日最新 CMAKE 版wget https://github.com/Kitware/CMake/releases/download/v3.23.1/cmake-3.23.1.tar.gz #接下来解压缩并运行tar -xvf cmake-3.23.1.tar.gzcd cmake-3.23.1 #这一步很关键，如果没有指定prefix，#后面使用时会报错 Could not find CMAKE_ROOT./bootstrap --prefix=/usr make && make install [root@API-01 /Data/tools/cmake-3.23.1]$ cmake --versioncmake version 3.23.1 CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

安装 CMake，执行 `./bootstrap` 的时候，有可能会遇到的坑，会提示如下错误：

```
---------------------------------------------CMake 3.23.1, Copyright 2000-2022 Kitware, Inc. and ContributorsC compiler on this system is: cc---------------------------------------------Error when bootstrapping CMake:Cannot find a C++ compiler that supports both C++11 and the specified C++ flags.Please specify one using environment variable CXX.The C++ flags are "".They can be changed using the environment variable CXXFLAGS.See cmake_bootstrap.log for compilers attempted.
```

原来是没有装 `C++` 的编译器

```
Ubuntu： apt-get install gcc g++CentOS：yum install gcc gcc-c++
```

再次执行 `./bootstrap --prefix=/usr` 即可。

### sudo 模式下，`source /etc/profile` 可能的坑：

___

执行完 source /etc/profile之后，当时一切正常，但是注销下次再进来执行类似 `php -v` 则提示 php: command not found，每次都要再 `source /etc/profile` 才好用，退出再次登录，就又不好用了，反反复复，不胜其烦……通常这种情况都是 `sudo` 模式下。

```
#解决这个坑，需要在 vim ~/.bashrc 最后一行中加入: # .bashrc # User specific aliases and functions alias rm='rm -i'alias cp='cp -i'alias mv='mv -i' # Source global definitionsif [ -f /etc/bashrc ]; then        . /etc/bashrcfi source /etc/profile #注意在这里加入即可。#如果用的 zsh，就在 .zshrc 里加入 source /etc/profile
```

### PHP 7.4.29 编译安装扩展阅读：

___

-   [PHP: Migration to pkg-config](https://www.php.net/manual/zh/migration74.other-changes.php#migration74.other-changes.pkg-config)
-   [Centos7 下 PHP 源码编译和通过 yum 安装的区别和以后的选择](https://segmentfault.com/a/1190000020106079)
-   [PHP 命令行 Commands](https://commandnotfound.cn/php/2/297/PHP-%E5%91%BD%E4%BB%A4%E8%A1%8C-Commands)

___

___

### 发表评论