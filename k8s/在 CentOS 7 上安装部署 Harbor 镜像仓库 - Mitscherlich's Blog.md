### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#255d91a29ef74b50a5c9e54d933a3124 "1. 背景需求")1\. 背景需求

公司内部项目计划在明年进行容器化改造，需要先搭建内部镜像仓库，经过调研后，可供选择的解决方案有：

-   Docker Registry + Docker Registry Web

-   Gitlab Registry

-   Harbor

经过对比分析，最终选择了 Harbor 作为生产实践，主要关注点有下：

1.  支持关联 LDAP 账户

2.  支持基于组的权限管理

3.  支持复制同步，未来可用于同步生产镜像

但目前中文互联网上的配置教程普遍比较旧，本文基于写作日 (2021-12-08) [最新的 Harbor 版本](https://github.com/goharbor/harbor/releases/tag/v2.3.4)，旨在记录下如何基于 CentOS 7 系统安装搭建 Harbor 平台。

### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#9fcb7b0f419f470689ee1066c2945887 "2. 开始之前")2\. 开始之前

原则上本文不详细赘述 CentOS 7 系统的安装部署过程，但这里还是假定基于全新安装的 CentOS 7 系统，后续需要准备的依赖以及安装配置步骤。

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#db2e182c049448488b1cedf93856cc48 "2.1. 系统最低/推荐配置")2.1. 系统最低/推荐配置

根据官方文档要求，Harbor 最低/推荐安装配置如下，本地或者虚拟机用户可后续调整，云主机用户可能需要注意，Harbor 平台不仅包含 Harbor 系统自身，同时也包含了 Redis/PostgreSQL 等配套服务，本文使用了 All-in-One 的安装模式，需要注意云主机是否满足最低配置要求：

> 本文使用的虚拟机配置为 `4C` / `8G` / `100GB`。

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#586618a1d0d041449b36c92aa2c77cde "2.2. 配置中科大镜像")2.2. 配置中科大镜像

进入 CentOS 系统后，可通过以下命令修改 `yum` 安装源为中科大镜像：

```
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Base.repo
```

完成后，使用 `yum makecache` 更新 `yum` 数据库。

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#1b1538f9c260490f873eb4acda2e1a1c "2.3. 安装 Docker")2.3. 安装 Docker

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo bash ./get-docker --mirror Aliyun # 国内使用阿里云镜像加速
```

完成后，可使用以下命令检查 docker 是否安装成功：

```
sudo docker info
```

有正常输出即可。如需 docker 随系统自动启动，还需要启用 `docker.service`

```
sudo systemctl enable docker
sudo systemctl start docker
```

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#fcc65d61e8f0491c99dce39526b36eb7 "2.4. 安装 docker-compose")2.4. 安装 docker-compose

这一步没有什么投机的方法，按官网脚本安装即可，可能需要科学上网：

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

下载完成后，通过以下命令修改权限：

```
sudo chmod +x /usr/local/bin/docker-compose
```

如果其他用户需要使用，可以软链接到 `/usr/bin` 目录下：

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#45955aa5eafb45e9b396792fc72cd3b4 "3. 安装 Harbor 平台")3\. 安装 Harbor 平台

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#90eac6f4985547f2b65b11537772d82d "3.1. 下载 Harbor 离线安装包")3.1. 下载 Harbor 离线安装包

有条件的用户推荐使用离线安装包，如果访问 Github/docker.io 困难，可以配置合适的 docker 镜像仓库，使用在线安装包（仅镜像拉取过程）

```
wget https://github.com/goharbor/harbor/releases/download/v2.3.4/harbor-offline-installer-v2.3.4.tgz
```

下载后，解压至当前目录：

```
tar -zxvf harbor-offline-installer-v2.3.4.tgz
```

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#27ecf8c195a64029b625c6c4b0f77905 "3.2. 修改 Harbor 安装配置")3.2. 修改 Harbor 安装配置

解压完成后，进入当前目录下的 `harbor` 目录（即解压后的路径）：

```
cd harbor && cp harbor.yml.tmpl harbor.yml
```

这一步也是很多教程缺失的地方，实际上，从最早能查阅的 `1.10` 版本文档起，Harbor 就开始使用 `harbor.yml` 文件作为安装配置了；拷贝完成后，修改 `harbor.yml` 配置内容，这里只附上需要主要关注的内容：

```
# 用于访问 harbor 的域名或 ip 地址，注意不要使用 localhost 或 127.0.0.1
hostname: harbor.your-domain.com

# http 相关配置
http:
  # 如果你不启用 https，仅通过 http 来访问 harbor，习惯上设置端口为 5000
  # 并需要为 docker 配置 insecure_registry
  port: 5000

# https 相关配置
# 推荐启用 https
https:
  # https 监听端口，必须保持 443 不变，如果与本机其他应用冲突，可通过 ./prepare 脚本
  # 生成的 docker-compose 修改，但这里应保持 443 端口不变
  port: 443
  # 启用 https，必须提供 ssl 证书，可以使用自签名证书，但推荐使用 let's encrypt 或者
  # 阿里云/腾讯云签发的公有证书，可以使用泛域名证书
  # 放置在 /data 目录下，是因为 /data 会作为容器卷挂载，可被容器内的 nginx 读取
  certificate: /data/cert/harbor.your-domain.com.crt
  private_key: /data/cert/harbor.your-domain.com.key
```

修改保存后，可以使用 `./install.sh` 脚本安装 Harbor，如果不需要 Harbor 立即启动，可以注释文件中的第 99 行：

![notion image](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F52244e75-329b-437d-bf8a-b977e1fff1ee%2FUntitled.png?table=block&id=63acfd12-573f-4590-817f-a3cd1d6d93d9&cache=v2)

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#cd2991d39bcf48f4b525cf4aeb2c1dc8 "3.3. 安装 Harbor 平台")3.3. 安装 Harbor 平台

`./harbor` 目录下提供了 `./install.sh` 脚本供安装使用，同时，Harbor 还继承了一些可选组件：

-   **Notary**：提供安全信任机制，可以给镜像加数字签名，使用 `--with-notary` 参数启用；

-   **Trivy**：针对容器的漏洞扫描器，使用 `--with-trivy` 参数启用；

-   **Chart 仓库**：顾名思义 Helm Chart 仓库，使用 `--with-chartmuseum` 参数启用；

我们需要使用 Chart 仓库，根据实际情况添加参数即可。

```
sudo ./install.sh --with-chartmuseum [--with-trivy]
```

这一步命令会导入 Harbor 需要的镜像并生成 `docker-compose.yml` ，如果上一步禁止安装脚本自动启动 Harbor 服务，那么在启动前我们还有机会修改 `docker-compose.yml` 里的内容（比如修改 https 实际监听的端口地址）：

![notion image](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbea70672-2446-4be9-b7cb-e50ea4dbf221%2FUntitled.png?table=block&id=fb5ba763-04d0-483e-b9d6-547e3894fa83&cache=v2)

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#7b7ed86a7b8141eeb25d491ef34c85dc "3.4. 启动 Harbor 平台")3.4. 启动 Harbor 平台

全部修改保存完成后，可以使用以下命令启动 Harbor 平台：

```
sudo docker-compose up -d
```

等待结束后，即可通过上面配置的 `hostname` 访问

![notion image](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fce5fe9b8-45d9-45b5-98e4-0dc489190034%2FUntitled.png?table=block&id=d7231c94-3dad-4ef7-81c0-e75e460ce11b&cache=v2)

完成，使用默认管理员账号密码 `admin` / `Harbor12345` 登录即可。🎉

### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#af2560b748df4f3081c62fd53a974eea "4. 其他事项")4\. 其他事项

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#2ebc271804c344d19decbdd90998bb54 "4.1. 关于 LDAP")4.1. 关于 LDAP

这一步各自单位情况不一，不作统一论述，使用管理员账号登录系统后，在左侧菜单中找到 **系统管理** - **配置管理**，然后选择 **认证模式** 标签页，将 **认证模式** 改为 LDAP 即可：

![notion image](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F624291d5-2d55-46cc-97ac-1cf78c36218c%2FUntitled.png?table=block&id=8e1839e1-2178-41ba-9ff0-4476416c6217&cache=v2)

#### [](https://blog.mitscherlich.me/2021/12/install-harbor-on-centos-7#45e5c60bd05f4e0090ebc2abfb17d670 "4.2. 关于反向代理")4.2. 关于反向代理

如果 Harbor 所在机器已经安装了 nginx 或其他占用 `443` 端口的服务，可以参考上面的步骤，配置 Harbor https 监听的端口实际代理到 host 的 `8443` 端口上，然后通过 nginx 反向代理来访问 Harbor 平台，这里给出一份示例配置，可根据实际情况修改

```
server {
  server_name   harbor.your-domain.com;
  listen        80;

  location / {
    rewrite ^/(.*)$ https://$host/$1 permanent;
  }
}

upstream harbor {
  server 127.0.0.1:8443; # 127.0.0.1 替换成实际的 Harbor 地址
}

server {
  server_name harbor.your-domain.com;
  listen      443 ssl;

  client_max_body_size 1G; # docker 镜像一般比较大

  ssl_certificate     ssl/harbor.your-domain.com.crt;
  ssl_certificate_key ssl/harbor.your-domain.com;
  ssl_session_timeout 5m;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA;
  ssl_prefer_server_ciphers on;

  location ~ / {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

    # make options request fast return
    if ($request_method = 'OPTIONS') {
        return 204;
    }

    proxy_pass       https://harbor;
    proxy_redirect   off;
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port  $server_port;
  }
}
```