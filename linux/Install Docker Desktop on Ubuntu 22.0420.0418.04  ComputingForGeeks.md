**Containerization** is the packaging of the required software code to run a lightweight executable referred to as a _container_. The packaged software includes the operating system, libraries, and dependencies. The containerization technology has been in existence for several decades, but the emergence of an open-source Docker Engine hastened its acquisition. **Docker Engine** was introduced in **_2013_** as an industry-standard tool with simple developer tools and a universal packaging approach. Today, this tool is used by organizations to create and modernize existing applications for the cloud.

**Docker Desktop** is an easy-to-install application that runs on Linux, macOS, and Windows systems. It enables one to build and share containerized applications. The components included in Docker Desktop are:

-   Docker Engine
-   Docker CLI client
-   Docker Compose
-   Docker Content Trust
-   Kubernetes
-   Credential Helper

Docker Desktop is preferred because it works with your choice of language and development tool as well as gives you access to innumerable images and templates from Docker Hub. With this, you can easily extend your environment, and rapidly auto-build, integrate, and collaborate via a secure repository.

The other features associated with Docker Desktop are:

-   **Easy to install** – It makes it easy to set up a complete Docker environment for development.
-   **Automatic updates** – this guarantees up to date versions and security
-   **Fast and reliable performance** with native Windows Hyper-V virtualization
-   **Ease of management** – automate the installation, scaling, and management of containerized workloads and services. On **_Windows_**, you have the ability to toggle between Linux and Windows Server environments to build applications.
-   **Volume mounting for code and data**, including file change notifications and easy access to running containers on the localhost network
-   In-container development and debugging with supported IDEs
-   **Share applications on any cloud platform** – It provides the ability to containerize and share applications in multiple languages and frameworks.

This guide provides the required steps on how to install and use Docker Desktop on Ubuntu 22.04|20.04|18.04

## System requirements

This setup will work best if your Linux host meets the below specifications:

-   64-bit kernel and CPU support for virtualization
-   Memory above 4GB
-   QEMU must be version 5.2 or newer
-   Gnome or KDE Desktop environment.
-   systemd init system.
-   KVM virtualization support

The above is required since Docker Desktop for Linux runs a Virtual Machine (VM).

To load the KVM module manually, run the command:

```
sudo modprobe kvm
sudo modprobe kvm_intel  # Intel processors
sudo modprobe kvm_amd    # AMD processors
```

Check if the module is enabled.

```
$ lsmod | grep kvm
kvm_intel             282624  0
kvm                   663552  1 kvm_intel
```

Add your system user to the KVM group.

```
sudo usermod -aG kvm $USER
```

## Step 1 – Install Docker Engine

Proceed and set up the Docker repository on Ubuntu 22.04|20.04|18.04. First, remove the existing repository

```
sudo apt remove docker docker-engine docker.io 2>/dev/null
sudo apt update
```

Install the required packages:

```
sudo apt -y install apt-transport-https ca-certificates curl software-properties-common
```

Add the Docker repository.

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Install Docker on Ubuntu with the command:

```
sudo apt install docker-ce docker-ce-cli containerd.io uidmap
```

Add your user account to docker group:

```
sudo usermod -aG docker $USER
newgrp docker
```

