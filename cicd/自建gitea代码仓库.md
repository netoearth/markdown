> 最近受不了服务器网络被墙导致拉取github代码龟速的问题，准备自建gitea代码仓库自己做托管了。下面记录一下主要的流程以及踩坑过程。

## 前因

因为我的整套博客代码都是放在github上，推送代码的时候会触发webhooks的钩子触发drone的自动化构建部署。但是因为github的原因导致drone在拉取代码更新的时候经常需要几分钟而且是不是的会拉取失败，一怒一下决定换仓库了。

本来想迁移到gitee上，发现gitee官网的从github导入出bug了想加入官方群反映一下没人鸟那就算了，自己搭吧。刚好看到了gitea这个开源的代码仓库，同时drone也支持gitea的使用。

## 使用docker-compose进行单机部署

根据官网上的推荐，一个docker-compose文件搞定即可：

```
version: "3"
services:
  gitea:
    image: gitea/gitea:1.13.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=gitea_db:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    networks:
      - harbor_harbor
    volumes:
      # 映射gitea的data文件
      - /home/git/gitea/data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3456:3000"
      - "2222:22"
    depends_on:
      - gitea_db

  db:
    image: mysql
    container_name: gitea_db
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
      - TZ=Asia/Shanghai # 设置时区
    networks:
      - harbor_harbor
    volumes:
      - ./mysql:/var/lib/mysql
networks:
  harbor_harbor:
    external: true
```

### 可用到的环境变量

您可以通过环境变量配置 Gitea 的一些设置：

![alt](https://image.carrotwu.com/pic_image/20211109080227__6LK22IHA.png)

### 启动并访问gitea

启动容器，根据`[主机ip地址]:3456`访问运行的gitea，初次访问的时候会叫你初始化一些配置。

大部分配置都不用改，如果需要外网访问的话可以把`ssh服务域名`以及`gitea基本url`替换成你的外网域名。例如我的就是`gitea.carrotwu.com`和`https://gitea.carrotwu.com/`

![alt](https://image.carrotwu.com/pic_image/20211109080228__G7DOISOF.png)

gitea的部署就这样子了，接下来你就可以通过https的形式愉快的管理代码仓库呢！对了ssh方式呢？要支持ssh方式还需要一些额外的配置。

### gitea与主机共享22端口

> 接下来的内容来自于[Docker 安装 Gitea/Gogs 与主机共享 22 端口](https://ehlxr.me/2021/01/06/docker-gitea-share-port-22-with-host/)。这边做个备份防止源栈丢失找不到了。

如果主机的 22 端口已被使用，使用 Docker 安装 Gitea 时只能把容器的 22 端口映射到主机的其它端口（如：2222），这是没有任何问题的。但是以 SSH 方式 `clone` 项目时，`URL` 长这样`ssh://git@git.example.com:2222:username/project.git`

如果我们想要类似以下这样的 `URL` 时就需要把 `Gitea` 容器的和主机共享 22 端口 `git@git.example.com:username/project.git`

下面总结一下使用 Docker 安装 Gitea 共享主机 22 端口的主要步骤，Gogs 应该是同理。

#### 创建 git 用户

```
# Create git user
adduser git

# Make sure user has UID and GID 1000
usermod -u 1000 -g 1000 git

# Create docker group
groupadd docker

# Add git user to docker group
usermod -aG docker git

# Create the gitea data directory

mkdir -p /home/git/gitea/data
```

#### 安装gitea

```
# 作者注释 注意我上面用的是docker-compose的方式启动项目  下面的这条命docker run命令无视即可
# docker run -d --name=gitea -p 2222:22 -p 3456:3000 -v /home/git/gitea/data:/data --restart=always gitea/gitea:latest

# Create a symlink between the container authorized_keys and the host git user authorized_keys
ln -s /home/git/gitea/data/git/.ssh /home/git/
```

#### 生成 SSH key

```
sudo -u git ssh-keygen -t rsa -b 4096 -C "Gitea Host Key"

echo "no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty $(cat /home/git/.ssh/id_rsa.pub)" >> /home/git/.ssh/authorized_keys

chmod 600 /home/git/.ssh/authorized_keys
```

#### 配置 SSH passthrough

配置 `passthrough` 连接到 `Gitea` 容器的 SSH 映射端口 10022.

```
mkdir -p /app/gitea/

cat >/app/gitea/gitea <<'END'

ssh -p 10022 -o StrictHostKeyChecking=no git@127.0.0.1 \
"SSH_ORIGINAL_COMMAND=\"$SSH_ORIGINAL_COMMAND\" $0 $@"
END

chmod +x /app/gitea/gitea
```

#### 反向代理配置

作者注释：其实就是用nginx或者traefik把`gitea.carrotwu.com`域名代理到上面配置的`3456`端口而已。

### 注意事项

1.  由于 docker 启动容器的默认 uid 和 gid 是 1000，所以 git 用户的 uid、gid 必须为 1000，如果 git 用户的 uid 和 gid 不是 1000（比如：1002） 。就是在启动gitea容器的时候我们配置的`uid`和`gid`环境变量
    
2.  保证 git 用户下的所有文件都属于 git 用户和 git 组
    

```
[git]$ ls -la /home/git/gitea/data
total 20
drwxrwxr-x  5 git git 4096 Jan  5 14:14 .
drwxrwxr-x  3 git git 4096 Jan  5 13:56 ..
drwxr-xr-x  5 git git 4096 Jan  5 14:20 git
drwxr-xr-x 10 git git 4096 Jan  5 14:40 gitea
drwx------  2 git git 4096 Jan  5 14:14 ssh
```

### 测试ssh

使用ssh测试能否链接gitea代码库：

```
ssh -T  git@gitea.carrotwu.com

# 出现下面的话就证明ssh链接成功
Hi there, xxx! You've successfully authenticated with the key named xx@xx.com.com, but Gitea does not provide shell access.
If this is unexpected, please log in with password and setup Gitea under another user.

# 不成功的话 加上-v会开启debug调试
ssh -T -v git@gitea.carrotwu.com
```

### webhooks触发两次的bug

一切搞完之后，发现推送代码触发的webhooks竟然会莫名奇妙的同时触发两次。

![alt](https://image.carrotwu.com/pic_image/20211109080227__02RGCQBP.png)

找了好久，翻到了官方的[issues](https://github.com/go-gitea/gitea/issues/7702),看到有位大佬说重启gitea就莫名其妙就好了，然后试了一下果不其然。。。猜测可能是在启动完gitea容器之后改动配置导致出了点问题。。。

## 效果

![alt](https://image.carrotwu.com/pic_image/20211109080227__DFG80OE4.png)

终于不用忍受拉个github代码需要几分钟的痛苦了，后续gitea的部署抽时间准备也放到k8s集群上。