 [![](https://cdn.learnku.com/uploads/avatars/6964_1479725880.jpeg!/both/100x100) KevinYan 的个人博客](https://learnku.com/blog/kevintech) / 7716 / 16 / 创建于 3年前 / 更新于 2年前

![CI](https://cdn.learnku.com/uploads/images/202001/09/6964/r3AINK2ITi.png!large)

### GitLab CI/CD[#](https://learnku.com/articles/17333#ef67ab)

Gitlab 持续集成是 Gitlab 提供的一整套持续集成、持续交付解决方案。Gitlab 自 9.0 版本开始增加了 CI 和 CD 功能，所以如果你的公司里的 Gitlab 上在 Settings 里找不到关于 CI/CD 的配置项那么你们确实该对公司的 GitLab 进行升级了。

我们公司之前项目部署一直在用一个叫瓦力的工具，虽然也能实现交付项目的功能但是也有不少弊端，比如：

1.  前置任务和后置任务功能不够强大，需要专门写 shell 脚本来完成复杂的任务。
2.  build 环境永远是一套，公司里有的 php 项目用的版本有 5.6、7.0、7.1 ，java 项目依赖的 jdk 版本不同，这些版本都会相互排斥，一旦一个版本的项目构建成功后必定会影响其他版本的项目。
3.  构建过程和构建结果不够可视化。

后来公司有的项目陆陆续续开始使用 GitLab CI，因为当时对这套解决方案研究不深不知道该如何在 CI 上进行代码回滚，如何管控生产环境的部署上线（比如只有权限高的人才能部署测试环境、构建完成后想手动部署生产环境而不是 push 后自动部署）所以只用来做构建和部署测试环境的代码。与此同时执行 CI Jobs 的机器仍然是一台物理机，上面需要全局安装了这些构建工具来完成项目构建工作，这仍然会遇到上面第二点项目代码版本依赖的冲突。

由于我自己现在在公司一个重点项目里做架构师，在项目开始之初就有打算将持续集成和持续交付这里好好梳理一下，解决上面这些比较突出的问题。最近由于这些问题爆发的越来越严重觉得有义务拿出一套比较好的解决方案来解决这些问题所以一直在研究解决这些问题的方案。

随着对 Gitlab CI 这套方案理解的加深慢慢制定了如下的策略：

1.  使用 Docker 来作为 git runner 的 executor（执行器），这样在每个 Job 完成后都会清理 build 环境。
2.  应用不同的 docker 镜像来解决构建代码版本依赖的问题（php7 的项目用 php7 的镜像起的容器来执行构建工作，5.6 的就用 php5.6 镜像起的容器去执行构建工作）
3.  控制 Git 工作流，针对不同功能的代码分支分别写 CI Job 去执行构建、测试和部署的工作。并且 master 只接受合并请求不允许任何人 push， 这样就能够控制发布正式环境的时间点了，而且部署操作除了 push 自动触发外也可以配置成在 GitLab 项目页面里手动点击按钮来部署。

我基本上是将 CI 分成 `build` , `test`, `deploy` 三个阶段，`build` 里主要就是完成项目代码依赖包的安装（composer 和 npm install 之类的工作， 我们前后端是两个项目，前端项目事先需要针对不同环境配置不同的打包命令）。

```
build:
  stage: build
  image: kevinyan001/git-runner:php7.1-node10
  script:
    - /usr/local/bin/composer install
  only:
    - develop
    - uat
  tags:
    - your-runner-tag
```

`test` 阶段会去执行项目中编写的测试用例:

```
test-dev:
  stage: test
  image: kevinyan001/git-runner:php7.1-node10
  script:
    - php artisan migrate --force
    - ./vendor/bin/phpunit
  only:
    - develop
  tags:
    - your-runner-tag
```

`deploy` 阶段完成项目最后的部署和一些服务器 reload 操作最终将项目交付上线。

```
deploy-testing:
  stage: deploy
  script:

	- rsync -az --delete --exclude=.git --exclude=.gitignore --exclude=.gitlab-ci.yml ./ $SERVER_TOKEN_TEST:$WEB_ROOT_TEST
    - ssh $SERVER_TOKEN_TEST -t "chown -R www:www ${WEB_ROOT_TEST}"
  environment:
    name: test
    url: http://test.example.com
  only:
    - develop
  tags:
    - your-runner-tag
```

说明：

-   任务中的 `$SERVER_TOKEN_TEST` 这些是提前在 GitLab 项目的 Settings --> CI/CD Pilelines 里定义的变量，执行任务时容器会在 BASH SHELL 中读入这些预先定义的变量。
-   `$SERVER_TOKEN_TEST` 里设置的内容是 `user@server_ip`, `${WEB_ROOT_TEST}` 里设置的是项目在服务端的路径。
-   我在容器的镜像里安装了 ansible, 发布正式环境时使用 ansible 将项目部署到正式环境对应的多个主机上。

git runner 会在每个 Job 的开始阶段通过镜像 `kevinyan001/git-runner:php7.1-node10` 跑一个容器，在容器中执行这些操作，等 Job 执行完后容器会被停止并清理掉，这就需要我们在每次容器起来的时候在容器里执行一些预备工作，比如与目标服务器建立信任关系这些基础的工作，我是通过将 SSH PRIVATE KEY 注入到容器中，目标服务器事先放上对应的公钥来建立容器与目标主机的信任关系的：

```
before_script:
  - mkdir -p ~/.ssh
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r'  > /root/.ssh/id_rsa
  - chmod -R 600 ~/.ssh
  - echo "$SSH_KNOWN_HOSTS" > ~/.ssh/known_hosts
  - chmod 644 ~/.ssh/known_hosts
  - mkdir -p /etc/ansible
  - echo 'xx.xx.xxx.xxx db_master' >> /etc/hosts #make container can connect to mysql server
  - echo 'xx.xx.xxx.xxx db_slave' >> /etc/hosts
```

具体可以参考 gitlab ci 关于这一块的说明文档：[https://docs.gitlab.com/ee/ci/ssh\_keys/](https://docs.gitlab.com/ee/ci/ssh_keys/)

由于 GitLab CI 的功能非常多，可配置像也很多所以具体某个配置的作用我就不细说了，贴几个我认为比较有用的说明文档出来节省大家的搜索时间。  
[https://docs.gitlab.com/runner/register/](https://docs.gitlab.com/runner/register/)  
[https://docs.gitlab.com/ee/ci/](https://docs.gitlab.com/ee/ci/)  
[https://docs.gitlab.com/ee/ci/examples/lar...](https://docs.gitlab.com/ee/ci/examples/laravel_with_gitlab_and_envoy/)  
[https://docs.gitlab.com/ee/ci/yaml/#jobs](https://docs.gitlab.com/ee/ci/yaml/#jobs)  
[https://docs.gitlab.com/ee/ci/environments...](https://docs.gitlab.com/ee/ci/environments.html)

另外提供一个我写的 Laravel 项目的 CI 配置文件供大家参考，这是一个完全可以应用在大型项目交付上的 CI 配置，实践的时候更换成你们具体的配置，它也同时适用于除 Laravel 以外的其他项目只需要把不同阶段执行的任务换成对应的命令即可。

.gitlab-ci.yml: [https://github.com/kevinyan815/gitlab-ci/b...](https://github.com/kevinyan815/gitlab-ci/blob/master/.gitlab-ci.yml)

[kevinyan001/git-runner:php7.1-node10](https://github.com/kevinyan815/git-runner-executor-images) 是我做的一个专门用来跑 CI 任务的容器的镜像， 你可以针对你的需求制作其他的镜像。

### Conclusion[#](https://learnku.com/articles/17333#6f8b79)

GitLab CI/CD 提供了一套通用的解决方案让你从最初的 Coding 开始到最后代码交付上线都能在它提供的工具集合中轻松完成，通过 Git Runner 的 Executor 执行不同阶段定制的任务进行代码 build、集成测试、和部署上线。除此之外还可以帮我们完成 API 文档生成、代码检查、Wiki 文档构建等工作，只要在 Linux Bash Shell 中能实现的任务它都能帮你实现。它支持用很多不同类型的 Executor 来执行 CI Jobs，其中我最推荐使用的类型是 Docker Executor，这样我们的 build 环境就不依赖于 Git Runner 宿主机上的环境，从而能够应用不同容器完成各种不同项目的构建工作。

___

推荐一个这两天很火的[数据结构和算法课程](http://gk.link/a/100D9)，之前买过不少讲持续交付、和微服务架构的课程，无奈东西太空看了两天就坚持不下去了，数据结构和算法是一个可以边学边实践练习比较基础的东西，不光对技术水平有提升，对参加一线公司的面试也是很有帮助的。

> 本作品采用[《CC 协议》](https://learnku.com/docs/guide/cc4.0/6589)，转载必须注明作者和本文链接

公众号：网管叨 bi 叨 | Golang、Laravel、Docker、K8s 等学习经验分享

本帖由系统于 3年前 自动加精