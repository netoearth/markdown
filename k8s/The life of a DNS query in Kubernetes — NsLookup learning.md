1.  [Home](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/)
    

3.  The life of a DNS query in Kubernetes
    

[Learning center](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/)Copy article link Updated December 22, 2022**In Kubernetes, DNS queries follow a specific path to resolve the IP address of a hostname. In this blog post, you will learn the life of a DNS query in Kubernetes step-by-step.**

Service discovery in Kubernetes is a crucial component of the overall architecture. It allows incoming requests to be routed to the correct workloads running in the cluster. DNS plays a key role in this process.

Understanding how DNS and service discovery work in Kubernetes can be helpful for debugging issues. It allows you to better understand the flow of traffic in the cluster and diagnose any issues that may arise.

**Here's the TLDR.** When a pod performs a DNS lookup, the query is first sent to the DNS cache on the node where the pod is running. If the cache does not contain the IP address for the requested hostname, the query is forwarded to the cluster DNS server. This server handles service discovery in Kubernetes.

The cluster DNS server determines the IP address by consulting the Kubernetes service registry. This registry contains a mapping of service names to their corresponding IP addresses. This allows the cluster DNS server to return the correct IP address to the requesting pod.

The information in this blog post might not apply to every single Kubernetes cluster. However, most of the well-known Kubernetes distributions will behave as shown.

![Email and spam prevention is configured in the DNS](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/img/the-life-of-a-dns-query-in-kubernetes.206f61f8.jpg "Email and spam prevention is configured in the DNS")

