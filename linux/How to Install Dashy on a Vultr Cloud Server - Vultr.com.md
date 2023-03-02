Dashy is a free self-hosted dashboard that lets you organize and keep track of your favorite applications or services. It's highly customizable and easy to deploy with extended functionality, including status checks, dynamic widgets, built-in authentication, among others. In this article, you will install Dashy on a Vultr cloud server running Docker.

## Prerequisites

-   [Deploy Docker with Vultrâ€™s One-Click App](https://www.vultr.com/docs/one-click-docker)
    
-   [SSH and Login to the server](https://www.vultr.com/docs/how-to-use-ssh-with-vultr-servers)
    
-   [Install Nginx](https://www.vultr.com/docs/install-a-lemp-stack-on-ubuntu-20-04-lts)
    

## Installation

Download the latest Dashy image from Docker Hub.

```
# docker pull lissy93/dashy:latest
```

Create a new Dashy configuration file, and sub-directory in the `/opt` directory.

```
# mkdir /opt/appdata/dashy



# touch /opt/appdata/dashy/dashy-config.yml
```

Now, create the Dashy Container.

```
docker run -d -p 8080:80 --name Dashy --restart=always -v /opt/appdata/dashy/dashy-config.yml:/app/public/conf.yml lissy93/dashy:latest
```

Below is what each command argument does:

-   `-d` Detached mode, app runs in the background.
    
-   `-p` The port Dashy should be mapped to in the format \[host machine port\]:\[container port\]
    
-   `--name` The Docker ID, can be used to start, check logs, and stop Dashy.
    
-   `--restart` Start the container when Docker starts
    
-   `--v` Specify a container files volume on the host system (create a link to the actual Docker files).
    
-   `lissy93/dashy:latest` Install the latest Dashy release
    

Start Dashy.

```
# docker start Dashy
```

View Docker processes, and verify that Dashy is running.

```
 # docker ps
```

Output:

```
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS                 PORTS                                   NAMES

89db3c6d51cf   lissy93/dashy:latest   "/sbin/tini -- yarn â€¦"   2 hours ago   Up 2 hours (healthy)   0.0.0.0:8080->80/tcp, :::8080->80/tcp   Dashy
```

## Configure Nginx as a Proxy Server

Create a new Nginx configuration file.

```
# nano /etc/nginx/conf.d/dashy.conf
```

Paste the following contents:

```
server {



listen 80;

server_name  dashboard.example.com;



location /{

proxy_set_header Host $host;

proxy_set_header X-Real-IP $remote_addr;

proxy_pass http://localhost:8080;

}

}
```

Save and close the file.

Check the Nginx configuration for errors.

```
# nginx -t
```

Restart Nginx.

```
# systemctl restart nginx
```

## Setup Firewall

Allow Nginx to serve files on HTTP Port 80.

```
# ufw allow 80/tcp
```

If you plan to install a free let's encrypt certificate, allow HTTPS port 443.

```
# ufw allow 443/tcp
```

Restart Firewall.

```
# ufw reload
```

## Configure Dashy

Open and edit your Dashy configuration file.

```
# nano /opt/appdata/dashy/dashy-config.yml
```

Then, paste the following contents to customize your dashboard.

```
pageInfo:

  title: My Dashboard

  navLinks:

        - title: Home

    path: /

  - title: About

    path: /about

  - title: Source Code

    path: https://github.com/Lissy93/dashy

appConfig:

  theme: dracula

  fontAwesomeKey: 13014ae648

sections:

- name: Productivity

  items:

  - title: ProtonMail

    description: Secure Encrypted Email

    icon: favicon

    url: https://mail.protonmail.com/

  - title: AnonAddy

    description: Mail aliasing and forwarding service

    icon: favicon

    url: https://app.anonaddy.com/

  - title: LessPass

    description: Deterministic password generator

   icon: favicon

   url: https://lesspass.com/

 - title: EteSync

   description: Calendar, contacts and tasks

   icon: favicon

   url: https://client.etesync.com/

  - title: Live Coin Watch

   description: Real-time crypto prices and read-only portfolio

    icon: favicon

    url: https://www.livecoinwatch.com/

  - title: Standard Notes

    description: Encrypted productivity suit

   icon: favicon

   url: https://app.standardnotes.org/

- name: Social

  items:

  - title: Discord

    icon: fab fa-discord

    url: https://discord.com/channels/

   - title: Mastodon

    icon: fab fa-mastodon

    url: https://mastodon.social/

  - title: Reddit

    icon: fab fa-reddit

    url: https://www.reddit.com/

  - title: Twitter

    icon: fab fa-twitter

    url: https://twitter.com/

  - title: YouTube

    icon: fab fa-youtube

    url: https://youtube.com/

  - title: Instagram

    icon: fab fa-instagram

    url: https://www.instagram.com

- name: Coding

  displayData:

    rows: 2

   items:

         - title: Vultr

    description: Affordable, High perfomance cloud services

    url: https://vultr.com/

    icon: https://www.vultr.com/favicon/android-chrome-512x512.png   

  - title: StackOverflow

     icon: fab fa-stack-overflow

     url: https://stackoverflow.com/

  - title: GitHub

    icon: fab fa-github

    url: https://github.com/

  - title: Grep App

    icon: fas fa-slash

    description: Allows you to search the contents of files within GitHub repos, with a RegEx option too

    url: https://grep.app/

  - title: Hoppscotch

    icon: fas fa-dice-d6

    description: API development tool

    url: https://hoppscotch.io/

  - title: Regexr

    icon: fas fa-code

    description: App for creating, testing and understanding regular expressions

    url: https://regexr.com/

   - title: Tmate

    icon: fas fa-terminal

    description: Instant TMUX-based terminal sharing

    url: https://tmate.io/

  - title: Debian Handbook

    icon: fab fa-linux

    url: https://debian-handbook.info/browse/stable/

  - title: Crontab Guru

    description: Editor for cron schedule expressions

    icon: fas fa-pen-square

    url: https://crontab.guru/

  - title: CyberChef

    description: All in one encryption, encoding, hashes etc

    icon: fas fa-user-astronaut

    url: https://gchq.github.io/CyberChef/

  - title: SSL Labs Tester

    description: Check SSL Certificates

    icon: fab fa-expeditedssl

    url: https://www.ssllabs.com/ssltest/

  - title: JSON Formatter

    description: JSON validator, beautifier and convertor

    url: https://jsonformatter.org/

    icon: fas fa-brackets-curly

  - title: JSFiddle

    url: https://jsfiddle.net/

    icon: fab fa-jsfiddle

  - title: HealthChecks.io

    description: Off-site cron job and service monitoring

    icon: fal fa-monitor-heart-rate

    url: https://healthchecks.io/

 - name: Utilities

  displayData:

    itemSize: small

    cols: 2

  items:

  - title: Chroniker Timer

    description: A productivity timer

    url: https://chroniker.co/

  - title: CopyChar

    description: easily search and copy any symbol

    url: https://copychar.cc/

  - title: DeepL

    description: Fast, accurate, secure online multi-language translator

    url: https://www.deepl.com/translator

  - title: Font Generator

    description: Use ASCII characters for different plaintext fonts

   url: https://www.fontgeneratoronline.com/

  - title: JSON Formatter

    description: CSV, JSON, YAML, XML and more formatting tools

    url: https://jsonformatter.org/

  - title: Time.Is

    description: Time.Is

    url: https://time.is/

  - title: Torrent Parts

    description: Inspect and edit what's in your Torrent file or Magnet link

    url: https://torrent.parts/

  - title: Photo College

     url: https://www.befunky.com/create/collage/

  - title: MD5 Hash

    url: http://www.miraclesalad.com/webtools/md5.php

  - title: GitHub Stars Over Time

    url: https://seladb.github.io/StarTrack-js/

  - title: Text & List Tools

    url: https://pinetools.com/c-text-lists/

  - title: Online Text Tools

    description: More text tools, allowing for RegEx actions

    url: https://onlinetexttools.com/

  - title: Conversao

    description: Instantly convert a unit to all others

    url: https://conversao.net/eng/

   - title: Omni Calculators

    description: A collection of thousands of calculators

    url: https://www.omnicalculator.com/

  - title: Dr Meme

    description: Meme Generator

    url: https://www.drmemes.com/

  - title: ASCII Tree Generator

    description: For displaying directory structure

    url: https://ascii-tree-generator.com/

  - title: ASCII Diagram Drawer

    description: For using box characters to draw boxes

    url: For using box characters to draw boxes

  - title: Bullet Converter

    description: Small script I use

    url: https://listed.to/p/LxO8yHsB9E#bc

  - title: Lenny Face Maker

   description: Easily generate ASCII Lenny emojis

   url: https://www.fontspace.com/lenny-face
```

Save and close the file.

For more customization templates, refer to the [Dashy documentation.](https://dashy.to/docs/)

Now rebuild the Dashy container using the following command:

```
# docker exec -it Dashy yarn build
```

Once complete, Visit your subdomain or Server IP through a web browser.

[http://dashboard.example.com](http://dashboard.example.com/)

![Dashy Dashboard](https://www.vultr.com/vultr-docs-graphics/6975/W20lnat.png)

Your current dashboard options will be displayed. You can fine-tune your dashboard by editing the Dashy configuration file.

In case Dashy throws an error after rebuilding the configuration, stop and start the container with the following commands:

Stop Dashy

```
# docker stop Dashy
```

Start Dashy

```
# docker start Dashy
```

## Conclusion

Congratulations, you have installed Dashy on a Vultr cloud server running Docker and set up a subdomain through Nginx as the reverse proxy server. For more information on configuring your dashboard, visit the Dashy documentation page.