本文介绍如何使用云效以ZIP包方式将PHP语言的应用部署至Serverless应用引擎SAE（Serverless App Engine）。

如果您是第一次使用SAE托管应用，需要预先在SAE控制台创建相应的应用。SAE支持代码包部署和镜像部署，当前代码包支持Java的WAR包和JAR包，以及PHP ZIP包。应用的部署方式必须与流水线的配置保持一致。

本文以PHP ZIP包部署为例。 具体操作，请参见[在SAE控制台使用ZIP包部署PHP应用](https://help.aliyun.com/document_detail/155135.htm#task-2426208 "应用开发完成后，您可以将应用部署到进行托管。本文介绍如何在控制台以ZIP包方式部署PHP应用。")。

## 步骤二：在云效创建企业

如果您是第一次使用[云效Flow](https://flow.aliyun.com/my)，则需要在云效上创建您的企业。

1.  登录[云效Flow](https://flow.aliyun.com/my)。
2.  设置企业名称并选择使用规模，单击完成创建。
    
    ![首次登录云效创建企业 ](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4157559951/p130308.png)
    

## 步骤三：在云效创建Codeup代码仓库

1.  登录[云效代码管理Codeup](https://codeup.aliyun.com/)。
2.  在代码库页面，单击右上角的。
3.  在弹出的新建代码库对话框中，输入代码库名称并单击确定。
    
    本文以输入sae-demo-php为例，其余参数保持默认设置。
    
4.  选择以下任一方式提交代码。具体操作，请参见[提交代码](https://thoughts.aliyun.com/sharespace/5e8c37eb546fd9001aee8242/docs/5e8c37e7546fd9001aee81fd)。
    
    -   方式一：在Web端直接修改并提交。
    -   方式二：在本地克隆的代码仓库中提交。
    
    成功提交代码后，代码库页面显示的文件如下所示。![sc_sae_demo_php_codeup](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4653034461/p399295.png)
    

## 步骤四：在云效创建流水线

1.  展开sae-demo-php代码库的左侧导航栏，选择流水线，然后单击创建流水线。
2.  在选择流水线模板对话框，选择模板，然后单击创建。
3.  在流程配置页签，在添加流水线源面板的左侧导航栏选择代码源，配置代码源信息，单击添加。
    
    -   选择代码源：单击Codeup。
    -   代码仓库：选择sae-demo-php。
    -   默认分支：选择master。
    

## 步骤五：在云效部署应用至SAE

本步骤将云效Codeup代码仓库中的业务代码，打包部署到SAE。

1.  在流程配置页签的构建区域，单击PHP构建上传到仓库。![bt_php_build_and_upload_to_repository_during_configuration_process](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6564754461/p400982.png)
2.  在弹出的编辑面板配置相关参数。![sc_edit_PHP_job](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6564754461/p401051.png)
    1.  在任务步骤区域，单击构建物上传右侧的减号图标，然后单击确认。
    2.  单击+添加步骤，在下拉列表中选择。
    3.  展开PHP构建，在构建命令区域输入以下参数。
        
        **说明**
        
        -   可以通过修改PACKAGE\_NAME值，管理不同应用的软件包名避免冲突。
        -   仅支持打包为ZIP格式。如果有更多目录需要打包，可修改zip -r命令添加目录或文件。
        
        ```
        # 安装打包软件
        apt-get update
        apt-get install zip -y
        
        # 定义软件包名称
        PACKAGE_NAME="hello-sae-php.zip"
        
        # 确保构建初始化为空
        rm -rf ${PACKAGE_NAME} || echo "Package not exists. Ignore to cleanup"
        
        # 添加./nginx/与./php到打包文件
        zip -r ${PACKAGE_NAME} ./nginx ./php
        ```
        
    4.  展开构建物上传（EDAS/SAE使用），在上传文件文本框中输入上一步中`PACKAGE_NAME`打包的文件名称，例如hello-sae-php.zip。
    5.  在编辑面板右上角单击关闭图标。
3.  在新阶段区域单击新的任务，选择Serverless（SAE）应用发布。
    
    ![SAE应用发布](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3750234461/p129875.png)
    
4.  在编辑面板，配置任务信息，单击保存并运行。
    
    ![云效镜像SAE部署v2](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3750234461/p399475.png)
    
    配置的任务信息说明如下表所示。
    
     
    | 参数名 | 说明 |
    | --- | --- |
    | 任务名称 | 自定义的任务名称。不可超过20个字符。 |
    | 构建集群 | 可为任务选择不同的[构建集群](https://thoughts.aliyun.com/sharespace/5e86a419546fd9001aee81f2/docs/5ecf17b33da58600236d80c8)。 |
    | 下载流水线源 | 开启下载流水线源后，配置流水线源的源文件将会被下载至对应的工作目录下。 |
    | 选择服务连接 | 选择任务的服务授权，使云效能在SAE上部署应用。
    **说明** 如果您从未连接过，请先单击添加服务连接，根据跳转完成阿里云RAM授权后再进行相应配置。
    
    
    
     |
    | 地域 | 选择步骤一所创建的应用所在地域。 |
    | 命名空间 | 选择步骤一所创建的应用所在命名空间。 |
    | SAE应用 | 选择步骤一所创建的应用。 |
    | 构建产物 | 选择步骤四所创建的标签名称。 |
    | 发布策略 | 可选择分批发布或灰度发布。 |
    | 分批方式 | 可选择手动确认或自动确认。例如，如需在完成第一批发布时先观察发布结果再决定后续操作，则可选择手动确认。 |
    | 灰度台数 | 要执行灰度发布的机器数量。
    
    **说明** 此字段仅在发布策略为灰度发布时显示。
    
    
    
     |
    | 发布批次 | 发布分批的数量。 |
    | 分批等待时间 | 相邻发布批次之间的等待时间。 |
    | 任务插件 | 您可以根据需要配置任务插件来发送流水线通知。
    
    -   钉钉机器人通知插件：具体操作，请参见[钉钉机器人发送群消息](https://help.aliyun.com/document_detail/153691.htm#topic-1861985)。
    -   邮件通知：输入邮件地址，多个地址间使用分号（;）分隔。
    -   Webhook通知插件：具体操作，请参见[使用 Webhook 插件发送通知](https://help.aliyun.com/document_detail/202424.htm#topic-2040201)。
    -   企业微信群通知：具体操作，请参见[企业微信机器人发送群消息](https://help.aliyun.com/document_detail/335104.htm#topic-2129525)。
    -   飞书群通知：具体操作，请参见[飞书机器人发送群消息](https://help.aliyun.com/document_detail/335105.htm#topic-2129527)。
    
     |
    
5.  在运行配置对话框中确认配置信息，单击运行。

## 执行结果

选择以下任一方式验证是否运行成功。

-   方式一
    
    云效开始部署后，默认进入最近运行页签，可查看流水线运行结果。如果运行失败，可通过云效流水线中的日志进行排查，重新保存并执行流水线调试。
    
    ![sc_DevOps_recent_runtime](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/8696934461/p399661.png)
-   方式二
    
    云效显示部署成功后，在SAE控制台查看应用的变更记录，是否产生应用重新部署的变更记录。
    
    ![sc_view_change_records](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/8696934461/p399663.png)