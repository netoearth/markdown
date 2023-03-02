## 1、背景环境

现将个人博客迁往 K8S 中，个人博客使用 WordPress，典型的 LNMP 架构。  
环境：

```
• K8S 版本为 1.22，单节点。
• MySQL 为一主一从集群，采用 StatefulSet 部署，位于 mysql 命名空间。
• Nginx 和 PHP，采用 Deployment 部署，位于 blog 命名空间。 
• Nginx 和 PHP 的 Service 使用 ClusterIP，使用 Ingress 进行转发。
• 数据持久化，采用 Local 类型 PV，已提前准备好 Local Storage Class。
```

## 2、准备迁移数据

先挂个维护界面吧：

```
# 使用 rewrite 将用户请求全部跳转到维护页面
server {
     # ......省略
     # 所有页面都转跳到维护页
     rewrite ^(.*)$ /maintain.html break;
}

# 维护页面
[root@cp ~]# cat /wordpress/maintain.html 
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<h1>网站正在维护中</h1>
```

准备好迁移数据：

```
[root@cp ~]# mysqldump -u root -p -B wordpress_blog  > wordpress-blog$(date +%F).sql
[root@cp ~]# zip -qr wordpress-blog$(date +%F).zip /web-project/wordpress-blog
[root@cp ~]# ll *blog*
-rw-r--r-- 1 root root  42914232 Jan 13 21:08 wordpress-blog2022-01-13.sql
-rw-r--r-- 1 root root 317531310 Jan 13 18:40 wordpress-blog2022-01-13.zip
```

## 3、部署 MySQL 主从集群

