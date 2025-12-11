---
title: "Flutter开发之路-环境搭建"
date: 2025-12-11T20:31:31+08:00
lastmod:
draft: flase
tags: ['flutter']
categories:
description: ""
---

# 前言
用了很久的 `microsoft todo` 是个很好的待办清单工具，但是有以下缺点：
- 同步速度较慢。
- 没有艾森豪威尔矩阵。
- 不能与 `obsidian` 打通, （其实也有开源的插件，但比较难用）。

故产生了自己开发一个自用 `TODO` 应用的的想法。初步设想有以下功能：
- 首要就是添加待办任务了。但是这个待办任务可以分类，可以有：临时待办、习惯、愿望清单、我的一天。
    - 我的一天。显示日程，主要是显示1天内可以完成的任务。就是说你要在某一天要做的事，如果到了那一天，都会再此分类下显示。
    - 习惯。没有期限、重复性比较强的任务。可设置执行时间，在时间点会提醒你。
    - 愿望清单。没有期限。只是做一个记录。

- 艾森豪威尔矩阵。设置优先级及重要性后显示，只显示本周内需要执行的任务。
- 番茄钟。用于设置专注模式。可将某些专注任务加进来。
- 统计模式。将任务的执行数据统计生成报告。
# 安装环境
本人是 `windows` 系统，在 `vscode` 上开发，故记录在 `win` 环境下安装 `flutter` 的过程。
`flutter` 开发所需要的资源如下：
- Flutter SDK
- Android Studio
- VSCode
- Dart SDK
- Git
- 科学上网

安装步骤：

**第一步**。安装 Git、VSCode。

**第二步**。安装 Flutter SDK。

```bash
git clone https://github.com/flutter/flutter.git -b stable -c http.proxy=http://127.0.0.1:7890
```
下载完毕后，找到下载目录并配置用户环境变量。如
```text
C:\src\flutter\bin
```
键入以下命令。如看到版本号即表示配置成功。
```bash
flutter --version
```
检查 `flutter` 运行时环境
```bash
flutter doctor
```
会打印一些报错，重点关注 `[✗] Android toolchain`。根据提示安装对应工具。

**第三步**。安装 Android Studio。`https://developer.android.com/studio`
安装完毕后，先配置 Android Studio 的 Http 代理。


再配置 `Android Sdk` 用户环境变量。名称为：`ANDROID_HOME`，目录为：`C:\Users\{用户名}\AppData\Local\Android\Sdk`。

再在系统环境变量添加如下：
- `%ANDROID_HOME%\cmdline-tools\latest\bin`
- `%ANDROID_HOME%\platform-tools`

接受 Android 许可证。`flutter doctor --android-licenses`，全部接受。

再检测。还有问题根据提示直接做相应操作即可。
```PowerShell
PS C:\Users> flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.38.4, on Microsoft Windows [版本 10.0.19044.2728], locale zh-CN)
[✓] Windows Version (10 家庭中文版 64 位, 21H2, 2009)
[✓] Android toolchain - develop for Android devices (Android SDK version 28.0.3)
[✗] Chrome - develop for the web (Cannot find Chrome executable at .\Google\Chrome\Application\chrome.exe)
    ! Cannot find Chrome. Try setting CHROME_EXECUTABLE to a Chrome executable.
[✗] Visual Studio - develop Windows apps
    ✗ Visual Studio not installed; this is necessary to develop Windows apps.
      Download at https://visualstudio.microsoft.com/downloads/.
      Please install the "Desktop development with C++" workload, including all of its default components
[✓] Connected device (2 available)
```

**第四步**。打开VS Code，安装 Flutter 和 Dart 插件（作者：Dart Code）。

# 运行 `demo`
先简单认识两个命令：
- `flutter create my_app`。自动生成一个 flutter 项目。
- `flutter run`。运行当前工程。


那么，先创建一个 flutter app, 会自己生成一个简单的app。
```bash
flutter create my_app
```
接着 vscode 打开该工程。**重点来了**。
由于 flutter run 运行时会下载很多包，非常耗时间，所以必须先配置国内源。
需要更改的文件有以下：

1. gradle-wrapper.properties
```ini
# android\gradle\wrapper\gradle-wrapper.properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
# distributionUrl=https\://services.gradle.org/distributions/gradle-8.14-all.zip
distributionUrl=https://mirrors.cloud.tencent.com/gradle/gradle-8.14-all.zip
```
2. build.gradle.kts

```kotlin
// android\build.gradle.kts
allprojects {
    repositories {
        // google()
        // mavenCentral()
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/nexus/content/groups/public") }
        maven { url = uri("https://maven.aliyun.com/nexus/content/repositories/jcenter") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
    }
}

val newBuildDir: Directory =
    rootProject.layout.buildDirectory
        .dir("../../build")
        .get()
rootProject.layout.buildDirectory.value(newBuildDir)

subprojects {
    val newSubprojectBuildDir: Directory = newBuildDir.dir(project.name)
    project.layout.buildDirectory.value(newSubprojectBuildDir)
}
subprojects {
    project.evaluationDependsOn(":app")
}

tasks.register<Delete>("clean") {
    delete(rootProject.layout.buildDirectory)
}
```
3. settings.gradle.kts
```kotlin
// android\settings.gradle.kts
pluginManagement {
    val flutterSdkPath =
        run {
            val properties = java.util.Properties()
            file("local.properties").inputStream().use { properties.load(it) }
            val flutterSdkPath = properties.getProperty("flutter.sdk")
            require(flutterSdkPath != null) { "flutter.sdk not set in local.properties" }
            flutterSdkPath
        }

    includeBuild("$flutterSdkPath/packages/flutter_tools/gradle")

    repositories {
        // google()
        // mavenCentral()
        // gradlePluginPortal()
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/nexus/content/groups/public") }
        maven { url = uri("https://maven.aliyun.com/nexus/content/repositories/jcenter") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
    }
}

plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.11.1" apply false
    id("org.jetbrains.kotlin.android") version "2.2.20" apply false
}

include(":app")
```

接着要创建一个文件夹 `.gradle` 在D盘，如 `"D:\myworkspace\flutter_config\.gradle"`，然后设置一个系统环境变量 `GRADLE_USER_HOME`，值为刚才的路径。再启动 Android Studio 的虚拟机。
最后，运行 `flutter run`。等一会就能看到在模拟器里看见app了。