## Introduction 

In this blog, we share how we improved the daily edit-build-run developer experience using DevPods, our remote development environment. We will start with some of the initial challenges, the pain points we addressed with Devpod, our architecture, and some of our recent successes in terms of adoption and cost reduction. We will finally leave you with some thoughts around the future of remote development at Uber.

___

## Initial Pain Points

### State of Uber’s Codebase in 2017

Uber’s codebase in 2017 was fragmented and distributed across thousands of repositories with 10+ programming languages, 4000+ services, 500+ Web Apps, 9+ build tools and 6+ configuration tools. There were separate repos per microservice and app. This fragmentation resulted in a number of developer pain points including dependency management issues, library version fragmentation in production, build tool fragmentation, support for diverse tools and frameworks, collaboration, and friction in code sharing.

___

### Move to Monorepo

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=770,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Screenshot-2022-12-09-at-11.39.57-AM.png "Figure 1 - Move to monorepo")

Figure 1: Move to Monorepo

To address these issues, a cross-organizational working group developed a Repository and Build strategy that recommended moving to a monorepo hosting all of Uber’s code with trunk-based development and one single version per third-party library.

Our move to monorepo provided several important benefits including:

-   Better dependency management
-   Consistent and universal library versions in production
-   Single, centralized  build platform (Bazel)
-   Easier support for a standard set of tools and processes
-   Improved code visibility, collaboration, and code sharing

___

### Monorepo Challenges

While switching to monorepo provided Uber a strong foundation for our longer term success, it made the daily code edit-build-run development loop challenging on laptops: 

-   Builds were larger and took longer 
-   Several GB of often-changing artifacts need to be downloaded to the laptop or built locally
-   It was challenging to develop away from the fast corporate network at Uber offices. Sometimes, cloning a new project and setting up from scratch would take hours remotely.

In addition to all of these, maintaining a consistent set of tools, and keeping the local development on laptops up to date is a constant struggle (e.g., non-atomicity of updates; opaqueness of the steps taken).

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=634,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-2-Devpod-overview-Remote-development-environment-@-Uber.png)

Figure 2: Devpod Overview – Remote Development Environment at Uber

___

## Remote Development Using Devpod

As we looked for solutions to provide a  faster, easier, and more secure experience for our developers, we started focusing on remote development as an alternative. The idea of building in the cloud on a faster machine, getting started in just seconds and having all of the code base and tooling kept in a secure, controlled environment was intriguing. This was the beginning of **_Devpod_****: Uber’s remote development environment.**

### What is a Devpod?

A Devpod is a remote development environment, tailored for our developers’ needs. It is designed and implemented to provide the best possible developer experience with multiple IDE integrations (JetBrains®\* IDEs and  VS Code<sup>®</sup>\*\*) in mind. It runs as a container on Kubernetes<sup>®</sup>, with the necessary tooling and compute resources to enable seamless development on Uber’s large-scale monorepos.

### Devpod Advantages

We recognize _improved performance_, nearly _zero setup_ and _security_ to be primary advantages of remote development environments, with the first two being productivity drivers. We greatly improve security by running everyone on the latest stable version. Vulnerable applications can be patched and updated during non-work hours. It is fairly trivial to monitor a lot of different aspects of the remote development environment, and in theory it allows us to detect and identify malicious behavior. Since Kubernetes pods running in the cloud do not have laptop battery constraints, it is trivial to scan disk for malicious artifacts or activity during off-hours.

##### **Performance Improves Productivity**

Faster Git, Build, and IDE experience by utilizing beefy cloud resources–up to 48 cores and 96 GB of RAM per environment and a host of additional features:

-   IDE experience:
    -   Install and configure the latest version supported by the central developer experience team
    -   Preload cached IDE index
-   Git performance was improved by:
    -   Optimizing Git configuration
    -   The Linux file system, which performs better compared to laptop file systems
-   Build were improved by:
    -   Preloading dev artifacts required for builds
    -   Providing more compute resources
    -   Providing nearby remote build caches
