目录

-   [
    
    CI/CD 选择
    
    ](https://flutter.cn/docs/deployment/cd#[[[[cicd]]]]-options)
    -   [
        
        内置 Flutter 的多合一 (All-in-one) 选择：
        
        ](https://flutter.cn/docs/deployment/cd#all-in-one-options-with-built-in-flutter-functionality)
    -   [
        
        使用 Fastlane 与现有工作流程集成
        
        ](https://flutter.cn/docs/deployment/cd#integrating-fastlane-with-existing-workflows)
-   [fastlane](https://flutter.cn/docs/deployment/cd#fastlane)
    -   [
        
        本地设置
        
        ](https://flutter.cn/docs/deployment/cd#local-setup)
    -   [
        
        在本地运行部署
        
        ](https://flutter.cn/docs/deployment/cd#running-deployment-locally)
    -   [
        
        云构建和部署设置
        
        ](https://flutter.cn/docs/deployment/cd#cloud-build-and-deploy-setup)
-   [Xcode Cloud](https://flutter.cn/docs/deployment/cd#xcode-cloud)
    -   [
        
        需要准备的
        
        ](https://flutter.cn/docs/deployment/cd#requirements)
    -   [
        
        自定义构建脚本
        
        ](https://flutter.cn/docs/deployment/cd#custom-build-script)
        -   [
            
            Post-clone 脚本
            
            ](https://flutter.cn/docs/deployment/cd#post-clone-script)
    -   [
        
        工作流配置
        
        ](https://flutter.cn/docs/deployment/cd#workflow-configuration)
        -   [
            
            变更分支
            
            ](https://flutter.cn/docs/deployment/cd#branch-changes)
    -   [
        
        下次构建的构建版本数字
        
        ](https://flutter.cn/docs/deployment/cd#next-build-number)

通过 Flutter 持续交付的最佳实践，确保您的应用程序交付给您的 Beta 版本测试人员并能够频繁予以验证，而无需借助手动工作流程。

## CI/CD 选择

有许多持续集成 (CI) 和持续交付 (CD) 的工具，帮助自动发布你的应用。

### 内置 Flutter 的多合一 (All-in-one) 选择：

-   [Codemagic](https://blog.codemagic.io/getting-started-with-codemagic/)
-   [Bitrise](https://devcenter.bitrise.io/getting-started/getting-started-with-flutter-apps/)
-   [Appcircle](https://appcircle.io/blog/guide-to-automated-mobile-ci-cd-for-flutter-projects-with-appcircle/)

### 使用 Fastlane 与现有工作流程集成

你可以通过下面的工具使用 fastlane：

-   [GitHub Actions](https://github.com/features/actions)
    -   样例：Flutter Galley 的 [GitHub Actions 工作流](https://github.com/flutter/gallery/tree/main/.github/workflows)
        
    -   样例：[适用于 Flutter 项目的 GitHub Actions](https://github.com/nabilnalakath/flutter-githubaction)
        
-   [Cirrus](https://cirrus-ci.org/)
-   [Travis](https://travis-ci.org/)
-   [GitLab](https://docs.gitlab.com/ee/ci/README.html#doc-nav)
-   [CircleCI](https://circleci.com/)
    -   [Building and deploying Flutter apps with Fastlane](https://circleci.com/blog/deploy-flutter-android)

这份指南展示了如何让设置 fastlane 以及将其集成到现有应用的测试和持续集成 (CI) 工作流当中去。更多相关的内容，请参考上面这部分的内容。

## fastlane

[fastlane](https://docs.fastlane.tools/) 是一个开源工具套件，帮助你自动的打包正式版以及部署你的应用。

### 本地设置

建议在迁移到基于云计算的系统之前，先在本地测试其构建和部署流程。您还可以使用本地机器执行连续交付。

1.  安装 fastlane `gem install fastlane` 或 `brew install fastlane`。访问 [fastlane docs](https://docs.fastlane.tools/) 以获得更多信息。
    
2.  创建一个名为 `FLUTTER_ROOT` 的环境变量，并将其设置为 Flutter SDK 的根目录。（这是为 iOS 部署的脚本所必需的。）
    
3.  创建您的 Flutter 项目，准备就绪后，确保通过如下途径构建项目：
    
4.  初始化各平台的 fastlane 项目：
    
    -   ![Android](https://flutter.cn/docs/assets/images/docs/cd/android.png)：在 `[project]/android` 目录中，运行 `fastlane init` 命令。
        
    -   ![iOS](https://flutter.cn/docs/assets/images/docs/cd/ios.png)：在 `[project]/ios` 目录下，运行 `fastlane init` 命令。
        
5.  编辑 `Appfile` 以确保它有应用程序的基本数据配置：
    
    -   ![Android](https://flutter.cn/docs/assets/images/docs/cd/android.png) 检查在 `[project]/android/fastlane/Appfile` 文件中的 `package_name` 是否匹配在 AndroidManifest.xml 中的包名。
        
    -   ![iOS](https://flutter.cn/docs/assets/images/docs/cd/ios.png) 检查在 `[project]/ios/fastlane/Appfile` 中的 `app_identifier` 是否匹配 Info.plist 文件中的 bundle identifier。将相应的 `apple_id`、`itc_team_id` 和 `team_id` 输入进去。
        
6.  设置应用商店的本地登录凭据。
    
    -   ![Android](https://flutter.cn/docs/assets/images/docs/cd/android.png) 按照 [Supply setup steps](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply) 文档操作，并且确保 `fastlane supply init` 成功同步了你在 Google Play 商店控制台中的数据。 **.json 文件与密码一样重要，切勿将其公开在任何公共源代码控制存储库。**
        
    -   ![iOS](https://flutter.cn/docs//assets/images/docs/cd/ios.png) iTunes Connect 用户名已经存在于您的 `Appfile` 的 `apple_id` 字段中，你需要将你的 iTunes 密码设置到 `FASTLANE_PASSWORD` 这个环境变量里。否则，上传到 iTunes/TestFlight时会提示你。
        
7.  设置代码签名：
    
    -   ![Android](https://flutter.cn/docs/assets/images/docs/cd/android.png) 参考文档 [为应用签名](https://flutter.cn/docs/deployment/android#signing-the-app)。
        
    -   ![iOS](https://flutter.cn/docs/assets/images/docs/cd/ios.png) 在iOS上，当您准备使用 TestFlight 或 App Store 进行测试和部署时，使用分发证书而不是开发证书进行创建和签名。
        
        -   在 [Apple Developer Account console](https://developer.apple.com/account/ios/certificate/) 创建并下载一个分发证书。
            
        -   打开 `[project]/ios/Runner.xcworkspace/` 在你的项目设置里选择一个分发证书。
            
8.  给每个不同的平台创建一个 `Fastfile` 脚本。
    
    -   ![Android](https://flutter.cn/docs/assets/images/docs/cd/android.png) 在 Android 上按照 [fastlane Android beta deployment guide](https://docs.fastlane.tools/getting-started/android/beta-deployment/) 指引操作。你可以简单的编辑一下文件，加一个名叫 `upload_to_play_store` 的 `lane`。为了使用 `flutter build` 命令编译 `aab`，要把 `apk` 参数设置为 `../build/app/outputs/bundle/release/app-release.aab`。
        
    -   ![iOS](https://flutter.cn/docs/assets/images/docs/cd/ios.png) 在 iOS 上，按照 [fastlane iOS beta 部署指南](https://docs.fastlane.tools/getting-started/ios/beta-deployment/) 指引操作。你可以指定 archive 的路径以避免重复构建。例如：
        
        ```
        build_app(
          skip_build_archive: true,
          archive_path: "../build/ios/archive/Runner.xcarchive",
        )
        upload_to_testflight
        ```
        

你现在已准备好在本地执行部署或将部署过程迁移到持续集成（CI）系统。

### 在本地运行部署

1.  构建发布模式的应用：
    
2.  在每个平台上运行 Fastfile 脚本。
    

### 云构建和部署设置

首先，按照“本地设置”中描述的本地设置部分，确保在迁移到 Travis 等云系统之前，该过程有效。

需要考虑的主要事项是，由于云实例是短暂且不可信的，因此你不能在服务器上保留你的凭据，如 Play Store 服务帐户 JSON 或 iTunes 分发证书。

持续集成 (CI) 系统通常支持加密的环境变量来存储私有数据。你可以使用 `--dart-define MY_VAR=MY_VALUE` 在构建应用时传递环境变量。

**采取预防措施，不要在测试脚本中将这些变量值重新回显到控制台。** 在合并之前，这些变量在拉取请求中也不可用，以确保恶意行为者无法创建打印这些密钥的拉取请求。在接受和合并的 pull 请求中，请注意与这些密钥。

1.  暂时性登录凭据。
    
    -   !\[Android\](https://flutter.cn/docs/assets/images/docs/cd/android.png 在 Android 上：
        
        -   从 `Appfile` 中删除 `json_key_file` 并将其存储在 CI 系统的加密变量里。从 `Fastfile` 中直接读取这些环境变量。
            
            ```
            upload_to_play_store(
              ...
              json_key_data: ENV['<variable name>']
            )
            ```
            
        -   序列化您的上传密钥（例如，使用 base64）并将其另存为加密环境变量。可以可以在安装阶段在 CI 系统上对其进行反序列化
            
            ```
            echo "$PLAY_STORE_UPLOAD_KEY" | base64 --decode > [path to your upload keystore]
            ```
            
    -   ![iOS](https://flutter.cn/docs/assets/images/docs/cd/ios.png) 在 iOS 上:
        
        -   将本地环境变量 `FASTLANE_PASSWORD` 转而使用 CI 系统的加密的环境变量。
            
        -   CI 系统需要有权限拿到你的分发证书。建议使用fastlane 的 [Match](https://docs.fastlane.tools/actions/match/) 系统在不同的机器上同步你的证书。
            
2.  建议每次使用 Gemfile 而不是 `gem install fastlane` 以避免其在 CI 系统上使用的不确定性，以确保 fastlane 依赖关系在本地和云计算机之间稳定且可重现。但是，此步骤是可选的。
    
    -   在 `[project]/android` 和 `[project]/ios` 文件夹中，创建一个 `Gemfile` 包含以下内容：
        
        ```
        source "https://rubygems.org"
        gem "fastlane"
        ```
        
    -   在两个目录中，运行 `bundle update` 并将两者的 `Gemfile` 和 `Gemfile.lock` 文件纳入源代码管理。
        
    -   当你在本地运行的时候,请使用 `bundle exec fastlane` 而不是 `fastlane`。
        
3.  在你的仓库根目录创建一个 CI 测试脚本，例如: `.travis.yml` 或 `.cirrus.yml`。
    
    -   有关特定于 CI 的设置，请参见 [fastlane CI 文档](https://docs.fastlane.tools/best-practices/continuous-integration)。
        
    -   分开你的脚本以便能在 Linux 和 macOS 两个平台运行。
        
    -   在 CI 的设置阶段，执行下列内容：
        
        -   通过执行 `gem install bundler` 确保 Bundler 可用。
            
        -   在 `[project]/android` 或 `[project]/ios` 目录下分别运行 `bundle install`命令。
            
        -   确保 Flutter SDK 已经正确了设置在了 `PATH` 环境变量中。
            
        -   在 Android 平台上，请确保已经设置正确的 `ANDROID_SDK_ROOT` 环境变量。
            
        -   在 iOS 平台上，你需要为 Xcode 指定依赖 (比如: `osx_image: xcode9.2`)
            
    -   在 CI 任务的脚本阶段：
        
        -   根据平台的不同可以运行 `flutter build appbundle` 或者 `flutter build ios --release --no-codesign`。
            
        -   然后执行 `cd android` 或 `cd ios` 命令。
            
        -   最后执行 `bundle exec fastlane [name of the lane]` 命令。
            

## Xcode Cloud

[Xcode Cloud](https://developer.apple.com/xcode-cloud) 是一项为分发 Apple 平台的持续构建集成并交付，以及测试分发的服务。

### 需要准备的

-   Xcode 13.4.1 或更高版本。
    
-   加入 [Apple 开发者计划](https://developer.apple.com/programs)。
    

### 自定义构建脚本

Xcode Cloud 可以识别 [自定义的构建脚本](https://developer.apple.com/documentation/xcode/writing-custom-build-scripts) 用于在特定阶段执行额外的任务。同时它还包含了一系列 [预定义的环境变量](https://developer.apple.com/documentation/xcode/environment-variable-reference)，例如用于你 clone 仓库地址的 `$CI_WORKSPACE`。

#### Post-clone 脚本

利用 post-clone 运行的自定义构建脚本 Xcode Cloud 按照以下说明克隆你的 Git 仓库：

在 `ios/ci_scripts/ci_post_clone.sh` 创建一个文件，然后添加下面的内容。

```
#!/bin/sh

# The default execution directory of this script is the ci_scripts directory.
cd $CI_WORKSPACE # change working directory to the root of your cloned repo.

# Install Flutter using git.
git clone https://github.com/flutter/flutter.git --depth 1 -b stable $HOME/flutter
export PATH="$PATH:$HOME/flutter/bin"

# Install Flutter artifacts for iOS (--ios), or macOS (--macos) platforms.
flutter precache --ios

# Install Flutter dependencies.
flutter pub get

# Install CocoaPods using Homebrew.
HOMEBREW_NO_AUTO_UPDATE=1 # disable homebrew's automatic updates.
brew install cocoapods

# Install CocoaPods dependencies.
cd ios && pod install # run `pod install` in the `ios` directory.

exit 0
```

该文件需要加入 git 仓库管理，并给予可执行权限。

```
$ git add --chmod=+x ios/ci_scripts/ci_post_clone.sh
```

### 工作流配置

[Xcode Cloud workflow](https://developer.apple.com/documentation/xcode/xcode-cloud-workflow-reference) 定义了你工作流触发时 CI/CD 处理进程的执行步骤。

要在 Xcode 中创建一个工作流，请参考以下步骤：

1.  选择 **Product > Xcode Cloud > Create Workflow** 以打开 **Create Workflow** 菜单。
    
2.  选择工作流需要作用的生产应用，然后点击 **Next** 按钮。
    
3.  下一步，菜单将会展示一个 Xcode 提供的默认工作流的浮层，然后可以通过点击 **Edit Workflow** 按钮进行定制。
    

#### 变更分支

默认 Xcode 建议每次分支变更后都为你仓库的默认分支开始一个全新的构建。

对于你应用的 iOS 变体，你通常会希望 Xcode Cloud 在对你的 Flutter packages 修改了 `lib\` 中的 Dart 或 `ios\` 中的 iOS 源文件目录之后，触发你的工作流。

这可以通过使用下列文件和文件夹条件来实现：

![Xcode Workflow Branch Changes](https://flutter.cn/docs/assets/images/docs/releaseguide/xcode_workflow_branch_changes.png)

### 下次构建的构建版本数字

Xcode Cloud 对于新的工作流来说默认的构建版本数字是 `1`，然后在每次成功构建后递增。如果你已经在一个已有应用中，使用了一个更高的构建版本数字，你需要配置 Xcode Cloud 使用正确的构建版本数字，只需要简单通过指定 `Next Build Number` 用于迭代即可。

你可以在 [设置 Xcode Cloud 构建下一次的构建版本数字](https://developer.apple.com/documentation/xcode/setting-the-next-build-number-for-xcode-cloud-builds#Set-the-next-build-number-to-a-custom-value) 查看更多信息。