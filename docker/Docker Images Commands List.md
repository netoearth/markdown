### Docker Images Commands List

All the important docker images related commands for your quick reference. This is a cheat sheet for docker commands for managing images.

## Build an image from a Dockerfile:

This command is used to build an image from a specified docker file

```
docker build -t <image-name> <path to docker file>
```

For example:

```
docker build -t springboot-docker-image .
```

This command lists all the locally stored docker images

## Display detailed information on one or more images:

This command is used to get the detailed information of specified docker images.

```
docker inspect <docker-image>
```

For example:

```
docker inspect springboot-docker-image
```

## Show the history of an image:

This command is used to get the history of a particular docker image.

```
docker image histroy <docker-image>
```

For example:

```
docker image history springboot-docker-image
```

## Create a tag TARGET\_IMAGE that refers to SOURCE\_IMAGE:

```
docker image tag <docker-image> <docker-image:tag>
```

For example:

```
docker image tag springboot-docker-image springboot-docker-image:0.1
```

## Remove or Delete a Docker Image

This command is used to delete an image from local storage

```
docker rmi <image-id>/<image-name>
```

For example:

```
docker rmi springboot-docker-image
```

## Remove or Delete a Docker Image Forcefully:

This command is used to delete an image from local storage

```
docker rmi -f <image-id>/<image-name>
```

For example:

```
docker rmi -f springboot-docker-image
```

## Pull an image or a repository from a registry:

This command is used to pull the docker image from the Dockerhub.

```
docker image pull <docker-image>/<repository-name>
```

For example:

```
docker image pull rabbitmq
```

## Remove unused images:

This command is used to remove all unused images from local storage.

## Push an image or a repository to a registry:

To push a local image to the docker registry, you need to associate the local image with a repository on the docker registry.

[Docker](https://www.javaguides.net/search/label/Docker?&max-results=10)

## Free Spring Boot Tutorial | Full In-depth Course | Learn Spring Boot in 10 Hours

___

Watch this course on YouTube at [**Spring Boot Tutorial | Fee 10 Hours Full Course**](https://youtu.be/_thI-4AF7M8)

<iframe allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" frameborder="0" height="450" src="https://www.youtube.com/embed/_thI-4AF7M8" title="YouTube video player" width="850"></iframe>