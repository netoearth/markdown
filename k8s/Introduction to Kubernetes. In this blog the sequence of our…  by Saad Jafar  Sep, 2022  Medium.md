**In this blog the sequence of our discussion will be as follows:**

1.  What is Kubernetes?
2.  Background on Container
3.  Why did we need Kubernetes in the first place?
4.  Discussion over Kubernetes architecture
5.  Kubernetes Benefits
6.  Conclusion

## **What is Kubernetes?**

![](https://miro.medium.com/max/1400/1*-v7aUpje1lN9cisuRvmZiA.png)

source: [image](https://www.a10networks.com/wp-content/uploads/OnDemandWebinar-Kubernetes-KeyGraphic-@2x-768x422.png)

In simple words, **Kubernetes** is an open source platform which is actually a container orchestrating service. It is written in Go programming language and was originally designed by engineers at Google.

To further elaborate on this point, it automates the process of deployment and scaling of the **containers**. _Interesting right?_ We will dive deep into how Kubernetes actually does this later on in the section. First let’s get a quick overview of what a container is so we have our basics correct.

> What is a **Container**?

**Container** is basically a process that virtualises your OS in the way that several workloads can be handled with a single OS instance. This takes up only the space required by your application and nothing more, which is why it can be preferred over Virtual Machines. Containers are also more portable — which means that it is easy to move the application from one host environment to another, given that it is from the same family of operating systems, and hence, you do not need to re-compline anything. A container is isolated from all other processes. It packages all of the dependencies and code so that the application works faster. **An interesting fact:** _It was in 2013 that Docker Container technology was actually launched!_ More about Docker Containers is available on [this link](https://docs.docker.com/get-started/#what-is-a-container).

![](https://miro.medium.com/max/1400/1*DV80EPvqFv6pkT6auMUWuQ.png)

source: [docker.com/resources](https://www.docker.com/resources/what-container/)

## **_Why did we need Kubernetes in the first place?_**

The need for Kubernetes can be explained in the points given below. The architecture diagram will illustrate the concept.

1.  Suppose you have set-up your containers and there are containers already running.
2.  There is an application running in a container and there are 2 nodes (a node is a physical or virtual machine) . Now, if the application is down in one node, how do we make sure that application is deployed on the other node?

> **_One possible solution_** could be to change the namespace in the application so it now belongs to the other node. However.. the issue is that you would still have to do this manually each time. Looking for server requests to see if it working fine and deploying another node would take time and incur costs. How do we automate this? Wouldn’t it be simpler if a machine did this for us?

**Another challenge:** running several containers on several nodes in a production environment.

With this, you have to **_schedule_** which container runs on which node. This is because you might need different types of containers for different nodes, e.g a memory-hungry node. You also have to maintain **_networking_** between containers on different nodes. Finally, you might also want to ensure your application is **always up and running** and can handle **concurrent users** at a given time. What solves these problems?

_That is where_ **_Kubernetes_** _comes into play_.

![](https://miro.medium.com/max/1400/1*9-wcLW_OSzBFvZcg7SF5FQ.png)

**Kubernetes architecture**, source : [https://tinyurl.com/simplilearn101](https://tinyurl.com/simplilearn101)

Kubernetes has two nodes — a **master node**, and a set of **worker nodes**. When you make use of Kubernetes, the nodes are in an entity called a **cluster.** Every cluster requires at least one node. Each worker node consists of a **pod**, which is a set of running containers in the cluster.

## Let’s now discuss the Kubernetes architecture shown above with respect to the components of the master node.

1.  **Control Manager**

-   It is a component in the control plane that gathers information and sends it to the API server.
-   It controls the controller processes, namely Node controller, Replication controller, Endpoints controller, and Service Account controllers.
-   For example, replication controller ensures that a certain number of containers are running at all times and a node controller is responsible for on-boarding new nodes to a cluster and handling situations where nodes become unavailable.

2. **Control Scheduler**

-   It picks out what node to place the container based on a number of things, including the worker node’s capacity and the container’s resource requirements.
-   Gives out tasks to the worker nodes.
-   It is responsible for distribution of workload between nodes.

3\. **API Server**

-   It manages all the components. This means that it makes communication between the components possible.
-   It is responsible for performing all operations within the cluster.
-   It observes the state of the cluster and make changes if required.
-   For example, consider a case where there are 3 worker nodes. If one node is down, then the **replica set** is transferred to another node. If all 3 nodes are down, **API Server** sees this and creates 3 more nodes.

> A **replica set** is another abstraction layer on top of Pods. It is there to make sure that there are a given number of similar pods running at a certain time.

4\. **ETCD**

-   At any given point of time, containers get loaded to a worker node or unloaded from a worker node. To track intricate details such as what container was on which node and the time the container was on-boarded, etcd is needed.
-   It is a key-value store.
-   It is based on the Raft Consensus Algorithm.
-   The algorithm basically makes sure that even if some members fail it can still work.
-   Stores configuration details such as subnets etc.

5\. **Kubetcl**

-   This is a Kubernetes command line tool.
-   Kubernetes cluster manager — assists in interacting with the Kubernetes cluster.

## With all of this in mind, what are the benefits Kubernetes provides?

![](https://miro.medium.com/max/1400/1*eLDBDioc_oCP263mVaVcgQ.png)

source: [https://tinyurl.com/cloudstackgroup](https://tinyurl.com/cloudstackgroup)

**Autoscaling:** Even when several users use the application at the same time, the application runs perfectly fine. Kubernetes makes use of horizontal scaling — automatically scaling the number of pods.

**Load Balancing**: Incase of high traffic to a container, Kubernetes distributes the workload and traffic so that everything remains firm.

**Storage orchestration**: Kubernetes is flexible in the way that it allows you to use your own storage, be it from a cloud provider or from your own machine.

**Self-healing:** Kubernetes replaces containers, restarts the failed containers, and kills the containers if they do not meet a set requirement.

**Automated bin packing:** If you tell Kubernetes that you have a cluster of nodes and the amount of memory each container needs, Kubernetes will automatically set the containers in the nodes for you based upon your requirement.

**Secret and configuration management:** You can store and manage classified information on Kubernetes, such as passwords.

## Conclusion

It is not hard to guess why many organisations are starting to familiarise themselves with a powerful tool that Kubernetes is. It is definitely the next big thing! Hoping the content was easy to digest! More information about Kubernetes is available on their [documentation](https://kubernetes.io/docs/home/). Support me if you liked the content :) Thank you for reading!