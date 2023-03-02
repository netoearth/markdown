In this post I’ll speak in details about how I have managed deploy of a web application on my local [kubernetes](https://kubernetes.io/it/) cluster (minikube).

This can be considered a good starting point to create a development local environment for a web application in a ‘_microservices based_’ context.

This specific case study is a Symfony project, Sylius based, but it doesn’t metter. Code development is not target of this post, i have used only skeleton provided by the platform at the time of initial installation.

You can see (and clone) a repository with this case study from follow link:

## **Schema**

![](https://miro.medium.com/max/700/1*6eNZC00BnMLQX2lI1-amRQ.png)

kubernetes node schema for nginx, php-fpm, mysql stack

Image above shows kubernetes node structure.

Pod on the left contains two containers: one about **nginx** and another about **php-fpm**.

Containers inside this pod shares a [persistent volume claim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) to host the web application source code. Another volume “hosts” the nginx configuration through the use of a [configMap](https://kubernetes.io/docs/concepts/configuration/configmap/).

The second pod is used to host **mysql** service. Here too there is a persistent volume claim necessary in this case for database storage.

## **Let’s start**

Before proceeding with the application deployment, of course, you need to have a working cluster node.

In order to work locally with k8s I have chosen to use **minikube** which is a “_light_” version of kubernetes that allows to run a single node cluster in a local machine.

```
minikube starteval $(minikube docker-env)
```

If you feel confident to works with **kubectl** from command line (you should be), that’s all you need to start with to follow this case study.

## **Nginx and PHP**

Repository that I created on github contains a [bash script to start minikube node and deploy application in cluster](https://github.com/sergioska/sylius-as-a-microservice/blob/master/kubernetes/start.sh).

Anyway I’ll show every command, script and yaml file to explain step by step this case study.

A detail that I have omitted so far concerns the fact that in the initial situation the web application was “dockerized”

In the docker-compose about nginx I used an official image from the docker registry while for php-fpm I created the following Dockerfile:

php-fpm Dockerfile

I therefore used the same docker images to create nginx / php-fpm deployment manifest.

So, I have builded image from the dockerfile above with command:

```
docker build -t v4 .
```

But we haven’t fineshed yet …

In addition to the images of the containers and before launching the deployment, we need to think about persistent volumes.

For this pod we will need a persistent volume claim for the sources of the project. Something like this:

Also we need to define a configMap to have service configuration decoupled from nginx and php-fpm images

configMap.yaml about nginx/php-fpm configuration

Here we are now.

At this point we have all ‘ingredients’ to write deployment manifest about ‘_www service_’

nginx/php-fpm kubernetes deployment manifest

Ok ‘_all together now_’ …

```
# 1. apply configMap for nginx configurationkubectl apply -f configMap.yaml# 2. apply persistent volume shared between nginx and php-fpm podskubectl apply -f volumes/# 3. apply deploymentkubectl apply -f pods/nginx-php-fpm-deployment.yaml
```

## MySQL with Helm

Instead of writing a deployment manifest like I did for the above services, for mysql I used [Helm](https://helm.sh/).

Helm is a tool that helps manage applications based on kubernetes.

Here are the steps that I followed to deploy a pod that contains a mysql container inside it using Helm.

First I searched mysql among the helm hub repositories

```
helm search hub mysql
```

so i installed stable/mysql with this command:

```
helm install stable-mysql --set mysqlRootPassword=root,mysqlUser=root,mysqlPassword=root,mysqlDatabase=sylius_dev,persistence.existingClaim=mysql-pvc stable/mysql
```

Now, it need only two things to finish and get project worked fine.

-   put source code on web server
-   expose web server to outside via port-forwarding

## Source Code

Before we can see own web application runs, we need to copy files from local machine to kubernetes node.

This can be do in following way:

```
# ask about web pod namekubectl get pods# copy files from local environment to kubernetes clusterkubectl cp ./ default/<pod_name>:/srv/app/./
```

Probably, use ‘_kubectl cp’_ command is not the best way to do this task; there are more clever way/tools to do that. For example Ksync.

I’ll speak about Ksync in another post (i hope).

## Port Forward

To expose application to the web, it need to be accessible from outside.

To do that it need to use port-forwarding throw kubectl of course

```
sudo kubectl port-forward <pod_name> 80:80
```

Note that sudo is mandatory if you try to forward 80 port.

That’s all at this moment, but as soon as possible i’ll post some enhancements about this target.