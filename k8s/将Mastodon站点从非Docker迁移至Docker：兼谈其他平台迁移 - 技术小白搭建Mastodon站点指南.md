一个站点运营久了总会遇到迁移的问题，比如要扩容、要更换服务商等等。关于如何在非docker系统之间迁移站点，官方文档中已经给出[详细步骤](https://docs.joinmastodon.org/zh-cn/admin/migrating/)，但对于如何从非docker迁移至docker系统、如何在docker系统中互相迁移，则尚无完整的教程。这篇将会对此详细描述。

本教程依旧完全依赖兔子（@star@b612.me）大佬的手把手指导，什么也不懂的记录员向ta鞠躬！

### 1\. 在新服务器中安装Mastodon

请依照[Docker安装Mastodon教程](https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html)，在新服务器中安装mastodon，**到“配置Mastodon”一步前停下。**

### 2\. 复制并修改旧配置文件

到旧服务器中，

```
   su - mastodon
   cd live
   rsync -azv ./.env.production root@新服务器IP:/home/mastodon/mastodon/
```

将`.env.production`复制入新服务器。

在新服务器中

```
   cd /home/mastodon/mastodon
   nano .env.production
```

修改`.env.production`文件，将下列条目修改成：

```
   DB_HOST=db
   REDIS_HOST=redis
   ES_HOST=es
```

保存退出。

此时，用`ls -a`命令查看`/home/mastodon/mastodon`文件夹，应该仅能看到你刚刚改好的`.env.production`，和你已经按照之前的安装教程改好的`docker-compose.yml`两个文件。

### 3\. 复制数据库

到旧服务器中：

关闭Mastodon服务：

```
   systemctl stop 'mastodon-*.service'
```

然后备份数据：

```
   su - mastodon
   cd live
   pg_dump -Fc mastodon_production -f backup.dump
```

生成`backup.dump`文件后

```
   rsync -azv ./backup.dump root@新服务器ip:/home/mastodon/mastodon/
```

复制到新服务器。

随后到新服务器中：

```
   cd /home/mastodon/mastodon
   docker-compose up -d db
   docker ps | grep mastodon_db | awk '{print $1}'
```

查看数据库容器`mastodon_db_1`的代码，然后

```
   docker cp ./backup.dump 刚查到的代码:/tmp/backup.dump
```

将数据库文件拷贝到docker内。

`cat .env.production`查看DB设置内容，如果你之前是DO一键注册，应该会看到：

```
   DB_NAME=mastodon_production
   DB_USER=mastodon
```

并且不设密码。建议添加一行，设置数据库密码：

进入docker db容器：

```
   docker exec -it  mastodon_db_1 /bin/bash
   su - postgres
   createdb -T template0 mastodon_production
```

建立新的空白数据库。

然后

进入数据库，依次输入以下几行，注意要包括分号：

```
   CREATE USER mastodon WITH PASSWORD '数据库密码';
   GRANT ALL PRIVILEGES ON DATABASE mastodon_production TO mastodon;
   \q
```

退出数据库，执行：

```
   pg_restore -U mastodon -n public --no-owner --role=mastodon \
     -d mastodon_production /tmp/backup.dump
```

然后两次`exit`退出docker容器。

### 4\. 复制媒体文件

如果你之前已经将媒体文件上传至云端，本步你可以跳过。

如果没有，则需将媒体文件复制到新服务器中。

在复制之前，建议在mastodon用户live文件夹中运行：

```
   RAILS_ENV=production bin/tootctl media remove
   RAILS_ENV=production bin/tootctl media remove-orphans
```

两步，移除远程媒体文件给整个文件夹瘦身。

由于所需传输数据量很大，为防止中途断网等意外情况，建议安装screen软件进行后台运行。

在root用户下：

然后

第一次运行会让你看简介，两次空格按到底后：

```
   rsync -azv ~/live/public/system/ root@新服务器ip:/home/mastodon/mastodon/public/system/
```

开始传输后，`ctrl+A+D`将任务转到后台。这样你断网了也不用担心。

如果要查看任务进度，`screen -ls`查看任务列表前面的数字，然后`screen -r 具体数字`查看。

### 5\. 继续配置

回到新服务器中：

```
   cd /home/mastodon/mastodon
   docker-compose down
   docker-compose run --rm web bin/tootctl feeds build              #构建用户首页时间流
   docker-compose up -d
```

如果你在新服务器上的版本高于原先版本，需要根据官方升级指导增加步骤，可能需要进行

```
   docker-compose run --rm web rails db:migrate
```

等步骤。

### 6\. 配置nginx

新服务器中按照[Docker安装Mastodon教程](https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html#6-%E5%AE%89%E8%A3%85%E5%B9%B6%E9%85%8D%E7%BD%AEnginx)配置nginx。

或者可以直接将旧服务器中位于`/etc/nginx/sites-available/`的文件复制到新服务器中，然后按照前述教程进行镜像投射，再用certbot申请新的证书。

注意，如果是直接复制，那么在安装certbot之前，需要将文件中ssl\_certificate和ssl\_certificate\_key部分先加#号注释掉，一会儿由certbot自动改回。

重启nginx。

### 7\. 赋权

这一步是Docker独有步骤。`/public/system/`文件夹传输完毕后，在新服务器：

```
   cd /home/mastodon/mastodon
   chown 991:991 -R ./public
   chown -R 70:70 ./postgres
```

如果启动了全文搜索，则需

```
   chown 1000:1000 -R ./elasticsearch
```

重启

```
   docker-compose down
   docker-compose up -d
```

随后重新建立elasticsearch索引。 　　

现在你的站点就迁移完毕啦！

## 将站点从Docker迁移至Docker

步骤和上文基本类似，但在第3步转移数据库时，仅需要将相应文件夹中的postgres和redis文件夹传输过去之后，对新服务器中的postgres文件夹赋权即可：

```
chown -R 70:70 ./postgres14
```

不需要使用pg\_dump和pg\_restore命令。

后续其他步骤同上。

## 将站点从Docker迁移至非Docker

请参考[How to migrate a dockerized Mastodon to a non-dockerized Mastodon instance](https://markus-blog.de/index.php/2019/10/02/how-to-migrate-a-dockerized-mastodon-to-a-non-dockerized-mastodon-instance/)。基本原理类似。

## 将站点从非Docker迁移至非Docker

请参考[官方迁移教程](https://docs.joinmastodon.org/zh-cn/admin/migrating/)。中间可能让人看不太懂的的只有rsync步骤，其实基本命令就是在旧服务器上：

```
rsync -azv 旧文件路径 root@新服务器ip:新文件路径
```

即可。

## 总结

总体而言，熟练掌握迁移技能，不仅能应用在服务器更换扩容，也能用在通过备份恢复数据上，是非常值得大家（以及动不动就要炸站重启的新手）掌握的技能。希望这篇教程能给大家带来帮助！

___