In this tutorial, we show you how to build a fully-functional continuous delivery pipeline for a simple PHP web application and deploy it to NGINX. We use Jenkins to build the code and run tests, and we use Octopus Deploy to deploy and promote releases.

To get up and running quickly, the [TestDrive VMs](https://octopus.com/testdrive) provide preconfigured environments demonstrating various continuous delivery pipelines documented in these guides.

Video walkthrough

![Video Thumbnail](https://embed-ssl.wistia.com/deliveries/b9a2fdb92cfc30eabc4dd62f75260a136110d753.webp?image_crop_resized=1280x720)

[![current](https://i.octopus.com/guides/badges/php-app-git-jenkins-builtin-nginx.svg)](https://octopus.com/blog/devops-documentation)

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#introduction)Introduction

The application we'll deploy is called **Random Quotes**, which is a simple web application that randomly displays a famous quote each time the page loads. It consists of a web front end and a database that contains the quotes. We'll build a complete Continuous Integration/Continuous Delivery (CI/CD) pipeline with automated builds, deployments to a dev environment, and sign offs for production deployments.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#deployment-pipeline)Deployment pipeline

For this tutorial, we assume you use **Git** for version controlling changes to your source code and **Jenkins** to compile code and run unit tests. **Octopus Deploy** will take care of the deployment. Here is what the full continuous integration and delivery pipeline will look like when we are finished:

Git php php php Jenkins Octopus (built-in) create release DEV TEST PRODUCTION Release 1.1

The development team's workflow is:

1.  Developers commit code changes to Git.
2.  Jenkins detects the change and performs the continuous integration build, this includes resolving any dependencies and running unit tests.
3.  When the Jenkins build completes, the change will be deployed to the **Dev** environment.
4.  When one of your team members (perhaps a tester) wants to see what's in a particular release, they can use Octopus to manually deploy a release to the **Test** environment.
5.  When the team is satisfied with the quality of the release and they are ready for it to go to production, they use Octopus to promote the release from the **Test** environment to the **Production** environment.

Since Octopus is designed to be used by teams, in this tutorial we also set up some simple rules:

-   Anyone can deploy to the dev or test environments.
-   Only specific people can deploy to production.
-   Production deployments require sign off from someone in our project stakeholders group.
-   We'll send an email to the team after any test or production deployment has succeeded or failed.

This tutorial makes use of the following tools:

Octopus is an extremely powerful deployment automation tool, and there are numerous ways to model a development team's workflow in Octopus Deploy, but this tends to be the most common for small teams. If you're not sure how to configure Octopus, we recommend following this guide to learn the basics. You'll then know how to adjust Octopus to suit your team's workflow.

This tutorial takes about an hour to complete. That sounds like a long time, but keep in mind, at the end of the tutorial, you'll have a fully-functional CI/CD environment for your entire team, and you'll be ready to deploy to production at the click of a button. It's worth the effort!

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#build-vs-deployment)Build vs. deployment

For any non-trivial application, you're going to deploy the software to multiple environments. For this tutorial, we're using the environments **Dev**, **Test**, and **Prod**. This means you need to choose between building your application once or building it before each deployment? To reduce the risk of a failed production deployment, Octopus strongly encourages the practice of building once, and deploying multiple times.

The following activities are a **build time** concern, so they will happen in Jenkins after any change to code is committed to Git:

1.  Check out the latest changes from Git.
2.  Resolve and install any dependencies from Composer.
3.  Run unit tests.
4.  Package the application by bundling all the files it needs to run into a ZIP file.

This results in a green CI build and a file that contains the application and everything it needs to run. Any configuration files will have their default values, but they won't know anything about dev vs. production settings just yet.

Lastly, it's very important that we **give this artifact a unique version number**. We will produce a new artifact file every time our CI build runs, and we don't want to accidentally deploy a previous version of the artifact.

An example of a package that is ready to be deployed is:

```
RandomQuotes.1.0.0.zip
```

At this point, we have a single _artifact_ that contains all the files our application needs to run, ready to be deployed. We can deploy it over and over, using the same artifact in each environment. If we deploy a bad release, we can go and find the older version of the artifact and re-deploy it.

The following activities happen at **deployment time** by Octopus Deploy:

1.  Changing any configuration files to include settings appropriate for the environment, e.g., database connection strings, API keys, etc.
2.  Running tasks that need to be performed during the deployment such as database migrations or taking the application temporarily offline.

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#prerequisites)Prerequisites

There are a number of tools you need to install to implement a complete CI/CD workflow. These include the Jenkins and Octopus servers, some command-line tools, and NGINX to host the final deployment.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#git)Git

The source code for the sample application is hosted on GitHub. To access the code, you need the Git client. The [Git documentation](https://g.octopushq.com/GitGettingStarted) has instructions to download and install the Git client.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#jenkins)Jenkins

Jenkins can be freely downloaded from the project's [download page](https://g.octopushq.com/JenkinsDownload). Native builds are available for Windows, macOS, and various Linux distributions.

Docker images are also available from [Docker Hub](https://g.octopushq.com/JenkinsDockerHub).

This guide makes use of the [Credentials](https://plugins.jenkins.io/credentials), [Credentials Binding](https://plugins.jenkins.io/credentials-binding), and [Plain Credentials](https://plugins.jenkins.io/plain-credentials) plugins.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#octopus-cli)Octopus CLI

Jenkins calls the Octopus CLI tool to create and deploy releases in the Octopus Server. The Octopus CLI can be downloaded from the [Octopus downloads](https://octopus.com/downloads) page or installed from [Chocolatey](https://g.octopushq.com/ChocolateyOctopusCLI).

The Octopus CLI tool must be installed on the Jenkins server or any agents that will execute the Jenkins project.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#php)PHP

PHP is available from its [website](https://www.php.net/manual/en/install.php), or can be installed using your Linux package manager. Ubuntu users can install PHP with the command:

```
sudo apt-get install php-cli
```

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#composer)Composer

Composer is a dependency manager for PHP. It can be downloaded from the [project homepage](https://getcomposer.org/download/).

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#phpunit)PHPUnit

[PHPUnit](https://phpunit.de/) is a unit testing framework for PHP. It is downloaded by the sample project as a dependency, making the executable available from **./vendor/bin/phpunit**. PHPUnit can also be installed using your Linux package manager. Ubuntu users can install PHP FPM with the command:

```
sudo apt-get install phpunit
```

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#php-fpm)PHP FPM

[PHP FPM](https://www.php.net/manual/en/install.fpm.php) is used to process requests to PHP pages. PHP FPM is most easily installed using your Linux package manager. Ubuntu users can install PHP FPM with the command:

```
sudo apt-get install php-fpm
```

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#octopus-deploy)Octopus Deploy

The installation file used to install Octopus Server on-premises can be downloaded [here](https://octopus.com/downloads/server).

Octopus requires a Microsoft SQL Server to host the database. Microsoft SQL Server Express can be downloaded for free [here](https://g.octopushq.com/SqlServerDownload).

Complete the following steps to install Octopus:

1.  Start the Octopus Installer, click **Next**, accept the Terms in the License Agreement and click **Next**.
2.  Accept the default Destination Folder or choose a different location and click **Next**.
3.  Click **Install**, and give the app permission to make changes to your device.
4.  Click **Finish** to exit the installation wizard and launch the Getting started wizard to configure your Octopus Server.
5.  Click **Get started...** and either enter your details to start a free trial of Octopus Deploy or enter your license key and click **Next**.
6.  Accept the default Home Directory or enter a location of your choice and click **Next**.
7.  Decide whether to use a **Local System Account** or a **Custom Domain Account**.

Learn more about the [permissions required for the Octopus Windows Service](https://octopus.com/docs/installation/permissions-for-the-octopus-windows-service) or using a [Managed Service Account](https://octopus.com/docs/installation/managed-service-account).

8.  On the Database page, click the dropdown arrow in the Server Name field to detect the SQL Server database. Octopus will create the database for you which is the recommended process; however, you can also create your own database.
9.  Enter a name for the database, and click **Next** and **OK** to create the database.

Be careful not to use the name of an existing database as the setup process will install Octopus into that pre-existing database.

10.  Accept the default port and directory or enter your own. This guide assumes Octopus is listening on port 80. Click **Next**.
11.  If you’re using a username and passwords stored in Octopus authentication mode, enter the username and password that will be used for the Octopus administrator. If you are using active directory, enter the active directory user details.
12.  Click **Install**.
13.  When the installation has completed, click **Finish** to launch the Octopus Manager.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#nginx)NGINX

NGINX is used as a reverse proxy to expose web applications running in Linux. NGINX can be downloaded from your Linux package manager.

NGINX may be configured to listen on port 80 by default. If this conflicts with other services (this guide assumes Octopus is running on port 80), you will need to change it. Open the file **/etc/nginx/sites-enabled/default** and locate the following line:

```
listen 80 default_server;
```

Change the port number:

```
listen 4444 default_server;
```

Restart the **nginx** service.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#clone-source-code)Clone source code

The source code for the random quotes application is hosted in [GitHub](https://github.com/OctopusSamples/RandomQuotes-PHP). The code can be cloned from [https://github.com/OctopusSamples/RandomQuotes-PHP.git](https://github.com/OctopusSamples/RandomQuotes-PHP.git) with the command:

```
git clone https://github.com/OctopusSamples/RandomQuotes-PHP.git
```

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#octopus-api-key)Octopus API key

In order to allow Jenkins to communicate with Octopus, we need to generate an API key. This is done in the Octopus web portal. The web portal can be opened from the **Browse** link in the Octopus Manager:

[![](https://i.octopus.com/guides/octopus/manager/octopus-manager.png)](https://i.octopus.com/guides/octopus/manager/octopus-manager.png)

From the Octopus Deploy web portal, sign in, and view your profile:

[![](https://i.octopus.com/guides/octopus/apikey/profile.png)](https://i.octopus.com/guides/octopus/apikey/profile.png)

Go to the **API keys tab**. This lists any previous API keys that you have created. Click on **New API key**:

[![](https://i.octopus.com/guides/octopus/apikey/keys.png)](https://i.octopus.com/guides/octopus/apikey/keys.png)

Give the API key a name so that you remember what the key is for, and click **Generate New**:

[![](https://i.octopus.com/guides/octopus/apikey/new-key.png)](https://i.octopus.com/guides/octopus/apikey/new-key.png)

Copy the new API key to your clipboard:

[![](https://i.octopus.com/guides/octopus/apikey/api-key.png)](https://i.octopus.com/guides/octopus/apikey/api-key.png)

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#continuous-integration)Continuous integration

Jenkins is a free and open source continuous integration server. In this tutorial, we rely on Jenkins to do the following:

-   Clone the code from Git.
-   Resolve and install any dependencies from Composer.
-   Run unit tests.
-   Package the application by bundling all the files it needs to run into a ZIP file.
-   Push the package to the Octopus (built-in) package repository.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#install-the-octopus-plugin)Install the Octopus plugin

Jenkins pushes artifacts to Octopus and creates and deploys releases via the [Octopus plugin](https://octopus.com/docs/packaging-applications/build-servers/jenkins). To install the plugin click **Manage Jenkins**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)

Click **Manage Plugins**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-manage-plugins.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-manage-plugins.png)

Click the **Available** tab, enter **Octopus** in the search box, select the **Octopus Deploy** plugin, and click **Install without restart**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-octopus-plugin.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-octopus-plugin.png)

Wait while the plugin is installed, and click **Go back to the top page** when the installation is complete:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-octopus-plugin-install.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-octopus-plugin-install.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#configure-the-octopus-plugin)Configure the Octopus plugin

To use the Octopus steps we first need to configure the Octopus server details. Click **Manage Jenkins**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)

Click **Configure System**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-configure-system.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-configure-system.png)

Define the **Server ID** as **Local**, enter the **URL** and **API Key**, and click **Save**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-configure-octopus.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-configure-octopus.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#configure-the-octopus-cli)Configure the Octopus CLI

The Octopus plugin makes use of the Octopus CLI to execute steps. The path to the CLI must be configured. Click **Manage Jenkins**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/005-manage-jenkins.png)

Click **Global Tool Configuration**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-configure-tools.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/010-configure-tools.png)

Configure the **Path** to the Octopus CLI, and click **Save**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-configured-tools.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/020-configured-tools.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#create-the-jenkins-project)Create the Jenkins project

We are now ready to create a project to build and test the sample application.

Click **New Item**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/170-new-item.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/170-new-item.png)

Enter **Random Quotes** as the **Project name**, select the **Freestyle project** option, and click **OK**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/180-freestyle-project.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/180-freestyle-project.png)

From the **Source Code Management** section, select the **Git** option, and enter **https://github.com/OctopusSamples/RandomQuotes-PHP.git**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/190-git-settings.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/190-git-settings.png)

In order to watch for changes to the source code, we need to poll the Git repository. From the **Build Triggers** section, select the **Poll SCM** option, and enter **H/5 \* \* \* \*** for the schedule. This instructs Jenkins to poll the Git repository every 5 minutes looking for changes to the code:

 [![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/200-git-polling.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/200-git-polling.png)[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/230-credentials-octopusapikey.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/230-credentials-octopusapikey.png)

Click **Add build step** and select the **Execute shell** option:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/240-shell-command-1.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/240-shell-command-1.png)

Enter the following script for the **Command**:

```
composer install
./vendor/bin/phpunit --bootstrap ./vendor/autoload.php ./tests/quotetest.php
```

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/250-npm-test.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/250-npm-test.png)

From the **Build** section, select the **Octopus Deploy: Package application** option:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/260-pack-command.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/260-pack-command.png)

Enter **RandomQuotes** for the **Package ID**, **1.0.${BUILD\_NUMBER}** for the **Version number>**, **${WORKSPACE}/\*\*** for the **Package include paths**, and **${WORKSPACE}** for the **Package output folder**:

Note that the slashes in the **include** option are operating system specific. Linux uses a forward slash, and Windows uses a backslash.

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/270-octo-plugin-pack.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/270-octo-plugin-pack.png)

From the **Build** section, select the **Octopus Deploy: Push packages** option:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/280-push-command.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/280-push-command.png)

Enter **${WORKSPACE}/RandomQuotes.1.0.${BUILD\_NUMBER}.zip** in the **Package paths** field, and click **Save**:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/290-octo-plugin-push.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/290-octo-plugin-push.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#build-the-project)Build the project

Our project is now ready to build the sample application.

Click the **Build Now** link. The first build will commence. Click the **#1** link to view the build:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/310-build-one.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/310-build-one.png)

Click the **Console Output** link to view the build output:

[![](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/320-console.png)](https://i.octopus.com/guides/jenkins/initialproject/php-app/nginx/builtin/git/320-console.png)

The build logs are streamed to the browser. In a minute or so, the build will complete, and the resulting package will have been uploaded to Octopus. Congratulations! You have completed the first half of the CI/CD pipeline.

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#deploying-with-octopus-deploy)Deploying with Octopus Deploy

Now that Jenkins has successfully built the application, we need to configure Octopus to deploy it into our environments.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#create-the-environments)Create the environments

Environments represent the stages that a deployment must move through as part of the deployment pipeline. We'll create three environments: **Dev**, **Test**, and **Prod**.

Log into Octopus, and click the **Infrastructure** link, then the **Environments** link, and click **ADD ENVIRONMENT**:

[![](https://i.octopus.com/guides/octopus/environments/015-add-environment-1.png)](https://i.octopus.com/guides/octopus/environments/015-add-environment-1.png)

Enter **Dev** as the New environment name, and click **SAVE**:

[![](https://i.octopus.com/guides/octopus/environments/020-environment-dev.png)](https://i.octopus.com/guides/octopus/environments/020-environment-dev.png)

Repeat the process for the **Test** and **Prod** environments:

 [![](https://i.octopus.com/guides/octopus/environments/035-environment-test.png)](https://i.octopus.com/guides/octopus/environments/035-environment-test.png)[![](https://i.octopus.com/guides/octopus/environments/050-environment-prod.png)](https://i.octopus.com/guides/octopus/environments/050-environment-prod.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#create-the-octopus-deployment-project)Create the Octopus deployment project

With the environments defined and a target created, we now need to create a deployment project in Octopus.

Log into Octopus, click the **Projects** link, and click **ADD PROJECT**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/010-octopus-add-project.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/010-octopus-add-project.png)

Enter **Random Quotes** for the project name, and click **SAVE**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/015-octopus-new-project-name.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/015-octopus-new-project-name.png)

We start by defining the variables that we will consume as part of the deployment. Click the **Variables** link, and then select the **Project** option:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/020-octopus-variables.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/020-octopus-variables.png)

We define a variable called **Nginx Port** with unique values bound to each environment. This means that when we deploy the project to each environment, a new port will be used to expose the application via NGINX.

By exposing each environment as a unique port, we can implement multiple environments with a single physical target.

-   Define a value of **8081** scoped to the **Dev** environment.
-   Define a value of **8082** scoped to the **Test** environment.
-   Define a value of **8083** scoped to the **Prod** environment.

Save the changes by clicking **SAVE**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/030-octopus-variables-populated.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/030-octopus-variables-populated.png)

We will now define the deployment process. Click the **Deployments** link, click **Overview**, and then click **DEFINE YOUR DEPLOYMENT PROCESS**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/040-octopus-define-process.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/040-octopus-define-process.png)

Click **ADD STEP**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/050-octopus-add-step.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/050-octopus-add-step.png)

Enter **Nginx** into the search box:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/060-octopus-search.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/060-octopus-search.png)

Click **ADD** on the **Deploy to NGINX** tile:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/070-octopus-add-containers.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/070-octopus-add-containers.png)

Enter **Deploy web app to Nginx** as the **Step Name**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/080-octopus-step-name.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/080-octopus-step-name.png)

Enter **web** as the **Role**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/090-octopus-step-role.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/090-octopus-step-role.png)

Select the **RandomQuotes** package:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/100-octopus-step-package.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/100-octopus-step-package.png)

Enter the following scripts in the **Post-deployment script** text area. This reloads NGINX with the new configuration:

```
nginx -s reload
```

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/110-octopus-variables-appsettings.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/110-octopus-variables-appsettings.png)

Remove the default binding by clicking the cross icon:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/120-octopus-step-remove-binding.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/120-octopus-step-remove-binding.png)

Click **ADD BINDING**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/130-octopus-step-add-binding.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/130-octopus-step-add-binding.png)

Enter **#{NGINX Port}** for the port, and click **OK**. This configures NGINX to listen on the specified port, which in this case is one of the variables we defined earlier.

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/140-octopus-step-binding-port.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/140-octopus-step-binding-port.png)

Click **ADD LOCATION**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/150-octopus-step-add-location.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/150-octopus-step-add-location.png)

The PHP application is made up of static HTML and JavaScript resources as well as PHP pages. To serve this application, NGINX must first attempt to match the HTTP request to a static file, and if that fails then pass the request to the PHP application. To implement this solution we need three locations.

The first location is used to redirect a request to the root path to the **index.html** file. Enter **\= /** for the **Location**. This location defines an exact match to the root path. Click **ADD DIRECTIVE** and define a directive called **index** with a value of **index.html**. The **index** directive redirects requests to the a default page.

Click **OK** to save the location:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/160-octopus-step-location-port.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/160-octopus-step-location-port.png)

Click **ADD LOCATION**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/170-octopus-step-add-location.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/170-octopus-step-add-location.png)

The second location attempts to serve static resources if they match the request. Enter **/** for the **Location**. This location defines a match for all requests that do not match another more specific location.

Add two directives to the location.

The first directive is called **root**, and has a value of **#{Octopus.Action.Package.InstallationDirectoryPath}/public**. This directive defines the directory that NGINX will attempt to find static resources in. The **Octopus.Action.Package.InstallationDirectoryPath** variable resolves to a path like **/home/Octopus/Applications/Dev/RandomQuotes/1.0.1**, which contains the files from the extracted package.

The second directive is called **try\_files**, and has a value of **$uri /index.php$is\_args$args**. This directive instructs NGINX to first attempt to serve a file that matches the request. If no file can be found, the request is redirected to the **index.php** file.

Click **OK** to save the location:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/180-octopus-step-location-port.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/180-octopus-step-location-port.png)

Click **ADD LOCATION**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/190-octopus-step-add-location.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/190-octopus-step-add-location.png)

The third location processes the request with PHP-FPM. Enter **~ \[^/\]\\.php(/|$)** for the **Location**. This location defines a regular expression to match any PHP page.

Add six directives to the location.

The first directive is called **fastcgi\_split\_path\_info**, and has a value of **^(.+?\\.php)(/.\*)$**. This directive defines a regular expression with two capture groups. The first capture group is assigned to the **$fastcgi\_script\_name** variable, and the second capture group is assigned to the **$fastcgi\_path\_info** variable. These variables are referenced later on.

The second directive is called **fastcgi\_pass**, and has a value of **unix:/run/php/php7.2-fpm.sock**. This directive defines how NGINX communicates with the PHP-FPM service. The actual path depends on how PHP-FPM has been configured. On a Ubuntu 18.04 system, this path is configured in **/etc/php/7.2/fpm/pool.d/www.conf**.

The third directive is called **fastcgi\_index**, and has a value of **index.php**. This directive defines the file to process the request. In our sample application all requests not satisfied by static resources are handled by the **index.php** file.

The fourth directive is called **include**, and has a value of **fastcgi.conf**. This directive include the contents of the file **fastcgi.conf** (more specificlly the **/etc/nginx/fastcgi.conf** file on a Ubuntu 18.04 system). The **fastcgi.conf** file maps a number of NGINX variables to parameters sent to the PHP-FPM service, including the **$fastcgi\_script\_name** variable we defined in the first directive.

The fifth directive is called **root**, and has a value of **#{Octopus.Action.Package.InstallationDirectoryPath}/public**. Like the **root** directive we defined in the **/** (root) location, this defines the directory where the PHP files are located.

The sixth directive is called **fastcgi\_param**, and has a value of **ENVIRONMENT\_NAME #{Octopus.Environment.Name}**. This directive set the environment variable called **ENVIRONMENT\_NAME** to the name of the Octopus environment that the application was deloyed to.

Click **OK** to save the location:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/200-octopus-step-location-port.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/200-octopus-step-location-port.png)

Click **SAVE**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/210-save.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/210-save.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#deploy)Deploy!

We now have a deployment project in Octopus ready to deploy our PHP application to our **Dev**, **Test**, and **Prod** environments. The next step is to create and deploy a release.

Click **CREATE RELEASE**.

The release creation screen provides an opportunity to review the packages that will be included and to add any release notes. By default, the latest package is selected automatically. Click **SAVE**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/170-octopus-create-release.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/170-octopus-create-release.png)

This screen allows you to select the environment that will be deployed into. Lifecycles can be used to customize the progression of deployments through environments (this is [demonstrated later in the guide](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#lifecycles)), however, we will accept the default option to deploy to the **Dev** environment by clicking **DEPLOY TO DEV...**:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/190-octopus-deploy-to-dev.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/190-octopus-deploy-to-dev.png)

Click **DEPLOY** to deploy the application into the **Dev** environment:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/200-octopus-deploy.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/200-octopus-deploy.png)

The application is then deployed:

[![](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/210-octopus-deployment.png)](https://i.octopus.com/guides/octopus/project/php-app/nginx/builtin/git/210-octopus-deployment.png)

Congratulations! You have now built and deployed your first application. Visit http://localhost:8081 in a browser to view a random quote. Note the port matches the variable **Nodejs Port** that we assigned to the environment. Ensuring each environment deploys to a unique port is how we can use a single virtual machine to host multiple environments:

[![](https://i.octopus.com/guides/app/php-app/nginx/builtin/git/nginx-random-quotes-dev-app.png)](https://i.octopus.com/guides/app/php-app/nginx/builtin/git/nginx-random-quotes-dev-app.png)

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#continuous-deployments)Continuous deployments

The process of deploying a successful build to the **Dev** environment is currently a manual one; Jenkins pushes the file to Octopus, and we must trigger the initial deployment to the **Dev** environment from within Octopus. Typically though, deployments to the **Dev** environment will be performed automatically if a build and all of its tests succeed.

To trigger the initial deployment to the **Dev** environment after a successful build, we will go back to the project we created in Jenkins and add an additional step to create an Octopus release and then deploy it to the **Dev** environment.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#add-a-create-and-deploy-release-step)Add a create and deploy release step

As before, we make use of the Octopus plugin to interact with the Octopus server. In this case, we use the plugin to create and deploy a release in Octopus after the package has been pushed.

Log back into Jenkins and click the **Random Quotes** project link:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/005-random-quotes-link.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/005-random-quotes-link.png)

Click the **Configure** link:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/010-create-release-configure.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/010-create-release-configure.png)

Click **Add post-build action**, and click the **Octopus Deploy: Create Release** option.

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/015-octo-plugin-create-release-build-step.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/015-octo-plugin-create-release-build-step.png)

Set the **Project Name** to **Random Quotes**:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/017-octo-plugin-create-release-command.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/017-octo-plugin-create-release-command.png)

Check the **Deploy the release after it is created** check box, set the **Environment** to **Dev**, and check the **Show deployment progress** checkbox. Then click **Save**:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/020-octo-plugin-create-release-command.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/020-octo-plugin-create-release-command.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#build-and-deploy-the-project)Build and deploy the project

Click **Build Now**. The second build will commence. Click the **#2** link to view the second build:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/035-build-two.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/035-build-two.png)

Click the **Console Output** link to view the build output:

[![](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/040-build-and-deploy-console.png)](https://i.octopus.com/guides/jenkins/createrelease/php-app/nginx/builtin/git/040-build-and-deploy-console.png)

The build logs are streamed to the browser. In a minute or so, the build will complete, and Octopus will perform a deployment using the new package to the **Dev** environment. You can confirm this by looking for the log message that says **Deploy Random Quotes release 0.0.2 to Dev: Success**.

We now have a complete CI/CD pipeline! Jenkins will watch for any changes committed to the Git repository, compile the code, run any tests, and then call Octopus to deploy the result.

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#additional-configuration)Additional configuration

Now we will explore some of the more advanced features of Octopus that allow us to customize the deployment progression through environments, secure deployments to production environments, add deployment sign offs, view the audit logs, and add notifications.

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#lifecycles)Lifecycles

Our project currently uses the default lifecycle, which defines a progression through all the environments in the order they were created.

A custom lifecycle allows the progression of a deployment to be further refined, by defining only a subset of environments that can be deployed to, allowing some environments to be skipped entirely, or requiring that a minimum number of environments are successfully deployed to before moving onto the next environment.

Here we will create a custom lifecycle that makes deployments to the **Dev** environment optional. This means that initial deployments can be made to the **Dev** _or_ **Test** environments, but a successful deployment _must_ be made to the **Test** environment before it can be progressed to the **Prod** environment.

Skipping the **Dev** environment like this may be useful for promoting a release candidate build directly to the **Test** environment for product owners or testers to review.

Click the **Library** link, click the **Lifecycles** link, and click **ADD LIFECYCLE**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/015-lifecycle-add-lifecycle.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/015-lifecycle-add-lifecycle.png)

Set the lifecycle name to **Dev, Test, and Prod**, and the description to **Progression from the Dev to the Prod environments**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/020-lifecycle-name.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/020-lifecycle-name.png)

Phases are used to group environments that can accept a deployment. Simple lifecycles, such as the lifecycle we are creating, have a one-to-one relationship between phases and environments.

Click **ADD PHASE**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/025-lifecycle-add-phase-1.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/025-lifecycle-add-phase-1.png)

