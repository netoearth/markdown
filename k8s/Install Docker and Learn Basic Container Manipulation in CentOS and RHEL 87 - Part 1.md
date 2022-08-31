In this 4-article series, we will discuss **Docker**, which is an open-source lightweight virtualization tool that runs at top of Operating System level, allowing users to create, run and deploy applications, encapsulated into small containers.

This type of Linux containers are proven to be fast, portable, and secure. The processes that run in a **Docker** container are always isolated from the main host, preventing outside tampering.

**Part 1**: **Install Docker and Learn Basic Container Manipulation in CentOS and RHEL 8/7**

This tutorial provides a starting point on how to install Docker, create and run Docker containers on **CentOS/RHEL 8/7**, but barely scratches the surface of Docker.

### Step 1: Install and Configure Docker

**1.** Earlier versions of **Docker** were called **docker** or **docker-engine**, if you have these installed, you must uninstall them before installing a newer **docker-ce** version.

```
# yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

```

**2.** To install the latest version of **the Docker Engine** you need to set up the Docker repository and install the **yum-utils** package to enable Docker stable repository on the system.

```
# yum install -y yum-utils
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

```

**3.** Now install the newer **docker-ce** version from the Docker repository and **containerd** manually, because due to some issues, Red Hat blocked the installation of `containerd.io > 1.2.0-3.el7`, which is a dependency of **docker-ce**.

```
# yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
# yum install docker-ce docker-ce-cli

```

**4.** After, Docker package has been installed, start the daemon, check its status and enable it system-wide using the below commands:

```
# systemctl start docker 
# systemctl status docker
# systemctl enable docker

```

