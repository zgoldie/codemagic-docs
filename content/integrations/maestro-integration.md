---
title: Maestro integration
description: How to integrate your workflows with Maestro using codemagic.yaml
weight: 16
---

[**Maestro UI testing framework**](https://maestro.dev/) lets you test your iOS and Android mobile apps using simple to-create test flows that are written in a declarative form using YAML. You can run those flows on Codemagic build machines with `maestro test`, or upload your app and flows to [Maestro Cloud](https://app.maestro.dev/) with `maestro cloud` if you prefer not to manage simulators and emulators in CI. For more information on how to get started with Maestro, please refer to the [Maestro documentation](https://docs.maestro.dev/).

A sample project that shows how to configure Maestro integration is available in our [Sample projects repository](https://github.com/codemagic-ci-cd/codemagic-sample-projects/tree/main/integrations/maestro_sample_project).


## Run tests on Codemagic build machines

After you build a simulator **`.app`** (iOS) or **`.apk`** (Android), install the Maestro CLI, boot a pre-installed simulator or emulator, install the app on it, and run your flows:

{{< highlight yaml "style=paraiso-dark">}}
scripts:
    - name: Install Maestro CLI
      script: |
        curl -Ls "https://get.maestro.mobile.dev" | bash
        export PATH="$PATH:$HOME/.maestro/bin"
        maestro --version
    - name: Run Maestro tests
      script: |
        export PATH="$PATH:$HOME/.maestro/bin"
        APP_PATH=$(ls -d build/ios/Build/Products/Debug-iphonesimulator/*.app | head -n 1)
        SIMULATOR_UDID=$(xcrun simctl list devices available | grep "iPhone" | sed -n 's/.*(\([A-F0-9-]\{36\}\)).*/\1/p' | head -n 1)
        xcrun simctl boot "$SIMULATOR_UDID" || true
        xcrun simctl bootstatus booted -b
        xcrun simctl install booted "$APP_PATH"
        maestro test .maestro/
{{< /highlight >}}

iOS Simulators are [pre-installed on macOS build machines](../specs/versions-macos/). For Android emulator tests, use a [Linux build instance](../specs/versions-linux/) — emulators are not available on macOS Apple Silicon machines.


## Get Maestro Cloud API Key

In order to use Maestro Cloud to run your tests on, you will need to signup here and get the `API Key` from your [console](https://app.maestro.dev/), click on your email, **View API Key** and copy the value.

After getting your `API KEY` you need to add it to your [environment variables](/variables/environment-variable-groups/#storing-sensitive-valuesfiles) in a group named ***maestro*** for example.

#### Configure environment variables

1. Open your Codemagic app settings, and go to the **Environment variables** tab.
2. Enter the desired **_Variable name_**, e.g. `MDEV_API_KEY`.
3. Enter the desired variable value as **_Variable value_**.
4. Enter the variable group name, e.g. **_maestro_**. Click the button to create the group.
5. Make sure the **Secret** option is selected.
6. Click the **Add** button to add the variable.

7. Add the variable group to your `codemagic.yaml` file
{{< highlight yaml "style=paraiso-dark">}}
  environment:
    groups:
      - maestro
{{< /highlight >}}

## Managing Maestro flows
After you have created your YAML tests flows inside the `.maestro` directory, you need to check the directory into your project repository.

## Installing Maestro CLI
Before you use maestro commands, you need first to simply install the CLI on the building machine using this command.
{{< highlight yaml "style=paraiso-dark">}}
scripts:
    - name: Download Maestro
      script: curl -Ls "https://get.maestro.mobile.dev" | bash
{{< /highlight >}}

## Uploading to Maestro Cloud

First, you need to build your **Android (.apk) / iOS (.app)** apps, then use the `maestro cloud` command to test your app.

{{< tabpane >}}
{{< tab header="Android" >}}

{{<markdown>}}
See how to build your native android app [here](../yaml-quick-start/building-a-native-android-app/) or your Flutter app [here](../yaml-quick-start/building-a-flutter-app/).

Add the following script to your `publishing` section:
{{< highlight yaml "style=paraiso-dark">}}
publishing:
    scripts:
    - name: Run tests on Maestro cloud
        script: | 
        export PATH="$PATH":"$HOME/.maestro/bin"
        apkPath="/build/app/outputs/apk/release/app-release.apk"
        maestro cloud \
        --apiKey $MDEV_API_KEY \
        $apkPath \
        .maestro/
{{< /highlight >}}

Don't forget to change the value of the `apkPath` to your actual apk path.
{{</markdown>}}
{{< /tab >}}

{{< tab header="iOS" >}}
{{<markdown>}}
For iOS, you need to upload your x86-compatible Simulator `.app` directory.

Here's the script on how you can build it.
{{< highlight yaml "style=paraiso-dark">}}
  scripts:
    - name: Build unsigned .app
    script: | 
        xcodebuild \
        -workspace "ios/$XCODE_WORKSPACE" \
        -scheme "$XCODE_SCHEME" \
        -configuration "Debug" \
        -sdk iphonesimulator \
        -derivedDataPath ios/output
    - name: Run tests on Maestro cloud
        script: | 
        export PATH="$PATH":"$HOME/.maestro/bin"
        iosAppPath="$ios/output/..."
        maestro cloud \
        --apiKey $MDEV_API_KEY \
        $iosAppPath \
        .maestro/
{{< /highlight >}}

Don't forget to add the environment variables that holds your XCode workspace name under `$XCODE_WORKSPACE` and the Scheme name under `$XCODE_SCHEME`. See the complete sample project [here](https://github.com/codemagic-ci-cd/codemagic-sample-projects/tree/main/integrations/maestro_sample_project/codemagic.yaml).
{{</markdown>}}
{{< /tab >}}
{{< /tabpane >}}

If your Codemagic's build has failed at the Maestro cloud step, then your tests have failed. Otherwise, everything went well and you can check out the build page for more details.
