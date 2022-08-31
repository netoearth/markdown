Following the previous **Docker** article, this tutorial will discuss how to save a Docker container into a new image, remove a container, and run an **Nginx** web server inside a container.

#### Requirements

-   [How to Install Docker and Run Containers in CentOS/RHEL 8/7 – Part 1](https://www.tecmint.com/install-docker-and-learn-containers-in-centos-rhel-7-6/ "Install Docker in CentOS 8/7")

### How To Run and Save a Docker Container

**1.** In this example, we will run and save an **Ubuntu-based** Docker container where **the Nginx** server will be installed. But before committing any changes to a container, first start the container with the below commands which updates and installs **Nginx** daemon into Ubuntu image:

```
# docker run ubuntu bash -c "apt-get -y update" 
# docker run ubuntu bash -c "apt-get -y install nginx" 

```

[![Install Nginx on Ubuntu Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Install-Nginx-on-Ubuntu-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Install-Nginx-on-Ubuntu-Docker-Container.png)

Install Nginx on Ubuntu Docker Container

If you get error ‘**E: Unable to locate package nginx**‘, then you need to connect to a container with interactive CLI and install nginx as shown.

```
# docker run -it ubuntu bash
# apt install nginx
# exit

```

**2.** Next, after **Nginx** package is installed, issue the command `docker ps -l` to get the **ID** or **name** of the running container.

```
# docker ps -l

```

[![Find Docker Container ID Name](https://www.tecmint.com/wp-content/uploads/2016/01/Find-Docker-Container-ID-Name.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Find-Docker-Container-ID-Name.png)

Find Docker Container ID Name

And apply changes by running the below command:

```
# docker commit 5976e4ae287c ubuntu-nginx

```

Here, `5976e4ae287c` represents the container `ID` and `ubuntu-nginx` represents the name of the new image that has been saved with committed changes.

In order to view if the new image has been successfully created just run `docker images` command and a listing of all saved images will be shown.

```
# docker images

```

[![Docker Container Changes](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Container-Changes.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Docker-Container-Changes.png)

Docker Container Changes

Chances are that the installation process inside the container finishes fast which leads to a non-running container (container is stopped). In this case the `docker ps` command won’t show any output because no container is running.

In order to be able to still get the container’s id run `docker ps -a | head -3` to output the most recent containers and identify the container based on the command issued to create the container and the exited status.

**3.** Alternatively, you can actively enter container sessions by running `docker run -it ubuntu bash` command and execute the further `apt-get install nginx` command. While the command is running, detach from the container using `Ctrl-p + Ctrl-q` keys and the container will continue running even if the Nginx installation process finishes.

```
# docker run -it ubuntu bash
# apt-get install nginx

```

[![Install Nginx on Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Docker-Container-and-Install-Nginx.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Docker-Container-and-Install-Nginx.png)

Install Nginx on Docker Container

Then, get the running container id with `docker ps` and commit changes. When finished, re-enter to container console using `docker attach` and type `exit` to stop the container.

```
# docker ps
# docker attach 3378689f2069
# exit

```

[![Attach Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Attach-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Attach-Docker-Container.png)

Attach Docker Container

**4.** To further test if the recent image has been committed properly (in this case **Nginx** service has been installed), execute the below command in order to generate a new container which will output if Nginx binary was successfully installed:

```
# docker run ubuntu-nginx whereis nginx

```

[![Generate New Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Generate-New-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Generate-New-Docker-Container.png)

Generate New Docker Container

**5.** To remove a container use the `rm` command against a container ID or name, which can be obtained using `docker ps -a` command:

```
# docker ps -a
# sudo docker rm 36488523933a

```

[![Remove Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Remove-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Remove-Docker-Container.png)

Remove Docker Container

### How to Run Nginx inside Docker Container

**6.** In this part we will concentrate on how you can run and access a network service, such as an **Nginx** web server, inside Docker, using the `ubuntu-nginx` image created earlier where the Nginx daemon was installed.

The first thing that you need to do is to create a new container, map host-container ports, and enter container shell by issuing the below command:

```
# docker run -it -p 81:80 ubuntu-nginx /bin/bash
# nginx &

```

Here, the `-p` option exposes the host port to the container port. While the host port can be arbitrary, with the condition that it should be available (no other host services should listen on it), the container port must be exactly the port that the inside daemon is listening to.

Once you’re connected to container session, start **Nginx** daemon in the background and detach from container console by pressing `Ctrl-p + Ctrl-q` keys.

[![Run Nginx Inside Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Nginx-Inside-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Run-Nginx-Inside-Docker-Container.png)

Run Nginx Inside Docker Container

**7.** Now, run `docker ps` to get the state of your running container. You can also view host network sockets by issuing the following command:

```
# docker ps
OR
# netstat -tlpn 

```

[![View Docker Container Running State](https://www.tecmint.com/wp-content/uploads/2016/01/View-Docker-Container-Running-State.png)](https://www.tecmint.com/wp-content/uploads/2016/01/View-Docker-Container-Running-State.png)

View Docker Container Running State

**8.** In order to visit the page served by the Nginx container, open a browser from a remote location in your LAN and type the IP address of your machine using the HTTP protocol.

[![Verify Nginx Running under Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Verify-Nginx-Page.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Verify-Nginx-Page.png)

Verify Nginx Running under Docker Container

**9.** To stop the container run the following command followed by container ID or name:

```
# docker ps
# docker stop fervent_mccarthy
# docker ps

```

[![Stop Running Docker Container](https://www.tecmint.com/wp-content/uploads/2016/01/Stop-Docker-Running-Container.png)](https://www.tecmint.com/wp-content/uploads/2016/01/Stop-Docker-Running-Container.png)

Stop Running Docker Container

As an alternative to stop the running container, enter container shell command prompt and type exit to finish process:

```
# docker attach fervent_mccarthy
# exit

```

Be aware that using this kind of container to run web servers or other kinds of services are best suited only for development purposes or tests due to the fact that the services are only active while the container is running. Exiting the container disrupts all running services or any changes made.

**Further Reading:**

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**