[![Check Docker Status](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-Status.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-Status.png)

Check Docker Status

**5.** Finally, run a container test image to verify if Docker works properly, by issuing the following command:

```
# docker run hello-world

```

If you can see the below message, then everything is in the right place.

##### Sample Output

Verify Docker Installation

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

**6.** Now, you can run a few basic Docker commands to get some info about Docker:

###### For system-wide information on Docker

```
# docker info

```

[![Check Docker Info](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-System-Info.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-System-Info.png)

Check Docker Info

###### For Docker version

```
# docker version

```

[![Check Docker Version](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-Version-in-CentOS.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Check-Docker-Version-in-CentOS.png)

Check Docker Version

**7.** To get a list of all available Docker commands type docker on your console.

```
# docker

```

[![List Docker Commands](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Help.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Help.png)

List Docker Commands

### Step 2: Download a Docker Image

**8.** In order to start and run a Docker container, first, an image must be downloaded from [Docker Hub](https://hub.docker.com/) on your host. Docker Hub offers a lot of free images from its repositories.

To search for a Docker image, Ubuntu, for instance, issue the following command:

```
# docker search ubuntu

```

[![Docker Search Ubuntu Images](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Search-Ubuntu-Images.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Search-Ubuntu-Images.png)

Docker Search Ubuntu Images

**9.** After you decided on what image you want to run based on your needs, download it locally by running the below command (in this case an **Ubuntu** image is downloaded and used):

```
# docker pull ubuntu

```

[![Download Docker Ubuntu Image](https://www.tecmint.com/wp-content/uploads/2016/01/Download-Docker-Ubuntu-Image.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Download-Docker-Ubuntu-Image.png)

Download Docker Ubuntu Image

**10.** To list all the available Docker images on your host issue the following command:

```
# docker images

```

[![List Docker Images](https://www.tecmint.com/wp-content/uploads/2016/01/Show-Docker-Images.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Show-Docker-Images.png)

List Docker Images

**11.** If you don’t need a Docker image anymore and you want to remove it from the host issue the following command:

```
# docker rmi ubuntu

```

[![Remove Docker Image](https://www.tecmint.com/wp-content/uploads/2016/01/Delete-Docker-Images.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Delete-Docker-Images.png)

Remove Docker Image

### Step 3: Run a Docker Container

When you execute a command against an image you basically obtain a container. After the command that is executing into the container ends, the container stops (you get a non-running or exited container). If you run another command into the same image again a new container is created and so on.

All the containers created will remain on the host filesystem until you choose to delete them by using the `docker rm` command.

**12.** In order to create and run a container, you need to run command into a downloaded image, in this case, **Ubuntu**, so a basic command would be to display the distribution version file inside the container using [cat command](https://www.tecmint.com/13-basic-cat-command-examples-in-linux/), as in the following example:

```
# docker run ubuntu cat /etc/issue

```

[![Run Docker Containers](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Ubuntu-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Ubuntu-Docker-Container.png)

Run Docker Containers

The above command is divided as follows:

```
# docker run [local image] [command to run into container]

```

**13.** To run one of the containers again with the command that was executed to create it, first, you must get the container **ID** (or the name automatically generated by Docker) by issuing the below command, which displays a list of the running and stopped (non-running) containers:

```
# docker ps -l 

```

[![List Running Docker Containers](https://www.tecmint.com/wp-content/uploads/2016/01/Show-Running-Docker-Containers.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Show-Running-Docker-Containers.png)

List Running Docker Containers

**14.** Once the container **ID** has been obtained, you can start the container again with the command that was used to create it, by issuing the following command:

```
# docker start 923a720da57f

```

Here, the string `923a720da57f` represents the container **ID**.

[![Start Docker Containers](https://www.tecmint.com/wp-content/uploads/2016/01/Start-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Start-Docker-Container.png)

Start Docker Containers

**15.** In case the container is running state, you can get its **ID** by issuing `docker ps` command. To stop the running container issue `docker stop` command by specifying the container **ID** or auto-generated name.

```
# docker stop 923a720da57f
OR
# docker stop cool_lalande
# docker ps

```

[![Stop Docker Containers](https://www.tecmint.com/wp-content/uploads/2016/01/Stop-Docker-Containers.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Stop-Docker-Containers.png)

Stop Docker Containers

**16.** A more elegant alternative so you don’t have to remember the container **ID** would be to allocate a unique name for every container you create by using the `--name` option on the command line, as in the following example:

```
# docker run --name ubuntu20.04 ubuntu cat /etc/issue

```

[![Add Name to Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Add-Docker-Container-Name.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Add-Docker-Container-Name.png)

Add Name to Docker Container

**17.** Then, using the name that you allocated for the container, you can manipulate container (**start**, **stop**, **remove**, **top**, **stats**) further just by addressing its name, as in the below examples:

```
# docker start ubuntu20.04
# docker stats ubuntu20.04
# docker top ubuntu20.04 

```

Be aware that some of the above commands might display no output if the process of command that was used to create the container finishes. When the process that runs inside the container finishes, the container stops.

### Step 4: Run an Interactive Session into a Container

**18.** In order to interactively connect into a container shell session, and run commands as you do on any other Linux session, issue the following command:

```
# docker run -it ubuntu bash

```

[![Start Docker Container Interactive Shell](https://www.tecmint.com/wp-content/uploads/2016/01/Start-Ubuntu-Docker-Container-Shell.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Start-Ubuntu-Docker-Container-Shell.png)

Start Docker Container Interactive Shell

The above command is divided as follows:

1.  `-i` is used to start an interactive session.
2.  `-t` allocates a TTY and attaches stdin and stdout.
3.  `ubuntu` is the image that we used to create the container.
4.  `bash` (or **/bin/bash**) is the command that we are running inside the Ubuntu container.

**19.** To quit and return to host from the running container session you must type `exit` command. The **exit** command terminates all the container processes and stops it.

```
# exit

```

**20.** If you’re interactively logged on container terminal prompt and you need to keep the container in running state but **exit** from the interactive session, you can **quit** the console and return to host terminal by pressing `Ctrl+p` and `Ctrl+q` keys.

[![Keep Docker Shell Session Active](https://www.tecmint.com/wp-content/uploads/2016/01/Keep-Docker-Shell-Session.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Keep-Docker-Shell-Session.png)

Keep Docker Shell Session Active

**21.** To reconnect to the running container you need the container **ID** or **name**. Issue `docker ps` command to get the **ID** or **name** and, then, run `docker attach` command by specifying container **ID** or **name**, as illustrated in the image above:

```
# docker attach <container id>

```

**22.** To stop a running container from the host session issue the following command:

```
# docker kill <container id>

```

That’s all for basic container manipulation. In the next tutorial, we will discuss how to save, delete, and run a web server into a Docker container.