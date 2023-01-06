## Introduction

If you are not new to [**Kubernetes**](https://kubernetes.io/), probably you heard this a million times: “**_Do not run databases on Kubernetes!_**”; and if you were in 2016 that may be true, but not anymore; in fact, I would go as far as saying that **Kubernetes is now the best platform to run stateful workloads.**

![](https://miro.medium.com/max/700/0*309-pDZ2bqZQDRHr)

Photo by [Venti Views](https://unsplash.com/@ventiviews?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

In this article, I will try to **change your mind about running stateful workloads in** [**Kubernetes**](https://kubernetes.io/) and try to convince you that in many cases, it is the best approach that you can take. I will also talk about different alternatives and when not to run databases in **Kubernetes**.

The article is divided in **two parts,** the first one focuses on how to run database in Kubernetes and the second one focuses on when you should run databases in Kubernetes.

Let’s begin with a quick history lesson and then we will dive into the uses cases.

## Brief History

Although many recruiters sometimes ask for engineers with more than 10 years of experience; Kubernetes 1.0 was released in **2015**. It was created by Google and released as open source under the [Cloud Native Computing Foundation](https://en.wikipedia.org/wiki/Cloud_Native_Computing_Foundation).

It gained popularity thanks to the rise of **cloud** computing and cloud native applications. Its adoption increased exponentially over the years and nowadays **Kubernetes is the most popular way to run containers**.

In my opinion, the key of its success is **having the right level of abstraction**; not too low so it is impossible to manage, neither too high or opinionated. Many engineers complain about the complexity and many frameworks and tools have been developed on top of Kubernetes to solve specific problems or use cases to make it more opinionated. The truth is that Kubernetes is becoming the de facto standard to run workloads in any environment whether is on cloud or on-prem. **The key is its API**, which can be extended in order to create higher level abstractions, which will allow us to deploy any workload using the same standards and familiar the familiar **YAML** files.

Thanks to the extensible [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/), new features were added over time and companies developed their custom APIs targeted to specific workloads.

![](https://miro.medium.com/max/700/0*9WsPjvATZDcXxEK3.png)

Kubernetes architecture

One problem with early versions of Kubernetes was that it was quite cumbersome to deploy complex applications since you needed to write and maintain many lines of **YAML** files. To simplify the deployment, [**Helm**](https://helm.sh/) was introduced in 2016 and nowadays is the de facto way to package and deploy workloads in Kubernetes although there are several [alternatives](https://github.com/kubernetes-sigs/kustomize) to Helm.

Thanks to **Helm**, it is very easy to package complex applications that can contain microservices, PVCs, services, secrets, etc. into a single chart. This is why Kubernetes is the best platform to run [**microservices**](https://en.wikipedia.org/wiki/Microservices) in an efficient and secure manner.

![](https://miro.medium.com/max/700/0*XSNM92M44vmdyrJm)

[**Microservices**](https://microservices.io/) are **stateless** services, so they are the perfect workload to run on Kubernetes, which allows them to scale up or down quickly. In case of any failure, the Kubernetes control loop will restart the pods. However, things get more complicated when we talk about **stateful workloads** such as databases. You cannot just add more replicas when you want to scale. Some databases just support read only replicas, others they need to be re-balance. Other databases cannot just be restarted and need a graceful shutdown. Furthermore, operating a database is more complex and we need to perform many operational tasks such as backups, restore, monitoring, alerting, etc.

Because of the complexities of running databases, many companies decided to **run their databases in VMs** or use 3rd party managed databases instead. These services will take care of all the operational overhead of running databases. This is the most common approach in Kubernetes where **applications run on a Kubernetes cluster and connect to databases running elsewhere**. This approach has many drawbacks, which we will talk extensibility later. Once thing is clear, we cannot assume that applications are stateless, they need state, the question is where does the state lives, in Kubernetes or outside of Kubernetes.

But before we discuss this, let’s dig deeper into how to run databases on Kubernetes…

## StatefulSets

The first step towards running stateful workloads in Kubernetes was the introduction of [**StatefulSets**](https://en.wikipedia.org/wiki/Kubernetes#StatefulSets) which provides the basic functionality to allow running stateful applications in Kubernetes.

[**StatefulSets**](https://en.wikipedia.org/wiki/Kubernetes#StatefulSets) provide predictable **order** and **state** for stateful workloads such as databases. They are scale up or down sequentially in an predictable order. This is very important for databases:

-   For primary-secondary architectures this allows to have a predictable primary node (the first one) and many secondary nodes.
-   For ring architectures this allows for self discovery and data re balancing.

**StatefulSets** also have a unique DNS name based on this order this is important for workloads such as [Apache Kafka](https://en.wikipedia.org/wiki/Apache_Kafka) which distribute the data amongst their brokers; hence, one broker is not the same as another. In this case, the notion of instance uniqueness is important.

In short, StatefulSets are controllers that enforce the properties of **uniqueness** and **ordering** amongst instances of a pod and can be used to run stateful applications. They also allow client side load balancing since clients can discover and the replicas by its fixed name with the order suffix.

As an example, let’s image we want to deploy [**PostgreSQL**](https://www.postgresql.org/) database with a primary node and two read only nodes. If we use regular Kubernetes deployments, Pods will get a random name and the Kubernetes service will redirect traffic to any of the replicas, clients will connect to the service and have no knowledge of who is the primary write only pod and who are the replicas. This will not work. We need a way to identity the primary and secondary nodes. With StatefulSets, pods have a predictable name comprised of the name + suffix. For example `postgres-0` will be the first pod, in our case the primary node. The secondary nodes will be `postgres-1, postgres-2.` This way clients have a way to identify the primary node.

Also, persistent volume claims([**PVCs**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)) are stable for **StatefulSets**, so if a pod dies and get re schedule in another node, the same volume is mounted in the new node with the same data.

> StatefulSets are valuable for applications that require one or more of the following.
> 
> \- Stable, unique network identifiers.
> 
> \- Stable, persistent storage.
> 
> \- Ordered, graceful deployment and scaling.
> 
> \- Ordered, automated rolling updates.

Although the features of **StatefulSets** are extremely important for running databases, they are **not enough to run a production grade database**, they are however the best abstraction common to all stateful applications.

Stateful workloads require daily operations such backups, compaction, etc. Also, scaling up may require copying data or self discovery. This is usually achieved by installing some kind of “agent” that monitors the cluster and perform actions when a new node is discovery.

All these operations, depend on each database and its architecture(ring, primary-secondary, etc), they are very application specific; we cannot build a Statefulset for each database in order to have agents or operators performing operations when nodes are added or removed.

However, as we mentioned before, the Kubernetes API is highly extensible allowing developers to extend it to create new objects. The Kubernetes continuous control loop can monitor these objects and operators which are specific for the given database can be implemented.

## How to run Stateful Workloads in Kubernetes

We have seen that with the introduction of **StatefulSets** we got the building blocks for running databases, but you need extra functionality to run production ready databases. Is this why people are saying that we shouldn’t run databases in Kubernetes? well not really…

## Can I run Stateful workloads in Kubernetes?

The answer is: **YES!**

According to the latest report from [Data on Kubernetes](https://dok.community/):

> **_90% of the customers believe it is ready for stateful workloads, and a large majority (70%) are running them in production with databases topping the list._** _Companies report significant benefits to standardization, consistency, and management as key drivers._

There are many advantages in terms of management and cost of running databases in Kubernetes, it is complex but the benefits outweigh the drawbacks. Kubernetes provides a unified framework which is familiar to everyone. It also provides the right level of abstraction to run any workload. [**DevOps**](https://azure.microsoft.com/en-us/resources/cloud-computing-dictionary/what-is-devops/) engineers who are familiar with Kubernetes will prefer using the same familiar APIs for databases rather than learn and maintain a parallel deployment for databases. This enables true **DevOps** principles where everyone is speaking the same language and using the same tools to deploy any workload no matter if it is a microservice, a batch job or a database.

![](https://miro.medium.com/max/433/1*TJiw-nr374vzO69kLA5rOQ.png)

The key is **automation**, for cloud native applications we want to reduce the technical toil which is the amount of manual work required. We do not want to take manual backups, not even doing a manual restore.

We would like to create new objects in the Kubernetes API and build operators that rely on the fantastic **Kubernetes control loop** to listen for events regarding these objects and take actions accordingly. For example, we could create a new Kubernetes API called `PostgresDB` which can be defined in YAML and applied just like a deployment. The missing piece is an operator that watches the deployment and when it is updated to add a new replica, it will call the other pods to apply the appropriate changes.

## What is the best way to run stateful workloads on Kubernetes?

The Answer is: **Using an** [**Operator**](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) **+** [**CRDs**](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)**.**

[**StatefulSets**](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) are the building blocks for Stateful workloads but they are too basic. For a database you need to create backups, replicate data, expand the cluster, create standby clusters, disaster recovery, monitoring and much more.

**CRDs** are used to extend the Kubernetes API with new objects, in our example a CRD will be defined to describe a PostgreSQL cluster using a new `PostgresDB` API.

The [**Operator**](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) pattern was created to allow cluster administrators to create applications that can use the existing Kubernetes control loops and APIs to watch and monitor resources in the cluster and take actions.

Using [**Operator**](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) **+** [**CRDs**](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) we can create Kubernetes objects to represent database clusters that can be manage as any other resource and the **Operator** will watch the Kubernetes API for any changes and take action. For example, if we increase the replicas it will handle data replication and backup, if we add an annotation it can trigger a restore, etc. This solution is very **powerful**, it uses what Kubernetes does best, which is watching resources and taking **real time actions**; and customizes it to target a specific database or technology.

Owners/Contributors of a database just need to write a **CRD** to describe the database and an operator to manage the new CRD. The CRD will be based on a StatefulSet object but will have extra attributes such as backup schedule, connection pooling, memory settings and much more. These properties will be made available to the **Operator** which just have to listen to events that happens on the CRD objects. So, if we change the YAML file to add a new node, the operator can trigger the data re balancing.

**StatefulSets** provide the building blocks which are required for all types of stateful workloads, then each database has to create its own operator; but where do I find these operators? The answer is the [**Operator Hub**](https://operatorhub.io/), this is where you can find thousands of operators to virtually any database.

A great benefit of **Operators** is that because they extend the Kubernetes API, they **are compatible with many Kubernetes tools**. For example, [**GitOps**](https://www.gitops.tech/) tools such [**ArgoCD**](https://argo-cd.readthedocs.io/en/stable/) can be used to deploy databases the same way they are used to deploy microservices. [**ArgoCD**](https://argo-cd.readthedocs.io/en/stable/) will treat the new CRDs the same way as any other resource.

## Example: PostgreSQL Operator

**PostgreSQL** is a popular database and the [Operator hub](https://operatorhub.io/?keyword=postgresql) has many [operators](https://operatorhub.io/?keyword=postgresql) ready to be used for production workloads. The most popular operator is the [**CrunchyData**](https://access.crunchydata.com/documentation/postgres-operator/v5/) **PostgreSQL** Operator which is the most feature rich of all them and it is used by many companies in production.

We need an operator that can handle common database operations for us while still leveraging on **Kubernetes** objects in order to manage stateful and stateless workloads the same way. We also want a product that is compatible with [**GitOps**](https://www.gitops.tech/) tools such as [**ArgoCD**](https://argo-cd.readthedocs.io/en/stable/) to manage and monitor our workloads in a declarative way.

[**CrunchyData**](https://access.crunchydata.com/documentation/postgres-operator/v5/) **PostgreSQL** Operator creates a custom **CRD** to represent the database which is managed by the Operator, allowing it to stop and start instances on its own. ArgoCD an manage the Operator and the Operator manages the DB instances creating another level of abstraction. Cluster managers will only need to maintain the operator and the developers can manage the databases allowing a true **DevOps** experience.

[**CrunchyData**](https://access.crunchydata.com/documentation/postgres-operator/v5/) **PostgreSQL Operator Features**

The [Postgres Operator](https://access.crunchydata.com/documentation/postgres-operator/v5/quickstart/) from Crunchy Data orchestrates a commercially distributed toolset for enterprise class PostgreSQL, production ready out of the box and it has the following features:

-   Crunchy Certified PostgreSQL
-   Backups
-   Monitoring and dashboard tools
-   TLS
-   Connection Pooling
-   Automatic self-healing high availability
-   Easy cluster deployment and management
-   Zero downtime updates

![](https://miro.medium.com/max/700/1*_zp5lBCUj1l5NsiSDSZz_A.png)

The [PostgreSQL Operator](https://access.crunchydata.com/documentation/postgres-operator/v5/(https://github.com/CrunchyData/postgres-operator)), gives you a **declarative PostgreSQL** solution that automatically manages your [PostgreSQL](https://www.postgresql.org/) clusters.

Designed for your **GitOps** workflows, it is [easy to get started](https://access.crunchydata.com/documentation/postgres-operator/v5/quickstart/) with PostgreSQL on Kubernetes with PGO.

With conveniences like cloning PostgreSQL clusters to using rolling updates to roll out disruptive changes with minimal downtime, **PGO**([Postgres Operator](https://access.crunchydata.com/documentation/postgres-operator/v5/quickstart/)) is ready to support your PostgreSQL data at every stage of your release pipeline. Built for resiliency and uptime, PGO will keep your desired PostgreSQL in a desired state so you do not need to worry about it.

In short, the [Postgres Operator](https://access.crunchydata.com/documentation/postgres-operator/v5/(https://github.com/CrunchyData/postgres-operator)) has support for ArgoCD, Helm, any version of Kubernetes, **any cloud** and provides all the requirements you need to run and maintain PostgreSQL clusters, especially zero downtime updates and easy backups.

This is a perfect example of the power of **Kubernetes** and its great **extensibility** which allows us to deploy any workload in any environment.

## Alternatives

The key decision that you need to make it is whether to run your databases in Kubernetes or outside Kubernetes. We have seem that running all your workloads in Kubernetes is possible and has many advantages. It **unifies your operational plane, reduces costs and provides the only realistic way to be completely cloud agnostic.**

At the same time, many databases were not developed with Kubernetes in mind and could be problematic to run in Kubernetes. Most modern databases have now support for Kubernetes but you should research first and make sure your targeted database has a mature operator for Kubernetes. Still, databases requires database specific skills that normal DevOps engineer will not have, this is why we have Database Administrators(**DBAs**) which specialize in database management and administration. Usually, one DBA will focus on one database such a PostgreSQL admin or Cassandra admin. Quite often these specialists do not have Kubernetes skills. However, this issue is mitigated in Kubernetes with Operators since they automate and perform the tasks that in the past DBAs would perform, such monitoring and backups.

But before we go any further, let’s review what options do you have if you are not using Kubernetes to run your databases.

## Virtual Machines

This is probably the most common option. You can use it in the cloud or on premises. This option is supported by all databases.

This option is cost effective and portable. You just need to make sure that the database binaries are available for your target platform.

If you are running your workloads in VMs and you are not using Kubernetes, then keep using VMs. It is better that you migrate your stateless workloads first and then decide what to do with the databases.

The **problem** with this option is that is **not cloud friendly**, **the level of abstraction is too low**. You need operation system skills, networking skills, monitoring skills, etc. You need to understand how to install and patch the different libraries for the given Operating System.

## Cloud Managed Databases

The major cloud providers such as **AWS** or [**GCP**](https://cloud.google.com/gcp/?hl=en), have database services such as [MySQL](https://www.mysql.com/), PostgreSQL or [Cassandra](https://cassandra.apache.org/_/index.html). These services are easy to use since the are **SaaS** services.

The cloud providers offer automation and configuration to easily backup and restore the data. This is the simplest way to deploy a database in the cloud.

However, you will need to build automation using [**Terraform**](https://en.wikipedia.org/wiki/Terraform_(software)) or another [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code) tool to provision the database and set the configuration. Note that if you are in a multi cloud environment, you will need to write different IaC for each provider which is time consuming.

The main **advantage** of this approach is that is very **easy** to setup and use. The **drawbacks** are **lack of portability** across cloud providers, cost and lack of control and customization options.

## 3rd Party Managed Databases

This option is similar to the previous one, the difference is that it is not the cloud provider who manages the operational aspects of the database but a 3rd party company.

The main **advantage** of this approach is that, this company in theory will take case of the multi cloud problem and allow you to run the database in any cloud. If you are need to deploy your applications in multiple clouds and you are not using PostgreSQL or MySQL then you will need to go this route if you do not want to manage the database.

Most of the Database vendors will have a cloud offering that you can use in any cloud, although the support and options may vary for each cloud provider. Some of the vendors will also provide a managed offering where they can deploy and operate the database inside your account instead on their account.

The drawbacks of this approach are its cost, the legal implications of having a 3rd party managing the data and the different support/customization level on each cloud provider.

## Pros and Cons

The following table summarizes the Pros and Cons of each option:

![](https://miro.medium.com/max/700/1*uEEXCrbj5EYgmBg_n7i1Jw.png)

**3rd party managed databases** are the easiest to get started with, it is a fully SaaS offering. Some examples are [Elastic Cloud](https://www.elastic.co/cloud/), [MongoDB Cloud](https://www.mongodb.com/cloud) or [ScyllaDB Cloud](https://www.scylladb.com/product/scylla-cloud/). These services manage **multi cloud** deployments for you and offer the best support. If you don’t want to run a database yourself this may be the easiest option; definitely it will the be the only option if the database is not MySQL or PostgreSQL since those are the only databases available among the main cloud providers. The problem with this approach is that is very **expensive**, it tends to be **difficult to customize** and it may be a no-go for many organizations due to data security and country laws. Some companies offer a way to keep the data within your cloud account and still manage it for you, so you still be the data owner, but this also has legal implications since there will be another company managing and accessing the data. If this is not a concern for you and you have the budget, this will ge a good option.

**Cloud vendor databases** are an easy way to run stateful services, the cloud vendors offer decent support and some customization options. This is not a good option for multi cloud deployments. Even if you use MySQL or PostgreSQL which are the only databases available among the main cloud providers, the version they support, the customization options, security or networking vary between cloud providers; making it very difficult to maintain in a multi cloud environment. For example, for PostgreSQL you only have a subset of extensions available and Azure support only version 11 at the time of writing. This cloud service is also expensive and there could be many hidden costs such as egress fees, backup fees, etc. You still need to write [**IaC**](https://en.wikipedia.org/wiki/Infrastructure_as_code) to provision the databases and manage the backup and restore. The support is not as good as 3rd party vendors but they are cheaper.

**Running databases in Kubernetes** using Operators is very **cheap** since you can use the Kubernetes share infrastructure. Thanks to the power of Kubernetes and the operators, it is easy to deploy and maintain; probably not as easy as the previous options but **DevOps** engineers will be using the familiar Kubernetes syntax. This option is very easy to deploy and get started; just install the operator and start provisioning databases. The main drawback is that this is the “do it yourself” approach and you will have no support.

The last option is running in **VMs**. This is the same idea as running any workload in **VM vs Kubernetes**, and probably we will need another article to discuss the pros and cons.

In short, if you are already using Kubernetes, deploy the stateful services also in Kubernetes instead of VMs if the databases have an operator. If the database is not Kubernetes friendly or you do not use Kubernetes, keep using VMs.

## Use Cases

Let’s have a look at the different options you have depending on where are you running your workloads. Let’s assume that you already have a Kubernetes cluster and you are running some workloads there.

## On Premises

If you are running **on-prem or in a private cloud**, Kubernetes in general is one of the best options since it unifies workload management and monitoring.

In this case, for databases, **Kubernetes** will be the best choice if the database has an operator and is Kubernetes friendly. If not, deploy the database in a VM and manage it separately.

Running the database in Kubernetes will be easier since the operator will take care of updates, backups, monitoring, etc. automating the operational overhead compared to **VMs**.

## Single Cloud

If you are committed to a single cloud provider (vendor lock-in), then you can leverage on the cloud managed services for **easy adoption and low operational overhead**. You need to take into account the costs, specially hidden costs like backup storage, network egress, etc.

**Always use the database that will fit your needs instead of trying to fit your needs into a specific database**. For example, if you have a hierarchical/graph model, do not try to use a relation database. In this case, if the cloud provider has a managed solution for the desired database and you can afford it, go for it; it will save you a lot of time.

If the database is not available in the cloud provider, you have 3 options:

-   Use a **3rd party managed service**: This is expensive but will have the same benefits as the cloud managed service. Check the legal implications as well. I personally prefer to use 3rd party managed databases in case I need to support many clouds since these companies will take care of the multi cloud problem for you.
-   Use **Kubernetes**, this is the best option if the database has an operator and it is Kubernetes friendly.
-   Use **VMs**: Use this option if the DB is not Kubernetes friendly and managed options are not available or too expensive.

If you are in a single cloud and you are committed to it, then use managed database when possible but watch out for the costs since they can be expensive, if you want to be more cost conscious, consider using Kubernetes.

## Multi Cloud

This is the most complex scenario. You need to deploy and support all major cloud providers and maybe even on-prem.

![](https://miro.medium.com/max/700/0*Ph9cVotPK2gWUGpI)

Cool image inserted so you don’t get tired and you keep reading, you are almost done!

In this case, **cloud managed databases are going to be problematic**. First, only **MySQL** and **PostgreSQL** are supported in the 3 major cloud providers, so you will be very limited. Even, if you are using one of these two databases, the setup, automation, cost structure and networking differs on each cloud provider; this means that the time you save by using a managed solution is used by the effort you will need to make to support the automation and configuration of the database across cloud providers. Surprisingly, I have seem this option a couple of times but I do not recommend it, it seems good on paper but you will soon start running into issues due to the differences between cloud providers.

**3rd party managed services** aim to solve this issue and usually provide multi cloud support. Note that they may not support all the regions, so check this in advance. Also, the customization options could be very **limited**, you may not be able to tune your database or the underlying hardware. You also need to check the security and legal implications, specially local laws around data protection if you have a 3rd party handling your data. This could be a no-go for many companies. Finally, this will be an **expensive** solution.

In my opinion, if you need to support a multi cloud environment, **Kubernetes** will be the best solution since it provides an unify solution across all cloud provider and on-prem. Also, more and more cloud providers and coming to the market and in the next few years we will see a lot more cloud providers and the only way to support them will be using Kubernetes.

Again, if the database does not support Kubernetes, you will need to use VMs.

## Conclusion

In this article, we have seems different options to deploy databases and what has become clear is that nowadays **Kubernetes provides the most portable and efficient platform to run stateful workloads**. It is the only way you have to support multiple cloud providers and on-prem. Thanks to the operators, you can easily deploy and manage databases. Do not use _helm charts_ to run databases in **production**, make sure an **Operator** exists for the targeted database.

If you run **microservices** in **Kubernetes**, [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) dictates that each microservice should have its own data store. This has always been a problem since deploying databases in Kubernetes was difficult in the past, so many companies what ended up doing was using managed databases, because of the cost and complexity, these managed services were run by a different team which will only support a very small number of databases. This means that developers were often sharing the database among different microservices which in an **anti pattern**. Kubernetes solves this issue **allowing each cross functional team to deploy microservices alongside databases** in an unify way where the team owns and manages the database without throwing it over the wall following **DevOps** best practices.

Kubernetes is still growing exponentially and nowadays most of the databases provide custom CRDs and **Operators** to run them in Kubernetes. If you are already using Kubernetes, running a database will be your best option since you will use a familiar API and syntax.

**Kubernetes is future proof**, has a huge user base and provides an unified management layer for any kind of workload allowing a true **DevOps** experience for everyone. You will define your resources in _YAML_ and deploy them regardless of the type of the resource, it can be a database, a microservice or a batch job. This makes management and monitoring a lot easier.

**Kubernetes** is also extremely **cost** **effective** and provides many different options to save costs, such as sharing resources, **spot instances** support and auto scaling.

I would make two important recommendations:

-   Always use the right database for the problem domain. Do not try to fit your model to a database but the other way around. I have seem companies trying to model graph data in relational databases because that is what it was available to them. This is a bad idea and Kubernetes solves this because you can run any database which opens the door to infinite possibilities.
-   Have a consisten and unify way to deploy workloads. Do not mix solutions if possible. Do not have some databases running in the cloud providers, other by 3rd party managed services, others in VMs, etc. The operational overhead to maintain different solutions is huge and should be avoided. Nowadays is normal that your company uses different databases, it will be hard that all of them are supported in your cloud provider and it will be very expensive or complex to have several 3rd party companies managing your data. **This is why I see Kubernetes as the only solution for modern cloud native SaaS companies who needs to support many environments and different workload types.**

To summarize, we have seem that due to its extreme growth, Kubernetes has become the de facto standard to deploy any workload in an efficient and cost effective way. Thanks to the new **operator pattern**, this now includes databases and other complex systems. **Kubernetes provide the most portable, secure, trustworthy, future proof, cost effective and reliable way to run databases in any environment.**