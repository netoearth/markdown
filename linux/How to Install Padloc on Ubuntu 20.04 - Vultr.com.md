## Introduction

Padloc is a self-hosted password manager with support for both individuals and teams. This tutorial explains how to install Padloc on Ubuntu 20.04. This tutorial will use Docker, docker-compose, and Nginx to secure the configuration.

## Prerequisites

Before you begin these steps, you should:

-   [Deploy an Ubuntu 20.04 server](https://my.vultr.com/deploy/).
-   [Update the server](https://www.vultr.com/docs/update-ubuntu-server-best-practices).
-   [Create a non-root user with sudo privileges](https://www.vultr.com/docs/create-a-sudo-user-on-ubuntu-best-practices/).
-   [Log in to your server](https://www.vultr.com/docs/how-to-access-your-vultr-vps) as a non-root user.
-   [Create an SMTP server](https://www.vultr.com/docs/how-to-set-up-a-mail-server-using-iredmail-on-ubuntu-16-04/).
-   Open port 433 on your [Vultr firewall](https://www.vultr.com/docs/vultr-firewall-quickstart-guide) or your [ufw](https://www.vultr.com/docs/how-to-configure-uncomplicated-firewall-ufw-on-ubuntu-20-04-67187/).

You should also create a DNS A record that points a host name to the IP address of your server. The DNS name is required for Certbot to install a TLS/SSL certificate.

## Installation

### Certbot and Git

1.  Install Git using `apt`.
    
    ```
    $ sudo apt install git
    ```
    
2.  Remove any apt-installed versions of Certbot. It is okay if `apt` reports that none are installed.
    
    ```
    $ sudo apt remove certbot
    ```
    
3.  Ensure that your version of snapd is up to date.
    
    ```
    $ sudo snap install core; sudo snap refresh core
    ```
    
4.  Install Certbot using `snap`.
    
    ```
    $ sudo snap install --classic certbot
    ```
    
5.  Run Certbot and follow the prompts to enter your domain name and redirect all traffic to HTTPS.
    
    ```
    $ sudo certbot certonly --standalone
    ```
    
6.  Take note of your certificate and private key paths when provided. It may be different depending on the domain used.
    
    ```
    Certificate Path: /etc/letsencrypt/live/example.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/example.com/privkey.pem
    ```
    

If you used a different SSL provider, please ensure they are on your server and know their full path. You may put them in the `/etc/nginx/` directory if you wish.

### Docker

1.  Remove any older versions of Docker and the Docker engine.
    
    ```
    $ sudo apt remove docker docker-engine docker.io containerd runc
    ```
    
2.  Install Docker using `snap`.
    
    ```
    $ sudo snap install docker
    ```
    

## Configuration

### Docker Container

1.  Create a directory called `padloc` in your home directory and enter it.
    
    ```
    $ mkdir ~/padloc
    $ cd ~/padloc
    ```
    
2.  Create and open a new `docker-compose.yml` file.
    
    ```
    $ nano docker-compose.yml
    ```
    
3.  Add the following lines to the file. Change `example.com` to your domain name.
    
    ```
    version: "3"
    services:
        server:
            image: padloc/server
            container_name: padloc_server
            expose:
                - 3000
            volumes:
                - ${PL_DB_DIR:-./db}:/data
                - ${PL_ATTACHMENTS_DIR:-./attachments}:/docs
            environment:
                - PL_PWA_URL
                - PL_EMAIL_SERVER
                - PL_EMAIL_PORT
                - PL_EMAIL_USER
                - PL_EMAIL_PASSWORD
                - PL_EMAIL_FROM
        pwa:
            image: padloc/pwa
            container_name: padloc_pwa
            expose:
                - 8080
            volumes:
                - ${PL_PWA_DIR:-./pwa}:/pwa
            environment:
                - PL_SERVER_URL
        nginx:
            image: nginx
            container_name: nginx
            volumes:
                - ./nginx.conf:/etc/nginx/nginx.conf
                - ${PL_SSL_CERT:-./ssl/cert.pem}:/ssl/cert
                - ${PL_SSL_KEY:-./ssl/key.pem}:/ssl/key
            ports:
                - 80:80
                - 443:443
    ```
    
4.  Exit the file using CTRL + X, then press Y, followed by ENTER.
    

### Nginx Configuration File

1.  Create and open a new `nginx.conf` file.
    
    ```
    $ nano nginx.conf
    ```
    
2.  Add the following lines to the file.
    
    ```
    http {
        client_max_body_size 10m;
        server {
            listen 80 default_server;
            listen [::]:80 default_server;
            server_name _;
            return 301 https://$host$request_uri;
        }
    
        server {
            server_name example.com;
            listen 443 ssl http2;
    
            location /server/ {
                proxy_pass http://padloc_server:3000;
                rewrite ^/padloc_server(.*)$ $1 break;
            }
    
            location / {
                proxy_pass http://padloc_pwa:8080;
                rewrite ^/padloc_pwa(.*)$ $1 break;
            }
    
            ssl_certificate /ssl/cert;
            ssl_certificate_key /ssl/key;
        }
    }
    
    events {}
    ```
    
3.  Exit the file using CTRL + X, then press Y, followed by ENTER.
    

### Create the .env File

1.  Create a new file called `.env` in your text editor.
    
    ```
    $ nano .env
    ```
    
2.  Copy and paste the values below into your text editor and change them accordingly for your configuration.
    
    ```
    # URL that points to the web application
    PL_PWA_URL=https://example.com/
    # URL that points to the server instance
    PL_SERVER_URL=https://example.com/server/
    
    # The port the server instance will listen on
    PL_SERVER_PORT=3000
    # The directory where database files will be stored
    PL_DB_DIR=./db
    # The directory where attachment files will be stored
    PL_ATTACHMENTS_DIR=./attachments
    
    # The port the web application will be served from
    PL_PWA_PORT=8080
    # The directory where the static code for the web application will be stored
    PL_PWA_DIR=./pwa
    
    # SMTP host
    PL_EMAIL_SERVER=smtp.example.com
    # SMTP username
    PL_EMAIL_USER=example@example.com
    # SMTP port
    PL_EMAIL_PORT=443
    # SMTP password
    PL_EMAIL_PASSWORD=CHANGE_ME
    # Use TLS when sending emails
    PL_EMAIL_SECURE=false
    
    # The path to your SSL certificate (change example.com)
    PL_SSL_CERT=/etc/letsencrypt/live/example.com/fullchain.pem
    # The path to your SSL private key (change example.com)
    PL_SSL_KEY=/etc/letsencrypt/live/example.com/privkey.pem
    ```
    
3.  Save and exit the text editor by using CTRL + X, then Y, followed by ENTER.
    
4.  Run Padloc by using `docker-compose` in detached mode. This may take a few seconds.
    
    ```
    $ sudo docker-compose up -d
    ```
    
5.  Check that Padloc is running by using `docker`. The status should be `Up`.
    
    ```
    $ sudo docker ps
    STATUS
    Up x seconds/minutes
    ```
    

You have now successfully installed and configured Padloc.

## Finishing Steps

You should now navigate to your Padloc installation and create an account.

```
https://example.com
```

This completes the steps to install Padloc and secure it using an SSL certificate and an Nginx reverse proxy.

### Documentation

-   [Padloc Documentation and Repository](https://github.com/padloc/padloc)
-   [Padloc Website](https://padloc.app/)
-   [Nginx Documentation](https://nginx.org/en/docs/)

## Want to contribute?