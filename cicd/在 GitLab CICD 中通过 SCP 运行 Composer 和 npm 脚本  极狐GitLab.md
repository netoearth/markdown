本指南涵盖了构建 PHP 项目的依赖项，同时使用 [GitLab CI/CD](https://docs.gitlab.cn/jh/ci/index.html) 通过 npm 脚本编译 assets。

虽然可以使用自定义 PHP 和 Node.js 版本创建自己的镜像，但为简洁起见，我们使用现有的 [Docker 镜像](https://hub.docker.com/r/tetraweb/php/)，其中包含 PHP 并安装了 Node.js。

下一步是安装 zip/unzip 包并使 composer 可用。我们将它们放在 `before_script` 部分：

```
before_script:
  - apt-get update
  - apt-get install zip unzip
  - php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  - php composer-setup.php
  - php -r "unlink('composer-setup.php');"
```

确保我们准备好所有要求。接下来，运行 `composer install` 来获取所有 PHP 依赖项，并运行 `npm install` 来加载 Node.js 包。然后运行 `npm` 脚本。我们需要将它们附加到 `before_script` 部分：

```
before_script:
  # ...
  - php composer.phar install
  - npm install
  - npm run deploy
```

在这种特殊情况下，`npm deploy` 脚本是一个 Gulp 脚本，它执行以下操作：

1.  编译 CSS & JS
2.  创建 sprites
3.  复制各种 assets（图像，字体）
4.  替换一些字符串

所有这些操作都将所有文件放入一个 `build` 文件夹中，该文件夹已准备好部署到实时服务器。

## 如何将文件传输到实时服务器[](https://docs.gitlab.cn/jh/ci/examples/deployment/composer-npm-deploy.html#%E5%A6%82%E4%BD%95%E5%B0%86%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%88%B0%E5%AE%9E%E6%97%B6%E6%9C%8D%E5%8A%A1%E5%99%A8 "Permalink")

您有多种选择：rsync、SCP、SFTP 等。现在，使用 SCP。

要完成这项工作，您必须添加一个 GitLab CI/CD 变量（可在 `gitlab.example/your-project-name/variables` 上访问）。将此变量命名为 `STAGING_PRIVATE_KEY` 并将其设置为服务器的 **私有** SSH 密钥。

### 安全提示[](https://docs.gitlab.cn/jh/ci/examples/deployment/composer-npm-deploy.html#%E5%AE%89%E5%85%A8%E6%8F%90%E7%A4%BA "Permalink")

创建一个对需要更新的文件夹**仅**具有访问权限的用户。

创建该变量后，请确保在运行时将该密钥添加到 Docker 容器中：

```
before_script:
  # - ....
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
  - mkdir -p ~/.ssh
  - eval $(ssh-agent -s)
  - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
```

按顺序：

1.  检查 `ssh-agent` 是否可用，如果不可用，安装它。
2.  创建 `~/.ssh` 文件夹。
3.  确保正在运行 bash。
4.  禁用主机检查（当第一次连接到服务器时，不要求用户接受，因为每个作业都等于第一次连接，所以我们需要这样做）。

基本上就是您在 `before_script` 部分中所需要的。

## 如何部署[](https://docs.gitlab.cn/jh/ci/examples/deployment/composer-npm-deploy.html#%E5%A6%82%E4%BD%95%E9%83%A8%E7%BD%B2 "Permalink")

如上所述，我们需要将 Docker 镜像中的 `build` 文件夹部署到我们的服务器。为此，我们创建了一个新作业：

```
stage_deploy:
  artifacts:
    paths:
      - build/
  only:
    - dev
  script:
    - ssh-add <(echo "$STAGING_PRIVATE_KEY")
    - ssh -p22 server_user@server_host "mkdir htdocs/wp-content/themes/_tmp"
    - scp -P22 -r build/* server_user@server_host:htdocs/wp-content/themes/_tmp
    - ssh -p22 server_user@server_host "mv htdocs/wp-content/themes/live htdocs/wp-content/themes/_old && mv htdocs/wp-content/themes/_tmp htdocs/wp-content/themes/live"
    - ssh -p22 server_user@server_host "rm -rf htdocs/wp-content/themes/_old"
```

以下是细节：

1.  `only:dev` 意味着这个构建只在某些东西被推送到`dev` 分支时运行。您可以完全删除并在每次推送时运行所有内容（但可能这是您不想要的）。
2.  `ssh-add ...` 我们将您在 Web UI 上添加的私钥添加到 Docker 容器中。
3.  通过 `ssh` 连接并创建一个新的 `_tmp` 文件夹。
4.  通过 `scp` 连接并将 `build` 文件夹（由 `npm` 脚本生成）上传到我们之前创建的 `_tmp` 文件夹。
5.  我们再次通过 `ssh` 连接并将 `live` 文件夹移动到 `_old` 文件夹，然后将 `_tmp` 移动到 `live`。
6.  我们连接到 SSH 并删除 `_old` 文件夹。

如何处理产物？告诉 GitLab CI/CD 保留 `build` 目录（稍后，您可以根据需要下载该目录）。

### 为什么这样做[](https://docs.gitlab.cn/jh/ci/examples/deployment/composer-npm-deploy.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%BF%99%E6%A0%B7%E5%81%9A "Permalink")

如果您仅将其用于 stage 服务器，则可以分两步执行此操作：

```
- ssh -p22 server_user@server_host "rm -rf htdocs/wp-content/themes/live/*"
- scp -P22 -r build/* server_user@server_host:htdocs/wp-content/themes/live
```

问题是您的服务器上没有应用程序的一小段时间。

因此，对于生产环境，我们使用额外的步骤来确保在任何给定时间，功能应用程序都已到位。

## 下一步[](https://docs.gitlab.cn/jh/ci/examples/deployment/composer-npm-deploy.html#%E4%B8%8B%E4%B8%80%E6%AD%A5 "Permalink")

由于这是一个 WordPress 项目，我提供了真实的代码片段。您可以追求一些进一步的想法：

-   默认分支的脚本略有不同，允许您从该分支部署到生产服务器，并从任何其他分支部署到 stage 服务器。
-   您可以将其推送到 WordPress 官方仓库，而不是实时推送。
-   您可以即时生成 i18n 文本域。

___

最终的 `.gitlab-ci.yml` 如下：

```
image: tetraweb/php

before_script:
  - apt-get update
  - apt-get install zip unzip
  - php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  - php composer-setup.php
  - php -r "unlink('composer-setup.php');"
  - php composer.phar install
  - npm install
  - npm run deploy
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
  - mkdir -p ~/.ssh
  - eval $(ssh-agent -s)
  - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'

stage_deploy:
  artifacts:
    paths:
      - build/
  only:
    - dev
  script:
    - ssh-add <(echo "$STAGING_PRIVATE_KEY")
    - ssh -p22 server_user@server_host "mkdir htdocs/wp-content/themes/_tmp"
    - scp -P22 -r build/* server_user@server_host:htdocs/wp-content/themes/_tmp
    - ssh -p22 server_user@server_host "mv htdocs/wp-content/themes/live htdocs/wp-content/themes/_old && mv htdocs/wp-content/themes/_tmp htdocs/wp-content/themes/live"
    - ssh -p22 server_user@server_host "rm -rf htdocs/wp-content/themes/_old"
```