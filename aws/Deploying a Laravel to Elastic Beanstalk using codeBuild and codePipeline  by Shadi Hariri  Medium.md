![Laravel and aws](https://miro.medium.com/max/700/1*wrLboCIMbgPtvybPzEca2A.png)

In this article, we will deploy a laravel application to Elastic Beanstalk for a specific environment using AWS codeBuild and codePipeline.

We will do everything in the most straightforward way possible so, I would avoid topics like DNS configuration, etc.

The main steps for what we are going to do is as follow:

1.  Setting up the Elastic Beanstalk
2.  Setting up a proper codePipeline for your environment
3.  Complete AWS ElasticBeanstalk configuration
4.  Setting up a codeBuild for your application
5.  Connecting and finalizing everything

We assume that you already have a laravel application ready to be deployed in one of AWS supporting git providers like Github or bitbucket.

## Step 1: Setting up Elastic Beanstalk

First, you need to create a server (environment) to which you want to deploy your code. We use ASW Elastic Beanstalk to cover everything you need to set up the server, like s3, EC2, etc.  
Before starting, create a .env file for your environment, like .env.staging, remove all sensitive data like APP\_KEY and database information. Push that file to your git repository.

1.  In the AWS search box, type Elastic Beanstalk and select it.
2.  Go to Applications And select `Create new application`.
3.  Select a proper name for the application. Remember to not include your environment name like app\_staging or app\_production.
4.  You will be redirected to the Application environment list. Here you can add different environments for your application. Click on `Create new environment`.
5.  Select `Web server environment` and click Next.
6.  Select a proper name that also includes the name of the environment.
7.  Select PHP from the Platform dropdown menu with the desired version.
8.  Leave everything the same and click `Create environment`.
9.  If you selected Amazon Linux for your environment, create `laravel.conf` in `.platform/nginx/conf.d/elasticbeanstalk` with following code. You can add more configs if you want.

```
location / {try_files $uri $uri/ /index.php?$query_string;gzip_static on;}
```

You will be redirected to the status page of your environment creation. It will take some time. You can continue with the next steps.

## Step 2: Setting up codePipeline

codePipeline helps you to implement continuous delivery and continuous integration. It connects your application to a proper source provider.

1.  In the AWS search box, type codePipeline and select the service.
2.  Click on `Create pipeline`.
3.  Enter a proper name for your pipeline. Remember to include the environment name in your pipeline since different environments get information from different source providers (like different branches) and then click on Next.
4.  Select desired source provider (here, we use bitbucket).
5.  Select `Connect to BitBucket` to authorise AWS to access your repository.
6.  If you did not have any connection from before you can create one.
7.  Select `Connect to Bitbucket`
8.  Enter a proper name for your connection, this does not need to have the environment or application name on it.
9.  Click on `Install a new app`.
10.  Grant access to your bitbucket account by clicking on `Grant access`.
11.  You will be redirected to the previous page. Click `connect`.
12.  Select desired repository name.
13.  Select desired branch for your environment and click `Next`.
14.  Select `AWS codeBuild`.
15.  Here you need to provide a codeBuild for your pipeline. For that, click on `Create project`. It will redirect you to the codeBuild environment to create a proper code build.
16.  Select the desired name, including your environment name as well.
17.  Select a preferred operating system for your build environment.
18.  Leave everything as it is and select `Continue to CodePipeline`
19.  You will be redirected to the codePipeline page. Click `Next`.
20.  In the next step, select `AWS Elastic Beanstalk`. Select desired application name and application environment, and click `Next`.
21.  Review everything and click `Create pipeline`. Your codePipeline will fail since we have not completed the codeBuild.

## Step 3: Complete AWS ElasticBeanstalk configuration

By now, your environment should be created. Go to AWS ElasticBeanstalk and select your environment.

1.  Select `configuration`.
2.  In row `Database`, click `Edit`.
3.  Enter desired username and password and click `Apply`.
4.  Wait until changes are applied.
5.  In the row called `Software`, click on `Edit`.
6.  As we know laravel applications index file is the public folder. In the `Document root` section, enter `/public` and click `Apply`.
7.  In Environments, enter all your variables from the .env file that you removed, including the database info that you created.
8.  To find database info, in the search box, search for `RDS` and select it.
9.  You can see there is one DB instance. Click on that.
10.  In tab `Connectivity and security`, you can find the `DB port` and `DB host`.
11.  In tab `Configuration`, you can see the `DB name`.
12.  Apply changes and wait until it applies. It will fail. Since our codeBuild is not configured, and there is no code in the server, but we want to read info from public folder. We will fix this issue when we finished the build configuration.

## Step 4: Setting up codeBuild

We have already created the basic codeBuild. Let’s configure it.

1.  In the AWS search box, type codeBuild and select the service.
2.  Select the created codeBuild.
3.  Click `Edit` and select `Environment`
4.  In the `Environment Variables` section, add the .env files info you entered in the previous section.
5.  Click `Update environment`.
6.  Click on `Edit` and select `buildspec`
7.  Click Insert `build commands`, then click `Switch to Editor`.
8.  In this editor, you enter all the steps you need for building the application. You can use the following example: remember to change the .env.staging to your desired environment.

```
version: 0.2phases:   install:     runtime-versions:         php: 7.4         nodejs: 12.x     commands:         - apt-get update -y         - apt-get install -y libpq-dev libzip-dev         - apt-get install -y php-pgsql         - curl -sS <https://getcomposer.org/installer> | php -- --install-dir=/usr/local/bin --filename=composer   pre_build:     commands:         - cp .env.staging .env         - composer install         - npm install   build:     commands:         - npm run production         - php artisan migrate --force         - php artisan db:seedartifacts:   files:         - '**/*'   name: $(date +%Y-%m-%dT%H:%M:%S).zipproxy:   upload-artifacts: yes   logs: yes
```

9\. Update the `buildspec`.

## Step 5: Finishing up

Now everything is ready to be deployed.

1.  Select `codePipeline` from the search box
2.  Select created pipeline
3.  Click on `Release changes`.
4.  Wait until all steps pass.
5.  If the build step failed in the migration step, try the following steps.
6.  Go to this page: [https://ip-ranges.amazonaws.com/ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json)
7.  Search for CODEBUILD and choose your region. Copy the IP.
8.  Go to the `RDS` and select your database instance.
9.  In the security group list, select the one with `EC2 Security Group Inbound`.
10.  Select the `Inbound rules` tab
11.  Click on `Edit` Inbound rules.
12.  Click Add rule. Select All TCP for type, custom for source, and add the copied IP in the search input in the opened form.
13.  Click on `Save rules` and try again.

## Conclusion

Now everything should work. If you check your website, you should be able to see Laravel’s first page. From now on every time you push something to your selected branch, it will also be deployed in your applications, which takes 5 to 10 minute.