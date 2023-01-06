携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第31天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 利用kibana的快照存储库备份es索引

@\[TOC\]

## 1.快照存储库备份es索引

es的索引库也是需要备份的，我们可以通过kibana上的快照存储库，给索引每天创建快照，定时备份

注意：使用nfs作为索引备份存储路径时，一定要为存储路径的属组和属主全部改成elasticsearch，否则会报没权限

但是修改属主属组有个问题，有的es集群的elasticsearch用户的uid和其他的会不同，这时就导致在nfs上改了属主属组但是在某个es节点不识别这个uid，这也会导致没权限，最好的解决办法就是在es安装的时候首先创建elasticsearch用户并且制定uid号，这样就能保证有权限使用nfs存储了

针对es索引的备份，可以创建一个快照，快照就相当于一个备份，这个快照里面包含了所有的索引，可以借助kibana上的快照存储库，定时的进行索引备份

es索引备份恢复大致思路：首先针对要备份的索引创建一个快照01，然后在往索引里面添加一些数据，再创建一个快照02，然后还原01查看效果，在还原02查看效果

kibana快照存储备份的好处就是：备份全部索引，每次备份占用的资源特别小，且备份时间在几秒左右

**环境准备**

| IP | 服务 |
| --- | --- |
| 192.168.16.106 | es |
| 192.168.16.105 | es+kibana+nfs |

## 2.部署nfs存储并在es节点进行挂载

### 2.1.部署nfs存储

```
[root@kibana ~]# yum -y install nfs-utils

[root@kibana ~]# vim /etc/exports
/data/es_backup 192.168.16.0/24(rw,sync,no_root_squash)
[root@kibana ~]# systemctl restart nfs
[root@kibana ~]# mkdir /data/es_backup

[root@kibana ~/soft]# showmount -e
Export list for kibana:
/data/es_backup 192.168.16.0/24
复制代码
```

### 2.2.配置es集群各节点增加nfs存储配置

所有节点都配置

```
1.挂载nfs
[root@elasticsearch ~]# mkdir /data/es_backup
[root@elasticsearch ~]# mount 192.168.16.105:/data/es_backup/ /data/es_backup

2.配置es指定存储路径
[root@elasticsearch ~]# vim /etc/elasticsearch/elasticsearch.yml 
path.repo: /data/es_backup
[root@elasticsearch ~]# systemctl restart elasticsearch

3.授权，在nfs上操作
[root@kibana ~]# chmod 777 /data/es_backup/
[root@kibana ~]# chown -R elasticsearch.elasticsearch /data/es_backup/

如果nfs没有elasticsearch用户需要创建出来，并且uid和gid号要和es每个集群都保持一致
复制代码
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c4c8b3e667b44dc9d410a53584002e6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 3.在kibana界面创建快照存储库

### 3.1.点击Managerment---快照存储库---注册存储库

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccce1bea9b0e458386dc63441ccc58d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 3.2.填写存储库名称，存储库类型选择共享文件系统

名称：es索引备份

存储类型选择共享文件系统

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a645b309f6474ef5bbb515ac3628c14d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 3.3.填写存储库信息

位置（共享存储路径）：/data/es\_backup

块大小：100mb，根据实际来填写

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6670d4764374f2b9a07935a87e95313~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 3.4.点击注册之后点击验证存储库

点击验证存储库

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a8cd94719724cb897e88516d15cc19c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 3.5.验证存储库成功

点击右侧的验证存储库，状态是已连接表示存储库可以正常使用了，如果状态是未连接很有可能是nfs存储权限的问题

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c75eeff2e05f46b78ead6506f2c3a9ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 4.es索引库备份

### 4.1.创建linuxbook索引库并插入数据

创建一个linuxbook索引库，用于还原前后的效果对比

```
[root@elaticsearch ~]# curl -XPOST '127.0.0.1:9200/linuxbook/book?pretty' -H 'Content-Type: application/json' -d '{
"id": 1,
"book_name": "nginx",
"book_jg": "35￥",
"book_ys": "206",
"book_group": "web"
}'
复制代码
```

插入5条数据数据，包含创建索引时的一条

```
[root@elasticsearch ~]#  curl -XPOST '127.0.0.1:9200/linuxbook/book?pretty' -H 'Content-Type: application/json' -d '{
"id": 2,
"book_name": "ansible",
"book_jg": "20￥",
"book_ys": "300",
"book_group": "zdh"
}'

