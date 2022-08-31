This month I started to read a book called “Continuous Delivery with Docker and Jenkins — Second edition” from Rafal LESZKO. I bought it on Amazon.

I’m a professor at the University EAFIT in Colombia, and I teach a software architecture course (I have a Ph.D. in computer science). I never used Jenkins and I’m relatively new to CI/CD topics. For my software architecture course, I use the Laravel framework. So, I thought it will be good, try to replicate the elements I learned from the “Continuous Delivery with Docker and Jenkins — Second edition” book, but applied to Laravel (to get some practical knowledge in these fields).

This story is about that. I will show you some results and some code, about how I applied the lessons I learned in that book (Chapters 1 to 5 / I plan to develop a second part of this story regarding Chapters 6 to 8) but applied to the Laravel framework (the book originally shows an example in Java with the Spring boot framework).

0\. **Concepts introduction**

**Docker** is a tool designed to make it easier to create, deploy, and run applications by using containers.

**Jenkins** is an open-source automation tool written in Java with plugins built for Continuous Integration purposes.

**GitHub** is a code hosting platform for version control and collaboration.

**Continuous integration (CI)** and **continuous delivery (CD)** embody a culture, set of operating principles, and collection of practices that enable application development teams to deliver code changes more frequently and reliably. The implementation is also known as the CI/CD pipeline.

**Laravel** is a free, open-source PHP web framework, created by Taylor Otwell and intended for the development of web applications following the model–view–controller (MVC) architectural pattern and based on Symfony.

**Amazon Elastic Compute Cloud (Amazon EC2)** is a web service that provides secure, scalable compute capacity in the cloud.

**Amazon Relational Database Service (Amazon RDS)** makes it easy to set up, operate, and scale a relational database in the cloud.

**Unit testing** is a software testing method by which individual units of source code — sets of one or more computer program modules.

**Code coverage** is a metric that can help you understand how much of your source is tested.

**Static code analysis** is a method of debugging by examining source code before a program is run.

**Acceptance Testing** is to assess whether the Product is working for the user, correctly for the usage.

1.  **Installing Docker on AWS EC2**

I use Amazon AWS EC2 to create virtual machines / EC2 instances (VMs). I created a very simple EC2 instance with the next characteristics:

-   Image: Amazon Linux AMI 2018
-   Instance: t2.micro
-   Security groups: I opened HTTP, HTTPS ports. And I opened TCP 50000, and TCP 49001.

I connected to my EC2 instance through SSH, and then I installed Docker in my EC2 instance:

```
sudo yum update -ysudo yum install -y dockersudo service docker startsudo usermod -a -G docker ec2-user
```

2\. **Installing Jenkins through Docker**

Then, I had to create a container to deploy Jenkins. At the first time, I created a simple container with the _docker run -d -p 49001:8080 -p 50000:50000_ \-v $HOME/jenkins\_home:/var/jenkins\_home --_name jenkins jenkins/jenkins_ command (DO NOT use this command, this is just an example). And I started to use Jenkins.

The book mentioned you need to have a Jenkins master and some Jenkins slaves. For simplicity, I decided to have only one Jenkins container, running as master and also as slave (running jobs). In order to do that, I had to create a Dockerfile with Jenkins, but also with PHP 7.4 (in order to run Laravel), Composer, and other packages. The final Dockerfile is presented next:

```
FROM jenkins/jenkins:2.257-centos7USER rootRUN yum install epel-release -yRUN rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpmRUN yum --enablerepo=remi-php74 install php php-mbstring php-xml php-pdo php-pdo_mysql php-xdebug -yRUN yum update -y RUN cd /tmpRUN curl -sS https://getcomposer.org/installer | phpRUN mv composer.phar /usr/local/bin/composer
```

Then I created a new Docker image (I called it jenkins-php), and I run a docker container:

```
docker image build -t jenkins-php .docker run -d -p 49001:8080 -p 50000:50000 \  -v /var/run/docker.sock:/var/run/docker.sock \  --name jenkins jenkins-php
```

In order to execute some CI tasks (such as acceptance tests), I had to install Docker in Docker. So I got inside the Jenkins container, and I installed Docker inside the Jenkins container, I used the next commands:

```
docker exec -it -u root jenkins bashyum install -y yum-utilsyum-config-manager \    --add-repo \    https://download.docker.com/linux/centos/docker-ce.repoyum install -y docker-ce docker-ce-cli containerd.iodocker --versiondocker run hello-worldchmod 666 /var/run/docker.sockexit
```