-   On top of that, the remote dev environment also provides:
    -   No more laptop freezing/overheating
    -   Improved provisioned laptop battery life
    -   Multiple remote environments per user
    -   Remote development activities independent from other processes running on laptop

##### **Zero Setup and Maintenance**

Devpod comes with a setup of tools and configurations required to start working on a project in any of the main monorepos right off the bat:

-   Easy to provision–just one client command or a few button clicks in a web application to provision a new environment on demand
-   Ready to use IDE of choice–preconfigured VS Code or JetBrains IDE with preloaded indices
-   Access to consistent dev environment in minutes
-   Pre-installed, configured, and tested tool set, required by each monorepo
-   Similar to production environment OS, storage layer, and libraries
-   Pre-cloned repository

##### **Secure**

No need to worry about keeping the environment up to date–Devpods are updated overnight automatically with latest tooling and security upgrades:

-   Controlled toolchain–pre-curated secure environment with hardened tool chain all ready to go
-   Seamless configuration changes
-   Fast & automatic updates with latest tooling
-   Security scan can be performed on the image before promoting it to stable

Devpods are persistent, therefore engineers do not need to worry about losing their individual setup, files, and code changes. This allows engineers to continue their work on different devices and enables collaboration between multiple engineers in a single environment.

___

### Devpods Are Available in all Major Regions for Low-latency Access 

It is critical to have low latency for UI operations otherwise typing will feel sluggish and very unpleasant. For that purpose we have deployed Devpod to multiple regions across the globe, currently it is running in:

-   Oregon, US
-   Virginia, US
-   Netherlands, EU
-   Mumbai, IN
-   São Paulo, BR

##### **Devpod Global Presence**

### Devpod Flavors

A “flavor” is a Devpod docker image tailored for a specific set of engineers. Flavors include all required tools with the default settings and configuration preloaded. We currently support the following flavors:

-   Go
-   Java
-   Android
-   Web
-   ML 
-   Base

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=584,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-4-Devpod-flavors-out-of-the-box-configurations.png)

Figure 3: Devpod Flavors – Out of the Box Configurations

___

### Devpod Central: Web-Based Management Interface

To provide our users a better first time experience with Devpods, and to improve troubleshooting, we built a simple web-based management interface for Devpods. Our goal is to extend Devpod Central and bring visibility into resource usage and consumption. 

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=535,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-5-Devpod-Central.png)

Figure 4: Devpod Central – Management Portal

___

## Architecture

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=520,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-6-Devpod-architecture.png "Figure 6 - Devpod architecture")

Figure 5: Devpod Architecture

### Devpod Images

Putting all the tooling and configuration into the docker image made perfect sense. Docker<sup>®</sup> images are trivial to deploy almost anywhere. Many engineers are familiar with Docker, which makes it easy for everyone to contribute.

Now each Docker image has to have a base to start from. After discussing it, we decided that matching production operating systems would make the most sense. Uber uses **Debian** for production workloads. This means that we can reuse already existing infrastructure for building and distributing internal packages. Critical software configuration can be reused by simply building on top of the production base image.

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=533,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-7-Devpod-image-hierarchy-1.png)

Figure 6: Devpod Image Hierarchy

### Using Kubernetes as a Platform

Moving from laptop to cloud enabled us to experiment with machines that have 90+ cores and hundreds of Gigabytes of RAM. On top of that we decided to go with kubernetes since it provided us with all the low-level building blocks we needed:

-   The ability to host containers on powerful hardware
-   Networking to connect containers
-   Persistent volumes to store engineering work between restarts

### Devpod Operator

As we started using Kubernetes to run Devpods, it made perfect sense for us to start using [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/), which is a way of extending Kubernetes by providing a data structure that would be stored inside the control plane. That removed the need for an internal database for Devpod metadata. By using a custom controller, we were able to create and access Devpods using _kubectl_. 