Enter **Dev** as the phase name, and select the **Optional phase** option. This means that deployments can skip this phase and any environments defined in it, and deploy directly to the next phase.

Because we are mapping each environment to its own phase, the name of the phase matches the name of the environment:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/030-lifecycle-phase-name-1.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/030-lifecycle-phase-name-1.png)

Click **ADD ENVIRONMENT**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/035-lifecycle-add-environment-1.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/035-lifecycle-add-environment-1.png)

Click the dropdown arrow, select the **Dev** environment, and click **OK**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/040-lifecycle-select-environment-1.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/040-lifecycle-select-environment-1.png)

Repeat the process to add a new phase for the **Test** and **Prod** environments, leaving the default **All must complete** option selected:

 [![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/060-lifecycle-select-environment-2.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/060-lifecycle-select-environment-2.png)[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/080-lifecycle-select-environment-3.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/080-lifecycle-select-environment-3.png)

Click **SAVE**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/090-lifecycle-save.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/090-lifecycle-save.png)

Now, we need to switch the deployment project from the **Default Lifecycle** to the newly created lifecycle.

Click the **Projects** link, and click the **Random Quotes** project tile:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/095-random-quotes-project.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/095-random-quotes-project.png)

Click the **Process** link, and click **CHANGE**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/100-random-quotes-change-lifecycle.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/100-random-quotes-change-lifecycle.png)

