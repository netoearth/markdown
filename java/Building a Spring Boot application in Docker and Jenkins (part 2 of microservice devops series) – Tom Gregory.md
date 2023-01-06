Last Updated on August 4, 2022

Welcome to the second of this three part series where you’ll learn how to take a Spring Boot microservice from inception to deployment, using all the latest continuous integration best practices.

**In this article we’ll update the Spring Boot service we built in [Part 1](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) and get it running in a Docker container. Then we’ll setup our Jenkins instance to work with Docker, and create a pipeline to build the image and push it to Docker Hub.**

This series of articles includes:

-   [Part 1: writing a Spring Boot application and setting up a Jenkins pipeline to build it](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/)
-   **Part 2: wrapping the application in a Docker image, building it in Jenkins, then pushing it to Docker Hub (this article)**
-   [Part 3: deploying the Docker image as a container from Jenkins into AWS](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/)

Did you read Part 1 already? If not, [check it out](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) as we’ll be building on top of the example from that article.

UPDATED in June 2021 to use the latest versions of Gradle and Jenkins.

In the previous article we created this [Spring Boot API application](https://github.com/tkgregory/spring-boot-api-example.git) to model a theme park, including getting and creating rides. ![🎢](https://s.w.org/images/core/emoji/14.0.0/svg/1f3a2.svg)

![](https://tomgregory.com/wp-content/uploads/2020/03/image-54.png)

Let’s take things forward and get the application running in Docker. Some reasons we might want to do this include:

-   more easily package and deploy the code (e.g. with a Docker image Java comes pre-installed)
-   use modern container orchestration services like _Kubernetes_ or _AWS ECS_, which help manage applications at scale
-   make development simpler, by allowing us to more easily start up multiple associated services or _microservices_

#### Dockerfile

A Docker image is created from a _Dockerfile_. It’s just a series of instructions describing how to create an image. At a high level, let’s think about what we’ll need in the image to run our Spring Boot application:

1.  have Java installed (we’re using Java 11 right now)
2.  copy the Spring Boot jar file into the image
3.  execute the Spring Boot jar file with the _java_ command

Let’s go ahead then and create a file _Dockerfile_ (no extension required) in the root of the Spring Boot project:

```
FROM openjdk:11
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

-   the `FROM` instruction tells Docker that we’re going to extend an existing base image, in this case an image for OpenJDK 11
-   the `ARG` instruction defines a variable that can be passed in at build time, in this case to provide the name of the Spring Boot jar file
-   the `COPY` instruction allows us to copy a file into the image, in this case the Spring Boot jar file
-   the `ENTRYPOINT` instruction describes what should be run when the container starts up, in this case we want to execute the jar file with the _java_ command

#### Building a Docker image in Gradle

Let’s extend the [_build.gradle_](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/#buildfile) from part 1 so that we can build and run our Docker image by executing a Gradle task. A useful plugin I often use to do this is the [Palantir Gradle Docker plugin](https://github.com/palantir/gradle-docker), which exposes useful tasks like `docker` to build the image and `dockerRun` to run it.

It consists of several plugins, which you can pick and choose depending on what functionality you need. Let’s apply the following two Gradle plugins in our _build.gradle_, so we can build images and run containers.

```
plugins {
    ...
    id 'com.palantir.docker' version '0.26.0'
    id 'com.palantir.docker-run' version '0.26.0'
}
```

##### Building the image

As you probably know, some Gradle plugins require additional configuration to set them up. We’ll configure the `com.palantir.docker` plugin with this configuration block:

```
String imageName = "tkgregory/spring-boot-api-example:$version"

docker {
    name imageName
    files "build/libs/${bootJar.archiveFileName.get()}"
    buildArgs([JAR_FILE: bootJar.archiveFileName.get()])
}
```

-   we define an `imageName` variable as we’ll need to reference it in multiple places (the `tkgregory` prefix you can change [later on](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/#docker-login) when we cover pushing the image to your own Docker Hub account)
-   we configure the Docker plugin `name` property to create an image with the specified name
-   the `files` property exposes the specified file to be available for `COPY` instructions in the Dockerfile. We’re referencing the `archiveFileName` from the Spring Boot Gradle plugin to get the correct jar file name.
-   we’re passing the `JAR_FILE` build argument to the Dockerfile, so it can execute the `COPY` instruction

Run `./gradlew assemble docker` and the image will be built:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-42.png)

**Info:** the first time you execute the `docker` Gradle task it may take some time as Docker needs to download the large `openjdk` base image.

We can verify that our image has been built by running `docker images`:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-43-1024x63.png)

##### Running the image

Next up, let’s configure the `com.palantir.docker-run` plugin:

```
dockerRun {
    name project.name
    image imageName
    ports '8080:8080'
    clean true
}
```

-   the `name` of our container can be the same as our project e.g. _spring-boot-api-example_
-   the `image` refers to the image which we will have built from our previous configuration. We’re reusing the `imageName` variable.
-   the `ports` we configure are 8080 on the host and 8080 on the container, which by default is where our Spring Boot application is listening
-   `clean true` means that when we stop our container it will be automatically deleted, saving us a bit of time

Now let’s run `./gradlew assemble docker dockerRun` to run a container from our image:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-44.png)

We can run `docker ps` to validate that the container is running:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-45-1024x118.png)

Awesome! Let’s hit the application’s [http://localhost:8080/actuator/health](http://localhost:8080/actuator/health) endpoint just to make sure:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-46.png)

If you want to stop the container, you can just run `./gradlew dockerStop`.

**GitHub repository**  
If you want to jump straight to the good stuff, the code above is available for you in the [spring-boot-api-example repository](https://github.com/tkgregory/spring-boot-api-example.git).

## 2\. Pushing to Docker hub

In a real-world scenario we’re going to need to _push_ our Docker image to a remote repository in order to then _pull_ it onto a server or cloud service to run it. Our setup, from the point of view of our _continuous integration_ service (i.e. Jenkins) might look like this:

![](https://www.lucidchart.com/publicSegments/view/e3f52c14-67db-4a13-80a8-c912508e2efa/image.png)

##### Docker registries

In Docker lingo, a _repository_ is where you can store one or more versions of a specific Docker image. A _registry_, on the other hand, is a remote service that stores a collection of repositories.

[Docker Hub](https://hub.docker.com/) is a Docker registry providing free storage of public and private images (you only get to store 1 private image with a free account).

If you’re using AWS, then you might want to consider [AWS ECR](https://aws.amazon.com/ecr/) (Elastic Container Registry) instead, as you’ll be able to keep everything within your private AWS infrastructure.

##### Docker login and Docker push

To use Docker Hub, go ahead and [setup an account](https://hub.docker.com/signup).

Before you can push images you’ll need to login on the command line with `docker login --username=<username>`. Enter your password when prompted.

![Docker login command](https://tomgregory.com/wp-content/uploads/2020/03/docker-login-300x70.png)

Edit the _build.gradle_ of the Spring Boot API application and replace _tkgregory_ in the `imageName` variable with your own Docker Hub username i.e. `String imageName = "<your-docker-hub-username>/spring-boot-api-example:$version"`

To push the image to Docker Hub just run `./gradlew docker dockerPush` and wait for it to upload:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-48-1024x591.png)

Now over in Docker Hub you’ll be able to see your image in your own account repository at `https://hub.docker.com/r/<your-docker-hub-username>/spring-boot-api-example`:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-50-1024x460.png)

**A word of warning**  
Note that by default when you push an image to Docker Hub it will be public for the world to see. See [this guide](https://docs.docker.com/docker-hub/repos/#private-repositories) to learn about setting up private repositories.

OK, so that’s our application all setup. Time to jump through a few hoops to get things working with Docker in Jenkins.

![](https://tomgregory.com/wp-content/uploads/2020/03/2aksyxynuco-1024x683.jpg)

## 3\. Configuring Jenkins to be able to run Docker in Docker

In [part 1](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) we made use of the [jenkins-demo](https://github.com/tkgregory/jenkins-demo.git) GitHub repository (the _theme-park-job_ branch) to run an instance of Jenkins locally using our own custom _Dockerfile_. We’re going to make some modifications to:

-   install Docker in the Jenkins Docker image
-   hook up the Docker command in the Jenkins image to our Docker installation on the host. This will allow us to run _Docker in Docker_, or in other words build Docker images from Jenkins.

##### Installing Docker in Jenkins (Docker in Docker)

Update the _Dockerfile_ for Jenkins to the one below, which now includes installing Docker from [https://get.docker.com](https://get.docker.com/):

```
FROM jenkins/jenkins:2.346.2-jdk11

USER root
RUN curl -sSL https://get.docker.com/ | sh
RUN usermod -a -G docker jenkins
USER jenkins

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

COPY seedJob.xml /usr/share/jenkins/ref/jobs/seed-job/config.xml

ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false
```

##### Making _/var/run/docker.sock_ available to Jenkins

In order that when we run Docker commands in Jenkins they communicate with the Docker process running on the host, we have to use a file _/var/run/docker.sock_. This file must be mounted as a volume inside the Jenkins container.

Update the `dockerRun` configuration in _build.gradle_ to as below:

```
Process process
if (DefaultNativePlatform.getCurrentOperatingSystem().isWindows()) {
    process = "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock alpine stat -c %g /var/run/docker.sock".execute()
} else if (DefaultNativePlatform.getCurrentOperatingSystem().isLinux()) {
    process = "stat -c %g /var/run/docker.sock".execute()
} else {
    throw new GradleException("Unsupported operating system. No way to find group of /var/run/docker.sock.")
}

def out = new ByteArrayOutputStream()
process.waitForProcessOutput(out, System.err)
String dockerSockGroupId = out.toString().trim()

String extraPrefix = DefaultNativePlatform.getCurrentOperatingSystem().isWindows() ? '/' : ''
dockerRun {
    name "${project.name}"
    image "${project.name}:${project.version}"
    ports '8080:8080'
    clean true
    daemonize false
    arguments '-v', "$extraPrefix/var/run/docker.sock:/var/run/docker.sock", '--group-add', dockerSockGroupId
}
```

You’ll also have to include this import at the top of the file:

```
import org.gradle.nativeplatform.platform.internal.DefaultNativePlatform
```

-   in the first 9 lines we’re determining if we’re on a Linux or Windows machine. Depending on the outcome, we run a specific command to find out which group owns _/var/run/docker.sock_ and then store that value as `dockerSockGroupId`.
-   `dockerRun` has been updated to pass 2 new arguments
    -   the `-v` argument mounts `/var/run/docker.sock` inside the container so that Jenkins can use the Docker process of the host. Note that in Windows an additional slash prefix is required.
    -   the `--group-add` argument makes sure the container is also run with the group that owns _/var/run/docker.sock_, so that Jenkins has the required access

**Docker host in Windows**  
If you’re using Docker for Windows, like me, the host from the container’s point of view is not Windows but a Linux VM running inside Windows. For a full explanation of Docker in Docker see my article _[Running Docker in Docker on Windows (Linux containers)](https://tomgregory.com/running-docker-in-docker-on-windows/)_.

At this point you can run Jenkins using `./gradlew docker dockerRun`, but there are still a few tweaks to add in the following section. Don’t forget to first stop the Spring Boot API application as it’s also running on port 8080.

To double check that you can run Docker in Docker, run `docker exec jenkins-demo docker ps`. If you see output like below, you’re good to continue.

![](https://tomgregory.com/wp-content/uploads/2020/03/image-55-1024x90.png)

## 4\. Building the Docker Spring Boot API application in Jenkins

In [part 1](https://tomgregory.com/building-a-spring-boot-application-in-jenkins/) we had the application building in this Jenkins pipeline, consisting of _checkout_, _build_, and _test_ stages:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-51-1024x566.png)

Let’s extend this pipeline to include stages to build the Docker image and push it to Docker Hub.

#### Updating the Jenkinsfile

Remember that the pipeline above is stored in a file called _Jenkinsfile_ in our Spring Boot API application [repository](https://github.com/tkgregory/spring-boot-api-example.git)? Add the additional _Build Docker image_ and _Push Docker image_ stages shown below:

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
                sh './gradlew dockerPush'
            }
        }
    }
}
```

-   for the _**Build Docker image**_ stage we’re using the Gradle Docker plugin to build the image
-   for the _**Push Docker image**_ stage we’re grabbing the _docker-hub_ credentials from Jenkins and storing it as an environment variable. We’ll cover how to add the credentials shortly. We then use this variable to run a `docker login` command, and once we’re logged in we can push the image to Docker Hub using the `dockerPush` Gradle command.

**Credentials in pipelines**  
When you store a _username with password_ type credential in an environment variable in a Jenkins pipeline, the username and password are automatically separated out into two additional variables. Read more about this here in _[How To Secure Your Gradle Credentials In Jenkins](https://tomgregory.com/how-to-secure-your-gradle-credentials-in-jenkins/)_.

If you’re following along with this example by building up your own project, make the change above and push it. Alternatively, the change is also available in _[Jenkinsfile-docker](https://github.com/tkgregory/spring-boot-api-example/blob/master/Jenkinsfile-docker)_, a custom Jenkinsfile I’ve pushed **just for you** (I’ll show you how to use it in the next section).

#### Adding the new Docker job definition

Lastly, in _createJobs.groovy_ of the _jenkins-demo_ project add the following job definition:

```
pipelineJob('theme-park-job-docker') {
    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url 'https://github.com/tkgregory/spring-boot-api-example.git'
                    }
                    branch 'master'
                    scriptPath('Jenkinsfile-docker')
                }
            }
        }
    }
}
```

This will create a job based on the [Jenkinsfile-docker](https://github.com/tkgregory/spring-boot-api-example/blob/master/Jenkinsfile-docker) file discussed earlier. If you want this to refer to your own repository, change the `url` and `scriptPath` appropriately. Also, don’t forget to change the location in `seedJob.xml` to point to your own appropriate git URL and branch to fetch the _createJobs.groovy_ file.

Now all we have to do is run `./gradlew docker dockerRun` to start up Jenkins:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-52-1024x600.png)

#### Adding credentials to Jenkins

Let’s add our Docker Hub credentials into Jenkins, by navigating through the somewhat long-winded following series of pages…

In the left hand navigation menu select _Manage Jenkin_s then _Manage Credentials_:

![Manage Jenkins](https://tomgregory.com/wp-content/uploads/2020/09/manage-jenkins-1024x350.png)

Select the _Jenkins_ credential store:

![](https://tomgregory.com/wp-content/uploads/2020/03/credentials-store-1024x395.png)

Select _Global credentials_:

![](https://tomgregory.com/wp-content/uploads/2020/03/credentials-global-1024x348.png)

Click _Add Credentials_:

![](https://tomgregory.com/wp-content/uploads/2020/03/credentials-add-1024x252.png)

Finally we’re here! Now add a _Username with password_ credential, entering your Docker Hub _username_, _password_, and an _id_ of _docker-hub_. Click _OK_ to save your credentials in Jenkins.

![](https://tomgregory.com/wp-content/uploads/2020/03/credentials-form-1024x413.png)

#### Running the new Docker pipeline

Now for the final act! ![🎭](https://s.w.org/images/core/emoji/14.0.0/svg/1f3ad.svg) Build the seed-job, then build the newly created _theme-park-job-docker_:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-53-1024x491.png)

Awesome! We now have a Jenkins job that builds, tests, and pushes an image for our Spring Boot API application.

## 5\. Final thoughts

Having a pipeline that will automatically take code as soon as it’s pushed and put it in a Docker image in Docker Hub is a very good place to be.

In the [final article](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/) in this series, we’ll be putting the cherry on top by taking our Docker image and deploying it into AWS.

## 6\. Resources

**GITHUB**

-   [Check out](https://github.com/tkgregory/spring-boot-api-example.git) the sample Spring Boot application for the theme park API. It now contains the _Jenkinsfile-docker_ file which includes Docker build and push stages in the pipeline.
-   [Here you can find](https://github.com/tkgregory/jenkins-demo/tree/theme-park-job-docker) the code for the _jenkins-demo_ project, for bringing up an instance of Jenkins. The _theme-park-job-docker_ branch contains the updated version of _createJobs.groovy_ as well as the modifications needed to run Docker in Docker.

**RELATED ARTICLES**  
Discover more details about how to secure credentials in Jenkins in [this article](https://tomgregory.com/how-to-secure-your-gradle-credentials-in-jenkins/).

**VIDEO**  
If you prefer to learn in video format then check out the accompanying video below. It’s part of the [Tom Gregory Tech](https://www.youtube.com/channel/UCxOVEOhPNXcJuyfVLhm_BfQ) YouTube channel.

<iframe title="Building a Spring Boot application in Docker and Jenkins (part 2 of microservice devops series)" width="752" height="423" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" data-src="https://www.youtube.com/embed/Kc3Vw5vk1Lw?feature=oembed" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw=="></iframe>