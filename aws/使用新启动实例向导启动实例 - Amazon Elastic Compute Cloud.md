## 使用新启动实例向导启动实例

您可以使用新的启动实例向导启动一个实例。启动实例向导指定启动实例所需的启动参数。在启动实例向导提供默认值的情况下，您可以接受默认值或指定自己的值。如果您接受默认值，则可以通过仅选择密钥对来启动实例。

在启动实例之前，请确保您已进行了相应设置。有关更多信息，请参阅[设置以使用 Amazon EC2](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)。

当您启动不在 [AWS 免费套餐](http://aws.amazon.com/free/)范围内的实例时，即使该实例处于闲置状态，您也需为该实例运行的时间付费。

**主题**

-   [关于新启动实例向导](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#about-liw)
-   [快速启动实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-quickly-launch-instance)
-   [使用定义的参数启动实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-launch-instance-with-defined-parameters)
-   [使用旧的启动实例向导启动实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/launching-instance.html)

## 关于新启动实例向导

欢迎使用全新改进后的启动体验，这是一种更快、更简单的实例启动方法。

我们正在推出新的启动实例向导。如果在当前选定的区域中不可用，则可以选择其他区域来检查该区域是否可用。

###### 最新改进

-   单页布局具有摘要侧面板
    
    使用新的单页设计实现快速启动和运行。在同一位置查看所有设置。无需在步骤之间反复导航以确保配置正确。使用 **Summary**（摘要）面板获得概览并轻松导航至页面。
    
-   改进的 AMI 选择器
    
    新用户 – 使用 **Quick Start** Amazon Machine Image（AMI）选择器来选择操作系统，以快速启动实例。
    
    有经验的用户 – AMI 选择器显示您最近使用的 AMI 和您拥有的 AMI，以使您更轻松地选择您关心的 AMI。您仍然可以浏览完整目录以查找新的 AMI。
    

###### 持续改善

我们不断努力，持续改善体验。以下是我们目前正在研究的内容：

-   默认值和依赖项协助
    
    -   **默认值**将运用于所有字段。
        
    -   **其它逻辑**的添加将帮助您正确设置实例配置（例如，我们将禁用当前设置中的不可用参数）。
        
    
-   更加简洁的设计
    
    -   **简洁的视图与摘要**和**响应速度更快的设计**将使一页式体验更具可扩展性。
        
    -   **便捷联网**功能将帮助您快速轻松地配置防火墙规则（例如，选择通用的预设规则）。
        
    

在接下来的几个月里，我们将针对启动体验完成更多改善。

###### 期待您的反馈

非常感谢您对新启动实例向导的反馈。在接下来的几个月里，我们将根据您的反馈继续改善体验。您可以通过 EC2 控制台向我们直接发送反馈，也可以使用此页面底部的**提供反馈**链接进行反馈。

## 快速启动实例

如需快速设置实例以进行测试，请按照以下步骤。您需要选择操作系统和密钥对，然后接受默认值。有关启动实例向导中所有参数的信息，请参阅 [使用定义的参数启动实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-launch-instance-with-defined-parameters)。

###### 快速启动实例

1.  通过以下网址打开 Amazon EC2 控制台：[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)。
    
2.  在屏幕顶部的导航栏中，会显示当前 AWS 区域 \[例如，美国东部（俄亥俄）\]。选择要在其中启动实例的区域。选择该内容是非常重要的，因为可以在区域之间共享某些 Amazon EC2 资源，而无法共享其他资源。有关更多信息，请参阅[资源位置](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/resources.html)。
    
3.  从 Amazon EC2 控制台控制面板中，选择**启动实例**。
    
    如果您看到旧的启动向导，则新的启动实例向导尚未成为当前选定区域中的默认视图。要打开新的启动实例向导，请选择位于屏幕右上角的 **Opt in to the new experience**（选择加入新体验）。
    
4.  （可选）在 **Name and tags**（名称与标签）下，为 **Name**（名称）输入实例的描述性名称。
    
5.  在 **Application and OS Images（Amazon Machine Image）**（应用程序和操作系统镜像（Amazon Machine Image））下，选择 **Quick Start**（快速启动），然后为您的实例选择操作系统（OS）。
    
6.  （可选）在 **Key pair (login)**（密钥对（登录））下，为 **Key pair name**（密钥对名称）选择一个现有密钥对或新建一个密钥对。
    
7.  在 **Summary**（摘要）面板中，选择 **Launch instance**（启动实例）。
    

## 使用定义的参数启动实例

除了密钥对外，启动实例向导还会为所有参数提供默认值。您可以接受任何或全部默认值，也可以通过为每个参数指定自己的值来配置实例。启动实例向导中这些参数进行了分组。以下说明将引导您完成每个参数组。

**实例配置的参数**

-   [发起实例启动](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-initiate-instance-launch)
-   [名称和标签](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-name-and-tags)
-   [应用程序和操作系统镜像 (Amazon Machine Image)](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-ami)
-   [实例类型](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-instance-type)
-   [密钥对（登录）](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-key-pair)
-   [Network settings (网络设置)](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-network-settings)
-   [配置存储](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-storage)
-   [高级详细信息](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-advanced-details)
-   [摘要](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html#liw-summary)

### 发起实例启动

1.  通过以下网址打开 Amazon EC2 控制台：[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)。
    
2.  在屏幕顶部的导航栏中，会显示当前 AWS 区域 \[例如，美国东部（俄亥俄）\]。选择要在其中启动实例的区域。选择该内容是非常重要的，因为可以在区域之间共享某些 Amazon EC2 资源，而无法共享其他资源。有关更多信息，请参阅[资源位置](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/resources.html)。
    
3.  从 Amazon EC2 控制台控制面板中，选择**启动实例**。
    
    如果您看到旧的启动向导，则新的启动实例向导尚未成为当前选定区域中的默认视图。要打开新的启动实例向导，请选择位于屏幕右上角的 **Opt in to the new experience**（选择加入新体验）。
    

### 名称和标签

实例名称是一个标签，其中密钥为 **Name**（名称），而值为您指定的名称。您可以标记实例、卷、弹性图形和网络接口。对于竞价型实例，您只能标记竞价型实例请求。有关标签的信息，请参阅[标记 Amazon EC2 资源](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/Using_Tags.html)。

指定实例名称和其它标签为可选项。

-   对于 **Name**（名称），为实例输入一个描述性名称。如果您没有指定名称，则可以通过其 ID 标识实例，该 ID 将在您启动实例时自动生成。
    
-   要添加其它标签，请选择 **Add additional tags**（添加其它标签）。选择 **Add tag**（添加标签），然后输入密钥和值，然后选择要标记的资源类型。为每个要添加的其它标签选择 **Add tag**（添加标签）。
    

### 应用程序和操作系统镜像 (Amazon Machine Image)

Amazon Machine Image (AMI) 中包含了创建实例所需的信息。例如，AMI 可能包含充当 Web 服务器所需的软件，例如 Linux、Apache 和您的网站。

您可以找到适合的 AMI，如下所示。通过查找 AMI 的每个选项，您可以选择右上角的 **Cancel**（取消）以返回启动实例向导，而不选择 AMI。

**搜索栏**

要搜索所有可用的 AMI，请在 AMI 搜索栏中输入关键字，然后按 **Enter** 键。要选择 AMI，请选择 **Select**（选择）。

**最近使用的项目**

您最近使用的 AMI。

选择 **Recently launched**（最近启动）或 **Currently in use**（当前使用），然后在 **Amazon Machine Image (AMI)** 中，选择一个 AMI。

**我的 AMI**

您拥有的私有 AMI，或与您共享的私有 AMI。

选择 **Owned by me**（我拥有的）或 **Shared with me**（与我共享），然后在 **Amazon Machine Image (AMI)** 中选择一个 AMI。

**Quick Start（快速入门）**

AMI 按操作系统 (OS) 分组以助您快速入门。

首先选择所需的操作系统，然后从 **Amazon Machine Image (AMI)** 中选择一个 AMI。要选择符合免费套餐条件的 AMI，请确保该 AMI 已标记 **Free tier eligible**（符合免费套餐条件）。

**浏览更多 AMI**

选择 **Browse more AMIs**（浏览更多 AMI）以浏览完整的 AMI 目录。

-   要搜索所有可用的 AMI，请在搜索栏中输入关键字，然后按 **Enter** 键。
    
-   要使用 Systems Manager 参数查找 AMI，请选择搜索栏右侧的箭头按钮，然后选择 **Search by Systems Manager parameter**（按 Systems Manager 参数搜索）。有关更多信息，请参阅[使用 Systems Manager 参数查找 AMI](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/finding-an-ami.html#using-systems-manager-parameter-to-find-AMI)。
    
-   要按类别搜索，请依次选择 **Quickstart AMI**、**My AMIs**（我的 AMI）、**AWS Marketplace AMI** 或者 **Community AMIs**（社区 AMI）。
    
    AWS Marketplace 是一个在线商店，您可以从中购买在 AWS 上运行的软件（包括 AMI）。有关从 AWS Marketplace 启动实例的更多信息，请参阅[启动 AWS Marketplace 实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/launch-marketplace-console.html)。在 **Community AMIs**（社区 AMI）中，您可以找到 AWS 社区成员提供给其它人使用的 AMI。将来自 Amazon 或经过验证的合作伙伴的 AMI 标记为**经过验证的提供商**。
    
-   要筛选 AMI 列表，请在屏幕左侧的 **Refine results**（优化结果）下方选中一个或多个复选框。筛选条件选项会因所选搜索类别而有所不同。
    
-   检查对每个 AMI 列出的 **Root device type (根设备类型)**。请注意哪些 AMI 是您需要的类型：即 **ebs**（由 Amazon EBS 支持）或 **instance-store**（由实例存储支持）。有关更多信息，请参阅[根设备存储](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ComponentsAMIs.html#storage-for-the-root-device)。
    
-   检查对每个 AMI 列出的 **Virtualization type (虚拟化类型)**。注意哪些 AMI 类型是您需要的类型：即 **hvm**（全虚拟化）或 **paravirtual**（半虚拟化）。例如，一些实例类型需要 HVM。有关更多信息，请参阅[Linux AMI 虚拟化类型](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/virtualization_types.html)。
    
-   检查对每个 AMI 列出的**启动模式**。请注意哪些 AMI 使用您需要的启动模式：即 **legacy-bios** 或 **uefi**。有关更多信息，请参阅[启动模式](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ami-boot.html)。
    
-   选择满足您的需求的 AMI，然后选择 **Select**。
    

###### 更改 AMI 时发出警告

如果您修改了与所选 AMI 关联的任何卷或安全组的配置，然后选择了其他 AMI，则会打开一个窗口，警告您当前的某些设置将被更改或删除。您可以查看对安全组和卷的更改。此外，您可以查看将添加和删除哪些卷，也可以仅查看要添加的卷。

### 实例类型

实例类型定义了实例的硬件配置和大小。更大的实例类型拥有更多的 CPU 和内存。有关更多信息，请参阅[实例类型](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)。

-   对于 **Instance type**（实例类型），请为实例选择实例类型。
    
    **免费套餐** – 如果您的 AWS 账户的使用时间未满 12 个月，您可以通过选择 **t2.micro** 实例类型（或在 **t2.micro** 不可用时选择“Regions”（区域）中的 **t3.micro** 实例类型）来使用免费套餐下的 Amazon EC2。如果实例类型符合免费套餐条件，则会标记为 **Free tier eligible**（符合免费套餐条件）。有关 t2.micro 和 t3.micro 的更多信息，请参阅 [可突增性能实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/burstable-performance-instances.html)。
    
-   **比较实例类型**：您可以通过以下属性比较不同的实例类型：vCPU 数、架构、内存量 (GiB)、存储量 (GB)、存储类型和网络性能。
    

### 密钥对（登录）

为 **Key pair name**（密钥对名称）选择一个现有密钥对，或选择 **Create new key pair**（创建新密钥对）来新建一个密钥对。有关更多信息，请参阅[Amazon EC2 密钥对和 Linux 实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-key-pairs.html)。

如果您选择 **Proceed without key pair (Not recommended)**（在没有密钥对的情况下继续（不推荐））选项，则将无法连接到此实例，除非您选择配置为允许用户以其它方式登录的 AMI。

### Network settings (网络设置)

根据需要配置网络设置。

-   **Networking platform**（网络平台）：如果适用，选择是将实例启动到 VPC 还是 EC2-Classic。如果选择 **Virtual Private Cloud (VPC)**，则在**网络接口**部分中指定子网。如果选择 **EC2-Classic**，请确保在 EC2-Classic 中支持指定的实例类型，然后为实例指定可用区。请注意，我们将于 2022 年 8 月 15 日停用 EC2-Classic。
    
-   **VPC**：选择要在其中创建安全组的现有 VPC。
    
-   **子网**：您可以在与可用区、本地扩展区、Wavelength 区域或 Outpost 关联的子网中启动实例。
    
    要在可用区中启动实例，请选择要在其中启动实例的子网。要创建新子网，请选择 **Create new subnet** 转到 Amazon VPC 控制台。完成此操作后，返回到启动实例向导并选择“Refresh”（刷新）图标，以便将您的子网加载到列表中。
    
    要在本地区域中启动实例，请选择您在本地区域中创建的子网。
    
    要在 Outpost 中启动实例，请在 VPC 中选择与 Outpost 关联的子网。
    
-   **自动分配公有 IP**：指定您的实例是否会收到公有 IPv4 地址。默认情况下，默认子网中的实例会收到公有 IPv4 地址，而非默认子网中的实例不会收到。可以选择 **Enable (启用)** 或 **Disable (禁用)** 以覆盖子网的默认设置。有关更多信息，请参阅[公有 IPv4 地址](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-public-addresses)。
    
-   **Firewall (security groups)**（防火墙（安全组））：使用安全组为实例定义防火墙规则。这些规则指定哪些传入的网络流量可传输到您的实例。所有其他的流量将被忽略。有关安全组的更多信息，请参阅 [适用于 Linux 实例的 Amazon EC2 安全组](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-security-groups.html)。
    
    添加网络接口时，您必须在网络接口中指定相同安全组。
    
    按如下所示进行选择或创建安全组：
    
    -   要选择现有安全组，请选择 **Select an existing security group**（选择现有安全组），然后从 **Common security groups**（通用安全组）中选择您的安全组。
        
    -   要创建新安全组，请选择 **Create security group**（创建安全组）。启动实例向导会自动定义 **launch-wizard-_x_** 安全组并提供以下复选框以快速添加安全组规则：
        
        **Allow SSH traffic from**（允许入站 SSH 流量）- 创建入站规则以允许您通过 SSH（端口 22）连接至您的实例。指定流量是来自 **Anywhere**（任何地方）、**Custom**（自定义）还是 **My IP**（我的 IP）。
        
        **Allow HTTPs traffic from the internet**（允许来自 Internet 的 HTTPs 流量）- 创建一个打开端口 443（HTTPS）的入站规则，以允许来自任何地方的互联网流量。如果您的实例将是 Web 服务器，则需要此规则。
        
        **Allow HTTP traffic from the internet**（允许来自 Internet 的 HTTP 流量）- 创建一个打开端口 80（HTTP）的入站规则，以允许来自任何地方的互联网流量。如果您的实例将是 Web 服务器，则需要此规则。
        
        您可以根据需要编辑规则并添加规则以适应您的需要。
        
        要编辑或添加规则，请选择位于右上角的 **Edit**（编辑）。要添加规则，请选择 **Add security group rule**（添加安全组规则）。对于 **Type**（类型），请选择网络流量类型。将使用为网络流量打开的协议自动填充 **Protocol**（协议）字段。对于 **Source type**（源类型），请选择一种源类型。若要让启动实例向导添加您电脑的公有 IP 地址，请选择 **My IP**（我的 IP）。但是，如果您在没有静态 IP 地址的情况下通过 ISP 或从防火墙后面进行连接，则您需要了解客户端电脑使用的 IP 地址范围。
        
        如果您要短时启动测试实例并将很快停止或终止，那么允许所有 IP 地址 (`0.0.0.0/0`) 通过 SSH 或 RDP 访问您的实例的规则是可以接受的，但此规则对生产环境来说是不安全的。您应该仅授权特定 IP 地址或特定范围内的 IP 地址访问您的实例。
        
    
-   **Advanced network configuration**（高级网络配置）– 仅在选择子网时可用。
    
    **网络接口**
    
    -   **Device index**（设备索引）：网卡的索引。必须将主网络接口分配给网卡索引 **0**。有些实例类型支持多个网卡。
        
    -   **Network interface**（网络接口）：选择 **New interface**（新接口）即可让 Amazon EC2 创建新的接口，或选择现有且可用的网络接口。
        
    -   **Description (描述)**：（可选）新网络接口的描述。
        
    -   **Subnet**（子网）：要在其中创建新网络接口的子网。对于主网络接口 (`eth0`)，这是在其中启动实例的子网。如果为 `eth0` 输入了现有的网络接口，将在该网络接口所在的子网中启动实例。
        
    -   **安全组**：VPC 中要与网络接口关联的一个或多个安全组。
        
    -   **Primary IP (主要 IP)**：您的子网范围内的一个私有 IPv4 地址。保留空白会让 Amazon EC2 为您选择一个私有 IPv4 地址。
        
    -   **Secondary IP**（辅助 IP）：您的子网范围内的一个或多个其它私有 IPv4 地址。选择 **Manually assign**（手动分配）然后输入 IP 地址。选择 **Add IP**（添加 IP）以添加另一个 IP 地址。或者，选择 **Automatically assign**（自动分配）以使 Amazon EC2 来为您选择，然后输入一个值来指明要添加的 IP 地址的数量。
        
    -   (仅限 IPv6) **IPv6 IP**：子网范围内的一个 IPv6 地址。选择 **Manually assign**（手动分配）然后输入 IP 地址。选择 **Add IP**（添加 IP）以添加另一个 IP 地址。或者，选择 **Automatically assign**（自动分配）以使 Amazon EC2 来为您选择，然后输入一个值来指明要添加的 IP 地址的数量。
        
    -   **IPv4 前缀**：网络接口的 IPv4 前缀。
        
    -   **IPv6 前缀**：网络接口的 IPv6 前缀。
        
    -   **Delete on termination (终止时删除)**：选择在删除实例时是否删除网络接口。
        
    -   **Elastic Fabric Adapter**：指示网络接口是否为 Elastic Fabric Adapter。有关更多信息，请参阅 [Elastic Fabric Adapter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html)。
        
    
    选择 **Add network interface**（添加网络接口）以添加辅助网络接口。辅助网络接口可以与 VPC 位于不同的子网中，但必须位于您的实例所在的可用区内。
    
    有关更多信息，请参阅[弹性网络接口](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-eni.html)。如果指定多个网络接口，则您的实例无法收到公有 IPv4 地址。此外，如果您为 eth0 指定某个现有网络接口，则无法使用 **Auto-assign Public IP** 覆盖子网的公有 IPv4 设置。有关更多信息，请参阅[在实例启动期间分配公有 IPv4 地址](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-instance-addressing.html#public-ip-addresses)。
    

### 配置存储

您选择的 AMI 包含一个或多个存储卷，包括根卷。您可以指定要附加到实例的其它卷。

您可以使用 **Simple**（简单）或 **Advanced**（高级）视图。借助 **Simple**（简单）视图，您可以指定卷的大小和类型。若要指定所有卷参数，请选择位于卡片的右上角的 **Advanced**（高级）视图。

在 **Advanced**（高级）视图中，您可以按如下方式配置每个卷：

-   **Storage type**（存储类型）：选择要与实例关联的 Amazon EBS 或实例存储卷。列表中可用的卷类型取决于您选择的实例类型。有关更多信息，请参阅 [Amazon EC2 实例存储](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/InstanceStorage.html) 和 [Amazon EBS 卷](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-volumes.html)。
    
-   **Device name**（设备名称）：从卷的可用设备名称列表中进行选择。
    
-   **Snapshot**（快照）：选择要从其中还原卷的快照。您可以通过在 **Snapshot**（快照）字段中输入文本来搜索可用的共享快照和公有快照。
    
-   **Size (GiB)**（大小 (GiB)）：对于 EBS 卷，您可以指定存储大小。如果您选择了有资格享用免费套餐的 AMI 和实例，请记住，若要享用免费套餐，您必须将总存储大小保持为 30GiB 以下。有关更多信息，请参阅[针对 EBS 卷的大小和配置的限制](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/volume_constraints.html)。
    
-   **卷类型**：对于 EBS 卷，请选择卷类型。有关更多信息，请参阅[Amazon EBS 卷类型](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-volume-types.html)。
    
-   **IOPS**：如果选择了Provisioned IOPS SSD 卷类型，则可以输入卷支持的每秒 I/O 操作数 (IOPS)。
    
-   **Delete on termination**（终止时删除）：对于 Amazon EBS 卷，选择 **Yes**（是）以在终止实例时删除此卷或选择 **No**（否）以保留此卷。有关更多信息，请参阅[在实例终止时保留 Amazon EBS 卷](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/terminating-instances.html#preserving-volumes-on-termination)。
    
-   **Encrypted**（加密）：如果实例类型支持 EBS 加密，则可以选择 **Yes**（是）以为此卷启用加密。如果默认情况下在此区域中启用了加密，则会为您启用加密。有关更多信息，请参阅[Amazon EBS 加密](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/EBSEncryption.html)。
    
-   **KMS key**（KMS 密钥）：如果您将 **Encrypted**（加密）选择为 **Yes**（是），则您必须选择一个客户托管式密钥来加密卷。如果默认情况下在此区域中启用了加密，则将为您选择默认的客户托管密钥。您可以选择不同的密钥或指定由您创建的任何客户托管密钥的 ARN。
    
-   **File systems**（文件系统）：将 Amazon EFS 或 Amazon FSx 文件系统挂载到实例。有关挂载 Amazon EFS 文件系统的更多信息，请参阅 [将 Amazon EFS 与 Amazon EC2 结合使用](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AmazonEFS.html)。有关挂载 Amazon FSx 文件系统的更多信息，请参阅 [将 Amazon FSx 与 Amazon EC2 结合使用](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/storage_fsx.html)
    

### 高级详细信息

对于**Advanced details (高级详细信息)**，请展开该部分以查看字段并为实例指定任何其他参数。

-   **Purchasing option**（购买选项）：选择 **Request Spot Instances**（请求竞价型实例）即可按照 Spot 价格请求竞价型实例，以按需价格为上限，而选择 **Customize**（自定义）即可更改默认的竞价型实例设置。您可以设置最高价（不建议），并更改请求类型、请求时长和中断行为。如果您未请求竞价型实例，则默认情况下 Amazon EC2 会启动按需型实例。有关更多信息，请参阅[创建竞价型实例请求](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/spot-requests.html#using-spot-instances-request)。
    
-   **域加入目录**：选择您的 Linux 实例在启动后加入到的 AWS Directory Service 目录（域）。如果选择一个域，则必须选择一个具有所需权限的 IAM 角色。有关更多信息，请参阅[将 Linux EC2 实例无缝加入到您的 AWS 托管的 Microsoft AD 目录中](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/seamlessly_join_linux_instance.html)。
    
-   **IAM instance profile**（IAM 实例配置文件）：选择 AWS Identity and Access Management (IAM) 实例配置文件以与实例关联。有关更多信息，请参阅[适用于 Amazon EC2 的 IAM 角色](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)。
    
-   **Hostname type**（主机名类型）：选择实例的来宾操作系统主机名将包括资源名称还是 IP 名称。有关更多信息，请参阅[Amazon EC2 实例主机名类型](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-instance-naming.html)。
    
-   **DNS Hostname**（DNS 主机名）：确定对资源名称或 IP 名称的 DNS 查询（根据您对 **Hostname type**（主机名类型）的选择）是否将以 IPv4 地址（A 记录）、IPv6 地址（AAAA 记录）响应，还是同时以两者响应。有关更多信息，请参阅[Amazon EC2 实例主机名类型](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-instance-naming.html)。
    
-   **Shutdown behavior (关闭行为)**：选择关闭时实例应该停止还是终止。有关更多信息，请参阅[更改实例启动的关闭操作](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingInstanceInitiatedShutdownBehavior)。
    
-   **Stop - Hibernate behavior**（停止 – 休眠行为）：要启用休眠，请选择 **Enable**（启用）。只有当实例满足休眠先决条件时，此字段才可用。有关更多信息，请参阅[休眠 Linux 按需型实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/Hibernate.html)。
    
-   **Termination protection**（终止保护）：要防止意外终止，请选择 **Enable**（启用）。有关更多信息，请参阅[启用终止保护](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingDisableAPITermination)。
    
-   **Stop protection**（停止保护）：为防止意外停止，请选择 **Enable**（启用）。有关更多信息，请参阅[启用停止保护](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/Stop_Start.html#Using_StopProtection)。
    
-   **Detailed CloudWatch monitoring**（详细的 CloudWatch 监控）：选择 **Enable**（启用）将使用 Amazon CloudWatch 开启对您的实例的详细监控。将收取额外费用。有关更多信息，请参阅[使用 CloudWatch 监控您的实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-cloudwatch.html)。
    
-   **Credit specification**（积分规范）：选择 **Unlimited**（无限）以允许应用程序只要有需要即突增到基准以上。此字段仅适用于 **T** 实例。可能收取额外费用。有关更多信息，请参阅[可突增性能实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/burstable-performance-instances.html)。
    
-   **Placement group name (置放群组名称)**：指定要在其中启动实例的置放群组。您可以选择现有置放群组或创建新组。并非所有实例类型都支持在放置群组中启动实例。有关更多信息，请参阅[置放群组](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/placement-groups.html)。
    
-   **EBS-optimized instance**（EBS 优化的实例）：Amazon EBS 优化的实例使用优化的配置堆栈，并为 Amazon EBS I/O 提供额外的专用容量。如果实例类型支持此功能，请选择 **Enable**（启用）来启用该功能。将收取额外费用。有关更多信息，请参阅[Amazon EBS 优化的实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-optimized.html)。
    
-   **Capacity Reservation**（容量预留）：指定是在任何开放容量预留 \[**Open**（开放）\]、特定容量预留 \[**Target by ID**（按 ID 定位）\]，还是容量预留组 \[**Target by group**（按组定位）\] 中启动实例。要指定不使用容量预留，请选择 **None**（无）。有关更多信息，请参阅[在现有 容量预留 中启动实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/capacity-reservations-using.html#capacity-reservations-launch)。
    
-   **Tenancy (租期)**：选择是在共享硬件（**Shared (共享)**）、隔离的专用硬件（**Dedicated (专用)**），还是在 专用主机（**Dedicated host (专用主机)**）上运行您的实例。如果您选择在专用主机上启动实例，则可以指定是否在主机资源组中启动实例，也可以定位特定专用主机。可能收取额外费用。有关更多信息，请参阅 [Dedicated Instances](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/dedicated-instance.html) 和 [Dedicated Hosts](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html)。
    
-   **RAM disk ID**（RAM 磁盘 ID）：\[仅对半虚拟 (PV) AMI 有效\] 为实例选择一个 RAM 磁盘。如果您选择了一个内核，则您可能需要选择带有可支持该内核的驱动程序的某个特定 RAM 磁盘。
    
-   **Kernel ID**（内核 ID）：\[仅对半虚拟 (PV) AMI 有效\] 为实例选择一个内核。
    
-   **Nitro Enclave**：允许您从 Amazon EC2 实例创建隔离的执行环境，称为 Enclave。选择 **Enable**（启用）以启用 AWS Nitro Enclaves 的实例。有关更多信息，请参阅 _AWS Nitro Enclaves 用户指南_中的[什么是 AWS Nitro Enclaves？](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html)
    
-   **许可证配置**：您可以根据指定的许可证配置启动实例，以跟踪您的许可证使用情况。有关更多信息，请参阅《AWS License Manager 用户指南》中的[创建许可证配置](https://docs.aws.amazon.com/license-manager/latest/userguide/create-license-configuration.html)。
    
-   **Metadata accessible (元数据可访问)**：您可以启用或禁用对实例元数据的访问。有关更多信息，请参阅[为新实例配置实例元数据选项](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html)。
    
-   **Metadata transport**（传输元数据）：使实例能够访问本地 IMDSv2 IPv6 地址链接（`fd00:ec2::254`）来检索实例元数据。此选项仅在您将 [基于 Nitro 系统构建的实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances) 启动到[仅限 IPv6 的子网](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-basics)中时可用。有关检索实例元数据的更多信息，请参阅 [检索实例元数据](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)。
    
-   **Metadata version (元数据版本)**：如果您启用对实例元数据的访问，您可以选择在请求实例元数据时要求使用 实例元数据服务版本 2。有关更多信息，请参阅[为新实例配置实例元数据选项](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html)。
    
-   **元数据响应跳数限制**：如果启用实例元数据，则可以为元数据标记设置允许的网络跃点数。有关更多信息，请参阅[为新实例配置实例元数据选项](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/configuring-IMDS-new-instances.html)。
    
-   **Allow tags in metadata**（允许元数据中的标签）：如果选择**Enable**（启用），该实例将允许从其元数据访问其所有标签。如果未指定值，则默认情况下，将不允许对实例元数据中的标签的访问。有关更多信息，请参阅[允许访问实例元数据中的标签](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/Using_Tags.html#allow-access-to-tags-in-IMDS)。
    
-   **User data**：您可以指定用户数据在启动时配置实例或运行配置脚本。有关更多信息，请参阅[启动时在 Linux 实例上运行命令](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/user-data.html)。
    

### 摘要

使用 **Summary**（摘要）面板来指定要启动的实例数、查看实例配置和启动实例。

-   **Number of instances (实例的数量)**：输入要启动的实例的数量。所有实例都将以相同的配置启动。
    
    为确保更快地启动实例，请将大量请求分成较小的批次。例如，创建五个独立的请求批次，每个批次包含 100 个实例启动请求，而不要创建一个包含 500 个实例的启动请求。
    
-   （可选）如果您指定的实例不止一个，为了帮助确保保持正确数量的实例来处理应用程序的需求，您可以选择 **consider EC2 Auto Scaling**（考虑 EC2 Auto Scaling）以创建启动模板和 Auto Scaling 组。Auto Scaling 将根据您的规格来扩展组中的实例数。有关更多信息，请参阅 [Amazon EC2 Auto Scaling 用户指南](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)。
    
    如果 Amazon EC2 Auto Scaling 将某个 Auto Scaling 组中的实例标记为运行状况不佳，则会自动安排该实例进行替换，此时将终止此实例而启动另一个实例，并且您将丢失原始实例上的数据。如果您停止或重新引导实例，或者其他事件将实例标记为运行状况不佳，则此实例将被标记为运行状况不佳。有关更多信息，请参阅 _Amazon EC2 Auto Scaling 用户指南_中的 [Auto Scaling 实例的运行状况检查](https://docs.aws.amazon.com/autoscaling/ec2/userguide/healthcheck.html)。
    
-   查看实例的详细信息并进行必要的更改。您可以在 **Summary**（摘要）面板选择某部分的链接以直接导航到该部分。
    
-   当您准备好启动您的实例时，请选择 **Launch instance**（启动实例）。
    
    如果实例无法启动或状态立即转至 `terminated` 而非 `running`，请参阅 [排查实例启动问题](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/troubleshooting-launch.html)。
    
    （可选）您可以为实例创建账单提醒。在确认屏幕上的 **Next Steps**（下一步）下选择 **Create billing alerts**（创建账单提醒）并按照指示操作。还可以在启动实例后创建账单提醒。有关更多信息，请参阅 _Amazon CloudWatch 用户指南_中的[创建账单告警以监控预估 AWS 费用](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)。