When running a web application, you normally want to make the most out of your resources without having to convert your software to use multithreading or complex event loops. Docker, however, does provide a simple way for you to load balance your application internally to make the most out of server resources. This article will show you how to use Nginx to load balance your web application using Docker on CentOS.

## Step 1: Create a simple application

We will be using rust to build this simple application. Assuming that you have rust installed, run `cargo new webapp â€“bin`. Upon success, you will see a directory called `webapp`. Inside of `webapp`, you will see a file called `Cargo.toml`. Append the following lines to it:

```
[dependencies.iron]

version = "*"
```

Next, inside the `src/main.rs` file, remove everything and populate it with the following:

```
extern crate iron;



use iron::prelude::*;

use iron::status;



fn main() {

    Iron::new(|_: &mut Request| {

        Ok(Response::with((status::Ok, "Hello Vultr :)")))

    }).http("0.0.0.0:3000").unwrap();

}
```

Note: Do not change the IP within the application. This is configured so that Docker can listen to your application.

Once you have finished, compile the application by executing `cargo build â€“release`. Depending on your server, it may take a few minutes. If there are no errors, test the application by following these steps:

-   Run `target/release/webapp`.
    
-   Navigate to `http://0.0.0.0:3000/` in your browser. Replace `0.0.0.0` with the IP address of your server.
    

If everything worked properly, you will see "Hello Vultr :)" on the page.

## Step 2: Create Docker containers

Create a `Dockerfile` and populate it with the following:

```
FROM centos:latest

MAINTAINER User <user@localhost>

RUN yum update -y

COPY ./webapp/target/release/webapp /opt/

EXPOSE 3000

WORKDIR /opt

CMD ./webapp
```

Save the file. Then create a file called `deploy.sh` and populate it with the following:

```
DEFAULT_PORT=45710

APP_PORT=3000

DEPLOY=5

NAME="webapp"

docker build -t webapp:example . 



for ((i=0; i<DEPLOY; i++)); do

        docker kill $NAME$i ; docker rm $NAME$i

        docker run --name $NAME$i -p 127.0.0.1:$(((i * 1000) + DEFAULT_PORT)):$APP_PORT -d webapp:example

done
```

When you run this script, it will build the image and deploy the container based on the amount you have set (default is 5). If the container exists, it will kill and remove it from registry before it is deployed again.

## Step 3: Configure Nginx

Now, create an Nginx configuration file and populate it with the following:

```
upstream application {

    server localhost:45710;

    server localhost:46710;

    server localhost:47710;

    server localhost:48710;

    server localhost:49710;

}



server {

    listen 0.0.0.0:80;    

    location / {

    expires 1w;

        proxy_pass http://application;

        proxy_redirect off;

        proxy_http_version 1.1;

        proxy_set_header X-Forwarded-Host $host;

        proxy_set_header Host $host;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Real-IP $remote_addr;

    }

}
```

Replace `0.0.0.0` with the IP address of your server.

Restart Nginx by doing `systemctl restart nginx`. Resolve any errors, then proceed to the next step.

## Step 4: Deploy application

Deploy the application by running `bash ./deploy.sh`.

You can check the status of your application with `docker ps` - there will be 5 images created that start with `webapp`. Now, navigate to `http://0.0.0.0:3000/` in your browser, you will see the "Hello, Vultr :)" message again.

So, what difference does this make, exactly?

If you run a benchmark test against the load balancer configuration, you would notice that more of your server resources are being used, which is what you would want, especially if your application is built in languages like Node where it would normally be single threaded. If you ever need to upgrade your application, you can do so and rerun the `deploy.sh` to rebuild the image and deploy your containers.