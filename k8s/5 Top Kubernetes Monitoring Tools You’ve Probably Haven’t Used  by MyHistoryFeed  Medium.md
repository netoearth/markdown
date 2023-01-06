_Follow me to support my article writing._

![](https://miro.medium.com/max/700/1*R27xLPjkKBTMLDkUtmid-Q.png)

Do you use Kubernetes on a regular basis? There are so many tools available that it can be difficult to decide which ones to use. Here are a few that I’ve found useful but may have slipped under your radar.

So these aren’t the first tools you’ll install on a cluster, but rather hidden gems deserving of more public attention. With the right tools, running Kubernetes is much less like brain surgery.

## Kubernetes Tools:

-   [5 of the Top Kubernetes Monitoring Tools for 2022 (Part 1)](https://myhistoryfeed.medium.com/5-of-the-top-kubernetes-monitoring-tools-for-2022-part-1-f6b417a9a4eb)
-   [Another 5 of the Top Kubernetes Monitoring Tools for 2022 (Part 2)](https://myhistoryfeed.medium.com/another-5-of-the-top-kubernetes-monitoring-tools-for-2022-part-2-67ed56230113)

## All About Kubernetes:

-   [_Adopting to Kubernetes: Does It Make Sense for You?_](https://myhistoryfeed.medium.com/adopting-to-kubernetes-does-it-make-sense-for-you-9b80046a9428)
-   [_Adopting to Kubernetes: Best Practices_](https://myhistoryfeed.medium.com/adopting-to-kubernetes-best-practices-c4d23b05706b)
-   [_What’s the Difference Between Kubernetes and Docker?_](https://myhistoryfeed.medium.com/whats-the-difference-between-kubernetes-and-docker-6d31aee1b426)
-   [_Major Kubernetes Security Issues and How to Solve Them_](https://myhistoryfeed.medium.com/major-kubernetes-security-issues-and-how-to-solve-them-8292583ac520)

## Top Kubernetes Monitoring Tools

## Kubewatch

Kubewatch is a Kubernetes watcher that sends notifications to collaboration hubs and notification channels. It runs in your Kubernetes cluster and sends event notifications via webhooks.

Once the Kubewatch pod is up and running, you will begin to see Kubernetes events in your configured Slack channel or any other webhook.

Pros:

-   Simple installation
-   Notifications to preferred locations are sent instantly.

Cons:

-   It only monitors basic events.
-   It does not support long-term storage, trending, or analysis.

## NetApp Cloud Insights

![](https://miro.medium.com/max/700/0*TAYIYFj-9vsNvXbp)

NetApp Cloud Insights is a tool for monitoring infrastructure. Cloud Insights allows you to monitor, troubleshoot, and optimize your resources across public and private clouds.

NetApp provides seamless cluster navigation and observability, persistent storage that allows you to correlate storage utilization to workloads, and full-stack visualization to help you understand individual metrics in context.

Simple yet dynamic dashboards ensure data visualization by allowing you to overview critical Kubernetes KPI, such as restart counts, calling metrics, pods, and containers that experience outages, instability, or resource-related issues.

Pros:

-   Full-stack visibility
-   Dashboards that are well-designed

## Kube-state-metrics

The Kubernetes API server publishes information about the number, health, and availability of pods, nodes, and other Kubernetes objects. The kube-state-metrics add-on simplifies the consumption of these metrics and aids in the detection of issues with cluster infrastructure, resource constraints, or pod scheduling.

How does kube-state-metrics function? It monitors the Kubernetes API and produces metrics about the state of Kubernetes objects. These include node status, node capacity such as CPU and memory, the number of desired/available/unavailable/updated replicas per Deployment, and various pod statuses such as waiting, running, and ready.

When you install kube-state-metrics on your cluster, it will provide a plethora of metrics in text format via an HTTP endpoint. Any monitoring system that can collect Prometheus metrics can easily consume these metrics.

Pros:

-   Simple installation
-   Prometheus-compatible

Cons:

-   It only monitors the most basic Kubernetes API metrics.
-   It does not support long-term storage, trending, or analysis.

## Sensu Go

![](https://miro.medium.com/max/700/0*TCbWs8SYnA_JPMlu)

Sensu Go provides a multi-cloud monitoring service health and telemetry solution. It enables you to comprehend how your servers, containers, services, apps, and devices function and interact across public and private clouds.

Sensu Go is frequently seen running alongside Prometheus. However, it is not required. Thanks to pre-defined templates, you can run custom scripts and plugins, collect metrics about resource usage, monitor and manage cloud endpoints, or deploy a monitoring solution without coding.

Pros:

-   Smart alerts possible thanks to PagerDuty, ServiceNow, and Jira integrations
-   Code-free workflow option

## Datadog

![](https://miro.medium.com/max/700/1*1ZwESAu4Cjjxj-OmidIBPA.png)

Datadog is an APM solution that allows you to extract real-time logs, metrics, events, and service states from Kubernetes. It allows you to monitor, troubleshoot, and optimize the performance of your applications.

Datadog has dashboards as well as high-resolution metrics and events for manipulation and graphing. You can also set up alerts and receive notifications through different channels, such as Slack and PagerDuty.

The Datadog Agent is simple to set up. You can run it by deploying a DaemonSet to every cluster node.

After successfully deploying the Datadog Agent, resource metrics and events will begin to flow into Datadog. The data can be viewed in Datadog’s built-in Kubernetes dashboard.

Pros:

-   Simple to set up
-   Excellent APM integration

Cons:

-   Log integrations that are perplexing
-   Plans are limited (only 1)
-   Expensive

## Conclusion

![](https://miro.medium.com/max/700/1*cu1yyN_pTZy9M8Bv6eg-cQ.jpeg)

The tool you select will be determined by your monitoring requirements and use case.

When you first enter the Kubernetes ecosystem, you will almost certainly come across CNCF projects, and you will almost certainly try them first. If you have the resources, you can run such tools in your own cluster and manage it yourself. They have the advantage of being supported by large, active communities dedicated to improving existing solutions, so you’ll almost certainly find assistance if you need it.

SaaS software is supported by experts and, most importantly, it eliminates the overhead of managing the tools to monitor Kubernetes yourself. It assists you in managing the complexity of it all while you focus on creating value and building your own product.

**See you in my next post!**

> **_Check out my article_**_:_ [_Adopting to Kubernetes: Best Practices_](https://medium.com/p/c4d23b05706b) _that is getting a lot of others interested and leave a comment to tell me what you think!_