![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this part of the Jenkins tutorial, I will explain **Jenkins Post Actions** and show you how you can write post actions for your pipelines. If you didn’t read previous articles, read them first. So let’s get this party started.

[Jenkins Tutorial — Part 1 — Pipelines](https://itnext.io/jenkins-tutorial-part-1-pipelines-bd1397cf5509)

[Jenkins Tutorial — Part 2 — Pipeline Variables](https://itnext.io/jenkins-tutorial-part-2-pipeline-variables-5e4783aa2c07)

[Jenkins Tutorial — Part 3 — Parameterized Pipelines](https://itnext.io/jenkins-tutorial-part-3-parameterized-pipeline-3898643ac6ad)

## What is Jenkins Pipeline Post Actions?

Post Actions are just like other normal stages but that running in specific conditions. Jenkins supports 10 special action conditions which are running when these conditions meet. They are related to run status and can be defined in the `post` block both for the whole pipeline and per-stage.

**always** runs always as its name says. This post action is not bound to any specific build/run statuses. Some actions like sending notifications and saving logs can be done within this block.

**changed** runs only when the status of the current build is **different** than the previous one. Some actions like sending notifications and running another pipeline to investigate the pipeline can be done within this block.

**fixed** runs when the current build status is **successful**, and the previous build failed or was unstable. Some actions like sending notifications or triggering another pipeline can be done within this block.

**regression** runs in situations that the current build status is failure, aborted or unstable, and the previous run **was** **successful**. Some actions like rollbacking, for example in CD pipelines, can be done in this post block.

**unstable** runs when the current status is marked unstable. For example, when the tests are failed, or application code coverage is less than the desired percent. Some actions like trying other tests and running lazy canary environment can be done within this block.

**aborted** runs when the build process is aborted, usually manually. For example, everything that happened before aborting the pipeline should be reverted with this post action.

**failure** runs when the current status of build is failure. Some actions like sending alerts, rollback CD pipelines, deleting intermediate artifacts, etc, should be done within this block.

**success** runs when the current status of build is success. Some actions like saving artifacts, uploading them, and triggering the CD pipeline must be done with this post action.

**unsuccessful** runs when the build status is anything except success. This condition is a mix of failure, aborted and unstable status. You can do related things that are done within those blocks in this block.

**cleanup** always runs after all other conditions are evaluated. All cleanup processes like deleting intermediate artifacts, logs, temp files and etc should be done with this action.

## The complete Jenkins Post Actions example:

I should remind you these conditions/blocks can be defined both for the whole pipeline and per-stage. The `post` block that is defined after the `stages` block is evaluated after running all stages and the `post` block that is defined per-stage, after `steps` block, is evaluated after running that stage. Where you define these conditions or which condition you need to define depends on your pipeline needs and the aim of the pipeline. In many real world pipelines, you need to define various conditions both on the pipeline and per-stage. Wish you all the bests.