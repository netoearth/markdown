## Kubernetes Ingress Overview

Kubernetes Ingress is an API object that manages external users’ access to the services within a Kubernetes cluster. Through the use of routing rules, access to the cluster can be configured, which are defined on the Ingress resource. Typically, HTTPS/HTTP routing protocols are used to facilitate the routing.

![Introduction_to_Kubernetes_Ingress1.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt1d65de5b03a51304/62c418a46a8fb133b0ff58c8/Introduction_to_Kubernetes_Ingress1.jpg)

Ingress provides a single entry point for the cluster, allowing for easier management of applications and the troubleshooting of any routing issues.

Some of the use cases for Ingress include load balancing traffic, SSL (secure sockets layer) termination, and name-based virtual hosting.

Instead of creating specialized load balancers or manually exposing services within the node, Ingress allows you to configure routing rules to manage inbound traffic to the cluster. This option greatly simplifies the production environment.

Kubernetes Ingress is comprised of two main components:

-   **Ingress API object:** it identifies the services that needs to be exposed for external users to access. It consists of the routing rules.
-   **Ingress Controller:** it is responsible for executing and implementing Ingress, usually with a load balancer. The load balancer route traffic from the API object to the desired services within the cluster.

## Why Kubernetes Ingress was Created

Before we delve into the specifics of Kubernetes Ingress, let’s take a step back and talk about why it was created in the first place.

In the Kubernetes landscape, pods are the smallest unit of computing application. A few simple facts about pods:

-   A Pod’s existence is by nature dynamic: they are created and destroyed according the state of the cluster and workload dynamic. So a set of Pods at any given moment may not exist later on.
-   Pods must be able to communicate with other pods, in the same cluster or in external clusters.
-   Each Pod has its cluster-private IP address.
-   For pods to communicate with one another, they need to know the IP address of that particular pod.

Given that Pods can go down any time and new Pods are created all the time in a cluster, we cannot rely on the Pod’s IP address for communication. This is where Kubernetes Services comes to the rescue.

Services are Kubernetes resources that identifies a set of pods which are grouped based on the type of service they provide. A Services is a Kubernetes object that enables network access to pods and enables pods to communicate with one another. Each Service is assigned a unique IP address (also called cluster IP) on creation and Pods can be configured to talk to the service. To route traffic to one of the concerned Pod container instances within a cluster, Kubernetes uses IP proxy techniques. Because this method relies on the use of “virtual IP address” (which is internal to the cluster), a different way is needed to access the Service endpoint. Enter Kubernetes Ingress.

## Ingress Controllers

To implement the created Ingress resources, you need Ingress Controllers. Ingress Controllers ‘read’ the Ingress Resource that have been created and then process the data accordingly. The Controllers handle all the routing logic; they accept traffic from the external sources, load balance it to applications, and route to the appropriate services in the Kubernetes cluster.

There are numerous Controllers available in the market, some of which are listed [here](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). One of the commonly used controllers is the NGINX Ingress controllers, a general-purpose implementation that’s compatible with most Kubernetes deployments.

![Introduction_to_Kubernetes_Ingress2.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltb99db3e85e985e9c/62c418b5b351a9369051d27c/Introduction_to_Kubernetes_Ingress2.jpg)

## Path Types

Every path in an Ingress must have a corresponding path type, otherwise they will fail validation. In Kubernetes, there are three paths types:

-   Implementation Specific: in this path type, based on the configuration in the IngressClass, the matching method is chosen. Implementations may treat the type as a separate path type or treat it the same way as the Prefix or Exact path types.
-   Exact: The URL path is matched exactly as it is and with case sensitivity.
-   Prefix: With this path type, matches are based on a URL path prefix split by forward slash / and the matching is case sensitive. This matching process is done from element to element in the URL.

To learn more about the matching scenarios, visit the official Kubernetes [documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/). 

## Kubernetes Ingress vs. LoadBalancer vs. NodePort

Kubernetes Ingress, Load Balancer, and NodePort are mechanisms used to enable external access to the Service. Each of them allows an external request (outside the Kubernetes cluster) to pass to a Service inside the cluster. For the three mechanisms, you can configure access by creating a collection of rules that specify which inbound connections reach which services.

### NodePort

A NodePort is used to expose a Service to on a Static Port number and can be set via a configuration setting in a Service YAML file. Kubernetes will allocate a specific port on each Node and any incoming requests to a cluster on that port will be be forwarded to the Service. Although easy to configure, it’s just not very robust and is used primarily for non-production environments.

![Introduction_to_Kubernetes_Ingress3.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltef730ca1a62d6dc1/62c418d33d042036dcb482c0/Introduction_to_Kubernetes_Ingress3.jpg)

### LoadBalancer

LoadBalancer supports multiple ports per service and is the default method for many Kubernetes installations. It exposes services to the internet by using an external LoadBalancer, which is typically implemented by a cloud provider

Similar to NodePort, you can configure a service to be of type LoadBalancer by specifying it in the service’s YAML file. You will need to create a new LoadBalancer and get an IP address every time you need to expose a service to the Internet.

![Introduction_to_Kubernetes_Ingress4.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt8dbd4bfb1209929a/62c418e5f1696e3810a01793/Introduction_to_Kubernetes_Ingress4.jpg)

### Kubernetes Ingress

While NodePort and LoadBalancer expose a Service by specifying the type property in the YAML file, Kubernetes Ingress is a completely independent resource. As part of Kubernetes cluster, it consolidates all the traffic rules into a single resource. With this mechanism, you can declare, create, and destroy Ingress separately to your Services.

It’s particularly useful for exposing Services in a production environment. Here’s why:

-   It’s decoupled from the Services you want to expose.
-   The Ingress Resource houses all the the traffic rules.
-   Since it’s managed from inside the cluster, this is a more cost-effective option (compared to the LoadBalancer which is heavily dependent on the cloud provider).

![Introduction_to_Kubernetes_Ingress5.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltac081997e24277fe/62c418f66a8fb133b0ff58cc/Introduction_to_Kubernetes_Ingress5.jpg)

## Conclusion

Kubernetes Ingress is an API object that is used by developers to expose their applications to the outside world and manage external access. One can simply configure access by creating a set of rule, outlining which inbound connection reach which Service. Some of the use cases of Ingress include load balancing, SSL termination, and name-based virtual hosting. 

The complexity around Kubernetes can make it daunting for those operating it and hinder your software delivery. [Weave GitOps](https://www.weave.works/product/gitops-enterprise/), a full-stack GitOps platform for continuous delivery, is based on the core principles of GitOps. It simplifies the operational complexity of software and delivery can help accelerate your journey to cloud native.

___

To find out more about how we can help you put GitOps to work in your organization, please book a meeting today.

[Request a Demo](https://www.weave.works/contact/)