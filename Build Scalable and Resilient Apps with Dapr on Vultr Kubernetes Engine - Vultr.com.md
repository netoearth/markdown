## Introduction

Dapr is a portable, event-driven runtime to build scalable and resilient applications. It is platform agnostic, meaning you can run Dapr-based applications anywhere, ranging from your local workstation and virtual machines to a Kubernetes cluster, different cloud providers, or even at the edge. Dapr is also language agnostic, so that you can choose any language or framework for your application.

This article explains how to start with Dapr on Vultr Kubernetes Engine (VKE). After setting up Dapr on VKE, you will deploy microservices that communicate with each other via the Dapr runtime and store data in a Vultr Managed Database for Redis.

The following topics will be covered in this article:

-   An overview of Dapr, including basic concepts.
    
-   How to set up VKE, deploy Dapr to it, and integrate Dapr with Redis.
    
-   An end-to-end example of microservices running on VKE, integrated via Dapr.
    
-   Best practices for running Dapr-based applications.
    

### Introduction to Dapr

Dapr runs alongside an application and provides capabilities such as persistence, service invocation, security, distributed tracing, and so on. This is possible because Dapr adopts a _sidecar_ architecture pattern and exposes its APIs via **HTTP** and **gRPC**. Thus, applications do not need to interact directly with the Dapr runtime and remain agnostic of the underlying implementation details. This provides a clear separation of concerns and makes it easy to integrate Dapr with other platforms, runtimes or services. Dapr also provides SDKs for multiple languages such as Java, JavaScript, Python, .NET, PHP, Go, Rust and C++. These SDKs expose the functionality of Dapr building blocks through a language-specific API.

#### Dapr Building Blocks

Although Dapr offers features similar to that of a Service Mesh, it's quite different. While a Service Mesh is focused on infrastructure concerns such as networking, Dapr is focused on the application layer. It exposes different capabilities in the form of APIs called _Building Blocks_, thereby codifying best practices for building platform and language-agnostic microservices. A building block is nothing but an **HTTP** or **gRPC** API that can be invoked from the application code.

Here is an overview of the building blocks that Dapr offers:

-   **State management** - Provides persistence for applications based on key/value-based query and state APIs backed by pluggable state store implementations.
    
-   **Inter-service invocation** - Allows applications to communicate with each other using **HTTP** or **gRPC** endpoints.
    
-   **Publish and subscribe** - This allows decoupled applications where senders (or publishers) publish messages to a topic that subscribers can consume.
    
-   **Bindings** - Enables bi-directional integration with Dapr by allowing applications to invoke an external service and triggering applications via events sent by an external service.
    
-   **Observability** - You can configure Dapr to emit tracing data using standard protocols such as Open Telemetry and integrate with observability tools.
    
-   **Secrets** - This building block integrates Dapr with custom secret stores.
    
-   **Configuration** - Enables you to integrate configuration stores to retrieve application-specific configuration information.
    
-   **Distributed lock** - Use the distributed lock API to allow multiple instances of an application to access a resource in a conflict-free manner using locks.
    
-   **Actors** - Dapr has a virtual actor based implementation and provides a single-threaded programming model where actors are garbage collected when not in use.
    

You can choose the building block(s) based on your application requirements. Because each of these building block APIs is independent, you integrate one or more such blocks in your application.

### Dapr State Store

You can use a _State Store_ to save and retrieve application states from underlying databases that Dapr integrates with. All the state store implementations offer a set of features that are commonly required by applications. These include traditional CRUD operations, transactions, and bulk operations, along with the ability to configure concurrency and consistency parameters.

You can query state with the Dapr state management query API or directly with the data store's native client library/SDK. It's recommended to use the Query API since it allows you to retrieve state store data irrespective of underlying database-specific implementation.

### Dapr on Kubernetes

In Kubernetes, Dapr runs alongside the application in the same **Pod** as a _sidecar_ container. All you need to do is add a few annotations to your application manifest **Pod** schema. Here is an example:

