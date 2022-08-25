![](https://miro.medium.com/max/1400/1*DYN-mt93vmFMtbEXXcCNvQ.png)

Powerfull Jenkins

So this is an article for the people who like to play around with a multitude of programming languages or who like their microservices architecture to a multi-lingual affair.

So in terms of development practices and the software lifecycle [[CICD]] plays an important role. now more so than ever given the rise of finely divided micro functioning services that collaborate with each other to produce or achieve the business goals. Where each service is tasked to do a single function or set of closely related functions. This may or in many cases already must have encouraged you to use specific tools or languages you want to write this functionality.

> Note- Please feel free to get back to me if any of the steps are not clear or need further elaboration. I look forward to it and will be a good thing for this extensive guide. No matter how small or primitive a question is welcome.

Now let’s talk about how we can get a single Jenkins server to handle the deployment needs for such a requirement without really having to install a single program-specific dependency on the server itself. Or in tech jargonic words **_Jenkins Configuration as Code._**

Some of you may have guessed by now, we are talking about Docker Agents for Jenkins pipelines. And these Agents act really as plug and play environments for your codebase, wherein they can be built, compiled, tested or even perform tasks that are not possible in a build environment like running a tiny load test server.

## STEP 1 — Installation Recipe

You can follow this beautiful guide to getting started with the installation of Dockerized Jenkins on the VM of your choice maybe EC2 or GCP Compute or whatever suits you.

1.  On your local machine or server of choice
2.  Install Docker

```
sudo docker run  --restart unless-stopped -u root -d  -p 80:8080   -v /home/ubuntu/data/volumes/jenkins-data:/var/jenkins_home  -v /var/run/docker.sock:/var/run/docker.sock  -v "$HOME":/home   jenkinsci/blueocean
```

3\. Now you need to get the logs of the running docker container to get the hexadecimal key which will be used to unlock the Jenkins Admin UI for the first time. It looks like this.  
_Do this at least after 30 seconds as the Jenkins container takes time to initialize_

```
docker logs my-jenkins-conatianer 
```

![](https://miro.medium.com/max/1400/0*2WkiELUcKpJHLgIV.png)

4\. Now open your favorite browser and go to the IP address of the server or instance or _localhost_ if you did this on your local machine.

— You will see a UI where you can enter the key just copied

— Press continue, select install suggested plugin,

— create admin user by entering the username and password of your choice. and once all this is done. This is what you will see.

![](https://miro.medium.com/max/1400/0*rb6QdTRg53Jrn2IP.jpg)

## STEP 2 — CREATE SECRETS & INSTALL PLUGINS

Once the setup is done we need to focus on creating secrets and environment variables which will be required for our pipeline / CI to run and interact with the other systems like docker registry and git and any other API, for example, our Kubernetes Rancher API

![](https://miro.medium.com/max/1400/1*FvRtiCzKrYNsROyAPqwWyw.png)

Now add the secret text that will be injected as global environment variable to each build at runtime. Make sure to note the ID(user-created), we need that later when we write the Jenkinsfile. And we can access the secret to use in your scripts.

In case you keep getting some issues while accessing Jenkins settings, this might help in troubleshooting.

For our setup, we required 3 secrets in total

-   Jenkins bitbucket key: Refer [this document](https://confluence.atlassian.com/bitbucket/set-up-an-ssh-key-728138079.html) to create an ssh key-pair on the server where we installed Jenkins to create an ssh key thus allowing us to use our server to clone bitbucket repo’s.
-   Rancher Kubernetes API Key
-   Rancher Kubernetes API Secret

To get Jenkins up and running without hurdles, we need to install plugins so that we can some cool features like credentials access via env variables that are injected directly to Jenkin's execution.

-   Bitbucket
-   Auth

We need some of these plugins to make sure we can integrate with Bitbucket or Github, etc.

## STEP 3 — TRIGGERING BUILDS

To trigger pipelines when we push code to bitbucket, we need to follow this guide. We are going to use bitbucket thus I will refer to respective links and guides, but doing this for GitHub or any other git server should be similar exercise.

-   Bitbucket Side and Jenkins Side change.
-   [https://mohamicorp.atlassian.net/wiki/spaces/DOC/pages/381386791/Configuring+Webhook+To+Jenkins+for+Bitbucket+Blue+Ocean](https://mohamicorp.atlassian.net/wiki/spaces/DOC/pages/381386791/Configuring+Webhook+To+Jenkins+for+Bitbucket+Blue+Ocean)

## STEP 4— CREATING BUILD PIPELINES

This is where we actually start writing down our build steps. We are going to use a sample GO project to do this. In the root directory of this project create a file name Jenkinsfile. This file will have a JSON like structure which is in fact called a scripted pipeline.

This is what it will look like

**Plug and play docker sandbox image Agents**

```
 agent {     docker { image 'damitj07/go-dep-docker-aws' }  }
```

The above lines tell the Jenkins pipeline to use docker engines as a runtime agent and also to pull the image _\`damitj07/go-dep-docker-aws\`_ to this agent and execute the steps in this runtime environment.

This is where the magic is, and where we can use any of the docker images to get the desired language like NodeJS or Go or PHP or Python or a specific version of that language and other dependency management tools which are required to do this. For example npm or dep, etc

## STEP 5 — THE PIPELINE IN ACTION

![](https://miro.medium.com/max/1400/1*q5rjEZ9l1Nq-R6CLhYn6Hw.png)

So this is what the end results will look like once all the build stages have been configured and ready to go. The main stages as defined in the Jenkinsfile are :

-   Start — Agent setup and allocation
-   Build — Fetch Code and workspace setup and cleanup, Docker Build, Tag
-   Test — Run unit test cases, functional and integration test cases, etc.
-   Artifact — Push the docker image artifact to the remote private repository
-   Deploy — Make API calls to deploy the latest images to Cluster with configuration, migrations, etc.

In our case, we are using Rancher API to connect with Rancher Kubernetes Cluster to deploy workloads in a jiffy. You can find a reference here

## Improvements & Future

-   Create multibranch pipelines to support deployment to various environments.
-   Support Manual Approval for builds Environments
-   Linking with Bitbucket or Github Signing for SSO experiences.

Above are just some things on my plate and I am sure this will vary for you.