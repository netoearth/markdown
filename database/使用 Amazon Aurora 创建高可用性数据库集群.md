## 数据库集群

在此教程中，您将了解如何配置 Amazon Aurora 集群来创建高可用性数据库。高可用性数据库由跨多个可用区复制的计算节点组成，可提高读取可扩展性并改善故障转移保护。 

Amazon Aurora 是一种关系数据库服务，具有与 MySQL 和 PostgreSQL 兼容的版本，该服务以极低的价格提供企业级数据库的性能和可用性。对于大多数生产工作负载，您需要设置高可用性数据库。

默认情况下，Amazon Aurora 集群只拥有一个执行读取/写入操作的主计算实例。通过在集群中添加一个或多个 Aurora 副本，您将获得数据库集群的读取可扩展性和高可用性。如果集群中的主实例发生故障，Aurora 会将现有副本自动提升为新的主实例。

一般情况下，您需要在与主实例不同的可用区 (AZ) 创建 Aurora 副本。以此方式，当主可用区的基础设施出现问题时，您的数据库可以将故障快速转移到其他可用区的副本。

在存储层，Aurora 使用会以六种方式在三个可用区间复制您的数据来对其进行保护。然而，如果您没有将 Aurora 副本添加到集群中，当发生故障时，您必须等到 Aurora 为您创建新的替换主实例，此过程可能需要更长时间。

本教程将使用兼容 MySQL 的 Amazon Aurora。您将通过 Amazon RDS 管理控制台创建 Aurora 集群，添加 Aurora 副本，测试故障转移情况，然后再终止教程环境。

本教程不支持免费套餐，如果您按照教程中的步骤操作并在教程结束时终止相应资源，所需费用将不超过 1 USD。  

### 1.注册 AWS

您需要一个 AWS 账户才能按照本教程操作。通过选择“注册 AWS”来注册账户。   

### 第 2 步 - 进入 Amazon RDS 控制台