Two developers tracing a DNS query through the Kubernetes landscape. Photo by [Caterina Beleffi](https://unsplash.com/@caterinabeleffi).

## Services

As previously mentioned, DNS is a necessary component of service discovery. The way service discovery works in Kubernetes is through the use of service resources. A service has an IP that, when accessed, redirects connections to a healthy pod backing that service.

When a service is created in Kubernetes, the cluster DNS server creates an [A record](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/dns-record-types/a/) for the service. This record maps the service's DNS name to its IP address. This allows pods to access the service using its DNS name. The DNS server also updates the A-record whenever the IP address of the service changes. This ensures that the DNS name always points to the correct IP address.

An example service definition looks like this:

```
apiVersion: v1
kind: Service
metadata:
  name: foo
  namespace: bar
spec:
  ports:
    - port: 80
      name: http
```

The [A record](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/dns-record-types/a/) and [SRV record](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/dns-record-types/srv/) (discussed later in the article) that are created in this instance look like this:

```
foo.bar.svc.cluster.local                  30   A   10.129.1.26
_http._tcp.nginx.default.svc.cluster.local 3600 SRV 0 100 80 10-129-1-26.foo.bar.svc.cluster.local.
```

To create the fully qualified domain name for this service, we use the name of the service (`foo`), the namespace (`bar`), and the cluster domain (`cluster.local`).

Any workload running in the cluster can now resolve the service's IP address using this DNS name.

## DNS lookups on services

![Email and spam prevention is configured in the DNS](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/img/flow-of-a-dns-query-in-kubernetes.8671b839.png "Email and spam prevention is configured in the DNS")

The flow of a DNS query in Kubernetes. By NsLookup.io. Licensed under [CC By 4.0](https://creativecommons.org/licenses/by/4.0/ "Creative Commons By 4.0 licence")

When a pod performs a DNS lookup, the query is first sent to the local DNS resolver in the pod. This resolver uses the resolv.conf configuration file. In this file, the nodelocaldns server is set up as the default recursive DNS resolver, which acts as a cache.

If this cache does not contain the IP address for the requested hostname, the query is forwarded to the cluster DNS server ([CoreDNS](https://coredns.io/)).

This DNS server determines the IP address by consulting the Kubernetes service registry. This registry contains a mapping of service names to their corresponding IP addresses. This allows the cluster DNS server to return the correct IP address to the requesting pod.

Any domains that are queried but are not in the Kubernetes service registry are forwarded to an upstream DNS server.

We will go through each of these components in more detail step-by-step.

### Pod foo

When a pod sends an API request to a service within the same Kubernetes cluster, it must first resolve the IP address of the service. To do this, the pod performs a DNS lookup using the DNS server specified in its [/etc/resolv.conf](https://en.wikipedia.org/wiki/Resolv.conf) configuration file.

This file, which is provisioned by the Kubelet, defines the settings for DNS lookups in the pod. It contains a reference to the cluster DNS server.

By default, this configuration file looks something like this:

```
search namespace.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.123.0.10
options ndots:5
```

### Pod config

By default, the `/etc/resolv.conf` file provided by the Kubelet will forward all DNS queries to the cluster's DNS server (10.123.0.10 in the example above). The Kubelet also defines search domains and the `ndots` option for DNS queries.

The search domains specify which domain suffixes should be searched when incomplete domains (non-FQDNs) are given. The `ndots` option determines when a query for the absolute domain is made directly instead of first appending the search domains.

To better understand how this works, let's look at an example. Suppose a pod named foo performs a DNS lookup for `bar.other-ns`. If the `ndots` option is set to 5 (the default value), the resolver will count the number of dots in the domain.

If there are fewer than 5 dots, the search domains will be appended before the DNS lookup is performed on the DNS server. If there are 5 or more dots, the domain will be queried as-is without appending the search domains. In this example, `bar.other-ns` has less than 5 dots, so the search domains will be appended before the DNS lookup is performed.

By default, the search domains are:

-   <requester namespace>.svc.cluster.local
-   svc.cluster.local
-   cluster.local

Until a valid response is found, these search domains are appended to the domain and queried. The resolver will try the following queries one by one:

-   bar.other-ns.<requester namespace>.svc.cluster.local
-   bar.other-ns.svc.cluster.local (⇐ match found!)

The bar service will be listening on `bar.other-ns.svc.cluster.local`, so a match is found and the proper A-record is returned.

To change the behavior of a pod's DNS resolver, you can change the DNS config of a pod:

```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

In the example above, the `dnsPolicy` is set to "None", which means that the pod will not use the default DNS settings provided by the cluster. Instead, the `dnsConfig` field is used to specify custom DNS settings for the pod.

The `nameservers` field is used to specify the DNS servers that the pod should use for DNS lookups. The searches field is used to specify the search domains that should be used for incomplete domains.

The `options` field is used to specify custom options for the DNS resolver, such as the `ndots` and `edns0` options in the example above.

These settings will be used by the pod's DNS resolver instead of the default settings provided by the cluster. For more information on pod DNS configuration, see the [official docs](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-dns-config).

### Authoritative DNS server

In Kubernetes clusters up to version 1.13, kube-dns acted as the [authoritative DNS server](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/recursive-vs-authoritative-dns/) for Kubernetes. In Kubernetes version 1.13, [CoreDNS replaced kube-dns](https://kubernetes.io/blog/2018/12/03/kubernetes-1-13-release-announcement/#coredns-is-now-the-default-dns-server-for-kubernetes) as the default component for authoritative DNS queries.

The DNS server adds all services to its authoritative [DNS zone](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/learning/what-is-a-dns-zone/), so that it can resolve domain names to IP addresses for Kubernetes services. Various software implementations exist for the authoritative DNS server in Kubernetes.

CoreDNS is a popular choice, as it supports building a DNS zone from the Kubernetes service registry. It also offers extra features such as caching, forwarding, and logging.

An example of a configuration file of CoreDNS:

```
.:53 {
    errors
    health {
        lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        fallthrough in-addr.arpa ip6.arpa
        ttl 30
    }
    forward . /etc/resolv.conf
    cache 30
}
```

Important to note are the kubernetes zone and the forward statement.

For more information on changing the kube-dns configuration, see [this documentation page](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/).

### Nodelocaldns

DNS queries are a common and essential part of network communication. They need to be processed quickly to avoid performance issues. Slow DNS queries can cause problems that are difficult to diagnose and troubleshoot.

To improve the performance of DNS queries in a Kubernetes cluster, a cache layer can be added on each node using the [nodelocaldns](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/) component. This component caches the responses to DNS queries.

If no response is found in the cache, it forwards the query to the authoritative nameserver (CoreDNS). The response is stored in the local cache so that it can be used to serve future queries from the same or other pods on the same node.

This reduces the amount of network traffic between pods and the DNS server. This means lower latencies and faster DNS query performance. The function of nodelocaldns is often fulfilled by CoreDNS as well.

## A note on the TTL (time-to-live) of records in Kubernetes

In Kubernetes, the time-to-live (TTL) of DNS records is determined by the DNS server implementation that is being used.

By default, CoreDNS sets the TTL of DNS records to 30 seconds. This means that when a DNS query is resolved, the response will be cached for up to 30 seconds before it is considered stale. The TTL of DNS records can be modified using the `ttl` option in the CoreDNS configuration file.

The TTL of DNS records is an important parameter because it determines how long a DNS response will be considered valid before a new query must be made.

A shorter TTL can improve the accuracy of DNS responses, but it can also increase the load on the DNS server. A longer TTL can reduce the load on the DNS server. However, it can also cause DNS responses to be outdated or inaccurate if the underlying DNS records are updated.

Therefore, the appropriate TTL should be chosen based on the specific requirements of the cluster.

## Bonus: SRV records

So far we've only talked about resolving IP addresses using A-records. Kubernetes also uses SRV (service) records to resolve the port numbers of named services. This allows clients to discover the port numbers of services by querying the DNS server for the appropriate SRV record.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  ports:
    - port: 80
      name: http
```

In this service, the container port 80 is exposed and is given the name "http". Because the port is named, Kubernetes will generate an SRV record with the following name: `_<port>._<proto>.<service>.<ns>.svc.<zone>`.

In this case, the SRV record will be named `_http._tcp.nginx.default.svc.cluster.local`. A DNS query for this record would return the port number and IP address of the named service:

```
dig +short SRV _http._tcp.nginx.default.svc.cluster.local
0 100 80 10-129-1-26.nginx.default.svc.cluster.local.
```

Some services, such as Kerberos, use SRV records for the discovery of the KDC (Key Distribution Center) servers.