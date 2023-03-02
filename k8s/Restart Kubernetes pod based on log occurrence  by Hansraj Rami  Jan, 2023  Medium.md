In this article, I will give a practical example of how we can restart a Kubernetes pod based on a particular occurrence in the logs of that pod.

This is the magic of [Kubernetes liveness probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) and [Linux grep](https://man7.org/linux/man-pages/man1/grep.1.html) commands.

![](https://miro.medium.com/max/330/1*rS3MoTFvh3LGsGj0KZrpNQ.png)

When you run workloads in production at a certain time you might get a requirement to restart services automatically in case of failure. In my case, I’m running an event listener to listen to Smart contract events from the Blockchain. Due to some reason event listener was losing connection randomly to the Blockchain node and re-connecting after some time. But we missed events that happened in this particular timeframe. So as a solution, we planned to restart a pod if we get a particular log message.

For this sample, I’ve created a sample GoLang program for the demonstration.

```
  LOG_FILE := "/tmp/event-listener.log"  logFile, err := os.OpenFile(LOG_FILE, os.O_APPEND|os.O_RDWR|os.O_CREATE, 0644) if err != nil {  log.Panic(err) } defer logFile.Close()  log.SetOutput(io.MultiWriter(logFile, os.Stdout)) var count uint64 = 0 for count < 10 {  count += 1  log.Println("Counting", count)  time.Sleep(time.Second) } log.Println("Disconnecting...") for {  count += 1  log.Println("Still counting", count)  time.Sleep(time.Second) }
```

Is this GoLang program I’m counting from 1 to 10 and printing **Disconnecting…** after that. Based on the occurrence of **Disconnecting…** it will restart a pod. If you want to deploy this you can refer to this Docker image [hansrajrami/log-occur-restart](https://hub.docker.com/r/hansrajrami/log-occur-restart).

Here I’m attaching Kubernetes YAML manifests for your reference. I will also explain the same below.

```
apiVersion: apps/v1kind: Deploymentmetadata:  name: log-occur-restartspec:  progressDeadlineSeconds: 600  selector:    matchLabels:      app: log-occur-restart  strategy:    rollingUpdate:      maxSurge: 25%      maxUnavailable: 25%    type: RollingUpdate  template:    metadata:      labels:        app: log-occur-restart        selector: log-occur-restart    spec:      containers:      - name: log-occur-restart        image: hansrajrami/log-occur-restart:latest        imagePullPolicy: IfNotPresent        resources:          limits:            cpu: 100m            memory: 100Mi        livenessProbe:          initialDelaySeconds: 5          periodSeconds: 3          exec:            command:              - "/bin/sh"              - "-c"              - "! grep -q 'Disconnecting...' /tmp/event-listener.log"      terminationGracePeriodSeconds: 30
```

If you check the [**livenessProbe**](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) block of the above Kubernetes manifest it will execute the below command after every 3 seconds.

```
! grep -q 'Disconnecting...' /tmp/event-listener.log
```

**grep** is a utility in Linux to search a file for a particular pattern of characters and display all lines that contain that pattern.

**\-q** flag I’ve added for silent output to not print anything even in case of occurrence.

! mark I’ve added to invert the exit code of the command.

When the liveness probe will execute the above command it will return either 0 or 1 as the exit code. If **grep** will find **Disconnecting…** in the log message it will return **0** as an exit code but **! an exclamation mark** will invert it and return the exit code as **1**. This will make the liveness probe believe that the pod is not healthy and it will restart the pod. If **grep** won’t find **Disconnecting…** in the log message it will return **1** as an exit code but **! an exclamation mark** will invert it and return the exit code as **0**. This will make liveness probe believe that everything is fine and the pod is healthy.

Alright!! We did it.

If you want to refer to the code and configuration of this demonstration please check this GitHub repository [https://github.com/hansrajrami/log-occur-restart](https://github.com/hansrajrami/log-occur-restart).

If you were with me till here you must’ve liked it. Please give this article some claps.