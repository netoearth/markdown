 [PHP](https://itlanyan.com/category/php/) [安全](https://itlanyan.com/category/%e5%ae%89%e5%85%a8/)

## 前言

[服务器被入侵挖矿](https://itlanyan.com/server-being-hacked-log/) 后，服务器和应用安全意识提高到了新的高度。近日排查早期应用代码，又意外发现了一个非常严重的远程执行漏洞，所幸没看到有被利用。

服务器被`getshell`意味着程序源码和数据的泄漏，是非常严重的安全事故。鉴于此，目前线上服务器执行的安全策略包括：

1.  必须开启防火墙，严格限制进出流量；
2.  不允许使用`root`权限运行应用；如果需要监听特权端口，工作进程必须`setuid/setgid`为普通或低权限账户；
3.  如果服务器运行多个应用，必须做好资源和权限隔离；
4.  数据和文件做好远程备份。

简单来说，就是：1. 尽量保证不会有服务器被入侵的情况发生；2. 万一出现了，不能获取系统控制权；3. 不要影响其他应用；4. 能尽快恢复系统。

资源和权限隔离的一个很好的实践是在容器中运行。但个人用户往往在单机上部署多个应用并且机器配置也不高，使用[docker](https://itlanyan.com/tag/docker/)隔离cpu和内存占用率可能直接飙上去，而且多个容器共享数据库实例的配置也略显复杂。

本文简要介绍服务器上运行多个 [PHP](https://itlanyan.com/category/php/) 应用时采用多个进程池隔离增强安全性。

## PHP-FPM使用多个进程池隔离增强安全性

单机上部署多个PHP应用是比较常见的场景，例如`www`、`api`和`admin`后台。默认情况下，都是用www这个PHP-FPM进程池，如果某个应用被入侵，则可访问其他应用的代码和数据，存在安全隐患。

幸运的是，FPM支持多个进程池，并能运行在不同的账号权限下，从而可实现资源隔离，增强安全性。

假设有A和B两个应用，下面介绍使用多个进程池进行隔离的操作步骤。

1\. 首先创建两个系统用户：

```
sudo useradd user1
sudo useradd user2
```

2\. 赋予用户对网站文件夹所有权：

```
# 假设A和B两个应用的存放位置是 /var/www/site1 和 /var/www/site2
sudo chown -R user1:user1 /var/www/site1
sudo chown -R user2:user2 /var/www/site2

# 禁止user1用户访问B应用的文件和数据，反之亦然
sudo chmod 770 /var/www/site1
sudo chmod 770 /var/www/site2
```

设置好文件夹权限后，万一A或者B应用被入侵，上述设置一定程度上保护了另外一个应用的安全。

此外，我们还须放行[Nginx](https://itlanyan.com/tag/nginx/)对文件夹的访问权限（假设Nginx的运行用户为nginx）：

```
sudo usermod -a -G site1 nginx
sudo usermod -a -G site2 nginx
```

> 注：也可以将site1和site2的用户组设置为nginx，就无需更改Nginx的用户组了：
> 
> ```
> sudo chown -R user1:nginx /var/www/site1
> sudo chown -R user2:nginx /var/www/site2
> ```

3\. 创建两个新的PHP-FPM进程池：

```
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm/site1.conf
sudo cp /etc/php-fpm.d/site1.conf /etc/php-fpm/site2.conf
```

4\. 配置进程池，分别使用user1和user2两个用户运行：

```
# 编辑配置文件
sudo vim /etc/php-fpm.d/site1.conf

# 修改进程池名称
[www] => [site1]
# 修改进程运行用户和组
user = apache => user = user1
group = apache => group => user1
# 修改监听地址
listen = /run/php-fpm/www.sock => listen = /run/php-fpm/site1.sock

# site2配置类似
```

5\. 修改Nginx的网站配置：

```
# site1的配置
server {
  server_name xxx.

  location ~ \.php$ {
    ...
    fastcgi_pass unix:/run/php-fpm/site1.sock
  }
}

# site2的配置类似
```

修改完成后，使用 `nginx -t` 测试有无语法错误。

6.  如果开启了Opcache，编辑 `10-opcache.ini` 文件，修改 `opcache.validate_permission` 和 `opcache.validate_root` 的值：

```
opcache.validate_permission=1
opcache.validate_root=1
```

7\. 重启Nginx和PHP-FPM：

```
sudo systemctl restart nginx php-fpm
```

使用 `ps` 命令查看PHP-FPM进程，应该能看到以不同用户在运行：

```
ps aux | grep php
```

## 总结

在多租户，多应用的情况下，对资源进行隔离能有效的保障系统安全。本文简要介绍了PHP-FPM使用多个进程池隔离增强安全性的操作步骤，如果条件允许，更建议使用docker进行资源隔离。

## 参考

1\. [Use PHP-FPM Pools to Secure Multiple Web Sites](https://www.vultr.com/docs/use-php-fpm-pools-to-secure-multiple-web-sites)

2\. [提供公开服务器请注意防范潜在风险](https://itlanyan.com/mind-potential-risk-with-public-services/)

3\. [使用Mysql主主同步/Redis主从复制备份网站](https://itlanyan.com/mysql-master-master-replication-to-backup-site/)

4\. [使用rsync daemon跨主机安全同步](https://itlanyan.com/use-rsync-daemon-to-sync-files-between-servers/)

5\. [Mitigating PHP’s long standing issue with OPCache leaking sensitive data](https://ma.ttias.be/mitigating-phps-long-standing-issue-opcache-leaking-sensitive-data/)

赞(4)