Then I used the next line, to catch the Jenkins initial password (in order to configure Jenkins from the browser)

```
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Then I opened my AWS EC2 instance from the browser (Accessing the 49001 port). Something like this: [http://EC2DNS.amazonaws.com:49001/](http://ec2dns.amazonaws.com:49001/)

I installed the recommended plugins and I installed another plugin called “GitHub integration”

3\. **A new Jenkins Item**

Before creating a new Jenkins item, I designed a very basic Laravel 8 application, which can be found here: [https://github.com/danielgara/laravel8cd](https://github.com/danielgara/laravel8cd)

![](https://miro.medium.com/max/700/1*qRO2HFgXX5py2-hFPKV2zQ.jpeg)

Figure 1. Laravel8cd project

Then, I created a new item, I called it “laravel” and I selected the “Pipeline” option.

I selected the “Pipeline script from SCM”, I choose “Git”, and I put my GitHub project address [https://github.com/danielgara/laravel8cd.git](https://github.com/danielgara/laravel8cd.git) — you can use that address if you want, or you can Fork my GitHub project.

That project already contains a Jenkins file with all the pipeline stages. In the next sections, I will explain most of those stages. Including what I did in order to: (i) run unit tests, (ii) run code coverage test, (iii) run static code tests, and (iv) run acceptance tests. The next figure shows what I implemented.

![](https://miro.medium.com/max/700/1*DeKhiXImPk_s2EPff21DvQ.jpeg)

Figure 2. Running a Job about a Laravel 8 project in Jenkins

4\. **Running unit tests**

In order to automatically run unit/feature tests, I create a simple test in my Laravel project. That test can be found here: /test/Feature/RouteTest.php

```
<?phpnamespace Tests\Feature;use Illuminate\Foundation\Testing\RefreshDatabase;use Illuminate\Foundation\Testing\WithFaker;use Tests\TestCase;class RouteTest extends TestCase{    public function testHome()    {        $this->get('/')            ->assertSee('Home');    }public function testLogin()    {        $this->get('/login')            ->assertSee('Remember Me');    }}
```

That test verifies that the “/” route displays a “Home” text. And the “/login” route displays a “Remember me” text. These are very simple tests; here you can apply some Class tests, model Tests, controllers, etc, and so on.

Then I developed an early version of the Jenkins file to run the unit tests:

```
pipeline { agent any stages {        stage("Build") {            steps {                sh 'php --version'                sh 'composer install'                sh 'composer --version'                sh 'cp .env.example .env'                sh 'php artisan key:generate'            }        }        stage("Unit test") {            steps {                sh 'php artisan test'            }        }  }}
```

The previous code builds the project and runs the “PHP artisan test” command which executes the unit tests (that command executes PHPUnit /a PHP framework for unit testing / which is already included in Laravel projects). More info here: [https://phpunit.de/](https://phpunit.de/)

I build the Laravel item (from Jenkins), and it shows me a message saying that it passed the “Declarative: Checkout SCM, Build, Unit test” stages. Job marked everything in green.

5. **Improved version with Jenkins credentials**

The previous pipeline didn’t change the .env variables (database username, password, host). So I used the Jenkins credentials (from the Jenkins admin panel), and I created four credentials. More info here: [https://www.jenkins.io/doc/book/using/using-credentials/](https://www.jenkins.io/doc/book/using/using-credentials/)

**Notice:** for simplicity, I implemented the project database in Amazon RDS. More info here: [https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER\_CreateDBInstance.html](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)

The update Jenkins pipeline is presented:

```
pipeline {    agent any    stages {        stage("Build") {            environment {                DB_HOST = credentials("laravel-host")                DB_DATABASE = credentials("laravel-database")                DB_USERNAME = credentials("laravel-user")                DB_PASSWORD = credentials("laravel-password")            }            steps {                sh 'php --version'                sh 'composer install'                sh 'composer --version'                sh 'cp .env.example .env'                sh 'echo DB_HOST=${DB_HOST} >> .env'                sh 'echo DB_USERNAME=${DB_USERNAME} >> .env'                sh 'echo DB_DATABASE=${DB_DATABASE} >> .env'                sh 'echo DB_PASSWORD=${DB_PASSWORD} >> .env'                sh 'php artisan key:generate'                sh 'cp .env .env.testing'                sh 'php artisan migrate'            }        }        stage("Unit test") {            steps {                sh 'php artisan test'            }        }    }}
```

6\. **Running code coverage test**

Running the code coverage test was very simple, PHPUnit already supports code coverage test. I updated the Jenkins pipeline and I included a new stage:

```
        stage("Code coverage") {            steps {                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"            }        }
```

I build the Laravel item (from Jenkins), and it shows me a message saying that it passed the “Declarative: Checkout SCM, Build, Unit test, Code coverage” stages. Job marked everything in green.

The code coverage report was found in a route like this: [http://EC2DNS.compute-1.amazonaws.com:49001/job/Laravel/21/execution/node/3/ws/reports/coverage/dashboard.html](http://ec2dns.compute-1.amazonaws.com:49001/job/Laravel/21/execution/node/3/ws/reports/coverage/dashboard.html) (you can navigate from your job workspace). That report showed some classes with no tests (0%), and other with (80%, 50% covered). I didn’t implement a minimum of code coverage restriction, but it is a good idea to define a minimum.

7\. **Static code analysis**

In order to run static code analysis I installed two packages in my Laravel project:

-   LaraStan [https://github.com/nunomaduro/larastan](https://github.com/nunomaduro/larastan)
-   PHP CodeSniffer [https://github.com/squizlabs/PHP\_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer)

For LaraStan I created a phpstan.neon file in the project root folder with the next content

```
includes: — ./vendor/nunomaduro/larastan/extension.neonparameters:paths: — app/Http/Controllers/level: 6checkMissingIterableValueType: false
```

Basically, it checks all my controller files with some predefined rules. More info here: [https://github.com/nunomaduro/larastan](https://github.com/nunomaduro/larastan) — those rules included to check valid comments before each method, some brackets spaces, and many more.

I also installed PHP CodeSniffer, and I modified the phpcs.xml file in the project root folder with the next content:

```
<?xml version=”1.0"?><ruleset name=”PSR2"> <description>The PSR2 coding standard.</description> <rule ref=”PSR2"/> <file>app/Http/Controllers/</file> <exclude-pattern>vendor</exclude-pattern> <exclude-pattern>resources</exclude-pattern> <exclude-pattern>database/</exclude-pattern> <exclude-pattern>storage/</exclude-pattern> <exclude-pattern>node_modules/</exclude-pattern></ruleset>
```

I only wanted to check the controllers (but obviously you can apply it to models, and many other files). It uses a PSR2 coding standard that check camelCase in the method names, some spaces, and many more. More info here: [https://www.php-fig.org/psr/psr-2/](https://www.php-fig.org/psr/psr-2/)

Finally, I update the Jenkins pipeline and I included two new stages:

```
        stage("Static code analysis larastan") {            steps {                sh "vendor/bin/phpstan analyse --memory-limit=2G"            }        }        stage("Static code analysis phpcs") {            steps {                sh "vendor/bin/phpcs"            }        }
```

I build the Laravel item (from Jenkins), and it shows me a message saying that it passed the “Declarative: Checkout SCM, Build, Unit test, Code coverage, Static code analysis larastan, Static code analysis phpcs” stages. Job marked everything in green.

8\. **Docker inside Docker (running a container of the Laravel project)**

Docker in Docker was challenging, I faced many issues but in the end I did it. After the static code tests, I had to implement the “Acceptance tests”. The acceptance test requires a staging version of the application running. And then, you run some tests over that running version.

It means, you need to execute some commands, to build a new docker image (inside the Jenkins container) based on the GitHub current code, and run a new container, and test that container.

I created a Docker Hub account, in order to put those images online. And avoid to create a Docker registry. In part 2 of this story, I already explained how to installed Docker in Docker. So, the next part was to add two new stages to the Jenkins pipeline:

```
        stage("Docker build") {            steps {                sh "docker rmi danielgara/laravel8cdpart2"                sh "docker build -t danielgara/laravel8cdpart2 --no-cache ."            }        }        stage("Docker push") {            environment {                DOCKER_USERNAME = credentials("docker-user")                DOCKER_PASSWORD = credentials("docker-password")            }            steps {                sh "docker login --username ${DOCKER_USERNAME} --password ${DOCKER_PASSWORD}"                sh "docker push danielgara/laravel8cdpart2"            }        }
```

Those stages build a new image based on the GitHub current code, and connect with Docker Hub (through the use of two Jenkins credentials that I registered), and then, it pushes the new image to my Docker Hub account. You can found that image here: [https://hub.docker.com/r/danielgara/laravel8cd](https://hub.docker.com/r/danielgara/laravel8cd)

Having that image online allows me to create new containers easily, and to apply the acceptance tests.

9\. **Acceptance tests**

I run two types of acceptance tests. With CURL command, and with co-deception package.

**CURL command was a first try** to check if I was able to run a new container and apply some tests to that running container.

I create a very simple functionality in laravel to sum two numbers, check the next code (which was added in the /app/Http/Controllers/HomeController.php):

```
    /**     * Return sumatory     *     * @return int     */    public function sum(int $num1, int $num2)    {        return $num1+$num2;    }
```

If you open the browser and go to /public/sum/2/4 it displays 6.

Then I create an acceptance\_test.sh file to use the CURL command and to test the previous functionality. This file obtains the running container IP address and invoke the /public/sum/4/2. Then, it compares the result with the number “6”. If both are equals, the test passed, otherwise it shows an error.

```
#!/bin/bashres=$(curl -s -w “%{http_code}” $(docker inspect -f ‘{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}’ laravel8cd)/public/sum/4/2)body=${res::-3}if [ $body != “6” ]; then echo “Error”fi
```

I updated the Jenkins pipeline with the next two stages:

```
        stage("Deploy to staging") {            steps {                sh "docker run -d --rm -p 80:80 --name laravel8cd danielgara/laravel8cd"            }        }        stage("Acceptance test curl") {            steps {                sleep 20                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"            }            post {                always {                    sh "docker stop laravel8cd"                }            }        }
```

It runs a container based on the update docker hub image, then it executes the acceptance\_test.sh file (but before, it waits 20 seconds to allows docker to create the container successfully). At the end of the stage and always, it stops the container.

Finally, the **second acceptance test** was with the use of PHP package called PHP co-deception.

I installed PHP co-deception [https://codeception.com/for/laravel](https://codeception.com/for/laravel) — and I created a very simple acceptance test (which can be found on /tests/acceptance/RegisterCest.php):

```
<?phpclass RegisterCest{    public function tryToTest(AcceptanceTester $I)    {        $I->amOnPage('/');        $I->click('Register');        $I->see('Confirm Password');    }}
```

This acceptance test checks that the “/” route contains a link called Register, and it clicks that link; therefore, it checks that the new loaded page contains a text called “Confirm password”.

I updated the Jenkins pipeline with the final stage:

```
       stage("Acceptance test codeception") {            steps {                sh "vendor/bin/codecept run"            }            post {                always {                    sh "docker stop laravel8cd"                }            }        }
```

That stage runs that co-deceptions tests, and at the end stops the container.

I build the Laravel item (from Jenkins), and it shows me a message saying that it passed the “Declarative: Checkout SCM, Build, Unit test, Code coverage, Static code analysis larastan, Static code analysis phpcs, Docker build, Docker push, Deploy to staging, Acceptance test curl, Acceptance test co-deception” stages. Job marked everything in green.

_The final result is presented in Figure 2._

**Summary**

This story explains how to apply some CI/CD principles in Laravel projects (with the use of Jenkins and Docker). The example was very basic in order to introduce some principal concepts. But you have to consider that there are many risks with the presented example. Jenkins slaves must be separated. Not always is possible to have the database in Amazon RDS, some authors didn’t recommend using Docker in Docker, and so on. However, for novice Laravel CI/CD developers (such as me), I think this is a good introduction.

**Note:** I plan to develop a second part of this story regarding Chapters 6 to 8 of the “Continuous Delivery with Docker and Jenkins — Second edition” book.

**UPDATE:** part 2 available here: [https://medium.com/@danielgara/configure-laravel-8-for-ci-cd-with-jenkins-and-github-part-2-80d9c337f0e4](https://medium.com/@danielgara/configure-laravel-8-for-ci-cd-with-jenkins-and-github-part-2-80d9c337f0e4)

![](https://miro.medium.com/max/700/0*Piks8Tu6xUYpF4DU)

👋 [**Join FAUN today and receive similar stories each week in your inbox!**](https://faun.dev/join) ️ **Get your weekly dose of the must-read tech stories, news, and tutorials.**

**Follow us on** [**Twitter**](https://twitter.com/joinfaun) 🐦 **and** [**Facebook**](https://www.facebook.com/faun.dev/) 👥 **and** [**Instagram**](https://instagram.com/fauncommunity/) 📷 **and join our** [**Facebook**](https://www.facebook.com/groups/364904580892967/) **and** [**Linkedin**](https://www.linkedin.com/company/faundev) **Groups** 💬

[

![](https://miro.medium.com/max/700/1*_cT0_laE4iPcqW1qrbstAg.gif)

](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumbanner)

## If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