```
annotations:

    dapr.io/enabled: "true"

    dapr.io/app-id: "myapp"

    dapr.io/app-port: "8080"

    dapr.io/config: "tracing"
```

The following control-plane components are the key to running Dapr on Kubernetes:

-   **dapr-sidecar-injector**: Responsible for injecting **Dapr** as a sidecar container into **Pod**s.
    
-   **dapr-operator**: It manages Dapr component updates and Kubernetes services endpoints.
    
-   **dapr-placement**: It is only used by actors to create mapping tables that map actor instances to **Pod**s.
    
-   **dapr-sentry**: Provides mutual TLS (**mTLS**) between services and also acts as a certificate authority.
    

## Application Overview

The solution demonstrated in this article consists of two microservices. One is a **Python** application that generates messages, and the other one is a **Node.js** application that consumes these messages. The **Node.js** application exposes two **HTTP** endpoints:

-   A **POST** endpoint to receive messages. The message data is persisted to a Vultr Redis Managed database instance.
    
-   A **GET** endpoint to query data. This invokes the Vultr Redis Managed database instance to extract the latest state.
    

Now you have an understanding of core Dapr concepts and the architecture overview, let's move on to the configuration and deployment of the solution.

Make sure you complete the steps below to follow the rest of the instructions in this article.

## Prerequisites

