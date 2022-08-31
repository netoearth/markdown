[![](https://www.unixmen.com/wp-content/uploads/2022/08/code-gb0336743c_1920-696x464.jpg "code-gb0336743c_1920")](https://www.unixmen.com/wp-content/uploads/2022/08/code-gb0336743c_1920.jpg)

![container registry for linux](https://www.unixmen.com/wp-content/uploads/2022/08/code-gb0336743c_1920-300x300.jpg)

Docker container images are delivered and stored by an application called Docker Registry. Registries consolidate container images and speed up developer build times. Virtualization provides the same runtime environment, but creating an image might require effort. Container orchestration is successful when employing containers. This tutorial teaches how to set up a private Docker registry and configure it for public access.

## Jfrog Container Registry: What is it?

All container deployment, administration, expansion, and networking facets are automated via container orchestration. As a result, businesses that control a large number of [Linux® servers](https://www.unixmen.com/monitor-linux-server-nagios-core-using-snmp/) and containers, for instance, may profit from container orchestration.

The largest and best registry for Kubernetes is the [container registry by JFrog](https://jfrog.com/container-registry/), which handles Helm Chart repositories and Docker containers. Organize your Docker images in one location, doing away with Docker Hub’s throttling and retention difficulties. JFrog connects with your development environment to enable secure, accurate, and speedy access to external Docker container registries. You will build your own Docker Registry on Linux in this tutorial.

## Installing and Configuring Private Docker Registry on Linux

### Step 01:

Docker on the command line is helpful for getting started and testing containers, but it becomes cumbersome for larger deployments requiring many parallel instances.

Docker Compose may use one .yml file to set up each container’s setup and the data it needs to interact with other containers. You may give instructions to each of the parts that make up your application and manage them [collectively](https://www.unixmen.com/7-benefits-of-mozillas-rapid-release/) using the docker-compose command-line tool.

You will use Docker Compose to administer Docker Registry because it is a program with several components on its own. You must create a docker-compose.yml file to specify the registry and the location on storage where its data will be kept before you can launch an instance of the registry.

The settings will be kept on the primary server in a docker-registry directory. Run the command to create it:

mkdir ~/docker-registry

Navigate it to:

cd ~/docker-registry

Then, make a subfolder named data where your registry’s photos will be stored:

            mkdir data

By running the following command, create and open a file named docker-compose.yml:

      nano docker-compose.yml

The following code defines a basic instance of a Docker Registry:

            version: ‘3’

services:

  registry:

    image: registry:2

    ports:

    – “5000:5000”

    environment:

      REGISTRY\_STORAGE\_FILESYSTEM\_ROOTDIRECTORY: /data

    volumes:

      – ./data:/data

You can initialize the configuration by running the “docker-compose up” command.

### Step 02:

You’ve configured HTTPS on your domain as part of the requirements. So you only need to set up Nginx to direct traffic from your domain to the registry container in order to expose your protected Docker Registry there.

Your server configuration is already set up in the file /etc/nginx/sites-available/your domain. Run this command to modify it:

sudo nano /etc/nginx/sites-available/your\_domain

Your registry will be waiting on port 5000, therefore, you must forward all traffic there. In order to get more information from the host about the request itself, you need additionally include headers to the request routed to the registry. The following lines should be added to the location block in place of the current content:

location / {

    # Do not allow connections from docker 1.5 and earlier

    # docker pre-1.6.0 did not properly set the user agent on ping, catch “Go \*” user agents

    if ($http\_user\_agent ~ “^(docker\\/1\\.(3|4|5(?!\\.\[0-9\]-dev))|Go ).\*$” ) {

      return 404;

    }

    proxy\_pass                          http://localhost:5000;

    proxy\_set\_header  Host              $http\_host;   # required for docker client’s sake

    proxy\_set\_header  X-Real-IP         $remote\_addr; # pass on real client’s IP

    proxy\_set\_header  X-Forwarded-For   $proxy\_add\_x\_forwarded\_for;

    proxy\_set\_header  X-Forwarded-Proto $scheme;

    proxy\_read\_timeout                  900;

}

After it is done, you may save and close the file and restart Nginx to apply the changes. After configuring port forwarding, you’ll advance to strengthening your registry’s security.

### Step 03:

You can use HTTP authentication to restrict access to your Docker Registry by configuring it in Nginx for the websites it administers. To do this, you’ll use htpasswd to construct an authentication file and add a valid username and password combinations to it.

By installing the apache2-utils package, you can get the htpasswd tool. Click here for the [steps on how to install the package](https://howtoinstall.co/en/apache2-utils).

The authentication file and credentials should be kept at /docker-registry/auth. Create it by running:

mkdir ~/docker-registry/auth

Navigate to it:

cd ~/docker-registry/auth

Replace username with the username you wish to use when creating the first user. The bcrypt algorithm is one that Docker requires, and the -B option commands its use:

htpasswd -Bc registry.password username

When asked, provide the password; registry.password will then be added with the combination of credentials.

You should change docker-compose.yml to instruct Docker to utilize the file you prepared to authenticate users now that the list of credentials has been established. The file can be opened by running the following command:

nano ~/docker-registry/docker-compose.yml

### Step 04:

By telling Docker Compose to keep it running, you can ensure that the registry container begins each time the system boots up or recovers from a crash. This is done by entering the following command under the registry block:

restart: always

### Step 05:

You must make sure that your registry can handle huge file uploads before you can push a picture to it.

Nginx’s default file upload size restriction is 1 m, which is far too little for Docker images. To raise it, you must change the /etc/nginx/nginx.conf main configuration file. Run this command to modify it:

sudo nano /etc/nginx/nginx.conf

Add the following line to the http section:

client\_max\_body\_size 16384m;

Save the file and restart Nginx.

## Final Thoughts

Your Docker Registry is up and running. First, you made your own Docker registry in this guide. Then you configured SSL and HTTP authentication and installed the required requirements. Read more about [backing up Docker Containers](https://www.unixmen.com/backup-docker-container-and-upload-to-docker-hub/).

Local Docker images are kept in a private Kubernetes registry that JFrog offers. It enables you to connect to each tier for each of your apps and have complete control and visibility over your code-to-cluster process.