[root@elasticsearch ~]#  curl -XPOST '127.0.0.1:9200/linuxbook/book?pretty' -H 'Content-Type: application/json' -d '{
"id": 3,
"book_name": "shell",
"book_jg": "20￥",
"book_ys": "3110",
"book_group": "shell"
}'
[root@elasticsearch ~]#  curl -XPOST '127.0.0.1:9200/linuxbook/book?pretty' -H 'Content-Type: application/json' -d '{
"id": 4,
"book_name": "tomcat",
"book_jg": "20￥",
"book_ys": "1024",
"book_group": "web"
}'
[root@elasticsearch ~]#  curl -XPOST '127.0.0.1:9200/linuxbook/book?pretty' -H 'Content-Type: application/json' -d '{
"id": 5,
"book_name": "python",
"book_jg": "20￥",
"book_ys": "11024",
"book_group": "python"
}'
复制代码
```

### 4.2.在es上查看新建索引的数据

linuxbook索引已经创建，数据插入成功

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23478f082c0e4306a1f32d80a913f429~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 4.3.创建一个快照01

一个快照里面包含所有的索引数据，可以每天定时制作一个快照，这就是es的备份方式

创建快照的命令只能在kibana上执行，在交互式使用curl命令，会报错，因为我们的存储库是中文的

```
PUT /_snapshot/es索引备份/es-backup-1-2021.1.26?wait_for_completion=true

解释：
PUT提交方式
/_snapshot快照目录
es索引备份   存储库
es-backup-1-2021.1.26快照名称
wait_for_completion参数指定是在初始化快照（默认）后立即返回请求还是等待快照完成
整体来说就是在es索引备份存储库中创建一个es-backup-2021.1.26快照信息

curl命令
curl -XPUT '127.0.0.1:9200/_snapshot/es索引备份/es-backup-1-2021.1.26?wait_for_completion=true'
复制代码
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cb548e3220447188a0f897fbb0df6dc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

创建完快照点击Managerment---快照存储库---快照 就可以看到我们创建的快照信息了 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5545396cbb154be5800a9ae824d506ed~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 4.4.查看快照信息

```
GET /_snapshot/es索引备份/es-backup-1-2021.1.26
复制代码
```

可以看到有6个分片，那是因为有6个索引，一个索引一个分片

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/836d0d05c8f14e1ebfc99815bd6bcf50~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**点击快照名也可以看快照的详细信息**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c01059f51f824cf18d05143b743de9e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**服务器上生成的快照文件**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/378e3a223cee4a378a7f945fdf7a336a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 5.在linuxbook索引中增加数据

### 5.1.在索引库中增加数据

在kibana上增加就可以了

```
POST /linuxbook/book?pretty
{
"id": 6,
"book_name": "linux",
"book_jg": "50￥",
"book_ys": "389",
"book_group": "system"
}
POST /linuxbook/book?pretty
{
"id": 7,
"book_name": "windows",
"book_jg": "50￥",
"book_ys": "789",
"book_group": "system"
}
POST /linuxbook/book?pretty
{
"id": 8,
"book_name": "jenkins",
"book_jg": "70￥",
"book_ys": "287",
"book_group": "jenkins"
}
POST /linuxbook/book?pretty
{
"id": 9,
"book_name": "elk",
"book_jg": "99￥",
"book_ys": "10000",
"book_group": "elk"
}
POST /linuxbook/book?pretty
{
"id": 10,
"book_name": "k8s",
"book_jg": "100￥",
"book_ys": "10000",
"book_group": "k8s"
}
POST /linuxbook/book?pretty
{
"id": 11,
"book_name": "prometheus",
"book_jg": "100￥",
"book_ys": "10000",
"book_group": "prometheus"
}
复制代码
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38d8c51b2ff34948910b7f0ae620d3b6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**查看es上是否新增了数据**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df77ae3e3dcc422e91b2e9b8f88d525b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 5.2.再创建一个快照02

这个快照里面包含了刚刚插入的6条数据

```
PUT /_snapshot/es索引备份/es-backup-2-2021.1.26?wait_for_completion=true
复制代码
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/089f3cafdbc642ebab878ea94b6ec6c9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 6.将索引还原出第一个快照的状态

es索引的还原可以选择不覆盖现有的索引，可以相当于将原来的索引复制一份出来

