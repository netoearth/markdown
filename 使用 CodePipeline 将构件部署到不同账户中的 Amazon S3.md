_上次更新日期：2022 年 2 月 14 日_

**我想将项目部署到其他账户中的 Amazon Simple Storage Service (Amazon S3) 存储桶。我还想将目标账户设置为对象所有者。有没有办法将 AWS CodePipeline 与 Amazon S3 部署操作提供程序来执行此操作？**

## 解决方法

**注意：**以下示例过程假定如下情况：

-   您有两个账户：开发账户和生产账户。
-   开发账户中的输入存储桶称为 **codepipeline-input-bucket**（启动了版本控制）。
-   开发账户中的默认构件存储桶称为 **codepipeline-us-east-1-0123456789**。
-   生产账户中的输出存储桶称为 **codepipeline-output-bucket**。
-   您正在将开发账户中的构件部署到生产账户中的 S3 存储桶。
-   您假设在生产账户中创建的跨账户角色来部署工件。该角色使生产账户成为对象所有者，而不是开发账户。要向生产账户中的存储桶拥有者提供对开发账户拥有的对象的访问权限，请参阅以下文章：[如何使用 CodePipeline 和预装 ACL 将构件部署到其他 AWS 账户中的 Amazon S3？](https://aws.amazon.com/cn/premiumsupport/knowledge-center/codepipeline-artifacts-s3-canned-acl/)[](https://aws.amazon.com/premiumsupport/knowledge-center/codepipeline-artifacts-s3-canned-acl)

### 创建一个 AWS KMS 密钥以用于开发账户中的 CodePipeline

**重要提示：**您必须使用 AWS Key Management Service (AWS KMS) 客户托管密钥进行跨账户部署。如果未配置密钥，则 CodePipeline 将使用默认加密方式加密对象，这些对象无法使用目标账户中的角色进行解密。

1.    打开开发账户中的 [](https://console.aws.amazon.com/kms/)[AWS KMS 控制台](https://console.aws.amazon.com/kms/home)。

2.    从导航窗格中选择**客户管理的密钥**。

3.    选择**创建密钥**。

4.    对于**密钥类型**，选择**对称密钥**。

5.    展开**高级选项**。

6.    对于**密钥材料来源**，选择 **KMS**。然后，选择 **下一步**。

7.    对于 **Alias**（别名），输入密钥的别名。例如：**s3deploykey**。

8.    选择 **下一步**。此时将打开 **“定义密钥管理权限**” 页面。

9.    在**密钥管理员**部分，选择一个 AWS Identity and Access Management (IAM) 用户或角色作为密钥管理员。

10.选择 **下一步**。此时将打开 **“定义密钥使用权限**” 页面。

11.    在 **Other AWS accounts**（其他 AWS 账户）部分，选择 **Add another AWS account**（添加其他 AWS 账户）。

12.    在出现的文本框中，添加生产帐号的账户 ID。然后，选择 **下一步**。

**注意：**您还可以在 “**此帐户**” 部分中选择现有服务角色。如果选择现有服务角色，请跳过**开发账户中的更新 KMS 使用策略部分中的**步骤。

13.    查看密钥策略。然后选择 **Finish**（完成）。

### 在开发账户中创建 CodePipeline

1.    打开 [CodePipeline 控制台](https://console.aws.amazon.com/codesuite/codepipeline/)。然后，选择**创建管道**。

2.    对于**管道名称**，输入您的管道名称。例如：**crossaccountdeploy**。

**注意：****角色名称**文本框会自动填充服务角色名称 **AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy**。您也可以选择有权访问 KMS 密钥的其他现有服务角色。

3.    展开**高级设置**部分。

4.    对于**构件存储**，请选择**默认位置**。  
**注意：**如果使用案例需要，则可以选择**自定义位置**。

5.    对于**加密密钥**，请选择**客户管理的密钥**。

6.    对于 **KMS 客户托管密钥**，请从列表中选择密钥的别名（本示例为 **s3deploykey**）。然后，选择 **Next**（下一步）。此时将打开 **“添加源阶段**” 页。

7.对于**源提供商**，请选择 **Amazon S3**。

8.    对于**存储桶**，输入您的开发输入 S3 存储桶的名称。例如：**codepipeline-input-bucket**。

**重要提示：**输入存储桶必须启动版本控制才能使用 CodePipeline。

9.    对于 **S3 对象键**，输入 **sample-website.zip**。

**重要提示：**要使用示例 AWS 网站而不是您自己的网站，请参阅[教程：创建一个使用 Amazon S3 作为部署提供商的管道](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-s3deploy.html)。然后，在 **1：将静态网站文件部署到 Amazon S3** 部分的**前提条件**中搜索“示例静态网站”。

10.    对于**更改检测选项**，选择 **Amazon CloudWatch Events（推荐）**。然后，选择 **下一步**。

11.    在**添加构建阶段**页面上，选择**跳过构建阶段**。然后，选择**跳过**。

12.    对于**添加部署阶段**页面上的**部署提供商**，选择 **Amazon S3**。

13.    对于**区域**，选择生产输出 S3 存储桶所在的 AWS 区域。例如：**美国东部（弗吉尼亚北部）**。

**重要提示：**如果生产输出桶的区域与管道的区域不同，则还必须验证以下内容：

-   您正在使用具有多个副本的 [AWS KMS 多区域密钥](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html)。
-   您的管道在两个区域都有构件存储。

14.    对于**存储桶**，输入生产输出 S3 存储桶的名称。例如：**codepipeline-output-bucket**。

15.    选中**在部署前提取文件**复选框。  
**注意：**如果需要，请为**部署路径**输入路径。

16.    选择**下一步**。

17.    选择**创建管道**。管道正在运行，但源阶段失败。出现以下错误：“密钥为 'sample-website.zip' 的对象不存在。”

此文章的**上传示例网站至输入存储桶**部分将向描述如何解决此错误。

### 更新开发账户中的 KMS 使用策略

**重要：**如果您使用的是现有 CodePipeline 服务角色，请跳过本节。

1.    打开开发账户中的 [AWS KMS 控制台](https://console.aws.amazon.com/kms/)。

2.    选择密钥的别名（在本例中为 **s3deploykey**）。

3.    在 **Key users（关键用户）**部分，选择 **Add（添加）**。

4.    在搜索框中，输入服务角色 **AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy**。

5.    选择**添加**。

### 在生产账户中配置跨账户角色

**为角色创建 IAM 策略，该策略向 Amazon S3 授予对生产输出 S3 存储桶的权限**

1.    打开生产账户中的 [IAM 控制台](https://console.aws.amazon.com/iam/)。

2.    在导航窗格中，选择**策略**。然后选择**创建策略**。

3.    选择 **JSON** 选项卡。然后，在 JSON 编辑器中输入以下策略：

**重要提示：**将 **codepipeline-output-bucket** 替换为生产输出 S3 存储桶的名称。

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:Put*"
      ],
      "Resource": [
        "arn:aws:s3:::codepipeline-output-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::codepipeline-output-bucket"
      ]
    }
  ]
}
```

4.    选择 **Review policy**（查看策略）。

5.    在 **Name（名称）**中，为策略输入名称。例如：**outputbucketdeployaccess**。

6.    选择 **创建策略**。

**为授予所需的 KMS 权限的角色创建 IAM 策略**

1.    在 [IAM 控制台](https://console.aws.amazon.com/iam/)中，选择 **Create policy**（创建策略）。

2.    选择 **JSON** 选项卡。然后，在 JSON 编辑器中输入以下策略：

**注意：**请替换您创建的 KMS 密钥的 ARN。请使用开发账户中的构件存储桶名称替换 **codepipeline-us-east-1-0123456789**。

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:DescribeKey",
        "kms:GenerateDataKey*",
        "kms:Encrypt",
        "kms:ReEncrypt*",
        "kms:Decrypt"
      ],
      "Resource": [
        "arn:aws:kms:us-east-1:<dev-account-id>:key/<key id>"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:Get*"
      ],
      "Resource": [
        "arn:aws:s3:::codepipeline-us-east-1-0123456789/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::codepipeline-us-east-1-0123456789"
      ]
    }
  ]
}
```

3.    选择 **Review policy**（查看策略）。

4.    在 **Name（名称）**中，为策略输入名称。例如：**devkmss3access**。

5.    选择 **创建策略**。

**创建一个跨账户角色，开发账户可以代入该角色来部署构件**

1.    打开生产账户中的 [IAM 控制台](https://console.aws.amazon.com/iam/)。

2.    在导航窗格中，选择 **Roles**（角色）。然后选择 **Create role**（创建角色）。

3.    选择**其他 AWS 账户**。

4.    对于**账户 ID**，输入开发账户的 AWS 账户 ID。

5.    选择**下一步: 权限**。

6.    在策略列表中，选择 **outputbucketfullaccess** 和 **devkmss3access**。

7.    选择**下一步: 标签**。

8.    （可选）添加标签，然后选择**下一步：查看**。

9.    对于**角色名称**，输入 **prods3role**。

10.    选择**创建角色**。

11.    在角色列表中，选择 **prods3role**。

12.    选择 **Trust Relationships**（信任关系）。选择 **Edit trust relationship**（编辑信任关系）。

13.    在**策略文档**编辑器中，输入以下策略：

**重要提示：**将 **dev-account-id** 替换为开发账户的 AWS 账户 ID。将 **AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy** 替换为管道的服务角色名称。

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::<dev-account-id>:role/service-role/AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy"
        ]
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```

14.    选择 **Update Trust Policy**（更新信任策略）。

### 更新开发账户中 CodePipeline 构件存储桶的存储桶策略

1.    打开开发账户中的 [Amazon S3 控制台](https://s3.console.aws.amazon.com/s3/)。

2.    在**存储桶名称**列表中，选择开发账户（例如 **codepipeline-us-east-1-0123456789）**中的构件存储桶的名称。

3.    选择**权限**。然后，选择**存储桶策略**。

4.    在文本编辑器中，更新现有策略并纳入以下策略语句：

**重要提示：**为了符合正确的 JSON 格式，请在现有语句之后添加一个逗号。将 **prod-account-id** 替换为您的生产账户的 AWS 账户 ID。请使用构件存储桶名称替换 **codepipeline-us-east-1-0123456789**。

```
{
    "Sid": "",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<prod-account-id>:root"
    },
    "Action": [
        "s3:Get*",
        "s3:Put*"
    ],
    "Resource": "arn:aws:s3:::codepipeline-us-east-1-0123456789/*"
},
{
    "Sid": "",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<prod-account-id>:root"
    },
    "Action": "s3:ListBucket",
    "Resource": "arn:aws:s3:::codepipeline-us-east-1-0123456789"
}
```

5.    选择**保存**。

### 将策略附加到开发账户中的 CodePipeline 服务角色，允许其代入您创建的跨账户角色

1.    打开开发账户中的 [IAM 控制台](https://console.aws.amazon.com/iam/)。

2.    在导航窗格中，选择**策略**。然后选择**创建策略**。

3.    选择 **JSON** 选项卡。然后，在 JSON 编辑器中输入以下策略：

**重要提示：**将 **prod-account-id** 替换为您的生产账户的 AWS 账户 ID。

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": [
      "arn:aws:iam::<prod-account-id>:role/prods3role"
    ]
  }
}
```

