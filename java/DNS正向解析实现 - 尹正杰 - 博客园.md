　　　　　　　　　　　　　**DNS正向解析实现**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰** 

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.资源记录**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　区域解析库是由众多资源记录(Resource Record，简称:"RR")组成。资源记录由 A，AAAA,，PTR,，SOA，NS，CNAME，MX，TXT，SPT等记录类型组成。　　接下来我们一起来介绍一下常用的记录类型含义:　　　　SOA:　　　　　　全称为: "Start Of Authority"，即起始授权记录。　　　　　　一个区域解析库有且仅能有一条SOA记录，必须位于解析库的第一条记录。　　　　A：　　　　　　Internet Address，作用是将FQDN解析为IPv4地址。　　　　AAAA:　　　　　　作用是将FQDN解析为IPv6地址。　　　　PIR:　　　　　　PoinTeR,反向解析记录，将IP地址解析为FQDN。　　　　NS:　　　　　　全称为:Name Server，专用于标明当前区域的DNS服务器。　　　　CNAME:　　　　　　Canonical Name，别名记录。　　　　MX:　　　　　　Mail eXchanger，邮件服务器。　　　　TXT:　　　　　　对域名进行标识和说明的一种方式，一般做验证记录时会使用此项，如SPF(反垃圾邮件)记录，https验证等。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**1>.资源记录定义的格式**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　无论你想定义的资源记录定义类型是哪个，他都遵循以下定义的格式
　　　　语法:　　　　　　name [TTL] IN rr_type value
　　　　解析说明:　　　　　　name:　　　　　　　　定义资源记录的名称。　　　　　　TTL:　　　　　　　　资源记录的生命周期，即记录缓存的时长，可以不写，但需要在全局定义，比如: "$TTL 1D"(也可以写成"$TTL 86400"，不写单位的话默认单位就是秒)，表示生命周期是1天。　　　　　　IN:　　　　　　　　代表Internet记录的意思，早起DNS还有别的功能，目前来讲DNS主要是做Inernet互联网解析记录的，因此当我们添加记录时直接照抄IN即可。　　　　　　rr_type:　　　　　　　　代表记录的类型，上面有提到过，比如A,AAAA,PIR等等记录类型。　　　　　　　　value:　　　　　　　　前面的name相当于key，而这个value就是用来记录值的。　　　　　　　　举个例子，对于A记录来讲，这个name就是FQDN的全称，而value就是IPv4的地址。同理，对于AAAA记录来讲，这个name就是FQDN的全称，而valpue就是IPv6的地址。
　　温馨提示:　　　　(1)TTL可从全局继承;　　　　(2)"@"符号可用于引用当前区域的名字;　　　　(3)同一个名字可以通过多条记录定义多个不同的值，此时DNS服务器会以轮询方式响应;　　　　(4)同一个值也可能有多个不同的定义名字，通过多个不同的名字指向同一个值进行定义，此仅表示通过多个不同名字可以找到同一个主机。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.**

**3>**

**二.配置"yinzhengjie.com"域名的区域解析记录**

**1>.修改主配置**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@dns53.yinzhengjie.com ~]# vim /etc/named.rfc1912.zones
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# egrep -v "^//|^$" /etc/named.rfc1912.zones  # 修改bind程序的自配置文件，添加我们要维护的域名信息。
...
zone "yinzhengjie.com" IN {
    type master;
    file "yinzhengjie.com.zone";
    allow-update { none; };
};
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2020.cnblogs.com/blog/795254/202012/795254-20201231222407439-1890330911.png)

**2>.配置区域解析数据库文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@dns53.yinzhengjie.com ~]# vim /var/named/yinzhengjie.com.zone  # 温馨提示，当前的用户是root，而运行bind服务的用户名是named哟~
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# cat /var/named/yinzhengjie.com.zone
$TTL 1D
@    IN SOA    @ rname.invalid. (
                    0    ; serial
                    1D    ; refresh
                    1H    ; retry
                    1W    ; expire
                    3H )    ; minimum
                NS    @
                A    127.0.0.1
        AAAA    ::1
docker201 300 IN A 172.200.1.201
dns53.yinzhengjie.com. IN A 172.200.1.53
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# chown root:named /var/named/yinzhengjie.com.zone  # 此步骤千万别忘记，否则named用户将无权限访问root创建的文件哟~
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# named-checkzone yinzhengjie.com /var/named/yinzhengjie.com.zone  # 验证区域文件的语法是否正确！
zone yinzhengjie.com/IN: loaded serial 0
OK
[root@dns53.yinzhengjie.com ~]# 
[root@dns53.yinzhengjie.com ~]# rndc reload  # 重新加载配置文件！无需重启named服务
server reload successful
[root@dns53.yinzhengjie.com ~]# 



温馨提示:
　　(1)上面的记录请主管朱最后两条记录，其他的照抄DNS已有的本地解析文件的模板即可;
　　(2)我们可以指定TTL的值，若不指定单位，默认为秒，上面为为docker201这个主机指定的TTL为300秒，但我并未dns53主机指定TTL，因此它使用配置文件首行定义的变量"$TTL"，即默认的TTL为1天;
　　(3)通常情况下，我们无需再第一列写上完整的FQDN，如果要写，请一定要将根域(就是最后一个点号)写上，否则bind程序会误以为你写的是主机名，后自动在给你补一个当前域名(本案是"yinzhengjie.com)：
　　　　　　a)以上面的案例为例，当我写的第一列是"docker201"是，则bind程序会隐式为我们补上域名，即:"docker.yinzhengjie.com.";
　　　　　　b)如果第一列写的是"docker201.yinzhengjie.com"，注意哈，此时我没有写最后一个点号(也就是我们所说的根域)，因此bind程序依旧会为我们补上域名，即:"docker201.yinzhengjie.com.yinzhecngjie.com."，很显然不是我们想要的结果;
　　　　　　c)如果第一列写的是"docker201.yinzhengjie.com."，注意哈，此时我写了最后一个点号(也就是此处我写了根域)，此时bind程序发现竟然是以"yinchengjie.com."结尾的，因此就不会在去补充一次啦。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2020.cnblogs.com/blog/795254/202012/795254-20201231230751605-1676430302.png)

**3>.验证解析结果**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.com ~]# dig  dns53.yinzhengjie.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 <<>> dns53.yinzhengjie.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44863
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;dns53.yinzhengjie.com.        IN    A

;; ANSWER SECTION:
dns53.yinzhengjie.com.    86400    IN    A    172.200.1.53

;; AUTHORITY SECTION:
yinzhengjie.com.    86400    IN    NS    yinzhengjie.com.

;; ADDITIONAL SECTION:
yinzhengjie.com.    86400    IN    A    127.0.0.1
yinzhengjie.com.    86400    IN    AAAA    ::1

;; Query time: 0 msec
;; SERVER: 172.200.1.53#53(172.200.1.53)
;; WHEN: 四 12月 31 23:49:45 CST 2020
;; MSG SIZE  rcvd: 124

[root@docker201.yinzhengjie.com ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2020.cnblogs.com/blog/795254/202012/795254-20201231235338419-2100755427.png)

**三.**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186