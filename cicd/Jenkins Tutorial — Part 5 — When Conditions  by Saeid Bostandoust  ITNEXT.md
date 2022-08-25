![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this article of the Jenkins tutorial series, I intend to explain When Conditions in Jenkins pipelines. Imagine you want to execute pipeline stages when a condition or some conditions are met. How can you do that? The answer is When Conditions. So, let’s get started.

If you are interested in this tutorial series, **STAR**ize the following GitHub repo. This repo is a special repo that I created for this tutorial.

## Jenkins “when” Directive:

Execution of the pipeline stages can be controlled with conditions. These conditions must be defined in the `when` block within each stage. Jenkins supports a set of significant conditions that can be defined to limit stage execution. Each `when` block must contain at least one condition. If more than one condition is declared in the `when` block, all conditions should return true for that stage being executed. Moreover, more complex conditions that will explain below can be defined using the nested ones.

Stage **Test** in the above example is run only and only one time at the first run of the pipeline job. This stage is not run from build two onwards.

## Built-in Conditions:

Conditions that Jenkins supports natively are called Built-in conditions. In addition to these conditions, some plugins may add more conditions. For such conditions see Jenkins plugins documents.

**branch** checks the source code branch name with the given pattern. If the branch name is matched to the pattern, the stage is executed. You should note that this condition only works on Multibranch pipelines.

The **Test** stage in the below example executes when the branch name is started with “release-” All ANT-style patterns are accepted.

**buildingTag** runs the following stage if the current git commit has a tag. This condition has been affected by an unfixed bug, if you see it didn’t work, you should set TAG\_NAME environment variable manually.

**changelog** gets a regular expression and matches it with the message of the last git commit. If the log message is matched to the given pattern, the following stage gets executed. You should note that this condition works only in Multibranch pipelines and those Pipelines that the script is from the SCM repo. In the below example, the stage is run when the git commit message contains “**Test**” string.

**changeset** watches files/directories changes with the given pattern. It sees the last git commit, and if any files/directories had changed which matches the given pattern, the stage is executed.

**environment** checks the environment variable value.

**equals** runs the stage if the actual value equals the expected one. For example, the following condition runs the stage if the current build number is one. So to speak, it runs only once.

**expression** gets a Groovy language expression and runs the following stage if that expression evaluates true. In the case of Strings, all values include “0” and “false” are returned **true**. If you intend to use strings as a part of the expression, you must set the value to **null** to evaluate it as **false**.

**tag** runs the stage if the TAG\_NAME variable is matched the given pattern. If the pattern is empty, it runs the stage if the TAG\_NAME variable exists.

**triggeredBy** executes the stage when the current build has been triggered by the given param. This condition is useful for notification purposes.

## Jenkins Complex Conditions:

Complex conditions are usually is a set of conditions explained above. Jenkins supports three complex/nested conditions. `not`, `allOf` and `anyOf` are complex conditions that are used in conjunction with conditions.

**not** executes the stage if the nested condition is **false**.

**allOf** executes the stage if all nested conditions are **true**.

**anyOf** executes the stage if at least one nested condition is **true**.

## When is the “when” directive evaluated?

By default, the `when` directive is evaluated after `agent`, `input` and `options` directives. You can change those ones with `beforeAgent`, `beforeInput` and `beforeOptions` within the `when` block.

```
when {    beforeAgent true    beforeInput true    beforeOptions true    // Write your conditions here...}
```

The Jenkins CI is a great and rich tool to implement CI/CD pipelines. Sometimes, you may find it very complex, but it doesn’t. You should own day-to-day practices to make your knowledge solid. If you have any questions, comment below or open an issue on the tutorial’s GitHub repo.