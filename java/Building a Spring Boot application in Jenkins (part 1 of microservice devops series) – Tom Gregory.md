Last Updated on May 4, 2022

Welcome to the first of this three part series where we’ll take a Spring Boot microservice from inception to deployment.

**In this article, we’ll create a simple Spring Boot API application with integration tests, and then build it in a Jenkins pipeline every time a change is pushed to version control.**

The full series of articles includes:

-   **Part 1: writing a Spring Boot application and setting up a Jenkins pipeline to build it (this article)**
-   [Part 2: wrapping the application in a Docker image, building it in Jenkins, then pushing it to Docker Hub](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/)
-   [Part 3: deploying the Docker image as a container from Jenkins into AWS](https://tomgregory.com/deploying-a-spring-boot-application-into-aws-with-jenkins/)

After this series you’ll have a solid foundation in the best practices to bring continuous integration to microservice architectures.

Ready? Let’s get right into it!

UPDATED in June 2021 to use latest Spring, Gradle, Jenkins and other plugin/dependency versions.

Spring Boot is a framework for easily writing Java web applications. In our case, we’ll use it to create an API service. This is a common requirement of microservice architectures, especially those that have a UI that communicates with a backend API.

Just for fun, we’ll base the application around theme park rides. ![🎢](https://s.w.org/images/core/emoji/14.0.0/svg/1f3a2.svg) It will be geared towards doing basic get & create operations on theme park rides, comprising a:

-   `GET` API at`/ride` to get all the rides the application knows about
-   `GET` API at `/ride/{id}` to get details of a specific ride by id
-   `POST` API at `/ride` to add a new ride

**Code repo:** following along with the code snippets and creating the project from scratch is the best way to get more familiar with Spring Boot. If you want a fast-pass ticket though, here’s [all the code in GitHub](https://github.com/tkgregory/spring-boot-api-example.git).

#### Application desgin

We’ll use a fairly typical Spring Boot _Model-View-Controller_ format, which normally looks like this:

![](https://www.lucidchart.com/publicSegments/view/faa413ae-6cfa-4ed2-ac23-1b32c5204f41/image.png)

In our case though, we won’t implement a service as there is no business logic to speak of.

#### Build file

We’ll be using the popular _Gradle_ tool to build this application. Make sure you’ve got a [recent version](https://gradle.org/releases/) installed on your machine before proceeding.

Create a new directory for the project, and navigate into it. Run `gradle init` to start the Gradle setup wizard, choosing to create a basic Groovy project with the default name. This creates a skeleton of a Gradle project, including the Gradle wrapper used for interacting with the application.

**Gradle help:**  
1) If you’re using IntelliJ IDEA check out [this article](https://tomgregory.com/5-tips-for-using-gradle-with-intellij-idea-2019/) to learn how to create Gradle projects from your IDE  
2) For a more general overview check out my introduction to Gradle [YouTube video](https://youtu.be/ojx49J1JCdQ)  
3) Here’s [an article](https://tomgregory.com/what-is-the-gradle-wrapper-and-why-should-you-use-it/) all about the infamous Gradle wrapper

A _build.gradle_ file is automatically generated. Modify it so it looks like the one below:

```
plugins {
    id 'java'
    id 'io.spring.dependency-management' version "1.0.11.RELEASE"
    id 'org.springframework.boot' version '2.5.0'
    id 'pl.allegro.tech.build.axion-release' version '1.13.2'
}

version = scmVersion.version
sourceCompatibility = 8

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-devtools'
    implementation group: 'com.h2database', name: 'h2', version: '1.4.200'

    compileOnly 'org.projectlombok:lombok:1.18.20'
    annotationProcessor 'org.projectlombok:lombok:1.18.20'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}
```

-   the Spring `dependency-management` and `boot` plugins bring Spring Boot functionality to our build
-   the `axion-release` plugin adds versioning using tags
-   we’re got several Spring Boot dependencies to allow us to use the Spring Boot web framework with databases
-   the `h2` dependency will enable us to use an in memory database
-   Lombok has been configured as an annotation processor to reduce boilerplate code through code generation

Now let’s create some Java classes. This, and all the following classes, should be created in _src/main/java_ in the package that appears at the top of each individual class definition.

#### Entity class

The entity class represents each instance of a theme park ride. Let’s create a `ThemeParkRide` clas_s_:

```
package com.tomgregory.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.validation.constraints.NotEmpty;

@Entity
@Getter
@ToString
@NoArgsConstructor
public class ThemeParkRide {
  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  @NotEmpty
  private String name;
  @NotEmpty
  private String description;
  private int thrillFactor;
  private int vomitFactor;

  public ThemeParkRide(String name, String description, int thrillFactor, int vomitFactor) {
    this.name = name;
    this.description = description;
    this.thrillFactor = thrillFactor;
    this.vomitFactor = vomitFactor;
  }

}
```

-   this is a simple POJO (plain old Java object) class with `id`, `name`, `description`, `thrillFactor`, and `vomitFactor` fields ![🤮](https://s.w.org/images/core/emoji/14.0.0/svg/1f92e.svg)
-   the class is annotated with `@Entity` so Spring Boot knows what kind of class it is
-   we’re using the Lombok annotation processor to generate a default constructor, a `toString()` method, and getter methods for each field
-   `@NotEmpty` annotations are applied to the _name_ and _description_ fields for validation when we create a theme park ride

![](https://tomgregory.com/wp-content/uploads/2020/03/Theme_Park-580x326-1.png)

Does anyone remember this game from the 1990s?

#### Repository class

We’ll use a repository interface to read and write `ThemeParkRide` entities from/to the database. The beauty of the _spring-boot-starter-data-jpa_ library that we imported in _build.gradle_ is that you only have to define the method signatures of the interactions you want with the database, and the library does the rest.

Create an interface called `ThemeParkRideRepository`:

```
package com.tomgregory.repository;

import com.tomgregory.entity.ThemeParkRide;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface ThemeParkRideRepository extends CrudRepository<ThemeParkRide, Long> {
    List<ThemeParkRide> findByName(String name);
}

```

-   the `@Repository` annotation lets Spring Boot know what kind of component this class is
-   extending `CrudRepository` provides some default methods such as `save` and `findById`
-   the `findByName` method definition will be implemented automatically by Spring Boot, matching to the `name` field of the `ThemeParkRide` entity class

#### Controller class

A controller class defines an API and how it interacts with the rest of the application. Create a class called `ThemeParkRideController`:

```
package com.tomgregory.controller;

import com.tomgregory.repository.ThemeParkRideRepository;
import com.tomgregory.entity.ThemeParkRide;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.server.ResponseStatusException;

import javax.validation.Valid;

@RestController
public class ThemeParkRideController {
    private final ThemeParkRideRepository themeParkRideRepository;

    public ThemeParkRideController(ThemeParkRideRepository themeParkRideRepository) {
        this.themeParkRideRepository = themeParkRideRepository;
    }

    @GetMapping(value = "/ride", produces = MediaType.APPLICATION_JSON_VALUE)
    public Iterable<ThemeParkRide> getRides() {
        return themeParkRideRepository.findAll();
    }

    @GetMapping(value = "/ride/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ThemeParkRide getRide(@PathVariable long id){
        return themeParkRideRepository.findById(id).orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, String.format("Invalid ride id %s", id)));
    }

    @PostMapping(value = "/ride", consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
    public ThemeParkRide createRide(@Valid @RequestBody ThemeParkRide themeParkRide) {
        return themeParkRideRepository.save(themeParkRide);
    }
}

```

-   the `@RestController` annotation tells Spring Boot what type of component this is
-   for the `getRides` method we use the `@GetMapping` annotation to tell Spring Boot we’re expecting a `GET` request at the specified path
-   the `getRide` method has a `@PathVariable` annotation so we can pass the ride id in the path `/ride/{id}`
-   the `createRide` method uses the `@RequestBody` annotation to automatically map the JSON request to the `ThemeParkRide` entity.
-   the `@Valid` annotation is included on the incoming `ThemeParkRide` entity to validate it according to any annotations on it’s fields (e.g. `@NotEmpty`)

#### Application class

The final class we have to add to have a fully working (but untested) application will be `ThemeParkApplication`:

```
package com.tomgregory;

import com.tomgregory.entity.ThemeParkRide;
import com.tomgregory.repository.ThemeParkRideRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class ThemeParkApplication  {
    public static void main(String[] args) {
        SpringApplication.run(ThemeParkApplication.class);
    }

    @Bean
    public CommandLineRunner sampleData(ThemeParkRideRepository repository) {
        return (args) -> {
            repository.save(new ThemeParkRide("Rollercoaster", "Train ride that speeds you along.", 5, 3));
            repository.save(new ThemeParkRide("Log flume", "Boat ride with plenty of splashes.", 3, 2));
            repository.save(new ThemeParkRide("Teacups", "Spinning ride in a giant tea-cup.", 2, 4));
        };
    }
}

```

-   the `@SpringBootApplication` annotation tells Spring Boot that this class is defining our main application
-   the `main` method means that when we run this class the application will start
-   the `sampleData` bean definition adds three default rides for us

## 2\. Trying out our Spring Boot application

Run the `ThemeParkApplication` class or `./gradlew bootRun` to start the application:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-29-1024x345.png)

Let’s make some requests to the application. I recommend the [Postman tool](https://www.postman.com/downloads/) for easily making requests via a nice UI.

#### Get all rides

Let’s call `GET` `/ride` to get all rides

![](https://tomgregory.com/wp-content/uploads/2020/03/image-30.png)

We can see the three rides are returned as per the `sampleData` bean definition in `ThemeParkApplication`.

#### Create ride

Let’s `POST` some JSON data to `/ride` to create a new ride:

```
{
"name":"Monorail",
"description":"Sedate ride that takes you around the park.",
"thrillFactor":2,
"vomitFactor":1
}
```

We’ll need to remember to also add a `Content-Type` header of `application/json`.

![](https://tomgregory.com/wp-content/uploads/2020/03/image-31.png)

That’s a `200` `OK` response. So far, so good.

#### Get ride

Let’s `GET` the ride we just added calling `/ride/4`:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-32.png)

The _Monorail_ ride is returned as expected!

#### Adding a test

It’s great that everything’s working, but let’s add an automated test to validate this functionality so our fingers don’t get too tired. ![🤞](https://s.w.org/images/core/emoji/14.0.0/svg/1f91e.svg)

What follows is an integration test class `ThemeParkApplicationIT` that should be added to `src/test/java`. We’ll use it to test interactions with the three APIs all the way through our application to the database.

```
package com.tomgregory;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

@ExtendWith( SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
public class ThemeParkApplicationIT {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void getsAllRides() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/ride")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andReturn();
    }

    @Test
    public void getsSingleRide() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/ride/1")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andReturn();
    }

    @Test
    public void returnsNotFoundForInvalidSingleRide() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/ride/4")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound())
                .andReturn();
    }

    @Test
    public void addsNewRide() throws Exception {
        String newRide = "{\"name\":\"Monorail\",\"description\":\"Sedate travelling ride.\",\"thrillFactor\":2,\"vomitFactor\":1}";
        mockMvc.perform(MockMvcRequestBuilders.post("/ride")
                .contentType(MediaType.APPLICATION_JSON)
                .content(newRide)
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andReturn();
    }
}
```

-   the 3 class level annotations are required so that Spring Boot initialises itself, creates the beans, and configures the `MockMvc` class
-   `MockMvc` can be used to make requests into our application. For example, in the first test we’re making a `GET` request to `/ride` and expecting a `status().isOk()` response

Run the test by executing `./gradlew test`. Open up `build/reports/tests/test/index.html` to see your test report.

![](https://tomgregory.com/wp-content/uploads/2020/03/image-33-1024x366.png)

Everything’s A-OK!

![](https://tomgregory.com/wp-content/uploads/2020/03/gzc4fnqsawq-1024x768.jpg)

## 3\. Running Jenkins

The next stage is to automatically build our Spring Boot application on commit, or in other words to use _continuous integration_. This will ensure that our master branch is always in a good state, and anything that gets deployed into production is a working version of our application.

We’ll use the popular build tool **Jenkins** for this purpose, and will set it up to run as a Docker container on our local machine. For the following steps you’ll need Docker, which you can get from the [Docker website](https://docs.docker.com/get-docker/).

#### Bootstrapping Jenkins

A good devops philosophy is to define all infrastructure and configuration as code. So, we’ll _bootstrap_ Jenkins to start up and configure whatever jobs we need automatically, with minimal manual setup required.

This concept has been discussed in detail in my [_How to bootstrap Jenkins for disaster recovery and accountability_](https://youtu.be/s7dw0ahriQY) YouTube video. We’ll build on top of the code example from the video, which can be found in this [GitHub repository](https://github.com/tkgregory/jenkins-demo.git) and cloned like this:

```
git clone https://github.com/tkgregory/jenkins-demo.git
```

Make sure the Spring Boot application is stopped as Jenkins uses the same port. Then build and run Jenkins:

`./gradlew build docker dockerRun`

Navigate to [http://localhost:8080](http://localhost:8080/) and you’ll have an instance of Jenkins running:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-35-1024x344.png)

You can see we already have one job called _seed-job_ automatically available. This job is responsible for creating any other jobs that we need. Let’s run it by clicking on the name and then clicking _Build Now_.

Once it finishes, go back to the Jenkins home page and you’ll see that _seed-job_ has generated an additional _pipelineJob_ for us.

![](https://tomgregory.com/wp-content/uploads/2020/03/image-36-1024x319.png)

Build that job and you’ll see we have a successful pipeline execution, with a _Build_ and _Test_ stage:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-38-1024x599.png)

**Info:** a _pipeline_ in Jenkins is just a specific kind of job that can be split into different stages, as shown in the UI above

If you click on the blue circle to the left of _#1_ in _Build History_, you can see the console output:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-39-1024x570.png)

These two stages just print out some text for now. The point is that we can very quickly have a working instance of Jenkins available. In the next section, we’ll add a new pipeline for our theme park application.

## 4\. Adding a Jenkins pipeline job to build the Spring Boot application

Let’s modify the [jenkins-demo](https://github.com/tkgregory/jenkins-demo.git) project to include a new pipeline to build our Spring Boot application. The diagram below summarises the flow from when we initially run _seed-job_ to when our pipeline job will run against our theme park application.

![](https://www.lucidchart.com/publicSegments/view/b2666652-b54d-4e23-8e47-784278cfbd8f/image.png)

Once again, make the code changes yourself for the best learning experience. To do this you’ll need to create and push two of your own repositories:

1.  **a repository for the Spring Boot theme park application** (everything created in sections 1 and 2 of this article). Once created, you’ll need to substitute your own Git repository URL in the _createJobs.groovy_ code snippet below.
2.  **a repository for the Jenkins code**. This should be based on the [jenkins-demo](https://github.com/tkgregory/jenkins-demo.git) project (you can fork the project if you like). Once created, edit the _seedJob.xml_, replacing the existing Git repository on line 10 with your own e.g. `<url>https://github.com/<your-username>/jenkins-demo.git</url>`

If you don’t want to create any repositories and instead prefer to use the ones I’ve made available for you, I’ll also explain how to do that below.

#### Creating a new job

Open up the _createJobs.groovy_ file, which is responsible for creating any jobs when _seed-job_ is run. We’ll add the following section which defines an additional pipeline job:

```
pipelineJob('theme-park-job') {
    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url 'https://github.com/tkgregory/spring-boot-api-example.git'
                    }
                    branch 'master'
                }
            }
        }
    }
}
```

This definition uses the Jenkins Job DSL Plugin syntax to create a pipeline job from a _Jenkinsfile_ in the _[spring-boot-api-example](https://github.com/tkgregory/spring-boot-api-example.git)_ repository (or use your own if you prefer). A _Jenkinsfile_ is a pipeline definition that lives inside the repository of the project that it applies to.

**Info:** the Jenkins Job DSL plugin provides a mechanism to bootstrap Jenkins jobs. Without it, you’d have to create the jobs from the UI.

Push the above change to your own repository, as Jenkins has to fetch the _createJobs.groovy_ file remotely.

If you don’t want to set that up, you can modify _seedJob.xml_ and change the branch name on line 15 from _master_ to _theme-park-job_.

```
<branches>
  <hudson.plugins.git.BranchSpec>
<name>*/theme-park-job</name>
  </hudson.plugins.git.BranchSpec>
</branches>
```

This way it will fetch the updated _createJobs.groovy_ file from a branch I’ve made available just for you.

#### Adding a Jenkinsfile to _spring-boot-api-example_

We’ll create a _Jenkinsfile_ in the top level of the project with the following contents:

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
    }
}
```

This is a basic pipeline that checks for any changes to our code every minute, then runs two stages:

-   **Build** – where we assemble the project
-   **Test** – where we test the project

Push these changes to your own repository, or just rely on the [repository I’ve already added](https://github.com/tkgregory/spring-boot-api-example.git) with the required Jenkinsfile.

#### Running the new job

Stop the previous Jenkins instance by running `./gradlew dockerStop` then bring up a fresh one with:

`./gradlew build docker dockerRun`

First run the _seed-job_ again to create the new _theme-park-job_:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-40-1024x333.png)

Then click on the new job and hit _Build Now_:

![](https://tomgregory.com/wp-content/uploads/2020/03/image-41-1024x526.png)

After some time you should have a successful build. Jenkins will highlight in green the two stages we defined, as well as the initial checkout.

## 5\. Final thoughts

Getting Jenkins running and building a Spring Boot project can be quite straightforward. Obviously in a proper setup you won’t run Jenkins on your local machine, as you’ll need high availability and most likely access by other team members. If you’re using AWS consider running Jenkins in an ECS or EKS cluster (see _[Deploy your own production-ready Jenkins in AWS ECS](https://tomgregory.com/deploy-jenkins-into-aws-ecs/)_).

In [part 2](https://tomgregory.com/building-a-spring-boot-application-in-docker-and-jenkins/) of this series we’ll move our Spring Boot application into Docker, build the Docker image in Jenkins, and then push that to a remote repository.

## 6\. Resources

GITHUB  
[Check out](https://github.com/tkgregory/spring-boot-api-example.git) the sample Spring Boot application for the theme park API  
[Here you can find](https://github.com/tkgregory/jenkins-demo/tree/theme-park-job) the code for the _jenkins-demo_ project, for bringing up an instance of Jenkins. The _theme-park-job_ branch contains the updated version of _createJobs.groovy_.

GRADLE  
[Gradle Tutorial](https://youtu.be/ojx49J1JCdQ) – why you should use it and how to get started

VIDEO  
If you prefer to learn in video format, check out the accompanying video to this post on the [Tom Gregory Tech](https://www.youtube.com/channel/UCxOVEOhPNXcJuyfVLhm_BfQ) YouTube channel.

<iframe title="Building a Spring Boot application in Jenkins (part 1 of microservice devops series)" width="752" height="423" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" data-src="https://www.youtube.com/embed/sCcuUMn1vdM?feature=oembed" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw=="></iframe>