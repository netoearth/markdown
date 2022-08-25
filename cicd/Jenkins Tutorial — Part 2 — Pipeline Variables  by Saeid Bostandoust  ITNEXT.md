![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this article, I will introduce ways of define and using variables in the pipelines. Read the [previous](https://itnext.io/jenkins-tutorial-part-1-pipelines-bd1397cf5509) part if you didn’t yet. Variables are the most important part of the pipelines that able us to making repeatable pipelines and separating codes from configurations. That is why you should learn how to define and use variables. So, let’s get started.

## Before continue reading:

If you need a complete Jenkins stack with a set of tools and plugins, you can deploy the one that I use in our company, training courses, and this article series. This stack is available for both Kubernetes and Docker.

## Jenkins pipeline environment variables:

Pipeline environment variables can be defined in the `environment` section. This section can be defined globally on the entire pipeline or in the specific pipeline stage. You can define your environment variables in both — global and per-stage — simultaneously. Globally defined variables can be used in all stages but stage defined variables can only be used within that stage. Environment variables can be defined using `NAME = VALUE` syntax. To access the variable value you can use these three methods `$env.NAME`, `$NAME` or `${NAME}` There are no differences between these methods. Here is a complete pipeline sample with environment variables example.

## Jenkins Credentials and Environment variables:

A useful helper method can be used as a value of environment variables to access pre-defined Jenkins credentials. All well-known credentials — Secret Text, Secret File, Username/Password and SSH with Private Key as well as other credential types that installed by other plugins — can be accessed using `credentials(id)` helper. Here is an example.

**The important tips:**

When we use one of the **“Username/Password”** or **“SSH with Private Key”** credentials, in addition to our defined variable name, two other environment variables are defined automatically by Jenkins too. These two automatically defined variables are named `VARNAME_USR` and `VARNAME_PSW` These extra variables are defined because these types of credentials are not single-value credentials. The value of these extra automatically defined variables is a part of the whole credential that has been accessed by our originally defined variable.

In the case of the **“Username/Password”** credential, suppose you define a variable called `MYUSERPASS` the value of `MYUSERPASS` is `username:password` the value of `MYUSERPASS_USR` is `username` and the value of `MYUSERPASS_PSW` is `password` respectively. Here is an example.

In the case of the **“SSH with Private Key”** credential, suppose you define an environment variable `MYSSHINFO` the value of `MYSSHINFO` is the location of the SSH key file. If that credential has an SSH username and passphrase, `MYSSHINFO_USR` and `MYSSHINFO_PSW` are defined automatically and those values are assigned to these variables respectively.

Where environment variables can be used and which Jenkins methods/functions can accept environment variables as well as other methods to define/access variables will be described in future articles. If you’re interested, subscribe to my profile.