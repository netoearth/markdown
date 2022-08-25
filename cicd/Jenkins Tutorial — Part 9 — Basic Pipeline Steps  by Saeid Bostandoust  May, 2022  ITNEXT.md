![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

Living in the pipeline cannot be accomplished without steps and commands. The Jenkins [**workflow-basic-steps**](https://plugins.jenkins.io/workflow-basic-steps) plugin provides several commonly used steps/commands to writing pipelines. In this part of the completest Jenkins tutorial, we go deep into the commands that are provided by the discussed plugin. You should note that a multitude of Jenkins plugins may add some steps and commands. All of those steps are available in the following Jenkins stack and will be explained in future articles.

## Jenkins Basic Pipeline Steps:

The following commands can be used in the `**steps**` block of the pipeline. In addition to the steps block, these commands can also be used in all condition blocks of the `**post**` block.

`**echo**` is used to **print a message**. The following code shows the echo method in addition to the ways you can use to call methods.

`**dir**` is used to change the current working directory. Any codes that exist inside the dir block are executed within that directory.

`**pwd**` is used to return the current working directory. It accepts a boolean argument called **tmp** that returns a temporary directory of the job workspace, a special directory that is used to store temporary files.

`**isUnix**` returns True if the running agent is a Unix-like machine (Linux, macOS, etc.) and False if the running agent is a Windows.

`**fileExists**` returns True if the given file is exist.

`**readFile**` reads the file and returns its content. It also accepts an optional argument called encoding to encode the file content with the specified encoding algorithm. Encoding may use for reading binary files.

`**writeFile**` writes the content to the given file. If the optional argument, encoding, is specified, the content is decoded, and the decoded content is written to the file.

`**deleteDir**` deletes the current directory, usually the job workspace. It deletes all files and subdirs in the active path. If you want to delete the specific directory, you should first **dir()** to it and use **deleteDir()** inside its block. The **deleteDir** command is usually used in the post stages.

`**catchError**` catches the error and continues the pipeline. The pipeline default behaviour is to fail the whole pipeline if any error occurs during each step. In most cases, you need to continue the pipeline to run some commands, send notifications, etc. The catchError is used to achieve this ability.

```
catchError(    stageResult: "SUCCESS",    buildResult: "FAILURE",    catchInterruptions: true,    message: "Error") {    // your pipeline codes.}
```

catchInterruptions is used to catch pipeline interruptions like aborting the pipeline manually. By default, if you abort the pipeline while the pipeline is in catchError block, the pipeline is not aborted, and other steps will be run either. To prevent this behaviour, set this parameter to false.

`**warnError**` is like catchError, but if any error occurs when running the pipeline inside its block, it sets the stageResult and the buildResult to UNSTABLE status.

```
warnError(    catchInterruptions: true,    message: "Unstable build message") {    // your pipeline codes.}
```

`**mail**` is used to send an email inside the pipeline.

```
mail(    subject: "Subject",    body: "Body",    from: "jenkins@localhost",    to: "ssbostan@yahoo.com")
```

Please note that you should have an SMTP server on the Jenkins machine or a port forwarder to send requests to an external SMTP server.

`**unstable**` sets the stage result to unstable.

```
unstable(message: "Your stage is unstable!")
```

`**timeout**` is used to set an execution time limit for commands that are executed inside its block. For example, you want to run a command to check something “service availability, healthcheck, etc.” and it should respond in less than 30s. In such cases, you have to use the timeout command. The default unit of this command is MINUTES, and it can be changed to NANOSECONDS, MICROSECONDS, MILLISECONDS, SECONDS, MINUTES, HOURS, DAYS. In addition to the traditional timeout scheme, it supports the activity-based timeout. The activity-based scheme can be enabled by setting the activity parameter to true.

```
timeout(    time: 15,    unit: "SECONDS",    activity: true) {    // your pipeline codes.}
```

If the timeout is reached, the pipeline will be terminated with ABORTED status. You can use catchError or warnError to catch it.

`**sleep**` pauses the pipeline for the specified amount of time. The default unit is SECONDS, and it can be changed like the timeout command.

```
sleep(10)sleep(time: 1, unit: "MINUTES")
```

`**retry**` retries its body (up to N times) if any exception happens during its body execution. If the final attempt returns an exception, it will exit with a failure status. For example, you want to check your database before running application tests. In such a situation, you need something like the following pipeline.

## Final words:

We have many things to learn about the Jenkins and Jenkins pipelines because it has many plugins which add various commands, functions, classes, etc. I will explain more about the Jenkins, pipeline directives and commands, and plugins in the next articles. You can find previous ones in the following GitHub repository. Good luck.