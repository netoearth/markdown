## Introduction

This tutorial explains how to install Watchtower on Ubuntu 20.04.

Watchtower will automatically keep all of your running Docker containers up to date. This can be useful if you run all of your self-hosted services or apps with Docker. Every few minutes, Watchtower will pull the latest image for your application and compare it to the one used to run the container. Suppose there are any changes to the image. In that case, Watchtower will automatically restart the container using the new image, and the same `docker run` or `docker-compose` configuration initially used to start it.

## Prerequisites

Before you begin, you should:

-   [Deploy an Ubuntu 20.04 server](https://my.vultr.com/deploy/).
    
-   [Update the server](https://www.vultr.com/docs/update-ubuntu-server-best-practices).
    
-   [Create a non-root user with sudo privileges](https://www.vultr.com/docs/create-a-sudo-user-on-ubuntu-best-practices/).
    
-   [Log in to your server](https://www.vultr.com/docs/how-to-access-your-vultr-vps) as a non-root user.
    

## Installation

### Docker

Ideally, you should already have Docker installed on your server, as you should already be using it. If you don't, you can install it by following these steps:

1.  Remove any older versions of Docker and the Docker engine.
    
    ```
    $ sudo apt remove docker docker-engine docker.io containerd runc
    ```
    
2.  Install Docker using `snap`.
    
    ```
    $ sudo snap install docker
    ```
    

### Docker Container

1.  If you don't have a Docker container running, create one. As an example, you can use the Docker `getting-started` image.
    
    ```
    $ sudo docker run -d -p 80:80 docker/getting-started
    ```
    

To check if you have any existing running containers, you can run `docker ps`.

1.  Create the Watchtower Docker container.
    
    ```
    $ sudo docker run --detach \
    
        --name watchtower \
    
        --volume /var/run/docker.sock:/var/run/docker.sock \
    
        containrrr/watchtower
    ```
    
2.  Check that Watchtower is running by using `docker`. The status should be `Up`.
    
    ```
    $ sudo docker ps
    
    STATUS
    
    Up x seconds/minutes
    ```
    

You have now successfully installed and configured Watchtower to update your Docker container images regularly.

## Additional Configuration

Watchtower has some additional configurations that can be changed using command line arguments.

### Setting the Timezone

You can set the timezone that Watchtower uses by mounting your host machine's `/etc/localtime` file into the container.

```
    $ sudo docker run --detach \

        --name watchtower \

        --volume /var/run/docker.sock:/var/run/docker.sock \

        --volume /etc/localtime:/etc/localtime:ro \

        containrrr/watchtower
```

### Automatically Removing Old Images

By default, Watchtower does not remove old images. Enabling this may be useful if you need to save disk space.

```
    $ sudo docker run --detach \

        --name watchtower \

        --volume /var/run/docker.sock:/var/run/docker.sock \

        --volume /etc/localtime:/etc/localtime:ro \

        containrrr/watchtower \

        --cleanup
```

## More Information

More Watchtower configuration options can be found in the documentation below.

-   [Watchtower Repository](https://github.com/containrrr/watchtower/)
    
-   [Watchtower Documentation](https://containrrr.dev/watchtower/)
    

## Want to contribute?