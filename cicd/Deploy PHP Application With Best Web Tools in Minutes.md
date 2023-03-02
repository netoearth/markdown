![Deploy PHP](https://www.cloudways.com/blog/wp-content/uploads/Deploy-PHP.jpg)

Developing web applications is an exciting exercise, as it involves new challenges that help us in gaining valuable experience and learning. The real pain comes in when the [developer](https://www.cloudways.com/blog/php-developer/) approaches to deploy PHP on the server . You might have noticed within a software development agency that they take couple of weeks to develop a single application.

But as soon as they get ready for deployment, anxiety starts to prevail. For instance, when an organization needs to migrate the website smoothly on production servers, sometimes, the size of application with database is quite big that it takes time to transfer all the files to live [PHP web hosting](https://www.cloudways.com/en/php-hosting.php) servers.

1.  [GitHub](https://www.cloudways.com/blog/deploy-php-application/#github)
    1.  [Signup & Launch Server>](https://www.cloudways.com/blog/deploy-php-application/#signup)
    2.  [Generating SSH Keys](https://www.cloudways.com/blog/deploy-php-application/#generate)
    3.  [Upload SSH Key To GitHub Repository](https://www.cloudways.com/blog/deploy-php-application/#upload)
    4.  [Copy the SSH Address Of Repository](https://www.cloudways.com/blog/deploy-php-application/#copy)
    5.  [Deploy Code from Your Repository](https://www.cloudways.com/blog/deploy-php-application/#deploy)
    6.  [Repository Successfully Cloned](https://www.cloudways.com/blog/deploy-php-application/#reposit)
2.  [DeployHQ](https://www.cloudways.com/blog/deploy-php-application/#delpoyHQ)
    1.  [Create a DeployHQ account](https://www.cloudways.com/blog/deploy-php-application/#account)
    2.  [Create a new project in DeployHQ](https://www.cloudways.com/blog/deploy-php-application/#project)
    3.  [Connect DeployHQ with your code repository](https://www.cloudways.com/blog/deploy-php-application/#connect)
    4.  [Add Path Of Github Repository](https://www.cloudways.com/blog/deploy-php-application/#path)
    5.  [Configure the server](https://www.cloudways.com/blog/deploy-php-application/#configure)
    6.  [Deploy](https://www.cloudways.com/blog/deploy-php-application/#deploy2)
3.  [DeployBot](https://www.cloudways.com/blog/deploy-php-application/#bot)
    1.  [Deployment using DeployBOT](https://www.cloudways.com/blog/deploy-php-application/#using)
    2.  [Create a DeployBot Account](https://www.cloudways.com/blog/deploy-php-application/#botaccount)
    3.  [Connect a repository](https://www.cloudways.com/blog/deploy-php-application/#botrepost)
    4.  [Configure the environment](https://www.cloudways.com/blog/deploy-php-application/#botenv)
    5.  [Configure the server](https://www.cloudways.com/blog/deploy-php-application/#botserv)
    6.  [Deploy](https://www.cloudways.com/blog/deploy-php-application/#deploy3)
4.  [Envoyer](https://www.cloudways.com/blog/deploy-php-application/#envoyer)
    1.  [Create Envoyer Account and Add a Project](https://www.cloudways.com/blog/deploy-php-application/#accountenv)
    2.  [Connect to a Repository](https://www.cloudways.com/blog/deploy-php-application/#repostenv)
    3.  [Add the Server](https://www.cloudways.com/blog/deploy-php-application/#addenv)
    4.  [Final Deployment](https://www.cloudways.com/blog/deploy-php-application/#finaldeploy)
    5.  [Deployment Folders on Cloudways](https://www.cloudways.com/blog/deploy-php-application/#folders)
5.  [Final Words](https://www.cloudways.com/blog/deploy-php-application/#final)

For databases, you must be very alert, otherwise, it can cause problems if not transferred properly.

Developers use different tactics to deploy applications from local to live PHP servers). If your website contains less files, then you can easily deploy it manually.

## Nothing as Easy as Deploying PHP Apps on Cloud

With Cloudways, you can have your PHP apps up and running on managed cloud servers in just a few minutes.

But, what if you are deploying a large number of files that contain dev, configs, assets etc. The developers tend to prefer using the best practices for PHP app deployment with which they can deploy PHP applications, test and track the [PHP error logs](https://www.cloudways.com/blog/php-error-logging/) of possible failures.

Cloudways provides a smooth function for deploying PHP applications that includes 1-click app install for PHP, Laravel and other applications. You just need to signup and launch your server along with desired application(s). There are also other ways to deploy PHP applications and automate the process using third party services like:

1.  GitHub
2.  Envoyer
3.  DeployBot
4.  DeployHQ

In this article, I’ll cover the above services and will deploy PHP application on Cloudways. These services will help you to automate deployment processes even if you don’t know much about Circleci and [Travis CI](https://www.cloudways.com/blog/php-continuous-integration-travis-ci/).

The biggest advantage is that you are just not limited to one-time deployment, instead you can connect the server with application every time, and just push the updated code to deploy it within minutes!

Let’s start with the GitHub.

## **GitHub**

If you are a developer, then you must know about Git for source code management. Developers use GitHub when it comes to interaction with multiple team members and open source contributors for developing coding solutions. I’ve written a short series on [Git for beginners](https://www.cloudways.com/blog/git-tutorial-for-beginners-version-control/) covering, commands cheat sheets, Branches, Conflicts etc.

The best thing about Git is that it allows developers to create custom workflows manually, or by integrating third party PHP deployment tools.

Cloudways allows you to deploy code of your application from your Git repositories. Your Git repository must support Git over SSH for this to work. For Git deployment, you must follow simple steps given below.

### Signup & Launch Server

First of all, signup at Cloudways and launch your server and application. Next, move to the **Application tab** by selecting any app from application page .

![cloudways-server](https://www.cloudways.com/blog/wp-content/uploads/1-54-300x148.png)

### Generating SSH Keys

Here, you must download SSH keys by moving to Deployment via Git tab,

We will use these keys to allow access from your Cloudways server to your git repository. Now click on the **Generate SSH Keys** button to generate the keys.

![generate-ssh-key](https://www.cloudways.com/blog/wp-content/uploads/2-54-300x147.png)

Now, click on **Download SSH Keys** to download SSH Public Key that we will use in the next step.

![view-ssh-key](https://www.cloudways.com/blog/wp-content/uploads/3-53-300x148.png)

![ssh-key](https://www.cloudways.com/blog/wp-content/uploads/4-21-300x148.jpg)

### Upload SSH Key To GitHub Repository

On Github, navigate to the repository and find the code which you want to deploy. If you are using another Git service, you will have to find the equivalent way of deploying them. Go to **Settings -> Deploy keys** and click on the **Add Deploy Key** button to add the SSH key. You can also give a name to this key in the title field and copy the key to the box. Click on the **Add Key** button to save the SSH key.

![](https://www.cloudways.com/blog/wp-content/uploads/cloudways-save-ssh-keys.png)

### Copy the SSH Address Of Repository

Copy the **repository address** as shown in the image below. Make sure to copy the **SSH address** as other formats (like HTTPS) are not supported.

![](https://www.cloudways.com/blog/wp-content/uploads/add-ssh-key-github.png)

### Deploy Code from Your Repository

1.  Go back to Cloudways console. Paste the SSH address you got in **Step 4** into the **Git Remote Address** ” field.
2.  Select the branch of your repository you want to deploy from. In this example, we are using the **master** ” branch.
3.  Type the deployment path (i.e. the folder in your server where the code will be deployed). Make sure to end it with a backslash **(/)** . If you will leave this field empty, the code will be deployed to _public\_html/_ .
4.  Click on the **Start Deployment** button to deploy your code to the selected path.

![deployment-via-git](https://www.cloudways.com/blog/wp-content/uploads/1-38-300x148.jpg)

### Repository Successfully Cloned

You will get a notification once the deployment process finishes.

You have further options to **delete** the repository from the server (no files will be deleted, see FAQ below). **Pull** the latest changes or **change** the branch you deploy from.

![successfully-deploy-cloudways](https://www.cloudways.com/blog/wp-content/uploads/dep-300x148.jpg)

## DeployHQ

DeployHQ is an amazing PHP deployment tool to automate your deployments from Git, Mercurial, and Subversion code repositories. Git-based deployment is now becoming de-facto standard in any good development agency. This makes lives easy as it reduces the hassle of uploading and downloading source files.

In this post, I will describe how you can integrate DeployHQ with your web app that you have [hosted on Cloudways](https://www.cloudways.com/). This integration will ensure on-the-go deployment of the code. With Cloudways [staging-friendly environment](https://www.cloudways.com/blog/announcing-enhanced-staging/), developers can experiment with their code as much as they want.

So, here are the steps.

### Create a DeployHQ account

Register an account on DeployHQ. (You can use this free account for one project and 10 deployments per day)

### Create a new project in DeployHQ

You will need to create a project to start your deployment process.

![](https://www.cloudways.com/blog/wp-content/uploads/image11-98.png) ![](https://www.cloudways.com/blog/wp-content/uploads/image4-172.png)

### Connect DeployHQ with your code repository

Enter the details of your code repository (or “code repo”). DeployHQ has out-of-the-box support for popular code hosting sites, like Github, Bitbucket, etc.

### Add Path Of Github Repository

To begin with deployment, you need to add the repository path from GitHub like this.

![](https://www.cloudways.com/blog/wp-content/uploads/image30-6.png)

### Configure the server

Select SSH/SFTP as protocol

![](https://www.cloudways.com/blog/wp-content/uploads/image24-15.png)

Then, fill up the SSH Configuration.

![](https://www.cloudways.com/blog/wp-content/uploads/image2-211.png)

For example, you may need to change the branch from master to any other branch that you like to deploy from.

Now, click “Save”. You have successfully configured your server

![](https://www.cloudways.com/blog/wp-content/uploads/image29-9.png)

### ![](https://www.cloudways.com/blog/wp-content/uploads/image1-218.png)

### Deploy

Click “Deploy Now”. On the deployment screen, you can click the “Deploy” button to start the deployment process instantly.

![](https://www.cloudways.com/blog/wp-content/uploads/image19-32.png)

![](https://www.cloudways.com/blog/wp-content/uploads/image7-111.png)

## DeployBot

For software development teams, automated deployments have become imperative to the process flow. Manual deployments are error-prone. They sap energy and effort of the team members.

Any managed cloud hosting platform which claims to be a high-quality solution must provide the ability to automate deployment. This is why Cloudways does exactly that. Earlier, I had explained how to automate deployment using DeployHQ.

You can also achieve deployment automation using DeployBOT (or dploy.io).

### Deployment using DeployBOT

DeployBot is a PHP deployment tool that connects your code repositories to your servers. In this article, I will describe how you can deploy your code on a Cloudways server using DeployBot.

### Create a DeployBot Account

You can create an account for free which you can use for a single repository.

### Connect a repository

DeployBot has out-of-the-box support for GitHub and BitBucket, but you may also connect to other repositories.

![](https://www.cloudways.com/blog/wp-content/uploads/image33-2.png)

### Configure the environment

By default, the deployment will be manual. However, you can change it to ‘automatic’ to start the deployment whenever a change occurs in your repo. You may also need to change ‘master’ to the desired branch which you want to use.

![](https://www.cloudways.com/blog/wp-content/uploads/image3-192.png)

### Configure the server

Select SFTP under Files section.

![](https://www.cloudways.com/blog/wp-content/uploads/image34-5.png)

You can get the login credentials from the Master Credentials section within the Cloudways Server Console.

![credentials](https://www.cloudways.com/blog/wp-content/uploads/Capture-3-300x136.jpg)

Enter the SFTP into DeployBot:

![](https://www.cloudways.com/blog/wp-content/uploads/image14-62.png)

### Deploy

Navigate to Dashboard and click “Deploy”. Then, on the deployment screen, click “Start deployment”.

![](https://www.cloudways.com/blog/wp-content/uploads/image5-143.png)

Note: You may click on “Preview the files to be deployed” button to have a look which files will change (or get deleted) by the deployment, as any manual change previously done on SFTP will be overwritten.

## Envoyer

Envoyer is another PHP deployer tool that helps in deploying web applications on the hosting platforms. The best thing about this PHP deployer tool is its zero downtime during deployment. This means that your application and the customers using it, are not even aware of the fact that a new version has been pushed

Envoyer works well with major repository management platforms such as GitLab and Bitbucket. Other benefits include unlimited deployments and team members.

The following blog will guide you through the process of deploying applications on Cloudways using Envoyer.

### Create Envoyer Account and Add a Project

[Create an Envoyer account](https://envoyer.io/) and login. Next, add a new project.

![](https://www.cloudways.com/blog/wp-content/uploads/image28-9.png)

### Connect to a Repository

Next, connect your repository. The good thing about Envoyer is that you can host this repository on any platform including Github, Bitbucket or any other self-hosted repository.

![](https://www.cloudways.com/blog/wp-content/uploads/image15-57.png)

### Add the Server

The next step is to integrate the Cloudways server. For this, navigate to the Server Tab and click the Add Server button.

Next, login to your Cloudways account and get the credentials from Master Credentials section for adding the server.

![](https://www.cloudways.com/blog/wp-content/uploads/m-300x148.jpg)

Fill out all the required fields and enter the complete path of your Cloudways application and save the server.

![](https://www.cloudways.com/blog/wp-content/uploads/add-server-envoyer.png)

After saving the server, you will get an SSH key.

![](https://www.cloudways.com/blog/wp-content/uploads/envoyer-public-ssh-key.png)

Copy the key and go to your Cloudways Console. Click the ‘SSH Public Keys’ button. Give a label to your key and click Submit.

![](https://www.cloudways.com/blog/wp-content/uploads/last-300x148.jpg)

After adding the SSH key, you should be able to connect to your server. Click the tiny Refresh button to test the connection status.

Envoyer allows you to manage your environment. Click Manage Environment and enter the SSH key. You can now set the contents of your environment.

### Final Deployment

At this point, everything is correctly set up. The final step is to click the Deploy button for actual project deployment. You can also deploy applications via Git Push by selecting ‘Deploy when code is pushed’ option in the settings.

![](https://www.cloudways.com/blog/wp-content/uploads/image27-11.png)

Envoyer will ask you from which branch or tag you need to deploy the application code. I have selected the Default Branch, which is the master branch as well.

Hit the Deploy button now. You can see the deployment process in the deployment tab. You can get additional information related to deployment by clicking the arrow button next to the deployment status.

You can clearly see that Envoyer takes minimal time to deploy the application on your server.

## Deployment Folders on Cloudways

Once deployed, login to the SSH terminal on Cloudways, and navigate to your application folder. You will find two folders, current & releases.

The current folder contains the main application which is in development and the release folder contains the previous releases folder, named with date and time of the deployments.

You can check the status of your application from three locations, (New York, London, Singapore). In case of a disaster, you have an option of rolling back the current deployment. For this, enable the option by providing the health check URL in the settings. Additionally, you can set up heartbeats to monitor [CRON jobs for your application](https://www.cloudways.com/blog/schedule-cron-jobs-in-php/). You can also [set up a notification channel](https://www.cloudways.com/blog/real-time-php-notification-system/) like Slack or Hipchat to receive deployment related notifications.

## **Final Words**

Now that you have learned multiple ways to deploy PHP application(s) on Cloudways, you can now test them by [creating the account on Cloudways](https://platform.cloudways.com/signup) and other connected services. A good deployment workflow always helps you in creating web applications smoothly. You don’t need to push individual files after every update.

I hope you must have liked the methods which I have detailed in this article. If you have developed any other method or PHP deployer tool already, feel free to share your valuable suggestions and opinions in the comment section below. I’ll test them as well, and will add in this article.

Share your opinion in the comment section. COMMENT NOW

### Share This Article

Customer Review at ![](https://www.cloudways.com/blog/wp-content/uploads/g2-crowd-logo.png)

#### “Cloudways hosting has one of the best customer service and hosting speed”

Sanjit C \[Website Developer\]

![](https://secure.gravatar.com/avatar/bcc103fd9b17c6c6edf5b820806b76de?s=128&d=mm&r=g)

#### [Shahroze Nawaz](https://www.cloudways.com/blog/author/shahrozenawaz/)

Shahroze is a PHP Community Manager at Cloudways - A Managed PHP Hosting Platform. Besides his work life, he loves movies and travelling.