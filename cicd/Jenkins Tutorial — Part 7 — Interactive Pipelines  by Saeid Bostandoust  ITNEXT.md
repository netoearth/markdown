![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this part of the Jenkins tutorial, I will talk about a pipeline `input` directive that is used to prompt an input during the job execution. The input directive will cause the build process to pause and waiting for direct human interactions. We will use this option usually for the CD, Continuous Delivery, pipelines. If you didn’t read the previous articles, I suggest you read them first. See them in addition to some examples on the tutorial’s official GitHub repository.

Please note that all parts of this tutorial are implemented on the following customized Jenkins stack. You can use it too.

## Why and When to use the Input directive:

As I said before, the input directive will cause the job to pause and waits for direct human interactions. So why do we need that? In some cases, usually in continuous delivery pipelines, we want to double-check everything before letting them go to our production environments. In such a situation, the input directive is the thing that we need. Another use of the input directive is to get user values during the build process.

You should note this **important tip**, pausing the build process will cause the Jenkins job executor in a **busy** state! If you have a limited number of job executors, you should not use the input directive. I will explain the Jenkins cloud agents in future articles to let us get around this limitation.

In the above example, simple use of the input directive is shown. The input directive will take effect just after evaluating the `options` block of the stage and before evaluating the `agent`, and `when` blocks of the stage.

## Who can submit the input:

By default, any user can submit the input, but we can limit those who can submit it by using the **submitter** option of the input directive.

**submitter**: A comma-separated list of users or groups who are allowed to submit this input. The default is any user.

**submitterParameter**: If the submitter option is used, this option can be used to set the submitter username in the environment variables.

## Input in combination with Parameters:

As I said before, one of the other use of the input directive is to get user values during the build process. By using the **parameters** option in the input block, we can ask the user to submit values.

For getting more hands-on parameters, see [this](https://itnext.io/jenkins-tutorial-part-3-parameterized-pipeline-3898643ac6ad) article.

Here is an example screenshot:

![](https://miro.medium.com/max/1400/1*cSggnnMFzJ6USbH0xsBQ0g.png)

## Input within the Pipeline steps:

The input function can be used within the pipeline steps to pause the build or wait for the user to enter values. The important difference between the input function and the input directive explained above is the input function gets executed after the `options`, `agent`, and `when` blocks.

## The final words:

The input directive is useful for the Jenkins environments which are using cloud agents that I introduce them in future articles. With the power of the input directive, you can pause the build process for human interactions or using them to enter values during the build.

Stargaze the following repository if you find this tutorial is useful: