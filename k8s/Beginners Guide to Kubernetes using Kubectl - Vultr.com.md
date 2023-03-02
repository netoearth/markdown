## Introduction

Kubernetes is a container orchestration platform that is built to automate application deployment and scaling. To scale and deploy container-based applications swiftly using Kubernetes, a command-line called Kubectl is used. This tool allows you to configure and control the Kubernetes cluster by communicating with your cluster API server.

In this tutorial, you will learn how to configure Kubernetes clusters using Kubectl and how to install it in Windows.

## Installation Steps

1.  Download the [latest Kubectl binary file](https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe). Ensure that you download and use a Kubectl version that is not different from your cluster by more than one version to prevent compatibility issues.
    
    You can also use the following curl command to download the binary file if you have already installed curl:
    
    ```
      curl -LO "https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe"
    ```
    
2.  Create a directory where you will store the binary file you just downloaded.
    
3.  Copy and paste the binary file to the directory you just created.
    
4.  Navigate to the environment settings and add a new environment path in the system variable section. Make sure that the path directory you add is the directory path for the folder you just created which contains the Kubectl binary file.
    
5.  Apply and save the new changes you just made.
    
6.  Use the following command to check the Kubernetes version installed on the client and server:
    
    ```
      kubectl version --client
    ```
    

The above command will output the following details if Kubectl has been installed successfully:

```
    Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0",   GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z",  GoVersion:"go1.17.3", Compiler:"gc", Platform:"windows/amd64"}
```

## Declarative Management of Kubernetes Objects

Kubernetes has two command approaches used to create and manage Kubernetes objects. Which are the declarative approach and the imperative approach. The declarative approach involves specifying the details of the task that you want to execute. The declarative approach involves the usage of the `kubectl apply` command when creating objects.

### How to Create an Object Using a Declarative Approach

Use the command shown below to create objects that you have not created in a specific directory. Use the -R flag to recursively process the directories:

```
    kubectl apply -f [add your directory here]
```

To update Kubernetes objects, you can also use the above command which will update the live configuration using the configuration file. Whatever has been deleted in the configuration file will also be removed in a live configuration:

### Deleting an Object Using a Declarative Approach

You can use the following declarative approach to delete an object. But this approach is not recommended for usage as it can delete important objects unintentionally. Henceforth, it is imperative to use an imperative approach when deleting objects as this approach allows you to explicitly state what you want to delete.

```
    kubectl delete -f [add your filename here]
```

## Manage Kubernetes Objects with Imperative Commands

The Kubernetes imperative approach to creating objects involves explicitly stating and creating objects using verbal commands like `kubectl run` which creates a new pod. Unlike the declarative approach, this approach does not use the kubectl apply command.

### How to Create Objects

The imperative approach uses the `run` command to create a new pod that will run a container and the `kubectl delete` command to delete objects. Use the `kubectl expose` command to create a new service object.

### How to Update Objects

Kubectl supports the `kubectl scale` command which removes or adds a pod by upgrading the replica count of the controller. If you want to modify certain fields of a live object you can use the `kubectl patch` command. While the `kubectl edit` command alters the raw configuration of a live object.

### How to View an Object

The `kubectl get` command is used to print the objectâ€™s information while the describe command prints detailed information about matching objects. On the other side of the spectrum, the `kubectl logs` command is used to print the stdout and stderr for a container running in a Pod.

### How to Create a Pod

A pod is the smallest deployable unit that encases containers in a Kubernetes node and represents a single instance of a process. All applications in a pod use and share the same resources in a pod. Pod specifications determine how a container runs.

Use the following command to create a pod but you need a Docker image when creating a pod:

```
    kubectl run <add pod name here  > --image <add image name here>:tag
```

You can delete a pod by running the following Kubectl command and the terminal will confirm that the terminal has been deleted by displaying pod <your podâ€™s name> deletedâ€:

```
    kubectl delete pod <add pod name here>
```

### Creating a Cluster Role

A resource in Kubernetes can be assigned to permissions called ClusterRoles. ClusterRoles define these permissions set at the cluster level.

Do not confuse rules with clusterRoles rules only apply to a specified namespace a ClusterRole applies to the cluster scope.

Use the below command to create a ClusterRole named "pod-reader" that allows the user to perform "get", "watch" and "list" on pods:

```
     kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
```

### Create a ConfigMap

A configuration map is an API object that stores non-confidential data in key pair values. A pod uses a configuration map as a configuration file.

Data stored in a config map has a limit of 1MiB. If you have data that exceeds these limits you can use a separate database file or service.

Create configmap using the following command, which requires a name and a data source:

```
    kubectl create configmap <add a configmap name here> <add a data-source here>
```

For example:

```
    kubectl create configmap app-settings --from-file=app-container/settings/app.properties
```

ConfigMaps determine application updates as they contain the configuration data. To prevent unplanned updates from causing application outages, you should use immutable ConfigMaps. Immutable ConfigMaps and immutable secrets do not change.

### How to Expose Your App with NodePort

In Kubernetes, a service is responsible for enabling network access to a group of deployed pods. There are five types of services namely:

-   LoadBalancer
    
-   NodePort
    
-   ClusterIP
    
-   ExternalName.
    
-   Headless
    

In this article, you will only learn how to expose an app using NodePort.

A NodePort is an open port on a node, all traffic that is directed to this node will be routed to the service. NodePort is supported by every cluster. This type of service [has downsides](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)as you can only have one service per port.

Use the following Kubectl command to expose your deployment and create a new service that will expose your application. Specify the service type as NodePort and enter the required ports.

```
    kubectl expose deployment <enter deployment name here> --name <enter your service name here> \

        --type NodePort --protocol TCP --port <enter your port here> --target-port <enter your targetport here>
```

## Kubectl Command Cheat Sheet

Here is a list of essential commands that you will use after you install kubectl:

-   **kubectl get namespaces:** Use this command to create a plain-text list of all namespaces.
    
-   **kubectl get daemonset**: This command is used to display a plain-text list of all daemon sets.
    
-   **kubectl create namespace \[namespace-name\]**: Use this command to create a namespace.
    
-   **kubectl describe pods \[pod-name\]**: Use this command to view details about a particular pod.
    
-   **kubectl config current-context**: Use this command to display and view kubeconfig files.
    
-   **kubectl logs \[pod-name\]**: Use this command to print logs from containers in a pod.