## Introduction

Redis is an open source, in-memory data store that supports high-availability, scalability and reliability through a combination of asynchronous replication and Redis Cluster. Redis List is extensively used for building asynchronous applications. For example it is the foundation of popular open-source frameworks such as `resque` and `sidekiq`. With Redis List, you can use the producer-consumer pattern with a combination of `LPUSH` and `BRPOP` commands.

This article will show how to use `KEDA` to scale Redis application built using Vultr Redis Managed Database and deployed to Vultr Kubernetes Engine. It covers the following topics:

-   `KEDA` overview and installation.
    
-   Prepare Producer and Consumer applications.
    
-   Deploy Producer and Consumer applications to Vultr Kubernetes Engine (VKE).
    
-   Test and observe the complete auto-scaling solution.
    

## Prerequisites

-   Deploy a Vultr Redis Managed Database by following the [Quickstart guide](https://www.vultr.com/docs/vultr-managed-databases/). After it's deployed, get the connectivity information and click the **Manage** icon to open the **Overview** tab to retrieve the `username`, `password`, `host` and `port` from the **Connection Details** section.
    
-   [Install kubectl](https://kubernetes.io/docs/tasks/tools/), which is a Kubernetes command-line tool that allows us to run commands against Kubernetes clusters.
    
-   Deploy a Vultr Kubernetes Engine (VKE) cluster using the [Reference Guide](https://www.vultr.com/docs/vultr-kubernetes-engine/). Once it's deployed,from the **Overview** tab, click the **Download Configuration** button in the upper-right corner to download your `kubeconfig` file and save it to a local directory.
    
-   To access the VKE cluster from your local machine, refer to the downloaded `kubeconfig` file as:
    
    ```
    kubectl --kubeconfig=<path to VKE kubeconfig file> get nodes
    ```
    
-   Make sure [Docker is installed](https://docs.docker.com/get-docker/) as well. You will need it to build and push Docker images.
    

## KEDA overview

`KEDA` (Kubernetes-based Event Driven Autoscaler) can be used to manage the auto-scaling of Kubernetes workloads based on the number of events or items that need to be processed. It is a component that can be added into any Kubernetes cluster and builds on top of native Kubernetes components like the Horizontal Pod Auto-scaler (HPA). With `KEDA`, we can directly reference the application (`Deployment`, `Job` or `StatefulSet`) that needs to be scaled.

`KEDA` plays two key roles - Scale agent, Metrics provider.

1.  It activates and deactivates `Deployment`s to scale to and from zero on no events. This is taken care of by the `keda-operator` container in the `KEDA` installation.
    
2.  `KEDA` acts as a Kubernetes metrics server that exposes Redis List length to the Horizontal Pod Autoscaler which is responsible for scaling `Pod`s. This is taken care of by the `keda-operator-metrics-apiserver` container in the `KEDA` installation.
    

### KEDA Custom Resources

`KEDA` has the following CRDs: `scaledobjects.keda.sh`, `scaledjobs.keda.sh`, `triggerauthentications.keda.sh` and `clustertriggerauthentications.keda.sh`.

Although this article demonstrates a `Deployment`, `KEDA` also allows us to map an event source to a `StatefulSet`, Custom Resource or `Job` for scaling.

-   `ScaledObject`s represent the mapping between an event source (such as Redis List) and the `Deployment`, `StatefulSet` or any Custom Resource that defines `/scale` subresource.
    
-   `ScaledObject` may also reference a `TriggerAuthentication` resource which contains the authentication configuration or secrets to monitor the event source.
    

## Application

The sample application presented in this article consists of a producer and consumer component. The producer application pushes data to a Redis List (using `LPUSH`) while the consumer application processes data from it (using `BRPOP`). Both applications are built using the Go Redis client and deployed to a Vultr Kubernetes Engine (VKE) cluster.

## Install KEDA

Deploy `KEDA` directly from `YAML` file (although it's also possible to use [Helm charts](https://keda.sh/docs/2.9/deploy/#helm) or [Operator Hub](https://keda.sh/docs/2.9/deploy/#operatorhub)). If required, replace the version as well - we are using version **2.9.0** in this article.

```
kubectl --kubeconfig=<path to VKE kubeconfig file> apply -f https://github.com/kedacore/keda/releases/download/v2.9.0/keda-2.9.0.yaml
```

You will see an output similar to this:

```
namespace/keda created

customresourcedefinition.apiextensions.k8s.io/clustertriggerauthentications.keda.sh created

customresourcedefinition.apiextensions.k8s.io/scaledjobs.keda.sh created

customresourcedefinition.apiextensions.k8s.io/scaledobjects.keda.sh created

customresourcedefinition.apiextensions.k8s.io/triggerauthentications.keda.sh created

serviceaccount/keda-operator created

clusterrole.rbac.authorization.k8s.io/keda-external-metrics-reader created

clusterrole.rbac.authorization.k8s.io/keda-operator created

rolebinding.rbac.authorization.k8s.io/keda-auth-reader created

clusterrolebinding.rbac.authorization.k8s.io/keda-hpa-controller-external-metrics created

clusterrolebinding.rbac.authorization.k8s.io/keda-operator created

clusterrolebinding.rbac.authorization.k8s.io/keda-system-auth-delegator created

service/keda-metrics-apiserver created

service/keda-operator created

deployment.apps/keda-metrics-apiserver created

deployment.apps/keda-operator created

apiservice.apiregistration.k8s.io/v1beta1.external.metrics.k8s.io created
```

Check the CRDs that were deployed:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get crd
```

In addition to other CRDs, you should see these newly added ones related to `KEDA`:

```
NAME                                    CREATED AT

.....

clustertriggerauthentications.keda.sh   2022-12-13T16:25:19Z

scaledjobs.keda.sh                      2022-12-13T16:25:19Z

scaledobjects.keda.sh                   2022-12-13T16:25:19Z

triggerauthentications.keda.sh          2022-12-13T16:25:19Z
```

Check the status of the `KEDA` deployment and wait for it to be `READY`:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get deployment -n keda



#output



NAME                     READY   UP-TO-DATE   AVAILABLE   AGE

keda-metrics-apiserver   1/1     1            1           57s

keda-operator            1/1     1            1           57s
```

## Prepare consumer application

Create a directory and switch to it:

```
mkdir redis-keda-vultr

cd redis-keda-vultr
```

### Initialize the project

Create a directory and switch to it:

```
mkdir redis-consumer

cd redis-consumer
```

Create a new Go module:

```
go mod init redis-consumer
```

> This will create a new `go.mod` file

Create a new file `consumer.go`:

```
touch consumer.go
```

### Import libraries

To import required Go modules, add the following to `consumer.go` file:

```
package main



import (

    "context"

    "crypto/tls"

    "fmt"

    "log"

    "math/rand"

    "os"

    "os/signal"

    "syscall"

    "time"



    "github.com/go-redis/redis/v8"

)
```

### Add the `init` function

Add the code below to `consumer.go` file:

```
var client *redis.Client

const redisURLTemplate = "rediss://%s:%s@%s:%s"

var listName string



func init() {

    redisHost := os.Getenv("REDIS_HOST")

    if redisHost == "" {

        log.Fatal("missing environment variable REDIS_HOST")

    }



    redisPort := os.Getenv("REDIS_PORT")

    if redisPort == "" {

        log.Fatal("missing environment variable REDIS_PORT")

    }



    redisUsername := os.Getenv("REDIS_USERNAME")

    if redisUsername == "" {

        log.Fatal("missing environment variable REDIS_USERNAME")

    }



    redisPassword := os.Getenv("REDIS_PASSWORD")

    if redisPassword == "" {

        log.Fatal("missing environment variable REDIS_PASSWORD")

    }



    redisURL := fmt.Sprintf(redisURLTemplate, redisUsername, redisPassword, redisHost, redisPort)

    fmt.Println("Redis URL", redisURL)



    opt, err := redis.ParseURL(redisURL)

    if err != nil {

        log.Fatal("invalid redis url", err)

    }



    opt.TLSConfig = &tls.Config{}



    client = redis.NewClient(opt)



    err = client.Ping(context.Background()).Err()



    if err != nil {

        log.Fatal("ping failed. could not connect", err)

    }

    fmt.Println("successfully connected to redis", redisURL)



    listName = os.Getenv("LIST_NAME")

    if listName == "" {

        log.Fatal("missing environment variable LIST_NAME")

    }

}
```

-   The `init` function reads the Redis connectivity information from environment variables. It uses `REDIS_HOST`, `REDIS_PORT`, `REDIS_USERNAME` and `REDIS_PASSWORD` to store Vultr Managed Redis database `host`, `port`, `username` and `password` respectively.
    
-   It uses the above data to create a Redis URL and passes it into `redis.ParseURL` to get a `redis.Option` object that is further used in creating a `redis.Client` instance.
    
-   `Ping` function is used to verify successful connectivity with Redis. If connectivity fails, the program exits with an error message.
    
-   Finally, the name of the Redis List to be used is also retrieved from the `LIST_NAME` environment variable.
    

### Add the `main` function

Add the code below to `consumer.go` file:

```
func main() {

    defer func() {

        err := client.Close()

        if err != nil {

            fmt.Println("failed to close client conn ", err)

            return

        }

        fmt.Println("closed client connection")

    }()



    ctx := context.Background()



    go func() {

        fmt.Println("waiting for items from list", listName)

        for {

            items, err := client.BRPop(ctx, 0, listName).Result()

            if err != nil {

                fmt.Println("unable to fetch item from list", err)

                continue

            }



            fmt.Println("processed item", items[1])

            time.Sleep(time.Duration(rand.Intn(5)+2) * time.Second)

        }

    }()



    exit := make(chan os.Signal, 1)

    signal.Notify(exit, os.Interrupt, syscall.SIGTERM)



    fmt.Println("press ctrl+c to exit...")

    <-exit

    fmt.Println("program exited")

}
```

-   First, we add a defer function to close the `redis.Client` connection at the end of the program.
    
-   Then a `goroutine` is started which uses `BRPop` to retrieve items from the Redis List in a blocking way.
    
-   Once an item is available in the List and received by the program, it is printed to the console.
    
-   There is an intentional `time.Sleep` (delay) added in the loop in order to simulate data processing.
    
-   Finally, the program is setup to respond to the `SIGTERM` signal and exit cleanly.
    

Save the `consumer.go` file.

## Prepare producer application

Go back to the top-level directory (`redis-keda-vultr`):

```
cd ..
```

### Initialize the project

Create a directory and switch to it:

```
mkdir redis-producer

cd redis-producer
```

Create a new Go module:

```
go mod init redis-producer
```

> This will create a new `go.mod` file

Create a new file `producer.go`:

```
touch producer.go
```

### Import libraries

To import required Go modules, add the following to `producer.go` file:

```
package main



import (

    "context"

    "crypto/tls"

    "fmt"

    "log"

    "math/rand"

    "os"

    "os/signal"

    "strconv"

    "syscall"

    "time"



    "github.com/go-redis/redis/v8"

)
```

### Add the `init` function

Add the code below to `producer.go` file:

```
var client *redis.Client

const redisURLTemplate = "rediss://%s:%s@%s:%s"

var listName string



func init() {

    redisHost := os.Getenv("REDIS_HOST")

    if redisHost == "" {

        log.Fatal("missing environment variable REDIS_HOST")

    }



    redisPort := os.Getenv("REDIS_PORT")

    if redisPort == "" {

        log.Fatal("missing environment variable REDIS_PORT")

    }



    redisUsername := os.Getenv("REDIS_USERNAME")

    if redisUsername == "" {

        log.Fatal("missing environment variable REDIS_USERNAME")

    }



    redisPassword := os.Getenv("REDIS_PASSWORD")

    if redisPassword == "" {

        log.Fatal("missing environment variable REDIS_PASSWORD")

    }



    redisURL := fmt.Sprintf(redisURLTemplate, redisUsername, redisPassword, redisHost, redisPort)

    fmt.Println("Redis URL", redisURL)



    opt, err := redis.ParseURL(redisURL)

    if err != nil {

        log.Fatal("invalid redis url", err)

    }



    opt.TLSConfig = &tls.Config{}



    client = redis.NewClient(opt)



    err = client.Ping(context.Background()).Err()



    if err != nil {

        log.Fatal("ping failed. could not connect", err)

    }

    fmt.Println("successfully connected to redis", redisURL)



    listName = os.Getenv("LIST_NAME")

    if listName == "" {

        log.Fatal("missing environment variable LIST_NAME")

    }

}
```

-   The `init` function reads the Redis connectivity information from environment variables. It uses `REDIS_HOST`, `REDIS_PORT`, `REDIS_USERNAME` and `REDIS_PASSWORD` to store Vultr Managed Redis database `host`, `port`, `username` and `password` respectively.
    
-   It uses the above data to create a Redis URL and passes it into `redis.ParseURL` to get a `redis.Option` object that is further used in creating a `redis.Client` instance.
    
-   `Ping` function is used to verify successful connectivity with Redis. If connectivity fails, the program exits with an error message.
    
-   Finally, the name of the Redis List to be used is also retrieved from the `LIST_NAME` environment variable.
    

### Add the `main` function

Add the code below to `producer.go` file:

```
func main() {



    defer func() {

        err := client.Close()

        if err != nil {

            fmt.Println("failed to close client conn ", err)

            return

        }

        fmt.Println("closed client connection")

    }()



    ctx := context.Background()



    go func() {

        pipe := client.Pipeline()

        num := 50



        for {

            log.Println("sending", num, "items into redis list", listName)



            for i := 1; i <= num; i++ {

                err := pipe.LPush(ctx, listName, "message-"+strconv.Itoa(rand.Intn(50))).Err()

                if err != nil {

                    fmt.Println("unable to pipe data to redis list", err)

                    continue

                }

            }



            _, err := pipe.Exec(ctx)

            if err != nil {

                fmt.Println("unable to use pipeline to send data to redis list", err)

            }



            time.Sleep(2 * time.Second)

        }

    }()



    exit := make(chan os.Signal, 1)

    signal.Notify(exit, os.Interrupt, syscall.SIGTERM)



    fmt.Println("press ctrl+c to exit...")

    <-exit

    fmt.Println("program exited")

}
```

Save the `producer.go` file.

-   First, we add a defer function to close the `redis.Client` connection at the end of the program.
    
-   Then, a `goroutine` is started to add items to the Redis List using `LPush`.
    
-   Items are added in batches of `50`.
    
-   Finally, the program is setup to respond to the `SIGTERM` signal and exit cleanly.
    

## Deploy consumer application

Go back to the consumer project directory:

```
cd ../redis-consumer
```

### Add Dockerfile

Create a file named `Dockerfile`:

```
touch Dockerfile
```

Enter the below contents in `Dockerfile`:

```
FROM golang:1.18-buster AS build



WORKDIR /app

COPY go.mod ./

COPY go.sum ./



RUN go mod download



COPY consumer.go ./

RUN go build -o /redis-go-app



FROM gcr.io/distroless/base-debian10

WORKDIR /

COPY --from=build /redis-go-app /redis-go-app

EXPOSE 8080

USER nonroot:nonroot

ENTRYPOINT ["/redis-go-app"]
```

-   This is a multi-stage `Dockerfile` where we use the `golang:1.18-buster` as the base image of the first stage.
    
-   Copy the application files, run `go mod download` and build the application binary.
    
-   In the second stage, you use `gcr.io/distroless/base-debian10` as the base image.
    
-   You copy the binary from the first stage and use the `ENTRYPOINT` to run the application.
    

### Build and push Docker image

Pull Go modules (this will create `go.sum` file):

```
go mod tidy
```

Login to Docker:

```
docker login
```

Build image:

```
docker build -t <enter repo name> .

# example - docker build -t mydockerlogin/redis-consumer-app .
```

Push image to Docker:

```
docker push <enter repo name>

# example - docker push mydockerlogin/redis-consumer-app
```

### Deploy to Kubernetes

Create a file `consumer.yaml`:

```
touch consumer.yaml
```

Enter below code in `consumer.yaml`. Make sure to replace the required attributes with the `host`, `port`, `username`, password for Vultr Redis Managed database.

```
apiVersion: apps/v1

kind: Deployment

metadata:

name: redis-consumer

spec:

  replicas: 1

  selector:

    matchLabels:

      app: redis-consumer

  template:

    metadata:

      labels:

        app: redis-consumer

    spec:

    containers:

        - name: redis-consumer

          image: <replace with docker image>

          imagePullPolicy: Always

          env:

            - name: REDIS_USERNAME

              value: <replace with username for vultr managed redis database>

            - name: REDIS_PASSWORD

              value: <replace with password for vultr managed redis database>

            - name: REDIS_PORT

              value: "16752"

            - name: REDIS_HOST

              value: <replace with host name for vultr managed redis database>

            - name: LIST_NAME

              value: mylist
```

In addition to the regular `Deployment` related manifest attributes, the key thing to note here is the usage of specific environment variables in the container. You use `REDIS_USERNAME`, `REDIS_PASSWORD`, `REDIS_PORT` and `REDIS_HOST` to point to the Vultr Redis Managed database. The values of these environemnt variables will be used by the `KEDA` `ScaledObject`.

To deploy the consumer application:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> apply -f consumer.yaml
```

Wait for it go into `Running` status:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get pod -l=app=redis-consumer



#output



NAME                              READY   STATUS    RESTARTS   AGE

redis-consumer-746f5ddf75-tzmxs   1/1     Running   0          12s
```

To check application logs:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> logs -f $(kubectl --kubeconfig=<path to VKE kubeconfig file> get pod -l=app=redis-consumer -o jsonpath='{.items[0].metadata.name}')
```

## Deploy KEDA scaled object

Go back to the top-level directory (`redis-keda-vultr`):

```
cd ..
```

### Create `ScaledObject` manifest:

Create file named `scaled-object.yaml`:

```
touch scaled-object.yaml
```

Enter below content in `scaled-object.yaml`:

```
apiVersion: keda.sh/v1alpha1

kind: ScaledObject

metadata:

name: redis-scaledobject

spec:

scaleTargetRef:

    name: redis-consumer

maxReplicaCount: 5

triggers:

- type: redis

    metadata:

        hostFromEnv: REDIS_HOST

        portFromEnv: REDIS_PORT

        usernameFromEnv: REDIS_USERNAME

        passwordFromEnv: REDIS_PASSWORD

        listName: mylist

        listLength: "50"

        enableTLS: "true"
```

The `ScaledObject` CRD is used to configure the triggers and define the way how `KEDA` should scale the application.

-   Since you are dealing with Redis List, you have specified the `redis` trigger `type` (using the Redis Scaler).
    
-   The name of the target `Deployment` this `ScaledObject` will scale is specificed by `scaleTargetRef`.
    
-   `maxReplicaCount` has been set to **5** - this means that the the target `Deployment` will scale to maximun of `5` `Pod`s.
    
-   In `metadata`, you specify the environment variables from where to pick Redis connectivity information and also specify secure `TLS` connectivity with `enableTLS: "true"`
    

### Deploy `ScaledObject` to Kubernetes:

To deploy `KEDA` `ScaledObject`:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> apply -f scaled-object.yaml
```

## Deploy producer application

Change to the producer project directory:

```
cd redis-producer
```

### Add Dockerfile

Create a file named `Dockerfile`:

```
touch Dockerfile
```

Enter the below contents in `Dockerfile`:

```
FROM golang:1.18-buster AS build



WORKDIR /app

COPY go.mod ./

COPY go.sum ./



RUN go mod download



COPY producer.go ./

RUN go build -o /redis-go-app



FROM gcr.io/distroless/base-debian10

WORKDIR /

COPY --from=build /redis-go-app /redis-go-app

EXPOSE 8080

USER nonroot:nonroot

ENTRYPOINT ["/redis-go-app"]
```

-   This is a multi-stage `Dockerfile` where you use the `golang:1.18-buster` as the base image of the first stage.
    
-   Copy the application files, run `go mod download` and build the application binary.
    
-   In the second stage, you use `gcr.io/distroless/base-debian10` as the base image.
    
-   You copy the binary from the first stage and use the `ENTRYPOINT` to run the application.
    

### Build and push Docker image

Pull dependent Go modules (this will create `go.sum` file):

```
go mod tidy
```

Login to Docker:

```
docker login
```

Build the image:

```
docker build -t <enter repo name>

# example - docker build -t mydockerlogin/redis-producer-app .
```

Push to image to Docker:

```
docker push <enter repo name>

# example - docker push mydockerlogin/redis-producer-app
```

### Deploy to Kubernetes

Create a file `producer.yaml`:

```
touch producer.yaml
```

Enter below code in `producer.yaml`. Make sure to replace the required attributes with the `host`, `port`, `username` and `password` for Vultr Redis Managed database.

```
apiVersion: apps/v1

kind: Deployment

metadata:

name: redis-producer

spec:

replicas: 1

selector:

    matchLabels:

    app: redis-producer

template:

    metadata:

    labels:

        app: redis-producer

    spec:

    containers:

        - name: redis-producer

        image: <replace with docker image>

            imagePullPolicy: Always

            env:

                - name: REDIS_USERNAME

                value: <replace with username for vultr managed redis database>

                - name: REDIS_PASSWORD

                value: <replace with password for vultr managed redis database>

                - name: REDIS_PORT

                value: "16752"

                - name: REDIS_HOST

                value: <replace with host name for vultr managed redis database>

                - name: LIST_NAME

                value: mylist
```

In addition to the regular `Deployment` related manifest attributes, the key thing to note here is the usage of specific environment variables in the container. You use `REDIS_USERNAME`, `REDIS_PASSWORD`, `REDIS_PORT` and `REDIS_HOST` to point to the Vultr Redis Managed database. The values of these environemnt variables will be used by the `KEDA` `ScaledObject`.

To deploy the producer application:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> apply -f producer.yaml
```

Wait for it to go into `Running` state:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get pod -l=app=redis-producer



#output



NAME                              READY   STATUS    RESTARTS   AGE

redis-producer-78fc5d84b5-cls5w   1/1     Running   0          7s
```

To check application logs:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> logs -f $(kubectl --kubeconfig=<path to VKE kubeconfig file> get pod -l=app=redis-producer -o jsonpath='{.items[0].metadata.name}')
```

## Observe `Pod` auto-scaling

Start observing the `Pod` activity:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get pods -w
```

Wait for a while. After sometime, you will witness number of `Pod`s increasing. You will see an output similar to this (`Pod` names will differ in your case):

```
NAME                              READY   STATUS    RESTARTS   AGE

redis-consumer-746f5ddf75-8xs8s   1/1     Running   0          4m24s

redis-consumer-746f5ddf75-dgczp   1/1     Running   0          4m39s

redis-consumer-746f5ddf75-ksbp7   1/1     Running   0          4m39s

redis-consumer-746f5ddf75-qhgd2   1/1     Running   0          4m39s

redis-consumer-746f5ddf75-tzmxs   1/1     Running   0          26m

redis-producer-78fc5d84b5-cls5w   1/1     Running   0          51s
```

This is because `KEDA` has _scaled up_ the application. Due to the large number of items in the Redis List, it's length (backlog) has increased and gone beyond the threshold you defined in the `scaled-object.yaml` (which was `50`). Thus, additional consumer `Pod`s were created to divide the processing load. Please note that the number of consumer `Pod`s will not go beyond **five**, because that's what you specified in `maxReplicaCount` in `scaled-object.yaml`.

Verify the consumer `Deployment` status:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get deployment/redis-consumer



#output



NAME             READY   UP-TO-DATE   AVAILABLE   AGE

redis-consumer   5/5     5            5           27m
```

Verify the Horizontal Pod Auto-scaler status:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> get hpa



#output



NAME                          REFERENCE                   TARGETS             MINPODS   MAXPODS   REPLICAS   AGE

keda-hpa-redis-scaledobject   Deployment/redis-consumer   1205400m/50 (avg)   1         5         5          6m21s
```

After sometime, stop the producer application:

```
kubectl --kubeconfig=<path to VKE kubeconfig file> delete -f producer.yaml
```

Wait for sometime. You will notice that the number of `Pod`s have decreased. This is because the producer load has decreased and consumer are able to catch up with the processing. Hence, the size of the target Redis List is within the threshold and the additional `Pod`s that were spawned were gradually terminated.

After you have completed the tutorial in this article, you can delete the Vultr Redis Managed Database and Vultr Kubernetes Engine cluster.

## Conclusion

This article showed how it's possible to auto-scale Redis List based consumer applications using `KEDA` based on the length of the Redis List. You deployed the applications on Vultr Kubernetes Engine (VKE) and used Vultr Managed Redis as the database.

You can also learn more in the following documentation:

-   [Vultr Kubernetes Engine (VKE) Reference Guide](https://www.vultr.com/docs/vultr-kubernetes-engine/)
    
-   [Vultr Kubernetes Engine (VKE) Changelog](https://www.vultr.com/docs/vultr-kubernetes-engine-changelog/)
    
-   [How to Install Jenkins on Vultr Kubernetes Engine](https://www.vultr.com/docs/how-to-install-jenkins-on-vultr-kubernetes-engine/)
    
-   [How to Monitor Your VKE Cluster with tobs](https://www.vultr.com/docs/how-to-monitor-your-vke-cluster-with-tobs/)
    
-   [Vultr Managed Databases Quickstart](https://www.vultr.com/docs/vultr-managed-databases/)
    
-   [Redis Managed Database Guide](https://www.vultr.com/docs/redis-managed-database-guide)
    
-   [How to Securely Connect to Redis with TLS/SSL in Go, NodeJS, PHP, Python, and redis-cli](https://www.vultr.com/docs/how-to-securely-connect-to-redis-with-tls-ssl-in-go-nodejs-php-python-and-redis-cli/)