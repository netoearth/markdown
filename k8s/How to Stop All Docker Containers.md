In this article, I am going to show you how to stop all Docker containers on your Docker host. So, let’s get started.

### Requirements:

You must have Docker installed in order to run the commands shown in this article.

If you don’t have Docker installed, you may check the following articles on installing Docker to install Docker on your desired Linux distribution.

-   How to Install and Use Docker on Ubuntu 18.04 LTS ([https://linuxhint.com/install\_docker\_ubuntu\_1804/](https://linuxhint.com/install_docker_ubuntu_1804/))
-   Install Docker on Debian 9 ([https://linuxhint.com/install\_docker\_debian\_9/](https://linuxhint.com/install_docker_debian_9/))
-   Install Docker on CentOS 7 ([https://linuxhint.com/install-docker-centos7/](https://linuxhint.com/install-docker-centos7/))
-   Install Docker on Raspberry Pi ([https://linuxhint.com/install\_docker\_raspberry\_pi/](https://linuxhint.com/install_docker_raspberry_pi/))

If you still have any problem installing Docker, you may contact me through [https://support.linuxhint.com](https://support.linuxhint.com/). I will be more than happy to help.

### Stopping A Running Container:

You can stop any running Docker container on your Docker host. To stop a container, you need the ID or name of the container that you want to stop.

To get the container ID and name of all the running containers, run the following command:

As you can see the container ID and name of all the running containers are listed.

![](https://linuxhint.com/wp-content/uploads/2019/02/1-33.png)

Now, let’s say, you want to stop the container **www1** or **c52585c7a69b**.

To do that, you may run one of the following commands:

$ docker container stop www1

Or,

$ docker container stop c52585c7a69b

The container **www1** or **c52585c7a69b** should be stopped.

![](https://linuxhint.com/wp-content/uploads/2019/02/2-34.png)

### Stopping All Running Containers:

You can also stop all the running Docker containers with a single command.

To stop all the running Docker containers, run the following command:

$ docker container stop $(docker container list -q)

All the running Docker containers should be stopped.

![](https://linuxhint.com/wp-content/uploads/2019/02/3-31.png)

Here, **docker container list -q** command returns the container ID of all the running Docker containers. Then the **docker container stop** command stops the containers using the container IDs.

As you can see, there is no running Docker containers in the list.

![](https://linuxhint.com/wp-content/uploads/2019/02/4-29.png)

Again, you can see that all the running Docker containers are stopped.

$ docker container list \-a

![](https://linuxhint.com/wp-content/uploads/2019/02/5-29.png)

### Stopping All Docker Containers:

You can also stop any Docker containers regardless of their status (running, paused etc).

To stop all the Docker containers regardless of their status, run the following command:

$ docker container stop $(docker container list -qa)

All the Docker containers regardless of their status should be stopped.

![](https://linuxhint.com/wp-content/uploads/2019/02/6-28.png)

Here, **docker container list -qa** command returns the container ID of all the Docker containers regardless of their status. Then the **docker container stop** command stops the containers using the container IDs.

You can verify whether the containers are stopped with the following command:

$ docker container list \-a

As you can see, all the containers are stopped.

![](https://linuxhint.com/wp-content/uploads/2019/02/7-25.png)

So, that’s how you stop all the Docker containers on your Docker host. Thanks for reading this article.

### About the author

![](https://linuxhint.com/wp-content/uploads/2018/01/photo2-150x150.png)

Freelancer & Linux System Administrator. Also loves Web API development with Node.js and JavaScript. I was born in Bangladesh. I am currently studying Electronics and Communication Engineering at Khulna University of Engineering & Technology (KUET), one of the demanding public engineering universities of Bangladesh.