4.    选择 **Review policy**（查看策略）。

5.    对于**名称**，输入 **assumeprods3role**。

6.    选择**创建策略**。

7.    在导航窗格中，选择 **Roles**（角色）。然后，为管道选择服务角色的名称（在本示例中，为 **AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy**）。

8.    选择 **Attach Policies**（附加策略）。然后，选择 **assumeprods3role**。

9.    选择 **Attach Policy**（附加策略）。

### 更新您的管道以在开发账户中使用跨账户角色

**请注意：**如果您在运行 AWS Command Line Interface (AWS CLI) 命令时遇到错误，[请确保您使用的是最新版本的 AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)。

1.    通过运行以下 AWS CLI 命令将管道定义检索为名为 **codepipeline.json** 的文件：

**重要：**将 **crossaccountdeploy** 替换为管道的名称。

```
aws codepipeline get-pipeline --name crossaccountdeploy > codepipeline.json
```

```
"roleArn": "arn:aws:iam::your-prod-account id:role/prods3role",
```

**包含跨账户 IAM 角色 ARN 的部署操作示例**

**重要提示：**将 **prod-account-id** 替换为您的生产账户的 AWS 账户 ID。

```
{
  "name": "Deploy",
  "actions": [
    {
      "name": "Deploy",
      "actionTypeId": {
        "category": "Deploy",
        "owner": "AWS",
        "provider": "S3",
        "version": "1"
      },
      "runOrder": 1,
      "configuration": {
        "BucketName": "codepipeline-output-bucket",
        "Extract": "true"
      },
      "outputArtifacts": [],
      "inputArtifacts": [
        {
          "name": "SourceArtifact"
        }
      ],
      "roleArn": "arn:aws:iam::<prod-account-id>:role/prods3role",
      "region": "us-east-1",
      "namespace": "DeployVariables"
    }
  ]
}
```

