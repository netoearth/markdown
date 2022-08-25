![gitlab ci deploy](https://www.cloudways.com/blog/wp-content/uploads/PHP-Test-and-Deployment-With-Gitlab-CICD.jpg)

## CI & CD – An Introduction 

Continuous Integration is a development workflow in which multiple developers continuously merge their code changes into a common, shared code repository. This is usually combined with code testing. There are several [best ci cd tools](https://www.cloudways.com/blog/continuous-deployment-tools/) that help businesses to deploy upgrades and maximize their productivity. In this article, I’ll help you understand the importance of GitLab CI deployment to Server and will show you how you can create a CI/CD pipeline using Cloudways.

The idea of CI is to merge all the changes to the main branch as often as possible so that changes can be tested to work with all the other introduced changes. 

Continuous Delivery is the practice of continuous shipping of changes to an environment where those changes can be reviewed in terms of the final product – whether it is delivered to production or staging environment. This enables users – testers or the final audience – to inspect the changes against the project. Changes are delivered regularly, in small batches, so that any issues can be detected early on.

Continuous Deployment is a streamlined development cycle of shipping code changes to production automatically, in small batches, after they pass automated tests.

While Continuous Delivery workflow usually comes with a staging environment, where the changes are tested before being deployed, Continuous Deployment setup means that changes are tested and shipped to the production environment automatically.

This process presumes an extensive setup of automated tests (including unit, integration, and acceptance tests.

1.  [CI & CD – An Introduction](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#intro)
2.  [Why Automate Application Testing and Deployment?](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#automate-application-testing)
3.  [Introduction to GitLab CI/CD](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#gitlab)
4.  [Complete Process of CI/CD with Gitlab for PHP app on Cloudways](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#process)
5.  [Creating Tests](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#tests)
6.  [Setting up deployment](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#deployment)
7.  [Conclusion](https://www.cloudways.com/blog/deploy-gitlab-ci-cd/#conclusion)

## Why Automate Application Testing and Deployment?

Continuous Deployment (CD) represents the full automation of the development cycle, where developers can rapidly ship new features and ideas to production. 

For a dev environment, the CD brings: 

-   Every code change is by default thoroughly tested. Frequent deployment of changes means that even the bugs that are not captured by automated testing can be easily tracked to specific commits.
-   When it is properly set up, Continuous Deployment with automated testing and atomic deployments makes codebase and bug tracking more manageable
-   Frequent, atomic deployment of changes means that the feedback cycle can be faster, not only in terms of application logs and monitoring for bugs but also in terms of customer feedback. This especially fits in with the lean startup paradigm – because business can quickly put new ideas before its users and observe how it is received.

[At Cloudways](https://www.cloudways.com/), our mission is to make the developer’s life easier, and to make the complexities of the deployment process simple, so that you can focus on things that make difference.

GitLab is an open-source end-to-end DevOps tool that integrates version control with user-definable Continuous Integration and Continuous Deployment pipelines. It is available under the MIT license and can be installed on third-party servers. [Gitlab.com](https://gitlab.com/) is a hosted, SaaS GitLab version that makes it possible to use GitLab without dealing with the complexities of maintaining a GitLab server on the own.

In this tutorial, I will introduce GitLab, and show how it can be used to automate a DevOps development cycle. I will take a look at the process of setting up a GitLab CI/CD deployment workflow with PHP and Laravel framework.

## Stop Wasting Time on Servers

Cloudways handle server management for you so you can focus on creating great apps and keeping your clients happy.

## Introduction to GitLab CI/CD

![gitlab logo](https://www.cloudways.com/blog/wp-content/uploads/image5-202-1024x419.png)

GitLab touts itself as the “complete DevOps platform”. The (hosted) platform offers a range of paid plans, but its free tier offers quite a lot – including unlimited collaborators and private repositories. I will use the free tier in this tutorial. 

Being an end-to-end tool, GitLab gives us the ability to manage source code with Git, and to connect a code repository with the production (or staging) infrastructure, it offers the streamlined workflow, and infrastructure for testing and deploying the application, all at a click of a button. It can also be set up to test and deploy code automatically, on every git push.

It features public and private package registry and container registry – where you can publish and later use software packages and Docker container images.

![gitlab ci cd pipelines](https://www.cloudways.com/blog/wp-content/uploads/image19-42.png)

GitLab provides the ability to define customized pipelines for the DevOps cycle – from preparing to building, testing, configuring, and deploying the software. It also features monitoring tools to gather feedback information about commits – this includes performance metrics, tools for viewing and examining [logs](https://docs.gitlab.com/ee/user/project/clusters/kubernetes_pod_logs.html#kubernetes-pod-logs), incident management, error tracking, [tracing](https://docs.gitlab.com/ee/operations/tracing.html) – monitoring the health of different microservices, etc.

### Main GitLab Terms

Before we start setting up a DevOps workflow with GitLab, I need to briefly lay out the GitLab terminology.

#### .gitlab-ci.yml – the main part of the GitLab workflow

.gitlab-ci.yml is a YAML file that is added to all repositories which use GitLab Continuous Deployment workflow – it is the main configuration file where we define stages of the DevOps cycle, where I define specific jobs, variables that will be used, scripts that will be executed, etc.

Some of the keywords used in this file are:

-   script – marking a section with the shell script to be executed
-   artifacts – listing of resources to be created by a job, uploaded to GitLab
-   cache – list of files to be cached between subsequent job runs
-   environment – used to specify what environment the respective job will deploy to
-   extends – designates entries to inherit from
-   image – specifying a Docker image to be used
-   include – specifying external YAML files to include 
-   services – Docker service images – e.g. MySQL service
-   stage – used to define different stages of the CI/CD workflow
-   trigger – for  defining triggers for the creation of downstream pipelines
-   variables – for defining job variables
-   when – for specifying when the job is run – e.g. on success, on failure, always (regardless of the result of earlier stages), manual (indicating that it needs to be executed manually), etc.

There are [more keywords that you can use](https://docs.gitlab.com/ee/user/project/clusters/kubernetes_pod_logs.html#kubernetes-pod-logs) to define jobs with GitLab, but I will not be using all of them in this tutorial.

Usually, the file will define stages with keyword stages, and then proceed to define different jobs that belong to defined stages.

#### Pipelines

Pipelines represent an entire CI/CD process, including multiple _jobs_ ran across different _stages._ Pipelines are defined through the .gitlab-ci.yml files, and they can be of varying complexity, including multi-project pipelines, parent-child pipelines, etc. In this tutorial, we will implement a very basic pipeline.

#### Jobs

![gitlab ci cd pipelines](https://www.cloudways.com/blog/wp-content/uploads/image12-101.png)

Jobs are a fundamental part of every GitLab pipeline, and they can be defined to run in different stages, as we can see in the image above

Jobs are runare ran by _runners,_ and at the  minimum, have _script_ element defined:

jobA:

  script: “…”

  stage: test

jobB:

  script: “…”

  stage: deploy

#### Runners

![gitlab runners](https://www.cloudways.com/blog/wp-content/uploads/image8-154-1024x756.png)

GitLab documentation defines runners as “lightweight, highly-scalable agents that pick up a CI job through the coordinator API of GitLab CI/CD, run the job, and sen the result back to the GitLab instance”

On its own, GitLab Runner is an open-source application written in Go, and can be run, apart from GitLab’s infrastructure, also on development machines, or on server infrastructure, inside Docker containers or deployed on Kubernetes clusters.

Runner agents are responsible for processing jobs, and they can process them on the same machine where the GitLab Runner is installed, or in containers and remote infrastructure

#### Environment Variables

Variables in GitLab are named values that affect how specific jobs are run. Environment variables mean they are a part of the environment in which the job is executed. 

GitLab CI/CD [predefines a set of variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) that we can use in pipelines, by their names. At the same time, I can define my own, custom variables. These can be _Variable_ or _File_ type. They can be defined [through the UI](https://docs.gitlab.com/ee/ci/variables/README.html#create-a-custom-variable-in-the-ui), [through API](https://docs.gitlab.com/ee/api/project_level_variables.html), or in the .gitlab-ci.yml file.

Without further ado, I will jump into the project – setting up Continuous Deployment to Cloudways server with GitLab CI/CD.

## Complete Process of CI/CD with Gitlab for PHP app on Cloudways

In this section, I will set up the automated Continuous Deployment workflow for a custom PHP app to one of the Cloudways servers. 

For this, I will use a Laravel app because of the framework’s popularity in dev circles. 

### Creating the Application on Cloudways Platform

The first thing I am going to do is create an application on Cloudways. I will start by launching a new managed server. 

After I have selected all the particulars of the server instance and launched it – which may take a minute or two – I will proceed to [install the web application](https://support.cloudways.com/how-to-launch-a-new-application-on-a-new-server/).

![cloudways laravel app](https://www.cloudways.com/blog/wp-content/uploads/image14-79-1024x451.png)

Cloudways makes it easy to launch an entire server environment with a preinstalled and pre-optimized software stack that contains all the elements I need to run a web application. The Cloudways web hosting stack  Apache, NGINX, MongoDB, and PHP.

Once the application is launched, it will be available on the provisional Cloudways subdomain until I assign a top-level domain name to it. 

I will also be able to add the public SSH key to it, to enable passwordless authentication.

![cloudways staging url](https://www.cloudways.com/blog/wp-content/uploads/image2-298-1024x567.png)

Under Application Settings, I will want to enable SSH access to the application, and disable Varnish for the duration of the initial setup. The varnish is a full-page caching system that speeds up the web app frontend, but I will not be able to see the changes in the application in real-time, so this is something that I will enable when I have deployed the final application version.

![cloudways application settings](https://www.cloudways.com/blog/wp-content/uploads/image22-27-1024x623.png)

Once I added the SSH key and checked these settings, I will be able to mirror the application code from the server to my local machine (if I [chose the Laravel application](https://support.cloudways.com/deploy-laravel-on-cloudways/)) so I can work on it.

If I installed the PHP application/stack on the server, or if I wish to have the newest Laravel version to work with, I will skip this step and do the initial app deployment via rsync or scp from the local machine, or via Git from the GitLab repository.

If I choose a Laravel application, the application directory structure on the server will look something like this:

![laravel application directory](https://www.cloudways.com/blog/wp-content/uploads/image9-128-985x1024.png)

–  public\_html is the directory where the application code will live, with public\_html/public/index.php as the entry script.

### Creating & pushing to a repository on GitLab.com

The next step will be to initiate a repository on GitLab. Presuming I have signed up on the website, I will simply create a project, which will give me an empty repository:

![create project in gitlab](https://www.cloudways.com/blog/wp-content/uploads/image11-131-1024x633.png)

I can also choose to import an existing repository from Github or Bitbucket.

Next, I will push the application to the repo. I assume that I already have a Laravel application on the local machine – either by downloading it from the public\_html of the application or by [initiating a Laravel application locally](https://laravel.com/docs/8.x/installation#via-laravel-installer) with laravel new.

The application should already have .gitignore, so I will initiate the Git repository, add the GitLab remote address, commit and push the app (this is presuming that I have already added the public SSH key to the GitLab profile):

1.  git init

3.  git remote add origin git@gitlab.com:username/repo-name.git

5.  git add .

7.  git commit -m "Initial commit"

9.  git push -u origin master

– naturally, I will replace the username and repo-name with the actual details of my GitLab repository. Having done this, the Laravel application will now be visible on GitLab.

Now, if I just want to be able to pull the code changes manually from GitLab, that is a fairly simple setup, which [we explained here](https://www.cloudways.com/blog/deploy-through-gitlab/). But in this tutorial, I will set up automated testing & deployment pipeline which will test my code and push it to the server automatically on every commit.

### Creating Tests

The next thing I will do is create some tests. The default Laravel scaffolded application already comes with default tests in the test folder of the application:

![unit test](https://www.cloudways.com/blog/wp-content/uploads/image16-59-1024x429.png)

If I open tests/Feature/ExampleTest.php, I will see something like this:

1.  <?php

3.  namespace Tests\\Feature;

5.  use Illuminate\\Foundation\\Testing\\RefreshDatabase;

7.  use Tests\\TestCase;

9.  class ExampleTest extends TestCase

11.  {

13.  /\*\*

15.  \* A basic test example.

17.  \*

19.  \* @return void

21.  \*/

23.  public function testBasicTest()

25.  {

27.  $response = $this->get('/');

31.  $response->assertStatus(200);

33.  }

35.  }

Function testBasicTest will always return true. If I run PHPUnitnow (or vendor/bin/phpunit, depending on the PHPUnit installation, we will see two tests passing (Feature and Unit tests):

![phpunit testing](https://www.cloudways.com/blog/wp-content/uploads/image17-53-1024x163.png)

Now, I will want to expand these tests and set up some real testing. I will add two more test cases to the tests/Feature/ExampleTest.php:

1.  public function testIndexView()

3.  {

5.  $view = $this->view('index');

9.  $view->assertSee('Homepage');

11.  }

13.  public function testAboutView()

15.  {

17.  $view = $this->view('about');

21.  $view->assertSee('About');

23.  }

If I save this file and try running the tests again, I will see the following:

![unit test errors](https://www.cloudways.com/blog/wp-content/uploads/image20-28-1024x653.png)

I can see that of four tests, two have failed. And that is what I want, for now. 

I will now add Dockerfile to the repository. Dockerfile serves as the container in which GitLab will run the tests. The base image for the Dockerfile will be php:7.4 image:

\# Set the base image for subsequent instructions

FROM php:7.4

1.  \# Updating packages

3.  RUN apt-get update

5.  \# Installing dependencies

7.  RUN apt-get install -qq apt-utils git curl libzip-dev libjpeg-dev libpng-dev libfreetype6-dev libbz2-dev

9.  \# Clearing out the local repository

11.  RUN apt-get clean

13.  \# Installing extensions

15.  RUN docker-php-ext-install pdo\_mysql zip

17.  \# Installing Composer

19.  RUN curl --silent --show-error https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

21.  \# Installing Laravel Envoy

23.  RUN composer global require "laravel/envoy=~1.0"

After Dockerfile is saved in the app root folder, I will log in to the GitLab docker registry and build & push the image:

1.  docker login registry.gitlab.com

3.  docker build -t registry.gitlab.com/username/repo-name .

5.  docker push registry.gitlab.com/username/repo-name

I will also add the.gitlab-ci.yml file, in which I will add the address of the image:

1.  image: registry.gitlab.com/username/repo-name:latest

3.  stages:

5.  \- test

7.  test\_basic:

9.  stage: test

11.  script:

13.  \- cp .env.example .env

15.  \- composer install

17.  \- php artisan key:generate

19.  \- vendor/bin/phpunit

To start, I have defined only one stage – test, and one job belonging to that stage: test\_basic.

I will now commit the changes and push them to GitLab:

1.  git add .

3.  git commit -m “added CI & Dockerfile”

5.  git push --set-upstream origin master

After the code was pushed, I will be able to observe GitLab at work. If I now go to the repository page on gitlab.com and navigate to CI / CD > Pipelines, I will see the job running:

![ci/cd pipeline](https://www.cloudways.com/blog/wp-content/uploads/image3-263-1024x259.png)

and after a couple of minutes I will see the failed job:

![running ci/cd pipeline](https://www.cloudways.com/blog/wp-content/uploads/image1-299-1024x234.png)

I can look into the details:

![dockerfile](https://www.cloudways.com/blog/wp-content/uploads/image7-163-1024x569.png)

By clicking through the pipeline/job button, I will eventually be presented with the command line output from the container.

The two test cases I added to feature tests presume certain text being present in index and about pages. asertSee() asserts that the specified string is present in the response. It is invoked on the instance of Illuminate\\Testing\\TestView

For tests to pass, I will add the appropriate views in resources/views I need to create index.blade.php and about.blade.php):

![laravel blade template](https://www.cloudways.com/blog/wp-content/uploads/image18-47-1024x590.png)

In routes/web.php I will replace existing routes with the following:

1.  Route::get('/', function () {

3.  return view('index');

5.  });

7.  Route::get('/about', function () {

9.  return view('about');

11.  });

– and now the tests should pass.

### Setting up deployment

The first thing to do will be to create the SSH keys on the Cloudways server and to add them to GitLab. Due to constraints of the application-level users, I [will log into the server as](https://support.cloudways.com/difference-between-master-credentials-and-application-credentials/) a master user and [create an SSH key pair](https://www.ssh.com/ssh/keygen/) with ssh-keygen. This will produce id\_rsa and id\_rsa.pub files in the master user’s home directory, under .ssh.

I will then copy id\_rsa (private key) file and add it as a variable in the GitLab project, under Settings > CI / CD:

![adding variable in gitlab](https://www.cloudways.com/blog/wp-content/uploads/image13-88-1024x808.png)

For this tutorial, I will name it ID\_RSA. Now I can use the private key in the jobs.

I will also copy the public key – id\_rsa.pub, and add it to GitLab (Project > Settings > Repository) as Deploy Key

Next, copy id\_rsa.pub to the authorized\_keys on the Cloudways server:

cat ~/.ssh/id\_rsa.pub >> ~/.ssh/authorized\_keys

This way, I will be able to pass the private key into the container environment, which means that the GitLab jobs can be authorized against the Cloudways server

Next, I will clone the GitLab repo to anywhere on the server. Adding GitLab to the local known\_hosts – might fail otherwise. 

Now, when I deploy to the Cloudways server, I need to remember that Cloudways supports two levels of users – master user and application-level user. Neither of them has root privileges, but the master user has some permissions that I need for deployments. For example, the application-level user is not able to create an SSH key pair which I will use, because the user cannot access the files in the .ssh folder. 

The second thing to pay attention to is the permissions of different directories. The public\_html application directories cannot be deleted, and the strategy in this deployment setup will be to have different releases in the private\_html application directory and then to link to the newest one from the webroot. 

This will make instant switching between code versions possible – there will be no downtime (copying or even manipulating Git commits/versions can still incur a certain time to finish, so changing the webroot symbolic link target is perhaps the fastest way, both in introducing new changes and reverting to previous versions.

This strategy means that I will need the webserver root folder to be a symbolic link that is (re)created on every deployment.

For this reason, I will empty the public\_html directory, and inside it create a webroot symbolic link, targeting the most recent release. Releases will be stored in the private\_html directory, and in case of need to revert the changes, I can instantly link back to the previous code version. This way, the application should have no downtime even if I push code changes that introduce errors in the application. I can roll back all the changes to a precise point at a click of a button.

#### Laravel Envoy

[Envoy](https://laravel.com/docs/8.x/envoy) is Laravel’s task runner.

In the configuration, when GitLab runs the pipeline and launches the container that I defined with the Dockerfile after the tests have successfully passed, it will run a deployment. This means that Envoy, acting running on a separate machine,

will:

-   log in to the Cloudways server, 
-   set up needed directories
-   clone the repository, 
-   run composer to install all the dependencies and 
-   update symbolic links

In the meantime, I may also need to set it up to change certain directory/file ownership and permissions 

To configure Envoy to do all this, I will create Envoy.blade.php in the root of the application:

1.  @servers(\['web' => '<master\_server\_username>@<server\_ip>'\])

3.  @setup

5.  $root\_app\_dir = '/home/<application\_directory\_path>';

7.  $releases\_dir = $root\_app\_dir .'/' . 'private\_html';

9.  $release = date('YmdHis');

11.  $current\_release\_dir = $releases\_dir .'/'. $release;

13.  $repository = 'git@gitlab.com:<gitlab username>/<repo name>.git';

15.  $webroot = 'webroot';

17.  @endsetup

19.  @story('deploy')

21.  clone\_repository

23.  run\_composer

25.  update\_symlinks

27.  @endstory

29.  @task('clone\_repository')

31.  echo 'Cloning repository'

33.  \[ -d {{ $releases\_dir }} \] || mkdir {{ $releases\_dir }}

35.  git clone --depth 1 {{ $repository }} {{ $current\_release\_dir }}

37.  cd {{ $current\_release\_dir }}

39.  git reset --hard {{ $commit }}

41.  @endtask

43.  @task('run\_composer')

45.  echo "Deploying ({{ $release }})"

47.  cd {{ $current\_release\_dir }}

49.  composer install --prefer-dist --no-scripts -q -o

51.  @endtask

53.  @task('update\_symlinks')

55.  echo "Linking storage directory"

57.  rm -rf {{ $current\_release\_dir }}/storage

59.  \[ -d {{ $releases\_dir }}/storage \] || mkdir {{ $releases\_dir }}/storage

61.  \[ -d {{ $current\_release\_dir }}/bootstrap \] || mkdir {{ $current\_release\_dir }}/bootstrap

63.  ln -nfs {{ $releases\_dir }}/storage {{ $current\_release\_dir }}/storage

65.  chown -R $USER:www-data {{ $current\_release\_dir }}/storage {{ $current\_release\_dir }}/bootstrap

67.  echo 'Linking .env:'

69.  ln -nfs {{ $releases\_dir }}/.env {{ $current\_release\_dir }}/.env

71.  echo 'Linking current release:'

73.  ln -nfs {{ $current\_release\_dir }} {{ $root\_app\_dir }}/public\_html/{{ $webroot }}

75.  @endtask

Here, I will need to update the parts of the @servers and @setup sections that contain actual application data like usernames, repository names, exact paths, surrounded with < and >.

Note that @servers contains the Cloudways server details, @setup sets up the environment by defining some variables, @story is a collection of tasks, which is followed by definitions of the actual tasks.

update\_symlinks task contains a lot of the little tasks which may make or break the deployment and you may need to update it to change webroot/publicpermissions or ownership of the directories on his system. I will link to the storage directory which will live under the private\_html folder. The bootstrap folder will need to be created, with proper write access, for the application to work.

Before deployment, I will put the initial .env file also into the private\_html and will link the app directory to it.

Because I cannot replace the public\_html itself, I am creating a symlink “webroot” under public\_html pointing to the current release. 

For this to work later, in Cloudways web administration panel, under Application Settings > General, I will add a string  webroot/public to the WEBROOT setting – webroot is the symlink to the current release and public is the actual folder name where laravel entry script – index.php – lives:

![cloudways webroot](https://www.cloudways.com/blog/wp-content/uploads/image4-236-1024x358.png)

After the Envoy configuration, I will also update the .gitlab-ci.yml file, and add the deployment job:

1.  image: registry.gitlab.com/<username>/<repo-name>:latest

3.  stages:

5.  \- test

7.  \- deploy

9.  test\_basic:

11.  stage: test

13.  script:

15.  \- cp .env.example .env

17.  \- composer install

19.  \- php artisan key:generate

21.  \- vendor/bin/phpunit

23.  deploy\_cloudways:

25.  stage: deploy

27.  script:

29.  \- 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'

31.  \- eval $(ssh-agent -s)

33.  \- ssh-add <(echo "$ID\_RSA")

35.  \- mkdir -p ~/.ssh

37.  \- '\[\[ -f /.dockerenv \]\] && echo -e "Host \*\\n\\tStrictHostKeyChecking no\\n\\n" > ~/.ssh/config'

39.  \- ~/.composer/vendor/bin/envoy run deploy

41.  environment:

43.  name: production

45.  url: <our\_app\_url>

47.  only:

49.  \- master

I am also adding the ID\_RSA variable to the environment.

Once all of this is done, the application directory structure, after some deployments, should look something like this:

![application directory after deployment](https://www.cloudways.com/blog/wp-content/uploads/image9-129-985x1024.png)

The numbered directories under private\_html represent different releases.

If all this has been done correctly, I can commit and push the code changes to GitLab.

I will be able to observe the pipeline running, now with two stages:

![gitlab pipeline](https://www.cloudways.com/blog/wp-content/uploads/image15-71-1024x111.png)

– and after the jobs are finished, if I click the throughout pipeline button, I should see that the pipeline ran successfully:

![successful gitlab pipeline](https://www.cloudways.com/blog/wp-content/uploads/image21-31-1024x336.png)

If I now visit the application’s public URL, I should see the live version:

![example](https://www.cloudways.com/blog/wp-content/uploads/image6-183-1024x621.png)

In case there are any issues, I can go back to the pipeline page and click the Deployment Job button to see the terminal output of the deployment, and to polish any details that may need fixing.

## Conclusion

In this tutorial, I took a close look at setting up a Continuous Deployment pipeline to a Cloudways server. I introduced all the main principles of CI/CD, terms used in working with GitLab CI/CD, and then went on to set up a small but functional test-deploy pipeline with Laravel.

This is by no means an exhaustive tutorial on the subject, but I am confident that it is a solid starting point for setting up more comprehensive workflow scenarios. If you are planning to use the GitLab CI Deploy feature then the above-explained example will help you get started with it.

At Cloudways, we strive every day to make developers’ lives simpler and hope we have helped do that with this tutorial. Let us know what you think!

Share your opinion in the comment section. COMMENT NOW

### Share This Article

Customer Review at ![](https://www.cloudways.com/blog/wp-content/uploads/g2-crowd-logo.png)

#### “Cloudways hosting has one of the best customer service and hosting speed”

Sanjit C \[Website Developer\]

![](https://secure.gravatar.com/avatar/a19ca97b6163594247b78eedc9574c1f?s=128&d=mm&r=g)

#### [Tonino Jankov](https://www.cloudways.com/blog/author/tonino-jankov/)

Tonino is an entrepreneur, OSS enthusiast and technical writer. He has over a decade of experience in software development and server management. When he isn't reading error logs, he enjoys reading comics, or explore new landscapes (usually on two wheels).