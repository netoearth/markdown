## 如何对 Amazon EC2 实例上 CodeDeploy 部署失败问题进行故障排查？

_上次更新日期：2022 年 12 月 22 日_

**我在 Amazon Elastic Compute Cloud（Amazon EC2）实例上部署 AWS CodeDeploy 失败。**

## 简短描述

您可以使用 AWS Systems Manager **AWSSupport-TroubleshootCodeDeploy** [Automation 运行手册](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-documents.html)来排查部署失败问题。运行手册可帮助您确定由于以下原因导致部署失败的时间：

-   CodeDeploy 代理尚未安装或未在实例上运行。
-   缺少所需的实例配置文件。
-   实例配置文件没有相应的 Amazon Simple Storage Service（Amazon S3）权限。
-   CodeDeploy 管理的其中一个生命周期挂钩存在问题，例如 **AllowTraffic** 或 **BlockTraffic**。
-   其中一个客户管理的生命周期挂钩存在问题。
-   在部署期间，Auto Scaling 组缩减事件出现了问题。
-   AppSpec 文件丢失或格式不正确。

## 解决方法

**重要提示：**在 CodeDeploy 应用程序所在的同一 AWS 区域中使用 **AWSSupport-TroubleshootCodeDeploy** 运行手册。

1.    打开 [AWS Systems Manager 控制台](https://console.aws.amazon.com/systems-manager/)。

2.    在导航窗格的 **Change Management**（更改管理）部分中，选择 **Automation**（自动化）。

3.    选择 **Execute automation**（执行自动化）。

4.    在 **Owned by Amazon**（由 Amazon 所拥有）选项卡的 **Automation document**（Automation 文档）搜索框中，输入 **AWSSupport-TroubleshootCodeDeploy**。然后，选择搜索图标或按键盘上的 Enter 键。

5.    选择 **AWSSupport-TroubleshootCodeDeploy** 卡片上的单选按钮。

**注意：**确保选择单选按钮而不是超链接的 Automation 名称。

6.    在 **Document details**（文档详细信息）部分，选择 **Next**（下一步）。

7.    在 **Input parameters**（输入参数）部分，对于 **DeplomentID**，请输入失败的部署 ID。

8.    对于 **InstanceID**，请输入部署失败的实例 ID。

9.    对于 **AutomationAssumeRole**，请输入允许 Systems Manager Automation 执行操作的角色的 [Amazon Resource Name（ARN）](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)。

**注意：**如果没有指定 AWS Identity and Access Management（IAM）角色，Systems Manager Automation 将使用运行运行手册的 IAM 用户角色的权限。有关为 Systems Manager Automation 创建所担任角色的详细信息，请参阅[任务 1：为 Automation 创建服务角色](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-setup-iam.html#create-service-role)。

**重要信息：****AutomationAssumeRole** 或用户角色必须具有执行以下操作的权限：[codedeploy:GetDeployment](https://docs.aws.amazon.com/codedeploy/latest/APIReference/API_GetDeployment.html)、[codedeploy:GetDeploymentTarget](https://docs.aws.amazon.com/codedeploy/latest/APIReference/API_GetDeploymentTarget.html) 和 [ec2:DescribeInstances](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html)。

10.    选择 **Execute**（执行）。

运行手册的输出提供了故障排查步骤，以及有关如何解决导致部署失败的问题的建议。

___

**这篇文章对您有帮助吗？**

___

**您是否需要账单或技术支持？**

AWS 对 Internet Explorer 的支持将于 07/31/2022 结束。受支持的浏览器包括 Chrome、Firefox、Edge 和 Safari。 [了解详情 »](https://aws.amazon.com/blogs/aws/heads-up-aws-support-for-internet-explorer-11-is-ending/)