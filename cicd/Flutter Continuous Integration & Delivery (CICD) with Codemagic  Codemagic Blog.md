> **This is a getting started guide on building Flutter apps with Codemagic CI/CD. Updated in August 2021.**

At the [Flutter Live](https://developers.google.com/events/flutter-live/) 2018 conference in London, [Nevercode](https://nevercode.io/) [partnered](https://nevercode.io/blog/press-release-nevercode-partners-with-google-and-launches-a-dedicated-ci-cd-tool-for-flutter-apps/) with Google and launched a dedicated CI/CD solution for [Flutter](https://flutter.io/?gclid=EAIaIQobChMInKCv0svY3wIV7LXtCh0aLgHTEAAYASAAEgLEYPD_BwE) apps – a solution called [Codemagic](https://codemagic.io/). With Codemagic, you can have your **Flutter** apps tested and released with zero configuration and no pain.

We’re the Flutter experts, but we are also so much more. If you want to treat your native apps to the same quality of service, you can now use the `codemagic.yaml` file for configuring and building your native [**Android**](https://blog.codemagic.io/native-android-getting-started-guide-with-codemagic-cicd/) and [**iOS**](https://blog.codemagic.io/native-ios-getting-started-guide-with-codemagic/) apps, as well as [**React Native**](https://blog.codemagic.io/react-native-getting-started-guide-with-codemagic/), [Ionic](https://codemagic.io/ionic-continuous-integration/) and [Cordova](https://codemagic.io/cordova-continuous-integration/) apps. If you want to have more control over the scripts, Codemagic allows developers to run custom scripts and create custom workflows for their apps. Developing high-quality apps fast just got even better!

In this article, you will learn how to build, test and deliver **Flutter** apps on Codemagic.

## Getting started

Make sure that your Flutter project is uploaded to a code hosting platform (like GitHub, Bitbucket or GitLab) using a version control system. To start building your apps using Codemagic CI/CD, first you have to sign up using a [GitHub](https://github.com/), [Bitbucket](https://bitbucket.org/) or [GitLab](https://about.gitlab.com/) account, or via email. Follow the steps below to access your project on Codemagic:

1.  Log in to [Codemagic](https://codemagic.io/signup). If you’re not a user, then sign up:

[Sign up](https://codemagic.io/signup)

2.  On the Applications page, click **Add application**:
    
    ![](https://blog.codemagic.io/uploads/2021/06/add_application.png)
    
3.  Select a Git provider (or select Other if you want to add using the clone URL of a repository) you want to use:
    
    ![](https://blog.codemagic.io/uploads/2021/06/connect_repository.png)
    
4.  Click **Next: Authorize integration** to authorize Codemagic. If you have already authorized your selected Git provider, click **Next: Select repository** instead.
    
    > If you are using **GitHub** as your Git provider then there is one additional step before selecting repository, click **Install GitHub App** to set up the integration. Know more about configuring GitHub App integration [here](https://docs.codemagic.io/getting-started/adding-apps-from-custom-sources/#configuring-the-github-app-integration).
    
5.  Now, select your repository (or add the clone URL if using Others) and select the project type, click **Finish: Add application**:
    
    ![](https://blog.codemagic.io/uploads/2021/06/set_up_application.png)
    
6.  You will be taken to the project settings. The **Workflow Editor** tab will be selected by default.
    
    ![](https://blog.codemagic.io/app_settings_workflow_editor_6922112133693267699_hu00874edb3068227b3d7e131513587642_0_1280x1800_fit_linear_3.png)
    

To set up the build workflow of your Flutter project, you can either use the **workflow editor** or the `codemagic.yaml` file.

## Deep dive into the workflow editor

Let’s take a look at how you can use the workflow editor to configure your build workflow and generate debug builds for your Flutter app.

![](https://blog.codemagic.io/workflow_settings_workflow_editor_4717270967842289050_hu238c5c8529ae1f38d9cc8bf12babedd9_0_1280x1800_fit_linear_3.png)

On the right side, you can find the **Workflow settings** panel, on which you can change the name of the workflow, set the maximum build duration, schedule builds and much more.

You can learn more about some workflow settings from the [documentation](https://docs.codemagic.io/flutter/creating-workflows/).

### Build triggers

In this section, you can automate the starting of your builds on Codemagic by specifying the build triggers. Here, you can configure when the build is triggered (on push, PR or tag creation) and watch for specific branch and tag patterns.

![](https://blog.codemagic.io/uploads/2021/05/build_triggers.png)

> You can learn more about the **build triggers** [here](https://docs.codemagic.io/flutter/automatic-build-triggering/).

### Environment variables

You can store your credentials, API keys or any kind of secret file on Codemagic by specifying them in the **Environment variables** section. Before storing the sensitive keys/files, make sure that you [encrypt](https://docs.codemagic.io/building/encrypting/) them by going to the **Encrypt environment variables** option under the **Configuration as code** section.

There are various Codemagic read-only environment variables. You can learn more about them [here](https://docs.codemagic.io/building/environment-variables/).

![](https://blog.codemagic.io/uploads/2021/05/environment_variables.png)

> Learn more about adding **environment variables** [here](https://docs.codemagic.io/flutter/env-variables/).

### Dependency caching

You can speed up the builds on Codemagic by caching dependencies. To use this, you have to check **Enable dependency caching**. Then you have to specify the path(s) to the dependencies that you want to cache.

![](https://blog.codemagic.io/uploads/2021/05/dependency_caching.png)

> Learn more about **dependency caching** [here](https://docs.codemagic.io/flutter/dependency-caching/).

### Test

In this section, you can configure what type of tests you want to run on your Flutter project. If you are running integration tests, then you can specify the type of device you want to run the tests on. It can be an emulator, a simulator or a physical device (AWS Device Farm).

![](https://blog.codemagic.io/uploads/2021/05/test.png)

> Learn more about testing Flutter apps on Codemagic [here](https://docs.codemagic.io/testing/running-automated-tests/).

### Build

You can configure the Flutter, Xcode and CocoaPods version. Additionally, you can define the root project path and the build mode (debug, release or profile) in this section.

![](https://blog.codemagic.io/uploads/2021/05/build.png)

> Learn more about building Flutter projects on Codemagic [here](https://docs.codemagic.io/flutter/flutter-projects/).

### Distribution

You can set up code signing and publishing for your Flutter app in this section. Codemagic allows you to automatically publish your app to various services, such as **Google Play Store**, **App Store Connect** & **Firebase App Distribution**. To publish Flutter web apps, you can use [Codemagic Static Pages](https://docs.codemagic.io/publishing/publishing-to-codemagic-static-pages/) without any hassle.

> You will have the distribution options based upon the build platforms selected.

![](https://blog.codemagic.io/uploads/2021/08/codemagic_distribution.png)

> You can learn more about the different types of publishing options [here](https://docs.codemagic.io/publishing/publishing-to-app-store/).

### Notifications

In this section, you have the options for setting up [Email and Slack notifications](https://docs.codemagic.io/publishing/email-and-slack-notifications/).

![](https://blog.codemagic.io/uploads/2021/08/codemagic_notifications.png)

### Custom scripts

You can define custom scripts by clicking on any of the “**+**” buttons located before the various sections of the Flutter workflow editor. The custom scripts can be defined in post-clone, pre-test, post-test, pre-build, post-build, pre-publish and post-publish phases.

![](https://blog.codemagic.io/uploads/2021/08/codemagic_pre_build.gif)

You can define custom scripts in any language. (The language can be specified with a shebang line.) For example, if you are using Firebase in your project, you need to decrypt the secret Firebase files before building the app. You can define the following in the **pre-build** script:

```
#!/usr/bin/env sh
set -e # exit on first failed command

echo $ANDROID_FIREBASE_SECRET | base64 --decode > $CM_BUILD_DIR/android/app/google-services.json
echo $IOS_FIREBASE_SECRET | base64 --decode > $CM_BUILD_DIR/ios/Runner/GoogleService-Info.plist
```

> Learn more about custom scripts [here](https://docs.codemagic.io/flutter/custom-scripts/).

## Deep dive into YAML

You can also configure your Flutter workflow using the `codemagic.yaml` file. To use YAML, click on **Switch to YAML configuration**. A dialog box will appear for updating your settings, and whether you want to export the current configuration as codemagic.yaml. Click **Save changes**.

> **NOTE:** The downloaded configuration file may not be an exact match of the settings defined in the Workflow Editor. Adjustment may be necessary.

![](https://blog.codemagic.io/uploads/2021/08/update_workflow_settings.png)

If you have your YAML file in your version control, it should be visible here. Otherwise, you can find links to the documentation for creating your YAML file.

> We recommend going through the article to gain a better understanding, but if you already have experience using the `codemagic.yaml` file, you can get the YAML template for the Flutter project [here](https://github.com/codemagic-ci-cd/codemagic-sample-projects/blob/main/flutter/flutter-yaml-demo-project/codemagic.yaml).

![](https://blog.codemagic.io/codemagic_yaml_config_8213902447309151557_hucfd2d94f9e04a69ece608c1050a224f9_0_1280x1800_fit_linear_3.png)

An example of a workflow for building a debug **Android** app is as follows:

```
workflows:
  simple-android-workflow:
    name: Simple Android Workflow
    instance_type: mac_mini
    max_build_duration: 60
    environment:
      flutter: stable
    scripts:
      - name: Set up debug keystore
        script: |
          rm -f ~/.android/debug.keystore
          keytool -genkeypair \
            -alias androiddebugkey \
            -keypass android \
            -keystore ~/.android/debug.keystore \
            -storepass android \
            -dname 'CN=Android Debug,O=Android,C=US' \
            -keyalg 'RSA' \
            -keysize 2048 \
            -validity 10000          
      - name: Set up local properties
        script: echo "flutter.sdk=$HOME/programs/flutter" > "$CM_BUILD_DIR/android/local.properties"
      - name: Get Flutter packages
        script: flutter packages pub get
      - name: Build debug apk
        script: flutter build apk --debug
    artifacts:
      - build/**/outputs/**/*.apk
      - build/**/outputs/**/*.aab
      - build/**/outputs/**/mapping.txt
      - flutter_drive.log
    publishing:
      email:
        recipients:
          - name@gmail.com
```

An example of a workflow for building a debug **iOS** app is as follows:

```
workflows:
  simple-ios-workflow:
    name: Simple iOS Workflow
    instance_type: mac_mini
    max_build_duration: 60
    environment:
      flutter: stable
      xcode: latest
      cocoapods: default
    scripts:
      - name: Set up local properties
        script: echo "flutter.sdk=$HOME/programs/flutter" > "$CM_BUILD_DIR/android/local.properties"
      - name: Get Flutter packages
        script: flutter packages pub get
      - name: Install Podfiles
        script: find . -name "Podfile" -execdir pod install \;
      - name: Build debug iOS app
        script: flutter build ios --debug --no-codesign
    artifacts:
      - build/**/outputs/**/mapping.txt
      - build/ios/ipa/*.ipa
      - /tmp/xcodebuild_logs/*.log
      - flutter_drive.log
    publishing:
      email:
        recipients:
          - name@gmail.com
```

Let’s take a closer look at the major sections of the `codemagic.yaml` file.

### Environment variables

In the environment variables section, you can define credentials (in [encrypted](https://docs.codemagic.io/building/encrypting/) format) and specify the versions of different tools, like Flutter, Xcode, CocoaPods, etc.

```
environment:
  flutter: stable
  xcode: latest
  cocoapods: default
```

> _**TIPS:**_ For simplicity, instead of defining these environment variable in the **codemagic.yaml** file you can also define these values in the **Application environment variables** section under the **Environment variables** tab. Learn more [here](https://docs.codemagic.io/variables/environment-variable-groups/).
> 
> ![](https://blog.codemagic.io/uploads/2021/08/codemagic_application_env_vars.png)

### Scripts

You can specify the commands to run in this workflow in the `scripts` section.

To get Flutter packages, use this:

```
- name: Get Flutter packages
  script: flutter packages pub get
```

To run Flutter analyze, you can use the following:

```
- name: Flutter analyze
  script: flutter analyze
```

Flutter unit tests can be run using:

```
- name: Flutter unit tests
  script: flutter test
```

Use the following to build the debug version of the app:

```
# Build debug Android .apk
- name: Build debug apk
  script: flutter build apk --debug
# Build debug iOS .app
- name: Build debug iOS app
  script: flutter build ios --debug --no-codesign
```

### Artifacts

You can get the generated **.apk** or **.app** file by adding its path under the `artifacts`. Normally, the path looks like this:

```
# For Android
artifacts:
  - build/**/outputs/**/*.apk
  - build/**/outputs/**/*.aab
  - build/**/outputs/**/mapping.txt
  - flutter_drive.log
# For iOS
artifacts:
  - build/**/outputs/**/mapping.txt
  - build/ios/ipa/*.ipa
  - /tmp/xcodebuild_logs/*.log
  - flutter_drive.log
```

### Publishing

To get a report of the build along with the generated artifacts in your **email**, specify the following:

```
publishing:
  email:
    recipients:
      - name@example.com # enter your email id here
```

> For more information about **publishing**, refer to this [link](https://docs.codemagic.io/publishing-yaml/distribution/).

## Code signing

You have to code sign your Android or iOS/macOS app in order to publish it to Google Play Store or App Store Connect, respectively.

You can follow the steps provided here to set up code signing:

-   [Android project code signing](https://blog.codemagic.io/native-android-getting-started-guide-with-codemagic-cicd/#setting-up-your-android-project-for-a-release-build)
-   [iOS project code signing](https://blog.codemagic.io/native-ios-getting-started-guide-with-codemagic/#setting-up-code-signing-for-ios-apps)
-   [macOS project code signing](https://blog.codemagic.io/macos-desktop-publishing-with-codemagic/#getting-ready-for-code-signing)

If you are using the **workflow editor**, then just go to the **Publish** section and set up code signing for the respective platforms.

If you are using the `codemagic.yaml` file to generate release builds of Flutter projects, you can use the following commands:

```
# For Android release build
- name: Build release APK with Flutter (automatic versioning)
  script: |
        flutter build apk --release --build-name=1.0.0 --build-number=$(($(google-play get-latest-build-number --package-name "$PACKAGE_NAME" --tracks="$GOOGLE_PLAY_TRACK") + 1))
# For iOS release build
- name: Build ipa with Flutter (automatic versioning)
  script: |
    flutter build ipa --release \
    --build-name=1.0.0 \
    --build-number=$(($(app-store-connect get-latest-testflight-build-number "$APP_STORE_ID") + 1)) \
    --export-options-plist=/Users/builder/export_options.plist    
```

> There are a few environment variables used in these commands. You can gain a better understanding of them by following the links to the articles on code signing given above.

## Releasing app

Codemagic allows you to automatically publish your Android or iOS/macOS app to Google Play Store or App Store Connect, respectively.

You can follow the steps for releasing your app given in the articles below:

-   [Releasing Android app to Google Play Store](https://blog.codemagic.io/native-android-getting-started-guide-with-codemagic-cicd/#releasing-to-the-google-play-store)
-   [Releasing iOS app to App Store Connect](https://blog.codemagic.io/native-ios-getting-started-guide-with-codemagic/#releasing-to-app-store-connect)
-   [Releasing macOS app to App Store Connect](https://blog.codemagic.io/macos-desktop-publishing-with-codemagic/)

> Check out the [Codemagic publishing documentation](https://docs.codemagic.io/publishing-yaml/distribution/).

## Building on Codemagic

If you are using the `codemagic.yaml` file to define your build configurations, make sure that you have committed it to the version control system. Otherwise, if you are using the **Flutter workflow editor**, you are ready to proceed with the steps for building.

Follow the steps below to start a build:

1.  Go to your project settings. If you are using the workflow editor, make sure that you have chosen the correct platforms and the build machine on which to run the build. Then, click **Start new build** or **Start your first build** (if it’s the first time).
    
    ![](https://blog.codemagic.io/uploads/2021/08/codemagic_start_new_build.png)
    
2.  Select a workflow, and click on **Start new build**.
    
    ![](https://blog.codemagic.io/uploads/2021/08/codemagic_configure_workflow.png)
    

This will start a new build for your Flutter project.

![](https://blog.codemagic.io/codemagic_flutter_build_8919410771806140932_hu2c35bb2288bb2b2e6e50d0abf64c10b1_0_1280x1800_fit_linear_3.png)

Congratulations! 🎉 Your first Flutter build on Codemagic CI/CD is complete!

> Some information to keep in mind if you are building **desktop** apps:
> 
> -   **macOS apps** can only be built on Mac machines
> -   **Linux apps** can only be built on Linux machines (paid minutes)
> -   **Desktop** and **mobile** apps cannot be built together

## Webhooks

For automatically triggering builds in response to events you need to set up webhooks. You can check out the instructions for setting up webhooks with Codemagic [here](https://docs.codemagic.io/building/webhooks/).

You can get the payload URL required for adding a webhook manually on the **Webhooks** tab inside the application settings. Also, the list of all triggered builds will be available on this page under the **Recent deliveries** section.

![](https://blog.codemagic.io/uploads/2021/08/codemagic_webhooks.png)

## Conclusion

Although **Codemagic** started as an official CI/CD solution dedicated solely to **Flutter** apps, it now welcomes all mobile projects to the fastest CI/CD. With the magic of Codemagic, you can build, test and publish Flutter apps with zero configuration and run builds in controlled environments using custom workflows. If you have a native **Android**, **iOS** or **React Native** app, Codemagic has got your back. Just use the `codemagic.yaml` file, and you are ready to lift off!

Happy building!

### Useful links and references

-   [Official Codemagic documentation](https://docs.codemagic.io/)
    
-   [iOS Continuous Integration & Delivery with Codemagic](https://blog.codemagic.io/native-ios-getting-started-guide-with-codemagic/)
    
-   [Android Continuous Integration & Delivery with Codemagic](https://blog.codemagic.io/native-android-getting-started-guide-with-codemagic-cicd/)
    
-   [How to add Flutter modules to native Android project and test it on Codemagic](https://blog.codemagic.io/flutter-module-android-yaml/)
    
-   [How to add Flutter modules to native iOS project and test it on Codemagic](https://blog.codemagic.io/how-to-add-flutter-modules-to-native-ios-project-and-test-it-on-codemagic/)