Appcircle is a mobile CI/CD platform that provides a fully automated environment to manage mobile app lifecycle end-to-end, transforming DevOps to NoOps with the best practices. Flutter framework is an open-source user interface toolkit created by Google. However, Flutter has a great community and, it is way beyond the “Google Flutter” nomenclature.

Flutter app development is done with the Dart language and you can develop iOS, Android, and web apps with Flutter.

We are happy to inform the Flutter app developers that Appcircle supports Flutter Android and Flutter iOS to build and deploy native apps. [Flutter web is also supported and its builds are explained in this post.](https://appcircle.io/blog/guide-to-building-flutter-web-apps-with-appcircle/)

Appcircle supports the full lifecycle of Flutter native mobile apps for all Flutter CI/CD needs. In this article, we will be building and deploying a Flutter app with Appcircle with no need for a Mac or any third-party tools.

_Build, distribute and test your Flutter project quickly with Appcircle_  

The first step is getting started with Appcircle. Just go to [https://appcircle.io/start](https://appcircle.io/start) and register. (Regardless of the login provider you select, you can connect any other provider for the repository connections.)

### Creating a Build Profile for a Flutter Project

In Appcircle, a build profile contains the build workflows and the configuration of an app per target platform. (i.e. Separate for iOS and Android).

To create your first build profile, click on the orange “Add New” button on top right of the screen.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Add-New-Flutter-App.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Add-New-Flutter-App.jpg)

Enter a name for your build profile and select target operating system (iOS or Android) and the target platform as Flutter. If you have two different targets in your project for iOS and Android, you need to create two separate profiles. This allows you to manage separate build workflows for different operating systems.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Select-Platform.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Select-Platform.jpg)

You will see your build profile once it has been created. Click on the build profile to connect to a repository that contains a valid Flutter app.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/New-Build-Profile.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/New-Build-Profile.jpg)

You can connect GitHub, Bitbucket and GitLab repositories to your build profile through oAuth apps. Alternatively, You can connect private repositories through SSH and public repositories directly on GitHub, Bitbucket, GitLab and other compatible git providers.

You can authorize Appcircle to connect to your cloud repository provider account. This will allow you to use auto-build your project with hooks. If you authorize Appcircle to access your repositories, you can select the repository that you want to connect. If you use a private repository using an SSH Key, you need to have an SSH key pair ready and enter your private key to Appcircle so Appcircle can access your repository.

To test drive the Appcircle platform for Flutter app builds, you can also use our sample Flutter App by forking it or adding it as a public repository: [https://github.com/appcircleio/appcircle-sample-flutter](https://github.com/appcircleio/appcircle-sample-flutter)

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Connect-Flutter-Repository.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Connect-Flutter-Repository.jpg)

Appcircle will then pull your branches, commits and other information from your repository.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Project-Details-.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Project-Details-.jpg)

## Flutter Application Build Configuration for iOS and Android

The build configuration has different flows for iOS and Android and the projects are configured on a branch basis. You can have different configurations for different branches and you can build any of your commits (assuming that they are compatible with the current configuration). You can build any app developed with the Flutter platform for iOS and Android regardless of the kit such as Flutter Material or Flutter Cupertino.

### iOS Build Configuration

Click on the gear icon on top right to access build configuration. First step will be the entering project details. You can enter these details manually or click on the Fetch button to retrieve them from your project to detect the correct path for the Xcode project automatically.

You can also select a specific Xcode version if you have certain dependencies or if you want to test your build on a specific version.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/iOS-Flutter-Build-Configuration.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/iOS-Flutter-Build-Configuration.jpg)

### Android Build Configuration

For Flutter Android apps, the fetch operation is not required. You can simply select the build mode (e.g. debug or release) and the output type (APK or Splik APK as AAB).

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Android-Build-Configuration.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Android-Build-Configuration.jpg)

### Build Triggers for Auto Builds

The next section, Triggers, is common for both iOS and Android.

Appcircle allows you to trigger builds manually or automatically using build triggers.

-   On push: Whenever code is pushed to a configured branch, the build is triggered.
-   On a tagged push: Whenever a tagged commit is pushed, the build is triggered for that commit. Commits without any tags are ignored.
-   On push with selective tags: Whenever a commit includes one of typed in tags, the build is triggered. You can specify tags with Unix shell-style wildcards to trigger builds.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Autobuild-configuration.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Autobuild-configuration.jpg)

### Signing Configuration

Appcircle allows you to sign your app for deployment.

Appcircle is unique that it doesn’t require you to generate signing identities with a third-party tool. You can simply create your iOS certificates/profiles and Android keystores within the platform and use them in any project in a centralized manner.

If you have readily available signing identities, you can also upload them once to the Appcircle Signing Identities module and use them centrally in any project. (No need for separate uploads of the same certificate for separate projects.)

-   For more information on iOS certificate and provisioning profile management, please refer to the following document: [https://docs.appcircle.io/signing-identities/ios-certificates-and-provisioning-profiles](https://docs.appcircle.io/signing-identities/ios-certificates-and-provisioning-profiles)
-   For more information on Android keystore management, please refer to the following document: [https://docs.appcircle.io/signing-identities/android-keystores](https://docs.appcircle.io/signing-identities/android-keystores)

For signing iOS apps, press add, select the bundle ID from the first dropdown and then select a compatible provisioning profile (added from the signing identities module) from the second dropdown.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/iOS-Signing-for-Flutter-845x355.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/iOS-Signing-for-Flutter.jpg)

For signing Android apps, simply select a keystore (added from the signing identities module).

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Android-Keystores-for-Flutter.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Android-Keystores-for-Flutter.jpg)

### Distribution (Deployment) Configuration

