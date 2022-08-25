![](https://miro.medium.com/max/1400/1*isD_dDi7u493JTm4ZudxOw.jpeg)

In this article, I will explain how to create parameterized pipelines in Jenkins. A parameterized pipeline allows us to set needed parameters dynamically at build time. Before continue reading, let’s read previous parts if you didn’t yet. Let’s get started.

[Jenkins Tutorial — Part 1 — Pipelines](https://itnext.io/jenkins-tutorial-part-1-pipelines-bd1397cf5509)

[Jenkins Tutorial — Part 2 — Pipeline Variables](https://itnext.io/jenkins-tutorial-part-2-pipeline-variables-5e4783aa2c07)

## Jenkins Pipeline Parameters:

A set of various parameters can be defined in the pipelines. All of them should be defined in the `parameters` section of the pipeline.

After running the pipeline for the first time, **Build with Parameters** is shown in the pipeline menu. From now on, when you want to build, you should set parameters before that being start. Along with parameter definition, you can also set their default values.

![](https://miro.medium.com/max/1400/1*Ih7YklyM7OJe4I6kcNK0wQ.png)

## Pipeline Parameter Types:

Jenkins supports a list of useful parameter types. You should note that, in addition to native parameter types, some plugins may add new types. You should read documents of that plugins to know how their work. In this section, I will introduce native parameter types.

**booleanParam** type lets you define boolean parameters. You can set the default value for this type. Note that all parameters have the description argument that you can use to describe the purpose of that parameter goal.

```
booleanParam(name: "NAME", defaultValue: true, description: "DESCRIPTION")
```

**string** type allows you to define single-line strings. In addition to default-value and description, it also supports an additional argument `trim` to remove white spaces on both sides of the entered value.

```
string(name: "NAME", defaultValue: "VALUE", trim: false, description: "DESCRIPTION")
```

**text** lets you define multi-line string texts.

```
text(name: "NAME", defaultValue: "VALUE", description: "DESCRIPTION")
```

**password** parameter lets you define password input on the build page. The value of this type is not shown — concealed — on both the build page and the pipeline console.

```
password(name: "NAME", defaultValue: "VALUE", description: "DESCRIPTION")
```

**choice** is for defining a multi-choice drop-down menu with a set of pre-defined values. You can define this type with a list of values.

```
choice(name: "NAME", choices: ["VALUE1", "VALUE2", "VALUE3"], description: "DESCRIPTION")
```

## Using Parameter Value in the Pipeline:

Parameter values can be accessed using **params** helper. In addition to this preferred method, they can be accessed likewise through the environment variables described previously in the previous [article](https://itnext.io/jenkins-tutorial-part-2-pipeline-variables-5e4783aa2c07). To access the parameter value use `$params.NAME` or `${params.NAME}` syntax. Parameters can be accessed within all pipeline stages.

```
echo "Hello $params.NAME"
```

By using parameters, you can create dynamic pipelines. Imagine you want to create a pipeline that can be worked on multiple environments — development, staging, production — and these environments have different configurations, or you want to supply configurations at build time. With Parameterized Pipeline you can reach these goals. How we can do that and making dynamic pipelines will be introduced in future articles.