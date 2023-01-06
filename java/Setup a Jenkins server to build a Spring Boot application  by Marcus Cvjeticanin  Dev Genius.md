![](https://miro.medium.com/max/700/1*9r2wqaWCJbX3NDJbr3aw9g.jpeg)

In this tutorial we will go through the steps to setup a Jenkins server using Docker and Docker Compose to be able to build a Spring Boot application from a GitHub repository.

## Jenkins Configuration as Code (JCasC)

We will start by looking at the Jenkins Configuration as Code (JCasC) part so we will clone the repository [https://github.com/mjovanc/medium-jenkins-casc](https://github.com/mjovanc/medium-jenkins-casc) and we will go through each file respectively what they do and why we need them. There is a lot to unpack in this tutorial so grab a refreshing beverage once in a while.

So why do we need something called Jenkins Configuration as Code? Just imagine the scenario that you need to install a Jenkins server manually and have to input the configuration inside the web UI of Jenkins. What if something happens to the server and we need to replicate a new server with Jenkins? Then we would need to manually add them again inside the web UI and install the plugins manually. This is something that takes a long time if you have a bigger project and could lead to a lot of problems with delivery of the product you are building.

I actually tried to find out about JCasC but there was **problem finding good tutorials**, thorough ones that goes through every step needed to do this. So my goal is to show you now what you need in a **basic way**. I will do the best I can. If there is some mistakes please add a comment and I will answer as quick as I can and correct the mistake.

So in the repository we will start by opening up the file **config/jenkins.yml**:

```
jenkins:  systemMessage: "Welcome to Jenkins for the Spring Project!"  numExecutors: 4  mode: NORMAL  scmCheckoutRetryCount: 3  labelString: "mjovanc"  # we need to specify something here that we will define in our pipeline jobs, choose whatever name you want  primaryView:    all:      name: Alltool:  jdk:    installations:    - name: "OpenJDK 16"      properties:      - installSource:          installers:          - zip:              subdir: "/var/jenkins_home/tools/hudson.model.JDK/OpenJDK_16/jdk-16.0.1"              url: "https://download.java.net/java/GA/jdk16.0.1/7147401fd7354114ac51ef3e1328291f/9/GPL/openjdk-16.0.1_linux-x64_bin.tar.gz"jobs:  - script: |      job('seedjob-mjovanc') {        description('Set up all jobs')                  scm {          git {            branches('*/master')            remote {              credentials('mjovanc-jenkins-key')              url('git@github.com:mjovanc/medium-mjovanc-job-dsl.git')            }          }        }        steps {          dsl {            external('*.groovy')            removeAction('DELETE')            ignoreExisting(ignore = false)            removeViewAction('DELETE')          }        }        triggers {          cron('@midnight')        }      }
```

There are two main sections here worth pointing out. We have **jenkins**, **tool** and **jobs**.

**jenkins** is the general configuration of the server such like how many executors we will need and so on.

**tool** is about what kind of extra software we needs to be installed to be able to build our software. I have added OpenJDK16 since OpenJDK11 that Jenkins uses will not work.

**jobs** section defines what kind of jobs should exist initially when starting Jenkins. So we add here a Groovy script to setup a job which will fetch more Groovy DSL jobs every midnight from an external repository which you could clone from here: [https://github.com/mjovanc/medium-mjovanc-job-dsl](https://github.com/mjovanc/medium-mjovanc-job-dsl). So if you would do any changes to the DSL jobs in the external repository it will change during midnight.

Now lets look at our **Dockerfile**:

```
FROM jenkins/jenkins:lts-jdk11USER rootCOPY plugins.txt /usr/share/jenkins/ref/plugins.txtRUN /usr/local/bin/install-plugins.sh << /usr/share/jenkins/ref/plugins.txtRUN apt-get update -yRUN apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common docker.ioRUN docker --versionRUN usermod -aG docker jenkinsUSER jenkins
```

Here we define our own **Dockerfile** since we need to install Docker inside our Docker container for it to be able to build Docker images on Jenkins.

Our **docker-compose.yml** file:

```
version: '3.7'services:  jenkins:    build: .    privileged: true    user: root    restart: on-failure:10    ports:      - 8080:8080    container_name: mjovanc-jenkins    volumes:      - ./:/jcasc      - ~/apps/jenkins:/var/jenkins_home      - /var/run/docker.sock:/var/run/docker.sock    environment:      - CASC_JENKINS_CONFIG=/jcasc/configs/jenkins.yml      - JENKINS_OPTS='--prefix=/'    logging:      driver: 'json-file'      options:        max-file: '3'        max-size: '10m'
```

This file defines that we will build from our own **Dockerfile** and set the service name as **jenkins**. We also map some volumes needed such as all the JCasC configuration and the Docker UNIX socket from our host operating system to be able to use that inside the Docker container.

We also set two environment variables. **CASC\_JENKINS\_CONFIG** which points to our **jenkins.yml** file we defined, so Jenkins will know that it should be loading that file during startup. Also we set a prefix that Jenkins should have on the URL with **JENKINS\_OPTS**. So the URL will be directly on **/**.

Some code that we need to run after first setting up the Jenkins server inside the Web UI of Jenkins. We go to Script Console and enter:

```
def pluginList = new ArrayList(Jenkins.instance.pluginManager.plugins)pluginList.sort { it.getShortName() }.each{  plugin ->     println ("${plugin.getDisplayName()} (${plugin.getShortName()}): ${plugin.getVersion()}")}
```

This will give us a long list of plugins that is installed so we can define that in a **plugins.txt** file so the next time we need to setup a new Jenkins server we will just load from that file when we run **docker-compose up**.

Next thing you will need to do is to change the name of the file **your-domain.com** to your own. This file is needed to have in order to setup the **NGINX** configuration as our NGINX will act as a **reverse proxy** to the Jenkins server. This is how the file looks like:

```
upstream jenkins {  keepalive 32; # keepalive connections  server 127.0.0.1:8080; # jenkins ip and port}# Required for Jenkins websocket agentsmap $http_upgrade $connection_upgrade {  default upgrade;  '' close;}server {    listen 80;    listen [::]:80;    return 301 https://$host$request_uri;}server {    server_name your-domain.com; # managed by Certbot   # this is the jenkins web root directory    # (mentioned in the /etc/default/jenkins file)    root            /root/apps/jenkins/war/;    access_log      /var/log/nginx/jenkins.access.log;    error_log       /var/log/nginx/jenkins.error.log;    # pass through headers from Jenkins that Nginx considers invalid    ignore_invalid_headers off;    location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {        # rewrite all static files into requests to the root        # E.g /static/12345678/css/something.css will become /css/something.css        rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;    }    location /userContent {        # have nginx handle all the static requests to userContent folder        # note : This is the $JENKINS_HOME dir        root /root/apps/jenkins/;        if (!-f $request_filename){            # this file does not exist, might be a directory or a /**view** url            rewrite (.*) /$1 last;            break;        }        sendfile on;    }   location / {        sendfile off;        proxy_pass         http://jenkins;        proxy_redirect     default;        proxy_http_version 1.1;        # Required for Jenkins websocket agents        proxy_set_header   Connection        $connection_upgrade;        proxy_set_header   Upgrade           $http_upgrade;        proxy_set_header   Host              $host;        proxy_set_header   X-Real-IP         $remote_addr;        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;        proxy_set_header   X-Forwarded-Proto $scheme;        proxy_max_temp_file_size 0;        #this is the maximum upload size        client_max_body_size       10m;        client_body_buffer_size    128k;        proxy_connect_timeout      90;        proxy_send_timeout         90;        proxy_read_timeout         90;        proxy_buffering            off;        proxy_request_buffering    off; # Required for HTTP CLI commands        proxy_set_header Connection ""; # Clear for keepalive    }    listen [::]:443 ssl ipv6only=on; # managed by Certbot    listen 443 ssl; # managed by Certbot    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem; # managed by Certbot    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem; # managed by Certbot    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot}
```

You need to change to your own domain in that file and the next file I will demonstrate will use that file to copy to its place after installing the NGINX server. This is the script **setup\_start.sh**:

```
#!/bin/bash# # Use this file to setup the environment for Jenkins# and run the server# VariablesJENKINS_DOMAIN=<your domain> # this domain should be the name of your nginx configuration file as wellNGINX_SSL_EMAIL=<your email># Installing necessary dependenciessudo apt-get update -ysudo apt-get install -y \    ca-certificates \    curl \    gnupg \    lsb-release \    nginx \    certbotapt-get install -y python3-certbot-nginx# Setup NGINXsudo mkdir -p /var/log/nginx/jenkinssudo cp $JENKINS_DOMAIN /etc/nginx/sites-available/$JENKINS_DOMAINsudo ln -s /etc/nginx/sites-available/$JENKINS_DOMAIN /etc/nginx/sites-enabled/# SSLsudo certbot --nginx -d $JENKINS_DOMAIN --non-interactive --agree-tos -m $NGINX_SSL_EMAIL# Reloadnginx -t && nginx -s reload# Add Docker’s official GPG key:curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg# Setup the stable repositoryecho \  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null# Install Docker Enginesudo apt-get update -ysudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose# Start Docker and orchestratedocker-compose up -d
```

This script is to install all the necessary dependencies for NGINX, Certbot, Docker and Docker Compose and then run Jenkins. You need to edit this file in the top by changing the value of **JENKINS\_DOMAIN** and **NGINX\_SSL\_EMAIL** to your own domain and email address.

I take for granted that you already setup a Linux instance (VM) and pointed your own domain with an DNS A record to it.

If you would like to see some output you could remove the **\-d** flag after **docker-compose up**. That could be good to use initially since we could follow the process of the orchestration to spot if we made any errors. However, when you are done you could add it back again.

Now we are ready to start the application, and start the application is done with first setting the executable permissions to the file setup\_start.sh with the following command:

```
chmod +x ./setup_start.sh
```

And as we always do when running a shell script we simply run it with:

```
./setup_start.sh
```

Great! Now it’s up and running!

Now we just need to do a final touch to Jenkins for it to work with the configuration part. We need first to generate a SSH key that we will use. Use the command below to generate one:

```
ssh-keygen -t rsa
```

Follow the instructions given by the CLI and select a proper name, perhaps jenkins-github? Then we need to just add that key to our ssh agent on our computer:

```
eval "$(ssh-agent -s)"ssh-add <path to your ssh key>
```

Now we will go to our GitHub account and add the content of our **public** (.pub) SSH Key here [https://github.com/settings/keys](https://github.com/settings/keys).

Finally for the JCasC part there is one manual step we need to do in the Jenkins Web UI to add our SSH credential that is needed for the jobs to load to Jenkins. If the repository is public you could skip this and remove the credential() method inside the DSL jobs. **Important** to note is that we need to add the corresponding credentials name to what says in the configuration files, see the job inside **jenkins.yml** how it looks.

![](https://miro.medium.com/max/700/1*k8O99K0bH7IPBxrT-FI1qQ.png)

Check out this tutorial on how to add credentials on Jenkins: [https://www.jenkins.io/doc/book/using/using-credentials/](https://www.jenkins.io/doc/book/using/using-credentials/)

Perhaps a good idea is to take some beverage now.

## Jenkins DSL jobs

That was a lot of files to go through. Brace yourselves, it will be more. We will now go through the DSL jobs how that looks like.

We will start by looking at the Spring API DSL job:

```
//---------------------------------------------------------------------------------// spring_api_job.groovy//---------------------------------------------------------------------------------// A pipeline job for the Spring Boot API application//---------------------------------------------------------------------------------pipelineJob('Spring-API') {    description('Build Spring Boot API')    triggers {        githubPush()    }definition {        cpsScm {            scm {                git {                    remote {                        url('git@github.com:mjovanc/medium-spring-api.git')                        credentials('mjovanc-jenkins-key')                    }branches('master')                    scriptPath('jenkins/main.jenkinsfile')                    extensions {                        cleanBeforeCheckout()                    }                }            }        }    }}
```

Here we define the type **pipelineJob** with its name and description. We also set a trigger method as **githubPush()** which will send us updates to Jenkins whenever we get a new commit. So we need to setup a **webhook**, but we will do that after we go through the files. Then we have a definition section that we describe what repository and what credentials that will be used to authenticate ourselves. We also specify what branch we will checkout and which **Jenkinsfile** we will be using in this pipeline.

As an extra DSL job we will add a build monitor view:

```
//---------------------------------------------------------------------------------// spring_build_monitor.groovy//---------------------------------------------------------------------------------// Defines build monitor views for Spring API//---------------------------------------------------------------------------------buildMonitorView('Spring-API') {    description('Spring API master')    recurse()    jobs {        name('master')    }configure { project ->         (project / config / displayCommitters ).value = true        (project / config / buildFailureAnalyzerDisplayedField).value = Name    }}
```

We could add many more pipelines with different Groovy scripts for different projects in this DSL repository we have and Jenkins will pick them up and add it.

One thing to note is that when Jenkins do these nightly checks if we changed something to the scripts or add a new one we need to manually go in to Jenkins and **approve** them.

![](https://miro.medium.com/max/700/1*kDMTa-vGRy9IQEZlkBtPaQ.jpeg)

Example how it looks when awaiting approval of script

Now we will setup the **webhook** in **GitHub** we go to the repository settings and go to webhooks and press add. Now we will add the payload URL, this makes sure that Jenkins get the push and registers it that something happend. Add the following:

```
https://your-jenkins-domain.com/github-webhook/
```

Select the option, **Let me select individual events** and select **Push** and save.

## Setting up Spring Boot with Jenkins and Docker

Now we are going to setup Spring Boot with Jenkins and Docker.

We are going to use the blog application I previously built for another tutorial: [https://medium.com/@mjovanc/spring-boot-with-postgresql-and-hibernate-part-2-5406f3c93665](https://medium.com/@mjovanc/spring-boot-with-postgresql-and-hibernate-part-2-5406f3c93665). But I added a new repository to keep them apart: [https://github.com/mjovanc/medium-spring-jenkins-docker](https://github.com/mjovanc/medium-spring-jenkins-docker).

We first need to start with create a directory called **jenkins** in our root of the repository then create a new file **main.jenkinsfile** inside it and place this code:

```
def SPRING_VERSION = "X.X.X"pipeline {    agent {        node {            label 'spring'        }    }    tools {        jdk 'OpenJDK 16'    }    stages {        stage('Clean') {            steps {                echo 'Cleaning leftovers from previous builds'                sh "chmod +x -R ${env.WORKSPACE}"                sh './gradlew clean'            }        }        stage('Compile') {            steps {                echo 'COMPILING JAVA'                sh './gradlew assemble'            }        }        stage('Static Code Analysis') {            steps {                echo 'Running Static Code Analysis'                echo 'Checking style'                sh './gradlew checkstyleMain'                echo 'Checking duplicated code'                sh './gradlew cpdCheck'                echo 'Checking bugs'                sh './gradlew spotbugsMain'                echo 'Checking code standard'                sh './gradlew pmdMain'            }        }        stage('Unit Test') {            steps {                echo 'Running all unit tests'                sh './gradlew test -Dspring.profiles.active=test'            }        }        /* stage('Coverage Test') {            steps {                echo 'Coverage Test'            }        } */        stage('Build Docker Image') {            steps {                script {                    echo 'Building Docker Image'                    // Getting project version                    SPRING_VERSION = sh(                            script: './gradlew -q printVersion',                            returnStdout: true).trim()                    echo "CURRENT VERSION: ${SPRING_VERSION}"                    sh "docker build -t spring-api:${SPRING_VERSION} ."                }            }        }        stage('Publish Docker Image') {            steps {                script {                    echo 'Publish Docker Image to Docker Hub'                    withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_TOKEN')]) {                        sh """                            docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_TOKEN                            docker image tag spring-api:${SPRING_VERSION} spring/spring-api:${SPRING_VERSION}                            docker image tag spring-api:${SPRING_VERSION} spring/spring-api:latest                            docker push spring/spring-api:${SPRING_VERSION}                            docker push spring/spring-api:latest                        """                    }                }            }        }    }}
```

This file defines all the pipeline stages the application should go through in Jenkins. We also define the agent with the corresponding label we already defined before in our **jenkins.yml** file in **JCasC** and the tool that we will use in this pipeline which is **OpenJDK 16**.

If there is some stage that doesn’t work at the moment, perhaps the static code analysis stage, comment that out and you could fix it later.

In the **Docker** stage where we will publish it asks for credentials, you need to add the following to your Jenkins credentials page as before but with these variables: **DOCKER\_HUB**, **DOCKER\_HUB\_USER** and **DOCKER\_HUB\_TOKEN**.

We need to change our build.gradle file as well for the pipeline to work:

```
buildscript {   repositories {      mavenCentral()   }   dependencies {      classpath("org.springframework.boot:spring-boot-gradle-plugin:2.5.2.RELEASE")   }}plugins {   id 'org.springframework.boot' version '2.5.2'   id 'io.spring.dependency-management' version '1.0.11.RELEASE'   id 'java'   id 'war'   id "com.dorongold.task-tree" version "2.1.0"   id 'checkstyle'   id 'pmd'   id 'de.aaschmid.cpd' version '3.3'   id "com.github.spotbugs" version "5.0.4"}group = 'com.mjovanc'version = '0.0.1'sourceCompatibility = '16'repositories {   mavenCentral()}dependencies {   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'   implementation 'org.springframework.boot:spring-boot-starter-web'   implementation 'org.springframework.session:spring-session-core'   implementation group: 'org.springframework.data', name: 'spring-data-commons', version: '2.5.2'   implementation group: 'org.springdoc', name: 'springdoc-openapi-ui', version: '1.5.9'   implementation group: 'com.h2database', name: 'h2', version: '1.4.200'   runtimeOnly 'org.postgresql:postgresql'   providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'   testImplementation 'org.springframework.boot:spring-boot-starter-test'}test {   useJUnitPlatform()}war {   enabled = true   archiveClassifier = ''}//---------------------------------------------------------------------------------// STATIC CODE ANALYSIS CONFIGURATION//---------------------------------------------------------------------------------checkstyle {   toolVersion = '8.12'   ignoreFailures = false   configFile = file("${projectDir}/gradle/static-code-analysis/checkstyle/checkstyle.xml")}pmd {   toolVersion = '6.7.0'   ignoreFailures = false   ruleSetFiles = files("${projectDir}/gradle/static-code-analysis/pmd/ruleset.xml")   ruleSets = []   rulesMinimumPriority = 3}cpd {   language = 'java'   toolVersion = '6.1.0'   minimumTokenCount = 200 // approximately 5-10 lines}cpdCheck {   reports {      text.enabled = true      xml.enabled = false   }   ignoreAnnotations = true   source = sourceSets.main.allJava // only java, groovy and scala classes in 'main' sourceSets}spotbugsMain {   reports {      html {         required = true         outputLocation = file("$projectDir/build/reports/spotbugs/main/spotbugs.html")         stylesheet = 'fancy-hist.xsl'      }   }   excludeFilter = file("${projectDir}/gradle/static-code-analysis/spotbugs/spotbugs-exclude.xml")}//---------------------------------------------------------------------------------// TASKS//---------------------------------------------------------------------------------tasks.withType(Checkstyle) {   reports {      xml.enabled false      html.enabled true   }}tasks.withType(Pmd) {   reports {      xml.enabled false      html.enabled true   }}task printVersion {   doLast {      println project.version   }}
```

This file have a lot of things I wont be covering in this tutorial, but there are some things that needs to be said. We have defined a task called **printVersion** that we use in our **Jenkinsfile** to get the current version. We also have a **war** block:

```
war {   enabled = true   archiveClassifier = ''}
```

This makes sure that we wont get two **.war** files and only one so our **Dockerfile** could pick up just the first one, because if we change the version it wont be good to hard code the version in this:

```
# Using Tomcat 9.0 since the latest doesn't work with Spring BootFROM tomcat:9.0-jdk16-openjdkARG WAR_FILE=build/libs/spring-*.warRUN rm -rf /usr/local/tomcat/webapps/*COPY ${WAR_FILE} /usr/local/tomcat/webapps/ROOT.warEXPOSE 8080CMD ["catalina.sh", "run"]
```

That is pretty much it. I might have forgot to mention some things so please ask questions if you don’t understand or if you encountering any problems. I will be updating this whenever I see something that needs clarification.