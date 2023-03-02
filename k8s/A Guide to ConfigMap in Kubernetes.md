For years, developers have adopted the practice of separating configuration data from code. As an integral part of the [Twelve-FactorApp Methodology](https://12factor.net/), this practice offers many benefits, top of which is application portability and agility. In Kubernetes, ConfigMaps provide that functionality: they exist to separate code and configuration. After all, the configuration can substantially vary across deployment, but code does not.

In this article, we’ll dive deep into ConfigMaps, what they are, why you need them, and the different ways you create them.

## What is Kubernetes ConfigMap?

In Kubernetes, a ConfigMap is an API object that holds key-value pairs of configuration data. Pods can process ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume. ConfigMaps are not encrypted in any way, with all the data available to all who have access to the file. Thus, they are unsuitable for confidential data storage.

Designed with application portability in mind, ConfigMaps decouple environment-specific configurations from containers. They were not, however, designed to store large volumes of data. Data held in a ConfigMap cannot exceed 1 MB.

Information in a ConfigMap can be stored in the data field and the values can be stored in one of two ways:

-   Individual key-value pair properties
-   Fragments of a configuration format (such as file-like keys).

**Further Reading**

-   [Kubernetes Containers Explained](https://www.weave.works/technologies/the-journey-to-kubernetes/)
-   [CI/CD for Kubernetes](https://www.weave.works/technologies/ci-cd-for-kubernetes/)
-   [Guide to GitOps - What you need to know](https://www.weave.works/technologies/gitops/)

## Why do you Need ConfigMaps?

There are different configuration settings for each stage of the software lifecycle (development, production, and testing). And while configuration settings may change, application code doesn't. This is where ConfigMaps come in. ConfigMaps are used to separate application code from the environment-specific configuration.

With ConfigMaps, there’s no need to hardcode the configuration data in the Pod specification. You can change the configuration settings during runtime and create complex configurations with ease. Finally, with Kubernetes v1.19, you can create immutable ConfigMaps and protect your deployments from accidental (or intentional) configuration changes.

## How does A ConfigMap Work?

When it comes to the Kubernetes infrastructure, different config files are required for the different environments (dev, stage, production). Once each ConfigMap is created, it is added to the Kubernetes cluster. Containers in the pod will then reference the ConfigMap and use the values stored.

![ConfigMap_Diagram.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt7118bc80b8cd018a/62f50128d3b8a57004568c03/ConfigMap_Diagram.jpg)

**Further Rea****ding:** [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)

## How to Create A ConfigMap?

Although primarily used to configure the settings of containers running inside Pods, ConfigMaps are also used by other parts of the system since they can be mounted as data volumes. They can be used as add-ons or operators that conform to the parameters set in the corresponding ConfigMap.

You need to consider the following when creating a ConfigMap:

-   Information stored in ConfigMaps is not encrypted. And thus ConfigMaps should not be used to store confidential information
-   ConfigMap file size should not exceed 1MB
-   ConfigMap object name must be a valid DNS subdomain name.
-   ConfigMaps reside in Namespace and only pods residing in the same namespace can reference them.
-   ConfigMaps can’t be used for static pods.

Here’s an example of a ConfigMap configuration, taken from the Kubernetes [documentation](https://kubernetes.io/docs/concepts/configuration/configmap/).

```
apiVersion: v1
kind: ConfigMap
metadata:
 name: game-demo
data:
 # property-like keys; each key maps to a simple value
 player_initial_lives: "3"
 ui_properties_file_name: "user-interface.properties"
 # file-like keys
 game.properties: |
   enemy.types=aliens,monsters
   player.maximum-lives=5    
 user-interface.properties: |
   color.good=purple
   color.bad=yellow
   allow.textmode=true
```

There are various methods to creating a ConfigMap, all of which is outlined in-depth below.

The basic syntax for creating a ConfigMap is:

```
kubectl create configmap [configmap_name] [attribute] [source]
```

### Method #1: Creating from a YAML file.

Here, you define the dictionary of key-value pairs of configuration data in a .yaml file. Using the kubectl command, you create the ConfigMap.

Use the following command:

```
kubectl create configmap [configmap_Dev] --from-file [path/to/yaml/file]
```

You then mount the ConfigMap as a Volume in your Pod’s YAML specification.

### Method #2: Creating from directories

A ConfigMap can be created from multiple files in the same directory. With this method, only regular files in the directories are packaged into the new ConfigMap and any other entries are ignored (e.g. subdirectories, symlinks, pipes, devices, etc).

Use this command to create a ConfigMap from a directory:

```
kubectl create configmap [configmap_Dev] --from-file [path/to/directory]
```

### Method #3: Creating from files

A ConfigMap can be created from an individual file or from multiple files in any plaintext format using the kubectl create configmap.

To create a ConfigMap from a single file, use this command:

```
kubectl create configmap [configmap_Dev] --from-file [path/to/file]
```

To create a ConfigMap from multiple files, use this command

```
kubectl create configmap [configmap_Dev] --from-file [path/to/file1] --from-file [path/to/file2] --from-file [path/to/file3]
```

### Method #4: Creating literal values

A ConfigMap can also be created using literal values using the kubectl command with the --from-literal argument.

Use this syntax to create the ConfigMap:

```
kubectl create configmap [configmap_Dev] --from-literal [key1]=[value1] --from-literal [key2]=[value]2
```

### Method #5: Create a ConfigMap from Generator

From Kubernetes version 1.14 and onwards, you can use kubectl to manage objects using Kustomize: a tool designed to customize Kubernetes configurations. One of its features is the configMapGenerator which generates ConfigMaps from files or literals.

The ConfigMap can be created using the generators and then applied to create the object on the Apiserver. You should specify the generators in the customization.yaml inside a directory.

Here’s an example of a ConfigMap built for Kustomization.

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: my-namespace
configMapGenerator:
    - name: my-map
      files:
       - data/file1.json
```

For more information about creating ConfigMaps using the kubectl command, use the following kubectl create configmap --help. To learn more about creating a ConfigMap from a generator, read the official documentation [here](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) or [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/).

## Conclusion

ConfigMaps are used to decouple configuration data from application code to keep containerized applications portable. There are various ways to create the ConfigMaps and multiple ways to consume them.