Select the **Dev, Test, and Prod** lifecycle. Notice the **Dev** environment is shown as optional.

Click **SAVE**:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/105-random-quotes-select-lifecycle.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/105-random-quotes-select-lifecycle.png)

Click **CREATE RELEASE**, and click **SAVE** to save the new release:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/110-random-quotes-create-release.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/110-random-quotes-create-release.png)

Because the **Dev** environment has been configured as optional, the initial release can be made to either the **Dev** or **Test** environments. We'll skip the **Dev** environment and deploy straight to **Test**.

Click **DEPLOY TO...**, and select the **Test** environment:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/115-random-quotes-select-test.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/115-random-quotes-select-test.png)

Click **DEPLOY** to deploy the application to the **Test** environment:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/120-random-quotes-deploy-test.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/120-random-quotes-deploy-test.png)

The deployment is then performed directly in the **Test** environment, skipping the **Dev** environment:

[![](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/125-random-quotes-deployed-test.png)](https://i.octopus.com/guides/octopus/lifecycle/php-app/nginx/builtin/git/125-random-quotes-deployed-test.png)

Opening http://localhost:8082 displays the copy of the Random Quotes application deployed to the **Test** environment. Note the port **8082** matches the variable **Nginx Port** that we assigned to the environment.

Also see the footer text that says **running in Test**. This is the result of the environment variable **ENVIRONMENT** being defined via the **fastcgi\_param** directive.

[![](https://i.octopus.com/guides/app/php-app/nginx/builtin/git/nginx-random-quotes-test-app.png)](https://i.octopus.com/guides/app/php-app/nginx/builtin/git/nginx-random-quotes-test-app.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#approvals)Approvals

It's a common business requirement to have testers or product owners manually verify that a particular build meets the requirements before a deployment can be considered successful.

Octopus supports this workflow through the use of manual intervention steps. We'll add a manual intervention step to the end of the deployment process, which requires a responsible party to verify the build meets the requirements.

Open the Random Quotes project, click the **Process** link, and click the **ADD STEP** button:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/010-add-step.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/010-add-step.png)

Search for the **Manual Intervention Required** step, and add it to the process:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/020-octopus-add-intervention.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/020-octopus-add-intervention.png)

Enter **Deployment Sign Off** for the **Step Name**:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/025-octopus-step-name.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/025-octopus-step-name.png)

