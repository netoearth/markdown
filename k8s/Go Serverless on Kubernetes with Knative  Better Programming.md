## How Knative is the best of both worlds

![](https://miro.medium.com/max/700/1*BM7ft0C4Ve3qsipgUCJwig.png)

If you’re already using Kubernetes, you’ve probably heard about serverless. While both platforms are scalable, serverless goes the extra mile by providing developers with running code without worrying about infrastructure and saves on infra costs by virtually scaling your application instances from zero.

Kubernetes, on the other hand, provides its advantages with zero limitations, following a traditional hosting model and advanced traffic management techniques that help you do things — like blue-green deployments and A/B testing.

Knative is an attempt to create the best of the two worlds. As an open-source cloud-native platform, it enables you to run your serverless workloads on Kubernetes, providing all Kubernetes capabilities, plus the simplicity and flexibility of serverless.

So while your developers can focus on writing code and deploying the container to Kubernetes with a single command, Knative can manage the application by taking care of the networking details, the autoscaling to zero, and the revision tracking.

It also takes care of allowing developers to write loosely coupled code with its eventing framework that provides the universal subscription, delivery, and management of events. That means you can declare event connectivity, and your apps can subscribe to specific data streams.

Spearheaded by Google, this open-source platform has been adopted by the Cloud Native Computing Foundation, which means you don’t experience vendor lock-in — a considerable limitation of the current cloud-based serverless FaaS solutions. You can run Knative on any Kubernetes cluster.

## Knative Audience

Knative helps a variety of audiences, each having a particular area of expertise and responsibility.

![](https://miro.medium.com/max/700/1*HIhgwe-PtGUGYeO6dRurhQ.png)

Image source: [Knative docs](https://knative.dev/docs/images/knative-audience.svg)

So while your operators can focus on managing your Kubernetes cluster and installing and maintaining Knative instances using kubectl, your developers can focus on building and deploying applications using the Knative API interface.

That gives any organization immense power, as now different teams can focus on their area of expertise without stepping on each other’s shoes.

## Why You Should Use Knative

Most organizations that run Kubernetes have a complex ops process in managing deployments and running workloads. This results in developers looking at details they’re not supposed to worry too much about. Developers should focus on writing code, rather than worrying about builds and deployments.

Kubeless helps simplify the way developers run their code on Kubernetes without learning too much about the internal nitty-gritty of Kubernetes.

A Kubernetes cluster occupies infrastructure resources, as it requires all applications to have at least one container running. Knative manages that aspect for you and takes care of automatically scaling your containers within the cluster from zero. That allows Kubernetes administrators to pack a lot of applications into a single cluster.

If you have multiple applications having different peak times or if you have a cluster that has worker-node autoscaling, you can benefit a lot from this in your idle time.

It’s a goldmine for microservices applications that may or may not need microservices running at a particular time. It fosters better resource utilization, and you can do more with limited resources.

Apart from that, it integrates quite well with the Eventing engine, and it provides you with a seamless experience in designing decoupled systems. Your application code can remain completely free of any endpoint configuration, and you can publish and subscribe to events by declaring configurations in the Kubernetes layer. That’s a considerable advantage for complex microservice applications.

## How Knative Works

Knative exposes the `kn` API interface using Kubernetes operators and CRDs. Using this, you can deploy your applications with the command line. In the background, Knative will create all of the Kubernetes resources (such as deployment, services, ingress, etc.) required to run your applications without you having to worry about it.

Bear in mind Knative doesn’t create the pods immediately. Instead, it provides a virtual endpoint for your applications and listens on them. If there’s a hit to those endpoints, it spins up the pods required to serve the requests. That allows Knative to scale your applications from zero to the required number of instances.

![](https://miro.medium.com/max/540/1*B6PXYFlpWOYNBDU-lpic4g.png)

Image source: [Knative serving](https://github.com/knative/serving/raw/master/docs/spec/images/object_model.png)

Knative provides application endpoints using a custom domain in the format `[app-name].[namespace].[custom-domain]`. That helps in identifying an application uniquely. It’s similar to how Kubernetes handles services, but you need to create A records of the custom domain to point to the Istio ingress gateway. Istio manages all of the traffic that flows through your cluster in the background.

Knative is an amalgamation of a lot of CNCF and open-source products — such as Kubernetes, Istio, Prometheus, Grafana, and event-streaming engines, such as Kafka and Google Pub/Sub.

## Running a “Hello, World!” Application

Let’s now run our first “Hello, World!” application on Knative to understand how simple it is to deploy once things are ready.

Let’s use a sample “Hello, World!” Go application for this demo. That’s a simple REST API that returns `Hello $TARGET`, where `$TARGET` is an environment variable you can set in the container.

Run the below to get started:

```
kubectl get podNo resources found in default namespace.
```

Let’s trigger the `helloworld` service.

```
$ curl http://helloworld-go.default.34.71.125.175.xip.ioHello World!
```

And we get a response after a while. Let’s look at the pods.

```
$ kubectl get podNAME                                               READY   STATUS    RESTARTS   AGEhelloworld-go-zglmv-1-deployment-6d4b7fb4f-ctz86   2/2     Running   0          50s
```

So as you can see, Knative spun up a pod in the background when we made the first hit. So we literally scaled from zero.

If we give it a while, we’ll see the pod start terminating. Let’s watch the pod.

```
$ kubectl get pod -wNAME                                               READY   STATUS    RESTARTS   AGEhelloworld-go-zglmv-1-deployment-6d4b7fb4f-d9ks6   2/2     Running   0          7shelloworld-go-zglmv-1-deployment-6d4b7fb4f-d9ks6   2/2     Terminating   0          67shelloworld-go-zglmv-1-deployment-6d4b7fb4f-d9ks6   1/2     Terminating   0          87s
```

That shows Knative is managing the pods according to the requirement. While the first request is slow, as Knative is creating workloads to process it, subsequent requests will be faster. You can fine-tune the time to spin down the pods based on your requirements or if you have a tighter SLA.

Let’s take this to the next level. If you look at the annotations, we had limited each pod to process a maximum of 10 concurrent requests. So what happens if we load our functions beyond that? Let’s find out.

We’ll use the `hey` utility to load the application. The following command sends 50 concurrent requests to the endpoint for 30 seconds.

Let’s look at the pods now.

As we see, Knative is scaling the pods as the load of the function increases, and it’ll spin it down when the load is no longer there.

## Conclusion

Knative combines the best possible aspects of both serverless and Kubernetes and is slowly heading toward a standard way of implementing FaaS. As the CNCF adopts it and as it continues to see significant interest among techies, we’ll soon find cloud providers implementing it into their serverless offerings.

Thanks for reading — I hope you enjoyed the article!