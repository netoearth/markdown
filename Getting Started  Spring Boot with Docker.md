This guide walks you through the process of building a [Docker](https://docker.com/) image for running a Spring Boot application. We start with a basic `Dockerfile` and make a few tweaks. Then we show a couple of options that use build plugins (for Maven and Gradle) instead of `docker`. This is a “getting started” guide, so the scope is limited to a few basic needs. If you are building container images for production use, there are many things to consider, and it is not possible to cover them all in a short guide.

<table><tbody><tr><td><i title="Note"></i></td><td>There is also a <a href="https://spring.io/guides/topicals/spring-boot-docker">Topical Guide on Docker</a>, which covers a wider range of choices that we have here and in much more detail.</td></tr></tbody></table>

## What You Will Build

[Docker](https://docker.com/) is a Linux container management toolkit with a “social” aspect, letting users publish container images and consume those published by others. A Docker image is a recipe for running a containerized process. In this guide, we build one for a simple Spring boot application.

## What You Will Need

-   About 15 minutes
    
-   A favorite text editor or IDE
    
-   [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
    
-   [Gradle 4+](http://www.gradle.org/downloads) or [Maven 3.2+](https://maven.apache.org/download.cgi)
    
-   You can also import the code straight into your IDE:
    
    -   [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
        
    -   [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
        
    

If you are NOT using a Linux machine, you need a virtualized server. If you install VirtualBox, other tools like the Mac’s `boot2docker` can seamlessly manage it for you. Visit [VirtualBox’s download site](https://www.virtualbox.org/wiki/Downloads) and pick the version for your machine. Download and install. Do not worry about actually running it.

You also need [Docker](https://docker.com/), which only runs on 64-bit machines. See [https://docs.docker.com/installation/#installation](https://docs.docker.com/installation/#installation) for details on setting Docker up for your machine. Before proceeding further, verify you can run `docker` commands from the shell. If you use `boot2docker`, you need to run that **first**.

## Starting with Spring Initializr

You can use this [pre-initialized project](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.5.5&packaging=jar&jvmVersion=11&groupId=com.example&artifactId=spring-boot-docker&name=spring-boot-docker&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.spring-boot-docker&dependencies=web) and click Generate to download a ZIP file. This project is configured to fit the examples in this tutorial.

To manually initialize the project:

1.  Navigate to [https://start.spring.io](https://start.spring.io/). This service pulls in all the dependencies you need for an application and does most of the setup for you.
    
2.  Choose either Gradle or Maven and the language you want to use. This guide assumes that you chose Java.
    
3.  Click **Dependencies** and select **Spring Web**.
    
4.  Click **Generate**.
    
5.  Download the resulting ZIP file, which is an archive of a web application that is configured with your choices.
    

<table><tbody><tr><td><i title="Note"></i></td><td>If your IDE has the Spring Initializr integration, you can complete this process from your IDE.</td></tr></tbody></table>

<table><tbody><tr><td><i title="Note"></i></td><td>You can also fork the project from Github and open it in your IDE or other editor.</td></tr></tbody></table>

## Set up a Spring Boot Application

Now you can create a simple application:

`src/main/java/hello/Application.java`

```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {

  @RequestMapping("/")
  public String home() {
    return "Hello Docker World";
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

The class is flagged as a `@SpringBootApplication` and as a `@RestController`, meaning that it is ready for use by Spring MVC to handle web requests. `@RequestMapping` maps `/` to the `home()` method, which sends a `Hello World` response. The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application.

Now we can run the application without the Docker container (that is, in the host OS):

If you use Gradle, run the following command:

```
./gradlew build && java -jar build/libs/gs-spring-boot-docker-0.1.0.jar
```

If you use Maven, run the following command:

```
./mvnw package && java -jar target/gs-spring-boot-docker-0.1.0.jar
```

Then go to [localhost:8080](http://localhost:8080/) to see your “Hello Docker World” message.

## Containerize It

Docker has a simple ["Dockerfile"](https://docs.docker.com/reference/builder/) file format that it uses to specify the “layers” of an image. Create the following Dockerfile in your Spring Boot project:

Example 1. Dockerfile

```
FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

If you use Gradle, you can run it with the following command:

```
docker build --build-arg JAR_FILE=build/libs/\*.jar -t springio/gs-spring-boot-docker .
```

If you use Maven, you can run it with the following command:

```
docker build -t springio/gs-spring-boot-docker .
```

This command builds an image and tags it as `springio/gs-spring-boot-docker`.

This Dockerfile is very simple, but it is all you need to run a Spring Boot app with no frills: just Java and a JAR file. The build creates a spring user and a spring group to run the application. It is then copied (by the `COPY` command) the project JAR file into the container as `app.jar`, which is run in the `ENTRYPOINT`. The array form of the Dockerfile `ENTRYPOINT` is used so that there is no shell wrapping the Java process. The [Topical Guide on Docker](https://spring.io/guides/topicals/spring-boot-docker) goes into this topic in more detail.

Running applications with user privileges helps to mitigate some risks (see, for example, [a thread on StackExchange](https://security.stackexchange.com/questions/106860/can-a-root-user-inside-a-docker-lxc-break-the-security-of-the-whole-system)). So, an important improvement to the `Dockerfile` is to run the application as a non-root user:

Example 2. Dockerfile

```
FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

You can see the username in the application startup logs when you build and run the application:

```
docker build -t springio/gs-spring-boot-docker .
docker run -p 8080:8080 springio/gs-spring-boot-docker
```

Note the `started by` in the first `INFO` log entry:

```
 :: Spring Boot ::        (v2.2.1.RELEASE)

2020-04-23 07:29:41.729  INFO 1 --- [           main] hello.Application                        : Starting Application on b94c86e91cf9 with PID 1 (/app started by spring in /)
...
```

Also, there is a clean separation between dependencies and application resources in a Spring Boot fat JAR file, and we can use that fact to improve performance. The key is to create layers in the container filesystem. The layers are cached both at build time and at runtime (in most runtimes), so we want the most frequently changing resources (usually the class and static resources in the application itself) to be layered _after_ the more slowly changing resources. Thus, we use a slightly different implementation of the Dockerfile:

Example 3. Dockerfile

```
FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
```

This Dockerfile has a `DEPENDENCY` parameter pointing to a directory where we have unpacked the fat JAR. To use the `DEPENDENCY` parameter with Gradle, run the following command:

```
mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*.jar)
```

To use the `DEPENDENCY` parameter with Maven, run the following command:

```
mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)
```

If we get that right, it already contains a `BOOT-INF/lib` directory with the dependency JARs in it, and a `BOOT-INF/classes` directory with the application classes in it. Notice that we use the application’s own main class: `hello.Application`. (This is faster than using the indirection provided by the fat JAR launcher.)

<table><tbody><tr><td><i title="Note"></i></td><td>Exploding the JAR file can result in the classpath order being <a href="https://github.com/spring-projects/spring-boot/issues/9128#issuecomment-510577157">different at runtime</a>. A well-behaved and well-written application should not care about this, but you may see behavior changes if the dependencies are not carefully managed.</td></tr></tbody></table>

<table><tbody><tr><td><i title="Note"></i></td><td>If you use <code>boot2docker</code>, you need to run it <strong>first</strong> before you do anything with the Docker command line or with the build tools (it runs a daemon process that handles the work for you in a virtual machine).</td></tr></tbody></table>

From a Gradle build, you need to add the explicit build arguments in the Docker command line:

```
docker build --build-arg DEPENDENCY=build/dependency -t springio/gs-spring-boot-docker .
```

To build the image in Maven, you can use a simpler Docker command line:

```
docker build -t springio/gs-spring-boot-docker .
```

<table><tbody><tr><td><i title="Tip"></i></td><td>Of course, if you use only Gradle, you could change the <code>Dockerfile</code> to make the default value of <code>DEPENDENCY</code> match the location of the unpacked archive.</td></tr></tbody></table>

Instead of building with the Docker command line, you might want to use a build plugin. Spring Boot supports building a container from Maven or Gradle by using its own build plugin. Google also has an open source tool called [Jib](https://github.com/GoogleContainerTools/jib) that has Maven and Gradle plugins. Probably the most interesting thing about this approach is that you do not need a `Dockerfile`. You can build the image by using the same standard container format as you get from `docker build`. Also, it can work in environments where docker is not installed (not uncommon in build servers).

<table><tbody><tr><td><i title="Note"></i></td><td>By default, the images generated by the default buildpacks do not run your application as root. Check the configuration guide for <a href="https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/#build-image">Gradle</a> or <a href="https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/maven-plugin/reference/html/#build-image">Maven</a> for how to change the default settings.</td></tr></tbody></table>

### Build a Docker Image with Gradle

You can build a tagged docker image with Gradle in one command:

```
./gradlew bootBuildImage --imageName=springio/gs-spring-boot-docker
```

### Build a Docker Image with Maven

To get started quickly, you can run the Spring Boot image generator without even changing your `pom.xml` (remember that the `Dockerfile`, if it is still, there is ignored):

```
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=springio/gs-spring-boot-docker
```

To push to a Docker registry, you need to have permission to push, which you do not have by default. Change the image prefix to your own Dockerhub ID and `docker login` to make sure you are authenticated before you run Docker.

### After the Push

A `docker push` in the example fails (unless you are part of the "springio" organization at Dockerhub). However, if you change the configuration to match your own docker ID, it should succeed. You then have a new tagged, deployed image.

You do NOT have to register with docker or publish anything to run a docker image that was built locally. If you built with Docker (from the command line or from Spring Boot), you still have a locally tagged image, and you can run it like this:

```
$ docker run -p 8080:8080 -t springio/gs-spring-boot-docker
Container memory limit unset. Configuring JVM for 1G container.
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -XX:MaxMetaspaceSize=86381K -XX:ReservedCodeCacheSize=240M -Xss1M -Xmx450194K (Head Room: 0%, Loaded Class Count: 12837, Thread Count: 250, Total Memory: 1073741824)
....
2015-03-31 13:25:48.035  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2015-03-31 13:25:48.037  INFO 1 --- [           main] hello.Application                        : Started Application in 5.613 seconds (JVM running for 7.293)
```

<table><tbody><tr><td><i title="Note"></i></td><td>The CF memory calculator is used at runtime to size the JVM to fit the container.</td></tr></tbody></table>

The application is then available on [http://localhost:8080](http://localhost:8080/) (visit that and it says, “Hello Docker World”).

<table><tbody><tr><td><i title="Note"></i></td><td><p>When using a Mac with boot2docker, you typically see things like this at startup:</p><div><pre><code><span>Docker</span><span> client to the </span><span>Docker</span><span> daemon</span><span>,</span><span> please </span><span>set</span><span>:</span><span>
    </span><span>export</span><span> DOCKER_CERT_PATH</span><span>=</span><span>/Users/</span><span>gturnquist</span><span>/.</span><span>boot2docker</span><span>/</span><span>certs</span><span>/</span><span>boot2docker</span><span>-</span><span>vm
    </span><span>export</span><span> DOCKER_TLS_VERIFY</span><span>=</span><span>1</span><span>
    </span><span>export</span><span> DOCKER_HOST</span><span>=</span><span>tcp</span><span>:</span><span>//192.168.59.103:2376</span></code></pre></div><p>To see the application, you must visit the IP address in DOCKER_HOST instead of localhost — in this case, <a href="https://192.168.59.103:8080/">https://192.168.59.103:8080</a>, the public facing IP of the VM.</p></td></tr></tbody></table>

When it is running, you can see in the list of containers, similar to the following example:

```
$ docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
81c723d22865        springio/gs-spring-boot-docker:latest   "java -Djava.secur..."   34 seconds ago      Up 33 seconds       0.0.0.0:8080->8080/tcp   goofy_brown
```

To shut it down again, you can run `docker stop` with the container ID from the previous listing (yours will be different):

```
docker stop goofy_brown
81c723d22865
```

If you like, you can also delete the container (it is persisted in your filesystem somewhere under `/var/lib/docker`) when you are finished with it:

### Using Spring Profiles

Running your freshly minted Docker image with Spring profiles is as easy as passing an environment variable to the Docker run command (for the `prod` profile):

```
docker run -e "SPRING_PROFILES_ACTIVE=prod" -p 8080:8080 -t springio/gs-spring-boot-docker
```

You can do the same for the `dev` profile:

```
docker run -e "SPRING_PROFILES_ACTIVE=dev" -p 8080:8080 -t springio/gs-spring-boot-docker
```

### Debugging the Application in a Docker Container

To debug the application, you can use [JPDA Transport](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/conninv.html#Invocation). We treat the container like a remote server. To enable this feature, pass Java agent settings in the `JAVA_OPTS` variable and map the agent’s port to localhost during a container run. With [Docker for Mac](https://www.docker.com/products/docker#/mac), there is a limitation because we can’t access the container by IP without [black magic usage](https://github.com/docker/for-mac/issues/171).

```
docker run -e "JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,address=5005,server=y,suspend=n" -p 8080:8080 -p 5005:5005 -t springio/gs-spring-boot-docker
```

## Summary

Congratulations! You have created a Docker container for a Spring Boot application! By default, Spring Boot applications run on port 8080 inside the container, and we mapped that to the same port on the host by using `-p` on the command line.