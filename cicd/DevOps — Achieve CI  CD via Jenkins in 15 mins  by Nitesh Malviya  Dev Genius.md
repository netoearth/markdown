![](https://miro.medium.com/max/1400/1*p1i2Ssnj74yNosBWSvG1vw.png)

CI / CD (Continuous Integration / Continuous Delivery) is one of the essential parts of the modern application development environment. CI/CD is a method to frequently integrate and deliver apps to end-users by introducing automation into the stages of application development. This approach helps development and operations teams to work together in an agile way. Although DevOps, Agile, and CI / CD are three different things, they are glued together somehow to produce a seamless application development and delivery experience. First, let’s understand the meaning of CI and CD, and then I will walk you through step by step to setup CI / CD pipeline via Jenkins.

## **CI: Continuous Integration**

The CI part of CI / CD is simply a _regular integration of development code_ to the master repository. The prime purpose of the CI is to enable early detection of integration bugs which might later result in delays in delivery. In the Agile world, every developer develops their code in their own feature branches which they periodically integrate to the master branch. CI along with automation ensures that this integration or merging would not break existing functionality. Automation is an essential element of CI and without automation, you cannot embrace the full power of CI. You can achieve the following goals by applying the CI pipeline to your application

1.  Automatic code quality scans
2.  Publish test coverage reports
3.  Automatic code builds

## CD: Continuous Deployment

The CD part of CI / CD is simply _releasing the code regularly_ for customers. CD minimizes the efforts of deploying the new code and along with CI, it minimizes the risk of potential failures after release. In today’s Microservices world, regular deployments and monitoring are hard to manage. CD makes it easy and frictionless. CI / CD together produce the best ROI in Agile software development. Continuous delivery is also one of the code ideology of Agile software development as per the [Agile Manifesto](https://agilemanifesto.org/principles.html).

> _“ Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.”_

## Application development without CI / CD — Traditional approach

Let’s follow with the traditional application development approach and find the pain points which can be resolve by applying CI / CD pipeline. In past, development and deployments were two separate departments in most organizations. The development team’s core focus was “_development”_ whereas the deployment team’s core focus was “_keeping the production server up and running and deploying updates with minimal downtime”_. This segregation of duties and responsibilities was the first and foremost problem in the traditional application development approach.

As per the traditional non Agile approach, the development team comprises of developers and QA basically. Their primary job is to develop and test the code on the dev server. As mentioned above, software developers tended to only focus on writing the code. Once the application is coded, it is handed over to the Q/A team. If the testers discovered “bugs” they would document the errors and send them back to the developers for correction. Once the application passed the testing stage, the development team's goal is achieved and they handover the code to the deployment team.

Deployment teams need to configure the operating system, perform database rollouts, configure network and security before deploying the code while keeping the production server alive. As the application development team mostly does not understand the production environment complication, this results in an unstable release to production and unnecessary rollouts.

CI/ CD plays a key role to overcome the problem of segregation of duties and responsibilities along with a long release cycle and manual dependency.

## Application development with CI / CD — Modern Agile approach

In the modern Agile approach with CI / CD, developers develop the code on their own feature branches and merge the code to master once complete. Once they merge the code to the master branch then CI / CD takes it from there. CI / CD tools like Jenkins receives a notification that a new code is pushed/merged to the master branch. It pulls the code from the repository and builds the required artifacts and performs the automation to test the code for integration. If the build fails then it sends the email to the developer with the issue and if the build pass then it deploys to a production server without any manual intervention.

So by CI / CD, we resolved the basic problem of responsibilities because now the entire team is responsible for both developments as well as production deployments i.e. DevOps. By this, we minimize the risk of faulty release by reducing manual interventions and coordination at every step.

## How can we achieve this :

CI / CD is not hard to achieve but it starts with changing the team’s culture where the team has to start thinking in an AGILE way. There are a lot of tools available i.e. Jenkins, CircleCI, etc by which we can achieve the goal of implementing CI / CD on the projects. Nowadays, bitbucket also provides “pipelines” in the repository which can be used the achieve CI / CD without installing any third-party software. The aim of this article is to provide the steps to install and setup Jenkins to enable CI/CD so let’s first identify the prerequisites of this implementation.

![](https://miro.medium.com/max/1400/1*8m8uLGCp5XZluzeDyn1OtQ.png)

## Prerequisites and assumptions :

I am focusing on installing and implementing CI/CD via Jenkins on Linux ubuntu machine. The steps and prerequisites might differ based on operating systems. I am also writing the article for implementing CI/CD for a LARAVEL project. Other applications CI / CD implementation might differ a bit but still, you can follow this guide to achieve significant knowledge to define CI/CD pipeline for any type of application. Please ensure to have the following prerequisites ready before start.

1.  A machine with at least 2 GB memory, 20 GB free hard disk, 2 core CPU, and Ubuntu — _For Jenkins installation and configuration_.
2.  A bitbucket account — _For managing code of the application under bitbucket repository_.
3.  A production machine — Depending upon the size and users of your application. (_Ubuntu machine with apache2 and MySQL installed which will be used as a production server._)

I am assuming that you have basic knowledge of ubuntu installation and server management here to keep this article for the intermediate level. I am using AWS EC2 machines for both Jenkins and production machines in my article. You are free to use any cloud-based servers which provide an SSH login facility with Ubuntu as an operating system. Once you have the above-required components with you then you are ready to go.

## Let’s do IT :

Please follow the following steps sequentially and at the end, you will be having the CI / CD pipeline implemented for your project.

i) Please log in to your server where you want to install Jenkins via SSH and run the following commands sequentially. Please note that you must have “sudo” rights enabled for the logged-in user.

ii) `sudo apt-get update`

iii)`sudo apt-get upgrade`

