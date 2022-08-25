## nginx泛解析

[![](https://upload.jianshu.io/users/upload_avatars/3816230/40cd33788a46?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/f9011441ade4)

2016.11.25 15:31:06字数 175阅读 3,253

网站的目录结构为：

## tree /home/wwwroot/[exehack.net](https://link.jianshu.com/?t=http://exehack.net)

/home/wwwroot/[exehack.net](https://link.jianshu.com/?t=http://exehack.net)  
├── bbs  
│ └── index.html  
└── www  
└── index.html  
2 directories, 2 files

/home/wwwroot/[exehack.net](https://link.jianshu.com/?t=http://exehack.net)为nginx的安装目录下默认的存放源代码的路径。  
bbs为论坛程序源代码路径；www为主页程序源代码路径；把相应程序放入上面的路径通过；[http://www.exehack.net](https://link.jianshu.com/?t=http://www.exehack.net) 访问的就是主页[http://bbs.exehack.net](https://link.jianshu.com/?t=http://bbs.exehack.net) 访问的就是论坛，其它二级域名类推。

有2种方法，推荐方法一

```
server {
listen 80;
server_name ~^(?<subdomain>.+).exehack.net$;
access_log /data/wwwlogs/exehack.net_nginx.log combined;
index index.html index.htm index.php;
root /home/wwwroot/linuxeye/$subdomain/;
location ~ .php$ {
  fastcgi_pass unix:/dev/shm/php-cgi.sock;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  include fastcgi_params;
  }
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|ico)$ {
  expires 30d;
  }
location ~ .*\.(js|css)?$ {
  expires 7d;
  }
}

```

方法二、

```
server {
listen 80;
server_name *.exehack.net;
access_log /home/wwwlogs/exehack.net_nginx.log combined;
index index.html index.htm index.php;
if ($host ~* ^([^\.]+)\.([^\.]+\.[^\.]+)$) {
  set $subdomain $1;
  set $domain $2;
}
location / {
  root /home/wwwroot/exehack.net/$subdomain/;
  index index.php index.html index.htm;
}
location ~ .php$ {
  fastcgi_pass unix:/dev/shm/php-cgi.sock;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  include fastcgi_params;
  }
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|ico)$ {
  expires 30d;
  }
location ~ .*\.(js|css)?$ {
  expires 7d;
  }
}
```

nginx将泛解析的匹配域名绑定到子目录的配置方法如下

```
server {
     listen        80;
     server_name   domain.com    *.domain.com;

    if ($host ~* ^([^\.]+)\.([^\.]+\.[^\.]+)$) {
         set $subdomain $1;
         set $domain $2;
     }

    location / {
         root    /home/wwwroot/$domain/$subdomain/;
         index   index.php index.html index.htm;
         #include /home/wwwroot/$domain/$subdomain/.ngx.htaccess;
     }

    error_page   500 502 503 504  /50x.html;

    location = /50x.html {
         root   html;
     }

    location ~ \.php$ {
         root           /home/wwwroot/$domain/$subdomain/;
         fastcgi_pass   127.0.0.1:9100;
         fastcgi_index  index.php;
         fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
         include        fastcgi_params;
     }
 }

```

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/3816230/40cd33788a46?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/f9011441ade4)

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智...
    
    [![](https://upload-images.jianshu.io/upload_images/7328262-54f7992145380c10.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/46fd0faecac1)
-   第一章 Nginx简介 Nginx是什么 没有听过Nginx？那么一定听过它的“同行”Apache吧！Ngi...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/7-0993d41a595d6ab6ef17b19496eb2f21.jpg)JokerW](https://www.jianshu.com/u/951a20323565)阅读 31,219评论 24赞 999
    
    [![](https://upload-images.jianshu.io/upload_images/2007846-c716637460dee7a5?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/630e2e1ca57f)
-   Android 自定义View的各种姿势1 Activity的显示之ViewRootImpl详解 Activity...
    
-   朋友圈经济欣欣向荣 吃喝玩乐应有尽有 工作创业两不误 唯我独坐窗台 看车水马龙 思绪万千 恨自己 无为 怨 人间 ...
    
    [![](https://upload-images.jianshu.io/upload_images/8746283-a1fe6c09ffbf9717.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a185eac27a87)
-   忙忙碌碌的日子已经在记忆里模糊了。淅淅沥沥的雨让我忘记了这是个夏天。高三的日子曾经充斥着生活啊，现在不但不...
    
-   我的命是一缕空气装在气球里用长长的线系紧了口子飘在空中让我自己的躯体伸出手去用力地握着可是它太轻飘飘的线太细了手指...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/3-9a2bcc21a5d89e21dafc73b39dc5f582.jpg)艾黑丫](https://www.jianshu.com/u/24dc3139d00f)阅读 200评论 11赞 22
    
-   我不是最好的，但我愿做最努力的！
    
    [![](https://upload.jianshu.io/users/upload_avatars/5017429/b2e55503-752f-482c-9673-363e02a1a076?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)浩圻妈](https://www.jianshu.com/u/72d099dcf3bd)阅读 139评论 0赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/5017429-97d92575add3bd4f.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/e2c5df3291fc)