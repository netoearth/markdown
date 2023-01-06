[Kubernetes](https://www.weave.works/blog/kubernetes-cluster) is witnessing wide adoption by organizations across industries. Its deployment capabilities with zero downtime, resilient infrastructure, automatic rollbacks, high scalability, and self-healing features are primarily due to its deployment capabilities. But the one benefit that separates it from others is that it simplifies the complexity of managing containers at scale. Kubernetes achieves this through a powerful concept called nodes. A node is simply a worker machine that executes Kubernetes workloads.

In this article, we learn in detail what a Kubernetes node is, the different types of nodes, how they work, and what a node status report is.

## What is a Node in Kubernetes?

### Node

A Kubernetes node is the smallest computing unit representing a single machine in your cluster. It can either be a physical machine housed in a data center or a virtual machine hosted on a cloud. Since it comprises a set of IT resources, a node can be made out of anything - a bare metal server, VM, or container. These resources are used to run Pods and assigned workloads, interact with components, and configure networks.

In simplest terms, a node can be considered a worker machine with CPU and RAM resources to run applications.

![Pod_Diagram_01.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt3c96ff2db6b0c1ff/6357f27c330ba0566cd66a9d/Pod_Diagram_01.jpg)

**Node Overview -** [**Source**](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/)

### Pod

_A Pod is a deployable group of one or more containers that you can create and schedule to run on a node in your cluster. The Pod stays on the assigned node until it is executed, deleted, or evicted due to a lack of resources._

A node has components like kubelet, container runtime, and the kube-proxy, that help it create the runtime environment and support Pods. The control plane, which is an orchestration layer, manages nodes. And there can be several nodes within a single cluster. A node, in turn, can run multiple Pods. For a Pod to work on a node, the node must fulfill two conditions:

1.  A node must have available resources to run the workloads
2.  A node must meet the Pod’s criteria for affinity or anti-affinity with other Pods

![Pod_Diagram_02.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltf25508ad47bc78d3/6357f290714c6a728bd01f63/Pod_Diagram_02.jpg)

**Pods Overview -** [**Source**](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/)

## Kubernetes Nodes vs. Pods

Although nodes and pods are very different in their contents and functionality, they’re often confused with each other. But before we compare nodes with pods, let’s first understand what pods entail.

### What is a Pod?

A pod is a group of one or more containerized applications along with the resources used to execute them. These resources include shared storage, network resources, and instructions to run containers like container image versions and ports. It is a logical host similar to a virtual machine that executes applications. Containers within a pod can share resources and other dependencies and interact with each other.

In short, pods are analogous to executable code, while nodes are similar to computer hardware. Here’s a hierarchical structure of pods and nodes to help you better understand how they differ.

-   Level 0: Applications and their associated infrastructure are packaged in Containers.
-   Level 1: Containers, in turn, are grouped in pods.
-   Level 2: Pods are then hosted on Nodes that offer computing resources to execute applications.

Efficient operations of your Kubernetes depend on how optimized your nodes and pods are. Although it is essential to keep the number of containers within a pod minimal, you can add as many pods onto a node as you need.

## How does a node work?

Having understood the importance of a node within Kubernetes, let’s go through an overview of how a node works. A node is a workstation where pods affixed with containerized workloads are executed. However, there are a few concepts that you need to be familiar with when you’re dealing with nodes.

### Creating a node

Adding a new node to an API server can be done in two ways. You can either manually add a node object to the control plane or the kubelet self-registers on a node. Whenever a node object is created, the control plan evaluates its validity. Only when it is certain that the node is healthy with required services up and running does it receive eligibility to run a Pod. If it is ineligible, Kubernetes repeatedly checks its health until it meets the criteria or is explicitly deleted.

When manually creating a node, you must submit capacity demands, unlike in the self-registration process, where the nodes request CPU and memory volume capacity. Following this, the scheduler assigns pods to a node making sure that the requests are not more than the node can manage.

**Node name uniqueness**

A node is identified by its name; therefore, no two nodes can have the same name simultaneously. Since a Kubernetes instance that uses the same name is expected to have the same state and attributes. Such an approach will make the system inconsistent when you modify the instance without changing the name.

## What are the different types of nodes in Kubernetes?

Kubernetes follows the simple and effective master-worker architecture with master components controlling and directing worker components to execute a workload. Similarly, in Kubernetes, master nodes decide how containerized applications are executed, and worker nodes simply follow the instructions given by master nodes.

### Kubernetes Master node

The master node runs the entire cluster through a control plane. A master node is a mandatory component of every cluster. Besides, there can be multiple master nodes for redundancy.

The master node comprises API Server, Etcd, Controller Manager, and Scheduler. Let’s go through them in detail.

### Components of Kubernetes Master Nodes

**kube-apiserver**

The Kubernetes API server acts as a control unit for your Kubernetes ecosystem, forming a frontend for your clusters. It receives REST requests for modifications to API objects, including pods, services, and controllers. The API server is the only master node component that interacts with Etcd to validate and configure data housed in Etcd in sync with the requirements.

kube-apiserver scales horizontally by adding additional deployment instances. It accommodates a high volume of instances and efficiently manages traffic between them.

**Etcd**

Etcd is a simple, consistent, and highly available database to store cluster states. It holds information such as the number of pods, namespace, API objects, and service discovery details. As a key-value storage space, Etcd is used as a Kubernetes backing store. Since it contains critical cluster information, protection is ensured by allowing only the API server to access the data.

Etcd triggers notifications in the form of API requests on the Etcd cluster node every time a configuration change is detected.

**kube-controller-manager**

Controller Manager ensures that the desired state is implemented by running a series of controller processes. Although these processes are directly unrelated, they are all viewed collectively as a single process to simplify operations. Two such processes are:

-   The replication controller ensures that the correct number of replicas are formed in a pod
-   The endpoints controller populates endpoint objects like services and pods

It primarily regulates the shared state of the cluster. It initiates processes to achieve the desired state whenever configuration changes are detected.

**kube-scheduler**

In simple words, the scheduler assigns Pods to Nodes. It is a control plane component that proactively identifies the creation of new pods without a designated node and starts scouting for an ideal node to place this new pod. The scheduler bases its selection on factors like resource requirements, policy constraints, affinity and anti-affinity specifications, data locality, and deadlines.

![k8sNode_Diagram.png](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltcea99e1247df23da/6352bca21fb4e757e110dfc5/k8sNode_Diagram.png)

**High-level Kubernetes architecture with a Master Node and two Worker Nodes**

### Worker nodes

Worker nodes are the ones that execute application containers. You can scale your Kubernetes clusters by adding more worker nodes to manage added workloads. The worker node streamlines the workload execution through three components as discussed below:

### Components of Kubernetes Worker Nodes

**Kubelet**

Kubelet is a Kubernetes communication agent that runs on worker nodes. It works as per the instructions from the control plane. Kubelet creates, terminates, or updates pods to ensure they are running and healthy. Kubelet carries Pod specifications in files known as PodSpec stored as YAML or JSON objects. It receives PodSpec from the API server, and these files contain details directing Kubelet to launch Pods and containers and check their health. It also reports Kubernetes on the status of its node.

**Kube-proxy**

Kube-proxy is a network proxy that makes Kubernetes networking services available at nodes. All the nodes contain a kube-proxy that can transmit data between pods, the host, and the outside world. It forms a layer between the Kubernetes network and the pods running on that particular node. Kube-proxy is a vital networking element that a communication link is maintained across clusters by proxying UDP, TCP, and SCTP. In addition to managing services defined in the cluster, kube-proxy also manages rules to load-balance requests.

**Container runtime**

Container runtime is software that triggers the execution of containers on each node in a cluster. Each node comes with a container runtime that focuses on namespace and groups for containers. A container engine like Docker and RKT translates into a platform to enable containerization by loading container images and storing them to be executed. Now, the container runtime starts and manages the deployment. Kubernetes supports runtimes compliant with the Open Container Initiative, a set of standards and specifications for container technology.

## Understanding Kubernetes node status

You can use the Kubernetes command-line tool Kubectl to run commands against clusters, deploy applications, and assess and manage cluster resources. To view the Kubernetes node status, you can use the following command:

<Kubectl describes nodes \[node-name\]>

A node status returns details like Addresses, Conditions, Capacity/Allocatable, and System Info, along with information that helps the Kubernetes scheduler map pods onto the appropriate nodes.

### Address

This section shows details such as the node's hostname, external IP, and internal IP. Their representation varies based on where the node is being run – on-premises or on the cloud.

**Hostname:** It displays the hostname as displayed by the node's kernel. You can override the name using kubelet <username-override> parameter.

**External IP:** Available from outside the cluster, this gives the node’s IP address that can be routed externally.

**Internal IP:** The node’s IP address can only be routed internally.

### Status Conditions

The conditions section provides the status of every node present. The common conditions that appear in a node status report are shown below. The status against each condition is defined as true, false, or unknown.

**Ready**

It displays ‘true’ when the node is healthy and is open to accepting pods. If the node is unhealthy and not accepting pods, it will return ‘false.’ ‘Unknown’ signifies that the node controller has no information regarding the node in the last 40 seconds, termed node-monitor-grace-period. If the status remains ‘false’ or ‘unknown’ for longer than the pod-eviction timeout, the node-controller will evict all the pods assigned to the node.

**DiskPressure**

If the value returned against this parameter is ‘true,’ then the disk capacity is low. In any other case, it will show ‘false.’

**MemoryPressure**

If the node memory runs low, it will return ‘true;’ otherwise, you will see ‘false’ as the status.

**PIDPressure**

If way too many processes are running on the node, then PIDPressure will display ‘true;’ otherwise, ‘false.’

**NetworkUnavailable**

If configuration issues exist in the node network, the report will show ‘true’ against this parameter. If the configuration is proper, it will be ‘false.’

### Capacity/Allocatable

This section of the node status report gives you an insight into the resources available on a node. It typically includes CPU, memory, and the number of pods a node can accept. While the capacity segment details a node's total resources, the allocatable segment lets you know what capacity of resources is available for consumption.

### System Info

This segment details the basic information about hardware and software on the node. This information is used by kubelet to publish it into the Kubernetes API. Some of the details include:

-   Kernel version
-   Kubelet & Kube-proxy version
-   Container runtime details
-   Operating system

## Assigning Pods to Nodes

Kubernetes lets you allow a pod to run on a particular node or particular nodes. While you can manually manage the mapping of pods with nodes, Kubernetes allows different methodologies to automate the process through labels.

### Labels

Labels are key-value pairs added in pods that form identifying attributes of the pods. However, they have no direct effect on the core system. Two different pods can carry the same labels. Identifying pods based on the associated labels is known as label selector.

The various methods to assign pods to nodes are:

### Node Selector

Node Selector is a practice of using an identifier known as Node Label to match the pods it can host. To link the pods with the nodes you wish, you need to add a nodeSelector field and node labels in the pods. The Kubernetes scheduler evaluates the labels on both pods and nodes to match the ones with each specified label.

### Affinity and Anti-Affinity

Unlike Node Selector, Affinity is an expansive and expressive way of defining which nodes to run the pods on. It uses the AND operator to execute exact matches. You can also include soft rules mentioning preferences for a certain node type. The scheduler, however, will decide on the nodes to deploy pods even if it doesn’t meet the preferred criteria. Further, using the affinity model, you can enable the colocation of pods. Anti-Affinity is declaring rules that prevent a pod from running on a particular node.

The scheduler uses the concept of taints and tolerations to decide on selecting nodes to run pods on. If a node has a taint for particular pods, it can repel them, while tolerations let schedulers link pods with matching taints.

## Conclusion

Kubernetes nodes are essential components in ensuring the seamless execution of application workloads. Since it follows a master-worker architecture, the process of defining the desired state and implementing it as the current state is streamlined through various node components. Although most of the required tasks to manage Kubernetes clusters are automated, you must understand the workings of the fundamental element of Kubernetes.

GitOps is an efficient way to manage [Kubernetes](https://www.weave.works/blog/category/kubernetes/) through a collection of open-source software and tools. But the quickest and easiest way to do GitOps - automate Kubernetes - is through Weave GitOps. Learn more about [Weave GitOps](https://www.weave.works/product/gitops-enterprise/) or [request a demo](http://weave.works/contact) to talk to one of our experts.