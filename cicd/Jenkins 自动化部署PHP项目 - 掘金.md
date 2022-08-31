### 安装 Jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

sudo yum upgrade

sudo yum install epel-release

sudo yum install java-11-openjdk-devel

sudo yum install jenkins

sudo systemctl daemon-reload

sudo systemctl status jenkins
复制代码
```

#### 修改Jenkins默认端口和用户

```
# 如用户名修改为:root; 端口修改为:8989 
vim /etc/sysconfig/jenkins
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e00b165c948943fbbf4f248c5bba9372~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 启动Jenkins

```
sudo systemctl start jenkins
复制代码
```

查看Jenkins状态

```
sudo systemctl status jenkins
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6877622bc88c4ebdb873fddf8f55d748~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 更新防火墙

```
sudo firewall-cmd --zone=public --permanent --add-port=8989/tcp
sudo firewall-cmd --reload
复制代码
```

#### 访问Jenkins

浏览器输入：[http://192.168.2.240:8989](https://link.juejin.cn/?target=http%3A%2F%2F192.168.2.240%3A8989 "http://192.168.2.240:8989")

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2019c0976d4e43f79fe21c7306eae518~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

安装推荐插件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e79b51e28c04ec2b1153c12e79001b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8515fb95b5094dffa64ea8fdf21928e5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

创建第一个管理员用户

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8354122b9ee40f7876147be37b1098c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

开始使用Jenkins

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b888a9ca44004b588df73f6a4d337c2d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 安装插件

1.  GitLab

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90a7ad84a3d14338ad60f44dbb824649~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

2.  Publish Over SSH

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1170f65503014eeb9eb590a26c1ee6d9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 前提准备

#### 配置证书(用于从GitLab中拉取代码)

Dashboard -> Manage Credentials -> Jenkins(global) -> 全局凭据 -> Add Credentials

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeaa1b54086d4399b7762acabed101b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 在 Jenkins 所在的发布机安装 Git

```
yum install git
复制代码
```

#### 在 Jenkins 所在的发布机安装php7

```
sudo yum install epel-release
sudo yum install yum-utils
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi-php74
sudo yum install php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysqlnd

# ext 
yum install php-xml
yum install php-intl
yum install php-mbstring
yum install php-pecl-zip

# 重启php
systemctl restart httpd
复制代码
```

#### 在 Jenkins 所在的发布机安装 node

```
curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -

sudo yum install nodejs

node --version

npm --version
复制代码
```

#### 在 Jenkins 所在的发布机安装 composer

```
# 下载 composer 安装程序脚本
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# 验证下载的完整性，签名匹配会输出 Installer verified
HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# 安装composer
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
复制代码
```

#### 配置 SSH 服务器

1.  **首先需要将发布机的公钥传输到生产机。**

将发布机上的 /root/.ssh/id\_rsa.pub 传送到 生产机的 .ssh/（目录没有.ssh请创建）

将 id\_rsa.pub 改名为 authorized\_keys

测试无密码链接 ssh 生产机IP

```
# 发布机生成rsa密钥
# ssh-keygen -t rsa -b 2048 这个是高版本的密钥，Jenkins不支持
ssh-keygen -m PEM -t rsa -b 4096

# 传送到生产机
scp /root/.ssh/id_rsa.pub root@192.168.2.134:/root/.ssh/authorized_keys
复制代码
```

2.  **配置 SSH Servers**

Dashboard -> Manage Jenkins -> Configure System

在页面找到Publish over SSH栏，配置 SSH Servers

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b91b205b3a7246aaa7d442ed7d2febc1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

记得勾选 Use password authentication,or use a different key

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21824b33fb7649e18f57ddd1e8f76a10~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

3.  **测试配置是否成功**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a292a5c200d4f72ac9307d879ce7c87~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 创建 Jenkins 项目配置

创建一个 Pipeline 风格的项目

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee1efbf7dbf94b0fb72aa3027efd0a0b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 参数化构建

Dashboard -> oa-pipeline -> Configure -> General 记得选择 String Parameter

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33e541971f3e402494755eaa4fcb4947~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### pull code

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e992cef0d1f4917be9c90833bbd734a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

```
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/${branch}']], 
                extensions: [], 
                userRemoteConfigs: [[credentialsId: 'fe3b2a84-557c-431d-84a5-e4d93e9e9717', url: ${git仓库地址}]]])
            }
        }
    }
}
复制代码
```

如果是从tag拉取代码，需要这么配置

```
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: 'refs/tags/${tag}']],
                extensions: [], 
                userRemoteConfigs: [[credentialsId: 'fe3b2a84-557c-431d-84a5-e4d93e9e9717', url: ${git仓库地址}]]])
            }
        }
    }
}
复制代码
```

#### build project

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/425eba253acd4da5b284222173c06420~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

```
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: 'refs/tags/${tag}']], 
                extensions: [], 
                userRemoteConfigs: [[credentialsId: 'fe3b2a84-557c-431d-84a5-e4d93e9e9717', url: ${git仓库地址}]]])
            }
        }
        stage('build project') {
            steps {
                sh '/usr/local/oa/oa_build.sh'
            }
        }
        
    }
}
复制代码
```

脚本内容如下：

```
#!/bin/bash
cd /var/lib/jenkins/workspace/php-oa && /usr/local/bin/composer install
cd /var/lib/jenkins/workspace/php-oa/slimvue-oa && npm install
cd /var/lib/jenkins/workspace/php-oa/slimvue-oa && npm run publish
复制代码
```

保存退出。并修改文件权限为可执行：

```
chmod 764 /usr/local/oa/oa_build.sh
复制代码
```

#### post build

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c7e2600c36940208655a83eaf575db2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

在目标主机中编写脚本

```
#!/bin/bash
cp /data/projects/oa_new/resource/config.yml /data/projects/oa_new/oa/config/config.yml
rm -rf /data/projects/oa_new/oa/cache/*
复制代码
```

保存退出。并修改文件权限为可执行：

```
chmod 764 deploy.sh
复制代码
```

最终Pipeline配置如下：

```
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: 'refs/tags/${tag}']], 
                extensions: [], 
                userRemoteConfigs: [[credentialsId: 'fe3b2a84-557c-431d-84a5-e4d93e9e9717', url: ${git仓库地址}]]])
            }
        }
        stage('build project') {
            steps {
                sh '/usr/local/oa/oa_build.sh'
            }
        }
        stage('post build'){
            steps{
                sshPublisher(publishers: [sshPublisherDesc(configName: '98', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /data/projects/oa_new
sh deploy.sh''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/data/projects/oa_new/oa', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
复制代码
```

### 开始部署

#### Build with Parameters

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38f25510bc094809909178887e5b2e59~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

点击 Build，查看日志可以看到部署已经成功。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/807b716fc05a4a7c97de270620a1f7d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

因为前端项目比较古老了，一直没优化，可以看到构建还是比较慢的。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4296418086d548d7b7dd99aafffda974~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)