iv) `sudo apt-get install openjdk-8-jre openjdk-8-jdk` (Currently, Jenkins support both JAVA 8 and 11, but I am using JAVA 8 for simplicity)

v) `wget -q -O — [https://pkg.jenkins.io/debian-stable/jenkins.io.key](https://pkg.jenkins.io/debian-stable/jenkins.io.key) | sudo apt-key add -`

vi) `sudo sh -c ‘echo deb [http://pkg.jenkins.io/debian-stable](http://pkg.jenkins.io/debian-stable) binary/ > /etc/apt/sources.list.d/jenkins.list’`

vii) `sudo apt-get update`

viii) `sudo apt install jenkins`

ix) `sudo systemctl start jenkins` (To start Jenkins)

x) `sudo systemctl status jenkins` (To check the status of Jenkins)

xi) `sudo ufw allow 8080` (if you have ufw installed then you have to open port 8080 on ufw)

xii) `sudo ufw allow OpenSSH`

xiii) `sudo ufw enable`

xiv) `sudo ufw status`

xv) Now your Jenkins is ready to install. You can run the Jenkins via [http://your\_server\_ip\_or\_domain:8080](http://your_server_ip_or_domain:8080/) and enter the password. Please follow the process of installation and install the recommended plugins. (You need to run the command “`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`” to get the initial password to login to Jenkins.)

![](https://miro.medium.com/max/1400/1*gM0c8e21TclAn-CadvhScw.png)

xvi) Now the Jenkins machine is ready so let's install composer by running the following command over SSH. Composer is used to installing dependencies for the Laravel project

```
 → php -r “copy(‘https://getcomposer.org/installer', ‘composer-setup.php’);” → php -r “if (hash_file(‘sha384’, ‘composer-setup.php’) === ‘756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3’) { echo ‘Installer verified’; } else { echo ‘Installer corrupt’; unlink(‘composer-setup.php’); } echo PHP_EOL;” → php composer-setup.php → php -r “unlink(‘composer-setup.php’);”
```

xvii) Now the composer installation is ready. You can test this by running the command “`php composer.phar`”

xviii) Now you need to move the composer to the “bin” directory to run this via direct command. Please run the command “`sudo mv composer.phar /usr/local/bin/composer`” to move the file

xix) Now you can run the command “`composer`” on the command prompt to test the proper installation of the composer.

xx) Once composer is installed then let’s install git on Jenkins machine. Git will be used to interact with bitbucket. You can install git via the command “`sudo apt-get install git`”

xxi) Now let’s install the ZIP module which will be used to zip or unzip the folders. You can install the ZIP module by the command “`sudo apt-get install zip unzip`”

xxii) Now open the Jenkins page in the browser by [http://your\_server\_ip\_or\_domain:8080](http://your_server_ip_or_domain:8080/) and Click “Manage Jenkins” and then “Manage Plugins”-> “Available” and search for “bitbucket” plugin and install that plugin and restart.

![](https://miro.medium.com/max/1400/1*f49qngtUhX4bMfBYeXfXpA.png)

xxiii) Now we need SSH keys of Jenkins server which will be used to interact with bitbucket over SSH. To generate a new ssh key on the Jenkins server please run the following command

```
 → ssh-keygen -t rsa -C “<your email>” (It will ask for the path of the file. Provide the path or use the default one ~/.ssh/id_rsa) → cat ~/.ssh/id_rsa.pub (To copy the public file of the key and copy. Please replace the path with the path where you installed the SSH keys)
```

xxiv) Your Jenkins machine is now almost ready. Now let’s move to bitbucket and configure the repository to interact with the Jenkins machine

xxv) Open the project repository on bitbucket and click on “project settings” and then “Access Keys” and then add the above key so the Jenkins server can interact with bitbucket over SSH

![](https://miro.medium.com/max/1400/1*6T7G1FdQ-ueuAhTBjrOXng.png)

xxvi) Now on bitbucket click on “project settings”-> “webhooks” and then click “Add webhook”. Then enter the URL as “http://<your jenkins url>:8080/bitbucket-hook/ “. Click “SSL/TLS” and “Enable request history collection” and then add webhook. (This hook will push the request to Jenkins as soon as any developer pushes the code to this repository. Please note that this will send a notification to Jenkins for every push in any branch, so we have to configure Jenkins in order to only accept and process the notification of a particular branch of the repository.)