3.    删除 **codepipeline.json** 文件末尾的元数据部分。

**重要提示：**请务必同时删除元数据部分之前的逗号。

**元数据部分示例**

```
"metadata": {
    "pipelineArn": "arn:aws:codepipeline:us-east-1:<dev-account-id>:crossaccountdeploy",
    "created": 1587527378.629,
    "updated": 1587534327.983
}
```

4.    要更新管道，请运行以下命令：

```
aws codepipeline update-pipeline --cli-input-json file://codepipeline.json
```

### 上传示例网站至输入存储桶

1.    打开开发账户中的 [Amazon S3 控制台](https://s3.console.aws.amazon.com/s3/)。

2.    在**存储桶名称**列表中，选择您的开发输入 S3 存储桶。例如：**codepipeline-input-bucket**。

3.    选择**上传**。然后，选择**添加文件**。

4.    选择之前下载的 **sample-website.zip** 文件。

5.    选择**上传**以运行管道。管道运行时，会出现以下情况：

-   源操作从开发输入 S3 存储桶 (**codepipeline-input-bucket**) 中选择 **sample-website.zip**。然后，源操作会将 zip 文件作为源构件放入开发账户中的构件存储桶 (**codepipeline-us-east-1-0123456789**)。
-   在部署操作中，CodePipeline 服务角色（**AWSCodePipelineServiceRole-us-east-1-crossaccountdeploy**）代入生产账户的跨账户角色（ **prods3role**）。
-   CodePipeline 使用跨账户角色 (**prods3role**) 访问开发账户中的 KMS 密钥和构件存储桶。然后，CodePipeline 会将提取的文件部署到生产账户中的生产输出 S3 存储桶 (**codepipeline-output-bucket**)。

**注意：**生产账户是生产输出 S3 存储桶 (**codepipeline-output-bucket**) 中提取的对象的拥有者。

___

**这篇文章对您有帮助吗？**  

___

**您是否需要账单或技术支持？**  

AWS 对 Internet Explorer 的支持将于 07/31/2022 结束。受支持的浏览器包括 Chrome、Firefox、Edge 和 Safari。 [了解详情 »](https://aws.amazon.com/blogs/aws/heads-up-aws-support-for-internet-explorer-11-is-ending/)