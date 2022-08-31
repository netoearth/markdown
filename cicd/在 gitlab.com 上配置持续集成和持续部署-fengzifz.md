![Gitlab使用持续集成-fengzifz](https://www.fengzifz.com/wp-content/uploads/2018/07/WX20210521-144041@2x.png)

这篇文章，拖了很久了。

Gitlab 和 Github 一样，是一个代码托管平台，而 Gitlab 还提供了部署到私有云的安装包。不管是直接使用 gitlab.com，还是自行部署 Gitlab，免费版的 Gitlab 都够我们大部分人使用了。而且免费的 Gitlab 也提供了 CI/CD 服务，免费版的 gitlab.com 提供每个分组每个月有 2000 个 pipeline，对于独立开发者或小团队，基本足够。

看这里可以了解 gitlab.com 的会员等级以及他们的特权：[gitlab.com pricing](https://about.gitlab.com/pricing/#gitlab-com)。

### 名词解析

-   \*\*Continuous Integration (CI)\*\*：这里说的 CI，是指持续集成，并不是 PHP 框架 CodeIgniter；
-   \*\*Continuous Development (CD)\*\*：持续部署；
-   **gitlab-runner**：运行在生产环境服务器上的一个服务；
-   **Pipeline**：每一次需要进行 CI 的提交，Gitlab 都会创建一个 pipeline，并根据用户自己指定的策略来进行测试/编译/部署；
-   **.gitlab-ci.yml**：运行 gitlab ci 的配置文件；
-   **stage**：阶段，一个 pipeline 可以指定几个阶段；
-   **Jobs**：任务，每一个 stage 都会生成一个任务。

### CI/CD 流程

如下图所示，在 `CI` 阶段，会进行编译，测试等操作

而在 `CD` 阶段，进行代码 review，发布等。在 gitlab.com 上面，购买 Starter ($4/月)，还提供了代码质量检查的服务。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311868894316-1024x397.jpeg)

下图通过流水线的方式，更加直观地展示出 gitlab 的 `CI/CD` 的作用。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311869180119.jpeg)

### 配置 Gitlab CI/CD

(**_本文所有的配置均在 gitlab.com 上面进行，理论上是和自行部署的 gitlab 是一样的。_**)

#### 1\. 注册帐号

首先，你得要有一个 gitlab.com 的帐号，没有的请先打开 [gitlab.com](https://gitlab.com/) 进行注册。

#### 2\. 创建项目

点击顶部菜单的“新建”按钮，然后点击 “New project”，按照提示填入项目信息。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311846367212.jpeg)

然后把仓库拉取到本地。这里我们以 Laravel 5.5 为例，我们创建一个新的 Laravel 项目（参考：[Laravel installation](https://laravel.com/docs/5.5)）。

#### 3\. 创建 Gitlab CI 配置文件

新建 `.gitlab-ci.yml` 文件，然后填入如下信息：

<table><tbody><tr><td>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55</td><td>stages:<br>– test<br>– deploy<p># Variables<br>variables:<br>MYSQL_ROOT_PASSWORD: root<br>MYSQL_USER: homestead<br>MYSQL_PASSWORD: secret<br>MYSQL_DATABASE: homestead<br>DB_HOST: mysql</p><p># Speed up builds<br>cache:<br>key: $CI_BUILD_REF_NAME # changed to $CI_COMMIT_REF_NAME in Gitlab 9.x<br>paths:<br>– vendor<br>– node_modules<br>– public<br>– .yarn</p><p>test:<br>stage: test<br>services:<br>– mysql:5.7<br>image: edbizarro/gitlab-ci-pipeline-php:7.2-alpine<br>script:<br>– sudo yarn config set cache-folder .yarn<br>– yarn install –pure-lockfile<br>– composer install –prefer-dist –no-ansi –no-interaction –no-progress<br>– cp .env.example .env<br>– php artisan key:generate<br>– php artisan migrate:refresh –seed<br>– ./vendor/phpunit/phpunit/phpunit -v –coverage-text –colors=never –stderr<br>artifacts:<br>paths:<br>– ./storage/logs # for debugging<br>expire_in: 1 days<br>when: always<br>only:<br>– develop<br>– master</p><p>deploy_production:<br>stage: deploy<br>script:<br>– &lt;在这里执行部署命令，这里后面会详细说&gt;<br>environment:<br>name: production<br>url: https://fengzifz.com<br>only:<br>– master<br>tags:<br>– production</p></td></tr></tbody></table>

-   stages：定义有哪些阶段，我们定义了 `test` 和 `deploy` 两个阶段；
-   variables：数据库配置信息，这里我们默认跟 `.env.example` 保持一致，因为下面测试时，使用了 homestead 来集成测试环境；
-   cache：缓存，像一些公共类库，可以直接使用缓存；
-   test：这个 test 对应我们之前在 stages 里面定义的 `test`；
-   deploy\_production：在测试通过之后，我们就直接部署到生产环境。

Gitlab 会根据 `stages` 定义阶段的顺序来执行，它会先创建一个 Job 来执行 `test` stage，当 `test` 通过之后，再创建一个新的 Job 来执行 `depoly`。如果 `test` 不通过，那么 pipeline 就会显示 fail 信息。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311855192057-1024x602.jpeg)