Now download the latest [Docker Desktop](https://docs.docker.com/desktop/release-notes/) package. Copy the link to the latest Debian package and download with `wget`.

```
wget https://desktop.docker.com/linux/main/amd64/docker-desktop-4.12.0-amd64.deb
```

Uninstall the previous installations of Docker Desktop tech, preview or beta versions

```
sudo apt remove docker-desktop
```

Clean up the system and completely remove the data files

```
rm -r $HOME/.docker/desktop
sudo rm /usr/local/bin/com.docker.cli
sudo apt purge docker-desktop
```

For those using non-Gnome Desktop environments, you need to install the package below.

```
sudo apt install gnome-terminal
```

Once downloaded, execute the command below to install Docker Desktop.

```
sudo apt install ./docker-desktop-*-amd64.deb
```

Accept the installation of any dependencies required:

```
....
0 upgraded, 44 newly installed, 0 to remove and 0 not upgraded.
Need to get 35.5 MB/439 MB of archives.
After this operation, 129 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

After the execution, a new entry is added in the /etc/hosts for Kubernetes as shown.

```
$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1kubernetes.docker.internal
# End of section
```

## Step 3 – Launch and Use Docker Desktop

Docker Desktop can be launched from the App Menu as shown

[![Install and Use Docker Desktop on Ubuntu 22.0420.0418.04](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-22.0420.0418.04.png?ezimgfmt=rs:492x236/rscb23/ng:webp/ngcb23 "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 1")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22492%22%20height=%22236%22%3E%3C/svg%3E)

Alternatively, launch it from the terminal using the command:

```
systemctl --user start docker-desktop
```

Agree to the License terms.

[![Install and Use Docker Desktop on Ubuntu](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu.png?ezimgfmt=rs:696x522/rscb23/ng:webp/ngcb23 "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 2")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22702%22%20height=%22526%22%3E%3C/svg%3E)

Docker Desktop will start as below.

[![Install and Use Docker Desktop on Ubuntu 1](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-1.png?ezimgfmt=rs:696x541/rscb23/ng:webp/ngcb23 "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 3")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22857%22%20height=%22666%22%3E%3C/svg%3E)

Once started, you will see this home page.

[![Install and Use Docker Desktop on Ubuntu 2](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-2.png?ezimgfmt=rs:696x541/rscb23/ng:webp/ngcb23 "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 4")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22842%22%20height=%22655%22%3E%3C/svg%3E)

Check the versions for the below executables.

```
$ docker compose version
Docker Compose version v2.6.0

$ docker --version
Docker version 20.10.17, build 100c701

$ docker version
Client: Docker Engine - Community
 Cloud integration: v1.0.25
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:02:46 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:00:51 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.6
  GitCommit:        10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc:
  Version:          1.1.2
  GitCommit:        v1.1.2-0-ga916309
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Remember both Docker Desktop and Docker Engine exist on the machine. But now Docker Desktop stores its images and containers in an isolated storage location in the VM.

Running these two simultaneously may result in errors. Probably you can stop the Docker Engine service with the command:

```
sudo systemctl stop docker docker.socket containerd
sudo systemctl disable docker docker.socket containerd
```

You can as well switch between Docker Desktop and Docker Engine. View the available contexts.

```
$ docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT                                   KUBERNETES ENDPOINT   ORCHESTRATOR
default             moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                                             swarm
desktop-linux *     moby                                                          unix:///home/ubuntu/.docker/desktop/docker.sock   
```

To use a preferred context, say **default**, then the command will be:

```
$ docker context use default
default
```

To use the **docker-desktop** context, the command will be:

```
$ docker context use desktop-linux
desktop-linux
```

## Step 4 – Configure Docker Desktop

You can configure Docker Desktop as per your preferences. These settings include updates, version channels, Docker Hub login e.t.c

-   **General tab**

[![Install and Use Docker Desktop on Ubuntu 3](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-3.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 5")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22816%22%20height=%22629%22%3E%3C/svg%3E)

The settings tab appears with the above options. In the general tab, you can make several configurations that include:

-   **Start Docker Desktop when you log in** to automatically start Docker Desktop when you open your session. This can also be enabled in the terminal with the command:

```
systemctl --user enable docker-desktop
```

-   **Send usage statistics** to send reports for Docker Desktop
-   **Show weekly tips** to display advice messages about the tool
-   **Open Docker Desktop dashboard at startup** to automatically display the dashboard on startup
-   **Enable Docker Compose V1/V2 compatibility mode** this option is used to enable docker-compose to use Docker Compose V2

-   **Resource tab**

The other tab is the **Resources** tab.

[![Install and Use Docker Desktop on Ubuntu 4](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-4.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 6")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22797%22%20height=%22611%22%3E%3C/svg%3E)

Here, you limit the resources to be used by Docker. Make the desired settings and save. You can as well configure **File Sharing** under the **advanced settings** to allow local directories to be shared with the Linux containers.

[![Install and Use Docker Desktop on Ubuntu 5](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-5.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 7")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22833%22%20height=%22591%22%3E%3C/svg%3E)

Add (+) and remove (-) directories appropriately then apply the changes.

Still in the Resources tab, there is the **network** tab. This tab is useful for network configurations in case the default choice of subnet clashes with something on your system. You can as well specify custom subnets here.

[![Install and Use Docker Desktop on Ubuntu 6](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-6.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 8")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22820%22%20height=%22593%22%3E%3C/svg%3E)

There are many other configurations such as the Software Updates, Kubernetes, Experimental Features e.t.c which you can explore on your own.

## Step 5 – Running containers with Docker Desktop

Now once the desired configurations have been made, run a sample container. Navigate to home and pull the desired container image. For this demonstration, I will pull and run Nginx.

[![Install and Use Docker Desktop on Ubuntu 7 1](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-7-1.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 9")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22586%22%20height=%22431%22%3E%3C/svg%3E)

Once pulled, the container will start as shown.

[![Install and Use Docker Desktop on Ubuntu 8](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-8.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 10")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22847%22%20height=%22657%22%3E%3C/svg%3E)

The containers can be managed from the **container** tab. Here, you can access the containers CLI, view it on the browser and stop/start.

[![Install and Use Docker Desktop on Ubuntu 9](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-9.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 11")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22859%22%20height=%22389%22%3E%3C/svg%3E)

Manage Docker images in the **images** tab

[![Install and Use Docker Desktop on Ubuntu 10](https://computingforgeeks.com/wp-content/uploads/2022/05/Install-and-Use-Docker-Desktop-on-Ubuntu-10.png "Install Docker Desktop on Ubuntu 22.04|20.04|18.04 12")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%22845%22%20height=%22473%22%3E%3C/svg%3E)

## Step 6 – Update/Uninstall Docker Desktop

Whenever there are newer Docker Desktop versions the UI shows notifications. Each time you want to upgrade manually, download a new package.

Before you upgrade, ensure that the instances running locally are stopped then follow the normal Docker Desktop installation steps.

To uninstall the package, run the command:

```
sudo apt remove docker-desktop
```

Completely clean up by deleting the files:

```
rm -r $HOME/.docker/desktop
sudo rm /usr/local/bin/com.docker.cli
sudo apt purge docker-desktop
```

You also need to remove the `credsStore`and `currentContext` form the file `$HOME/.docker/config.json`

## Final Verdict

That was a brief demonstration on how to install and get started with Docker Desktop on Ubuntu 22.04|20.04|18.04. There are many other features that you can explore on your own. I hope this was significant to you.

Related posts:

-   [Install Docker CE on Linux Systems](https://computingforgeeks.com/install-docker-ce-on-linux-systems/)
-   [Run ONLYOFFICE Workspace as a Docker Container](https://computingforgeeks.com/how-to-run-onlyoffice-workspace-as-docker-container/)
-   [Run Ghost CMS in Docker Containers](https://computingforgeeks.com/how-to-run-ghost-cms-in-docker-containers/)
-   [Install Mirantis cri-dockerd as Docker Engine shim for Kubernetes](https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/)
-   [How To Run Graylog Server in Docker Containers](https://computingforgeeks.com/how-to-run-graylog-server-in-docker-containers/)