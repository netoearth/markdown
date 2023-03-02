Table of Contents

-   [What Are Tomcat Logs?](https://sematext.com/blog/tomcat-logs/#what-are-tomcat-logs)
-   [How to Enable Tomcat Logging](https://sematext.com/blog/tomcat-logs/#how-to-enable-tomcat-logging)
-   [Tomcat Log Levels](https://sematext.com/blog/tomcat-logs/#tomcat-log-levels)
-   [Tomcat Logs Location](https://sematext.com/blog/tomcat-logs/#tomcat-logs-location)
-   [Tomcat Log File Types](https://sematext.com/blog/tomcat-logs/#tomcat-log-file-types)
-   [Catalina Logs](https://sematext.com/blog/tomcat-logs/#catalina-logs)
-   [Localhost Log](https://sematext.com/blog/tomcat-logs/#localhost-log)
-   [Localhost Access Log](https://sematext.com/blog/tomcat-logs/#localhost-access-log)
-   [Access Log](https://sematext.com/blog/tomcat-logs/#access-log)
-   [Manager Log](https://sematext.com/blog/tomcat-logs/#manager-log)
-   [Log Rotation](https://sematext.com/blog/tomcat-logs/#log-rotation)
-   [How to View Tomcat Logs](https://sematext.com/blog/tomcat-logs/#how-to-view-tomcat-logs)
-   [How to Disable Tomcat Logging](https://sematext.com/blog/tomcat-logs/#how-to-disable-tomcat-logging)
-   [Apache Tomcat Log Analysis and Monitoring with Sematext](https://sematext.com/blog/tomcat-logs/#apache-tomcat-log-analysis-and-monitoring-with-sematext)
-   [Conclusion](https://sematext.com/blog/tomcat-logs/#conclusion)

Apache Tomcat is the Java web server that implements many Java features like web site APIs, Java server pages, Java Servlets, etc. It’s an open-source software widely used in the industry. Tomcat sits on top of your application and is the entry point for reaching your application code. It is crucial to monitor its performance and make sure everything works, get notified when unexpected errors occur, and take action in real-time. Like other web servers, it also produces a lot of logs that are instrumental in troubleshooting.

The Apache Tomcat logs are information about your resources—what they are, who is accessing which ones, and at what time. In this article, I’ll show you **how to look at Tomcat logs, their different types, log locations, and log rotation**.

## What Are Tomcat Logs?

Tomcat logs are the files that keeps information related to web server issues and server tasks in proper files in given locations. It’s important to understand logging on the Tomcat server because this tells you if there’s any issue with the server and if so, whether it’s functional or performance related. There can be multiple types of logs segregated into different file locations. Knowing these locations will help you debug issues faster.

Tomcat global logs, called Catalina logs, are some of the most important log types present. There are others too, like application logs and localhost logs. I’ll be discussing all of these and how they can help you. But before that, let’s look at the configuration needed to enable logging in Tomcat.

To collect Tomcat logs, you first need to enable logging. Logging in Tomcat is handled by the Java Utility Logging Implementation, also known as JULI. JULI is enabled by default, and you can perform this configuration using the logging configuration file option -Djava.util.logging.config.file=”logging.properties”:

-   Globally: `${catalina.base}/conf/logging.properties.` This file is specified by the `java.util.logging.config.file` system property. If you don’t configure it, the default file will be `${java.home}/lib/logging.properties` in the JRE.
-   In the web application: `Use WEB-INF/classes/logging.properties.`

If you don’t want to use JULI, there are other logging frameworks like [log4j](https://sematext.com/blog/log4j-tutorial/), [log4j2](https://sematext.com/blog/log4j2-tutorial/), or [Logback](https://sematext.com/blog/logback-tutorial/). Let’s take log4j as example.

To configure log4j for logging, just create a `log4j.properties` file and use log4j.jar. You can put the `log4j.properties` file into the `$CATALINA_BASE/lib` folder.

There are multiple options provided to configure the log4j logs. A few of the important ones are `log4j.appender.CATALINA.File`, which configures the location of the file; `log4j.appender.CATALINA.Encoding` can configure the encoding; and the `UTF-8.log4j.appender.LOCALHOST.DatePattern` option, which helps in configuring the log rotation based on date pattern.

For configuring Manager, Localhost, Host-Manager, and Catalina logs, the configuration parameters start with prefixes: `log4j.appender.MANAGER` for Manager, `log4j.appender.HOST-MANAGER` for Host Manager, `log4j.appender.LOCALHOST` for localhost, and `log4j.appender.CATALINA` for Catalina. You can see the exact instructions in the [Tomcat documentation](https://tomcat.apache.org/tomcat-8.0-doc/logging.html).

## Tomcat Log Levels

As with any other logging framework, JULI also has [log levels](https://sematext.com/blog/logging-levels/) that define the severity of the log and the information it contains. JULI log levels consist of:

1.  **SEVERE**: Information pertaining to a serious problem
2.  **WARNING**: Could be an issue in the future
3.  **INFO**: Informational messages
4.  **CONFIG**: Configuration information (listed when log level is set to config)
5.  **FINE**: Trace logs
6.  **FINER**: More-detailed trace logs
7.  **FINEST**: Highest-detailed trace logs

These log levels are sorted in the order of the details that they feature. So, **SEVERE** has only very important information, while FINEST presents you with the most information. Please note that having a lot of information in your logging can have a performance impact on your disk’s I/O and increase your system’s load average.

Log levels can be set for different handlers from the `logging.properties` file found in `$CATALINA_BASE/conf`:

## For example, set the com.xyz.foo logger to only log SEVERE # messages:

```
org.apache.catalina.startup.ContextConfig.level = SEVERE 
org.apache.catalina.startup.HostConfig.level = SEVERE 
org.apache.catalina.session.ManagerBase.level = SEVERE
```

## Tomcat Logs Location

The location of your Tomcat logs can differ depending on your operating system or containerzied platform. Below are a few of those default locations:

-   **Linux**: The log will be present by default at `/var/log/tomcat/*.log`. In some cases, you will also find it at `/opt/tomcat/logs.`
-   Windows: In Windows, the logs will be present at the location where your software is, something like this: `C:\program files\apache software foundation\apache-tomcat{ver}\logs\catalina.out`
-   [Docker](https://sematext.com/glossary/docker/): By default, the logs will be printed to STDOUT and STDERR so that the logging agents on the nodes can get these logs and push them to a centralized place.
-   [Kubernetes](https://sematext.com/glossary/kubernetes/): You can configure your logging agent to read the logs from the standard output and standard error from the [containers](https://sematext.com/glossary/containers/).

You can easily change where your Tomcat logs are stored using the `CATALINA_OUT` variable.

## Tomcat Log File Types

There are multiple [log files](https://sematext.com/glossary/log-file/) that Tomcat produces for its different components. The most important are all the Catalina logs since you can easily find most information in these files.

### Catalina Logs

Tomcat Catalina produces two types of Catalina log files: **catalina.log** and **catalina.out**.

#### Catalina.log

This is the global log file used to save information like server admin operations, for example, server stop, start, restart, deployment of a new application, and failure of a subsystem. If you want to make changes to this log format, do so in the `java.util.logging.SimpleFormatter.format` variable. You can configure the location of the file in the `$CATALINA_OUT` parameter, or it will be present by default at `CATALINA_HOME/logs`.

#### Catalina.out

The servlet and error logs will be written in the catalina.out log file. This file can also contain thread dumps and uncaught exceptions.

You can make changes to the format of catalina.out in the _java.util.logging.SimpleFormatter.format_ variable. The location of this file is defined in the `$CATALINA_OUT` parameter or is present by default at `CATALINA_HOME/logs`.

### Localhost Log

These are the logs that track the activity log of your web application, e.g., the transactions between the application server and clients. These log files are present in the Tomcat home directory inside the logs folder.

### Localhost Access Log

There can be cases where the application server has to make a call to another service, i.e., outbound HTTP requests. The localhost access log keeps track of all these requests. Localhost access logs are also found in the same location as the localhost logs inside the log directory of the Tomcat home directory.

### Access Log

Tomcat access logs can give you information about who has access to your application, what resources were accessed, the IP, queries, date, etc. These are some of the most important logs for traffic analysis and can be configured via the `server.xml` file inside the conf directory in the Tomcat home directory.

### Manager Log

These logs keep information on all the activities performed by the Tomcat manager, including lifecycle management, application deployment, etc. These logs can be configured in the `logging.properties` configuration file present at `TOMCAT_HOME/conf`.

## Log Rotation

There can be lots of logs produced by Tomcat. You need a way to archive the older ones so that your disk doesn’ get filled up. To achieve this, you have to hand over the Tomcat log rotation to the default log rotate engine in the operating system. For example, in the case of Linux, you can do this easily using logrotate. Simply create a file using the command:

```
touch /etc/logrotate.d/tomcat
```

And then put the following into the file; this can be tweaked per your needs for log rotation:

```
/{PATH_TO_CATALINA_FILE}/catalina*.* {
    copytruncate
    daily
    rotate 7
    compress
    missingok
    size 100M
}
```

This configuration will start the log rotation for you. You can also write similar files if you want to implement log rotation separately for different types of Tomcat logs.

## How to View Tomcat Logs

There is no utility to look at Tomcat logs, so you have to rely on your operating system tools. For example, in the case of Linux, you can use less or tail commands to look at your logs:

```
less filename.log
tail -f filename.log
```

Given that you generally have a lot of machines running as Tomcat servers, it’s a very tricky task to run these commands on all those machines and get the logs. You need to have a place where you can push these logs and query on top of it. This is centralized logging, which you need when you have a lot of machines and it is humanly impossible to look at all the logs.

## How to Disable Tomcat Logging

Logging can be costly for disk IOPS if we don’t do it properly, so you may want to disable the logging of Tomcat in some cases. To do this, you’ll need to make changes in the `logging.properties` file. To do that, comment the block below in the `logging.properties` file:

```
handlers = 1catalina.org.apache.juli.AsyncFileHandler,
2localhost.org.apache.juli.AsyncFileHandler…
```

To disable the access log, you have to remove the following block from the `server.xml` file:

```
<Valve className="org.apache.catalina.valves.AccessLogValve"
 directory="logs" prefix="localhost_access_log" suffix=".txt"
 pattern="%h %l %u %t "%r" %s %b" />
```

## Apache Tomcat Log Analysis and Monitoring with Sematext

[Sematext Logs](https://sematext.com/logsene/) is a log management solution that can get all your distributed logs for Apache Tomcat servers in one place. Sematext collects the logs from your Tomcat servers by installing a small agent on the host machine. It then starts collecting the data and sends it to Sematext collector servers, where the data will be available for you to view, analyze, or create dashboards from. Sematext has advanced features like auto-discovery, out-of-the-box dashboarding, alerting, anomaly detection, and communication support for different channels.

In addition, Sematext Logs is part of the [Sematext Cloud](https://sematext.com/cloud/) observability solution, which includes a [monitoring solution](https://sematext.com/spm/) that can capture [Tomcat metrics](https://sematext.com/integrations/apache-tomcat/). When correlated with logs, these metrics present great value, enabling you to debug issues faster than before. For example, if you see any 5xx or 4xx issues, and there are logs for the same, it becomes easy to see why these errors are occurring.

## Conclusion

When you’re running any application with Tomcat, it’s good to have visibility into what Tomcat and its different components are actually doing. To do this, you need access to logs and a way to look at them. Logs present you with information like if the servers are restarted, if there was any error, etc. You also want to have visibility on the type of traffic coming in and throughput.

Sematext provides a platform to achieve centralized logging and analysis to easily keep an eye on your application. With its advanced features like dashboarding, alerting, and anomaly detection, it becomes simple to monitor your Tomcat infrastructure.

Watch the video below to learn more about Sematext Logs or start a [14-day free trial](https://sematext.com/pricing/) to see how it can help your log management for Apache Tomcat web servers.

<iframe src="https://www.youtube-nocookie.com/embed/TSlp3ru1BNA"></iframe>

**Author Bio**

![](https://sematext.com/wp-content/uploads/2021/11/gaurav-150x150.jpeg.webp)

**Gaurav Yadav**  
Gaurav has been involved with systems and infrastructure for almost 6 years now. He has expertise in designing underlying infrastructure and observability for large-scale software. He has worked on Docker, Kubernetes, Prometheus, Mesos, Marathon, Redis, Chef, and many more infrastructure tools. He is currently working on Kubernetes operators for running and monitoring stateful services on Kubernetes. He also likes to write about and guide people in DevOps and SRE space through his initiatives Learnsteps and Letusdevops.