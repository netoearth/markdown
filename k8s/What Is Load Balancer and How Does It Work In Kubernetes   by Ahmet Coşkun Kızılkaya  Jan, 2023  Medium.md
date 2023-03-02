**Load balancing** refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a _server farm_ or _server pool_.This provides us with effective server management, for example, let’s say 50 requests come in, it distributes them among 5 Pods (Pods are the smallest, most basic deployable objects in Kubernetes), with 10 requests each to devices (each with equal number of CPU cores) and if it detects that the incoming requests are from the same user, it directs them to the same POD to manage the session.

![E.g. NGINX Load Balancer System in Kubernetes](https://miro.medium.com/max/770/1*D76F2Pruo6VtOHBGJ2Sbsw.png)

In Kubernetes, we refer to each **unique IP address** as a cluster. You don’t need to link or use **NAT** for these IP addresses within Kubernetes, Kubernetes does this for you and links them together. These clusters can be virtual or physical computers.Kubernetes IP addresses exist at the Pod scope — containers within a Pod share their network namespaces — including their **IP address and MAC address**. This means that containers within a Pod can all reach each other’s ports on localhost. This also means that containers within a Pod must coordinate port usage, but this is no different from processes in a VM. This is called the “**IP-per-pod**” model.

![](https://miro.medium.com/max/770/1*WHXv2Z0bBfC7GW4egoIwTw.png)

Each of the two methods of load distribution that exist in Kubernetes operate through the kube-proxy feature. Services in Kubernetes use the virtual IPs which the kube-proxy feature manages. In this way, the user’s IP address is hidden. We can reveal the user’s IP address on the back-end by setting a Header configuration.The default method retrieves the IP addresses of the users in the next field and returns a list, while the second method consists of advanced rule-based IP rules called IPTables, where the second method allows us to choose from these IPs, but neither of these provide efficient load balancing.

## Ingress

At this point, Kubernetes’s own definition tells us: An API object that manages external access to the services in a cluster, usually HTTP. Ingress may provide **load balancing, SSL termination and name-based virtual hosting**. In these ways, **INGRESS** can be thought of as a “**smart-router**” and one of its tasks may be load balancing.

![](https://miro.medium.com/max/770/1*WjyJHBR8TEBJan9NqxtuMQ.png)

Ingress does not show connection points or ports. When **Service.type = LoadBalancer**, it acts as a load balancer. This method allows us to control the incoming flow, which gives us the ability to manage it perfectly. With our already divided PODs, controlling the entries allows us to prevent unnecessary, unwanted, or excessively fast requests and keep our server up and running well. You can choose and integrate the Ingress controllers from companies such as **NGINX** into your own Kubernetes structure according to your needs, and these structures will not affect your resource consumption. Here is an example code:

![](https://miro.medium.com/max/770/1*U9pkN8Byoi72oaHoG9szsw.png)

Use this [**address**](https://kubernetes.io/docs/concepts/services-networking/ingress/) to get detailed information on how to edit it.

## Which OSI layer does INGRESS operate ?

**INGRESS** can operate between the OSI layers 4 and 7, which can be adjusted according to your needs and what you will be doing. Layer 4 does this with less information while Layer 7 does it with more information. At this point, Layer 7 should be preferred for larger scale projects, as the choice of Layer 7 enables the **INGRESS** controller to make more logical decisions and helps to provide faster traffic flow.

![](https://miro.medium.com/max/660/1*FIMSfWEPyahjhFqYT5dWww.png)

That’s all the information I have for today. If you have any comments or opinions about what I’ve written or about me, you can reach me on [LinkedIn](https://www.linkedin.com/in/ahmetcoskunkizilkaya/). Stay healthy, happy coding :)

Resources :