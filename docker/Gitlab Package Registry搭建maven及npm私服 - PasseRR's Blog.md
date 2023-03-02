### 配置`gitlab.rb`

```
# 开启package
gitlab_rails['packages_enabled'] = true
# 设置保存路径
gitlab_rails['packages_storage_path'] = "/mnt/packages"
```

### 仓库开启Package Registry

仓库/分组 -> Settings -> General -> Visibility, project features, permissions -> 开启Packages

## 准备工作

gitlab支持每个仓库独立的Package管理，但是便于package查找，按照`maven`和`npm`类别，分别创建两个仓库管理对应的包。

1.  创建一个内部的`package-registry`组
2.  生成一个用于包发布的`Group Access Token`
    
    Settings -> Access Token -> 选择scope中的`api` -> 生成token并复制
    
3.  分别创建`maven-packages`和`npm-packages`两个仓库，并记录对应的项目id

## 配置说明

以下配置中需要替换的地方

| 名称 | 描述 |
| --- | --- |
| **YOUR.GITLAB.COM** | 你的gitlab地址 |
| **YOUR\_SCOPE\_NAME** | npm scope名称 |
| **PROJECT\_ID** | npm对应项目id |
| **YOUR\_ACCESS\_TOKEN** | 对应项目/分组的access\_token |

## maven

### settings.xml配置

```
<server>
    <id>gitlab-maven</id>
    <configuration>
        <httpHeaders>
            <property>
                <name>Private-Token</name>
                <!-- 复制的token -->
                <value>YOUR_ACCESS_TOKEN</value>
            </property>
        </httpHeaders>
    </configuration>
</server>
```

### pom.xml配置

`$PROJECT_ID`对应为创建的maven-packages仓库的项目id

```
<!-- 发布包仓库配置 -->
<distributionManagement>
    <repository>
        <id>gitlab-maven</id>
        <url>http://YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/maven</url>
    </repository>
    <snapshotRepository>
        <id>gitlab-maven</id>
        <url>http://YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/maven</url>
    </snapshotRepository>
</distributionManagement>

<!-- 获取包仓库配置 -->
<repositories>
    <repository>
        <id>gitlab-maven</id>
        <url>http://YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/maven</url>
    </repository>
</repositories>
```

## npm

### registry配置(两种方案)

-   使用`.npmrc`配置文件
    
    ```
    //YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/npm/:_authToken=YOUR_ACCESS_TOKEN
    @YOUR_SCOPE_NAME:registry=http://YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/npm/
    ```
    
-   使用命令行配置
    
    ```
    # 设置所有YOUR_SCOPE_NAME下的包对应的registry的url
    npm config set @YOUR_SCOPE_NAME:registry http://YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/npm/
    # 设置安装包地址
    npm config set -- //YOUR.GITLAB.COM/api/v4/projects/PROJECT_ID/packages/npm/:_authToken YOUR_ACCESS_TOKEN
    ```
    

> 若gitlab上存放了多个scope的依赖，每个scope都需要配置一次registry地址

### package.json示例

```
{
  "name": "@YOUR_SCOPE_NAME/test1",
  "version": "1.0.0",
  "description": "description",
  "main": "index.js",
  "dependencies": {
      // OTHER_SCOPE_NAME需要额外配置
      "@OTHER_SCOPE_NAME/test": "^1.0.0"
  },
  "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "author",
  "license": "ISC"
}
```

### 发布/安装

```
# 发布包
npm publish
# 发布测试版本
npm publish --tag=beta
# 安装包
npm install
```