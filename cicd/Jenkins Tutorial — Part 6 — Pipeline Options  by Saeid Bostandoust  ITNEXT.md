![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this part of the Jenkins tutorial series, we will dive into Jenkins Pipeline Options. With the options, you can configure the pipeline and change some pipeline behaviors. I suggest you read the previous articles before continue reading. You can find all published articles in the following GitHub repo.

If you interested in this tutorial series, **STAR**gaze this repo:

## Jenkins Pipeline Options:

Several pipeline options exist in the Jenkins CI. Some of them are native options, and some of them are added due to installing plugins. I intend to explain the options that are available in the Jenkins stack. You can deploy this stack both on Kubernetes and Docker environments. The following repositories are everything you need to deploy the Jenkins complete stack.

## How to configure Pipeline options:

Pipeline Options are set inside the `options` block of the pipeline.

Here is an example:

So let’s introduce available options and their works.

**buildDiscarder** that is shown in the above example, is used to delete old builds and artifacts. You can set the rotator based on the days and numbers of the builds and artifacts. Look at the above example.

**checkoutToSubdirectory** is used to change the directory of `checkout scm` command to a subdirectory of the pipeline workspace.

```
checkoutToSubdirectory("mycheckout")
```

**copyArtifactPermission** is used to define the list of projects that they are allowed to copy the artifacts of this job. You can learn more about copying artifacts from its [plugin](https://plugins.jenkins.io/copyartifact) (Copy Artifacts Plugin) document.

```
copyArtifactPermission("job1,myjob,testjob5")
```

**disableConcurrentBuilds** is used to disable concurrent builds.

```
disableConcurrentBuilds()
```

**disableResume** disables the build resume function if the controller of the Jenkins restarts. The controller is the very the Jenkins master.

```
disableResume()
```

**durabilityHint** is used to override the speed/durability option of the job. You can make jobs recoverable after the controller outage (it consumes more time and more disk writes), or you can make jobs faster (cannot recoverable in the case of unplanned outage) This option is advanced.

-   **MAX\_SURVIVABILITY**: Maximum durability but slowest.
-   **SURVIVABLE\_NONATOMIC**: Mix of durability and better speed.
-   **PERFORMANCE\_OPTIMIZED**: Run like a rabbit.

In the case of **PERFORMANCE\_OPTIMIZED** the Jenkins controller should be shutdown gracefully to save running pipelines.

```
durabilityHint("PERFORMANCE_OPTIMIZED")
```

**newContainerPerStage** is used in conjunction with the **docker** and **dockerfile** top-level agent. The normal behavior of running job stages is that all stages are run within one container, but with this option, each stage is run in a separate container in the same node.

**parallelsAlwaysFailFast** is used to failFast all parallel stages. If any of the parallel stages is failed, the other ones are also aborted, and the build status will be set to failure. It’s an advanced option.

```
parallelsAlwaysFailFast()
```

**preserveStashes** is used to preserve stashed files. During building a job, some files may be created, and they are removed at the end of the build. In some scenarios, you need to restart the build from a specific stage. In this case, you need the ephemeral files that are created during the previous build. Stash is here to achieve this need.

It will be explained deeply in future articles.

```
preserveStashes()
```

**quietPeriod** is used to quite a build for a specific period of time. When a job is triggered, a new build is put on the queue, but for a period of time that is set by this option, the Jenkins does not start it.

```
quietPeriod(30)
```

**retry** the pipeline body for N times if any exception happens.

```
retry(3)
```

**skipDefaultCheckout** is used to stop automatic checkout during entering a new agent. You must checkout manually with `checkout scm` command.

```
skipDefaultCheckout(true)
```

**skipStagesAfterUnstable** skips the further stages if the build status becomes UNSTABLE. It is used to stop the whole pipeline.

```
skipStagesAfterUnstable()
```

**timeout** is used to specify the build timeout. By default, builds don’t have any timeout limit. After the timeout is reached, the **FlowInterruptedException** is thrown, and the pipeline is aborted.

```
timeout(time: 300, unit: "SECONDS")
```

## Final words:

A great many options exist in the Jenkins. Several of them are native options, and many of them are added by the plugins. In this article, I tried to explain some of them, but I intend to dive deeply into each one in future articles. If you have any questions, file an issue on the tutorial GitHub repo.