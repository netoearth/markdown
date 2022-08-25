![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

Running CI/CD jobs by hand is not a good method to start a pipeline process since everything that is done manually competes with automation. Jenkins provides various methods to start jobs automatically. These methods are categorized in the **Build Triggers** section. In this part of the Jenkins tutorial, I will explain common methods for triggering pipelines. Please note that some methods existed due to installing plugins and may not be found in Jenkins bare installations. So, I suggest you use the following Jenkins stack that has everything you need.

If you find this series of articles are helpful, please STARgaze the official repository of the tutorial. You can find all examples in this repository too.

## Jenkins Build Triggers:

As I said, Jenkins has various build triggering methods, cron, webhook, URL, upstream, etc. I will explain the popular ones. All triggers should be defined in the **triggers** block of the pipeline.

`**cron**` is something like Unix/Linux cron. To write cron trigger use `cron "MINUTE HOUR DOM MONTH DOW"` syntax. You should note that scores of different triggers can be used for the same job.

**DOM** is Day Of the Month and **DOW** is Day Of the Week.

Here is a simple example:

`**cron "0 * * * *"**` runs at minute 0 of each hour.

`**cron "0 0 * * *"**` runs at 00:00 (once a day).

`**cron "0 0 * 2,4,6,8,10,12 *"**` runs once a day in even months.

`**cron "* 1-7 * * *"**` runs at every minute from hour one through seven.

`**cron "*/10 * * * *"**` runs every 10th minute.

**Important tip (H “hash” magic keyword):** If you have a lot of jobs using the trigger `cron "0 0 * * *"` all of them will start at midnight (00:00). Starting a high number of jobs at the same time will put a lot of strain on Jenkins and may cause it to fail. To solve this strain, we can use the **H** symbol. This symbol tells Jenkins to use the hash model for the given cron trigger. So, to solve our example, we should use `cron "H H * * *"` instead of `cron "0 0 * * *"`. This cron will run the job once a day like a previous one, but not all at the same time.

`**cron "H H * * *"**` runs once a day.

`**cron "H H(0-4) * * *"**` runs between 00:00 to 04:59 (once).

`**pollSCM**` accepts a cron-style pattern and looks for source changes in your project version control system, for example, Git. The cron-style pattern is used to define the desired interval. You should note that it only works when the pipeline script is read from the SCM source.

The following trigger checks for source changes every 10 minutes. If any changes are detected, the pipeline will be triggered.

`**upstream**` accepts a comma-separated list of upstream projects, and the current job is triggered when any of them completes. In addition to a list of projects, you can set the desired state of upstream projects, UNSTABLE, FAILURE, etc. The **default** is **SUCCESS** build status.

Here is an example with threshold:

**hudson.model.Result.ABORTED**: The build was manually aborted.

**hudson.model.Result.FAILURE**: The build had a fatal error.

**hudson.model.Result.SUCCESS**: The build had no errors.

**hudson.model.Result.UNSTABLE**: The build had an unstable result.

## Jenkins Generic Webhook Trigger:

The Generic Webhook Trigger is a plugin that allows triggering pipelines from webhook. This plugin is very useful for integrating Jenkins with external applications like SCM, scripts, applets, etc.

To trigger the pipeline from command line, run the following command:

```
curl -i http://YOUR_JENKINS_URL/generic-webhook-trigger/invoke?token=unique-token-to-start-the-current-pipeline
```

To find more, see the [plugin](https://plugins.jenkins.io/generic-webhook-trigger) documentation.

## All-in-one example:

The following pipelines check for source changes every 10 minutes and run when any changes are detected. Runs periodically every one hour. Runs when the upstream project, my-base-project, is built successfully and can be triggered by the webhook with the given token.

## The end letter:

Build Triggers are very helpful to automate pipelines. The Generic Webhook Trigger is an awesome one that I will explain deeply later. With triggers, we can create an extraordinary lifecycle for our environments. You can find all the used examples in the following repository. Good luck.