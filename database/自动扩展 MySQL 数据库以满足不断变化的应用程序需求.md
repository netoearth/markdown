## 借助 Amazon Aurora 自动扩展 MySQL 数据库以满足

## 不断变化的应用程序需求

Amazon Aurora 是一种与 MySQL 和 PostgreSQL 兼容的关系数据库，既具有传统企业数据库的性能和可用性，又具有开源数据库的简单性和成本效益。在本教程中，您将了解如何通过添加或删除只读副本来创建 Amazon Aurora 数据库及将其配置为自动扩展，以满足您不断变化的应用程序需求。

本教程不在免费套餐范围内，如果您按照教程中的步骤操作并在教程结束时终止相应资源，所需费用不超过 1 USD。  

## 第 1 步：创建 Aurora 数据库集群

1.1 – 打开浏览器并导航到 [Amazon RDS 控制台](https://console.aws.amazon.com/rds/home)。如果您已有 AWS 账户，请登录控制台。否则，请创建一个新的 AWS 账户，以开始使用。

1.2 – 在右上角，选择要启动 Aurora 数据库集群的区域。  

[![在 Amazon Aurora 中创建数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.2.7492c24552d0be91cbfdfd6d603d6c01f552bff2.png "在 Amazon Aurora 中创建数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.3 – 单击“Amazon Aurora”窗口中的“创建数据库”。  

[![在 Amazon Aurora 中创建数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.3.1.4f1479d5b9cde064538c4f4aaf479ac893135776.png "在 Amazon Aurora 中创建数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![切换到新建数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.3.2.60b5e57cb445029eb29f2f0fd5299debb4267d4d.png "切换到新建数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 引擎选项

1.4 – 在“数据库引擎”中，选择“Amazon Aurora”。  

[![选择 Amazon Aurora 数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.4.2f8f85673625fc52b18604f39998742479ea3ace.png "选择 Amazon Aurora 数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.5 – 在“版本”中，选择“兼容 MySQL 的 Amazon Aurora”。  

[![兼容 MySQL 的 Amazon Aurora](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.5.0b60ac0ef58c35d9f3e85007fbb406971bdb211c.png "兼容 MySQL 的 Amazon Aurora")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.6 – 在“版本”中，请选择您想使用的 Aurora 版本。  

[![选择 Amazon Aurora 数据库版本](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.6.0abf12afd092721466c71329f05ea05ed0204e6c.png "选择 Amazon Aurora 数据库版本")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![使用 Amazon Aurora 创建区域数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.7.e42c6a83a0cabad9ba6f2987fe0195d1fd89abc8.png "使用 Amazon Aurora 创建区域数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 数据库功能

1.8 – 选择“单写入器，多读取器”。  

[![Amazon Aurora 数据库功能](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.8.ea8bbf4a01faeeb836715a25810ff52c43fe957d.png "Amazon Aurora 数据库功能")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![选择生产模板](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.9.84a725ca2fe559ce6c4894bdd2ad4f9d5b45a73b.png "选择生产模板")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 设置

1.10 – 为 Aurora 数据库集群选择一个标识符，例如“database-1”。  

[![为集群选择一个标识符](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.10.5a485df49d06c7aba9d2d442021ac6641c1c1646.png "为集群选择一个标识符")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 数据库实例大小

1.11 – 在“数据库实例大小”下，选择大型实例（以 .large 结尾的选项）。  

[![选择实例大小](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.11.94203fd1e2f0379669adcd53b246cdb04329cc76.png "选择实例大小")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 可用性和持久性

1.12 – 选择“创建 Aurora 副本/读取器”。  

[![创建 Aurora 副本](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.12.06d79ed424a4cf3d50b020079010b25cbafe36a9.png "创建 Aurora 副本")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 连接

1.13 – 选择要在其中创建数据库的 VPC。

请注意，一经创建，数据库便无法迁移到其他 VPC。  

[![选择 VPC](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.13.608f43a3c074e4acf3bd53bac32065fca4cf92db.png "选择 VPC")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![其他连接配置](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.14.026df7ad1a87d03b4065bafe8ecd8b058ea97a6e.png "其他连接配置")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![为子网组选择值](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.15.f39ed5aa5218e63725c5d4f30417ca5b7e6126c2.png "为子网组选择值")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.16 – 在“可公开访问”下，选择“否”。

这意味着您必须从同一个 VPC 中的 EC2 实例连接到数据库。  

[![在“可公开访问”下，选择“否”](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.16.2d834899735d48c9d6a27dfb04d2f91a56091a92.png "在“可公开访问”下，选择“否”")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.17 – 在“VPC 安全组”下，选择“新建”。如果您恰好有一个安全组允许在端口 3306 上传入 TCP 连接，也可以选择该安全组。该安全组将控制您 Aurora 集群的入口。  

[![新建 VPC 安全组](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.17.db3318fecef0dd2e671b3dba82a3944effd26546.png "新建 VPC 安全组")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.18 – 在“新 VPC 安全组名称”中，键入“aurora-tutorial”。  

[![新 VPC 安全组](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.18.2ecfc8089da42026d054a3d65bbd02314ae445c7.png "新 VPC 安全组")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![新建 VPC 安全组](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.19.aaf016afa807091b781412f468f91b21d29cd2b8.png "新建 VPC 安全组")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 其他配置

保留“其他配置”的默认值。

最好启用“删除保护”。如果要在本教程结束时删除数据库，可以不选中该选项。

1.20 – 在“删除保护”下，取消选中“启用删除保护”。  

[![其他配置](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.20.0014228f6dc2f1f0b0b943898cce849008177618.png "其他配置")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 检查并创建

快速检查表单中的所有字段后，您可以继续操作。

1.21 – 单击“创建数据库”。  

[![检查并创建数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.21.d3506eed91281375a5d1688c5a9931b233595083.png "检查并创建数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

在创建实例时，您将看到一个说明如何获取凭证的横幅。此时建议您将凭证保存到某个位置，因为之后您将无法再次查看该密码。

1.22 – 单击“查看凭证详细信息”。  

[![凭证详细信息](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.22.7e7188e5e8154385fd71665ef8679e870a3c3fd6.png "凭证详细信息")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![保存集群终端节点](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.23.15498aff35d62a775db138ee966926fc993c862f.png "保存集群终端节点")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.24 – 关闭凭证详细信息弹出窗口后，单击您创建的数据库的名称。  

[![选择您的数据库](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.24.06eaa25f084a3cb16ef92b3dea1fb053484cee5e.png "选择您的数据库")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

1.25 – 复制写入器和读取器终端节点。您可以将任何读取/写入流量定向到写入器终端节点，但最好是将只读流量定向到读取器终端节点。  

[![复制读取器和写入器终端节点](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-1.25.5cccc084865f5032264100279196a734c60501b1.png "复制读取器和写入器终端节点")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

## 第 2 步：添加扩展策略

Aurora Auto Scaling 可以根据您定义的扩展策略创建和删除副本。当工作负载或与数据库的连接数量突然增加时，Aurora Auto Scaling 可以添加 Aurora 副本。工作负载或连接数量减少后，Aurora Auto Scaling 会删除多余的 Aurora 副本，这样您就不必再消耗额外的容量了。  

[![选择您的集群](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.1.b256ae3ead837d859dffe04cf4f32a67db0ebb91.png "选择您的集群")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

2.2 – 单击“操作”并选择“添加副本自动扩展”。  

[![添加副本自动扩展](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.2.c65d22f5634635aede52b2705193a3cf657bdb35.png "添加副本自动扩展")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 策略详细信息

2.3 – 选择一个策略名称，如“policy-1”。  

[![创建策略名称](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.3.77f919399032bfb488679d46a07d4bbf0e136ec3.png "创建策略名称")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

2.4 – 选择用于自动扩展的指标。

可供使用的目标指标有两个：“Aurora 副本的平均 CPU 利用率”和“Aurora 副本的平均连接数”。Aurora Auto Scaling 会创建和管理 CloudWatch 警报，这些警报根据指标和目标值触发扩展策略并计算扩展调整。扩展策略可按需添加或删除 Aurora 副本，以确保指标接近指定的目标值。

使用何种指标取决于应用程序的架构和工作负载。如果您必须运行 CPU 密集型数据库查询，那么测量 CPU 利用率可能比较适合。如果查询很简单，但是您需要扩展读取和写入，那么测量连接数可能是解决之道。

请注意，扩展策略只能基于一个指标，但是您可以创建多项扩展策略。在本教程中，您可以选择“Aurora 副本的平均连接数”。  

[![选择自动扩展的指标](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.4.f9967bf2725b63cdf064f9d7a8115eefe6efc8c3.png "选择自动扩展的指标")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

2.5 – 在“目标值”部分输入“20”。

这意味着如果连接数达到目标值 20，Aurora Auto Scaling 将添加 Aurora 副本，如果连接数在该目标值以下，它将删除多余的副本。无论任何情况，Aurora Auto Scaling 都只删除其创建的 Aurora副本，而不会删除您创建的 Aurora 副本。  

[![输入目标值](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.5.0d457c5fff6477a275a40c3ac72e27030ff319cd.png "输入目标值")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 集群容量详细信息

2.6 – 在“最小容量”部分输入“1”。  

[![输入最小容量](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.6.bfc7a745ae0e670f6e40979cb4ceb9ed07cb07fe.png "输入最小容量")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

2.7 – 在“最大容量”部分输入“2”。

可在稍后对最小和最大容量数值进行修改。在生产环境中使用哪个数值将取决于您对工作负载、连接数和预算的估计。由 Aurora Auto Scaling 创建的 Aurora 副本与用于主实例的数据库实例类为同一数据库实例类。  

[![输入最大容量](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.7.29e9f928d11ec76ddc4017debd3ca5096a3c3d7c.png "输入最大容量")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 检查并继续

2.8 – 检查字段并单击“添加策略”。  

[![添加策略](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-2.8.2b617950160d7405129db408f63e1be6394dd8ac.png "添加策略")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 第 3 步：修改扩展策略

[![选择您的 Aurora 数据库集群](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-3.1.08e61cae7dbdae7685bc9a610ba85797cca232dd.png "选择您的 Aurora 数据库集群")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![选择日志和事件](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-3.2.05e25e483e9fab58062afaba63730cac8a492e36.png "选择日志和事件")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![选择自动扩展策略](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-3.3.bf680da90f0ca64d1ea606797f2e39d44a369c04.png "选择自动扩展策略")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![编辑自动扩展策略](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-3.4.b44a8db217564493d470de9a20d40f92eb71e4af.png "编辑自动扩展策略")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

### 集群容量详细信息

3.5 – 将最大容量更改为 4。

3.6 – 单击“保存”。  

### 第 5 步：删除集群

为完成此教程，您将学习如何在不再需要 Aurora 数据库集群时将其删除。要删除 Aurora 数据库集群，请转到 [RDS 控制面板](https://console.aws.amazon.com/rds/)，然后按照以下说明进行操作： 

5.2 – 请选择您为本教程创建的 Aurora 数据库集群的读取器实例。  

[![选择读取器实例](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.2.6abdf801ffbac756d2d0015602501d081a929c76.png "选择读取器实例")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![在操作中选择删除](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.3.726f26741c80313e20a0532ab16cb64dca81a011.png "在操作中选择删除")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

5.4 – 系统将提示您确认删除操作。键入“delete me”，然后单击“删除”。  

[![确认删除](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.4.2944e52e255bc360eb433fb7f15615b524c77467.png "确认删除")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

5.5 – 请选择您为本教程创建的 Aurora 数据库集群的写入器实例。  

[![选择写入器实例](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.5.2777b4d432597b55f1bc84ef9648057d50d62e50.png "选择写入器实例")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

[![删除写入器实例](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.6.a1e00b3d3a3a3ec7871f71ca58df8204c24dcca0.png "删除写入器实例")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

系统将询问您是否要创建最终备份。通常情况下建议您创建快照，但在本教程中没有必要这样做。

5.7 – 取消选中“创建最终快照”，然后选中“我确认...”。  

[![取消选中“创建最终快照”](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.7.58f3e3264a5006bbecd53e3742b5d4cc12ae0c26.png "取消选中“创建最终快照”")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

5.8 – 键入“delete me“，然后单击“删除”。

集群的状态将更改为“正在删除”。此时，如果确定不再需要创建的安全组，也可以将其删除。  

[![确认删除](https://d1.awsstatic.com/screenshots/10-min-tut-aurora-autoscaling/autoscaling-5.8.2456aa86b32478263ce3120d039662334a56f998.png "确认删除")](https://aws.amazon.com/cn/getting-started/hands-on/aurora-autoscaling-with-readreplicas/#)

## 恭喜

您已通过 Auto Scaling 创建了 Aurora 数据库集群。您已学习了如何根据应用程序需求，通过添加或删除只读副本来自动调整 Aurora 数据库集群的容量。  

### 本教程对您是否有帮助？

## 建议的后续步骤

### 阅读文档

AWS 对 Internet Explorer 的支持将于 07/31/2022 结束。受支持的浏览器包括 Chrome、Firefox、Edge 和 Safari。 [了解详情 »](https://aws.amazon.com/blogs/aws/heads-up-aws-support-for-internet-explorer-11-is-ending/)