Another great feature of Kubernetes’ custom resource definitions is the ability to write a piece of software that reacts to custom resources changes. It’s called the [operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)–by using it we have combined our custom resources and small custom controller to translate a high-level Devpod description into standard Kubernetes primitives that can be handled by standard Kubernetes controllers. That custom controller was so small at first–we even started to joke that it could be ported to Bash, but luckily we didn’t.

Later on we extended our custom controller to react to VolumeSnapshotContents and PersistentVolumeClaim in order to cut the cost when Devpod is not used.

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=541,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-8-Devpod-operator.png "Figure 8 - Devpod operator")

Figure 7: Devpod Operator

### Devpod Deployment

We wanted to run Devpods on Kubernetes clusters outside of our production data centers for the following reasons:

-   Keeping the shell services for developers outside of Uber’s production data centers provided us with strong isolation.
-   We wanted to be able to cater to our growing number of engineers in APAC and EMEA. By keeping our Kubernetes clusters in closer proximity to our development centers we were able to provide a smooth experience for our developers (Low latency remains crucial for good developer experience while remote development has evolved to be somewhat latency-independent i.e. local files; no round-trip per keystroke).
-   Leveraging Kubernetes as a platform allowed us to focus on what is strategic to us: fast, easy, and secure developer experience for Uber developers. 
-   Builds are inherently file-system reliant. Having fast storage connected to our Kubernetes clusters was critical to run Devpods. While it is possible to work with ephemeral file systems backed by object storage, doing so is hard work and inconvenient, but still something we would like to explore in the future.

___

## Challenges 

Creating the perfect environment for engineers does not come easy–we have encountered a handful of challenges along the way. Balancing between performance and cost efficiency, providing automatic upgrades, ensuring uninterrupted work, and pre-configuring IDE settings that work for everyone were some of the obstacles that we had to overcome.

### IDE Experience

Engineers use IDEs daily, so a remote environment without a great IDE experience cannot be successful. Over the course of Devpod’s development we have provided multiple different IDE options:

-   VS Code remote
-   VS Code over web
-   JetBrains IDEs via web using [JetBrains Projector](https://lp.jetbrains.com/projector/) (deprecated) 
-   JetBrains IDEs via ssh using [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway)
-   Vim and Emacs via terminal

#### Devpod Currently Supports the the Following Ways to Connect:

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=396,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-9-Devpod-IDE-options-.png)

Figure 8: Devpod IDE Options

Besides providing multiple IDE options, we focused on fine-tuning the hands-on experience by:

-   Pre-loading indices
-   Pre-configuring settings
-   Pre-installing tools, extensions, and custom add-ons

Devpod provides a fast, local development experience with the powerful instances, however the IDE experience has been one of our biggest challenges from the early days. In particular, the latency issues with the popular web-based IDEs in early days and subsequently some of the stability issues created setbacks for us. 

VS Code remote and WEB have superior performance when it comes to remote workloads, yet there is still much to be desired when it comes to intelligent features like language understanding. Language Server Protocol is a great step forward, yet a lot of implementations still have a long way to go to meet our needs. For example, saving and reloading indexes to and from disk would be a great feature that not a lot of language servers support. Luckily this is a protocol and most of the language servers are open source, which gives us an opportunity to contribute back to the community.

### Keeping the Environments Up to Date

As engineers value their time, it is important to provide an environment that does not require manual maintenance, therefore, we have automated upgrading the environments with the latest tooling and security updates during non-working hours. 

To accommodate everyone’s needs, we allow engineers to choose from one of the four release cadence channels: 

-   stable – default one
-   rc – release candidate for next stable
-   dev – updated nightly to latest successful build
-   none – no automated updates

Whether an engineer  wants the most stable environment and cutting edge features or doesn’t want any upgrades at all, we’ve got them covered.

Later on we improved automated updates to support a gradual new version rollout. This was added to reduce the blast radius in case a bug slips through our automated tests and release candidate, dogfooding process.

### Cost Efficiency

It’s not uncommon to find yourself in a build vs. buy situation–to justify building an in-house solution, we continuously track and improve the cost per Devpod and make sure the price/performance for Devpod is better than its over the shelf alternatives.

