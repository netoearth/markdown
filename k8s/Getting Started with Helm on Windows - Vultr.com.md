## Introduction

Kubernetes makes application deployment easy and efficient as it gives you various container orchestration tools. Helm is one of those orchestration tools that make the deployment process easy.

Helm is a Kubernetes package manager used to share and distribute charts used for deploying and managing Kubernetes applications. Helm is an open-source project, and it is free.

Helm comprises the following entities:

-   **Chart**: A Helm chart contains Kubernetes YAML files which can be installed on your cluster to configure your applications. Helm charts are helpful because they provide an easy way of deploying microservices without having to do it manually using Kubectl. These charts can be reused in different environments, which saves a lot of time. You can download a Helm chart on [ArtifactHub](https://artifacthub.io/) containing all the files and YAML files needed to deploy an application.
    
-   **Package**: A package contains your application's resources for deployment.
    
-   **Release**: A release is a version of an instance or chart running on your cluster.
    
-   **Repository**: A repository contains a list of charts that can be downloaded and used by other Kubernetes administrators and developers. Anyone can create a Helm repository and host it on GitHub.
    

In this article, you will learn how to install the Helm command-line on Windows. You will also learn how to initialize and build Helm charts.

## Prerequisites

You need a running [Kubernetes cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/) and [Kubectl](https://www.vultr.com/docs/beginners-guide-to-kubernetes-using-kubectl/).

## How to Install the Helm CLI

The Helm command-line enables you to install, update, and uninstall a chart and its repository.

1.  Use the following chocolatey command to download the Helm command line:
    
    ```
    choco install kubernetes-helm
    ```
    
2.  Allow `kubernetes-helm` to run 'chocolateyInstall.ps1' to complete the installation process by pressing the y key on your keyboard:
    
    ```
    The package kubernetes-helm wants to run 'chocolateyInstall.ps1'.
    
    Note: If you don't run this script, the installation will fail.
    
    Note: To confirm automatically next time, use '-y' or consider:
    
    choco feature enable -n allowGlobalConfirmation
    
    Do you want to run the script?([Y]es/[A]ll - yes to all/[N]o/[P]rint): y
    ```
    
3.  When the installation process has been completed, the Helm installed successfully" message and the location where Helm has installed will be displayed:
    
    ```
    ShimGen has successfully created a shim for helm.exe
    
    The install of Kubernetes-helm was successful.
    
    Software installed to 'C:\ProgramData\chocolatey\lib\kubernetes-helm\tools'
    ```
    
4.  Use the following command to check the Helm command-line version:
    
    ```
    helm version --help
    ```
    
    You will get the following output:
    
    ```
    version.BuildInfo{Version:"v3.2.1", GitCommit:"fe51cd1e31e6a202cba7dead9552a6d418ded79a", GitTreeState:"clean",                 GoVersion:"go1.13.10"}
    ```
    
5.  Use the following command to get a list of commands that will help you operate Helm charts:
    
    ```
    helm get -h
    ```
    

## How to install a Helm Chart

Start by downloading a chart's repository using the following command:

```
    helm repo add [repository name] [repository address]
```

For example:

```
    helm repo add bitnami https://charts.bitnami.com/bitnami
```

After you have installed your desired repository, use the search command to browse and discover new charts available in the chart repository you just added:

```
    helm search repo [repository name]
```

After successfully installing the chart repository, use the following command to install your desired chart:

```
    helm install [release name] [chart name]
```

For example:

```
    helm install my-release bitnami/mysql
```

After you have installed the Helm chart, use the following command to see all installed and deployed Helm charts:

```
    helm list
```

## How to Create a Helm Chart

Use the following command to create a chart:

```
    Helm create [chart name]
```

## The Structure of a Helm Chart Directory

A Helm chart directory will be created when creating a Helm chart. This chart directory will have all the necessary files needed for configuring your applications. Charts also have child charts and dependent charts.

Here is the structure of a Helm chart:

```
    â”œâ”€â”€ .helmignore  

    â”œâ”€â”€ values.yaml  

    â”œâ”€â”€ charts  

    â”œâ”€â”€ Chart.yaml   

    â””â”€â”€ templates 

        â””â”€â”€ tests   
```

The following list explains files and directories found in a Helm chart directory:

1.  `.helmignore` contains files that should be ignored when packaging a Helm chart.
    
2.  The `values.yaml` file contains all the chart specifications and values that will be passed to the running chart. This file lets you declare the charts services, volume, and RBAC.
    
3.  The charts folder contains all of the charts that your chart depends on.
    
4.  The `Chart.yaml` file contains all the information and details about the chart. This is where you add the chart's version number.
    
5.  The templates directory contains all the files necessary for deploying the charts.
    

## How to Manage Helm Charts

Look on [ArtifactHub](https://artifacthub.io/) to find a Helm repository and choose a specific version of a chart suitable for your deployment configuration.

After installing the Helm command-line and adding a repository you can now browse and search for production-ready applications that are packaged from your favorite vendors through repositories. Use the following commands to manage Helm charts:

-   Use the following command to upgrade a release:
    
    ```
    helm upgrade [release] [chart]
    ```
    
-   Use the following command to download the release information:
    
    ```
    helm get all [release]
    ```
    
-   A Helm [hook](https://helm.sh/docs/topics/charts_hooks/#:~:text=Helm%20provides%20a%20hook%20mechanism,any%20other%20charts%20are%20loaded.) is used to load ConfigMaps. Use the following command to download all hooks:
    
    ```
    helm get hooks [release]
    ```
    
-   Use the following command to download the generated release manifest:
    
    ```
    helm get manifest [release]
    ```
    

## How to Uninstall Helm Charts

Use the following command to uninstall a release and wipe out the chart history:

```
    helm uninstall [chart name]
```

If you want to keep the chart history, add the following flag to the Helm uninstall command:

```
    --keep-history
```

The following command removes a repository from your system:

```
    helm repo remove [repository-name]
```

## Preventing Common Vulnerabilities

Here are ways you can prevent common vulnerabilities arising from Helm charts:

-   Use tools like Elasticsearch and Prometheus to monitor and track deployed charts to detect security issues that may arise from charts. Also, download and use official, trusted, and secure charts.
    
-   Test your charts on Kubernetes platforms such as [Vultr Kubernetes Engine](https://www.vultr.com/kubernetes/) before deploying them and make changes immediately after detecting vulnerabilities.
    
-   Secure secrets by encrypting them. Unencrypted secrets stored in templates and `values.yaml` files can be exploited by unauthorized people who can access the charts and templates.