上面是一个简单的例子，我们在 pipeline 里面只定义了简单的 `test` 和 `deploy`，流程是当测试通过之后，就部署到生产环境。

而在一些更加复杂的工程里面，我们需要做一些灰度发布时，可以在 `deploy` 阶段指定多个 Job，例如：

<table><tbody><tr><td>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31</td><td>stages:<br>– test<br>– deploy<p>(略…)</p><p>deploy_staging:<br>stage: deploy<br>script:<br>– &lt;在这里执行部署命令，这里后面会详细说&gt;<br>environment:<br>name: staging<br>url: https://staging.fengzifz.com<br>only:<br>– master<br>tags:<br>– staging</p><p>deploy_production:<br>stage: deploy<br>script:<br>– &lt;在这里执行部署命令，这里后面会详细说&gt;<br>environment:<br>name: production<br>url: https://fengzifz.com<br>when: manual<br>only:<br>– master<br>tags:<br>– production</p></td></tr></tbody></table>

按照上面的配置文件，当 master 分支有代码更新时，Gitlab 就会自动把更新内容发布到 staging 环境。注意在 `deploy_production` 里面，指定了 `when:manual`，即当我们在必须手动触发部署时，更新内容才会发布到生产环境。  
更多配置请参考官方文档：[Introduction to environments and deployments](https://docs.gitlab.com/ee/ci/environments.html)。

#### 4\. 上传代码

为了方便测试，我们直接在 master 上面上传代码。上传后，打开 gitlab.com 上面，对应的项目页面，点击左菜单的 `CI/CD > Pipelines`，然后会看到它会自动生成一个 pipeline，并显示 `running`。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311862886920-1024x580.jpeg)

打开 `CI/CD > Jobs`，会看到，gitlab 也根据在 `.gitlab-ci.yml` 所定义的任务，自动创建对应的 Job。

打开 `Operations > Environments`，gitlab 会根据在 `deploy` 阶段定义的 `environment`，自动创建对应的环境，在这里，可以进行一些重新发布或回滚等操作。

#### 5\. 配置自动部署

根据上面 1 – 4 配置好之后，虽然 pipeline 显示 pass，但最终是没有自动部署的，因为我们还没配置好自动部署。

Gitlab 的 CD，主要是使用 `gitlab-runner` 服务。

##### 安装 gitlab-runner

通过 ssh 登录到你的服务器，然后安装 gitlab-runner。

