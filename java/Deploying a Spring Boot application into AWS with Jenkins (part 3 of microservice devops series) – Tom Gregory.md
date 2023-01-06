Last Updated on August 4, 2022

Welcome to the third and final part of this three part series where we take a Spring Boot application from inception to deployment, using Docker and all the current best practices for continuous integration with Jenkins.

**In this final article, we’ll take the Docker image we pushed to Docker Hub in [Part 2](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/) and deploy it into AWS Elastic Container Service (ECS). AWS ECS represents the simplest way to orchestrate Docker containers in AWS, and it integrates with other services such as the Application Load Balancer and CloudFormation. Of course we’ll be automating everything, and deploying our microservice from Jenkins.**

This series of articles includes:

-   [Part 1: writing a Spring Boot application and setting up a Jenkins pipeline to build it](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/)
-   [Part 2: wrapping the application in a Docker image, building it in Jenkins, then pushing it to Docker Hub](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/)
-   **Part 3: deploying the Docker image as a container from Jenkins into AWS (this article)**

If you haven’t yet read Part 1 or Part 2, then check them out. We’ll be building on top of the concepts introduced there. If you’re all up to date, then let’s get right into it!

UPDATED in June 2021 to use the latest versions of Gradle and Jenkins.

## 1\. Overview

In [part 1](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) we built a Spring Boot API application and got it building in a bootstrapped version of Jenkins. In [part 2](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/), we took this a step further by Dockerising our application, then pushing it to Docker Hub.

We ended up with a nifty Jenkins pipeline that looked like this:

