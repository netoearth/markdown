   

[![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/leancloud-assets/5de5a40ba9bf72c2805a.jpg~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3650034331813288)

2022年07月26日 19:34 ·  阅读 39

在这个例子中，我们将运行 composer 来安装应用程序的依赖性，然后将`vendor` 文件夹和其余的应用程序文件复制到容器镜像中。通过这种方式，我们的镜像将总是默认带有应用程序，所以我们所要做的就是创建一个容器并开始使用它。这是一个众所周知的做法，但我只想谈谈一个基本的实现。其主要目的是隔离应用程序代码，不将其暴露在主机操作系统上。我不打算在这里列举这些好处，因为它们都已经在互联网上写过了。这个解决方案的主要问题是，最终的docker镜像的大小将取决于应用程序和`vendor` 文件夹的大小。

### 结构

```
.
├── composer.json
├── docker
│   ├── docker-compose.yml
│   ├── Makefile
│   └── php
│       ├── Dockerfile
│       ├── php.ini
│       └── www.conf
├── .dockerignore
├── .gitignore
└── src
    └── index.php
复制代码
```

### 文件

我只是展示了重要文件的内容。其余的就不太重要了。

#### Makefile

```
build:
@docker-compose build

up:
@docker-compose up -d
复制代码
```

#### Docker文件

```
#
# STAGE 1: composer
#
FROM composer:1.8.5 as composer

# Copy composer files from project root into composer container's working dir
COPY composer.* /app/

# Run composer to build dependencies in vendor folder
RUN set -xe \
 && composer install --no-dev --no-scripts --no-suggest --no-interaction --prefer-dist --optimize-autoloader

# Copy everything from project root into composer container's working dir
COPY . /app

# Generated optimized autoload files containing all classes from vendor folder and project itself
RUN composer dump-autoload --no-dev --optimize --classmap-authoritative

#
# STAGE 2: php
#
FROM php:7.2.13-fpm-alpine3.8

# Set container's working dir
WORKDIR /app

# Copy everything from project root into php container's working dir
COPY . /app
# Copy vendor folder from composer container into php container
COPY --from=composer /app/vendor /app/vendor

# Copy necessary files
COPY docker/php/php.ini /usr/local/etc/php/conf.d/php.override.ini
COPY docker/php/www.conf /usr/local/etc/php-fpm.d/www.conf

CMD ["php-fpm", "--nodaemonize"]
复制代码
```

#### .dockerignore

```
.dockerignore
.gitignore
*.md
.git/
.idea/
.DS_Store/
复制代码
```

#### docker-compose.yaml

这是可选的，但如果你想使用它，我就把它留下。如果你不想像我一样使用本地的docker命令，你可以在项目根目录下使用`$ make -sC docker/ build up` 命令。由你自己决定。

```
version: "3.4"

services:

  php:
    build:
      context: ".."
      dockerfile: "docker/php/Dockerfile"
    hostname: "php"
复制代码
```

### 构建

#### 构建镜像

```
$ docker build -t my_php_app:latest -f /absolute/path/to/project/docker/php/Dockerfile .

Successfully built c62632b82433
Successfully tagged my_php_app:latest
复制代码
```

```
$ docker images
REPOSITORY        TAG                        IMAGE ID            CREATED             SIZE
my_php_app        latest                     c62632b82433        5 minutes ago       77.7MB
复制代码
```

#### 创建容器

```
$ docker run -itd --name my_php_app my_php_app:latest sh
4ec090d70437cf3312d735f754bcb5abcd96759fd8f76f24fbeb43e42c04f140
复制代码
```

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4ec090d70437        my_php_app:latest   "docker-php-entrypoi…"   4 seconds ago       Up 3 seconds        9000/tcp            my_php_app
复制代码
```

#### 验证容器内容

```
$ docker exec -it my_php_app sh
/app # ls -l
-rw-r--r--    1 root     root           133 Jun  3 20:23 composer.json
drwxr-xr-x    4 root     root          4096 Jun  3 20:33 docker
drwxr-xr-x    2 root     root          4096 Jun  3 20:39 src
drwxr-xr-x    4 root     root          4096 Jun  3 20:56 vendor
复制代码
```

如果你在容器内运行`php src/index.php` 命令，它应该可以工作。

相关小册

![「Git 原理详解及实用指南」封面](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/11/27/15ffbb05174a57f8~tplv-t2oaga2asx-no-mark:420:420:300:420.awebp)

![「Neovim 配置实战：从0到1打造自己的IDE」封面](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37d63d614f1b4b8dbd3b34a5c5d721d9~tplv-k3u1fbpfcp-no-mark:420:420:300:420.awebp?)

VIP

Neovim 配置实战：从0到1打造自己的IDE

[nshen ![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-2.99ba5b2.png "创作等级")](https://juejin.cn/user/1873223543167326)   

2031购买

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 赞

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