Enter the following for the **Instructions**:

```
Open the application and confirm it meets all the requirements.
```

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/030-octopus-step-instructions.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/030-octopus-step-instructions.png)

Because every build is automatically deployed to the **Dev** environment, it doesn't make sense to force someone to manually approve all those deployments. To accommodate this, we do not enable the manual intervention step for deployments to the **Dev** environment.

Expand the **Environments** section under the **Conditions** heading, select the **Skip specific environments** option, and select the **Dev** environment.

Click **SAVE** to save the step:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/045-octopus-step-skip-dev.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/045-octopus-step-skip-dev.png)

When this application is deployed to the **Test** or **Prod** environments, a prompt will be displayed requiring manual sign off. Click **ASSIGN TO ME** to assign the task to yourself:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/050-octopus-deploy-assign-intervention.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/050-octopus-deploy-assign-intervention.png)

Add a note in the provided text box, and click **PROCEED** to complete the deployment:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/055-octopus-deploy-intervention-notes.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/055-octopus-deploy-intervention-notes.png)

The deployment will then complete successfully:

[![](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/060-octopus-deploy-success.png)](https://i.octopus.com/guides/octopus/intervention/php-app/nginx/builtin/git/060-octopus-deploy-success.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#email-notifications)Email notifications

Octopus has native support for sending email notifications as part of the deployment process. We will add a step to the deployment process to let people know when a release has been deployed to an environment.

To start, we need to configure an SMTP server to send our emails. For this guide, we'll use the free SMTP server provided by Google.

Click the **Configuration** link:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/005-octopus-configuration.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/005-octopus-configuration.png)

