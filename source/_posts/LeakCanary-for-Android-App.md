---
title: 为 Android 软件接入 LeakCanary 实现内存泄漏检测
date: 2016-08-22 14:04:20
categories:
- Test
tags:
- Test
- LeakCanary
---

LeakCanary 是一款检测 Android 内存泄漏的开源类库，GitHub 地址为：[https://github.com/square/leakcanary](https://github.com/square/leakcanary)。LeakCanary 的方便之处在于，只需要在 Android 软件代码中做一点微小的改动，就可以实现内存泄漏的检测。甚至对于测试人员来说，即使你并不是特别熟悉代码（但起码懂一点），也可以做到对软件的接入。

<!-- more -->

# 说在前面

我是因为工作原因接触到 LeakCanary，因为负责公司各产品的 LeakCanary 接入，因此对如何接入和接入中可能遇到的问题有一定的理解。我在做 LeakCanary 接入前，并没怎么接触过 Java 和 Android 软件打包，但有一定代码基础（C/C++、Python、Shell 脚本）。如果你对代码一窍不通，那么在做接入时可能会遇到一些困难。

本文主要介绍 LeakCanary 的接入方法，和可能遇到的各种坑。不对 LeakCanary 的原理和如何修复发现的内存泄漏问题做解答，请见谅。

# 如何接入 LeakCanary

## 官方教程

官方教程说的很简单，主要是两步：

1、在你软件的 `build.gradle`（一般情况，此处的 `build.gradle` 不是根目录下的那个，而且软件文件夹下的，通常是 app 文件夹；但也有一些软件结构比较特殊，根目录下的 `build.gradle` 就是此处需要添加的）中，添加 dependencies：

``` gradle
dependencies {
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4-beta2'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4-beta2'
    testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4-beta2'
}
```

2、在你软件的入口类（通常都继承了 Application 类，一般可以在 `AndroidManifest.xml` 中 `<application android:name="xxx"...` 中找到）的 OnCreate 函数下添加 `LeakCanary.install(this);`，即：

``` java
public class ExampleApplication extends Application {
    @Override public void onCreate() {
        super.onCreate();
        LeakCanary.install(this);
    }
}
```

完成了这两步，打包后安装后，你会发现多了一个叫 Leaks 的软件，图标如下：

![](https://github.com/square/leakcanary/raw/master/assets/icon_512.png)

其包名和你软件的包名一致，这样最简单的 LeakCanary 接入就完成了。

## 更进一步

按照官方教程接入 LeakCanary 后，当出现内存泄漏问题后，下拉通知里会多出一条形如 `xxxActivity leaked xx KB` 的通知，点击后进入该条泄漏信息的详细页面，如下图所示：

![](http://i1.buimg.com/567571/022ce389c36ca4d6.png)

但是，更多时候，我们希望泄漏的信息能够直接上传到数据库，这样就更方便做后续的处理。好在 LeakCanary 已经提供了一个方法，就是继承 `DisplayLeakService` 类。

**1. 创建一个 `LeakUploadService` 类，位置一般在软件入口类所在的包下，让其继承 `DisplayLeakService` 类：**

``` java
public class LeakUploadService extends DisplayLeakService {
    @Override
    protected void afterDefaultHandling(HeapDump heapDump, AnalysisResult result, String leakInfo) {
        if (!result.leakFound || result.excludedLeak){
            return;
        }
        // 下面是上传到数据库的代码
    }
}
```

其中，leakInfo 为泄漏信息，形如：

```
In com.example.leakcanary:1.0:1.
* com.example.leakcanary.MainActivity has leaked:
* GC ROOT thread java.lang.Thread.<Java Local> (named 'AsyncTask #3')
* references com.example.leakcanary.MainActivity$2.this$0 (anonymous subclass of android.os.AsyncTask)
* leaks com.example.leakcanary.MainActivity instance

* Retaining: 131 KB.
* Reference Key: 53591da9-6668-423c-90d1-ff83a797d94a
* Device: HTC htc HTC M9w himauhl_htccn_chs_2
* Android Version: 6.0 API: 23 LeakCanary: 1.4-SNAPSHOT 44787a1
* Durations: watch=5140ms, gc=172ms, heap dump=1381ms, analysis=18835ms

* Details:
...
```

你可以将整个 leakInfo 全部上传到数据库，也可以先对其做一些处理。比如我在上传前，会先处理出泄漏的类名、软件包名、软件版本和泄漏信息。当然，你也可以处理出泄漏的大小等等。

对于上面的泄漏信息，通过以下处理：

``` java
String className = result.className.toString();
String pkgName = leakInfo.trim().split(":")[0].split(" ")[1]
String pkgVer = leakInfo.trim().split(":")[1]
String leakDetail = leakInfo.split("\n\n")[0] + "\n\n" + leakInfo.split("\n\n")[1];
```

可以得到 `className` 为 `com.example.leakcanary.MainActivity`、`pkgName` 为 `com.example.leakcanary`、`pkgVer` 为 `1.0`、`leakDetail` 为 `* Details:` 以上的部分。

同时，为了排除重复泄漏数据的干扰，我们还设置了一个 leakKey。但是，关于这个 leakKey 的算法，至今还在不断改进中。最初负责这块的同事认为，同一个类泄漏的内容都一样，所以是将 `className.hashCode() & 0x7FFFFFFF` 作为 leakKey。后来发现，同一个类泄漏的内容，GC ROOT 可能完全不同，而且同一 GC ROOT 下的 leak trace 可能也有很大不同，当时也咨询了研发，证实了此事。因此当时将 leakKey 的算法改成了：

``` java
String[] infoDetailArray = leakInfo.trim().split("\n\n")[0].split("\n");
String infoDetail = "";
for (int i = 1; i < infoDetailArray.length; i++) {
    infoDetail += infoDetailArray[i];
}
Integer leakKey = infoDetail.trim().hashCode() & 0x7FFFFFFF;
```

但是这样一来，又出问题了。部分泄漏信息中包含一些数组信息，而有一些 leak trace 仅仅是中间某处 [] 内的值不同，但 leak trace 整理结构都是一样的。最后现在采用的方法是，去掉上面 `infoDetail` 里所有 [] 中的数字，即：

``` java
String[] infoDetailArray = leakInfo.trim().split("\n\n")[0].split("\n");
String infoDetail = "";
for (int i = 1; i < infoDetailArray.length; i++) {
    infoDetail += infoDetailArray[i];
}
infoDetail = infoDetail.replaceAll("\\[\\d*\\]", "[]");
Integet leakKey = infoDetail.trim().hashCode() & 0x7FFFFFFF;
```

也就是加了一句 `infoDetail = infoDetail.replaceAll("\\[\\d*\\]", "[]");`。现在暂时采用这个算法来计算 leakKey，不排除后续再修改，如果你有更好的方法，欢迎留言讨论。

**2. 在 AndroidManifest.xml 中注册 service**

上面的 `LeakUploadService` 类完成后，需要在 AndroidManifest.xml 中注册这个 service，在 `<application android:name="xxx">` 和 `</application>` 之间添加 `<service android:name="xxx.LeakUploadService"/>` 即可。其中 `xxx.LeakUploadService` 中的 `xxx` 替换为 `LeakUploadService` 所在的包名。

**3. 修改入口类**

前面讲到官方接入方法是在入口类的 OnCreate 函数下添加 `LeakCanary.install(this);`，这里需要做一个修改，改成：

``` java
public class ExampleApplication extends Application {
    private RefWatcher refWatcher;
    protected RefWatcher installLeakCanary(){return LeakCanary.install(this, LeakUploadService.class, AndroidExcludedRefs.createAppDefaults().build());}
    @Override public void onCreate() {
        super.onCreate();
        refWatcher = installLeakCanary();        
    }
}
```

这样，再出现内存泄漏，就会自动将泄漏信息上传到数据库了。

为了后续做持续集成，这里可以将所有修改写成一个 Shell 脚本，每次打包前执行一次，具体不再做演示。

# 接入 LeakCanary 可能遇到的坑

由于我在公司负责各个产品的 LeakCanary 接入，因此接触到了各种代码结构的软件，接入过程中也遇到了各种各样的问题，这里做个总结。

**1. uses-sdk:minSdkVersion 1 cannot be smaller than version 8 declared in library [com.squareup.leakcanary:leakcanary-android:1.4-beta2]**

这是我接入第一个软件时遇到的问题，原因是这个软件包的结构特别混乱，包含了一堆 sdk，每个支持的 Android 版本还不一致。解决方法是，在 `AndroidManifest.xml` 中添加 `<uses-sdk tools:overrideLibrary="com.squareup.leakcanary"/>`。即：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    ...
    <uses-sdk tools:overrideLibrary="com.squareup.leakcanary"/>
    <application ...>
    ...
    </application>
    ...
</manifest>
```

**2. 找不到继承 Application 的入口类**

这个问题也是我在接入第一个软件时最初遇到的问题，由于软件包结构混乱，没有找到入口类。所以当时的解决方案是，询问研发找到主 Activity，在这个 Activity 中接入 LeakCanary。

其中 `LeakCanary.install(this)` 需要改成 `LeakCanary.install(this.getApplication())`

**3. The number of method references in a .dex file cannot exceed 64K**

这是典型的 64K 问题，具体 Google 官方给了解决方案，地址是：[https://developer.android.com/studio/build/multidex.html](https://developer.android.com/studio/build/multidex.html)

简单来说，需要在 build.gradle 中添加 `multiDexEnabled true`，并在 dependencies 中加入 `compile 'com.android.support:multidex:1.0.0'`，具体如下：

``` gradle
android {
    compileSdkVersion 21
    buildToolsVersion "21.1.0"

    defaultConfig {
        ...
        minSdkVersion 14
        targetSdkVersion 21
        ...

        // Enabling multidex support.
        multiDexEnabled true
    }
    ...
}

dependencies {
  compile 'com.android.support:multidex:1.0.0'
}
```

同时，还需要在 `AndroidManifest.xml` 中添加 multidex 的依赖库：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.multidex.myapplication">
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
</manifest>
```

最后，将继承 Application 类，改成继承 MultiDexApplication 类；重载 `attachBaseContext()`，并调用 `MultiDex.install(this)`。即：

``` java
@Override
protected void attachBaseContext(Context base) {
    super.attachBaseContext(base);
    MultiDex.install(this);
    ...
}
```

**4. 接入完成后，运行软件发生崩溃：java.lang.RuntimeException: Unknown Process com.example.ExampleApp:leakcanary**

这是我遇到最崩溃的问题，当时网上各种找，各种问同事都没有解决。最后是找了负责这款软件的研发才找到问题的根源：研发对未知进程做了限制，此处 LeakCanary 被识别为未知进程，无法初始化。

具体解决方案就不赘述了，不同的软件做的限制也不一样。遇到这种问题，直接找对应的研发帮忙解决。
