If you’re running Docker micro-services in your Infrastructure, you may be interested in building an internal Private Docker Registry to host Docker images. This could be for security reasons or to have faster builds. This article will show you the steps you need to Setup Docker Private Registry on Ubuntu 22.04/20.04/18.04.

[![docker local private registry](https://computingforgeeks.com/wp-content/uploads/2018/08/docker-local-private-registry.png?ezimgfmt=rs%3Adevice%2Frscb23-1 "How to Setup Docker Private Registry on Ubuntu 18.04 / Ubuntu 16.04 with Letsencrypt SSL 1")](https://computingforgeeks.com/wp-content/uploads/2018/08/docker-local-private-registry.png)

The assumption here is that you have running Docker Engine hosts. If you don’t have docker or Kubernetes clusters, here are the guides to get you started:

-   [Install Docker CE on a Linux System](https://computingforgeeks.com/install-docker-ce-on-linux-systems/)

We also have an article on how to [Install and Configure Docker Registry on CentOS 7](https://computingforgeeks.com/install-and-configure-docker-registry-on-centos-7/), check it out if you’re interested in setting up Docker Registry on CentOS 7.

Let’s start to build Private Registry for Docker images. First, install Docker Engine on the host to act as a registry.

### Step 1: Install Docker Runtime Engine

Update the `apt` package index:

```
sudo apt update
```

Install packages to allow apt to use a repository over HTTPS:

```
sudo apt -y install apt-transport-https ca-certificates curl software-properties-common
```

Add Docker’s official GPG key:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add official Docker stable repository:

**_Ubuntu 22.04:_**

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

**_Ubuntu 20.04|18.04:_**

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Install `docker-ce` pakage:

```
sudo apt update
sudo apt -y install docker-ce docker-ce-cli containerd.io
```

If you would like to use Docker as a non-root user, you should now consider adding your user to the “**docker”** group:

```
sudo usermod -aG docker $USER
newgrp docker
```

Run the command below to see a version of docker installed.

```
$ docker --version
Docker version 20.10.10, build b485636
```

Check status, it should be in a running state:

```
$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-04-29 17:46:22 UTC; 34s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 20303 (dockerd)
      Tasks: 8
     Memory: 52.9M
     CGroup: /system.slice/docker.service
             └─20303 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.070599784Z" level=warning msg="Your kernel does not support CPU realtime scheduler"
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.070755456Z" level=warning msg="Your kernel does not support cgroup blkio weight"
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.071118279Z" level=warning msg="Your kernel does not support cgroup blkio weight_device"
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.071693508Z" level=info msg="Loading containers: start."
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.249710604Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/1>
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.400056246Z" level=info msg="Loading containers: done."
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.498785594Z" level=info msg="Docker daemon" commit=8728dd2 graphdriver(s)=overlay2 version=20.10.6
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.499364281Z" level=info msg="Daemon has completed initialization"
Apr 29 17:46:22 ubuntu systemd[1]: Started Docker Application Container Engine.
Apr 29 17:46:22 ubuntu dockerd[20303]: time="2021-04-29T17:46:22.566845201Z" level=info msg="API listen on /run/docker.sock"
```

### Step 2: Get Let’s Encrypt SSL Certificates

In this Docker Registry setup, we will use Let’s Encrypt SSL Certificates which expire every 90 days and you’ll need to renew.

Install `certbot` tool that will be used to request for Let’s Encrypt certificate.

```
sudo apt update
sudo apt install certbot -y
```

Request for SSL certificate:

```
export DOMAIN="registry.domain.com"
export EMAIL="email@domain.com"
certbot certonly --standalone -d $DOMAIN --preferred-challenges http --agree-tos -n -m $EMAIL --keep-until-expiring
```

Expected output:

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for registry.computingforgeeks.com
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/registry.computingforgeeks.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/registry.computingforgeeks.com/privkey.pem
   Your cert will expire on 2021-07-28. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Your certs will be saved under `/etc/letsencrypt/live/`

```
/etc/letsencrypt/live/registry.domain.com/fullchain.pem
/etc/letsencrypt/live/registry.domain.com/privkey.pem
```

Where:

**fullchain.pem** ⇒ combined file `cert.pem` and `chain.pem`  
**chain.pem** ⇒ intermediate certificate  
**cert.pem** ⇒ SSL Server cert(includes public-key)  
**privkey.pem** ⇒ private-key file

### Step 3: Configure and Start Docker Registry container

You can either run docker registry with SSL or without. But first, create a directory that will hold Docker registry images:

```
sudo mkdir /var/lib/docker/registry
```

#### Run Local Docker registry without SSL

```
docker run -d -p 5000:5000 \
  --name docker-registry \
  --restart=always \
  -v /var/lib/docker/registry:/var/lib/registry \
  registry:2
```

#### Run Local Docker registry with SSL

Create directory and place certs on the host:

```
export DOMAIN="registry.domain.com"
mkdir /certs
sudo cp /etc/letsencrypt/live/$DOMAIN/fullchain.pem /certs/fullchain.pem
sudo cp /etc/letsencrypt/live/$DOMAIN/privkey.pem /certs/privkey.pem
sudo cp /etc/letsencrypt/live/$DOMAIN/cert.pem /certs/cert.pem
```

Create a Docker Registry container:

```
docker run -d --name docker-registry --restart=always \
-p 5000:5000 \
-e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.pem  \
-e REGISTRY_HTTP_TLS_KEY=/certs/privkey.pem \
-v /certs:/certs \
-v /var/lib/docker/registry:/var/lib/registry \
registry:2
```

Replace `registry.domain.com` with your registry subdomain name.

It will download registry:2 docker image if it doesn’t exist and create a container

```
Unable to find image 'registry:2' locally
2: Pulling from library/registry
ddad3d7c1e96: Pull complete
6eda6749503f: Pull complete
363ab70c2143: Pull complete
5b94580856e6: Pull complete
12008541203a: Pull complete
Digest: sha256:bac2d7050dc4826516650267fe7dc6627e9e11ad653daca0641437abdf18df27
Status: Downloaded newer image for registry:2
3c686b882def8d03abe7a6b6dce6d5cd6ae870fb0260c820e4a08ac9e560c27d
```

Check container state:

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
5f2ddfba15df        registry:2          "/entrypoint.sh /etc…"   2 minutes ago       Up 2 minutes        0.0.0.0:5000->5000/tcp   docker-registry
```

To push images to Registry Container server, set like below:

```
$ curl https://$DOMAIN:5000/v2/_catalog
{"repositories":[]}
```

Let’s download two docker images and push them to this local repository:

```
docker pull alpine
docker pull ubuntu
```

Then set a tag and push images to our Registry:

```
# Tag images
$ docker tag ubuntu $DOMAIN:5000/ubuntu:v1
$ docker tag alpine $DOMAIN:5000/alpine:v1

# Push images to private registry
$ docker push $DOMAIN:5000/alpine:v1
The push refers to repository [registry.computingforgeeks.com:5000/alpine]
b2d5eeeaba3a: Pushed
v1: digest: sha256:def822f9851ca422481ec6fee59a9966f12b351c62ccb9aca841526ffaa9f748 size: 528

$ docker push $DOMAIN:5000/ubuntu:v1
The push refers to repository [registry.computingforgeeks.com:5000/ubuntu]
268a067217b5: Pushed 
c01d74f99de4: Pushed 
ccd4d61916aa: Pushed 
8f2b771487e9: Pushed 
f49017d4d5ce: Pushed 
v1: digest: sha256:958eaeb7e33e6c4f68f7fef69b35ca178c7f5fb0dd40db7b44a8b9eb692b9bc5 size: 1357
```

### Step 4: Pulling image from Private Docker Registry

On Docker hosts, you can pull the image using:

```
$ docker pull registry.computingforgeeks.com:5000/ubuntu:v1
v1: Pulling from ubuntu
c64513b74145: Pull complete 
01b8b12bad90: Pull complete 
c5d85cf7a05f: Pull complete 
b6b268720157: Pull complete 
e12192999ff1: Pull complete 
Digest: sha256:958eaeb7e33e6c4f68f7fef69b35ca178c7f5fb0dd40db7b44a8b9eb692b9bc5
Status: Downloaded newer image for registry.computingforgeeks.com:5000/ubuntu:v1
```

The image should now be visible:

```
$ docker images
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
registry.computingforgeeks.com:5000/ubuntu   v1                  735f80812f90        8 days ago          83.5MB
busybox                                      latest              22c2dd5ee85d        2 weeks ago         1.16MB
gliderlabs/herokuish                         latest              ce23b0b51ca7        2 months ago        914MB
gliderlabs/herokuish                         v0.4.2              ce23b0b51ca7        2 months ago        914MB
postgres                                     10.2                c12289de6f88        5 months ago        288MB
dokkupaas/s3backup                           0.8.0               7c096c5ae8fd        11 months ago       98.5MB
dokkupaas/wait                               0.2                 f191c874dade        23 months ago       9.08MB
svendowideit/ambassador                      latest              8f4da6f7d98a        2 years ago         8.03MB
```

## Start Registry with Authentication support

Except for registries running on secure local networks, registries should always implement access restrictions. The simplest way to achieve access restriction is through basic authentication:

Create a password file with one entry for the user, `dockadmin` with password `registrypassword`:

```
$ docker run \
--entrypoint htpasswd \
registry:2 -Bbn dockadmin registrypassword > ~/.htpasswd

$ cat ~/.htpasswd
dockadmin:$2y$05$WN6moKCdUB1i3lFloCPR/Oaw5b4aVcphXSUlEHuRMD2knofj1FuBW
```

Remove current docker registry.

```
docker rm -f docker-registry
```

Start the registry with basic authentication.

```
$ docker run -d --name docker-registry --restart=always \
-p 5000:5000 \
-v ~/.htpasswd:/auth_htpasswd \
-e "REGISTRY_AUTH=htpasswd" \
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth_htpasswd \
-e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.pem \
-e REGISTRY_HTTP_TLS_KEY=/certs/privkey.pem \
-v /certs:/certs \
-v /var/lib/docker/registry:/var/lib/registry \
registry:2
```

Try to pull an image from the registry, or push an image to the registry. These commands fail.

```
$ docker pull registry.computingforgeeks.com:5000/ubuntu:v1
Error response from daemon: Get https://registry.computingforgeeks.com:5000/v2/ubuntu/manifests/v1: no basic auth credentials
```

You need to log in to the registry.

```
$ docker login registry.computingforgeeks.com:5000
Username: dockadmin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

# cat /root/.docker/config.json
{
"auths": {
"registry.computingforgeeks.com:5000": {
"auth": "ZG9ja2FkbWluOnJlZ2lzdHJ5cGFzc3dvcmQ="
}
},
"HttpHeaders": {
"User-Agent": "Docker-Client/18.06.0-ce (linux)"
}
}
```

You should now be able to download and push images to the repository:

```
$ docker tag alpine registry.computingforgeeks.com:5000/alpine_local
$ docker push registry.computingforgeeks.com:5000/alpine_local
The push refers to repository [registry.computingforgeeks.com:5000/alpine_local]
73046094a9b8: Pushed 
latest: digest: sha256:0873c923e00e0fd2ba78041bfb64a105e1ecb7678916d1f7776311e45bf5634b size: 528
```

## Stop a local Docker registry

To stop the registry, use the same docker container stop command as with any other container.

```
docker container stop docker-registry
```

To remove the container, use docker container rm.

```
docker container stop registry
docker container rm -v registry
docker container rm -f -v registry # Force remove running
```

## Conclusion

You now have a working Local Docker registry, you’re free to choose the deployment that suits your need; **registry without SSL**, **registry with SSL but now authentication** or **Registry with SSL and Basic Authentication**. I recommend setting up **secure** private Docker registry for production environments – This will have both SSL and Authentication.

Similar articles:

-   [Install Harbor Container Image Registry on CentOS / Debian / Ubuntu](https://computingforgeeks.com/how-to-install-harbor-docker-image-registry-on-centos-debian-ubuntu/)
-   [Install Harbor Image Registry on Kubernetes / OpenShift with Helm Chart](https://computingforgeeks.com/install-harbor-image-registry-on-kubernetes-openshift-with-helm-chart/)
-   [Setup Red Hat Quay Registry on CentOS / RHEL / Ubuntu](https://computingforgeeks.com/how-to-setup-red-hat-quay-registry-on-centos-rhel/)