Click the **SMTP** link:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/010-octopus-smtp.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/010-octopus-smtp.png)

-   Enter **smtp.gmail.com** as the **SMTP Host**.
-   Enter **587** as the **SMTP Port**.
-   Enable the **Use SSL/TLS** option.
-   Enter your Gmail address as the **From Address**.
-   Enter your Gmail address and password in the **credentials**.

You will enable the [Less secure apps](https://g.octopushq.com/GoogleLessSecureApps) option on your Google account for Octopus to send emails via the Google SMTP server.

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/015-octopus-smtp-populated.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/015-octopus-smtp-populated.png)

Open the Random Quotes project, click the **Process** link, and click **ADD STEP**:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/025-add-step.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/025-add-step.png)

Search for the **Send an Email** step, and add it to the process:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/035-octopus-add-email.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/035-octopus-add-email.png)

Enter **Random quotes deployment status** for the **Step Name**:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/040-octopus-step-name.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/040-octopus-step-name.png)

Enter the email address to receive the notification in the **To field**:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/045-octopus-step-to.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/045-octopus-step-to.png)

Enter **Random quotes deployment status** as the **Subject**:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/050-octopus-step-subject.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/050-octopus-step-subject.png)

Enter the following as the **Body**:

```
Deployment to #{Octopus.Environment.Name}
#{each step in Octopus.Step}
StepName: #{step}
Status: #{step.Status.Code}
#{/each}
```

