## Introduction

This guide walks you through the process of running an example PHP (PHP-FPM) application on a Kubernetes cluster. PHP-FPM (FastCGI Process Manager) is an alternative PHP FastCGI implementation with some additional features useful for sites of any size, especially busier sites. This guide will use PHP-FPM with Nginx. Doing so will allow us to set up more complex configuration and load balance to different PHP-FPM instances.

Kubernetes is the best way for running application deployments across clusters of hosts. This solution allows you to automate the deployment, the scaling and management of the application containers. To more easily deploy and manage the application containers in a Kubernetes cluster, you can use Helm charts. In the same way that a _Dockerfile_ contains instructions on how to build images in running containers, the charts define how the application will be deployed in Kubernetes. They specify the objects definitions, services, dependencies, number of pods, and so on.

The first step is to obtain the application source code, _Dockerfile_ and the _docker-compose.yml_ file and use them as a starting point for creating a custom Helm chart to automate the application deployment in a Kubernetes cluster. Once the application is deployed and working, this guide also explores how to modify the source code for publishing a new application release and how to perform rolling updates in Kubernetes using the Helm CLI.

## Assumptions and prerequisites

-   You have basic knowledge of how to [build Docker images](https://docs.bitnami.com/tutorials/deploy-custom-nodejs-app-bitnami-containers/#step-3-build-the-docker-image).
-   You have basic knowledge of Helm charts and how to create them.
-   You have a [Docker environment](https://www.docker.com/get-docker) running.
-   You have an account in a container registry (this tutorial assumes that you are using [Docker Hub](https://hub.docker.com/)).
-   You have [Minikube installed](https://docs.bitnami.com/kubernetes/get-started-kubernetes/#option-1-install-minikube) in your local computer.
-   You have a [Kubernetes cluster](https://docs.bitnami.com/kubernetes/get-started-kubernetes/#option-1-create-a-cluster-using-minikube) running.
-   You have the [_kubectl_ command line (_kubectl_ CLI)](https://docs.bitnami.com/kubernetes/get-started-kubernetes/#step-3-install-kubectl-command-line) installed.
-   You have [Helm v3.x](https://docs.bitnami.com/kubernetes/get-started-kubernetes/#step-4-install-helm) installed.

To create your own application in PHP-FPM and deploy it on Kubernetes using Helm, you will typically follow these steps:

-   Step 1: Obtain the application source code
-   Step 2: Build the Docker image
-   Step 3: Publish the Docker image
-   Step 4: Create the Helm Chart
-   Step 5: Deploy the example application in Kubernetes
-   Step 6: Updating the source code and the Helm chart

## Step 1: Obtain the application source code

To begin the process, ensure that you have access to the application source code. The application code for the current example is stored in a Git repository. Follow these steps to obtain the application source code:

-   Clone the repository. This will clone the sample repository and make it the current directory:
    
    ```
    git clone https://github.com/bitnami/tutorials
    cd tutorials/phpfpm-k8s/
    ```
    

In the _app-code_ folder you will see a file named _phpminiadmin.php_. This is a small PHP application for accessing and managing MySQL databases which we will use as an example application. Feel free to use any other PHP application instead.

The steps below must be performed within the _tutorials/phpfpm-k8s/_ folder.

## Step 2: Build the Docker image

The source code already contains the _Dockerfile_ and the _docker-compose.yml_ file needed for this example. Replace the USERNAME placeholder in the _app-code/docker-compose.yml_ file with your Docker ID. When you open the _docker-compose.yml_ file, you can see how the services are defined for MariaDB, Nginx and our PHP-FPM application as well as the environment variables used for the database creation.

-   Build the image using the command below. Remember to replace the USERNAME placeholder with your Docker ID:
    
    ```
    docker build . -t  USERNAME/phpfpm-app:0.1.0
    ```
    
-   Run the _docker-compose up_ command in order to create and start the containers:
    
    ```
    docker-compose -f app-code/docker-compose.yml up
    ```
    
-   Check if the application is running correctly by entering _[http://localhost/phpminiadmin.php](http://localhost/phpminiadmin.php)_ in your default browser.
    
-   To log in to the application, you must connect first to the database. Click the "advanced settings" link and enter the information below. Then, click "Apply".
    
    -   DB user name: _mini_
    -   Password: _mini_
    -   DB name: _mini_
    -   MySQL host: _mariadb_
    -   port: 3306

![Access the database](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-access-db-6c7a4268e88436ab5f0e4d3b574ec8aa.png)

After logging in, you should see the default application page:

![Access the application](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-8-3e7d3e0e583b0ef84c1cffeb44875b9e.png)

-   Check the images that have been created in your local repository by executing the command below:
    
    ```
    docker images | grep -E 'fpm|mariadb|nginx'
    ```
    

![Check application images](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-2-d69dc67d2e6a08d4412791041c00d198.png)

## Step 3: Publish the Docker image

Now that your Docker image is built and contains your application code, you can upload it into a public registry. This tutorial uses [Docker Hub](https://hub.docker.com/), but you can select one of your own choice such as:

-   [Google Container Registry](https://cloud.google.com/container-registry/)
-   [Amazon EC2 Container Registry](https://aws.amazon.com/ecr/)
-   [Azure container Registry](https://azure.microsoft.com/en-us/services/container-registry/)

To upload the image to Docker Hub, follow the steps below:

-   Log in to Docker Hub:
    
    ```
    docker login
    ```
    
-   Push the image to your Docker Hub account. Replace the DOCKER\_USERNAME placeholder with the username of your Docker Hub account and _my-custom-app:latest_ with the name and the version of your Docker image:
    
    ```
    docker push DOCKER_USERNAME/my-custom-app:latest
    ```
    
-   Confirm that you see the image in your Docker Hub repositories dashboard.
    

## Step 4: Create the Helm chart

To create a brand new chart, you just need to run the _helm create_ command. This creates a scaffold with sample files that you can modify to build your custom chart. For the current tutorial, we provide you with a ready-made Helm chart. If you examine the repository you just downloaded, there is a directory named _helm-chart_ that already contains the files you need.

-   Move into the _kubernetes_ directory by executing the command below:
    
    ```
    cd ../kubernetes
    ```
    
-   List the files in the folder and open some of them to understand the structure of a Helm chart:
    
    -   _Chart.yaml_: This file includes the metadata of the Helm chart like the version or the description.
        
    -   _requirements.yaml_: This file specifies the requirements for deploying this chart - in this case, the MariaDB container.
        
    -   _values.yaml_: This file declares variables to be passed into the templates. It is important to replace the USERNAME with your Docker ID, and also to check if the container name and version exist.
        
        ```
        image:
        repository: USERNAME/phpfpm-app
        tag: 0.1.0
        ```
        
        This file also defines the ports of the container images:
        
        ```
        nginxService:
            name: nginx
            type: NodePort
            externalPort: 80
            internalPort: 80
        phpfpmService:
            name: phpfpm
            type: NodePort
            phpfpmPort: 9000
        ```
        
-   Change to the _templates/_ folder and check the files there:
    
    -   _NOTES.txt_: This file contains the output to be printed after installation.
        
    -   _configMap.yaml_: This file stores configuration data. It is used to save the Nginx configuration file:
        
        ```
        config:
        nginx.conf: |-
            server {
            listen 0.0.0.0:80;
            root /app;
            location / {
                index index.html index.php;
            }
            location ~ \.php$ {
                fastcgi_pass phpfpm-php-app-phpfpm:9000;
                fastcgi_index index.php;
                include fastcgi.conf;
            }
            }
        ```
        
    -   _deployment.yaml_: This is the manifest file for creating the Kubernetes deployment. In this example, it includes two containers, one for PHP-FPM and the other form Nginx. This file also configures the _configMap_ for the Nginx configuration file:
        
        ```
        volumes:
           - name: nginx-config
           configMap:
               name: {{ template "fullname" . }}
        ```
        
    -   _service.yaml_ and _nginx-service.yaml_: These are manifests for creating service endpoints for both containers.
        

## Step 5: Deploy the example application in Kubernetes

Before performing the following steps, make sure you have a Kubernetes cluster running, and that Helm and Tiller are installed correctly. For detailed instructions, refer to our [starter tutorial](https://docs.bitnami.com//kubernetes/get-started-kubernetes).

At this point, you have built a Docker image, published it in a container registry and created your custom Helm chart. It's now time to deploy the example PHP application on Kubernetes. Follow these instructions:

-   First, make sure that you are able to connect to your Kubernetes cluster by executing the command below:
    
    ```
    kubectl cluster-info
    ```
    
-   Deploy the Helm chart with the _helm install_ command. This will create three pods within the cluster, one for the Nginx service, another for the MariaDB service, and the third for the PHP-FPM application.
    

The database name, root password, and user credentials have been specified by adding the `--set` options. If you don't add the chart name, one will be assigned automatically. DB\_ROOTPASSWORD, DB\_USERNAME, DB\_USERPASSWORD, and DB\_NAME are placeholders for the database root password, user credentials and database name respectively, remember to replace them with the right values.

```
helm install phpfpm . --set mariadb.mariadbRootPassword=DB_ROOTPASSWORD,mariadb.mariadbUser=DB_USERNAME,mariadb.mariadbPassword=DB_USERPASSWORD,mariadb.mariadbDatabase=DB_NAME
```

Once the chart has been installed, you will see a lot of useful information about the deployment. The application won't be available until database configuration is complete. Follow these instructions to check the database status:

-   Run the _kubectl get pods_ command to get a list of running pods:
    
    ```
    kubectl get pods
    ```
    

![Check pods](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-4-8c55bb55573384b9fdbb0eea4a3d4c0f.png)

-   Check the logs of the application pod. APP-POD-NAME is a placeholder; remember to replace it with the right value.
    
    ```
    kubectl logs -f APP-POD-NAME
    ```
    

![Check pod logs](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-5-f5622731b92eccb5cf9017fc36a6c523.png)

-   To get the application URL, copy and paste the commands shown in the "Notes" section included in the information displayed when you executed the _helm install_ command, append _/phpminiadmin.php_ or the name of your own application, and run the command.

![Get the application URL](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-6-5906fbe6e661fdc4da826d2f8104adfc.png)

If you are using Minikube, you can also check the application service to get the application's URL:

```
minikube service phpfpm-php-app-nginx --url
```

-   Enter the resulting URL in your default browser to access the application.

![Access the application](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-8-3e7d3e0e583b0ef84c1cffeb44875b9e.png)

Congratulations! Your PHP application has been successfully deployed on Kubernetes!

## Step 6: Update the source code and the Helm chart

As a developer, you'll understand that your application may need new features or bug fixes in the future. To release a new Docker image, you only have to perform a few basic steps: change your application source code, rebuild and republish the image in your selected container registry. Once the new image release has been pushed to the registry, you need to update the Helm chart.

Follow these instructions to complete the application update process:

-   In the _docker-compose/app_ directory, change the version number in _version.php_ to 0.1.1. Save the file.
    
-   In the _docker-compose/docker-compose.yml_ file change the version number to 0.1.1 and rebuild the image by running the _docker-compose up_ command.
    
    ```
    docker-compose up
    ```
    

Before performing the next step, make sure that you are logged in Docker Hub. Run the _docker login_ command to access your account (if applicable).

-   Publish the new image following the same steps as in [step 3](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-3-publish-the-docker-image) but using the new version number. Remember to replace the USERNAME placeholder with your Docker ID:
    
    ```
    docker push USERNAME/phpfpm-app:0.1.1
    ```
    
-   Change to the _kubernetes_ directory, where you have the Helm chart files. Edit the _values.yaml_ file to replace the current image tag with the new one:
    

![Update the application version tag](data:image/png;base64,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)

-   Run the _helm upgrade_ command followed by the name of the chart. After that, you will see the information about the new deployment:
    
    ```
    helm upgrade phpfpm .
    ```
    

![Update the Helm chart](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-10-3f149e8f5aff67fd7d3d2448615be3f2.png)

-   See what revisions have been made to the chart by executing the _helm history_ command:
    
    ```
    helm history phpfpm .
    ```
    

![Helm chart revisions](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-11-bec0a82f21cae1902c460ba9f876c82c.png)

-   Enter the application URL from [step 5](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-5-deploy-the-example-application-in-kubernetes) and append _/version.php_ in your default browser. You should see the new welcome message:

![Access the application](https://docs.bitnami.com/tutorials/_next/static/images/k8-php-12-c1bbb523d55ff9b6f3b76b59ec53f679.png)

Follow these steps every time you want to update your Docker image and Helm chart.

## Useful links

To learn more about the topics discussed in this guide, use the links below:

-   [Bitnami GitHub](https://github.com/bitnami)
-   [Minikube](https://github.com/kubernetes/minikube)
-   [Kubernetes](https://kubernetes.io/)
-   [Bitnami development containers](https://bitnami.com/containers)
-   [Build your NodeJs App Docker container from a Bitnami base container](https://docs.bitnami.com/tutorials/deploy-custom-nodejs-app-bitnami-containers/)
-   [Get Started with Kubernetes](https://docs.bitnami.com//kubernetes/get-started-kubernetes/)
-   [Kubernetes Async Watches](https://docs.bitnami.com/tutorials/kubernetes-async-watches)
-   [An example of real Kubernetes](https://docs.bitnami.com/tutorials/an-example-of-real-kubernetes-bitnami)
-   [Check out the Kubernetes service catalog](https://medium.com/bitnami-perspectives/service-catalog-in-kubernetes-78c0736e3910)

## In this tutorial

-   [Introduction](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#introduction)
-   [Assumptions and prerequisites](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#assumptions-and-prerequisites)
-   [Step 1: Obtain the application source code](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-1-obtain-the-application-source-code)
-   [Step 2: Build the Docker image](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-2-build-the-docker-image)
-   [Step 3: Publish the Docker image](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-3-publish-the-docker-image)
-   [Step 4: Create the Helm chart](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-4-create-the-helm-chart)
-   [Step 5: Deploy the example application in Kubernetes](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-5-deploy-the-example-application-in-kubernetes)
-   [Step 6: Update the source code and the Helm chart](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#step-6-update-the-source-code-and-the-helm-chart)
-   [Useful links](https://docs.bitnami.com/tutorials/deploy-php-application-kubernetes-helm/#useful-links)