Appcircle is an end-to-end mobile CI/CD tool and enables you to build and deploy your apps to mobile devices with full automation.

Distribution is a critical step when it comes to test your application on real devices. You may need multiple testers and test groups to download, install and test your application and make sure it works on different devices and operating system versions.

Distribution configuration allows you to set up which testing groups will receive your application after the build is complete. You can manually send your binary file to testers or Appcircle can do this for you.

For more information on how to set up testing groups and deploy your app after the build, please refer to the following document: [https://docs.appcircle.io/distribute/create-or-select-a-distribution-profile](https://docs.appcircle.io/distribute/create-or-select-a-distribution-profile).

You can select a previously created distribution profile or create a new one in this window. Use the top input box to enter a name for the new distribution profile you want to create. Press enter or click on the green + icon on the right to create the distribution profile.

Finally, check Auto Distribute if you want your build to be sent to the distribution profile automatically. If the target profile has autosend configured, your testers will receive the latest version automatically.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Deploy-Flutter-Apps.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Deploy-Flutter-Apps.jpg)

There are three main automation points in Appcircle:

-   Autobuild: builds new pushes automatically
-   Autodistribute: deploys new binaries automatically to the Appcircle Distribute module, where you can use it with the online mobile device simulator/emulator or share with the testers.
-   Autosend: Sends new versions of the apps automatically to the selected testing groups.

### Environment Variables Configuration

The final tab is to add environment variables to the build. For advanced use cases, you can define variables and secrets to be incorporated during the build in the Environment Variables submodule so that you don’t need to store certain keys and configurations within the repository.

## Workflows for Advanced Build Configuration

Appcircle is a simple, but flexible platform. You can make basic changes in a simple user interface with the build configuration and use the workflows for advanced use cases. A workflow is a ladder of steps taken to build your applications. Each step has a different purpose and the workflow can be customized by modifying step parameters and inputs, running custom scripts or re-ordering steps. Workflows allow you to have complete control on your build process and enhance it with third-party services and features.

Also note that the workflows are set up by default, so you don’t need to set them up for builds.

For more information on the workflows, you can refer to the following document: [https://docs.appcircle.io/workflows/why-to-use-workflows](https://docs.appcircle.io/workflows/why-to-use-workflows)

You have a wide variety of options with workflows. For instance, if you want to run a specific Flutter command before or after the build, you can add a custom script step at any point in the workflow. If you want to deploy your app with a third-party service instead of Appcircle, you can configure it with the workflows.

You can find commands like `flutter analyze` and `flutter build` available as a workflow step. Additional parameters such as `flutter build apk` or `flutter version` can be specified within the workflow step configuration. If you want to run additional commands like `flutter doctor` or `flutter upgrade`,  you can add a Custom Script step at any point in the workflow and run any bash or ruby script.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Build-Workflows.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Build-Workflows.jpg)

## Starting a Build and After a Build

To start your first build, just press the start build button – the play button under the actions columns (or push some code to your repo if autobuild is configured.)

You will see the build progress and the log in realtime. Flutter setup is handled with the flutter install command during the build.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-App-Cloud-Build.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-App-Cloud-Build.jpg)

Once your build is complete, you can now download the binary file or deploy it to distribute module manually (if autodistribute is enabled, it will be sent automatically after a successful build).

You can also view or download your build logs anytime.

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Build-Actions.jpg)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Build-Actions.jpg)

## Testing the Flutter App

With Appcircle, you have two options to test your app. You can use the standard Flutter testing workflows with the Flutter Analyze and Flutter Test steps or you can use the Appcircle in-browser mobile device emulator/simulator to run your app for manual testing.

### Using Flutter Analyze and Flutter Test

You can use the `flutter analyze` command to run static code tests and the `flutter test` command to run unit tests. Both are available as workflow components within the Appcircle workflow marketplace and you can easily add them [from the workflow marketplace](https://blog.appcircle.io/article/guide-to-automated-mobile-ci-cd-for-flutter-projects-with-appcircle#workflows-for-advanced-build-configuration) without the need for command-line operations or custom scripts.

Please note that both the Flutter Analyze and the Flutter Test step must be preceded by the Flutter Install step.

The Flutter Analyze step will run the `flutter analyze` command and print the output to the build log.

The Flutter Test step will run the `flutter test` command if there are unit tests present in your project and print the output to the build log. To add unit tests to your Flutter project, please [refer to the official Flutter documentation here](https://flutter.dev/docs/cookbook/testing/unit/introduction).

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Sample-Android-Testing-Workflow.png)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Sample-Android-Testing-Workflow.png)

Sample Android Workflow with the Analyze and Test Steps

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Sample-iOS-Testing-Workflow.png)](https://ac.appcircle.io/wp-content/uploads/2020/06/Flutter-Sample-iOS-Testing-Workflow.png)

Sample iOS Workflow with the Analyze and Test Steps

### Deploying Your App to the In-Browser Emulator

If you set a distribution profile in the distribution configuration, you can deploy your app and share it with the testers or preview them right in your browser in the online Android emulator or the iOS Simulator. (Please note that this does not use the Flutter run command.)

In the Distribute module, click on the selected profile, select your OS on the top left and then select a version. To run that version in the online emulator, just press the “Preview on Device” button on the top right. (Your app must support the x86 architecture for this feature to work.)

[![](https://ac.appcircle.io/wp-content/uploads/2020/06/Online-Mobile-Device-Emulator-for-Flutter.png)](https://ac.appcircle.io/wp-content/uploads/2020/06/Online-Mobile-Device-Emulator-for-Flutter.png)

_With Appcircle, you can manage the CI/CD of your Flutter apps for iOS and Android for free._