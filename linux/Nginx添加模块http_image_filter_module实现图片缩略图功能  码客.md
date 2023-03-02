

Nginx添加模块http\_image\_filter\_module实现图片缩略图功能

## [](https://www.psvmc.cn/article/2020-06-13-nginx-image-filter.html#%E5%89%8D%E8%A8%80 "前言")前言

> 不推荐使用，Windwos环境下载暂时没有找到添加该模块的方式，只支持Linux环境。

我们可能服务器上使用yum安装的Nginx，这时要新增模块的思路大致如下

1.  [官网](http://nginx.org/en/download.html)下载相同版本的nginx源码包
2.  编译安装并指定需要的模块(第三方模块需要单独下载对应的包)
3.  注意只编译`make`，不要编译安装`make install`. 编译完成会在objs目录下生成可执行文件nginx
4.  复制nginx 可执行文件`cp objs/nginx /usr/sbin/`
5.  复制模块`objs/ngx_http_image_filter_module.so`
6.  配置模块

## [](https://www.psvmc.cn/article/2020-06-13-nginx-image-filter.html#%E5%85%B7%E4%BD%93%E6%AD%A5%E9%AA%A4 "具体步骤")具体步骤

## [](https://www.psvmc.cn/article/2020-06-13-nginx-image-filter.html#%E6%9F%A5%E7%9C%8B%E5%8E%9FNginx "查看原Nginx")查看原Nginx

查看当前的版本及模块

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>nginx -V</span><br></pre></td></tr></tbody></table>

会看到如下

> nginx version: nginx/1.12.2
> 
> configure arguments: –prefix=/usr/share/nginx –sbin-path=/usr/sbin/nginx –modules-path=/usr/lib64/nginx/modules –conf-path=/etc/nginx/nginx.conf –error-log-path=/var/log/nginx/error.log –http-log-path=/var/log/nginx/access.log –http-client-body-temp-path=/var/lib/nginx/tmp/client\_body –http-proxy-temp-path=/var/lib/nginx/tmp/proxy –http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi –http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi –http-scgi-temp-path=/var/lib/nginx/tmp/scgi –pid-path=/run/nginx.pid –lock-path=/run/lock/subsys/nginx –user=nginx –group=nginx –with-file-aio –with-ipv6 –with-http\_auth\_request\_module –with-http\_ssl\_module –with-http\_v2\_module –with-http\_realip\_module –with-http\_addition\_module –with-http\_xslt\_module=dynamic –with-http\_image\_filter\_module=dynamic –with-http\_geoip\_module=dynamic –with-http\_sub\_module –with-http\_dav\_module –with-http\_flv\_module –with-http\_mp4\_module –with-http\_gunzip\_module –with-http\_gzip\_static\_module –with-http\_random\_index\_module –with-http\_secure\_link\_module –with-http\_degradation\_module –with-http\_slice\_module –with-http\_stub\_status\_module –with-http\_perl\_module=dynamic –with-mail=dynamic –with-mail\_ssl\_module –with-pcre –with-pcre-jit –with-stream=dynamic –with-stream\_ssl\_module –with-google\_perftools\_module –with-debug –with-cc-opt=’-O2 -g -pipe -Wall -Wp,-D\_FORTIFY\_SOURCE=2 -fexceptions -fstack-protector-strong –param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic’ –with-ld-opt=’-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E’

查找

> –with-http\_image\_filter\_module=dynamic

如果已经包含该模块就可以跳过编译安装部分，直接查看配置部分。

## [](https://www.psvmc.cn/article/2020-06-13-nginx-image-filter.html#%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85 "编译安装")编译安装

> 如果本地已有该模块 这一步则跳过

查看Nginx的路径

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span><span>which</span> nginx</span><br></pre></td></tr></tbody></table>

结果如下

> /usr/sbin/nginx

备份配置文件

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>cp -r /etc/nginx /etc/nginx.bak</span><br></pre></td></tr></tbody></table>

从[官网](http://nginx.org/en/download.html)下载对应版本的源码

比如我的是1.12.2 下载地址[http://nginx.org/download/nginx-1.12.2.tar.gz](http://nginx.org/download/nginx-1.12.2.tar.gz)

下载

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span><span>cd</span> /usr/<span>local</span></span><br><span>wget http://nginx.org/download/nginx-1.12.2.tar.gz</span><br><span>tar -xzvf nginx-1.12.2.tar.gz</span><br></pre></td></tr></tbody></table>

HttpImageFilterModule模块需要依赖gd-devel的支持

安装

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>yum install gd-devel</span><br><span>yum install libxslt-devel</span><br><span>yum -y install perl-devel perl-ExtUtils-Embed</span><br></pre></td></tr></tbody></table>

编译

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span><span>cd</span> nginx-1.12.2</span><br></pre></td></tr></tbody></table>

在上面的命令里面加上 `--with-http_image_filter_module=dynamic` 开始执行编译，编译的时候依赖的模块没有安装导致错误，只需安装对应的模块即可。

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/<span>log</span>/nginx/error.log --http-log-path=/var/<span>log</span>/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug</span><br></pre></td></tr></tbody></table>

编译

执行 make，切记不需要执行make install，此处只需要得到nginx重新编译的可执行文件，并不需要重新安装替换掉原来的nginx

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>make</span><br></pre></td></tr></tbody></table>

执行完成会在 objs 目录下生成对应的可执行文件nginx

复制

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>cp objs/ngx_http_image_filter_module.so /usr/lib64/nginx/modules/</span><br><span>cp -rfp objs/nginx /usr/sbin/</span><br></pre></td></tr></tbody></table>

查看模块是否加载

查看

`/etc/nginx/nginx.conf`中如果有

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>include /usr/share/nginx/modules/*.conf;</span><br></pre></td></tr></tbody></table>

打开对应目录

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span><span>cd</span> /usr/share/nginx/modules/</span><br></pre></td></tr></tbody></table>

如果包含`mod-http-image-filter.conf`就不用再加载了。

如果没有`/etc/nginx/nginx.conf`中加载模块

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>load_module <span>"modules/ngx_http_image_filter_module.so"</span>;</span><br></pre></td></tr></tbody></table>

## [](https://www.psvmc.cn/article/2020-06-13-nginx-image-filter.html#%E9%85%8D%E7%BD%AE "配置")配置

/etc/nginx/conf.d/mysite.conf 配置图片处理

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br></pre></td><td><pre><span>location ~* /(.+)\.(jpg|jpeg|gif|png)!(\d+)x(\d+)$ {</span><br><span>        <span>set</span> <span>$w</span> <span>$3</span>;</span><br><span>        <span>set</span> <span>$h</span> <span>$4</span>;</span><br><span>        image_filter resize  <span>$w</span> <span>$h</span>;</span><br><span>        image_filter_buffer  10M;</span><br><span>        image_filter_jpeg_quality 75;</span><br><span>        try_files /<span>$1</span>.<span>$2</span>  /notfound.jpg;</span><br><span>        expires 1d;</span><br><span>        add_header <span>'Access-Control-Allow-Origin'</span> <span>'*'</span>;</span><br><span>        add_header <span>'Access-Control-Allow-Credentials'</span> <span>'true'</span>;</span><br><span>        root  /data/wwwjarapi/8908schoolfiletest/;  </span><br><span>    }</span><br><span></span><br><span>location ~* /static {</span><br><span>add_header <span>'Access-Control-Allow-Origin'</span> <span>'*'</span>;</span><br><span>add_header <span>'Access-Control-Allow-Credentials'</span> <span>'true'</span>;</span><br><span>root  /data/wwwjarapi/8908schoolfiletest/;  </span><br><span>    }</span><br></pre></td></tr></tbody></table>

如果原图的地址为

> [www.psvmc.cm/abc/1.jpg](http://www.psvmc.cm/abc/1.jpg)

那么缩放后的图片地址为

> [www.psvmc.cm/abc/1.jpg!200x200](http://www.psvmc.cm/abc/1.jpg!200x200)

**Nginx配置生效的优先级**

[https://www.psvmc.cn/article/2020-02-11-nginx-config.html](https://www.psvmc.cn/article/2020-02-11-nginx-config.html)

**错误示例**

比如我按照如下配置就不行

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br></pre></td><td><pre><span>location ~* /(.+)\.(jpg|jpeg|gif|png)!(\d+)x(\d+)$ {</span><br><span>        <span>set</span> <span>$w</span> <span>$3</span>;</span><br><span>        <span>set</span> <span>$h</span> <span>$4</span>;</span><br><span>        image_filter resize  <span>$w</span> <span>$h</span>;</span><br><span>        image_filter_buffer  10M;</span><br><span>        image_filter_jpeg_quality 75;</span><br><span>        try_files /<span>$1</span>.<span>$2</span>  /notfound.jpg;</span><br><span>        expires 1d;</span><br><span>        add_header <span>'Access-Control-Allow-Origin'</span> <span>'*'</span>;</span><br><span>        add_header <span>'Access-Control-Allow-Credentials'</span> <span>'true'</span>;</span><br><span>        root  /data/wwwjarapi/8908schoolfiletest/;  </span><br><span>    }</span><br><span></span><br><span>location ^~ /static {</span><br><span>add_header <span>'Access-Control-Allow-Origin'</span> <span>'*'</span>;</span><br><span>add_header <span>'Access-Control-Allow-Credentials'</span> <span>'true'</span>;</span><br><span>root  /data/wwwjarapi/8908schoolfiletest/;  </span><br><span>    }</span><br></pre></td></tr></tbody></table>

因为后面的配置优先级高 导致第一个配置不生效