**ctop** is a free open source, simple and cross-platform [top-like command-line tool](https://www.tecmint.com/12-top-command-examples-in-linux/) for monitoring container metrics in real-time. It allows you to get an overview of metrics concerning CPU, memory, network, I/O for multiple containers and also supports inspection of a specific container.

[![Docker Container Monitoring](https://www.tecmint.com/wp-content/uploads/2018/07/Docker-Container-Monitoring.gif)](https://www.tecmint.com/wp-content/uploads/2018/07/Docker-Container-Monitoring.gif)

Docker Container Monitoring

At the time of writing this article, it ships with built-in support for [Docker](https://www.tecmint.com/install-docker-and-learn-containers-in-centos-rhel-7-6/) (default container connector) and **runC**; connectors for other container and cluster platforms will be added in future releases.

### How to Install ctop in Linux Systems

Installing the latest release of **ctop** is as easy as running the following commands to download the binary for your Linux distribution and install it under **/usr/local/bin/ctop** and make it executable to run it.

```
$ sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.1/ctop-0.7.1-linux-amd64  -O /usr/local/bin/ctop
$ sudo chmod +x /usr/local/bin/ctop

```

Alternatively, install **ctop** via Docker using following command.

```
$ docker run --rm -ti --name=ctop -v /var/run/docker.sock:/var/run/docker.sock quay.io/vektorlab/ctop:latest

```

Once you have installed **ctop**, you can run it to list all your containers whether active or not.

```
$ ctop

```

[![Monitor Docker Containers](https://www.tecmint.com/wp-content/uploads/2018/07/List-All-Docker-Containers.png)](https://www.tecmint.com/wp-content/uploads/2018/07/List-All-Docker-Containers.png)

Monitor Docker Containers

You can use the **Up** and **Down** arrow keys to highlight a container and click **Enter** to select it. You will see a menu as shown in the following screenshot. Choose **“single view”** and click on it to inspect the selected container.

[![Monitor Single Docker Container](https://www.tecmint.com/wp-content/uploads/2018/07/Monitor-Docker-Container.png)](https://www.tecmint.com/wp-content/uploads/2018/07/Monitor-Docker-Container.png)

Monitor Single Docker Container

The following screenshot shows the single view mode for a specific container.

[![Inspect a Single Container](https://www.tecmint.com/wp-content/uploads/2018/07/inspect-a-single-container.png)](https://www.tecmint.com/wp-content/uploads/2018/07/inspect-a-single-container.png)

Inspect a Single Container

To display active containers only, use the `-a` flag.

```
$ ctop -a 

```

[![Check Active Docker Container](https://www.tecmint.com/wp-content/uploads/2018/07/only-show-active-containers.png)](https://www.tecmint.com/wp-content/uploads/2018/07/only-show-active-containers.png)

Check Active Docker Container

To display CPU as `%` of system total, use the `-scale-cpu` option.

```
$ ctop -scale-cpu

```

You can also filter containers using the `-f` flag, for example.

```
$ ctop -f app

```

Additionally, you can select initial container sort field using the `-s` flag, and see the **ctop** help message as shown.

```
 
$ ctop -h

```

Note that connectors for other container and cluster systems are yet to be added to **ctop**. You can find more information from the [Ctop Github repository](https://github.com/bcicen/ctop).

**ctop** is a simple top-like tool for visualizing and monitoring container metrics in real-time. In this article, we’ve expalined how to install and use ctop in Linux. You can share your thoughts or ask any questions via the comment form below.

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**