部署过程：请参阅 **[部署 MySQL 主从集群](https://www.cpweb.top/2278 "部署 MySQL 主从集群")**，注意本例 MySQL 集群属于 mysql 命名空间。  
导入数据：

```
# 导入sql数据
[root@cp ~]# kubectl exec mysql-0 -it -n mysql -- mysql < wordpress-blog2022-01-13.sql

# 验证
[root@cp ~]# kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never -n mysql -- mysql -h mysql-read -e "show databases;"
If you don't see a command prompt, try pressing enter.
+------------------------+
| Database               |
+------------------------+
| information_schema     |
| mysql                  |
| performance_schema     |
| sys                    |
| wordpress_blog         |
| xtrabackup_backupfiles |
+------------------------+
pod "mysql-client" deleted
```

创建用户：

```
# 创建wordpress远程用户
[root@cp ~]# kubectl exec mysql-0 -it -n mysql -- mysql -e "grant all on wordpress_blog.* to wordpress@'%' identified by '***********';"

# 验证
[root@cp ~]# kubectl exec mysql-0 -it -n mysql -- mysql -e "select user,host,authentication_string from mysql.user where user='wordpress';"
Defaulted container "mysql" out of: mysql, xtrabackup, init-mysql (init), clone-mysql (init)
+-----------+------+--------------------------+
| user      | host | authentication_string    |
+-----------+------+--------------------------+
| wordpress | %    | ************             |
+-----------+------+--------------------------+
```

  在本例中，MySQL 集群属于 mysql 命名空间，而 Nginx+PHP 属于 blog 命名空间，这就涉及到跨命名空间访问服务。  
  那么如何访问，这分情况：  
  同一集群相同命名空间中的任何 Pod 中，通过解析 `<Pod 名称>.<Service 名称>`（例如：mysql-0.mysql）来访问数据库。  
  同一集群不同命名空间中的任何 Pod 中，通过解析 `<Pod 名称>.<Service 名称>.<Namespace 名称>`（例如：mysql-0.mysql.mysql）来访问数据库。  
 

## 4、PHP 镜像构建

构建的 PHP 镜像，包含了 WordPress 所需要的 PHP 扩展。

```
[root@cp php-fpm]# cat dockerfile 
FROM php:7.4.27-fpm
RUN apt-get update && apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng-dev zlib1g-dev libzip-dev \
    && docker-php-ext-install gd pdo pdo_mysql mysqli exif zip \
    && apt-get install -y --no-install-recommends libmagickwand-dev \
    && pecl install imagick-3.4.3 \
    && docker-php-ext-enable imagick \
    && apt-get clean

[root@cp php-fpm]# docker build -t cp/php:fpm-wordpress .
```

## 5、创建 configmap

Nginx：

```
[root@cp blog]# cat nginx-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: blog
data:
  nginx.conf: |
    user  www-data;
    worker_processes auto;
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        set_real_ip_from 10.244.0.0/16;
        real_ip_header X-Forwarded-For;
        real_ip_recursive on;

        sendfile        on;
        keepalive_timeout  300s;
        gzip on;
        gzip_types text/html text/plain text/css text/xml application/javascript font/woff font/woff2;
        gzip_comp_level 5;
        client_max_body_size 2000m;
        include /etc/nginx/conf.d/*.conf;

        server {
            listen 80 default_server;
            server_name cpweb.top www.cpweb.top;
            root /usr/share/nginx/html;

            if (-f $request_filename/index.html){
                rewrite (.*) $1/index.html break;
            }
            if (-f $request_filename/index.php){
                rewrite (.*) $1/index.php;
            }
            if (!-f $request_filename){
                rewrite (.*) /index.php;
            }
            rewrite /wp-admin$ $scheme://$host$uri/ permanent;

            location / {
                index index.php;
            }

            location ~ \.php$ {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params; 
            }
        }
    }
```

PHP：

```
[root@cp blog]# cat php-configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
  namespace: blog
data:
  php.ini: |
    [PHP]
    engine = On
    short_open_tag = Off
    precision = 14
    output_buffering = 4096
    zlib.output_compression = Off
    implicit_flush = Off
    unserialize_callback_func =
    serialize_precision = -1
    disable_functions =
    disable_classes =
    zend.enable_gc = On
    zend.exception_ignore_args = On
    expose_php = On
    max_execution_time = 30
    max_input_time = 60
    memory_limit = 128M
    error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
    display_errors = Off
    display_startup_errors = Off
    log_errors = On
    log_errors_max_len = 1024
    ignore_repeated_errors = Off
    ignore_repeated_source = Off
    report_memleaks = On
    variables_order = "GPCS"
    request_order = "GP"
    register_argc_argv = Off
    auto_globals_jit = On
    post_max_size = 1024M
    auto_prepend_file =
    auto_append_file =
    default_mimetype = "text/html"
    default_charset = "UTF-8"
    doc_root =
    user_dir =
    enable_dl = Off
    file_uploads = On
    upload_max_filesize = 1024M
    max_file_uploads = 20
    allow_url_fopen = On
    allow_url_include = Off
    default_socket_timeout = 60
    [CLI Server]
    cli_server.color = On
    [Date]
    [filter]
    [iconv]
    [imap]
    [intl]
    [sqlite3]
    [Pcre]
    [Pdo]
    [Pdo_mysql]
    pdo_mysql.default_socket=
    [Phar]
    [mail function]
    SMTP = localhost
    smtp_port = 25
    mail.add_x_header = Off
    [ODBC]
    odbc.allow_persistent = On
    odbc.check_persistent = On
    odbc.max_persistent = -1
    odbc.max_links = -1
    odbc.defaultlrl = 4096
    odbc.defaultbinmode = 1
    [MySQLi]
    mysqli.max_persistent = -1
    mysqli.allow_persistent = On
    mysqli.max_links = -1
    mysqli.default_port = 3306
    mysqli.default_socket =
    mysqli.default_host =
    mysqli.default_user =
    mysqli.default_pw =
    mysqli.reconnect = Off
    [mysqlnd]
    mysqlnd.collect_statistics = On
    mysqlnd.collect_memory_statistics = Off
    [OCI8]
    [PostgreSQL]
    pgsql.allow_persistent = On
    pgsql.auto_reset_persistent = Off
    pgsql.max_persistent = -1
    pgsql.max_links = -1
    pgsql.ignore_notice = 0
    pgsql.log_notice = 0
    [bcmath]
    bcmath.scale = 0
    [browscap]
    [Session]
    session.save_handler = files
    session.use_strict_mode = 0
    session.use_cookies = 1
    session.use_only_cookies = 1
    session.name = PHPSESSID
    session.auto_start = 0
    session.cookie_lifetime = 0
    session.cookie_path = /
    session.cookie_domain =
    session.cookie_httponly =
    session.cookie_samesite =
    session.serialize_handler = php
    session.gc_probability = 1
    session.gc_divisor = 1000
    session.gc_maxlifetime = 1440
    session.referer_check =
    session.cache_limiter = nocache
    session.cache_expire = 180
    session.use_trans_sid = 0
    session.sid_length = 26
    session.trans_sid_tags = "a=href,area=href,frame=src,form="
    session.sid_bits_per_character = 5
    [Assertion]
    zend.assertions = -1
    [COM]
    [mbstring]
    [gd]
    [exif]
    [Tidy]
    tidy.clean_output = Off
    [soap]
    soap.wsdl_cache_enabled=1
    soap.wsdl_cache_dir="/tmp"
    soap.wsdl_cache_ttl=86400
    soap.wsdl_cache_limit = 5
    [sysvshm]
    [ldap]
    ldap.max_links = -1
    [dba]
    [opcache]
    [curl]
    [openssl]
    [ffi]

  www.conf: |
    [www]
    user = www-data
    group = www-data
    listen = 127.0.0.1:9000
    pm = dynamic
    pm.max_children = 5
    pm.start_servers = 2
    pm.min_spare_servers = 1
    pm.max_spare_servers = 3
```

执行：

```
[root@cp blog]# kubectl apply -f nginx-configmap.yaml -f php-configmap.yaml 
[root@cp blog]# kubectl get cm -n blog
NAME               DATA   AGE
kube-root-ca.crt   1      4h7m
nginx-config       1      6m47s
php-config         2      3m20s
```

## 6、创建 PV 和 PVC

  将网站数据解压到 /web-project/wordpress-blog/ 中，然后创建 Local 类型 PV 和 PVC，Local Storage Class 已经提前准备好了。

```
[root@cp blog]# kubectl get sc local 
NAME    PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  46h

[root@cp blog]# cat wordpress-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
spec:
  capacity:
    storage: 10Gi
  accessModes: ["ReadWriteMany"]
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local
  local:
    path: /web-project/wordpress-blog/
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cp
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
  namespace: blog
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 10Gi
  storageClassName: local

[root@cp blog]# kubectl apply -f wordpress-pv.yaml
[root@cp blog]# kubectl get pv wordpress-pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
wordpress-pv   10Gi       RWX            Retain           Available           local                   42s
[root@cp blog]# kubectl get pvc -n blog
NAME            STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
wordpress-pvc   Pending                                      local          44s
```

这里还要特别注意的是，挂载的网站数据权限问题导致容器数据无法正常写入。  
解决办法，基于本次迁移网站：

```
• 使用 Init Container 对容器进行初始化，修改挂载的网站数据权限。
• 因为使用的是 Local 卷，直接在宿主机上开放网站数据权限。
• 直接进入容器修改权限，保持应用和数据目录属主属组一致。
```

  这里选择用第一种方式。这里提到一个用户 www-data，这个用户一般是用于运行 web 服务的，一般很多关于 web 的容器镜像都会有这个用户，例如 nginx、httpd、tomcat、php-fpm、busybox、ubuntu 等，且 id（33）一般都是一样的。  
  这样，我们可以将 nginx 和 php-fpm 运行用户都设置为 www-data，这个直接在 ConfigMap 中改。然后在后面的 Deployment 中使用 Init Container 将挂载的网站数据属主属组都设置为 www-data 即可。  
 

## 7、创建 Service

```
[root@cp blog]# cat nginx-php-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-php-svc
  namespace: blog
spec:
  selector:
    app: nginx-php
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80

[root@cp blog]# kubectl apply -f nginx-service.yaml 
[root@cp blog]# kubectl get svc -n blog
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx-php-svc   ClusterIP   10.1.178.68   <none>        80/TCP    6s
```

## 8、创建 Ingress

  使用 Nginx Ingress 控制器，已提前准备好，对外暴露 80 和 443。还修改了 Nginx Ingress 控制器名为 ingress-nginx-controller 的 Configmap ，在 data 块下添加了几个配置，如下：

```
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    helm.sh/chart: ingress-nginx-4.0.15
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 1.1.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
data:
  allow-snippet-annotations: 'true'
  compute-full-forwarded-for: "true"
  use-forwarded-headers: "true"
  proxy-body-size: "2048m"
```

创建 Secret 包含网站 SSL 证书：

```
[root@cp blog]# kubectl create secret tls cpweb.top --key cpweb.top.key --cert cpweb.top.pem -n blog
[root@cp blog]# kubectl get secrets -n blog
NAME                  TYPE                                  DATA   AGE
cpweb.top             kubernetes.io/tls                     2      11s
```

创建 Ingress：

```
[root@cp blog]# cat blog-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cpweb.top
  namespace: blog
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - www.cpweb.top
    - cpweb.top
    secretName: cpweb.top 
  rules:
  - host: www.cpweb.top
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx-php-svc
            port:
              number: 80
  - host: cpweb.top
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service: 
            name: nginx-php-svc
            port:
              number: 80

[root@cp blog]# kubectl apply -f blog-ingress.yaml 
[root@cp blog]# kubectl get ing -n blog
NAME        CLASS   HOSTS                     ADDRESS       PORTS     AGE
cpweb.top   nginx   www.cpweb.top,cpweb.top   10.60.77.22   80, 443   86s
```

## 9、部署 Nginx 和 PHP

```
[root@cp blog]# cat nginx-php-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-php-deployment
  labels:
    app: nginx-php
  namespace: blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-php
  template:
    metadata:
      labels:
        app: nginx-php
    spec:
      initContainers:
      - name: init
        image: nginx
        command:
        - "/bin/sh"
        - "-c"
        - "chown -R www-data:www-data /usr/share/nginx/html/"
        volumeMounts:
        - name: wordpress
          mountPath: /usr/share/nginx/html/
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: wordpress
          mountPath: /usr/share/nginx/html/
      - name: php-fpm
        image: cp/php:fpm-wordpress
        ports: 
        - containerPort: 9000
        volumeMounts:
        - name: wordpress
          mountPath: /usr/share/nginx/html/
        - name: php-config
          mountPath: /usr/local/etc/php/php.ini
          subPath: php.ini
        - name: php-config
          mountPath: /usr/local/etc/php-fpm.d/www.conf
          subPath: www.conf
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config
      - name: php-config
        configMap:
          name: php-config
          items:
          - key: php.ini
            path: php.ini       
          - key: www.conf
            path: www.conf       
      - name: wordpress
        persistentVolumeClaim:
          claimName: wordpress-pvc

[root@cp blog]# kubectl get pod -n blog
NAME                                   READY   STATUS    RESTARTS   AGE
nginx-php-deployment-89dfc655b-sk42x   2/2     Running   0          34s
```

  因为使用了 CDN，进入 CDN 设置修改源站配置，指向新的地址。等待几分钟后，网站成功显示，测试功能都正常，至此迁移完成。