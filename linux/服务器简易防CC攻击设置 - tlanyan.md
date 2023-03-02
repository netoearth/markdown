 [Linux](https://itlanyan.com/category/linux/) [安全](https://itlanyan.com/category/%e5%ae%89%e5%85%a8/)2020年5月27日

本文目录

-   [Nginx防CC设置](https://itlanyan.com/server-protect-cc-attack/#bnp_i_1)
-   [firewalld/iptables防CC设置](https://itlanyan.com/server-protect-cc-attack/#bnp_i_2)
-   [修改Nginx可打开的最大文件数](https://itlanyan.com/server-protect-cc-attack/#bnp_i_3)
-   [总结](https://itlanyan.com/server-protect-cc-attack/#bnp_i_4)
-   [参考](https://itlanyan.com/server-protect-cc-attack/#bnp_i_5)

本人服务器前段时间受到了DDos和CC攻击(参考 [本站近期发生的几起安全事件](https://itlanyan.com/site-recent-security-events/))，DDoS流量型攻击只能靠带宽来扛住，但CC攻击可以从服务器和应用层面防御和减轻影响。本文介绍受到攻击后，本人在服务器上采取的简易防CC攻击设置。

## Nginx防CC设置

不同于DDoS靠流量蛮力攻击，CC攻击模拟正常用户与服务器交互。CC攻击一般需找到网站/应用的薄弱处，然后通过大量连接/请求消耗服务器资源，让CPU、带宽能资源占用飙升。

### 并发请求和请求速率

**防护CC攻击的主要手段是限制并发请求数量和请求速率**。并发请求数量指的是单ip可以与服务器建立多少个连接，请求速率指的是单位时间内的请求数。两者有共同部分但意义不同，例如建立十个连接每个连接请求一次，也可以建立一个连接请求十次。前者是并发连接多，后者是请求速率高。

Nginx既支持并发请求的限制，也支持按限制请求速率。防御CC攻击主要是限制速率，理论上速率限制得好就够了。实践中建议两者结合，既防止太多连接消耗服务器资源，也防止请求速率过快。

### Nginx防CC设置步骤

[Nginx](https://itlanyan.com/tag/nginx) 防CC详细设置步骤如下：

1.  编辑Nginx配置文件，例如 `/etc/nginx/nginx.conf` ， 在 `http` 段分配计数内存：

```
limit_conn_zone $binary_remote_addr zone=limit_conn:10m;
limit_req_zone $binary_remote_addr zone=limit_req:10m rate=10r/s;
```

上述配置分配了 limit\_conn 和 limit\_req 两个10M大小的内存块（1M内存可记录16000个会话），并设置每秒最大请求速率是10次。

2， 打开网站配置文件，例如 `/etc/nginx/conf.d/tlanyan.conf`，在 `server` 或者 `location` 段中开启限制：

```
server {
    limit_conn limit_conn 5; # 并发连接数不超过5
    limit_req zone=limit_req burst=10 nodelay;
    # 其他设置
```

http中设置每秒允许10个请求，即100毫秒一个，如果突然来10个连接，后面9个直接返回503错误，这是我们不愿意看到的。limit\_req中的 `burst` 参数用来处理突发请求，此时10个请求都会被接受以应对突发流量。如果再来10个，那就超出了允许的突发限制，Nginx直接返回503错误。

`nodelay` 表示在允许突发请求的情况下，直接处理所有请求，而不是每100毫秒处理一个。

实践中请根据具体情况设置并发连接数和请求速率。数值过大会影响防御效果，太小则会影响正常用户使用。此外，建议总是启用 `burst` 和 `nodelay` 以应对突发流量。

3\. 保存配置文件，`nginx -t` 检查有无语法错误，然后 `systemctl reload nginx` 重新加载配置。

### 其他事项

1\. 可以将某些ip放入白名单，避免受到速率限制。操作是在 http 段新增开启变量：

```
geo $whitelist {
    default 1;
    10.0.0.17 0; # 按照这个格式添加白名单ip
}
map $whitelist $limit {
    0 "";
    1 $binary_remote_addr;
}

# limit_conn_zone 和 limit_req_zone更改为：
limit_conn_zone $limit zone=limit_conn:10m;
limit_req_zone  $limit zone=limit_req:10 rate=10r/s;

#其他配置无需改变
```

2\. `limit_conn` 和 `limit_req` 指令可以放置在 http、server、location 中使用，分别表示对所有站点、特点站点、特定url使用限制；

3\. 可以同时使用多个 `limit_conn` 和 `limit_req` 应付复杂场景。[WordPress](https://itlanyan.com/category/wordpress) 的搜索页面最消耗服务器资源，因此可以单独限制，例如每秒最多一次请求；

4\. 超过允许速率后，服务器直接返回503错误，自定义错误页请参考：[自定义Nginx错误页](https://itlanyan.com/nginx-custom-error-page/)。

## firewalld/iptables防CC设置

Nginx可以限制并发请求数量，防火墙（[iptables](https://itlanyan.com/tag/iptables) / [firewalld](https://itlanyan.com/tag/firewalld)）也可以做到，并且性能更好。

防火墙限制主要是两方面：

1\. 将某个ip拉黑：

```
#iptables
iptables -I INPUt -p tcp -s 黑名单ip -j DROP
# firewalld
firewall-cmd --add-rich-rule="rule family='ipv4' source address='黑名单ip' reject"
```

2\. 限制ip的并发连接：

```
# iptables
iptables -I INPUt -p tcp --dport 443 -m connlimit --connlimit-above 5 -j DROP
# firewalld
firewall-cmd --add-rich-rule 'rule port port=443 protocol=tcp accept limit value="5/s"'
```

## 修改Nginx可打开的最大文件数

默认Nginx最大可打开1024个文件/连接，在大流量时明显不够，因此需要加大。

操作方法如下：

1\. 编辑 `/etc/systemd/system/multi-user.target.wants/nginx.service`，添加文件数限制：

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
# 增加这一行
LimitNOFILE=65535
#其他不用变
```

2\. 重启Nginx：`systemctl daemon-reload; systemctl restart nginx`；

3\. 查看Nginx打开文件限制是否已经生效：`ps aux | grep nginx` 找到 nginx 的进程号，然后 `cat /proc/进程号/limits`，输出如下：

[![Nginx可打开最大文件数](https://itlanyan.com/wp-content/uploads/2020/04/Nginx%E5%8F%AF%E6%89%93%E5%BC%80%E6%9C%80%E5%A4%A7%E6%96%87%E4%BB%B6%E6%95%B0-1024x465.jpg)](https://itlanyan.com/server-protect-cc-attack/nginx%e5%8f%af%e6%89%93%e5%bc%80%e6%9c%80%e5%a4%a7%e6%96%87%e4%bb%b6%e6%95%b0/)

Nginx可打开最大文件数

## 总结

经过上述设置，配置好参数后，小型的CC攻击能防住或者减轻影响。大量ip/大流量的攻击没办法，只能通过WAF/大带宽做清洗处理。

## 参考

1.  [网站如何防CC攻击–巧用nginx](https://www.xiaoweigod.com/webserver/1545.html)
2.  [NGINX上的限流（译）](https://www.jianshu.com/p/2cf3d9609af3)
3.  [nginx防止DDOS攻击配置](http://www.52os.net/articles/nginx-anti-ddos-setting.html)
4.  [Add support to per-source-IP rate limiting](https://github.com/firewalld/firewalld/issues/70)

赞(4)