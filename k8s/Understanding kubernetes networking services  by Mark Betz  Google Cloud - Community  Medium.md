In the [first post](https://medium.com/@betz.mark/understanding-kubernetes-networking-pods-7117dd28727) of this series I looked at how [kubernetes](https://kubernetes.io/) employs a combination of virtual network devices and routing rules to allow a pod running on one cluster node to communicate with a pod running on another, as long as the sender knows the receiver’s pod network IP address. If you aren’t already familiar with how pods communicate then it’s worth a read before continuing. Pod networking in a cluster is neat stuff, but by itself it is insufficient to enable the creation of durable systems. That’s because pods in kubernetes are ephemeral. You can use a pod IP address as an endpoint but there is no guarantee that the address won’t change the next time the pod is recreated, which might happen for any number of reasons.

You will probably have recognized this as an old problem, and it has a standard solution: run the traffic through a reverse-proxy/load balancer. Clients connect to the proxy and the proxy is responsible for maintaining a list of healthy servers to forward requests to. This implies a few requirements for the proxy: it must itself be durable and resistant to failure; it must have a list of servers it can forward to; and it must have some way of knowing if a particular server is healthy and able to respond to requests. The kubernetes designers solved this problem in an elegant way that builds on the basic capabilities of the platform to deliver on all three of those requirements, and it starts with a resource type called a service.

![](https://miro.medium.com/max/1400/1*pIHETdF_CVLor_Qws9XUAA.png)

## Services

In the first post I showed a hypothetical cluster with two server pods and described how they are able to communicate across nodes. Here I want to build on that example to describe how a kubernetes service enables load balancing across a set of server pods, allowing client pods to operate independently and durably. To create server pods we can use a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) such as this:

This deployment creates two very simple http server pods that respond on port 8080 with the hostname of the pod they are running in. After creating this deployment using `**kubectl apply**` we can see that the pods are running in the cluster, and we can also query to see what their pod network addresses are:

```
$ kubectl apply -f test-deployment.yamldeployment "service-test" created$ kubectl get podsservice-test-6ffd9ddbbf-kf4j2    1/1    Running    0    15sservice-test-6ffd9ddbbf-qs2j6    1/1    Running    0    15s$ kubectl get pods --selector=app=service_test_pod -o jsonpath='{.items[*].status.podIP}'10.0.1.2 10.0.2.2
```

We can demonstrate that the pod network is operating by creating a simple client pod to make a request, and then viewing the output.

After this pod is created the command will run to completion, the pod will enter the “completed” state and the output can then be retrieved with `**kubectl logs**`:

```
$ kubectl logs service-test-client1HTTP/1.0 200 OK<!-- blah --><p>Hello from service-test-6ffd9ddbbf-kf4j2</p>
```

Nothing in this example shows which node the client pod was created on, but regardless of where it ran in the cluster it would be able to reach the server pod and get a response back thanks to the pod network. However if the server pod were to die and be restarted, or be rescheduled to a different node, its IP would almost certainly change and the client would break. We avoid this by creating a service.

A [service](https://kubernetes.io/docs/concepts/services-networking/service/) is a type of kubernetes resource that causes a proxy to be configured to forward requests to a set of pods. The set of pods that will receive traffic is determined by the selector, which matches labels assigned to the pods when they were created. Once the service is created we can see that it has been assigned an IP address and will accept requests on port 80.

```
$ kubectl get service service-testNAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGEservice-test   10.3.241.152   <none>        80/TCP    11s
```

Requests can be sent to the service IP directly but it would be better to use a hostname that resolves to the IP address. Fortunately kubernetes provides an internal cluster DNS that resolves the service name, and with a slight change to the client pod we can make use of it:

After this pod runs to completion the output shows that the service forwarded the request to one of the server pods.

```
$ kubectl logs service-test-client2HTTP/1.0 200 OK<!-- blah --><p>Hello from service-test-6ffd9ddbbf-kf4j2</p>
```

You can continue to run the client pod and you’ll see responses from both server pods with each getting approximately 50% of the requests. If your goal is to understand how this actually works then a good place to start is with that IP address that our service was assigned.

## The service network

The IP that the test service was assigned represents an address on a network, and you might have noted that the network is not the same as the one the pods are on.

```
thing        IP               network-----        --               -------pod1         10.0.1.2         10.0.0.0/14pod2         10.0.2.2         10.0.0.0/14service      10.3.241.152     10.3.240.0/20
```

It’s also not the same as the private network the nodes are on, which will become clearer below. In the first post I noted that the pod network address range is not exposed via `**kubectl**` and so you need to use a provider-specific command to retrieve this cluster property. The same is true of the service network address range. If you’re running in Google Container Engine you can do this:

```
$ gcloud container clusters describe test | grep servicesIpv4CidrservicesIpv4Cidr: 10.3.240.0/20
```

The network specified by this address space is called the “service network.” Every service that is of type “ClusterIP” will be assigned an IP address on this network. There are other types of services, and I’ll talk about a couple of them in the next post on ingress, but ClusterIP is the default and it means “the service will be assigned an IP address reachable from any pod in the cluster.” You can see the type of a service by running the `**kubectl describe services**` command with the service name.

```
$ kubectl describe services service-testName:                   service-testNamespace:              defaultLabels:                 <none>Selector:               app=service_test_podType:                   ClusterIPIP:                     10.3.241.152Port:                   http    80/TCPEndpoints:              10.0.1.2:8080,10.0.2.2:8080Session Affinity:       NoneEvents:                 <none>
```

Like the pod network the service network is virtual, but it differs from the pod network in some interesting ways. Consider the pod network address range `**10.0.0.0/14**`. If you go looking on the hosts that make up the nodes in your cluster, listing bridges and interfaces you’re going to see actual devices configured with addresses on this network. Those are the virtual ethernet interfaces for each pod and the bridges that connect them to each other and the outside world.

Now look at the service network `**10.3.240.0/20**`. You can `**ifconfig**` to your heart’s delight and you will not find any devices configured with addresses on this network. You can examine the routing rules at the gateway that connects all the nodes and you won’t find any routes for this network. The service network does not exist, at least not as connected interfaces. And yet as we saw above when we issued a request to an IP on this network somehow that request made it to our server pod running on the pod network. How did that happen? Let’s follow a packet and see.

Imagine that the commands we ran above created the following pods in a test cluster:

![](https://miro.medium.com/max/1400/1*7Txw4Xy_sAgr6iE8zrtucA.png)

Here we have two nodes, the gateway connecting them (that also has the routing rules for the pod network), and three pods: the client pod on node 1, a server pod also on node 1, and another server pod on node 2. The client makes an http request to the service using the DNS name `**service-test**`. The cluster DNS system resolves that name to the service cluster IP `**10.3.241.152**`, and the client pod ends up creating an http request that results in some packets being sent with that IP in the destination field.

IP networks are typically configured with routes such that when an interface cannot deliver a packet to its destination because no device with that specified address exists locally it forwards the packet on to its upstream gateway. So the first interface that sees the packets in this example is the virtual ethernet interface inside the client pod. That interface is on the pod network `**10.0.0.0/14**` and doesn’t know any devices with address `**10.3.241.152**`, so it forwards the packet to its gateway which is the bridge cbr0. Bridges are pretty dumb and just pass traffic back and forth so the bridge sends the packet on to the host/node ethernet interface.

![](https://miro.medium.com/max/1400/1*1f98Bf_2yoggG_skjqTC6g.png)

The host/node ethernet interface in this example is on the network `**10.100.0.0/24**` and it doesn’t know any devices with the address `**10.3.241.152**` either, so again what would normally happen is the packet would be forwarded out to this interface’s gateway, the top level router shown in the drawing. Instead what actually happens is that the packet is snagged in flight and redirected to one of the live server pods.

![](https://miro.medium.com/max/1400/1*UetnYP8uE05GAqQD0tbtBQ.png)

When I first started working with kubernetes three years ago what was happening in the diagram above seemed pretty magical. Somehow my clients were able to connect to an address with no interface associated with it and those packets popped out at the right place in the cluster. As I later learned the answer to this mystery is a piece of software called kube-proxy.

## kube-proxy

Like everything in kubernetes a service is just a resource, a record in a central database, that describes how to configure some bit of software to do something. In fact a service affects the configuration and behavior of several components in the cluster, but the one that’s important here, the one that makes the magic described above happen, is kube-proxy. Many of you will have a general idea of what this component does based on the name, but there are some things about kube-proxy that make it quite different from a typical reverse-proxy like [haproxy](http://www.haproxy.org/) or [linkerd](https://linkerd.io/).

The general behavior of a proxy is to pass traffic between clients and servers over two open connections. Clients connect inbound to a service port, and the proxy connects outbound to a server. Since all proxies of this kind run in user space this means that packets are marshaled into user space and back to kernel space on every trip through the proxy. Initially kube-proxy was implemented as just such a user-space proxy, but with a twist. A proxy needs an interface, both to listen on for client connections, and to use to connect to back-end servers. The only interfaces available on a node are either a) the host’s ethernet interface; or b) the virtual ethernet interfaces on the pod network.

Why not use an address on one of those networks? I don’t have any inside knowledge but I imagine that it became clear pretty early in the project that doing so would have complicated the routing rules for those networks, which are designed to serve the needs of pods and nodes, both of which are ephemeral entities in the cluster. Services clearly needed their own, stable, non-conflicting network address space, and a system of virtual IPs makes the most sense. However, as we noted there are no actual devices on this network. You can make use of a pretend network in routing rules, firewall filters, etc., but you can’t actually listen on a port or open a connection through an interface that doesn’t exist.

Kubernetes gets around this using a feature of the linux kernel called netfilter, and a user space interface to it called iptables. There’s not enough room in this already-long post to go into exactly how this works. If you want to read more about it the [netfilter page](http://www.netfilter.org/) is a good place to start. Here’s the tl;dr: netfilter is a rules-based packet processing engine. It runs in kernel space and gets a look at every packet at various points in its life cycle. It matches packets against rules and when it finds a rule that matches it takes the specified action. Among the many actions it can take is redirecting the packet to another destination. That’s right, netfilter is a kernel space proxy. The following illustrates the role netfilter plays when kube-proxy is running as a user space proxy.

![](https://miro.medium.com/max/1400/1*iezVQZMVEqzic_yfREO8Jw.png)

In this mode kube-proxy opens a port (10400 in the example above) on the local host interface to listen for requests to the test-service, inserts netfilter rules to reroute packets destined for the service IP to its own port, and forwards those requests to a pod on port 8080. That is how a request to `**10.3.241.152:80**` magically becomes a request to `**10.0.2.2:8080.**` Given the capabilities of netfilter all that’s required to make this all work for any service is for kube-proxy to open a port and insert the correct netfilter rules for that service, which it does in response to notifications from the master api server of changes in the cluster.

There’s one more little twist to the tale. I mentioned above that user space proxying is expensive due to marshaling packets. In kubernetes 1.2 kube-proxy gained the ability to run in iptables mode. In this mode kube-proxy mostly ceases to be a proxy for inter-cluster connections, and instead delegates to netfilter the work of detecting packets bound for service IPs and redirecting them to pods, all of which happens in kernel space. In this mode kube-proxy’s job is more or less limited to keeping netfilter rules in sync.

![](https://miro.medium.com/max/1400/1*4XyIJ5Tdvs8f6zhLwdVt3Q.png)

To wrap up let’s compare everything described above to the requirements set out for a reliable proxy at the beginning of the post. Is the services proxy system durable? By default kube-proxy runs as a systemd unit, so it will be restarted if it fails. In Google Container Engine it runs as a pod controlled by a [daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/). This will be the future default, possibly with version 1.9. As a user space proxy kube-proxy still represents a single point of connection failure. When running in iptables mode the system is highly durable from the perspective of local pods attempting connections, since if the node is up so is netfilter.

Is the service proxy aware of healthy server pods that can handle requests? As mentioned above kube-proxy listens to the master api server for changes in the cluster, which includes changes to services and endpoints. As it receives updates it uses iptables to keep the netfilter rules in sync. When a new service is created and its endpoints are populated kube-proxy gets the notification and creates the necessary rules. Similarly it removes rules when services are deleted. Health checks against endpoints are performed by the kubelet, another component that runs on every node, and when unhealthy endpoints are found kubelet notifies kube-proxy via the api server and netfilter rules are edited to remove this endpoint until it becomes healthy again.

All of this adds up to a highly-available cluster-wide facility for proxying requests between pods while allowing pods themselves to come and go as the needs of the cluster change. The system is not without its downsides, however. The most basic one is that it only works as described for requests that originate inside the cluster, i.e. requests from one pod to another. Another is a consequence of the way the netfilter rules work: for requests arriving from outside the cluster the rules obfuscate the origin IP. This has been a source of some debate and solutions are under active consideration. I’ll look more closely at both of these issues when we discuss ingress in the final post of this series.

\[Update\] Part 3 has been posted:

[Understanding kubernetes networking: ingress](https://medium.com/@betz.mark/understanding-kubernetes-networking-ingress-1bc341c84078)