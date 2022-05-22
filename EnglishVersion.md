

# Shrink Apk Analyzer

[EnglishVersion](EnglishVersion.md)

### Porting apkanalyzer designed for PC in Android SDK to Android

`${SDK_HOME}/tools/bin/apkanalyzer` provides access to axml files (eg AndroidManifest.xml, res/layout/*.xml), which is suitable for situations where we only want to dump a single axml.

Comparison of various tools:

| platform | tool | performance |
| ---- | ---- | ---- |
| PC | JEB | Too heavy, requires GUI, does not provide functionality to handle a single axml |
| PC | apktool | Too heavy to provide functionality to handle a single axml |
| PC | AXMLPrinter2 | Tool from 2008, too old |
| PC | aapt/aapt2 | cannot convert to xml format |
| PC | apkanalyzer | **Very good, meets requirements** |
| Android | drozer | The effect is average, there are escape bugs, and there are incomplete display bugs |
| Android | dumpsys package | Can only print part of the information, cannot convert to xml format |

To sum up, there is apkanalyzer on PC, but there is a lack of a useful tool on Android.

In this project, port apkanalyzer to Android (remove some functions that depend on `aapt` and `dexlib`), and print the AndroidManifest.xml in the mobile phone in the most official and elegant way.

# Advantages and applicable scenarios

- Runs directly on the phone, no need to `adb pull` files to the computer
- Works better than `drozer` (replaces `app.package.manifest` in drozer)

Scenario 1: Dump the manifest of all apps in the phone
Scenario 2: Print the manifest of an APP

# usage

### CLI

````
adb shell
export CLASSPATH=`pm path com.leadroyal.shrink.analyzer | cut -d: -f2`
app_process /system/bin com.android.tools.apk.analyzer.ApkAnalyzerCli manifest print com.android.shell
app_process /system/bin com.android.tools.apk.analyzer.ApkAnalyzerCli manifest print /data/local/tmp/1.apk
````


### MainActivity demo

Of course, you can also call the API directly. Clicking the `FloatingActionButton` will execute all the instructions.

# Known bugs

- Due to the java8 feature `Arrays.stream` in the source code, at least Android N is supported
- Since `java.nio.Path` is used in the source code, it supports at least Android O

# accomplish

1. apkanalyzer source code

````
git clone https://android.googlesource.com/platform/tools/base
git checkout studio-4.0.0
````

There are mainly the following projects:

- `apkparser/cli`
- `apkparser/analyzer`
- `apkparser/binary-resources`
- `common`
- `sdk-common`
- `layoutlib-api`


2. Compatible with Android

Extract the main files from each lib, add functions, and delete useless APIs.

Make the following additions

| Changes | Reason | Location |
| ---- | ---- | ----|
| Support input package name | By default, only input apk path is supported, it is too troublesome to obtain the path in the shell | com.android.tools.apk.analyzer.ApkAnalyzerCli |

Mainly make the following patches

| Changes | Reason | Location |
| ---- | ---- | ----|
| Depends on zipfs.jar | Android does not support ZipFileSystem | com.android.tools.apk.analyzer.internal.ZipArchive |
| Remove aapt related code | Android does not have aapt | com.android.tools.apk.analyzer.ApkAnalyzerCli |
| Remove SaxFactory's defense code about XXE | Android will throw an exception when setting these features | com.android.ide.common.xml.AndroidManifestParser |
| Remove AndroidX related code in SdkConstants | Depends on kotlin, difficult to compile | com.android.SDKConstant |
| Remove the "android.support.design.widget" string in SdkConstants | When checking the lib, the compilation fails | com.android.SDKConstant |


Most of the dependencies have been implanted through source code, and currently only rely on

- zipfs.jar
- com.android.tools.apkparser:binary-resources
- net.sf.jopt-simple:jopt-simple
- com.google.guava:guava

