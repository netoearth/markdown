![A High-level diagram of the components used to monitor QuestDB.](https://questdb.io/img/blog/2022-12-13/banner.png)

Photo by [Chris Liverani](https://unsplash.com/@chrisliverani) via [Unsplash](https://unsplash.com/)

One of our Cloud engineers, [Steve Sklar](https://github.com/sklarsa), shares with us how to use some of the most popular tools in the Kubernetes ecosystem to build monitoring infrastructure for your QuestDB instances.

## Monitoring QuestDB in Kubernetes[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#monitoring-questdb-in-kubernetes "Direct link to heading")

As any experienced infrastructure operator will tell you, monitoring and observability tools are critical for supporting production cloud services. Real-time analytics and logs help to detect anomalies and aid in debugging, ultimately improving the ability of a team to recover from (and even prevent) incidents. Since container technologies are drastically changing the infrastructure world, new tools are constantly emerging to help solve these problems. Kubernetes and its ecosystem have addressed the need for infrastructure monitoring with a variety of newly emerging solutions. Thanks to the orchestration benefits that Kubernetes provides, these tools are easy to install, maintain, and use.

Luckily, QuestDB is built with these concerns in mind. From the presence of core database features to the support for orchestration tooling, QuestDB is easy to deploy on containerized infrastructure. This tutorial will describe how to use today's most popular open source tooling to monitor your QuestDB instance running in a Kubernetes cluster.

## Components[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#components "Direct link to heading")

Our goal is to deploy a QuestDB instance on a Kubernetes cluster while also connecting it to centralized metrics and logging systems. We will be installing the following components in our cluster:

-   A [QuestDB](https://questdb.io/) database server
-   [Prometheus](https://prometheus.io/) to collect and store QuestDB metrics
-   [Loki](https://grafana.com/oss/loki/) to store logs from QuestDB
-   [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/) to ship logs to Loki
-   [Grafana](https://grafana.com/oss/grafana) to build dashboards with data from Prometheus and Loki

These components work together as illustrated in the diagram below:

![Diagram showing how components work together](https://questdb.io/img/blog/2022-12-13/overview.png)

Overview of the architecture

## Prerequisites[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#prerequisites "Direct link to heading")

To follow this tutorial, we will need the following tools. For our Kubernetes cluster, we will be using [kind](https://kind.sigs.k8s.io/) (Kubernetes In Docker) to test the installation and components in an isolated sandbox, although you are free to use any Kubernetes flavor to follow along.

-   [docker](https://www.docker.com/) or [podman](https://podman.io/)
-   [kind](https://kind.sigs.k8s.io/)
-   [kubectl](https://kubernetes.io/docs/reference/kubectl/)
-   [jq](https://stedolan.github.io/jq/)
-   [curl](https://curl.se/)

### Getting started[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#getting-started "Direct link to heading")

Once you've [installed kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation), you can create a Kubernetes cluster with the following command:

This will spin up a single-node Kubernetes cluster inside a Docker container and also modify your current `kubeconfig` context to point `kubectl` to the cluster's API server.

## QuestDB[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#questdb "Direct link to heading")

### QuestDB endpoint[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#questdb-endpoint "Direct link to heading")

QuestDB exposes an HTTP metrics endpoint that can be scraped by Prometheus. This endpoint, on port `9003`, will return a wide variety of QuestDB-specific metrics including query, memory usage, and performance statistics. A full list of metrics can be found [in the QuestDB docs](https://questdb.io/docs/third-party-tools/prometheus/).

### Helm installation[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#helm-installation "Direct link to heading")

[QuestDB](https://questdb.io/get-questdb/) can be installed using [Helm](https://helm.sh/). You can add the official Helm repo to your registry by running the following commands:

This is only compatible with the Helm chart version 0.25.0 and higher. To confirm your QuestDB chart version, run the following command:

Before installing QuestDB, we need to enable the metrics endpoint. To do this, we can override the QuestDB server configuration in a `values.yaml` file:

Once you've added the repo, you can install QuestDB in the default namespace:

To test the installation, you can make an HTTP request to the metrics endpoint. First, you need to create a Kubernetes port forward from the QuestDB pod to your localhost:

Next, make a request to the metrics endpoint:

You should see a variety of Prometheus metrics in the response:

## Prometheus[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#prometheus "Direct link to heading")

Now that we've exposed our metrics HTTP endpoint, we can deploy a [Prometheus](https://prometheus.io/) instance to scrape the endpoint and store historical data for querying.

### Helm installation[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#helm-installation-1 "Direct link to heading")

Currently, the recommended way of installing Prometheus is using the [official Helm chart](https://github.com/prometheus-community/helm-charts). You can add the Prometheus chart to your local registry in the same way that we added the QuestDB registry above:

As of this writing, we are using the Prometheus chart version `19.0.1` and app version `v2.40.5`

### Configuration[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#configuration "Direct link to heading")

Before installing the chart, we need to configure Prometheus to scrape the QuestDB metrics endpoint. To do this, we will need to add our additional scrape configs to a `prom-values.yaml` file:

This config will make Prometheus scrape our QuestDB metrics endpoint every 15 seconds. Note that we are using the internal [service URL](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#namespaces-of-services) provided to us by Kubernetes, which is only available to resources inside the cluster.

We're now ready to install the Prometheus chart. To do so, you can run the following command:

It may take around a minute for the application to become responsive as it sets itself up inside the cluster. To validate that the server is scraping the QuestDB metrics, we can query the Prometheus server for a metric. First, we need to open up another port forward:

Now we can run a query for available metrics after waiting for a minute or so. We are using [jq](https://stedolan.github.io/jq/) to filter the output to only the QuestDB metrics:

You should see a list of QuestDB metrics returned:

## Loki[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#loki "Direct link to heading")

Metrics are only part of the application support story. We still need a way to aggregate and access application logs for better insight into QuestDB's performance and behavior. While `kubectl logs` is fine for local development and debugging, we will eventually need a production-ready solution that does not require the use of admin tooling. We will use Grafana's [Loki](https://grafana.com/oss/loki/), a scalable open-source solution that has tight Kubernetes integration.

### Helm installation[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#helm-installation-2 "Direct link to heading")

Like the other components we worked with, we will also be installing Loki using an official Helm chart, [loki-stack](https://github.com/grafana/helm-charts/tree/main/charts/loki-stack). The loki-stack helm chart includes Loki, used as the log database, and [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/), a log shipper that is used to populate the Loki database.

First, lets add the chart to our registry:

Loki and Promtail are both enabled out of the box, so all we have to do is install the Helm chart without even supplying our own `values.yaml`.

After around a minute or two, the application should be ready to go. To test that Promtail is shipping QuestDB logs to Loki, we first need to generate a few logs on our QuestDB instance. We can do this by `curl`ing the QuestDB HTTP frontend to generate a few `INFO`\-level logs. This is exposed on a different port than the metrics endpoint, so we need to open up another port forward first.

Now navigate to [http://localhost:9000](http://localhost:9000/), which should point to the QuestDB HTTP frontend. Your browser should make a request that causes QuestDB to emit a few INFO-level logs.

You can query Loki to check if Promtail picked up and shipped those logs. Like the other components, we need to set up a port forward to access the Loki REST API before running the query.

Now, you can run the following [LogQL](https://grafana.com/docs/loki/latest/logql/) query against the Loki server to return these logs. By default, Loki will look for logs at most an hour old. We will also be using `jq` to filter the response data.

You should see a list of logs with timestamps that correspond to the logs from the above sample:

## Grafana[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#grafana "Direct link to heading")

Now that we have all of our observability components set up, we need an easy way to aggregate our metrics and logs into meaningful and actionable dashboards. We will install and configure [Grafana](https://grafana.com/oss/grafana/) inside your cluster to visualize your metrics and logs in one easy-to-use place.

### Helm Installation[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#helm-installation-3 "Direct link to heading")

The `loki-stack` chart makes this very easy for us to do. We just need to enable Grafana by customizing the chart's `values.yaml` and upgrading it.

With this setting enabled, not only are we installing Grafana, but we are also registering Loki as a data source in Grafana to save us the extra work.

Now we can upgrade our Loki stack to include Grafana:

To get the admin password for Grafana, you can run the following command:

And to access the Grafana frontend, you can use a port forward:

### Configuration[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#configuration-1 "Direct link to heading")

First navigate to [http://localhost:3000](http://localhost:3000/) in your browser. You can log in using the username `admin` and the password that you obtained in the previous step.

![Grafana login](https://questdb.io/assets/images/grafana-login-21ddb15f59224e1b7c750466f7a728d1.png)

Once you've logged in, use the sidebar to navigate to the "data sources" tab:

![Grafana](https://questdb.io/assets/images/grafana-main-page-ec647c64ca2c42ec6813b399f8e33a02.png)

Here, you can see that the Loki data source is already registered for us:

![Grafana data sources](data:image/png;base64,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)

We still need to add our Prometheus data source. Luckily, Grafana makes this easy for us.

Click "Add Data Source" in the upper right and select "Prometheus". From here, the only thing you need to do is enter the internal cluster URL of your Prometheus server's Service: `http://prometheus-server.default.svc.cluster.local`. Scroll down to the bottom, click "Save & test", and wait for the green checkmark popup in the right corner.

![Grafana data source config](https://questdb.io/assets/images/grafana-data-source-config-c894e00111ca5776be0081d5b3436d36.png)

Now you're ready to create dashboards with QuestDB metrics and logs!

## Conclusion[#](https://questdb.io/blog/2022/12/13/using-prometheus-loki-grafana-monitor-questdb-kubernetes/#conclusion "Direct link to heading")

I have provided a step-by-step tutorial to install and deploy QuestDB with a monitoring infrastructure in a Kubernetes cluster. While there may be additional considerations to make if you want to improve the reliability of the monitoring components, you can get very far with a setup just like this one. Here are a few ideas:

-   Add alerting to a number of targets using [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
-   Build interactive dashboards that combine metrics and logs using [Grafana variables](https://grafana.com/docs/grafana/latest/dashboards/variables/)
-   Configure Loki to use [alternative deployment modes](https://grafana.com/docs/loki/latest/fundamentals/architecture/deployment-modes/) to improve reliability and scalability
-   Leverage [Thanos](https://thanos.io/) incorporate high availability into your Prometheus deployment

If you like this content, we'd love to know your thoughts! Feel free to share your feedback or just come and say hello in the [QuestDB Community Slack](https://slack.questdb.io/).