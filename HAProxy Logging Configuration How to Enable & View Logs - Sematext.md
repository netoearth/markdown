Table of Contents

-   [What Are HAProxy Logs?](https://sematext.com/blog/haproxy-logs/#what-are-haproxy-logs)
-   [Configuration: How to Enable HAProxy Logging](https://sematext.com/blog/haproxy-logs/#configuration-how-to-enable-haproxy-logging)
-   [Default Logging](https://sematext.com/blog/haproxy-logs/#default-logging)
-   [Backend Logging](https://sematext.com/blog/haproxy-logs/#backend-logging)
-   [Frontend Logging](https://sematext.com/blog/haproxy-logs/#frontend-logging)
-   [Listen Section Logging](https://sematext.com/blog/haproxy-logs/#listen-section-logging)
-   [Logging to a File](https://sematext.com/blog/haproxy-logs/#logging-to-a-file)
-   [Logging in Containerized Environments](https://sematext.com/blog/haproxy-logs/#logging-in-containerized-environments)
-   [HAProxy Log Location](https://sematext.com/blog/haproxy-logs/#haproxy-log-location)
-   [HAProxy Log Levels](https://sematext.com/blog/haproxy-logs/#haproxy-log-levels)
-   [HAProxy Log Format](https://sematext.com/blog/haproxy-logs/#haproxy-log-format)
-   [Default Log Format](https://sematext.com/blog/haproxy-logs/#default-log-format)
-   [TCP Format](https://sematext.com/blog/haproxy-logs/#tcp-format)
-   [HTTP Format](https://sematext.com/blog/haproxy-logs/#http-format)
-   [Custom Log Formatting](https://sematext.com/blog/haproxy-logs/#custom-log-formatting)
-   [HAProxy Log Rotation](https://sematext.com/blog/haproxy-logs/#haproxy-log-rotation)
-   [How to Read HAProxy Logs](https://sematext.com/blog/haproxy-logs/#how-to-read-haproxy-logs)
-   [HAProxy Stats Page](https://sematext.com/blog/haproxy-logs/#haproxy-stats-page)
-   [Log Monitoring and Analysis with Sematext](https://sematext.com/blog/haproxy-logs/#log-monitoring-and-analysis-with-sematext)
-   [Conclusion](https://sematext.com/blog/haproxy-logs/#conclusion)

HAProxy is generally the frontend layer of your application, which means it plays a critical role since all traffic first lands on this layer. Because of this, you need to make sure everything is working at this layer all the time, as any issue can directly impact your business. Therefore, having visibility on this layer is crucial.

Visibility can come from two aspects: the metrics HAProxy emits and the logs it generates while handling requests.

In this article, I’ll focus mainly on the logging aspect of HAProxy—how to enable different types of logs and how to work with them to make more informed decisions.

## What Are HAProxy Logs?

Haproxy logs are the record of events that indicate what’s happening with HAProxy while starting up, serving a request, and during an error. HAProxy uses [log files](https://sematext.com/glossary/log-file/) to record events including requests received by HAProxy, the response to those requests, and their status code. It also logs any errors that occur.

HAProxy emits log messages for processing by a syslog server. These syslog messages are compatible with the basic tools for reading syslog messages like rsyslog. HAProxy syslog messages are also compatible with the systemd-journald service.

You can enable HAProxy log messages by specifying the syslog server, then configuring it for different proxies like frontend and backend. The first step is to enable logging in a global configuration:

```
global
        log 127.0.0.1:514 local0
```

HAProxy natively supports syslog logging, which you can enable as shown in the above examples. Syslog facilities and severity levels are also at your disposal, as well as the ability to forward the logs to journald, rsyslog, or any supported syslog server:

```
log 127.0.0.1:514 facility serverity_level
```

With the log directive, you can specify the syslog server where you want to send the HAProxy logs. Local0 is one facility used to send logs. Simply add the logging level in the log directive to control what information to log:

```
log 127.0.0.1:514 local0 info
```

This will start logging the messages that have the log level of “info” or above. Then, add the log in the default directive, which will enable the same logging in all three frontend, backend, and listen parts.

```
defaults
        log global
```

The listen section actually combines the information in the backend section and the frontend section. So instead of defining the backend and frontend separately and mapping them, you can define them in one listen section, and the mapping will be done automatically.

HAProxy has four major sections: global, default, backend, and frontend.

In the global config level, you can define values that work process-wide for security and performance tuning. In the default configs, you have to keep the best configuration, which is used across all/multiple sections depending on your use case. After that, you can override these values in the frontend or backend as required if you want to add another option or override any existing one. You can also have multiple log directives to forward the logs to different files.

### Default Logging

In the default section of the configuration, you can mention the default configs that you want to use. This will be used by default in all the sections if not otherwise specified. You can configure things like where to log and the log format, as seen below:

```
defaults
        log global
        option httplog
        Timeout client 30s
```

These values will be used automatically in the section’s frontend and backend unless you override these values.

### Backend Logging

HAProxy backend servers are the set of servers among which a request is actually load-balanced. Create a logging line separately for these backends using the following configuration:

```
backend webservers
        balance roundrobin
        server server1
        log 127.0.0.1:514 local0 info
```

With the above, you’ll be able to easily see the logs of different backends at different syslog servers.

### Frontend Logging

Similar to the backend, frontend servers can also have their own logging configuration. The frontend in HAProxy is where traffic actually lands. Based on the frontend configuration, backend servers are chosen and the traffic is forwarded to these. As to logs, these are defined separately as seen in the configuration below:

```
frontend webui
        bind 0.0.0.0:80
        use_backend webservers
        log 127.0.0.1:514 local0 error
```

### Listen Section Logging

The listen section lets you define all the parts of the proxy together with the frontend and backend. This makes it easy to understand, as you only need to check the listen section for each proxy as described below:

```
listen fe_main
   log 127.0.0.1:514 local0 error
   # frontend configuration settings
   mode http
   bind :80
   http-request redirect scheme https unless { ssl_fc }

   # backend configuration settings
   balance roundrobin
   option httpchk
   server s1 192.168.1.25:80 check
   server s2 192.168.1.26:8080 check
```

### Logging to a File

Most tools support logging to a file for better accessibility and easier setup. HAProxy, however, doesn’t natively support this. The reason for this design decision is that HAProxy is supposed to proxy requests, not write to the file. HAProxy instead offloads this responsibility to existing solutions like syslog.

If you want to write to a file, you need to configure the syslog server to do so. Below is the config for writing syslog messages to files using the rsyslog service in Linux:

```
local0.* /var/log/haproxy.log
local0.notice /var/log/hap_notifications.log
```

### Logging in Containerized Environments

By default, [containers](https://sematext.com/glossary/containers/) tend to log to stdout and stderr. HAProxy can easily log to stdout with the configuration below, or you can send the log to syslog server:

```
global
    log stdout format raw local0
```

The above config will start logging to stdout. To send the log to the syslog server from the HAProxy container, you can use the following configurations:

```
global
    log rsyslog_location:514 local0
```

Read more about this in the [documentation](https://www.haproxy.com/documentation/hapee/latest/observability/logging/log-with-docker/) on Docker logging with HAProxy.

## HAProxy Log Location

HAProxy logs location depends on the operating system you are using:

-   On Ubuntu or Debian-based distributions in Linux, the log location is `/var/log/haproxy.log`. This is automatically configured by the scripts provided in the installation package.
-   In CentOS or other Red Hat distributions, HAProxy does not output logs to a file by default, meaning you’ll need to configure it separately. [This documentation](https://www.digitalocean.com/community/tutorials/how-to-configure-haproxy-logging-with-rsyslog-on-centos-8-quickstart) tells you how to configure logs to a file on CentOS.

## HAProxy Log Levels

Similar to other logging agents and tools, HAProxy divides logs into different log levels to make sure they’re properly segregated based on severity and verbosity:

-   **E****merg:** Emergency alerts that can come from underlying operating systems, including errors such as OOM and file descriptor exhaustion
-   **Alert:** Cases that happen rarely and cause unexpected exceptions
-   **Crit:** Not used by HAProxy
-   **Err:** HAProxy configuration errors like unable to parse the configuration file, maps file, etc.
-   **Warning:** Important messages but not critical; errors like failed modification to a request or failed DNS nameserver connection
-   **Notice:** Changes to HAProxy states, e.g., HAProxy emitting a restart, stop, or start event; also for logging module loading and health-check logging
-   **Info:** TCP and HTTP request details and error details
-   **Debug:** Not used by default with HAProxy, but can be used with custom Lua scripts to log extra details

All these log levels are easily configured in the log directives discussed above. It’s important to segregate different HAProxy log levels to facilitate more visibility with less hassle.

## HAProxy Log Format

HAProxy can work with TCP and HTTP. It comes with a default log format, or you can customize the log format and use it with whatever field you want.

### Default Log Format

The default log format of HAProxy is TCP mode. You can access the information present in TCP mode by logging in as shown in the TCP section below. Default log formats have lots of information present, but there are missing elements like SSL encryption information, headers, and payload. Due to this, users end up creating custom log formats most of the time.

### TCP Format

When it’s working with TCP, you can enable it by adding the following configuration:

```
mode tcp
option tcplog
```

TCP mode will form a full duplex connection, and you will not be able to look at the application-level details. With TCP, you have to configure the logging mode to TCP so that the log format will comply with the field present in the logging options like byte count, timers, etc.

Fields present in the TCP mode are as below:

![](https://sematext.com/wp-content/uploads/2022/10/haproxy-logging-3.jpg)

The above details are from the [HAProxy official documentation](https://www.haproxy.com/documentation/hapee/1-8r1/onepage/#8.2.2). You can also define the log format in HAProxy using the log-format directive:

```
log-format "%ci:%cp [%t] %ft %b/%s %Tw/%Tc/%Tt %B %ts
%ac/%fc/%bc/%sc/%rc %sq/%bq"
```

### HTTP Format

When using the HTTP protocol, you have to use the HTTP mode and option as httplog:

```
mode http
option httplog
```

In HTTP mode, HAProxy can log more than the TCP details, including log details such as HTTP request URL, status code, bytes sent, request and response header, etc. A full list of details is mentioned in the following:

![](https://sematext.com/wp-content/uploads/2022/10/haproxy-logging-2.jpg)

Similar to the TCP logging format, the log-format directive lets you log HTTP requests:

```
log-format "%ci:%cp [%tr] %ft %b/%s %TR/%Tw/%Tc/%Tr/%Ta %ST %B
%CC %CS %tsc %ac/%fc/%bc/%sc/%rc %sq/%bq %hr %hs %{+Q}r"
```

The log-format directive can go in the default, frontend, or listen sections. If you put it in the default section, it will consider default options for all the sections.

### Custom Log Formatting

The HAProxy logging format has variables to define the logging format properly; below are the rules that all variables have to follow:

-   Every variable is preceded by a percentage character “%”.
-   Variables can take flags with the help of braces {}.
-   Multiple flags can be separated with commas.
-   A + or – sign adds or removes flags.
-   Spaces that are not between variables have to be escaped with a backslash.

There are three flags:

-   Q: To quote a string
-   X: For hexadecimal representation
-   E: For escapes characters

You can add configuration for headers, SS, etc with the following configurations:

-   Connection handshake time (%Th)
-   SSL ciphers and versions (%sslc / %sslv)
-   Request headers (%hr or %hrl for CLF formatting)
-   Response headers (%hs or %hsl for CLF formatting)
-   The full HTTP request (%r)

You can look more into these variables and how to use them in [HAProxy’s custom logging format documentation](https://www.haproxy.com/blog/haproxy-log-customization/#:~:text%3DHAProxy%2520comes%2520with%2520a%2520few,as%2520a%2520Layer%25204%2520proxy.).

## HAProxy Log Rotation

Like any system, HAProxy can generate large volumes of log data. To limit the amount of disk used by logs, you need to rotate them. Log rotation means keeping the older log data in another format, compressing it, and archiving it at another location.

For log rotation, make use of the logrotate utility in Linux by creating a haproxy.logrotate file with a [basic configuration in Linux](https://linux.die.net/man/8/logrotate):

```
/var/log/haproxy {
    missingok
    notifempty
    sharedscripts
    postrotate
        /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
        /bin/kill -HUP `cat /var/run/rsyslogd.pid 2> /dev/null` 2> /dev/null || true
        /bin/kill -HUP `cat /var/run/syslog-ng.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

## How to Read HAProxy Logs

HAProxy has lots of fields and provides great in-depth details about logs, but it can be hard to understand these variables and fields. Fortunately, HAProxy ships with HALog, a very powerful, production-ready [log analysis tool](https://sematext.com/blog/log-analysis-tools/). Unfortunately, HALog doesn’t support custom-log format parsing at the moment. But you can use tools like [Sematext Logs](https://sematext.com/blog/log-analysis-tools/#1-sematext-logs) to solve this problem.

This means it can be deployed in production and is really fast, parsing both TCP and HTTP logs, up to 1-2 GB of logs per second. You can even pass different flags with HALog to make it filter out the information you need.

HALog presents you with metrics that give you critical insight into your servers. It can parse logs to show how many requests failed per server, the total number of requests, the number of occurrences of different status codes, and much more.

These commands will retrieve information including status codes, total requests, and response times:

```
halog -srv -H < hap.log | column -t
```

```
195000 lines in, 10 lines out, 0 parsing errors
#srv_name       1xx  2xx    3xx  4xx  5xx  other  tot_req  req_ok  pct_ok  avg_ct  avg_rt
api/users       0    17969  163  851  0    1      13984    13983   100.0   0       60
api/profile     0    12976  149  854  5    0      13984    13979   100.0   1       150
httpd/<NOSRV>   0    0      8    0    0    0      8        0       0.0     0       0
httpd/users     84   534    0    6    2    0      626      626     100.0   0       342
httpd/profile   72   3096   0    9    10   0      3187     3183    99.9    1       1509
```

### HAProxy Stats Page

HAProxy has a stats page to see all the above details in your web browser; to enable, use the following configuration:

```
frontend stats
   bind *:8404
   stats enable
   stats uri /stats
   stats refresh 10s
   stats admin if LOCALHOST
```

This will display the stats page on port 8404:

![](https://sematext.com/wp-content/uploads/2022/10/haproxy-logging-1.jpg)

_Stats page for HAProxy server. (Source: [HAProxy](https://www.haproxy.com/blog/exploring-the-haproxy-stats-page/))_

## Log Monitoring and Analysis with Sematext

Generally, companies run a huge fleet of HAProxy servers to handle traffic; I myself have run some 200 HAProxy servers. These servers generate a huge amount of logs, and looking at them can be a challenge. HALog can help, but it’s still difficult to log into such a huge number of servers, run HALog on each of them, and then accumulate the results.

What you really need is a [centralized logging solution](https://sematext.com/blog/log-aggregation/) such as [Sematext Logs](https://sematext.com/logsene/), where you can ship all the logs and build analytics on top of it.

Sematext is a [log management and monitoring tool](https://sematext.com/blog/best-log-management-tools/) with great support for HAProxy logs. It is easy to install and configure. In no time, you’ll be able to see logs in your dashboard and leverage pre-built dashboards to filter out key information.

Sematext comes with many other features that give you confidence in running your HAProxy clusters in production. You can add alerting on the dashboard and forward alerts to different channels such as email, Slack, PagerDuty, and many more.

Sematext Logs is part of the [Sematext Cloud](https://sematext.com/cloud/), a full-stack observability platform with support for [monitoring](https://sematext.com/integrations/haproxy/) [HAProxy clusters](https://sematext.com/integrations/haproxy/) as well as the majority of tools present in your infrastructure. This means that it also gathers [HAProxy metrics](https://sematext.com/blog/haproxy-monitoring/). Correlating metrics and logs from your HAProxy servers is greatly valuable, reducing the time needed to fix issues in production. Plus, anomaly detection lets you identify issues before they happen based on past metrics.

Watch the video below to learn more about what Sematext Logs can do for you.

<iframe src="https://www.youtube-nocookie.com/embed/TSlp3ru1BNA?ab_channel=Sematext"></iframe>

## Conclusion

HAProxy is a critical component of your infrastructure, and you need visibility on these servers to monitor for errors and performance. You need a centralized location to see logs and receive alerts if you’re running a huge fleet of HAProxy servers.

Sematext provides an effective dashboard to catch issues and alert you in advance via its integration with logging and monitoring solutions around HAProxy.

[Sign up](https://sematext.com/) for a free trial today.

**Author Bio**

![](https://sematext.com/wp-content/uploads/2021/11/gaurav-150x150.jpeg.webp)

**Gaurav Yadav**  
Gaurav has been involved with systems and infrastructure for almost 6 years now. He has expertise in designing underlying infrastructure and observability for large-scale software. He has worked on Docker, Kubernetes, Prometheus, Mesos, Marathon, Redis, Chef, and many more infrastructure tools. He is currently working on Kubernetes operators for running and monitoring stateful services on Kubernetes. He also likes to write about and guide people in DevOps and SRE space through his initiatives Learnsteps and Letusdevops.