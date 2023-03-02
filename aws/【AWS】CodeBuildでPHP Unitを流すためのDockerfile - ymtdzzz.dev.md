## CodeBuildでユニットテストを流したい

半年前くらいにAWSのCodeBuildで「Gitから引っ張ってきたソースでPHPUnitを動かしたいんだけど」と言われたので作成したdockerfileを一部修正して紹介する。

### 前提

前提とする構成は下記の通り。

![](https://ymtdzzz.dev/assets/static/785d0628.0df8471.a74a14ff4cb7ff2c71a7343960ac1e1f.png)

（created with cloudcraft.io）

-   開発者がCodeCommitにホスティングしているGitリポジトリにpush
-   CodeBuildが走る（**UnitTest**, デプロイ資材の作成）
-   CodeDeployからEC2にデプロイ 上記の流れをCodePipelineで組んでいる。

で、CodeBuildでUnitTestを走らせるために今回のdockerfileを作成した・・・というのが背景。

テストについては、単体テスト（DB関連含む）になるので、テスト用のDBや使用するPHP extensionとかも準備することになった。

## dockerfile作成

CodeBuildは、実行時にECRからdocker imageをpullしてくる。そのため、CodeBuild設定前の

-   dockerfileの作成
-   docker imageのbuild及びpush(to AWS ECR)

がキモになってくる。というかそれが全て。

で、色々試行錯誤して作成したdockerfileは下記の通り。

```
# dockerfile
FROM php:7

ARG composer_checksum=48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5
ARG composer_url=https://raw.githubusercontent.com/composer/getcomposer.org/ba0141a67b9bd1733409b71c28973f7901db201d/web/installer

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV PATH=$PATH:vendor/bin
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update

RUN apt-get install -y --no-install-recommends 
      apt-utils 
      locales 
      mariadb-server-10.3 
      wget 
  && echo ja_JP.UTF-8 UTF-8 > /etc/locale.gen 
  && locale-gen 
  && update-locale LANG=ja_JP.UTF-8

RUN mkdir -p /etc/sysconfig
RUN echo "NETWORKING=yes" > /etc/sysconfig/network

RUN apt-get install -y --no-install-recommends 
      curl 
      git 
      python3 
      python3-dev 
      python3-pip 
      libzip-dev 
      zlib1g-dev 
      libxslt1-dev 
      nodejs 
      npm 
      zip 
      unzip 
  && pip3 install --upgrade setuptools 
  && pip3 install awscli --upgrade 
  && pip3 install --upgrade pip 
  && export PATH=$PATH:$HOME/.local/bin 
  && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" 
        && php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" 
  && php composer-setup.php 
  && php -r "unlink('composer-setup.php');" 
  && mv composer.phar /usr/local/bin/composer 
  && cd /tmp 
  && export PATH=$PATH:$HOME/.local/bin 
  && composer create-project --prefer-dist laravel/laravel codebuild

ADD ./setup.sql ./setup.sql
# ADD php.ini /usr/local/etc/php/php.ini

RUN docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd 
  && docker-php-ext-configure mysqli --with-mysqli=mysqlnd 
  && docker-php-ext-install pdo_mysql

RUN /usr/bin/mysqld_safe & 
      sleep 10s && 
      cat setup.sql | mysql

CMD ["/usr/bin/mysqld_safe"]

ENV LC_ALL ja_JP.UTF-8
```

上記のdockerfile内では、PHPUnit用のデータベースを作成している。その際に実行されるSQLは下記の通り。

```
-- setup.sql
CREATE DATABASE IF NOT EXISTS test_database;
USE test_database;
GRANT ALL PRIVILEGES ON *.* TO test_user@localhost IDENTIFIED BY 'password';
```

このdockerfileにおいて実行される内容をざっくりと説明すると以下のようになる。

-   MySQLのインストール
-   PHPのインストール
-   composerのインストール
-   空のLaravelプロジェクトの作成
-   AWS CLIに必要なpython3のインストール
-   AWS CLIのインストール
-   PHP Extension（主にテストDB接続用）のインストール
-   mysqlサーバーの起動

空のlaravelプロジェクト作成はいらないかもしれない。結局CodeBuildで実行されるshell（buildspec.yaml）内部にて、インプットソース丸ごとcomposer installしちゃうので。

## docker

imageのbuild&ECRへのpush ここで、dockerイメージのビルドとAWS ECRへのpushを行うが、AWS CLIの設定は既に済んでいるものとする。

また、あらかじめECRに任意のリポジトリを作成しておいて、ARNをメモっとくこと。

準備が整ったら、先述のdockerfileとsetup.sqlの配置ディレクトリにおいて、下記のコマンドを実行する。

```
$ docker build -t codebuild .
$ docker tag codebuild:latest <作成したECRのARN>
$ docker push <作成したECRのARN>/codebuild:latest
```

こうすると、AWSのECRに作成したリポジトリに、dockerイメージがpushされる。

これでCodeBuildでPHP、MySQL、Composer、PHPUnitが動かせる環境が整ったので、buildspecを作成し、CodeBuildのプロジェクトを作成してインプット・アウトプットをCodePipelineで指定してやればOK。

サンプルのbuildspec.yaml（抜粋）も記載しておく。

```
# buildspec.yaml
version: 0.2

phases:
  build:
    commands:
      - service mysql restart
      - cd ${CODEBUILD_SRC_DIR}/src
      - composer install
      - composer dump-autoload
      - vendor/bin/phpunit
      - composer install --no-dev
      - php artisan config:cache
artifacts:
  secondary-artifacts:
    BuildArtifact:
      files:
        - '**/*'
```