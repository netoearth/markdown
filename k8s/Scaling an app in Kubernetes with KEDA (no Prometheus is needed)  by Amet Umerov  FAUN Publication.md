_I can recommend you also read_ [_an article_](https://itnext.io/tutorial-auto-scale-your-kubernetes-apps-with-prometheus-and-keda-c6ea460e4642) _about KEDA. In this article author tell us about KEDA, its integration with Prometheus, and how can we scale our app based on Redis metrics in Kubernetes._

_In this article, I’ll try to be simple. Together we get to know KEDA by example in just a few minutes. No Prometheus is needed._

## Short intro

Everyone knows that Kubernetes works great with stateless applications. Some serious people even run databases in k8s, and the bravest — in production. But most often in production, we use services like RDS or self-hosted databases deployed outside the Kubernetes cluster.

For those who want to scale their applications inside Kubernetes, but at the same time use some external metrics (for example, the number of messages in AWS SQS or Kafka, connections in PostgreSQL/MySQL/etc.), it is necessary to use [external metrics](https://cloud.google.com/kubernetes-engine/docs/concepts/custom-and-external-metrics#external_metrics). Most often, we can do that by importing metrics into Prometheus, using various adapters, and then scale the application based on these metrics.

So, KEDA allows us to simplify this process a bit. We don’t need to do all these manipulations with Prometheus metrics. All we need is to deploy a few CRDs, a couple of YAML files to describe scaling and permissions. Let’s go ahead.

![](https://miro.medium.com/max/1400/1*GnypSGx6HFWA2T7fhCqFCg.png)

How KEDA works: [https://www.codit.eu/blog/exploring-kubernetes-based-event-driven-autoscaling-keda/](https://www.codit.eu/blog/exploring-kubernetes-based-event-driven-autoscaling-keda/)

## KEDA installation

_I do my actions on macOS with Docker for Mac. But you can use any Kubernetes._

The [installation](https://keda.sh/docs/2.3/deploy/#install) of KEDA is very simple:

We deployed a chart, let’s check that `keda-operator` and `metrics-apiserver` pods are alive:

That’s all, we installed KEDA-operator. We can continue.

## Let’s deploy “Hello World”

In Kubernetes world “Hello World” is [nginx deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment), so let’s deploy it:

Checking the pods status:

Alright, 3 pods in the deployment, that is what we expected to see.

Let’s do some autoscaling by RabbitMQ metrics (by a number of messages in a queue).

_For now, KEDA supports_ [_36 different scalers_](https://keda.sh/docs/2.3/scalers/)_, my example is based on RabbitMQ._

## A bit of RabbitMQ preparation

As I mentioned earlier, I do my tests on the local machine with Docker for Mac. So I’m going to simulate a real situation, when the RabbitMQ runs outside the k8s cluster, in another network. So I’m going to install rabbit from the `brew`. But before doing that, let’s note my local IP address, we will use it in the future:

Ok, installing and starting RabbitMQ:

Validating that everything is fine:

Good. One moment, by default, the user `guest` can access RabbitMQ only by IP `127.0.0.1`, so we need to add a new user (let it be `demo`/`demo`). Also, let’s create a new queue for tests `demo_queue`:

After that we can connect to our rabbit by “external” IP:

## A bit of YAML

RabbitMQ is ready, now we need to connect it with KEDA, so we need to create three objects:

-   `Secret` — stores a string for connection to RabbitMQ (`amqp://demo:demo@192.168.2.101:5672/`)
-   `TriggerAuthentication` — the object for authentication, uses data from secret above
-   `ScaledObject` — the object needed for scaling, where we can set some [scaling parameters](https://keda.sh/docs/2.3/concepts/scaling-deployments/#scaledobject-spec)

Deploying:

Checking that everything is working:

## Testing

Let’s send 5 messages to our RabbitMQ queue:

Waiting for 10 seconds while KEDA goes for the metric, and check our pods (we expect to see 2 pods, as we have 5 messages in queue):

Let’s add 7 more messages to the queue:

12 messages in the queue, we expect to see 4 pods, checking:

Good. Now purging the queue, we expect to see the minimum value (only one pod):

Everything is good, it scales up and down.

Let’s do some cleanup after experiments:

As you see, no Prometheus is needed, everything is pretty simple. We’ve spent a few minutes on KEDA and autoscaling setup (of course if we skip steps with RabbitMQ setup).

## References

-   KEDA project on Github: [https://github.com/kedacore/keda](https://github.com/kedacore/keda)
-   Documentation: [https://keda.sh/docs/2.3/](https://keda.sh/docs/2.3/)
-   Samples: [https://github.com/kedacore/samples](https://github.com/kedacore/samples)

_This is a translation of my_ [_original article_](https://habr.com/en/post/569760/) _from habr.com._

[

![](https://miro.medium.com/max/1400/1*t_MOgMTr_f1N4qHN_R7KVA.png)

](https://faun.to/i9Pt9)

Join FAUN: [**Website**](https://faun.to/i9Pt9) 💻**|**[**Podcast**](https://faun.dev/podcast) 🎙️**|**[**Twitter**](https://twitter.com/joinfaun) 🐦**|**[**Facebook**](https://www.facebook.com/faun.dev/) 👥**|**[**Instagram**](https://instagram.com/fauncommunity/) 📷|[**Facebook Group**](https://www.facebook.com/groups/364904580892967/) 🗣️**|**[**Linkedin Group**](https://www.linkedin.com/company/faundev) 💬**|** [**Slack**](https://faun.dev/chat) 📱**|**[**Cloud Native** **News**](https://thechief.io/) 📰**|**[**More**](https://linktr.ee/faun.dev/)**.**

**If this post was helpful, please click the clap 👏 button below a few times to show your support for the author 👇**