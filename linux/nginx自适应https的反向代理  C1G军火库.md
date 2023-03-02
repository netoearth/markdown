## 方式一,写死地址,用变量

## 最简洁

map $scheme $online\_proxy\_www {  
default 39.156.66.10;  
}

proxy\_pass $scheme://$online\_proxy\_www:$server\_port;

## 方式二,写两个upstream,再用proxy\_pass覆盖法

## 缺点需要维护2个upsteam

upstream online\_proxy\_www {  
server 39.156.66.10:80;  
}  
upstream online\_proxy\_www\_https {  
server 39.156.66.10:443;  
}

proxy\_pass $scheme://online\_proxy\_www;

## 自适应https

if ( $scheme = https) {  
proxy\_pass $scheme://online\_proxy\_www\_https;  
}

## 方式三,upstream backup法

## 最简单,缺点会多一次请求,多个错误日志

upstream online\_proxy\_www {  
server 39.156.66.10:80;39.156.66.10  
server 39.156.66.10:443 backup;  
}  
proxy\_pass $scheme://online\_proxy\_www;

\==========================

nginx.conf示例

```
upstream online_proxy_www {
    server   39.156.66.10:80;
    #server   39.156.66.10:443 backup;
}
upstream online_proxy_www_https {
    server   39.156.66.10:443;
}


server
{
    listen       80;
    listen       443 ssl;
    server_name  blog.c1gstudio.com;
    index index.html index.htm index.php;
    root  /opt/htdocs/www;
    access_log  /var/log/nginx/blog.c1gstudio.com.log  access ;

    include ssl.conf;

    location /
    {
        proxy_set_header Host  $host;
        proxy_set_header X-Forwarded-For $proxypass_forwarded_for;
        proxy_pass $scheme://$online_proxy_www:$server_port;

        add_header      X-Cache   C1GPROXY1;
    }


}
```

Posted in [Nginx](http://blog.c1gstudio.com/archives/category/technology/nginx-technology).

Tagged with [nginx](http://blog.c1gstudio.com/archives/tag/nginx), [反向代理](http://blog.c1gstudio.com/archives/tag/%e5%8f%8d%e5%90%91%e4%bb%a3%e7%90%86).

– 2023/02/28