![](https://miro.medium.com/max/1400/1*LNQx6XLmKx_TPx87povjow.png)

xxvii) Now go to the Jenkins page and click on “New Item” from the left menu. Enter the name and select “Freestyle Project” and click “OK”. Then select “Source code management” and enter the repository “_SSH URL_” and then click on “_Add (under credentials) -> Jenkins_” to add SSH credentials to login on bitbucket. It will open a pop up to add the credentials. Then select “_SSH Username with private key_” in the “_kind”_ drop-down and enter the username as an email which you used during your ssh-keygen generation in the previous steps. Then run “cat ~/.ssh/id\_rsa” (to copy the private file. Please replace the path with your generated private key path) and paste in the “_private key_” section and save.

![](https://miro.medium.com/max/1400/1*yxIDsyGalQapepPP0smt8A.png)

![](https://miro.medium.com/max/1400/1*F7ij23qLKTxvO9dtGVni2Q.png)

![](https://miro.medium.com/max/1400/1*8cyAYwVHfSUWHJ7Th3aH0A.png)

xxviii) Then under the build trigger select “_Build when a change is pushed to BitBucket_” and enter the “_HTTP URL_” of the repository here.

![](https://miro.medium.com/max/1400/1*7goqFiBcFYYy4uTORwd5ng.png)

xxix) Now under the build section select “_Execute Shell_” and enter the following lines  
→ `composer install` (to install the LARAVEL dependencies)  
→ `mv .env.example .env` (you should add your production server details in .env.example file so that you can copy them during continuous integration which will be used to connect to the production server)  
→ `php artisan key:generate` (to generate the LARAVEL keys)  
→ `./vendor/bin/phpunit ./tests` (Run the automation tests)

![](https://miro.medium.com/max/1400/1*LTW2Ge40ewIq0z21uwFbYg.png)

xxx) Now your Jenkins server is ready to receive a push notification from bitbucket and do the CI part. It will pull the latest code from the master branch and install the dependencies and run the automation test case to validate the integration. So now continuous integration is done. Let’s finalize the CD part now.

xxxi) Jenkins uses the “jenkins” user to connect to any server for doing continuous deployment. So we will change our permission to that user and generate the SSH keys. Don’t get confused here. The previous SSH key was to connect to bitbucket for the CI part and this SSH key will be used to connect to the production server over SSH for the CD part. Login to Jenkins server via ssh and run the following commands.  
→ `sudo su -s /bin/bash jenkins`  
→ `ssh-keygen -f ~/.ssh/id_rsa -t rsa -b 4096 -C “devops”`  
→ `cat ~/.ssh/id_rsa.pub` (Copy ssh public key on the clipboard)  
→ Now login to the **_AWS cloud production_** **_server (the server which you will use as a production server)_** over SSH and go to “~/.ssh” folder and then run the following commands  
→`sudo nano authorized_keys`  
→Now paste the key copied from the JENKINS server and save this file  
→`sudo /etc/init.d/ssh restart` (To restart the SSH server)

xxxii) Now open the Jenkins via URL [http://your\_server\_ip\_or\_domain:8080](http://your_server_ip_or_domain:8080/) and edit the previously created pipeline by clicking on configure and then amend the following commands in the same build section. Your build section should look like the following now (Please note that the following command will also run the DB migrations and seeder to fill the database. Please amend the commands as per your use case)

```
 composer install mv .env.example .env php artisan key:generate ./vendor/bin/phpunit ./tests rsync -vrzhe “ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa” — exclude vendor/ . ubuntu@<your production server IP>:/var/www/html ssh ubuntu@<your production server IP>-i ~/.ssh/id_rsa -o StrictHostKeyChecking=no <<EOF cd /var/www/test composer install — no-dev sudo chgrp -R www-data storage bootstrap/cache sudo chmod -R ug+rwx storage bootstrap/cache php artisan migrate  php artisan db:seed EOF
```

![](https://miro.medium.com/max/1400/1*tTbEb-Cw1RdSdLoOOEqVLg.png)

By this step now your Jenkins server can run the “rsync” command to this web server. Congratulations you have implemented the CI / CD using Jenkins successfully. Now once the developer will push the code in the bitbucket master branch then it will generate a push notification to the Jenkins server via webhook. Jenkins server will pull the code from the bitbucket master branch and run the automation test case to verify the build. Once the test goes successfully then it will copy the code to the production server over SSH via “rsync” command to complete the CI / CD pipeline. You can run the production server by opening http://<your production server IP> now.

This is just the start. You can do a lot from here. You can add an automatic code quality check by installing and configuring SONACUBE on Jenkins. I will add another article to describe the step by step process to install SONARCUBE on Jenkins and enable email notifications on Jenkins.