![Part 2 Jenkins pipeline](https://tomgregory.com/wp-content/uploads/2020/03/image-53-1024x491.png)

Our Docker image was pushed to Docker Hub, the central Docker registry, during the final _Push Docker image_ stage of the pipeline.

![](https://www.lucidchart.com/publicSegments/view/e3f52c14-67db-4a13-80a8-c912508e2efa/image.png)

The last piece to this puzzle is to pull the image back onto a server or cloud service, or in other words _deploy_ the image. To achieve this our weapon of choice will be the popular cloud service _AWS_. ![☁️](https://s.w.org/images/core/emoji/14.0.0/svg/2601.svg)

To create resources in AWS, we’ll be using the AWS templating language CloudFormation. CloudFormation uses simple YAML templates that describe the end state of your desired infrastructure. This will all be orchestrated from Jenkins, as shown below:

[![](https://www.lucidchart.com/publicSegments/view/cadc2da7-d34e-449c-83b3-03f81604d98d/image.png)](https://www.lucidchart.com/publicSegments/view/cadc2da7-d34e-449c-83b3-03f81604d98d/image.png)

Jenkins will deploy a CloudFormation template into AWS. AWS will then go ahead and provision all the resources that we’ll describe in our template, including:

1.  **ECS cluster** – a location where we can deploy ECS tasks
2.  **ECS task definition** – a description of how our Docker image should run, including the image name and tag, which for us is [image pushed to Docker Hub](https://hub.docker.com/r/tkgregory/spring-boot-api-example) from the example in [the previous article](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/). The task definition is used to run an ECS task, which represents a running Docker container.
3.  **ECS service** – creates and manages ECS tasks, making sure there’s always the right number running
4.  **Security group** – defines what ports and IP addresses we want to allow inbound and outbound traffic from/to

The end result will be that we, the user, will be able to make API calls to our Spring Boot API application running as a Docker container in AWS, via its public IP address.

If you want to follow along with this working example, then clone the [Spring Boot API application](https://github.com/tkgregory/spring-boot-api-example.git) from GitHub.

```
git clone https://github.com/tkgregory/spring-boot-api-example.git
```

## 2\. Pre-requisites to working example

#### Default VPC and subnet details

If you don’t already have an AWS account head on over to [sign up](https://portal.aws.amazon.com/billing/signup).

Once you have an account, you can use the default VPC and subnets that AWS creates for you in the rest of this example. Just go to _Services_ > _VPC_, and select _Subnets_ from the left-hand menu:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-1024x340.png)

You can pick any of these subnets to deploy into. Make note of the _Subnet ID_ and also your region (top right), as you’ll need this information later.

#### Access credentials

You’ll also need an AWS access key and secret key. You can generate this by clicking on your name in the top bar, then clicking on _My Security Credentials_ > _Access key_s. Then select _Create New Access Key_:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-1-1024x453.png)

Make sure to securely save the _Access Key ID_ and _Secret Access Key_ as you won’t be able to get these details again later on.

#### AWS CLI and credentials configuration (optional)

For the most interactive experience [download](https://aws.amazon.com/cli/) the AWS Command Line Interface (CLI). This is only required if you want to run CloudFormation from your machine, not Jenkins.

With the AWS CLI installed you can run `aws configure` to setup your credentials:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-2.png)

Then they get stored in the _~/.aws/credentials_ file under the _default_ profile:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-3.png)

You should now be able to run AWS CLI command against your account such as `aws s3 ls`.

#### Docker Hub

You’ll need a [Docker Hub](https://hub.docker.com/signup) account as the Jenkins pipeline pushes a Docker image. Later on it also pulls it when deploying to AWS. Check out [this section](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins#dockerhub) of the part 2 article for full instructions.

## 3\. Creating a CloudFormation template for the application

We’ll add our CloudFormation template in a file named _ecs.yml_ in the root of the project:

```
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  SubnetID:
    Type: String
  ServiceName:
    Type: String
  ServiceVersion:
    Type: String
  DockerHubUsername:
    Type: String
Resources:
  Cluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: deployment-example-cluster
  ServiceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: ServiceSecurityGroup
      GroupDescription: Security group for service
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0
  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: !Sub ${ServiceName}-task
      Cpu: 256
      Memory: 512
      NetworkMode: awsvpc
      ContainerDefinitions:
        - Name: !Sub ${ServiceName}-container
          Image: !Sub ${DockerHubUsername}/${ServiceName}:${ServiceVersion}
          PortMappings:
            - ContainerPort: 8080
      RequiresCompatibilities:
        - EC2
        - FARGATE
  Service:
    Type: AWS::ECS::Service
    Properties:
      ServiceName: !Sub ${ServiceName}-service
      Cluster: !Ref Cluster
      TaskDefinition: !Ref TaskDefinition
      DesiredCount: 1
      LaunchType: FARGATE
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: ENABLED
          Subnets:
            - !Ref SubnetID
          SecurityGroups:
            - !GetAtt ServiceSecurityGroup.GroupId
```

-   **_AWSTemplateFormatVersion_** defines the template version (you’d never have guessed, I know). If using IntelliJ IDEA make sure to add this section to enable helpful parsing by the [AWS CloudFormation](https://plugins.jetbrains.com/plugin/7371-aws-cloudformation) plugin.
-   the _**Parameters**_ section defines what information needs to be passed into the template:
    -   **SubnetId** – the id of the subnet (i.e. internal AWS network) into which our container will be deployed
    -   **ServiceName** – the name of our application
    -   **ServiceVersion** – the version of our application
    -   **DockerHubUsername** – the Docker Hub account from which the Docker image should be pulled
-   the _**Resources**_ section defines the AWS resources to be created
    -   **ServiceSecurityGroup** – a security group allowing inbound traffic to our service from any IP address on port 8080
    -   **TaskDefinition** – our ECS task definition, defining the Docker image to deploy, the port to expose, and the CPU & memory limits
    -   **Service** – our ECS service, defining how many instances we want, selecting the launch type as _FARGATE_, and referencing our task definition and security group

**Fargate**  
The _Fargate_ launch type means that AWS will handle provisioning any underlying resources (i.e. VMs) required to run our container, so there’s less for us to manage

## 4\. Building a CloudFormation environment from Gradle

Doing as much as possible from our build automation tool, in this case Gradle, makes sense. We’ll setup Gradle to deploy our CloudFormation to AWS by simply running a task. Fortunately a plugin exists to do the hard work, called [jp.classmethod.aws.cloudformation](https://plugins.gradle.org/plugin/jp.classmethod.aws.cloudformation).

Apply the plugin in the plugins section of _build.gradle_:

```
plugins {
    ...
    id 'jp.classmethod.aws.cloudformation' version '0.41'
}
```

We’re going to define a variable for `imageName`, and allow passing in a custom Docker Hub username:

```
String dockerHubUsernameProperty = findProperty('dockerHubUsername') ?: 'tkgregory'
String imageName = "${dockerHubUsernameProperty}/spring-boot-api-example:$version"
```

-   **dockerHubUsername** – defines the account to/from which the Docker image should be pushed/pulled. You can pass in your own (we’ll set this up later) or use mine. In order for the _Push Docker Image_ stage of the pipeline to work though, you’ll need your own Docker Hub account.
-   **imageName** – we’ve improved this from [part 2](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/), by allowing you to dynamically pass in your own Docker Hub account name

We’ll now configure the Gradle CloudFormation plugin to call AWS in the right way. Add the following configuration at the bottom of _build.gradle_:

```
cloudFormation {
    stackName "$project.name-stack"
    stackParams([
            SubnetID: findProperty('subnetId') ?: '',
            ServiceName: project.name,
            ServiceVersion: project.version,
            DockerHubUsername: dockerHubUsernameProperty
    ])
    templateFile project.file("ecs.yml")
}
```

-   _**stackName**_ will just be the project name. A CloudFormation _stack_ is an instance of a CloudFormation template once it’s in AWS, and represents all the resources that have been created.
-   _**stackParams**_ are the parameters that need to be passed into the CloudFormation template:
    -   **_SubnetID_** – we’ll pass this into the Gradle build as a property
    -   **_ServiceName_** – once again we can use the project name
    -   **_ServiceVersion_** – we’ll use the project version
    -   **_DockerHubUsername_** – using the `dockerHubUsernameProperty` variable defined above
-   **_templateFile_** is a reference to our CloudFormation file

Lastly, we add the following which configures the plugin to run against a specific AWS region, by passing in the Gradle property `region`.

```
aws {
    region = findProperty('region') ?: 'us-east-1'
}
```

The full build.gradle should now look [like this](https://github.com/tkgregory/spring-boot-api-example/blob/master/build.gradle).

#### Testing out the CloudFormation stack creation

If you setup the AWS CLI with your credentials in [the AWS prerequisites section](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/#prerequisites), then you can run the following command to test out your changes (if not we’ll be running it in Jenkins in the next section):

`./gradlew awsCfnMigrateStack awsCfnWaitStackComplete -PsubnetId=<your-subnet-id> -Pregion=<your-region>`

This will initiate the creation of a CloudFormation stack (`awsCfnMigrateStack`) then wait for it to complete successfully (`awsCfnWaitStackComplete`). It may take a few minutes.

You’ll be able to see the stack in the AWS Console too, by going to _Services_ > _CloudFormation_ > _Stacks_:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-4-1024x386.png)

Delete the stack for now as we’ll be creating it in Jenkins in the next section. Run `./gradlew awsCfnDeleteStack awsCfnWaitStackComplete`.

## 5\. Deploying to AWS from Jenkins

Now that we can build our CloudFormation stack from Gradle, let’s get it working in an automated way by integrating it into our Jenkins continuous integration pipeline.

You’ll remember from the [part 2 article](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/) that we had a project setup to get Jenkins running in a Docker container, with all the jobs being created automatically from a _seed-job_. These jobs would reference pipeline definitions in _Jenkinsfile_ files in our Spring Boot API application project.

**Fast pass**  
To get up and running straight away just grab the [jenkins-demo](https://github.com/tkgregory/jenkins-demo.git) project from GitHub. You can make the below changes yourself, by starting off from the part 2 _[theme-park-job-docker](https://github.com/tkgregory/jenkins-demo/tree/theme-park-job-docker)_ branch.

Alternatively, jump right in by checking out the [_theme-park-job-aws_](https://github.com/tkgregory/jenkins-demo/tree/theme-park-job-aws) branch which has everything setup ready to go (fast forward to the [_Starting Jenkins_](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/#startingjenkins) section, if you like).

#### AWS plugins for Jenkins

To allow integration with AWS we’ll add two plugins to our Jenkins Docker image. Remember that these are defined in _[plugins.txt](https://github.com/tkgregory/jenkins-demo/blob/master/plugins.txt)_, and get added when we build the Docker image.

Let’s add these additional entries to the end of _plugins.txt_:

```
aws-credentials:191.vcb_f183ce58b_9
pipeline-aws:1.43
```

-   the **_aws-credentials_** plugin allows us to configure AWS credentials from the Jenkins credentials configuration screen
-   the **_pipeline-aws_** plugin allows us to use the AWS credentials in a pipeline stage. We’ll use them in the stage which calls the Gradle task to deploy the CloudFormation to AWS.

#### A new job to deploy to AWS

The _createJobs.groovy_ file uses the Jenkins Jobs DSL plugin to create various jobs that reference a _Jenkinsfile_ in a specific repository. We’re going to add a new job to this file to reference a new Jenkinsfile which we’ll create shortly in the Spring Boot API application repository.

Add the following section to _createJobs.groovy_:

```
pipelineJob('theme-park-job-aws') {    definition {        cpsScm {            scm {                git {                    remote {                        url 'https://github.com/tkgregory/spring-boot-api-example.git'                    }                    branch 'master'                    scriptPath('Jenkinsfile-aws')                }            }        }    }}
```

-   this new job will be called _theme-park-job-aws_. Remember this project is all about exposing an API to get and create theme park rides? ![🎢](https://s.w.org/images/core/emoji/14.0.0/svg/1f3a2.svg)
-   the repository for this Jenkins job will be our Spring Boot API example
-   we won’t use the standard Jenkinsfile location, but a file called _[Jenkinsfile-aws](https://github.com/tkgregory/spring-boot-api-example/blob/master/Jenkinsfile-aws)_. This file defines the pipeline to run. We have several other Jenkinsfile files, for the three different parts of this series of articles.

If you want to point this Jenkins instance at your own repo, don’t forget to change the remote `url` property above. Also, don’t forget to update _seedJob.xml_ to point at the correct `url` and branch `name` (e.g. _theme-park-job-aws_) for grabbing the _createJobs.groovy_ file.

#### Defining the pipeline

Back in the [Spring Boot API application project](https://github.com/tkgregory/spring-boot-api-example.git), let’s add the _Jenkisfile-aws_ file to define our new pipeline:

```
pipeline {
    agent any

    triggers {
        pollSCM '* * * * *'
    }
    stages {
        stage('Build') {
            steps {
                sh './gradlew assemble'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }
        stage('Build Docker image') {
            steps {
                sh './gradlew docker'
            }
        }
        stage('Push Docker image') {
            environment {
                DOCKER_HUB_LOGIN = credentials('docker-hub')
            }
            steps {
                sh 'docker login --username=$DOCKER_HUB_LOGIN_USR --password=$DOCKER_HUB_LOGIN_PSW'
                sh './gradlew dockerPush -PdockerHubUsername=$DOCKER_HUB_LOGIN_USR'
            }
        }
        stage('Deploy to AWS') {
            environment {
                DOCKER_HUB_LOGIN = credentials('docker-hub')
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: env.AWS_REGION) {
                    sh './gradlew awsCfnMigrateStack awsCfnWaitStackComplete -PsubnetId=$SUBNET_ID -PdockerHubUsername=$DOCKER_HUB_LOGIN_USR -Pregion=$AWS_REGION'
                }
            }
        }
    }
}
```

-   the first 4 stages are exactly the same as in the [previous article](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/)
-   we’ve added an additional _Deploy to AWS_ pipeline stage
-   we use the _environment_ section to inject the _docker-hub_ credential. We’ll need this only to get the username to pass as the `dockerHubUsername` Gradle property.
-   in the _steps_ definition of this stage we use the [_withAWS_](https://jenkins.io/doc/pipeline/steps/pipeline-aws/#withaws-set-aws-settings-for-nested-block) method. This uses the credentials named _aws-credentials_ (which we’ll setup shortly) to create a secure connection to AWS.
-   the main part of this stage is to run the Gradle `awsCfnMigrateStack` to apply the CloudFormation stack and `awsCfnWaitStackComplete` to wait for it to complete. We also pass in the `subnetId` and `dockerHubUsername` as Gradle properties.

#### Starting Jenkins

Start up Jenkins from the _jenkins-demo_ project now by running `./gradlew docker dockerRun`. This may take some time, but will build the new Docker image and then run the image as a Docker container:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-5-1024x595.png)

Browse to [http://localhost:8080](http://localhost:8080/) and you should have an instance of Jenkins running with a default _seed-job_.

![](https://tomgregory.com/wp-content/uploads/2020/04/image-6-1024x317.png)

#### Configuring Jenkins credentials

Click on _Manage Jenkins_ in the left hand navigation, then _Manage Credentials_. Click on the _Jenkins_ scope, then _Global credentials_. Click _Add Credentials_ on the left-hand side:

![add Jenkins credentials](https://tomgregory.com/wp-content/uploads/2020/03/credentials-add-1024x252.png)

1.  select _AWS credentials_ from the _Kind_ drop down list
2.  enter an _ID_ of _aws-credentials_
3.  enter your _Access Key ID_ and _Secret Access Key_ that you setup in the [prerequisites](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/#prerequisites) section
4.  select _OK_

![](https://tomgregory.com/wp-content/uploads/2020/04/aws-credentials-1024x453.png)

Since our pipeline also includes a stage to deploy to Docker Hub, you’ll also need to add a _Username with password_ type credential with your Docker Hub login details. See [part 2](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins#credentials) for full details on settings this up.

![Docker Hub credentials](https://tomgregory.com/wp-content/uploads/2020/03/credentials-form-1024x413.png)

#### Configuring Jenkins Global properties

There are 2 properties that we need to configure as _Global properties_. These are environment variables that are made available in any job or pipeline. Go to _Manage Jenkins_ > _Configure System_.

Select _Environment variables_ under _Global properties_, and click the _Add_ button to add an environment variable. We need two variables, _AWS\_REGION_ and _SUBNET\_ID_:

![](https://tomgregory.com/wp-content/uploads/2020/09/jenkins-environment-1-1024x436.png)

Don’t forget to hit the _Save_ button when you’ve configured these.

#### Running the pipeline

Now it’s showtime! ![🎦](https://s.w.org/images/core/emoji/14.0.0/svg/1f3a6.svg) On the Jenkins home page run the _seed-job_, which will bootstrap all the other jobs via the _createJobs.groovy_ script:

![](https://tomgregory.com/wp-content/uploads/2020/04/new-job-1024x333.png)

Importantly, we now have a _theme-park-job-aws_ job. This references the [_Jenkinsfile-aws_](https://github.com/tkgregory/spring-boot-api-example/blob/master/Jenkinsfile-aws) in the Spring Boot API project.

Shall we run it? OK, go on then:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-8-1024x494.png)

Your build should be all green. If it’s not, and you need a hand, feel free to comment below or email me at [tom@tomgregory.com](mailto:tom@tomgregory.com)

Our pipeline has built, including running the _Deploy to AWS_ stage! Awesome, but maybe we need to check it’s _really_ deployed?

## 6\. Accessing the deployed application

Over in the AWS Console, navigate to _Services_ > _CloudFormation_ > _Stacks_ and you’ll see our _spring-boot-api-example_ stack:

![CloudFormation stack](https://tomgregory.com/wp-content/uploads/2020/04/image-4-1024x386.png)

Go to _Services_ > _Elastic Container Service_, and you’ll see our ECS cluster _deployment-example-cluster_:

![](https://tomgregory.com/wp-content/uploads/2020/04/cluster-info-1024x699.png)

It’s reporting one service with one running task. Good news, as that means we’ve got ourselves a Docker container running. ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg)

Click on the cluster name, then the _Tasks_ tab. There should be a single task listed, so click on the task id:

![](https://tomgregory.com/wp-content/uploads/2020/04/task-id.png)

On the page that follows there’s lots of information about our task, but importantly for us under the network section we have a public IP:

![](https://tomgregory.com/wp-content/uploads/2020/04/task-network-info.png)

Sadly, by the time you read this article this IP will no longer exist. Fear not though, as you can create your own!

You’ve probably guessed what comes next then? If you remember [part 1](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) of this series, we were using Postman to make requests to our service. Let’s fire up Postman again and make a `GET` request to our service running in AWS, substituting in the public IP in the format `http://<public-ip>:8080/ride`:

![](https://tomgregory.com/wp-content/uploads/2020/04/image-9.png)

Of course if you prefer, you can just use a browser.

Success! That was quite a ride, but we got there in the end.

## 7\. Final thoughts

You’ve now seen how to add an AWS deployment to a Jenkins pipeline, using a CloudFormation template that lives in the application repository. This is a very powerful concept, and can be rolled out to infrastructures that contain many microservices.

For the purposes of keeping this article concise, we’ve deployed a simplified setup. For production purposes, you’ll want to also consider:

-   **high availability** – deploy to multiple replicas in multiple availability zones. We only have one instance of our service running right now.
-   **a load balancer** – to jump between the above replicas
-   **a DNS record** – to access the service via a friendly domain, instead of an IP address
-   **move Jenkins to a hosted service** – right now it’s just running on your machine. This could be deployed into AWS too, much like the microservice. See _[Deploy your own production-ready Jenkins in AWS ECS](https://tomgregory.com/deploy-jenkins-into-aws-ecs/)_ for exactly how to do this.
-   **centralised logging** – consider pushing container logs out to CloudWatch to be able to check for errors

Don’t forget to delete your AWS resources, which you can do by going to _Services_ > _CloudFormation_, then select the stack and click _Delete_.

## 8\. Resources

**GITHUB**

-   Grab the [sample Spring Boot application](https://github.com/tkgregory/spring-boot-api-example.git). It now contains the _Jenkinsfile-aws_ file which includes the _Deploy to AWS_ pipeline stage.
-   Clone the [jenkins-demo](https://github.com/tkgregory/jenkins-demo/tree/theme-park-job-aws) project, for bringing up an instance of Jenkins. The _theme-park-job-aws_ branch contains the updated version of _createJobs.groovy_ as well as the new plugins required.

**RELATED ARTICLES**  
Learn more about how ECS works with [these AWS docs](https://aws.amazon.com/ecs/)

**VIDEO**  
If you prefer to learn in video format then check out the accompanying video below. It’s part of the [Tom Gregory Tech](https://www.youtube.com/channel/UCxOVEOhPNXcJuyfVLhm_BfQ) YouTube channel.

<iframe title="Deploying a Spring Boot application into AWS with Jenkins (part 3 of microservice devops series)" width="752" height="423" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" data-src="https://www.youtube.com/embed/5xh0nAYeZNc?feature=oembed" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw=="></iframe>