Since Devpod are not ephemeral, we had to ensure that we use the compute and storage resources  efficiently, and so we implemented several improvements:

-   Shutting down inactive environments: To save compute resources, an automated job periodically checks whether an environment has been used lately, and upon detecting an inactive one, its container is deleted.
-   Rebalancing VMs: Since multiple environments are placed in one big virtual machine and we are paying for the VMs, it only makes sense to use one to its fullest capacity. Therefore, if there’s two VMs that are only used for half of their capacity, we move the load to one machine and shut the other one down.
-   Snapshotting disks of shutdown environments: A Devpod requires a container and a disk to function. When we shut down the container, the disk is no longer used, and so we convert it into a low-cost storage option until the environment is started up again.

So far we managed to keep the price way lower compared to off-the-shelf solutions with significant performance improvements.

___

## Tracked SLO Metrics 

Uber’s developer platform serves 5000 core software engineers to build, deploy, and manage high-quality software productively and at scale. We understand the heavy responsibility that comes with providing a stable experience for Uber developers. The Devpod team tracks service level indicator (SLI) metrics to stay on top of its service level objectives (SLOs).

As of right now, the key indicator metrics are:

-   **System availability:** Providing as many nines as possible in the availability SLO is crucial for a system on which most of Uber engineers rely. Unfortunately, here we are limited by the cloud provider availability that’s capped at 99.5%.
-   **Devpod readiness**: We are closely monitoring the expected to run vs. ready-to-use Devpods, as each instance impacts an engineer. If an environment does not start for longer than expected, we receive an alert and investigate the root cause of the issue.
-   **Connection health**: Since it’s important to mirror the local development capabilities on Devpods, we must ensure connectivity between our infrastructure and Uber’s internal services, thus we are constantly probing multiple endpoints to know about the connectivity status.

One of the issues that we encountered in the availability metric is our maintenance windows: even when the system is not supposed to be fully operational, we used to receive critical alerts, which introduced additional load for the team and sometimes even woke us up during the night time. Hence, we started tracking the maintenance window periods and excluding this timeframe from alerts logic. Grafana<sup>®</sup> dashboards allow us to compare the maintenance window together with the uptime metric that we’re monitoring:

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=389,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-10-Devpod-key-SLOs-Uptime.png "Figure 10 - Devpod key SLOs - Uptime")

Figure 9: Devpod Key SLOs – Uptime

___

## Our Progress So Far 

Devpods have been effective in reducing the local Git command performance, as shown in the figure below. The times shown below are p95 git status times in seconds for our largest monorepo with over 70 million lines of code. 

This chart below shows that p95 Git status times have been consistently below 4 seconds while the Git status performance on laptops have gone up with the growing monorepo sizes initially, until we implement the filesystem monitor and sparse checkouts on laptops. 

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=393,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-11-Improvements-in-Git-Status-p95.png "Figure 11 - Improvements in Git Status(p95)")

Figure 10: Improvements in Git Status (p95)

Similarly, Devpods consistently provide better performance for our longest full builds, as shown in the following chart for p95 latency for Go builds. Devpods provide a 2.5X improvement over laptops on average for longer, more complex builds and 1.8X improvement over laptops for average builds (p75). 

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=387,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-12-Improvements-in-Go-build-times-p95.png "Figure 12 - Improvements in Go build times(p95)")

Figure 11: Improvements in Go Build Times (p95)

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=398,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-13-Improvements-in-Go-build-times-p75-1.png "Figure 13 - Improvements in Go build times(p75)")

Figure 12: Improvements in Go Build Times (p75)

With all these improvements, we have seen an organic adoption in Devpod usage across the organization. As of November 2022, over 60% of Uber software engineers have adopted Devpods. 

