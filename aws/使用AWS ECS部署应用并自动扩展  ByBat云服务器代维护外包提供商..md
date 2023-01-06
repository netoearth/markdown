尝试用docker去部署，研究了一下如何用AWS ECS做部署和自动扩展，跟传统的使用EC2来做部署使用ASG做扩展还是有一些不同的。

Amazon EC2 Container Service 是基于docker的具有高可扩展性，快速的容器管理服务，可以让你在EC2 instance集群中简单快速的部署和扩展应用程序。

[![1511850639-4822-KfTcly1fg78xsfn3jj315u0e60w0](https://www.bypat.com/wp-content/uploads/2017/11/1511850639-4822-KfTcly1fg78xsfn3jj315u0e60w0.jpg)](https://www.bypat.com/wp-content/uploads/2017/11/1511850639-4822-KfTcly1fg78xsfn3jj315u0e60w0.jpg)

ECS中有一些基本概念需要先了解一下

-   ECR – Elastic Container Registry 用来建立一个安全，可扩展，高可用的应用程序私有docker registry。
-   Cluster – 逻辑上的一组由EC2 instance组成的集群，你的container跑在这些集群上面。
-   Service – 在指定的cluster上来维护和启动一定数量的任务容器实例的服务。
-   Task Definition – 定义了由service维护的任务，比如image是哪个，CPU和Memory如何分配，端口号如何映射，容器跑起来时候执行哪些命令，有哪些环境变量设置等等。

## 如何使用ECS部署

接下来会介绍如何基于Application Load Balancer和ECS来部署自己的application

### 1\. 打包docker image并push到ECR

我们从Dockerfile来打包一个demo的image:

```
FROM nodejs:7.6.0

COPY . /source

COPY ./env/production.env /source/.env

WORKDIR /source

RUN yarn install --production

EXPOSE 3000

CMD ["yarn", "start"]

```

执行 `docker build -t 12345678.dkr.ecr.ap-northeast-1.amazonaws.com/demo:0.0.1 .` 来打包image

成功后push到ECR中

```
$ aws ecr get-login --region ap-southeast-1
$ docker push 12345678.dkr.ecr.ap-northeast-1.amazonaws.com/demo:0.0.1
```

可以自己写个shell脚本把这些都自动化起来

### 2\. 使用CloudFormation建立一个Application Load Balancer

可以参考AWS CF模板中ALB部分来写，主要是有AWS::ElasticLoadBalancingV2::LoadBalancer， AWS::ElasticLoadBalancingV2::Listener，AWS::ElasticLoadBalancingV2::TargetGroup这些资源，参考这个[demo模板](https://github.com/liul85/Autoscale-with-AWS--ECS/blob/master/alb.yml)

### 3\. 利用CloudFormation 来建立一个cluster

Cluster就是一组运行的EC2 instance，它需要在autoscaling组里面来管理保证cluster是可以随时扩展，所有这部分模板中应该保证有LaunchConfiguration， Cluster，Role，SecurityGroup参考这个[demo的模板](https://github.com/liul85/Autoscale-with-AWS--ECS/blob/master/cluster.yml)

### 4\. 建立一个ECS service

有了cluster之后，我们就可以利用service把容器任务运行起来了，这里我们再建一个CF模板来创建service的资源，

要想用service来管理容器任务，首先定义一个task，在TD里面我们定义了一个container，它有name，cpu，memoery等一些基本属性，注意这里的PortMappings，我们把HostPort设置为0(或者也可以不设置)，这里是为了使用动态端口映射，这样当这个container运行起来之后，ECS会给它随机分配一个主机端口，目前范围是从32768~61000，这样我们就可以在一个instance上同时跑多个container了，具体可以参考[这里](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)的portMappings中对hostPort的说明.

```
  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      ContainerDefinitions:
        - Name: !Join ['-', [!Ref 'ServiceName', !Ref 'BuildNumber']]
          Cpu: 512
          Essential: 'true'
          Memory: 512
          PortMappings:
            - HostPort: 0
              Protocol: tcp
              ContainerPort: !Ref 'ContainerPort'
          Image: !Join [':', [!Join ['/', [!Ref 'ECR', !Ref 'ServiceName']], !Ref 'BuildNumber']]
```

然后我们就可以创建一个service来使用这个task definition，这个service会关联到之前创建的LoadBalancer中，我们使用的是Application Load Balancer，需要在这里指定它关联的TargetGroupArn，这样当容器任务跑起来之后，service会把它注册到这个targetGroup下面。

```
  ECSService:
    Type: AWS::ECS::Service
    Properties:
      Cluster:
        Ref: ECSCluster
      DesiredCount:
        Ref: DesiredCount
      LoadBalancers:
        - ContainerName: !Join ['-', [!Ref 'ServiceName', !Ref 'BuildNumber']]
          ContainerPort: 3000
          TargetGroupArn:
            Ref: TargetGroup
      DeploymentConfiguration:
        MaximumPercent: '200'
        MinimumHealthyPercent: '50'
      TaskDefinition:
        Ref: TaskDefinition
      Role:
        Ref: ECSRole
```

当这三个CF创建的资源全都成功后，你的application就已经在容器中运行了，你可以通过ALB的地址来访问它。

### 5\. 部署新版本

当有新的提交之后，我们会有新的版本，就会打包新的image并push到ECR中，这个时候只需要把service中的BuildNumber修改为新版本，然后update这个Stack，service会自动创建新的TaskDefinition并运行起来，然后把老的Task从targetGroup中去掉，这里会有一个draining时间，默认5分钟，`MinimumHealthyPercent` 参数指定了在部署过程中原来的版本最小百分比，保证在部署过程中业务不会中断，这样就完成了新旧版本的替换。

## 自动扩展

上面我们的application已经可以运行在ECS服务中了，但是还没有任何自动扩展功能，当遇到很大的访问量时候就会有问题了，我们需要添加自动扩展，在ECS中自动扩展包括两部分

#### Cluster 扩展

Cluster扩展主要是通过EC2的autoScalingGroup来完成的，我们定义了ASG，scale up和scale down的policy，以及触发这些policy的alarm就可以完成了。

#### Service 扩展

service的扩展我们需要建立ServiceScalingTarget，它指定了最小和最大的capacity，以及哪个service扩展

```
  ServiceScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    DependsOn: ECSService
    Properties:
      MaxCapacity: !Ref 'ServiceMaxASG'
      MinCapacity: !Ref 'ServiceMinASG'
      ResourceId: !Join ['', [service/, !Ref 'ECSCluster', /, !GetAtt [ECSService, Name]]]
      RoleARN: !GetAtt [ECSAutoscalingRole, Arn]
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs
```

然后需要创建scale up policy，与serviceScaleTarget关联起来，当需要scale up的时候，会触发这个policy，我们配置的AdjustmentType是`ChangeInCapacity`同时stepAdjustments里面配置了ScalingAdjustment是1，即policy执行时候会增加一个容器任务做到横向service扩展，详细参考[这里](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-applicationautoscaling-scalingpolicy-stepscalingpolicyconfiguration.html#cfn-applicationautoscaling-scalingpolicy-stepscalingpolicyconfiguration-stepadjustments)。

```
ServiceScalingUpPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: ScaleUpPolicy
      PolicyType: StepScaling
      ScalingTargetId: !Ref 'ServiceScalingTarget'
      StepScalingPolicyConfiguration:
        AdjustmentType: ChangeInCapacity
        Cooldown: 300
        MetricAggregationType: Average
        StepAdjustments:
        - MetricIntervalLowerBound: 0
          ScalingAdjustment: 1
```

需要一个alarm来触发这个sale up policy，通过cloudWatch中AWS/ECS 命名空间下面的以ServiceName为维度的指标来触发，当CPU占用率在5分钟内大于60%即上报告警，触发scale up操作。

```
  ECSCPUHighAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      EvaluationPeriods: 5
      Statistic: Average
      Threshold: 60
      AlarmDescription: Alarm if CPU utilization if great than 60
      Period: 60
      AlarmActions:
        - Ref: ServiceScalingUpPolicy
      Namespace: AWS/ECS
      Dimensions:
      - Name: ServiceName
        Value:
          Ref: ECSService
      ComparisonOperator: GreaterThanThreshold
      MetricName: CPUUtilization
```

以上即是在AWS上使用ECS部署应用程序，并配置自动扩展策略的最简单实践。上面3个CloudFormation的模板可以参考[这里](https://github.com/liul85/Autoscale-with-AWS--ECS)

参考：

[https://aws.amazon.com/blogs/compute/automatic-scaling-with-amazon-ecs/](https://aws.amazon.com/blogs/compute/automatic-scaling-with-amazon-ecs/)