这里不做搬运工了，安装方法请直接请参考官方文档：[Install Gitlab Runner](https://docs.gitlab.com/runner/install/)。

##### 注册 gitlab-runner

在服务器上面运行 `gitlab-runner register` 来注册 gitlab-runner，期间需要输入如下信息：

<table><tbody><tr><td>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18</td><td>Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )<br># 使用 gitlab.com 的朋友，直接填写 https://gitlab.com；<br># 如果是部署私有云的朋友请填写自己的 gitlab 服务器，如果是内容，还需要在路由器做端口转发<p>Please enter the gitlab-ci token for this runner<br># 打开并登录 gitlab.com，点击左菜单的 settings &gt; CI/CD &gt; Runners，<br># 在 Specific Runners 下面，找到 token，并复制粘贴到 ternimal。</p><p>Please enter the gitlab-ci tags for this runner (comma separated):<br># 这里填入标签，gitlab-runner 是根据标签来区分的，这里可以按照自己的需求，<br># 来确定填入标签的形式，例如可以按照项目来区分，如：fz-oa，这样，<br># 在 .gitlab-ci.yml 里面，deploy 阶段的 tags，也需要填入 fz-oa 来匹配<br># 对应的 gitlab-runner</p><p>Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:<br># 选择执行 script 的方式，大家可以根据自己服务器的环境来选择<br># 下面会以 shell 为例</p></td></tr></tbody></table>

gitlab-runner 运行成功之后，在 `settings > CI/CD > Runners > Specific Runners`，会看到对应的 runner。

![](http://fengzifz.ontopic.top/wp-content/uploads/2021/05/15311880019470.jpeg)

##### 编写 deploy 脚本

因为在上面创建 gitlab-runner 时，执行 script 的方式，我们选择了 shell，所以，我们需要在服务器上面，编写一个脚本来进行部署。

(1) 切换到 gitlab-runner 用户

<table><tbody><tr><td>1</td><td>su gitlab-runner</td></tr></tbody></table>

(2) 创建 ~/.local/bin 目录

<table><tbody><tr><td>1<br>2</td><td>cd ~/.local # 如没有 .local，请自行创建<br>mkdir bin</td></tr></tbody></table>

(3) 编写 deploy 脚本

<table><tbody><tr><td>1</td><td>vi deploy</td></tr></tbody></table>

在 `deploy` 输入下面内容：

<table><tbody><tr><td>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16</td><td>#!/bin/bash<br>if [ $# -ne 2 ]<br>then<br>echo “arguments error!”<br>exit 1<br>else<br>deploy_path=”/var/www/$1/$2″<br>if [ ! -d “$deploy_path” ]<br>then<br>project_path=”git@gitlab.com:”$1/$2″.git”<br>git clone $project_path $deploy_path<br>else<br>cd $deploy_path<br>git pull<br>fi<br>fi</td></tr></tbody></table>

(4) 加上执行权限

<table><tbody><tr><td>1</td><td>chmod +x deploy</td></tr></tbody></table>

(5) 把 deploy 添加到环境变量

<table><tbody><tr><td>1</td><td>vi ~/.profile</td></tr></tbody></table>

在 `.profile` 末尾加上：

<table><tbody><tr><td>1</td><td>export PATH=”$HOME/.local/bin:$PATH”</td></tr></tbody></table>

reload 一下 `.profile`：

<table><tbody><tr><td>1</td><td>source ~/.profile</td></tr></tbody></table>

现在，就可以直接运行 `deploy xx xx` 命令了。

(7) 配置 SSH 登录  
因为脚本是通过 ssh 去克隆 gitlab.com 的仓库，所以还需要配置 gitlab-runner 的密钥对。

创建 ssh 密钥对的过程，请自行 Google 吧。创建完成之后，复制公钥到 gitlab.com 里面任意用户下面 (或者创建一个对应的叫 gitlab-runner 的用户，把公钥复制到 SSH 管理里面)。

**注意：因为 ssh 访问 gitlab 时，需要把 gitlab.com 添加到你服务器的 `know_hosts`，所以，先在服务器上面，手动执行一次 deploy 命令。**

至此，大功告成。

再次在项目提交代码时，当 gitlab.com 里面的 pipeline 运行完成之后，服务器的代码就会自动更新了。

Tags: [CI/CD](https://www.fengzifz.com/tag/ci-cd)[gitlab](https://www.fengzifz.com/tag/gitlab)