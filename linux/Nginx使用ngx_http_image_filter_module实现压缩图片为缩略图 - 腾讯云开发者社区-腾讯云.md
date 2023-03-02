ubuntu系统下，先安装ngx\_http\_image\_filter\_module这个模块

先看看自己的源

cat /etc/apt/sources.list.d/nginx-stable.list

deb http://nginx.org/packages/ubuntu/ xenial nginx deb-src http://nginx.org/packages/ubuntu/ xenial nginx

```
apt-get install -y nginx nginx-module-image-filter
```

在/etc/nginx/nginx.conf 的开头部分引入模块

load\_module modules/ngx\_http\_image\_filter\_module.so;

配置vhost

例如我这个例子，一定注意看if判断部分 ， 语法错误也不行，这个就是当传递width和height参数的时候，就按照参数的进行裁剪

```
    location ~* .*\.(JPG|jpg|gif|png|PNG)$ {
        root /var/www/html/go-fly2;
        # 图片默认高度
       set $width -;  
        # 图片默认高度
        set $height -;  
        if ( $arg_width != "" ) { 
                set $width $arg_width;
        }   
        if ( $arg_height != "" ) { 
                set $height $arg_height;
        }   
        image_filter test;
    
        image_filter resize $width $height; 
        image_filter_buffer 100M;  
        image_filter_jpeg_quality 95; 
} 
```

https://xxxx/xxxxx.jpg?width=100

类似上面这样调用

本文分享自作者个人站点/博客：https://www.cnblogs.com/taoshihan复制

如有侵权，请联系

cloudcommunity@tencent.com

删除。