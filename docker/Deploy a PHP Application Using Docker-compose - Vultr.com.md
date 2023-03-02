PHP applications are usually composed by a webserver, a relational database system and the language interpreter itself. In this tutorial we will be leveraging a full PHP application stack using docker. This is an in-depth tutorial in which we are going to build and orchestrate containers for Nginx (the webserver), MySQL (the database system) and PHP.

For the sake of this tutorial, we will write a simple application that reads a list of cities from a database and displays it on a web page, this way we will demonstrate a basic, but working, PHP application.

This guide assumes that you have Docker-CE already installed and at least a minimal working knowledge of docker. For that matter you may review the following tutorials:

-   [Installing Docker CE on Ubuntu 16.04](https://www.vultr.com/docs/installing-docker-ce-on-ubuntu-16-04)
    
-   [Installing Docker CE on CentOS 7](https://www.vultr.com/docs/installing-docker-ce-on-centos-7)
    
-   [How to Use Docker: Creating Your First Docker Container](https://www.vultr.com/docs/how-to-use-docker-creating-your-first-docker-container)
    

## Configuring our working environment

A real life docker-based application will typically be composed of several containers. Managing these manually can easily become quite messy and cumbersome. That's where docker-compose comes into play. It helps you to manage a number of containers through a simple `yaml` configuration file.

Install docker-compose.

```
curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

Create a folder to hold all the necessary files of this example and then `cd` into it. From now on, this is our working directory and every command will be executed inside this folder and every path will be referenced relative to it. This folder may be referenced later as `WORKING_DIR`.

```
mkdir ~/docker

cd ~/docker
```

Now create three more folders.

```
mkdir php nginx app
```

The `php` folder is where we will build our custom PHP image, the `nginx` folder will hold the necessary files for our custom nginx image and the `app` folder is where we will be putting the source code and configuration of our sample application.

## Configuring the PHP container

In this example, we are going to use `php-fpm` to connect to the Nginx webserver. We will be using the official PHP base image. However, we also need to install and enable some extensions so that we may access the database. Inside the `php` folder create a file named `Dockerfile` and put the following contents into it.

```
FROM php:7.1-fpm-alpine3.4

RUN apk update --no-cache \

    && apk add --no-cache $PHPIZE_DEPS \

    && apk add --no-cache mysql-dev \

    && docker-php-ext-install pdo pdo_mysql
```

Note that we are using the Alpine version of the official PHP image. Alpine is a very tiny distribution targeted towards containers by providing much smaller footprints. Also, note the use of the command `docker-php-ext-install`, the official PHP image provides this command to ease the process of installing and configuring PHP extensions.

Now, let's build this Docker image by issuing the following (inside our `WORKING_DIR`):

```
docker build -t vultr-php php/
```

### The `docker-compose.yml` file

As already mentioned, `docker-compose` allows you to manage a number of containers through a simple configuration file. This configuration file is typically named `docker-compose.yml`. Create this file inside the `app` folder.

```
touch app/docker-compose.yml
```

Now put the following contents into this file.

```
version: '2'

services:

  php:

    image: vultr-php

    volumes:

      - ./:/app

    working_dir: /app
```

We will explain this syntax. First, note the first line.

```
version: '2'
```

This specifies the version of the `docker-compose.yml` configuration file used. The next line specifies the services, or in other words, the containers to be provisioned.

```
services:

  php:

    image: vultr-php

    volumes:

      - ./:/app

    working_dir: /app
```

Note that every service has a specific key inside the `services` block. The name specified here will be used to reference this specific container later. Also note that inside the `php` configuration, we define the image used to run the container (this is the image we built previously). We also define a volume mapping.

```
volumes:

  - ./:/app
```

This tells `docker-compose` to map the current directory (`./`) to the `/app` directory inside the container. The last line sets the `/app` folder inside the container as the working directory, which means that this is the folder where all future commands inside a container are by default executed from.

We can now orchestrate our containers.

```
cd ~/docker/app

docker-compose up -d
```

You can run the following command to make sure that the PHP container was executed:

```
docker ps
```

### How to execute commands inside the containers

Still inside the `app` folder, we can run any command inside a defined service container with the help of the `docker-compose` command.

```
docker-compose exec [service] [command]
```

The `[service]` placeholder refers to the service key. In our case, this was `php`. Let's run a command inside the container

to check our PHP version.

```
docker-compose exec php php -v
```

You will see the following output.

```
PHP 7.1.14 (cli) (built: Feb  7 2018 00:40:45) ( NTS )

Copyright (c) 1997-2018 The PHP Group

Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
```

## Configuring the Nginx container

Just like the PHP container, we need to create a custom image for the webserver. But in this case, we just need to provide a configuration for our `virtual host`. Make sure you are inside our `WORKING_DIR` and create a `Dockerfile` inside the `nginx` folder:

```
cd ~/docker

touch nginx/Dockerfile
```

Now put the following contents into this `Dockerfile`:

```
FROM nginx:1.13.8-alpine

COPY ./default.conf /etc/nginx/conf.d/default.conf
```

We are using the default Nginx image based on Alpine. On this Docker file we simply copy a configuration file into our application setup. Before building this image, create a configuration file.

```
touch nginx/default.conf
```

Now populate it with this content.

```
server {

    listen 80 default_server;

    listen [::]:80 default_server ipv6only=on;



    root /app;

    index index.php;



    #server_name server_domain_or_IP;



    location / {

        try_files $uri $uri/ /index.php?$query_string;

    }



    location ~ \.php$ {

        try_files $uri /index.php =404;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        fastcgi_pass php:9000;

        fastcgi_index index.php;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        include fastcgi_params;

    }

}
```

Note that at the `fastcgi_pass php:9000` line we are referencing the PHP container by it's name inside the `service` block of the `docker-compose.yml` configuration file. Internally `docker-compose` creates a network and assigns the service name as the host name to each of the services defined. We can now build the Nginx image.

```
docker build -t vultr-nginx nginx/
```

### Updating `docker-compose.yml`

Now update the `app/docker-compose.yml` file.

```
version: '2'

services:

  php:

    image: vultr-php

    volumes:

      - ./:/app

    working_dir: /app

  web:

    image: vultr-nginx

    volumes:

      - ./:/app

    depends_on:

      - php

    ports:

      - 80:80
```

We have only added a new service. The configuration is nearly the same, except for the following.

```
depends_on:

  - php

ports:

  - 80:80
```

Once the Nginx container needs the PHP service to be fully initialized, we force this requirement in the `depends_on` option. The `ports` configuration key maps a host port to a container port, here we map the port `80` in the host to the port `80` in the container.

Now create a file called `index.php` inside the `app` folder and put the following in it.

```
<?php phpinfo();
```

Make sure the port `80` is accessible through your firewall and execute the following.

```
cd ~/docker/app

docker-compose up -d
```

Once again, double check that the service is up.

```
docker ps
```

Open a browser and access `[vultr-instance-ip]`. You may find out your Vultr instance IP address by running the following.

```
hostname -I
```

You will see the PHP info page.

## Configuring the MySQL container

The official MySQL image allows you to configure the container through simple environment variables. This can be done with an `environment` option inside the service block definition. Update the `~/docker/app/docker-compose.yml` file

to the following.

```
version: '2'

services:

  php:

    image: vultr-php

    volumes:

      - ./:/app

    working_dir: /app

  web:

    image: vultr-nginx

    volumes:

      - ./:/app

    depends_on:

      - php

    ports:

      - 80:80

  mysql:

    image: mysql:5.7.21

    volumes:

      - ./:/app

      - dbdata:/var/lib/mysql

    environment:

      - MYSQL_DATABASE=world

      - MYSQL_ROOT_PASSWORD=root

    working_dir: /app

volumes:

  dbdata:
```

Now we've defined a new service for the database. Notice the line `dbdata:/var/lib/mysql`. This mounts the path on the container `/var/lib/mysql` to a persistent volume managed by Docker, this way the database data persists after the container is removed. This volume needs to be defined in a top-level block as you can see in the end of the file.

Before orchestrating our new configuration, let's download a sample MySQL database. The [official MySQL documentation](https://dev.mysql.com/doc/index-other.html) provides some sample databases. We will be using the well-known world database. This database provides a listing of countries and cities. To download this sample, execute the following inside our app folder.

```
curl -L http://downloads.mysql.com/docs/world.sql.gz -o world.sql.gz

gunzip world.sql.gz
```

Now lets orchestrate our containers.

```
docker-compose up -d
```

As you may have already noticed, the `docker-compose up` command starts only the containers that are not already started. It checks for the differences between your `docker-compose.yml` file and the current configuration of running containers.

One more time, check that the MySQL container was started.

```
docker ps
```

Now populate the world database.

```
docker-compose exec -T mysql mysql -uroot -proot world < world.sql
```

You can verify that the database was populated by selecting data directly from the database. First access the MySQL prompt inside the container.

```
docker-compose exec mysql mysql -uroot -proot world
```

In the MySQL prompt, run the following.

```
select * from city limit 10;
```

You will see a list of cities. Now quit the MySQL prompt.

```
mysql> exit
```

## Building our application

Now that all of the necessary containers are up and running, we can focus on our sample application. Update the `app/index.php` file to the following.

```
<?php



$pdo = new PDO('mysql:host=mysql;dbname=world;charset=utf8', 'root', 'root');



$stmt = $pdo->prepare("

    select city.Name, city.District, country.Name as Country, city.Population

    from city

    left join country on city.CountryCode = country.Code

    order by Population desc

    limit 10

");

$stmt->execute();

$cities = $stmt->fetchAll(PDO::FETCH_ASSOC);



?>



<!doctype html>

<html>

<head>

    <meta charset="UTF-8">

    <title>Vultr Rocks!</title>

</head>

<body>

    <h2>Most Populous Cities In The World</h2>

    <table>

    <thead>

        <tr>

            <th>Name</th>

            <th>Country</th>

            <th>District</th>

            <th>Population</th>

        </tr>

    </thead>

    <tbody>

        <?php foreach($cities as $city): ?>

            <tr>

                <td><?=$city['Name']?></td>

                <td><?=$city['Country']?></td>

                <td><?=$city['District']?></td>

                <td><?=number_format($city['Population'], 0)?></td>

            </tr>

        <?php endforeach ?>

    </tbody>

    </table>

</body>

</html>
```

If you access `[vultr-instance-ip]` in a web browser, you will see a list of the most populous cities in the world. Congratulations, you have deployed a fully working PHP application using docker.

## Conclusion

In this tutorial, I have demonstrated step by step how to configuring a fully working PHP application. We built custom images for PHP and Nginx, and configured docker-compose to orchestrate our containers. Despite being very basic and simple, this setup reflects a real life scenario.

In this guide, we have built and tagged our images locally. For a more flexible setup, you can push these images to a [docker registry](https://docs.docker.com/registry/). You may push to the official docker registry or even setup your own docker registry. In any case, this will allow you to build your images on one host and use them on another.

For a more detailed usage of `docker-compose`, you should refer to the [official documentation](https://docs.docker.com/compose/compose-file/compose-file-v2/).

Depending on your application requirements and the PHP framework you use, you may want to add more extensions. This can easily be done by modifying the `Dockerfile` used to build our custom PHP image. However, some extensions need extra dependencies to be installed in the container. You should refer to the list of extensions in the [PHP official documentation](http://php.net/manual/en/extensions.alphabetical.php) to review the basic requirements of each extension.