Here we use the **#{Octopus.Environment.Name}** variable provided by Octopus to add the name of the environment that was deployed to, and then loop over the status codes in the **#{Octopus.Step}** variable to return the status of each individual step.

The complete list of system variables can be found in the [Octopus documentation](https://octopus.com/docs/deployment-process/variables/system-variables).

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/055-octopus-step-body.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/055-octopus-step-body.png)

We want to be notified of the status of this deployment regardless of whether the deployment succeeded or failed. Click the **Run Conditions** section to expand it.

Select the **Always run** option, which ensures the notification email is sent even when the deployment or the manual intervention fail:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/058-octopus-step-always-run.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/058-octopus-step-always-run.png)

Given every change to the source code will result in a deployment to the **Dev** environment, we do not want to generate reports for deployments to this environment.

Click the **Environments** section to expand it. Select the **Skip specific environments** option, and select the **Dev** environment to skip it.

This is the last piece of configuration for the step, so click **SAVE** to save the changes:

[![](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/059-octopus-step-skip-dev.png)](https://i.octopus.com/guides/octopus/email/php-app/nginx/builtin/git/059-octopus-step-skip-dev.png)

Deploy the project to the Test or Prod environments. When the deployment succeeds, the notification email will be sent:

[![](https://i.octopus.com/guides/octopus/email/notification-email.png)](https://i.octopus.com/guides/octopus/email/notification-email.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#permissions)Permissions

One of the strengths of Octopus is that it models environments as first-class entities. This means the security layer can apply rules granting access only to specific environments. We'll take advantage of this ability to model and secure environments to create two users, an internal deployer who can deploy to the **Dev** and **Test** environments, and a production deployer who can deploy to the **Prod** environment.

We start by creating the users. Click the **Configuration** link:

[![](https://i.octopus.com/guides/octopus/permissions/005-octopus-configuration.png)](https://i.octopus.com/guides/octopus/permissions/005-octopus-configuration.png)

Click the **Users** link:

[![](https://i.octopus.com/guides/octopus/permissions/010-octopus-users.png)](https://i.octopus.com/guides/octopus/permissions/010-octopus-users.png)

Click **ADD USER**:

[![](https://i.octopus.com/guides/octopus/permissions/015-octopus-add-user.png)](https://i.octopus.com/guides/octopus/permissions/015-octopus-add-user.png)

Enter **internaldeployer** as the **Username**:

[![](https://i.octopus.com/guides/octopus/permissions/020-octopus-username.png)](https://i.octopus.com/guides/octopus/permissions/020-octopus-username.png)

Enter **Internal Deployer** as the **Display Name**:

[![](https://i.octopus.com/guides/octopus/permissions/025-octopus-displayname.png)](https://i.octopus.com/guides/octopus/permissions/025-octopus-displayname.png)

Enter the user's **email address**. We have used a dummy address of **internaldeployer@example.org** here:

[![](https://i.octopus.com/guides/octopus/permissions/030-octopus-email.png)](https://i.octopus.com/guides/octopus/permissions/030-octopus-email.png)

Enter a password and confirm it. Then click **SAVE** to create the user:

[![](https://i.octopus.com/guides/octopus/permissions/035-octopus-password.png)](https://i.octopus.com/guides/octopus/permissions/035-octopus-password.png)

Repeat the process for a user called **productiondeployer**. The summary for the **productiondeployer** user is shown below:

[![](https://i.octopus.com/guides/octopus/permissions/067-octopus-user-overview.png)](https://i.octopus.com/guides/octopus/permissions/067-octopus-user-overview.png)

The newly created users are not assigned to any teams and have no permissions to do anything. To grant them permissions, we must first create two teams. The internal deployment team will grant access to deploy to the **Dev** and **Test** environments, while the production deployment team will grant access to deploy to the **Prod** environment.

Click the **Configuration** link:

[![](https://i.octopus.com/guides/octopus/permissions/005-octopus-configuration.png)](https://i.octopus.com/guides/octopus/permissions/005-octopus-configuration.png)

Click the **Teams** link:

[![](https://i.octopus.com/guides/octopus/permissions/075-octopus-teams.png)](https://i.octopus.com/guides/octopus/permissions/075-octopus-teams.png)

Click **ADD TEAM**:

[![](https://i.octopus.com/guides/octopus/permissions/080-octopus-add-team.png)](https://i.octopus.com/guides/octopus/permissions/080-octopus-add-team.png)

Enter **Internal Deployers** for the **New team name**, and **Grants access to perform a deployment to the internal environments** for the **Team description**. Click **SAVE** to create the team:

[![](https://i.octopus.com/guides/octopus/permissions/085-octopus-new-team.png)](https://i.octopus.com/guides/octopus/permissions/085-octopus-new-team.png)

We need to add the **internaldeployer** user to this team. Click **ADD MEMBER**:

[![](https://i.octopus.com/guides/octopus/permissions/090-octopus-add-member.png)](https://i.octopus.com/guides/octopus/permissions/090-octopus-add-member.png)

Select the **Internal Deployer** user from the dropdown list, and click **ADD**:

[![](https://i.octopus.com/guides/octopus/permissions/095-octopus-add-internal-user.png)](https://i.octopus.com/guides/octopus/permissions/095-octopus-add-internal-user.png)

The team does not grant any permissions yet. To add permissions click the **USER ROLES** tab, and click **INCLUDE USER ROLE**:

[![](https://i.octopus.com/guides/octopus/permissions/100-octopus-user-roles.png)](https://i.octopus.com/guides/octopus/permissions/100-octopus-user-roles.png)

Select the **Deployment creator** role from the dropdown list. As the name suggests, this role allows a user to create a deployment, which results in the deployment process being executed.

Click **DEFINE SCOPE**:

[![](https://i.octopus.com/guides/octopus/permissions/105-octopus-add-role.png)](https://i.octopus.com/guides/octopus/permissions/105-octopus-add-role.png)

We only want to allow the internal deployer to deploy to the internal environments. Select the **Dev** and **Test** environments, and click **APPLY**:

[![](https://i.octopus.com/guides/octopus/permissions/110-octopus-role-scope.png)](https://i.octopus.com/guides/octopus/permissions/110-octopus-role-scope.png)

The permissions are then applied. We need a second permission to allow the internal deployer to view the projects dashboard. Click **INCLUDE USER ROLE** again:

[![](https://i.octopus.com/guides/octopus/permissions/115-octopus-user-roles.png)](https://i.octopus.com/guides/octopus/permissions/115-octopus-user-roles.png)

Select the **Project viewer** role. This role does not need to be scoped, so click the **APPLY** button:

[![](https://i.octopus.com/guides/octopus/permissions/120-octopus-add-role.png)](https://i.octopus.com/guides/octopus/permissions/120-octopus-add-role.png)

Here are the final set of roles applied to the team:

[![](https://i.octopus.com/guides/octopus/permissions/123-octopus-team-roles.png)](https://i.octopus.com/guides/octopus/permissions/123-octopus-team-roles.png)

Repeat the process to create a team called **Production Deployers** that includes the **productiondeployer** user, and grants the **Deployment creator** role scoped to the **Prod** environment:

[![](https://i.octopus.com/guides/octopus/permissions/178-octopus-team-roles.png)](https://i.octopus.com/guides/octopus/permissions/178-octopus-team-roles.png)

When we log in as the **internaldeployer** user, we see that the Random Quotes project dashboard shows **DEPLOY...** buttons for the **Dev** and **Test** environments. Any deployments in the production environment will be visible, but they cannot be created by this user:

[![](https://i.octopus.com/guides/octopus/permissions/180-octopus-internal-deployer-overview.png)](https://i.octopus.com/guides/octopus/permissions/180-octopus-internal-deployer-overview.png)

When we log in as the **productiondeployer** user, we see that the Random Quotes project dashboard shows **DEPLOY...** buttons only for the **Prod** environment. Also note that the lifecycle rules still apply, and only successful deployments to the **Test** environment are available to be promoted to the **Prod** environment:

[![](https://i.octopus.com/guides/octopus/permissions/185-octopus-proudction-deployer-overview.png)](https://i.octopus.com/guides/octopus/permissions/185-octopus-proudction-deployer-overview.png)

### [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#audit-log)Audit Log

Important interactions in Octopus are tracked in the audit log. This is useful for teams that have security or legislative requirements to track the changes and deployments made to their infrastructure.

To view the audit log, click the **Configuration** link:

[![](https://i.octopus.com/guides/octopus/audit/005-octopus-configuration.png)](https://i.octopus.com/guides/octopus/audit/005-octopus-configuration.png)

Click the **Audit** link:

[![](https://i.octopus.com/guides/octopus/audit/010-octopus-audit.png)](https://i.octopus.com/guides/octopus/audit/010-octopus-audit.png)

A complete list of records are shown, with filtering available to help find specific events:

[![](https://i.octopus.com/guides/octopus/audit/015-octopus-audit-logs.png)](https://i.octopus.com/guides/octopus/audit/015-octopus-audit-logs.png)

## [](https://octopus.com/docs/guides/deploy-php-app/to-nginx/using-octopus-onprem-jenkins-builtin#conclusion)Conclusion

In this guide we ran through the process of building a complete CI/CD pipeline with:

-   Jenkins building and testing the source code and pushing the package to Octopus.
-   Octopus deploying the package to the **Dev**, **Test**, and **Prod** environments.
-   Email notifications generated when deployments succeed or fail.
-   Manual sign off for deployments to the **Test** and **Prod** environments.
-   Users with limited access to create releases in a subset of environments.

This is a solid starting point for any development team, but Octopus offers so much more! Below are more resources you can access to learn about the advanced functionality provided by Octopus:

-   [The Octopus documentation](https://octopus.com/docs)
-   [The Octopus blog](https://octopus.com/blog)
-   [Upcoming events and webinars](https://octopus.com/blog/tag/Events%20and%20Webinars)
-   [The Octopus community slack channel](https://octopus.com/slack)

Need support? [We're here to help](https://octopus.com/support).