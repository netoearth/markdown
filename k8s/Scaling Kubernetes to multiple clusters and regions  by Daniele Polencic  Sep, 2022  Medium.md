_TL;DR: In this tutorial, you will learn how to create, connect and operate three Kubernetes clusters in different regions: North America, Europe and South East Asia._

One interesting challenge with Kubernetes is deploying workloads across several regions.

**While you can technically have a cluster with several nodes located in different regions, this is generally regarded as something you should avoid due to the extra latency.**

Another popular alternative is to deploy a cluster for each region and find a way to orchestrate them.

![Placing nodes in a multicluster setup](https://miro.medium.com/max/1400/0*NMWJCLMIa6IVIpQQ.png)

But before discussing solutions, let’s look at the challenges of a multicluster & multi-cloud setup.

When you orchestrate several clusters, you have to face the following issues:

-   _How do you decide how to split the workloads?_
-   _How does the networking work across regions?_
-   _What should I do with stateful apps/data?_

![Challenges of running a Kubernetes multicluster setup](https://miro.medium.com/max/1400/0*Dtrpa2aKWu91RbgS.png)

_Let’s try to answer some of those questions._

To tackle the first (scheduling workloads), I used [Karmada.](https://karmada.io/)

**With Karmada, you can create deployments with kubectl and distribute them across several clusters using policies.**

Karmada takes care of propagating them to the correct cluster.

The project is similar (in spirit) to [kubefed.](https://github.com/kubernetes-sigs/kubefed)

![Goegraphically distributed Kubernetes cluster with Karmada](https://miro.medium.com/max/1400/0*YTLIHT7gvq0kN4YS.png)

**Karmada uses a Kubernetes cluster as the manager and creates a second control plane that is multicluster aware.**

This is particularly convenient because kubectl “just works” and is now multicluster aware.

**In other words, you can keep using kubectl, but all the commands can apply resources across clusters and aggregate data.**

Each cluster has an agent that issues commands to the cluster’s API server.

The Karmada controller manager uses those agents to sync and dispatch commands.

![Karmada client-server architecture and control plane](https://miro.medium.com/max/1400/0*YNIdOKLuzDPdTs02.png)

[Karmada uses policies to decide how to distribute your workloads.](https://karmada.io/docs/userguide/scheduling/resource-propagating)

You could have policies to have a deployment equally distributed across regions.

Or you could place your pods in a single region.

![Orchestrating workloads across several regions and clouds with Karmada policies](https://miro.medium.com/max/1400/0*TzFZgVfZLMxUi6g0.png)

**Karmada is essentially a multicluster orchestrator but doesn’t provide any mechanism to connect the clusters’ networks.**

Traffic routed to a region will always reach pods from that region.

![Traffic routed to a cluster will always reach pods from that cluster](https://miro.medium.com/max/1400/0*PkEbFzUHLDohMXYR.png)

But you can use a service mesh like Istio to create a network that spans several clusters.

**Istio can discover other instances in other clusters and forward the traffic to other clusters.**

![Istio multi-cluster setup](https://miro.medium.com/max/1400/0*EBlqWQFfOZdWt_M0.png)

_But how does the traffic routing work?_

For every app in your cluster, Istio injects a sidecar proxy.

All traffic from and to the app goes through the proxy.

**The Istio control plane can configure the proxy on the fly and apply routing policies.**

![Architecture of a service mesh with proxy side cars](https://miro.medium.com/max/1400/0*OmXq0BuFaQjofckh.png)

**In a multicluster setup, Istio instances share endpoints.**

When a request is issued, the traffic is intercepted by the proxy sidecar and forwarded to one of the endpoints amongst all endpoints in all clusters.

![Kubernetes endpoints are shared so that traffic can be forwarded from one cluster to the other](https://miro.medium.com/max/1400/0*MJupoNLda8TJkWbI.png)

[Since Istio’s traffic routing rules let you easily control the flow of traffic and API calls between services,](https://istio.io/latest/docs/concepts/traffic-management/) you can have traffic reaching a single region even if pods are deployed in each region.

Or you could create rules to shift traffic from one region to another.

![Multi cluster traffic management with Istio](https://miro.medium.com/max/1400/0*N0Jx7cPO8zkbf1tk.png)

_Nice in theory, but does it work in practice?_

I built a proof of concept with Terraform so that you can recreate it in 5 clicks here: [https://github.com/learnk8s/multi-cluster](https://github.com/learnk8s/multi-cluster)

_And here’s a demo of it._

![Multi cluster Kubernetes setup](https://miro.medium.com/max/1400/0*AJr6Depdnp3MEwjS.gif)

I also installed [Kiali to visualise the traffic flowing in the clusters in real time.](https://kiali.io/docs/features/multi-cluster/)

![Multi cluster Kiali demo](https://miro.medium.com/max/1400/0*Q0iw2-XJve3nZSxd.gif)

If you wish to see this in action, [you can watch my demo here.](https://event.on24.com/wcc/r/3919843/B94BC548B8902DD70F5EB512A308BB41?partnerref=learnk8s)

And finally, if you’ve enjoyed this thread, you might also like the Kubernetes workshops that we run at Learnk8s [https://learnk8s.io/training](https://learnk8s.io/training) or this collection of past Twitter threads [https://twitter.com/danielepolencic/status/1298543151901155330](https://twitter.com/danielepolencic/status/1298543151901155330)

Until next time!