Amazon Aurora 是与 MySQL 和 PostgreSQL 兼容的关系数据库，专为云而打造。它是 Amazon Relational Database Service ([Amazon RDS](https://aws.amazon.com/cn/rds/?&trk=el_a131L000005usSIQAY&trkCampaign=pac_AWSsite_q2419_tutorial_aurora_cluster&sc_channel=el&sc_campaign=pac_q2-2019_AWS_Aurora_10mintutorial&sc_outcome=PaaS_Digital_Marketing&sc_geo=mult)) 的引擎。在此步骤中，您将进入 Amazon RDS 控制台。

打开 [AWS 管理控制台](https://console.aws.amazon.com/console/home?&trk=el_a131L000005usSIQAY&trkCampaign=pac_AWSsite_q2419_tutorial_aurora_cluster&sc_channel=el&sc_campaign=pac_q2-2019_AWS_Aurora_10mintutorial&sc_outcome=PaaS_Digital_Marketing&sc_geo=mult)，以便使本分步指南处于打开状态。此屏幕加载后，请输入您的用户名和密码以便开始操作。选择**服务** > **RDS** 以进入 RDS 管理控制台。

[![](https://d1.awsstatic.com/screenshots/10-minute-tutorial-document-db/Aurora%20HA%20-%20Step1.8bb1d33cf5074a8925dba8f30f589426b454cf10.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

### 第 3 步 - 创建 Amazon Aurora 集群

在此步骤中，您将创建一个 Amazon Aurora 集群，该集群由一个 Aurora 数据库实例组成。

a.在 Amazon RDS 控制台的右上角，选择要在其中创建数据库实例的[区域](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html?&trk=el_a131L000005usSIQAY&trkCampaign=pac_AWSsite_q2419_tutorial_aurora_cluster&sc_channel=el&sc_campaign=pac_q2-2019_AWS_Aurora_10mintutorial&sc_outcome=PaaS_Digital_Marketing&sc_geo=mult)，然后选择**创建数据库**。  

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3a.524a0b1d32a803a434c8f318111bf522816499db.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

b.在“选择引擎”页面上，选择 **Amazon Aurora**。 然后，选择您想要的版本和**下一步**。  

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3b.f8c7f4f2fec2f9d19d0b90e8e57fd4501e41866b.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

c.  现在将配置您的数据库。保留容量类型和数据库引擎版本的默认设置。在**数据库实例类**上，您将选择数据库实例的计算和内存容量。Amazon Aurora 按实例类型按小时收费，在此教程中，选择 **db.t2.small** (1 vCPU, 2 GiB RAM) 来保持较低的成本。

在多可用区部署下，当 Amazon Aurora 将多可用区部署提供为默认选项时，选择**否**，此教程将带您逐步了解如何在您选择的可用区间创建副本。 

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3c2.414b35ae3285643d7a63da13c3bb483cf9e59263.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

d.  输入您的数据库实例标识符的名称、主用户名和密码。选择**下一步**。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3c.d642acc530932eb9ef77df106c5c6e004d0a476b.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

e.  Amazon RDS 有很多高级配置选项。在此教程中，保留默认配置并选择**创建数据库**。

根据数据库实例类的不同，数据库实例可能需要高达几分钟时间才可用。选择**查看数据库实例详细信息**。 

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3d.9c7f2796e229c28187fab2e17cf78d7403104776.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

f.新的 Aurora 数据库实例会显示在 RDS 控制台上的**数据库**列表中。数据库实例的状态将一直为_正在创建_，直到该实例可供使用且状态更改为_可用_。如果状态在几分钟内未改变，请刷新页面。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-3e.e864cff155dfbff34c62f9a125a47904d92439d0.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

### 第 4 步 - 创建 Aurora 副本以获得高可用性

默认情况下，每个 Amazon Aurora 实例均具有强大的数据保护。您可以通过在一个 AWS 区域内的不同可用区 (AZ) 中添加只读副本来提高计算可用性。对于数据库集群在一个区域内跨越的可用区，最多可以跨这些可用区分布 15 个 Aurora 副本。 

a.  在 Amazon RDS 控制台中，Aurora 集群中的主（写入器）实例将列在**数据库**中。选择实例名称并记下_网络_下的可用区。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4a.088101ee7937cf9637c477f87b1903008cc4a935.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

b.选择集群的单选按钮并通过选择**操作** > **添加读取器**来创建 Aurora 副本。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4b.d7ed45d0da48c9f41790fef549c4669a9dab797d.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

c.从主数据库实例中选择另一个**可用区**。对于**实例规格**，选择与主实例类似的实例类（在本示例中为 _db.t2.small_），因此，当发生故障转移时，我们将不会看到数据库执行情况的任何变化。在**设置**下，为 Aurora 只读副本数据库实例输入唯一的名称。

选择**添加读取器**。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4c.1c3d4a865d778665b717045d9f8a9ec723b127b4.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

d.在数据库列表中，我们看到_读取器_角色正在创建新副本。向右滚动，直到您看到**多可用区**属性，现在，您将看到 _2 个可用区_，表示集群分布在两个可用区中（在计算层）。 

您已成功实现计算层的高可用性。接下来，我们将测试数据库故障转移过程。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4d.1d408a2e2dd6dd0ed238de42ac09df4a890d62f3.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

### 第 5 步 - 设置数据库集群以进行故障转移

要提高高可用性，您可以将 Aurora 副本用作故障转移目标。如果主实例故障，则 Aurora 副本会提升为主实例。副本可用于获得读取可扩展性和可用性。在此步骤中，您将设置用于故障转移的 Aurora 副本优先顺序。

a.  选择您的读取器数据库实例旁的单选按钮，然后选择**修改**。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4-4a.07a0b50dc272c886cfc6c1b6645a016a27abc01f.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

b.  在故障转移期间，Amazon RDS 会将优先级最高（从 0 级开始）的副本提升为新的主实例。在本示例中，我们没有任何现有副本，因此我们将副本设置为最高优先级。在**故障转移**下，选择 _0 级_。

如果同一优先级分层中的 2 个或更多副本出现冲突，Amazon RDS 会将大小相同的副本提升为主实例。

选择**继续**，然后**修改数据库实例**。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-4-4b.0862438410ea15600163e305f10ac7f8c1091fe4.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

### 第 6 步 - 测试数据库故障转移

a.  选择目标实例上的单选按钮。然后选择**操作** > **故障转移**。这将导致副本提升为新的主（或写入器）实例，旧的主（或写入器）实例成为新的只读副本。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-5a.72250cb41d9e6cbbab28e2907761ad5ab8a06646.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-5b.0a699fc63ad9c1fc7d3463ebceaa0365ca148e12.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

c.完成故障转移所需的时间取决于故障转移时的数据库活动量，但通常低于 60 秒。您可以在 **日志与事件** \> **最近的事件**下监控故障转移过程。

在使用终端节点的情况下，故障转移对应用程序是透明的。虽然集群和读取器终端节点被用作数据库的 DNS，但实例连接将保持相同，并且会自动使用新的数据库实例。 

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-5c.f6c90f0bba02174433affea5a0ab7b2bd3926412.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

### 7.终止资源

在此步骤中，您将终止 Aurora 数据库集群和环境。

**重要说明：**终止当前未在使用的资源可降低成本，是最佳实践。不终止资源将产生费用。

a.选择您的 Amazon Aurora 集群名称进行终止，然后单击集群名称以显示所有集群实例的列表。选择读取器角色数据库实例上的单选按钮，然后选择**操作** > **删除**。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-6a.fb00376f2730909e6d078adc70f2164c46060251.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

b.输入短语 _delete me_ 并选择**删除**以确认您的删除。我们将看到“状态”更改为_正在删除_。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-6c.68f9e3f72d83e9f1d41ea2b50184348a7968f980.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

c.使用写入器数据库实例重复步骤 6a-b。作为最佳实践，系统将要求您在删除前拍摄最后一次快照。由于这是您的测试数据库集群，请取消选中**创建最终快照**选项并选择确认。输入 _delete me_，然后选择**删除**。此步骤将删除 Aurora 集群，包括存储和所有的自动化数据库备份。

[![](https://d1.awsstatic.com/tmt/configure-aurora-high-availability/r3/step-6d.f3754e0a0233d5b1259307063714b710d7010e72.png)](https://aws.amazon.com/cn/getting-started/hands-on/create-high-availability-database-cluster/#)

## 恭喜

您已了解如何使用 AWS 管理控制台设置具有高可用性的 Amazon Aurora 数据库集群。现在，您可以将 Amazon Aurora 的高可用性、性能和持久性用于您的关键应用程序。  

### 本教程对您是否有帮助？

### 建议的后续步骤

AWS 对 Internet Explorer 的支持将于 07/31/2022 结束。受支持的浏览器包括 Chrome、Firefox、Edge 和 Safari。 [了解详情 »](https://aws.amazon.com/blogs/aws/heads-up-aws-support-for-internet-explorer-11-is-ending/)