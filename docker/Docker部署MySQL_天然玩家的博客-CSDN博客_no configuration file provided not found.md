## 1.1 docker-compose.yml

使用Docker Compose编排MySQL，如下：

```
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql-5.7
    privileged: true
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_USER: xindaqi
      MYSQL_PASS: 123456
      TZ: Asia/Shanghai
    command:
      --wait_timeout=28800
      --interactive_timeout=28800
      --max_connections=1000
      --default-authentication-plugin=mysql_native_password
    volumes:
      - "/home/xindaqi/mysql-5.7/data:/var/lib/mysql"
      - "/home/xindaqi/mysql-5.7/config/my.cnf:/etc/mysql/my.cnf"
```

## 1.2 参数解析

| 序号 | 参数 | 描述 |
| --- | --- | --- |
| 1 | image | MySQL对应版本的镜像 |
| 2 | container\_name | 容器名称 |
| 3 | privileged | 授权 |
| 4 | restart | 重启MySQL容器，always，启动失败时一直尝试重启 |
| 5 | ports | MySQL运行端口，格式：host-port:docker-port |
| 6 | environment | MySQL环境，包括用户名和密码 |
| 7 | command | 连接超时配置及授权配置 |
| 8 | volumes | Docker容器中的MySQL数据挂载到宿主机路径，格式：host-path:docker-path |
| 9 | /var/lib/mysql | 数据库文件路径 |
| 10 | /etc/mysql/my.cnf | 数据库配置文件路径 |

## 2 MySQL配置

在[挂载](https://so.csdn.net/so/search?q=%E6%8C%82%E8%BD%BD&spm=1001.2101.3001.7020)MySQL配置文件的目录下添加配置文件  
/home/xindaqi/mysql-5.7/config/my.cnf：

```
[client]
# 客户端来源数据的默认字符集
default-character-set=utf8mb4

[mysqld]
# 服务端默认字符集
character-set-server=utf8mb4
# 连接层默认字符集
collation-server=utf8mb4_unicode_ci

[mysql]
# 数据库默认字符集
default-character-set=utf8mb4
```

## 3 启停

-   前台启动

```
docker-compose -f docker-compose.yml up 
```

-   守护进程启动

```
docker-compose -f docker-compose.yml up -d
```

-   停止

```
docker-compse -f docker-compose.yml stop
```

-   停止并删除容器

```
docker-compose -f docker-compose.yml down
```

## 4 连接MySQL

## 4.1 MySQL控制台

```
# 进入docker-compse.yml文件夹执行命令
docker-compose exec mysql bash
mysql -u root -p123456
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7a077af86f4e4952bda2787a49813e53.png#pic_center)

## 4.2 Workbench连接

![在这里插入图片描述](https://img-blog.csdnimg.cn/1a491265aed04f57a5441f3c91871af1.png#pic_center)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0822820bffe46819bf05d7a8f253b81.png#pic_center)

## 5 Q&A

## 5.1 启动异常

-   描述  
    mysql-5.7 | 2022-06-11 10:24:17+08:00 \[Note\] \[Entrypoint\]: Switching to dedicated user ‘mysql’  
    mysql-5.7 | 2022-06-11 10:24:17+08:00 \[Note\] \[Entrypoint\]: Entrypoint script for MySQL Server 5.7.38-1debian10 started.  
    mysql-5.7 | 2022-06-11 10:24:17+08:00 \[ERROR\] \[Entrypoint\]: MYSQL\_USER=“root”, MYSQL\_USER and MYSQL\_PASSWORD are for configuring a regular user and cannot be used for the root user  
    mysql-5.7 | Remove MYSQL\_USER=“root” and use one of the following to control the root user password:  
    mysql-5.7 | - MYSQL\_ROOT\_PASSWORD  
    mysql-5.7 | - MYSQL\_ALLOW\_EMPTY\_PASSWORD  
    mysql-5.7 | - MYSQL\_RANDOM\_ROOT\_PASSWORD
    
-   原因  
    MYSQL\_USER参数使用root
    
-   方案  
    MYSQL\_USER使用其他用户，如xindaqi
    

## 5.2 启动异常

-   描述  
    Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting “/home/xindaqi/install/mysql/config/my.cnf” to rootfs at “/etc/mysql/my.cnf”: mount /home/xindaqi/install/mysql/config/my.cnf:/etc/mysql/my.cnf (via /proc/self/fd/6), flags: 0x5000: not a directory: unknown: Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type
-   原因  
    文件挂载失败，没有在宿主机新建配置文件：my.cnf，如上
-   方案  
    在挂载的宿主机新建配置文件：my.cnf如上。

## 5.3 进入MySQL控制台异常

-   描述  
    no configuration file provided: not found
-   原因  
    执行：docker-compose exec mysql bash时没有指定docker-compse.yml文件
-   方案  
    在docker-compse.yml文件路径下执行：docker-compose exec mysql bash

## 6 小结

（1）必须将容器的数据库挂载到宿主机目录；  
（2）宿主机添加my.cnf配置文件；  
（3）进入MySQL控制台需通过docker compose命令：docker-compose exec mysql bash  
（4）停止当前容器时，选择stop仅停止，选择down会删除当前镜像。