### 6.1.还原es-backup-1-2021.1.26快照中的linuxbook索引

es-backup-1-2021.1.26这个快照中的linuxbook索引只有5条数据，我们还原一下，查看是否正确

找到要还原的快照，点击类似于下载的按钮，可以看到提示的是还原

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c76c6b959d740559fe92376cd641590~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

勾选取消所有索引，包括系统所有的按钮，如果这个勾选了，那么该快照中的所有索引都会被还原

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a7d26a7c23947a681f34ebf5e5da07a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

只选择我们要还原的linuxbook索引，选择后下面会提示将还原1个索引

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4292894279d42349372677ce4a7e7d1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

勾选重命名索引，将还原的索引库还原到es时改个名字，避免数据覆盖

将linuxbook索引重命名为linuxbook\_huanyuan\_20200126

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/668a02ce38d840c08588545bc567e61d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

继续点击下一步

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71a36b0f5ff14018950b7a9396f0da1c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

摘要信息确认无误后点击还原快照

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6ae12069abe40dc8f59704ea28eb37f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

还原结束后会在还原状态那边看到是否成功

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/905172e413b44725971917ab39028583~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 6.2.查看es上是否有还原的索引库

已经有我们还原的linuxbook索引并且也已经重命名了

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3665ec6f0c5842e3a70509cfe16cdf64~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 6.3.查看数据是否是快照01时的5条数据

数据没问题，还原成功

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27f6693c97fa42b2954d22ba74de33b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 7.还原第二个快照

第二个快照还原方式和第一个快照还原一模一样，只需要修改一下还原的索引名即可

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/150d1c096e1b4fd697564411e6e07009~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

还原成功

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08935502ba104be2a95fee524137edbe~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

在es上查看数据，数据11条，完整无误

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d575ecb5791d4e47aa28f22a13ee5249~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 8.定时创建索引快照形成es索引库备份

### 8.1.通过shell脚本实现

```
1.编写脚本[root@elasticsearch /data/es_backup]# vim es_index_bak.sh
#!/bin/bashday=`date +%Y.%m.%d`
curl -XPUT "http://192.168.75.21:9200/_snapshot/es索引备份/es-index-backup-${day}"

2.赋权
[root@elasticsearch /data/es_backup]# chmod a+x es_index_bak.sh

3.制作定时任务，每天的零点30分进行备份
[root@elasticsearch /data/es_backup]# crontab -e
30 0 * * * /data/es_backup/es_index_bak.sh
复制代码
```

### 8.2.通过快照存储库策略实现

#### 8.2.1.创建策略

点击Managerment---快照存储库---策略---创建策略

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8682a09388584ee99f8b6fca1baf8313~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

填写策略信息

名称：es索引每日备份

快照名称：<es-backup-{now/d}> 官网配置：[www.elastic.co/guide/en/el…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fen%2Felasticsearch%2Freference%2F7.6%2Fdate-math-index-names.html "https://www.elastic.co/guide/en/elasticsearch/reference/7.6/date-math-index-names.html")

存储库选择es索引备份，频率选择day，时间选择03:30

> 这样生成的下一次快照时间就是每天上午的11:30进行备份，设置11:30主要为了测试，一般还是要在凌晨进行备份，具体为什么选择03:30对应11:30，差了8个小时，这里也不是很清楚，估计是时区的问题，我是上午10点24创建的，时间写03:30就对应了下午的11:30，可以来回调整，直到根据自己的合理匹配了就可以

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53999241e2d54a2b872f71fb8b47d24f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

快照设置保持默认即可

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be90f474498642c3aebd74e8404e0588~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

设置快照的保留天数，这里保留30天

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd1722371db7462f921ab3215167b5ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

信息复查没问题后点击创建策略 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00572cd471b3434dbcb46558255a50b6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 8.2.2.查看策略信息

只要下次快照的时间是我们理想中的即可

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/961fdf35dbdb4d02b55ed84ee5079e58~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

快照策略也可以立即运行、编辑、删除

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a177d52dbc745db8dfbe99ce763896a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 8.3.查看快照是否定时创建

备份已经自动创建

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc6b4049e7f74669865c0d2ae09f2aaf~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

还原linuxbook索引，验证备份是否可用

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e01b21245e60498085fdcb7c4d5e375b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

查看es索引数据

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e74d44327cd4f148ad82947bc43a415~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)