-   Deploy a Vultr Managed Database for Redis by following the [Quickstart guide](https://www.vultr.com/docs/vultr-managed-databases/).
    
-   After Redis is deployed, click the **Manage** icon to open the **Overview** tab to retrieve the **username**, **password**, **host**, and **port** from the **Connection Details** section.
    
-   [Install kubectl](https://kubernetes.io/docs/tasks/tools/), which is a Kubernetes command-line tool that allows you to run commands against Kubernetes clusters.
    
-   Deploy a VKE cluster using the [Reference Guide](https://www.vultr.com/docs/vultr-kubernetes-engine/). Once it's deployed, from the **Overview** tab, click the **Download Configuration** button in the upper-right corner to download your **kubeconfig** file and save it to a local directory.
    
    Point **kubectl** to the VKE cluster by setting the **KUBECONFIG** environment variable to the path where you downloaded the cluster **kubeconfig** file in the previous step.
    
    ```
    export KUBECONFIG=<path to VKE kubeconfig file>
    ```
    
    Verify the same using the following command:
    
    ```
    kubectl config current-context
    ```
    
-   [Install curl](https://curl.se/download.html) which is a popular command-line **HTTP** client.
    
-   Install the Redis CLI.
    

## Install Dapr CLI

Use the following command to install Dapr CLI locally:

```
curl -fsSL https://raw.githubusercontent.com/dapr/cli/master/install/install.sh | /bin/bash
```

Verify Dapr CLI installation:

```
dapr --version
```

Run the following command to deploy **Dapr** on the VKE cluster. Because the `--wait` flag is specified, the CLI will wait until **Dapr** is completely deployed and running on the cluster.

```
dapr init --kubernetes --wait
```

You should see the following output:

```
Deploying the Dapr control plane to your cluster... 
```

Unless specified, **Dapr** will be installed to the **dapr-system** namespace by default (it will be created if it does not already exist). Use `-n` to specify a different namespace.

Once Dapr is deployed, you should see the following output:

```
Success! Dapr has been installed to namespace dapr-system. To verify, run `dapr status -k` in your terminal. To get started, go here: https://aka.ms/dapr-getting-started
```

To verify the deployment, run the following command in the terminal:

```
dapr status -k
```

You should see an output similar to this (the **VERSION** might differ in your case):

```
NAME                   NAMESPACE    HEALTHY  STATUS   REPLICAS  VERSION  AGE              

dapr-sentry            dapr-system  True     Running  1         1.9.5    37s  

dapr-dashboard         dapr-system  True     Running  1         0.11.0   37s  

dapr-placement-server  dapr-system  True     Running  1         1.9.5    36s    

dapr-operator          dapr-system  True    Running   1         1.9.5    37s    

dapr-sidecar-injector  dapr-system  True     Running  1         1.9.5    37s    
```

## Configure a Dapr State Store

Create a directory and switch to it:

```
mkdir vultr-dapr

cd vultr-dapr
```

Create a new file `redis-state-store.yaml`:

```
touch redis-state-store.yaml
```

Enter the below contents in `redis-state-store.yaml` and save the file. Make sure to replace the required attributes with the **host**, **username**, and **password** for the Redis database.

```
apiVersion: dapr.io/v1alpha1

kind: Component

metadata:

  name: statestore

spec:

  type: state.redis

  version: v1

  metadata:

  - name: redisHost

    value: <replace with hostname for Vultr managed redis database>:16752

  - name: redisUsername

    value: <replace with username for Vultr managed redis database>

  - name: redisPassword

    value: <replace with password for Vultr managed redis database>

  - name: enableTLS

    value: true
```

The `redis-state-store.yaml` file defines a statestore component of type **state.redis**.

-   **redisHost** - Connection string for the Redis host in the form of **host:port**.
    
-   **redisUsername** - Username for Redis host.
    
-   **redisPassword** - Password for Redis host.
    
-   **enableTLS** - Whether the Redis instance supports TLS.
    

Create the state store:

```
kubectl apply -f redis-state-store.yaml
```

You should see the following output:

```
component.dapr.io/statestore created
```

## Deploy the Node.JS Application to VKE

Create a new file `node-app.yaml`:

```
touch node-app.yaml
```

Enter below contents in `node-app.yaml` and save the file.

```
kind: Service

apiVersion: v1

metadata:

  name: nodeapp

  labels:

    app: node

spec:

  selector:

    app: node

  ports:

  - protocol: TCP

    port: 80

    targetPort: 3000

---

apiVersion: apps/v1

kind: Deployment

metadata:

  name: nodeapp

  labels:

    app: node

spec:

  replicas: 1

  selector:

    matchLabels:

      app: node

  template:

    metadata:

    labels:

      app: node

    annotations:

        dapr.io/enabled: "true"

        dapr.io/app-id: "nodeapp"

        dapr.io/app-port: "3000"

        dapr.io/enable-api-logging: "true"

    spec:

      containers:

      - name: node

        image: ghcr.io/dapr/samples/hello-k8s-node:latest

        env:

        - name: APP_PORT

          value: "3000"

        ports:

        - containerPort: 3000

        imagePullPolicy: Always
```

Deploy the **Node.js** application to the cluster:

```
kubectl apply -f node-app.yaml
```

You should see the following output:

```
deployment.apps/nodeapp created
```

Check the deployment. Wait for the **STATUS** to transition to _Running_.

```
kubectl get pods -l=app=node



#output



NAME                       READY   STATUS    RESTARTS   AGE

nodeapp-7bbf67cd77-jhv2r   2/2     Running   0          47s
```

Verify the **Service** (**CLUSTER-IP** might differ in your case):

```
kubectl get service/nodeapp



#output



NAME      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE

nodeapp   ClusterIP   10.109.80.71   <none>        80/TCP    10s
```

## Deploy the Python Application to VKE

Create a new file, `python-app.yaml`:

```
touch python-app.yaml
```

Enter the below contents in `python-app.yaml` and save the file.

```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: pythonapp

  labels:

    app: python

spec:

  replicas: 1

  selector:

    matchLabels:

      app: python

  template:

    metadata:

      labels:

        app: python

      annotations:

        dapr.io/enabled: "true"

        dapr.io/app-id: "pythonapp"

        dapr.io/enable-api-logging: "true"

    spec:

      containers:

    - name: python

      image: ghcr.io/dapr/samples/hello-k8s-python:latest
```

To deploy the Python application to the cluster:

```
kubectl apply -f python-app.yaml
```

You should see the following output:

```
deployment.apps/pythonapp created
```

Check the deployment. Wait for the **STATUS** to transition to _Running_.

```
kubectl get pods -l=app=python



#output



NAME                         READY   STATUS    RESTARTS   AGE

pythonapp-6bdf8dd995-fvz6h   2/2     Running   0          12s
```

## Test the Solution

Use port forwarding to access the **Node.js** application running in the VKE cluster.

```
kubectl port-forward service/nodeapp 8080:80
```

You should see the following output:

```
Forwarding from 127.0.0.1:8080 -> 3000

Forwarding from [::1]:8080 -> 3000
```

Open a new terminal and point **kubectl** to the VKE cluster by setting the **KUBECONFIG** environment variable to the path where you downloaded the cluster **kubeconfig** file in the previous step.

```
export KUBECONFIG=<path to VKE kubeconfig file>
```

Check the logs of the **Node.js** application.

```
kubectl logs -f -l=app=node -c node
```

You should see logs similar to this:

```
Got a new order! Order ID: 1

Successfully persisted state

Got a new order! Order ID: 2

Successfully persisted state

Got a new order! Order ID: 3

Successfully persisted state
```

Use **curl** to invoke the **Node.js** application HTTP **GET** endpoint to query the latest order ID.

```
curl localhost:8080/order
```

You should see a JSON output similar to this (the order ID might differ in your case):

```
{"orderId":419}
```

### Query Data from Redis

To connect to the Vultr Redis Managed database using Redis CLI, click the **Manage** icon to open the **Overview** tab and click **Copy Connection String** in the **Connection Details** section.

Paste the contents in the terminal. Once you are connected to the Redis instance, execute the below command:

```
keys *
```

This will give you the name of the key in which the **Node.js** application data is stored. You should see the following output:

```
1) "nodeapp||order"
```

This is a Redis **Hash** data structure. To check the data, execute this command:

```
hgetall "nodeapp||order"
```

You should see an output similar to this (the order ID and version might differ in your case):

```
1) "data"

2) "{\"orderId\":800}"

3) "version"

4) "799"
```

## Best Practices for Production

So far, you have deployed a set of microservices to a Kubernetes cluster and integrated them using Dapr. Although this was a simple way to get started, there are a few best practices you should bear in mind while using Dapr in production.

-   **Cluster capacity** - The recommended cluster size is at-least _three_ Kubernetes worker nodes to support a highly-available Dapr installation.
    
-   **Dapr HA mode** - A HA Dapr control plane configuration consists of three replicas of each control plane **Pod** in the **dapr-system** namespace.
    
-   **Sidecar settings** - Explicitly configure the resource assignments for the Dapr sidecar using the following annotations:
    
    ```
    dapr.io/sidecar-cpu-limit
    
    dapr.io/sidecar-memory-limit
    
    dapr.io/sidecar-cpu-request
    
    dapr.io/sidecar-memory-request
    ```
    
-   **Tracing and metrics configuration** - Configure distributed tracing and metrics for your applications and the Dapr control plane in production.
    
-   **Security configuration** - Make sure to enable **mTLS**, Dapr API authentication, Dapr to App API authentication, and install the Dapr control plane on a dedicated namespace such as **dapr-system**.
    

## Conclusion

In this article, you learned how to integrate microservices on Kubernetes using Dapr. Along the way, you learned the key concepts of Dapr and also configured a Vultr Managed Database for Redis as the Dapr state store. Finally, you explored some best practices for production-grade Dapr deployments.

You can also learn more in the following documentation:

-   [Vultr Managed Databases Quickstart](https://www.vultr.com/docs/vultr-managed-databases/)
    
-   [Redis Managed Database Guide](https://www.vultr.com/docs/redis-managed-database-guide)
    
-   [Vultr Kubernetes Engine (VKE) Reference Guide](https://www.vultr.com/docs/vultr-kubernetes-engine/)
    
-   [Vultr Kubernetes Engine (VKE) Changelog](https://www.vultr.com/docs/vultr-kubernetes-engine-changelog/)