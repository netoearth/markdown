## Introduction

Django is a powerful web framework for Python that allows developers to build complex web applications in an organized manner. It provides many features that speed up the development process, such as a template engine, an object-relational mapper, and an authentication system. It also contains many built-in tools, such as an automated admin interface, caching, RSS and Atom feed, and so on. With Django, developers can easily and quickly create powerful web applications with minimal effort. It includes a web server for testing and development, but for deploying it to production, you must use other solutions such as Gunicorn.

This article demonstrates the steps to use the Vultr Object Storage to host the static files, configure the Vultr Managed PostgreSQL Database as the database backend, containerize the Django application and deploy the Django application on the Vultr Kubernetes Engine.

## Prerequisites

Before you begin, you should:

-   Deploy a [Vultr Kubernetes Engine](https://www.vultr.com/kubernetes/) cluster.
    
-   Deploy a [Vultr Managed Database](https://www.vultr.com/products/managed-databases/) for PostgreSQL cluster.
    
-   Deploy an [Object Storage](https://www.vultr.com/products/object-storage/) resource.
    
-   Have access to the DNS settings of a domain name. This article uses **django.example.com** for demonstration.
    
-   Deploy an [Ubuntu 22.04](https://www.vultr.com/servers/ubuntu/) server to use as a management workstation. On the management workstation:
    
    -   Install [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux).
        
    -   Download your VKE configuration and [configure Kubectl](https://www.vultr.com/docs/vultr-kubernetes-engine/#How_to_Manage_a_VKE_Cluster).
        

> Ensure that you provision all the services in the same region for low latency between the services.

## Initialize the Django Project

A Django project contains all the logic of a single project divided into partial chunks called Django applications. This section demonstrates the steps to initialize a new Django project using the `django-admin` command. You can skip these steps and move to the next steps while working on an existing project.

Create a new directory named `django-demo`.

```
# mkdir ~/django-demo

# cd ~/django-demo
```

Install the `virtualenv` package.

```
# pip install virtualenv
```

The above command installed the `virtualenv` package. It is a Python library that allows you to create isolated environments for your Python projects. It is commonly used to manage package dependencies and avoid conflicting dependencies between different projects.

Create a new virtual environment.

```
# virtualenv venv
```

The above command creates a new directory named `venv` in the working directory that contains all the Python virtual environment files.

Activate the virtual environment.

```
# source venv/bin/activate
```

The above command executes the `activate` binary file that manipulates the `$PATH` variable so that the `python` and `pip` commands refer to virtual environment binary files instead of the global binary files.

Install the `django` package.

```
(venv) # pip install django
```

Create a new Django project.

```
(venv) # django-admin startproject PROJECT_NAME
```

Append the hosts in the `ALLOWED_HOSTS` list.

```
(venv) # cd PROJECT_NAME

(venv) # nano PROJECT_NAME/settings.py
```

Find the variable and append your hosts to the list.

```
ALLOWED_HOSTS = ['PUBLIC_IP', 'django.example.com']
```

The above statement instructs Django to allow connections on the `PUBLIC_IP` and the `django.example.com` subdomain.

Save the file and close the file editor using CTRL+X then ENTER.

Initialize the database.

```
(venv) # python manage.py migrate

(venv) # python manage.py createsuperuser
```

Disable the firewall.

```
(venv) # ufw disable
```

The above command disables the firewall to allow inbound network connections on any port.

Start to Django Server.

```
(venv) # cd PROJECT_NAME

(venv) # python manage.py runserver 0.0.0.0:8000
```

The above commands spawn an instance of the inbuilt Django web server that listens on port `8000`. You can open the interface on your web browser by opening `http://PUBLIC_IP:8000`. After confirming the access, stop the server using CTRL+C to follow along with the next steps.

## Configure Vultr Object Storage as Static File Backend

Django does not serve static files without debugging mode, which requires you to host them separately. You can use an object storage service to host the static files as a workaround. This section demonstrates the steps to configure the Vultr Object Storage service as the backend for hosting and serving the static files.

Install the required Python packages.

```
(venv) # pip install django-storages boto3
```

The above command installs the `django-storages` and the `boto3` packages in the virtual environment, allowing you to integrate the Vultr Object Storage with your Django project.

Edit the `settings.py` file.

```
(venv) # nano PROJECT_NAME/settings.py
```

Add the following line in the file header.

```
import os
```

The above statement imports the `os` library, a built-in library that provides a way of using operating system dependent functionality.

Add the following contents under the static file variables.

```
DEFAULT_FILE_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"

STATICFILES_STORAGE = "storages.backends.s3boto3.S3StaticStorage"



AWS_S3_REGION_NAME = os.environ.get("OBJECT_STORAGE_REGION")

AWS_S3_ENDPOINT_URL = f"https://{AWS_S3_REGION_NAME}.vultrobjects.com"

AWS_S3_USE_SSL = True

AWS_STORAGE_BUCKET_NAME = os.environ.get("OBJECT_STORAGE_BUCKET_NAME")

AWS_ACCESS_KEY_ID = os.environ.get("OBJECT_STORAGE_ACCESS_KEY")

AWS_SECRET_ACCESS_KEY = os.environ.get("OBJECT_STORAGE_SECRET_KEY")

AWS_DEFAULT_ACL="public-read"
```

The above statements define the settings for the static file backend and use the `os.environ()` function to fetch the credentials from the environment variables. Save the file and close the file editor using CTRL+X then ENTER.

Create a new file named `.env`.

```
(venv) # nano .env
```

Add the following contents to the file.

```
OBJECT_STORAGE_REGION=

OBJECT_STORAGE_BUCKET_NAME=

OBJECT_STORAGE_SECRET_KEY=

OBJECT_STORAGE_ACCESS_KEY=
```

Populate the values using the information on the resource page. Save the file and close the file editor using CTRL+X then ENTER.

Export the environment variables.

```
(venv) # export $(xargs < .env)
```

The above command reads the `.env` file and exports it to the host machine's environment variables.

Apply the settings.

```
(venv) # python manage.py collectstatic
```

The above command collects all the static files and uploads it onto the specified Vultr Object Storage bucket. The `django-storages` package ensures that your Django project uses the object storage URLs to serve the static files.

Verify the changes.

```
(venv) # python manage.py runserver 0.0.0.0:8000
```

The above command spawns a Django web server. You can verify the changes by opening `http://PUBLIC_IP:8000/admin` in your web browser and viewing the source code to check the resource URLs. Stop the server using CTRL+C to follow along with the next steps.

## Configure Vultr Managed Database as Database Backend

By default, Django uses a SQL lite database to store all the models and data. This section demonstrates the steps to switch the database engine to PostgreSQL and use the Vultr Managed Database PostgreSQL Cluster as the database backend. You need to make these changes even if your Django project already uses PostgreSQL, as you need to use environment variables for fetching the database credentials.

Install the required Python packages.

```
(venv) # pip install psycopg2-binary
```

The above command installs the `psycopg2-binary` package in the virtual environment, allowing you to integrate the Vultr Managed PostgreSQL Database cluster with your Django project.

Edit the `settings.py` file.

```
(venv) # nano PROJECT_NAME/settings.py
```

Replace the `DATABASES` variable with the following content.

```
DATABASES = {

    "default": {

        'ENGINE': 'django.db.backends.postgresql',

        'NAME': os.environ.get("DATABASE_NAME"),

        'USER': os.environ.get("DATABASE_USER"),

        'PASSWORD': os.environ.get("DATABASE_PASS"),

        'HOST': os.environ.get("DATABASE_HOST"),

        'PORT': os.environ.get("DATABASE_PORT"),

        'OPTIONS': {'sslmode': 'require'}

    }

}
```

The above statements define the settings for the database backend and use the `os.environ()` function to fetch the credentials from the environment variables. Save the file and close the file editor using CTRL+X then ENTER.

Update the `.env` file.

```
(venv) # nano .env
```

Add the following contents to the file.

```
DATABASE_HOST=

DATABASE_PORT=

DATABASE_USER=

DATABASE_PASS=

DATABASE_NAME=
```

Populate the values using the information on the resource page. Save the file and close the file editor using CTRL+X then ENTER.

Export the environment variables.

```
(venv) # export $(xargs < .env)
```

The above command reads the `.env` file and exports it to the host machine's environment variables.

Apply the settings.

```
(venv) # python manage.py makemigrations

(venv) # python manage.py migrate

(venv) # python manage.py createsuperuser
```

The above commands read all the models defined in the Django project to initialize the specified PostgreSQL database. You may use different credentials for the superuser account to verify the database switch.

Verify the changes.

```
(venv) # python manage.py runserver 0.0.0.0:8000
```

The above command spawns a Django web server. You can verify the changes by opening `http://PUBLIC_IP:8000/admin` in your web browser and trying to log in using the new superuser credentials. Stop the server using CTRL+C to follow along with the next steps.

## Containerize the Django Project

You need to containerize the Django project to create a Kubernetes `Deployment` resource for handling the Django requests. This section demonstrates the steps to containerize the Django project so that you can build an image to use in the Kubernetes cluster.

Delete the old SQL lite database file.

```
(venv) # rm -f db.sqlite3
```

Disable the Django debugger.

```
(venv) # nano PROJECT_NAME/settings.py
```

Find and replace the following value in the file.

```
DEBUG = False
```

The above statement disables the Django debugger to hide the errors in the production environment that could lead to revealing the sensitive codebase.

Install the required Python package.

```
(venv) # pip install gunicorn
```

The above command installs the `gunicorn` package in the virtual environment. Gunicorn is a production-grade web server for running Python web applications. You will use it for serving the Django project leveraging better performance, reliability and scalability.

Create a new file named `requirements.txt`.

```
(venv) # pip freeze > requirements.txt
```

The above command fetches all the required packages and appends them to the `requirements.txt` file.

Deactivate the virtual environment.

```
(venv) # deactivate
```

Create a new Dockerfile.

```
# nano Dockerfile
```

Add the following contents to the file.

```
FROM python:3.8



WORKDIR /app



ENV PYTHONDONTWRITEBYTECODE 1

ENV PYTHONUNBUFFERED 1



RUN pip install --upgrade pip 

COPY ./requirements.txt /app

RUN pip install -r requirements.txt



COPY . /app



EXPOSE 8000



CMD ["gunicorn", "PROJECT_NAME.wsgi","-b", "0.0.0.0:8000"]
```

The above configuration inherits the official Python docker image and uses the `requirements.txt` to fulfill all the requirements. It executes the `gunicorn PROJECT_NAME.wsgi -b 0.0.0.0:8000` command to spawn a Gunicorn web server inside the container to serve the Django project.

Create a new private repository on DockerHub.

1.  Go to the [DockerHub website](https://hub.docker.com/) and log in to your account.
    
2.  Navigate to the "Repositories" tab in the top menu.
    
3.  Click the "Create Repository" button.
    
4.  Enter a name for your repository and select "Private" from the visibility options.
    
5.  Click the "Create" button to create your new private repository.
    

Log in to the DockerHub account.

```
# docker login
```

The above command prompts you to enter your DockerHub credentials to build and push the image on the DockerHub.

Build & push the image.

```
# docker build -t DOCKERHUB_USERNAME/REPO_NAME:latest .

# docker push DOCKERHUB_USERNAME/REPO_NAME:latest
```

The above commands build and push the image to the DockerHub, which you will use in the Kubernetes manifest in the later steps.

## Prepare the Kubernetes Cluster

You must prepare the Kubernetes cluster for the Django deployment by installing the required plugins and creating a few resources. This section demonstrates the steps to install the `nginx-ingress` controller, install the `cert-manager` plugin, create a `ClusterIssuer` resource for issuing Let's Encrypt certificates, and create a secret resource for the DockerHub credentials.

Install the `nginx-ingress` controller and the `cert-manager` plugin.

```
# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml

# kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
```

The above commands install the `nginx-ingress` controller and the `cert-manager` plugin on the Kubernetes cluster using the official manifest files. The `nginx-ingress` controller provisions a load balancer add-on to handle incoming HTTP requests on the `Ingress` resources.

Fetch the load balancer IP address.

```
# kubectl get services/ingress-nginx-controller -n ingress-nginx
```

You must point the `A` record for django.example.com to the IP address of the load balancer.

Create a `ClusterIssuer` resource for Let's Encrypt CA.

```
# nano ~/le_clusterissuer.yaml
```

Add the following contents to the file.

```
apiVersion: cert-manager.io/v1

kind: ClusterIssuer

metadata:

  name: letsencrypt-prod

spec:

  acme:

    server: https://acme-v02.api.letsencrypt.org/directory

    email: "YOUR_EMAIL"

    privateKeySecretRef:

      name: letsencrypt-prod

    solvers:

      - http01:

          ingress:

            class: nginx
```

The above manifest creates a `ClusterIssuer` resource for issuing Let's Encrypt certificates. It uses the `HTTP01` challenge solver to verify the ownership.

Apply the configuration.

```
# kubectl apply -f ~/le_clusterissuer.yaml
```

Create the secret resource for DockerHub credentials.

```
# kubectl create secret docker-registry regcred --docker-username=DOCKERHUB_USER --docker-password=DOCKERHUB_PASS --docker-email=DOCKERHUB_EMAIL
```

The above command creates a new secret resource with your DockerHub credentials, which you will use in the next section for downloading the image you built in the previous section.

## Deploy the Django Project

You took the necessary steps in the previous sections to create and set up a Django project and a Kubernetes cluster. This section will walk you through the steps to deploy the Django project on the Kubernetes cluster. It involves creating the necessary deployment and service objects within the cluster and configuring them to host and run the Django project.

Create a new file named `django.yaml`.

```
# nano ~/django.yaml
```

Add the following contents to the file.

```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: django-deployment

spec:

  replicas: 3

  selector:

    matchLabels:

      name: django-app

  template:

    metadata:

      labels:

        name: django-app

    spec:

      imagePullSecrets:

        - name: regcred

      containers:

        - name: django

          image: DOCKERHUB_USERNAME/REPO_NAME:latest

          imagePullPolicy: Always

          ports:

            - containerPort: 8000

          envFrom:

          - secretRef:

              name: django-secrets



---

apiVersion: v1

kind: Service

metadata:

  name: django-service

spec:

  ports:

    - name: http

      port: 80

      protocol: TCP

      targetPort: 8000

  selector:

    name: django-app



---

apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: django-ingress

  annotations:

    kubernetes.io/ingress.class: nginx

    cert-manager.io/cluster-issuer: letsencrypt-prod

spec:

  tls:

    - secretName: django-tls

      hosts:

        - django.example.com

  rules:

    - host: django.example.com

      http:

        paths:

          - path: /

            pathType: Prefix

            backend:

              service:

                name: django-service

                port:

                  number: 80
```

The above manifest creates three resources. The `Deployment` resource uses the image you pushed to DockerHub. The `Service` resource exposes connections to the `Deployment` resource inside the cluster. The `Ingress` resource exposes the `Service` resource globally and uses the `ClusterIssuer` resource to issue a Let's Encrypt certificate.

Create the secret resource for the environment variables.

```
# kubectl create secret generic django-secrets --from-env-file=.env
```

Apply the manifest file.

```
# kubectl apply -f ~/django.yaml
```

The above command applies the `django.yaml` manifest on the cluster. After a few minutes, you can verify the deployment by opening `https://django.example.com` on your web browser. It should have an SSL certificate, and the response should remain unchanged.

## Conclusion

This article demonstrated the steps to use the Vultr Object Storage to host the static files, configure the Vultr Managed PostgreSQL Database as the database backend, containerize the Django application and deploy the Django application on the Vultr Kubernetes Engine. You can follow these steps to deploy your existing Django application on the Vultr Cloud Infrastructure.

## More Information

-   [django-storages Module Documentation](https://django-storages.readthedocs.io/en/latest/)
    
-   [Gunicorn Module Documentation](https://docs.gunicorn.org/en/stable/)
    
-   [Nginx Ingress Controller Documentation](https://docs.nginx.com/nginx-ingress-controller/)
    
-   [cert-manager Plugin Documentation](https://cert-manager.io/docs/)