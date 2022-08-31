[![intecontinuetitle](http://blogs.quovantis.com/wp-content/uploads/2016/02/900x240xintecontinuetitle.jpg.pagespeed.ic.4ByAF148nZ.webp "Setting a PHP Project in Jenkins")](http://blogs.quovantis.com/wp-content/uploads/2016/02/intecontinuetitle.jpg)

Continuous Integration (CI) is the process by which a developer makes his/her contribution a part of the whole and ensures that it fits well in the whole, regularly. Using CI, its easy to cross-check all the updates/check-ins into the project automatically with pre-defined validations. Their are number of CI tools available that are either free or very cheap.

Most popular CI tools available are Jenkins, Travis CI, Cruise Control, Teamcity etc.

After some deliberations and discussion I short-listed Jenkins to use for CI in one of our PHP projects. Here are my experiences for the benefit of any one else trying to accomplish the same.

First time ever when I started evaluating Jenkins, following questions came to my mind:-

A) **How to install Jenkins with basic configuration?**

B) **Plugins used to build a PHP project?**

C) **How to create a PHP project in Jenkins?**

## A. Commands used to install Jenkins in Linux:-

1.  yum install -y wget
2.  sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
3.  sudo rpm –import https://jenkins-ci.org/redhat/jenkins-ci.org.key
4.  sudo yum install jenkins
5.  sudo yum install java-1.7.0-openjdk
6.  firewall-cmd –zone=public –add-port=8080/tcp –permanent
7.  firewall-cmd –reload
8.  sudo /etc/init.d/jenkins restart
9.  systemctl restart jenkins.service
10.  Open **http://localhost:8080** URL in browser and the below screen will open. It shows Jenkins is installed and running successfully.

[![Dashboard [Jenkins]](http://blogs.quovantis.com/wp-content/uploads/2016/02/1317x642xDashboard-Jenkins.png.pagespeed.ic.6NAWHfMEZX.png "Jenkins is installed ")](http://blogs.quovantis.com/wp-content/uploads/2016/02/Dashboard-Jenkins.png)

## B. Their are number of php plugins, the following are the basic ones those can be used:-

-   [PHPUnit](http://phpunit.de/) : – allows running automated tests on code (required).
-   [PHP\_CodeSniffer](http://pear.php.net/package/PHP_CodeSniffer/) : analyse our code and detect errors in the coding standards.
-   [PHPLOC](https://github.com/sebastianbergmann/phploc) : is used to analyse the size and structure of the code.
-   [PHP\_Depend](http://pdepend.org/) : does static code analysis and provide metrics to evaluate the quality of code.
-   [PHPMD](http://phpmd.org/) : detects problem in the code. For analysis it use s the PHP\_Depend analysis data.
-   [PHPCPD](https://github.com/sebastianbergmann/phpcpd) : detects the duplicacy or copy/paste instances throughout the code base.
-   [Clover PHP Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Clover+PHP+Plugin) : used to provide code coverage reports from **PHPUnit**.
-   [xUnit Plugin](https://wiki.jenkins-ci.org/display/JENKINS/xUnit+Plugin) : This tools is used to publish the build execution results into Jenkins tool.

## C. Set up PHP job into Jenkins:-

Before doing that, we have to check whether [Ant](http://ant.apache.org/) installed on our server or not. If not, please install it.

##### Commands used to install Ant in Linux:

1.  sudo apt-get install ant

##### Now, what is ANT?

It helps in automating the software build process. It reads and processes a configuration file (contains tasks to be performed), also defines our project’s dependencies. Most used ANT configuration file name is build.xml. I also created the same and added to our project, so that Ant will work during Jenkins build creation process.

To get a sample build.xml file template, please [click here](http://jenkins-php.org/download/build.xml).

##### PHPUnit Configuration:

We can also configure the PHPUnit XML details into build.xml. So during the build process, the PHPUnit xml configuration also gets processed and the PHP unit test cases will be executed.

After all these set-ups, lets start creating/configuring our first PHP job in Jenkins.

**Step 1:** Click on “New Item” from the left menu. Enter the Item Name like “testPHPProject” and select “Freestyle project” option and hit the “OK” button.

[![New Item [Jenkins]](http://blogs.quovantis.com/wp-content/uploads/2016/02/1317x642xNew-Item-Jenkins.png.pagespeed.ic.PBTXyU4h3u.png "New Item [Jenkins]")](http://blogs.quovantis.com/wp-content/uploads/2016/02/New-Item-Jenkins.png)

**Step 2:** A new project with name “testPHPProject” created and its configuration page opens. Here we need to configure our PHP project.

In below screen, basic details are shown which has to be filled to configure the project. The details are as follows:-

1: In “Source Code Management” section, we have to provide how would Jenkins fetch the code base. For example, from GIT or the File System. Here, I have shown an example using a File System.

2: In “Build Triggers” section, we specify which method should be used for automatic build creation. If we have used GIT as “Source Code Management” tool, then we can choose an option to create build when anything changes in the repository. But in our example, we selected “Build periodically” and added vaue as “0 14 \* \* \*”, which means everyday at 2 PM the build creation process will be started.

3: In “Build” section, a number of “build step” options are provided. Here since we are using Ant for build process, please choose “Invoke Ant” option and provide the absolute path of the Ant configuration file path in “Build File” textbox.

4: Jenkins also provides an option to add multiple “Post-build Actions”. Using this we can add multiple actions like Publish HTML, JAVADoc, Clover PHP coverage report and many others.

Here, I have chosen “Publish Clover PHP Coverage Report”. To publish it we have to provide the Clover XML Location.

[![testPHPProject Config [Jenkins]](http://blogs.quovantis.com/wp-content/uploads/2016/02/1353x1888xtestPHPProject-Config-Jenkins.png.pagespeed.ic.Pbw0TrkmJk.png "Publish Clover PHP Coverage Report")](http://blogs.quovantis.com/wp-content/uploads/2016/02/testPHPProject-Config-Jenkins.png)

**Step 3:** Click Save. The project will be configured.

**Step 4:** Click use **Build Now** option to create our first project build.

Feel free to comment if you liked this quick guide.