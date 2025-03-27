---
title: Build for Android
parent: Build ONNX Runtime
nav_order: 5
redirect_from: /docs/how-to/build/android-ios,/docs/build/android-ios
---

# Build ONNX Runtime for Android
{: .no_toc }

Follow the instructions below to build ONNX Runtime for Android.

## Contents
{: .no_toc }

* TOC placeholder
{:toc}

## Prerequisites

The SDK and NDK packages can be installed via Android Studio or the sdkmanager command line tool.

Android Studio is more convenient but a larger installation.
The command line tools are smaller and usage can be scripted, but are  a little more complicated to setup. They also require a Java runtime environment to be available.

Generally, you'll want to use the latest stable NDK version. We'll refer to the version that you use as `<NDK version>` from here on.

Resources:

* [API levels](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html)
* [Android ABIs](https://developer.android.com/ndk/guides/abis)
* [System Images](https://developer.android.com/topic/generic-system-image)

### Android Studio

1. [Install](https://developer.android.com/studio) Android Studio

2. Install any additional SDK Platforms if necessary

   * File->Settings->Appearance & Behavior->System Settings->Android SDK to see what is currently installed
   * Note that the SDK path you need to use as `--android_sdk_path` when building ORT is also on this configuration page
   * Most likely you don't require additional SDK Platform packages as the latest platform can target earlier API levels.

3. Install an NDK version

   * File->Settings->Appearance & Behavior->System Settings->Android SDK
   * 'SDK Tools' tab
      * Select 'Show package details' checkbox at the bottom to see specific versions. By default the latest will be installed which should be fine.
   * The NDK path will be the `ndk/<NDK version>` subdirectory of the SDK path shown

### sdkmanager from command line tools

* If necessary install the Java Runtime Environment and set the JAVA_HOME environment variable to point to it
  * https://www.java.com/en/download/
  * Windows note: You MUST install the 64-bit version (https://www.java.com/en/download/manual.jsp) otherwise sdkmanager will only list x86 packages and the latest NDK is x64 only.
* For sdkmanager to work it needs a certain directory structure. First create the top level directory for the Android infrastructure.
  * in our example we'll call that `.../Android/`
* Download the command line tools from the 'Command line tools only' section towards the bottom of https://developer.android.com/studio
* Create a directory called 'cmdline-tools' under your top level directory
  * giving `.../Android/cmdline-tools`
* Extract the 'tools' directory from the command line tools zip file into this directory
  * giving `.../Android/cmdline-tools/tools`
  * Windows note: preferably extract using 7-zip. If using the built in Windows zip extract tool you will need to fix the directory structure by moving the jar files from `tools\lib\_` up to `tools\lib`
    * See https://stackoverflow.com/questions/27364963/could-not-find-or-load-main-class-com-android-sdkmanager-main
* You should now be able to run Android/cmdline-tools/bin/sdkmanager[.bat] successfully
  * if you see an error about it being unable to save settings and the sdkmanager help text,
      your directory structure is incorrect.
  * see the final steps in this answer to double check: https://stackoverflow.com/a/61176718

* Run `.../Android/cmdline-tools/bin/sdkmanager --list` to see the packages available

* Install the SDK Platform
  * Generally installing the latest is fine. You pick an API level when compiling the code and the latest platform will support many recent API levels e.g.

    ```bash
    sdkmanager --install "platforms;android-29"
    ```

  * This will install into the 'platforms' directory of our top level directory, the `Android` directory in our example
  * The SDK path to use as `--android_sdk_path` when building is this top level directory

* Install the NDK
  * Find the available NDK versions by running `sdkmanager --list`
  * Install
    * install the desired version, e.g., `sdkmanager --install "ndk;<NDK version>"`
    * NDK path in our example with this install would be `.../Android/ndk/<NDK version>`

## Android Build Instructions

### Cross compiling on Windows

The [Ninja](https://ninja-build.org/) generator needs to be used to build on Windows as the Visual Studio generator doesn't support Android.

```powershell
./build.bat --android --android_sdk_path <android sdk path> --android_ndk_path <android ndk path> --android_abi <android abi, e.g., arm64-v8a (default) or armeabi-v7a> --android_api <android api level, e.g., 27 (default)> --cmake_generator Ninja
```

e.g. using the paths from our example

```
./build.bat --android --android_sdk_path .../Android --android_ndk_path .../Android/ndk/<NDK version> --android_abi arm64-v8a --android_api 27 --cmake_generator Ninja
```

### Cross compiling on Linux and macOS

```bash
./build.sh --android --android_sdk_path <android sdk path> --android_ndk_path <android ndk path> --android_abi <android abi, e.g., arm64-v8a (default) or armeabi-v7a> --android_api <android api level, e.g., 27 (default)>
```

### Build Android Archive (AAR)

Android Archive (AAR) files, which can be imported directly in Android Studio, will be generated in your_build_dir/java/build/android/outputs/aar, by using the above building commands with `--build_java`

To build on Windows with `--build_java` enabled you must also:

* set JAVA_HOME to the path to your JDK install
  * this could be the JDK from Android Studio, or a [standalone JDK install](https://www.oracle.com/java/technologies/javase-downloads.html)
  * e.g. Powershell: `$env:JAVA_HOME="C:\Program Files\Java\jdk-15"` CMD: `set JAVA_HOME=C:\Program Files\Java\jdk-15`
* run the build from an admin window
  * the Java build needs permissions to create a symlink, which requires an admin window

#### Note: Proguard rules for R8 minimization Android app builds to work

For Android consumers using the library with R8-minimized builds, currently you need to add the following line to your `proguard-rules.pro` file inside your Android project to use package `com.microsoft.onnxruntime:onnxruntime-android` to avoid runtime crashes:

```
-keep class ai.onnxruntime.** { *; }
```

*Reference to similar issues:

There are similar open issues for this on the [protobuf Github project](https://github.com/protocolbuffers/protobuf/issues/6463) and [R8](https://issuetracker.google.com/issues/144631039).

## Android NNAPI Execution Provider

If you want to use NNAPI Execution Provider on Android, see [NNAPI Execution Provider](../execution-providers/NNAPI-ExecutionProvider).

### Build Instructions

Android NNAPI Execution Provider can be built using building commands in [Android Build instructions](#android-build-instructions) with `--use_nnapi`

## QNN Execution Provider

If your device has a supported Qualcomm Snapdragon SOC, and you want to use QNN Execution Provider on Android, see [QNN Execution Provider](../execution-providers/QNN-ExecutionProvider).

### Build Instructions

Download and install [Qualcomm AI Engine Direct SDK](https://qpm.qualcomm.com/main/tools/details/qualcomm_ai_engine_direct)
QNN Execution Provider can be built using building commands in [Android Build instructions](#android-build-instructions) with `--use_qnn static_lib --qnn_home [QNN_SDK path]`

## Test Android changes using emulator (not applicable for QNN Execution Provider)

See [Testing Android Changes using the Emulator](https://github.com/microsoft/onnxruntime/blob/main/docs/Android_testing.md).

## Building a Custom Android Package

Refer to the documentation for [custom builds](./custom.md). In particular, see the section about the [Android Package](./custom.md#android).