![](https://blog.uber-cdn.com/cdn-cgi/image/width=1024,height=504,fit=crop,quality=80,onerror=redirect,format=auto/wp-content/uploads/2022/12/Figure-14-Devpod-adoption.png "Figure 14 - Devpod adoption")

Figure 13: Devpod Adoption

### Uber Developers Love Devpods

Devpod has consistently been seen as one of the top productivity boosters by Uber developers, based on their NPS feedback. Our NPS survey shows that most Uber developers love Devpods primarily for the build speed improvements it provides. Developers mentioned that they feel much more productive and efficient with Devpods. 

___

## What’s Next 

![Roadmap banner](https://lh3.googleusercontent.com/A6bV3WFUfvxLQuvsOfF3bscAbBavRFngzM3e8kx3vi6a1-UjdTsGlEsZoWldGM-nFipd86bgduxHiSUdPXxP4whGDp4E3du8jzBHtT-pe9eIxpWk8_awjkrd52TdP8ENfp0FOhzrDUC4q154Yb_KPkjhF1IRjFLxW4-q6aKMggxiTLd04LtLZi28PSxfwg "Roadmap banner")

We see Devpod as the future of Uber’s engineering. Our goal is to make remote development completely seamless for Uber engineers. A few areas that we would like to explore in the near future include the following:

##### **Devpod-On-Demand**

Currently it takes 2-3 minutes to create a new Devpod. We would like to reduce this initial creation time to less than 3 seconds by pre-provisioning a set of Devpods for each flavor and applying final touches before assigning them to the requesting user.

##### **Ephemeral Devpod: Short-Lived Devpod Environments**

Once we deliver the on-demand Devpod and reduce the time it takes to get a new Devpod creation to sub 3 seconds, we would like to build purposeful Devpod integrations from most common tools in developers’ day to day experience. We foresee the following use cases for the short-lived Devpod:

-   Devpod specific to feature development
-   Easy and fast failed CI debugging
-   Ready-to-go code review environment
-   Analyzing mobile crashes

##### **Non-Disruptive Auto-Upgrade and Maintenance Workloads** 

Currently, there are maintenance windows set for the environments during non-work hours, however, some engineers may want to work at odd hours or run a longer running workload on their environment. Therefore, an improvement can be done here by monitoring active connections and deferring maintenance workloads.

##### **Auto-Pilot for Cost Savings and Performance**

We would like to have a more granular scheduling and power-off policies for further cost reductions and eliminate idle Devpods without disrupting the user flows.

##### **Seamless IDE Experience** 

It should hide the remote environment implementation in the background from the engineers as they use IDEs locally on a laptop. The IDE would silently connect to the remote environment in the background and provide all the benefits of big computational power while there’s a good enough network available–if anything goes wrong with the remote environment, they are not blocked and can seamlessly continue working locally.

##### **Centralized Build Farm for All Environments (CI/ Devpod/Local)**

We would like to introduce a single BuildFarm cluster with both remote cache and remote execution serving CI, laptops, and Devpods. Our plan is to run this compute-intensive cluster on our data centers and leverage the excess compute capacity with preemptible instances. We would like to also have a geo-distributed remote cache to minimize the latencies and provide fast downloads for the build artifacts.

##### **Team-Specific Devpod Configuration**

We are looking for ways to minimize the first time setup for teams using Devpod. One of the areas we are looking to improve is to allow teams to customize their Devpod configuration so that new team members can get a consistent development environment tailored to their needs with a single click.

##### **Seamless File Transfer Between Laptops and Devpods**

Uber engineers use Devpods for compute intensive tasks and long-running builds. In some cases, developers need to move files between their laptops and Devpods (and vice versa). Our goal is to auto-mount the Devpod drive to user laptops so the files can be moved seamlessly.

##### **Android Emulator Support for Devpod**

Devpods improve build times significantly for Android developers. When it comes to testing and debugging changes in the remote IDE, the experience is far from ideal since we do not have an Android emulator or an actual divide that can be connected with ADB from Devpods. Our goal is to provide a seamless debugging experience for Android  developers with Devpods (including built-in plugins with the Devpod-supported  IDEs).

\* Copyright © 2022 JetBrains s.r.o. JetBrains Gateway and the Gateway logo are registered trademarks of JetBrains s.r.o. \*\* Microsoft and VS Code are trademarks of the Microsoft group of companies.