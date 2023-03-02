## An interesting ChatGPT bot project for K8s

![](https://miro.medium.com/max/667/1*02a4Q1CXV8UVmFVbaBA2zA.png)

Today I’d like to introduce an interesting projected called “[K8s ChatGPT Bot](https://github.com/robusta-dev/kubernetes-chatgpt-bot)”. The purpose of this project is to deploy a ChatGPT bot for your K8s issues. You can ask ChatGPT how to solve your Prometheus alerts, get pithy responses, no more solving alerts alone in the darkness!

This setup does require access to [Robusta platform](https://home.robusta.dev/), if you don’t have access yet, you can check out my last article for how to set it up: “[K8s — Robusta, K8s Troubleshooting Platform](https://medium.com/dev-genius/k8s-robusta-k8s-troubleshooting-platform-efd389b47f24)”

Below is a screenshot of how it work:

![](https://miro.medium.com/max/700/1*p3A_9ZV0PqgAXn4oOjEL5g.png)

Pic from [robusta](https://www.loom.com/share/964cd8735a874287a9155c77320bdcdb)

You can check out the full video here ([K8s ChatGPT Bot Demo](https://www.loom.com/share/964cd8735a874287a9155c77320bdcdb)).

## How it Works

This bot is implemented using [Robusta.dev](https://github.com/robusta-dev/robusta), an open source platform for responding to K8s alerts. The process looks like:

-   Prometheus forwards alerts to the bot using a webhook receiver.
-   The bot asks ChatGPT how to fix your alerts.
-   You stockpile food in your pantry for the robot uprising.

## Prerequisites

-   An existing Slack workspace
-   An existing K8s cluster
-   Python3.7 and plus

## How to Install

## Generate Robusta Config file

-   Prepare Python venv for `robusta` ([https://home.robusta.dev/](https://home.robusta.dev/))

```
$ python3.10 -m venv robusta$ source robusta/bin/activate(robusta) $ pip install -U robusta-cli --no-cacheCollecting robusta-cliDownloading robusta_cli-0.10.10-py3-none-any.whl (223 kB)     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 223.8/223.8 kB 30.0 MB/s eta 0:00:00Collecting pymsteams<0.2.0,>=0.1.16  Downloading pymsteams-0.1.16.tar.gz (7.6 kB)  Preparing metadata (setup.py) ... done...Successfully installed PyJWT-2.4.0 appdirs-1.4.4 autopep8-2.0.1 black-21.5b2 cachetools-5.2.1 certifi-2022.12.7 cffi-1.15.1 charset-normalizer-3.0.1 ... ruamel.yaml.clib-0.2.7 six-1.16.0 slack-sdk-3.19.5 tenacity-8.1.0 toml-0.10.2 tomli-2.0.1 typer-0.4.2 typing-extensions-4.4.0 urllib3-1.26.14 watchgod-0.7 webexteamssdk-1.6.1 websocket-client-1.3.3
```

-   Once `robusta` is installed, let’s generate a configuration file:

```
$ robusta gen-configRobusta reports its findings to external destinations (we call them "sinks").We'll define some of them now.Configure Slack integration? This is HIGHLY recommended. [Y/n]: YIf your browser does not automatically launch, open the below url:https://api.robusta.dev/integrations/slack?id=xxxx
```

## Configure Slack Integration

Open a web browser and go to the above url: https://api.robusta.dev/integrations/slack?id=xxxx

![](https://miro.medium.com/max/700/1*mH5-2jIdl7ROfNTsZt8DPg.png)

-   Update the permissions:

![](https://miro.medium.com/max/700/1*8FXoEO-Y9LDP50cQh6K0mA.png)

-   Then you should see the following screen:

![](https://miro.medium.com/max/700/1*k-piMzWsWJnaM4_tAI-eGA.png)

-   Now back to your terminal, you should see the following and just finish up based on the instructions:

```
$ robusta gen-configRobusta reports its findings to external destinations (we call them "sinks").We'll define some of them now.Configure Slack integration? This is HIGHLY recommended. [Y/n]: YIf your browser does not automatically launch, open the below url:https://api.robusta.dev/integrations/slack?id=xxxxYou've just connected Robusta to the Slack of: devopsfansWhich slack channel should I send notifications to? Configure Robusta UI sink? This is HIGHLY recommended. [Y/n]: YEnter your Gmail/Google address. This will be used to login: xxx@gmail.comChoose your account name (e.g your organization name): devopsfansSuccessfully registered.Robusta can use Prometheus as an alert source.If you haven't installed it yet, Robusta can install a pre-configured Prometheus.Would you like to do so? [y/N]: yPlease read and approve our End User License Agreement: https://api.robusta.dev/eula.htmlDo you accept our End User License Agreement? [y/N]: yLast question! Would you like to help us improve Robusta by sending exception reports? [y/N]: NSaved configuration to ./generated_values.yaml - save this file for future use!Finish installing with Helm (see the Robusta docs). Then login to Robusta UI at https://platform.robusta.devBy the way, we'll send you some messages later to get feedback. (We don't store your API key, so we scheduled future messages using Slack's API)
```

and in your slack channel, you should see:

![](https://miro.medium.com/max/653/1*PTZhhyyQP6_9RhcadeEPQQ.png)

## Install Robusta with Helm3

Install `robusta` repo and update:

```
$ helm repo add robusta https://robusta-charts.storage.googleapis.com && helm repo update"robusta" has been added to your repositoriesHang tight while we grab the latest from your chart repositories......Successfully got an update from the "kedacore" chart repository...Successfully got an update from the "robusta" chart repository...Successfully got an update from the "grafana" chart repository...Successfully got an update from the "prometheus-community" chart repository...Successfully got an update from the "stable" chart repositoryUpdate Complete. ⎈Happy Helming!⎈
```

## Update generated\_values.yaml File

Update the `generated_values.yaml` file with the following:

```
playbookRepos:  chatgpt_robusta_actions:    url: "https://github.com/robusta-dev/kubernetes-chatgpt-bot.git"customPlaybooks:- triggers:  - on_prometheus_alert: {}  actions:  - chat_gpt_enricher: {}globalConfig:  chat_gpt_token: YOUR KEY GOES HERE
```

## Deploy Robusta to K8s

```
$ helm install robusta robusta/robusta -f ./generated_values.yaml \ --set clusterName=dev-cluster
```

Verify the two Robusta pods and running with no errors in the logs:

```
$ kubectl get pods -A | grep robustadefault       alertmanager-robusta-kube-prometheus-st-alertmanager-0   2/2     Running     1 (4m19s ago)   9m25sdefault       prometheus-robusta-kube-prometheus-st-prometheus-0       2/2     Running     0               9m25sdefault       robusta-forwarder-6b7d8d9d88-2rv9d                       1/1     Running     0               9m29sdefault       robusta-grafana-64944bfcdc-v97xh                         3/3     Running     0               9m29sdefault       robusta-kube-prometheus-st-admission-patch-6zj4b         0/1     Completed   0               9m28sdefault       robusta-kube-prometheus-st-operator-7b985d7fb-c9f9t      1/1     Running     0               9m29sdefault       robusta-kube-state-metrics-688d794968-ll6gf              1/1     Running     0               9m29sdefault       robusta-prometheus-node-exporter-2k5f7                   1/1     Running     0               5m24sdefault       robusta-prometheus-node-exporter-zxsrg                   1/1     Running     0               9m29sdefault       robusta-runner-5868b494d6-m6292                          1/1     Running     0               9m29s$ robusta logssetting up colored logging2023-01-14 22:57:01.428 INFO     logger initialized using INFO log level2023-01-14 22:57:01.429 INFO     Creating hikaru monkey patches2023-01-14 22:57:01.429 INFO     Creating yaml monkey patch2023-01-14 22:57:01.429 INFO     Creating kubernetes ContainerImage monkey patch2023-01-14 22:57:01.430 INFO     watching dir /etc/robusta/playbooks/ for custom playbooks changes2023-01-14 22:57:01.431 INFO     watching dir /etc/robusta/config/active_playbooks.yaml for custom playbooks changes2023-01-14 22:57:01.431 INFO     Reloading playbook packages due to change on initialization2023-01-14 22:57:01.431 INFO     loading config /etc/robusta/config/active_playbooks.yaml2023-01-14 22:57:01.467 INFO     No custom playbooks defined at /etc/robusta/playbooks/storage2023-01-14 22:57:01.468 INFO     Cloning git repo https:...2023-01-14 22:57:07.364 INFO     connecting to server as account_id=8302df56-c554-4129-8b95-d143d1f2e3a2; cluster_name=dev-cluster2023-01-14 22:57:07.977 INFO     Initializing services cache2023-01-14 22:57:08.203 INFO     Initializing nodes cache2023-01-14 22:57:08.395 INFO     Initializing jobs cache2023-01-14 22:57:08.603 INFO     Getting events history2023-01-14 22:57:10.403 INFO     Cluster historical data sent.2023-01-14 23:04:43.681 INFO     cluster status {'account_id': '8302df56-c554-4129-8b95-d143d1f2e3a2', 'cluster_id': 'dev-cluster', 'version': '0.10.10', 'last_alert_at': '2023-01-14 23:04:18.959377', 'light_actions': ['related_pods', 'prometheus_enricher', 'add_silence', 'delete_pod', 'delete_silence', 'get_silences', 'logs_enricher', 'pod_events_enricher', 'deployment_events_enricher', 'job_events_enricher', 'job_pod_enricher', 'get_resource_yaml', 'node_cpu_enricher', 'node_disk_analyzer', 'node_running_pods_enricher', 'node_allocatable_resources_enricher', 'node_status_enricher', 'node_graph_enricher', 'oomkilled_container_graph_enricher', 'pod_oom_killer_enricher', 'oom_killer_enricher', 'volume_analysis', 'python_profiler', 'pod_ps', 'python_memory', 'debugger_stack_trace', 'python_process_inspector', 'prometheus_alert', 'create_pvc_snapshot'], 'updated_at': 'now()'}
```

## Robusta in Action

Finally we can see some actions here! By default, `Robusta` sends notifications when K8s pods crash.

So let’s create a crashing pod:

```
$ kubectl apply -f https://gist.githubusercontent.com/robusta-lab/283609047306dc1f05cf59806ade30b6/rawdeployment.apps/crashpod created$ kubectl get pods -A | grep crashdefault       crashpod-64db77b594-cgz4s                                0/1     CrashLoopBackOff   2 (21s ago)     36s
```

Once the pod has reached two restarts, check your Slack channel for a message about the crashing pod. You can see I got one already:

![](https://miro.medium.com/max/665/1*lNu7k61oF_vdSu8IIjUN4w.png)

## Interact with ChatGPT

Now we confirmed that `Robusta` is integrated with our Slack and K8s cluster, let’s interact with ChatGPT bot!

Trigger a Prometheus alert immediately, skipping the normal delays:

```
$ robusta playbooks trigger prometheus_alert alert_name=KubePodCrashLooping namespace=default pod_name=example-pod======================================================================Triggering action...======================================================================running cmd: curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"alert_name": "KubePodCrashLooping", "namespace": "default", "pod_name": "example-pod"}}'{"success":true}======================================================================Fetching logs...======================================================================2023-01-14 23:14:33.463 INFO     Error loading kubernetes pod default/example-pod. reason: Not Found status: 4042023-01-14 23:14:33.481 INFO     Error loading kubernetes pod default/example-pod. reason: Not Found status: 4042023-01-14 23:14:33.503 INFO     Error loading kubernetes pod default/example-pod. reason: Not Found status: 4042023-01-14 23:14:33.505 ERROR    cannot run pod_events_enricher on alert with no pod object: PrometheusKubernetesAlert(sink_findings=defaultdict(<class 'list'>, {'main_slack_sink': [<robusta.core.reporting.base.Finding object at 0x7fab53074e20>], 'main_ms_teams_sink': [<robusta.core.reporting.base.Finding object at 0x7fab53074700>], 'robusta_ui_sink': [<robusta.core.reporting.base.Finding object at 0x7fab40773a30>]}), named_sinks=['main_slack_sink', 'main_ms_teams_sink', 'robusta_ui_sink'], response={'success': True}, stop_processing=False, _scheduler=<robusta.integrations.scheduled.playbook_scheduler_manager_impl.PlaybooksSchedulerManagerImpl object at 0x7fab4088e0a0>, _context=ExecutionContext(account_id='8302df56-c554-4129-8b95-d143d1f2e3a2', cluster_name='dev-cluster'), obj=None, alert=PrometheusAlert(endsAt=datetime.datetime(2023, 1, 14, 23, 14, 33, 430401), generatorURL='', startsAt=datetime.datetime(2023, 1, 14, 23, 14, 33, 430406), fingerprint='', status='firing', labels={'severity': 'error', 'namespace': 'default', 'alertname': 'KubePodCrashLooping', 'pod': 'example-pod'}, annotations={}), alert_name='KubePodCrashLooping', alert_severity='error', label_namespace='default', node=None, pod=None, deployment=None, job=None, daemonset=None, statefulset=None)2023-01-14 23:14:33.524 INFO     Error loading kubernetes pod default/example-pod. reason: Not Found status: 4042023-01-14 23:14:33.696 ERROR    CallbackBlock not supported for msteams2023-01-14 23:14:33.697 ERROR    error sending message to msteamse=Invalid URL 'False': No schema supplied. Perhaps you meant http://False?======================================================================Done!======================================================================
```

Now switch to Slack, you will see a new alert, this time with a “Ask ChatGPT” button!

![](https://miro.medium.com/max/700/1*zKZkkCEJ1yWjXfqTLTVZfw.png)

That’s it! Congratulations, we just successfully installed our first K8s ChatGPT bot!

## Example2: Node at 100% Capacity

Below is another example for one node reaches 100% capacity:

![](https://miro.medium.com/max/563/1*5aKCbnHOdQf9TW5zQLcZSQ.png)

## Robusta UI

Robusta has a UI for the integration and also has a pre-configured Promethus system, if you don’t have your own K8s cluster yet, and wanted to try it out this ChatGPT bot, you can use their existing one!

![](https://miro.medium.com/max/700/1*1qpeRUSSejhFmGCpKmVc5A.png)

## Conclusion

It does takes some efforts to get the entire bot setup, but this is a fan project and got a great potential. I hope you enjoyed this article.

If you don’t have your own K8s cluster and an existing Prometheus monitoring system, you can use Robusta’s pre-configured Promethus: