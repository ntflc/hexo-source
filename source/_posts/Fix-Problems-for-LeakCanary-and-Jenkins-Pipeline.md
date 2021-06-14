---
title: 解决 LeakCanary 和 Jenkins Pipeline 中遇到的问题
date: 2016-09-20 16:41:35
categories:
- 测试
tags:
- 测试
- Jenkins
- LeakCanary
---

之前一篇文章“[使用 Jenkins Pipeline 为接入 LeakCanary 的软件做持续集成](http://ntflc.com/2016/08/23/Jenkins-Pipeline-for-Leakcanary/)”中提到几个问题，一个是 Leaks 进程的干扰，一个是跑 Monkey 过程中误触下拉通知栏导致 Wifi 被关闭，本篇文章将就这两个问题给出一些解决方案，可能不是最好的，但能够一定程度上解决这些问题。

<!-- more -->

# Leaks 进程的干扰

这个问题之前采用的解决方案是，在跑 Monkey 之前，先使用`adb shell am start`开启软件本身的进程。但其实这并没有什么用，即使不开启本身进程，跑 Monkey 的过程中还是会跳转到软件主进程内，并不会减少位于 DisplayLeakService 进程的时间。

## 思路分析

如果我们想解决这个问题，只有一个方法，就是修改 LeakCanary 的源码，屏蔽掉DisplayLeakActivity。

分析了下 LeakCanary 的源码（[https://github.com/square/leakcanary](https://github.com/square/leakcanary)），发现主要功能在`leakcanary-android`、`leakcanary-analyzer`和`leakcanary-watcher`中。其中涉及 DisplayLeakActivity 的代码均在`leakcanary-android`中，我们可以通过修改`leakcanary-android/src/main/java/com/squareup/leakcanary/LeakCanary.java`中的`enableDisplayLeakActivity()`函数来控制 DisplayLeakActivity 进程的启用与否。方法很简单，将

``` java
public static void enableDisplayLeakActivity(Context context) {
    setEnabled(context, DisplayLeakActivity.class, true);
}
```

改为

``` java
public static void enableDisplayLeakActivity(Context context) {
    setEnabled(context, DisplayLeakActivity.class, false);
}
```

即可。

我们之前采用的接入方法是，直接在 `build.gradle` 文件中添加依赖，然后编译时会自动下载源码进行编译。根据上面的分析，我们可以通过两个方法来实现：一个方法是，将 LeakCanary 代码拉取到本地，然后合并到项目代码中。这样的好处是，以后如果还有其他修改，可以直接修改本地的 LeakCanary 代码，然后打包。另一个方法是，因为现在需要修改的仅仅是一个类中的一个函数，因此我们可以新建一个类，继承需要修改的类，然后重写这个函数。接下来，我会就这两个方法具体分析。

## 方案1 将 LeakCanary 代码直接合并入项目中

首先，拉取 LeakCanary 的源码。

先将`leakcanary-android`、`leakcanary-analyzer`、`leakcanary-watcher`三个文件夹拷贝到自己项目的根目录。

然后修改`leakcanary-android/src/main/java/com/squareup/leakcanary/LeakCanary.java`文件，将`enableDisplayLeakActivity()`函数中的`setEnabled(context, DisplayLeakActivity.class, true);`改为`setEnabled(context, DisplayLeakActivity.class, false);`

其次，将`gradle`文件夹中的`checkstyle.gradle`和`gradle-mvn-push.gradle`文件拷贝到自己项目的`gradle`文件夹中。

接着，将根目录的`build.gradle`中`ext`部分拷贝到自己项目根目录的`build.gradle`中，即：

``` gradle
ext {
  minSdkVersion = 8
  compileSdkVersion = 23
  targetSdkVersion = compileSdkVersion
  buildToolsVersion = '23.0.2'
  javaVersion = JavaVersion.VERSION_1_7

  GROUP = 'com.squareup.leakcanary'
  VERSION_NAME = "1.5-SNAPSHOT"
  POM_PACKAGING = "pom"
  POM_DESCRIPTION= "Leak Canary"

  POM_URL="http://github.com/square/leakcanary/"
  POM_SCM_URL="http://github.com/square/leakcanary/"
  POM_SCM_CONNECTION="scm:git:https://github.com/square/leakcanary.git"
  POM_SCM_DEV_CONNECTION="scm:git:git@github.com:square/leakcanary.git"

  POM_LICENCE_NAME="The Apache Software License, Version 2.0"
  POM_LICENCE_URL="http://www.apache.org/licenses/LICENSE-2.0.txt"
  POM_LICENCE_DIST="repo"

  POM_DEVELOPER_ID="square"
  POM_DEVELOPER_NAME="Square, Inc."
}
```

这一部分可能会随着 LeakCanary 版本的变化而变化，具体以 LeakCanary 源码中的为准。

最后是添加依赖信息，修改自己项目跟目录的`settings.gradle`，添加：

``` gradle
include ':leakcanary-android'
include ':leakcanary-analyzer'
include ':leakcanary-watcher'
```

之前方法在项目的`build.gradle`中添加`debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4'`，现在改为添加`compile project(':leakcanary-android')`。

其余修改和之前保持一致。

## 方案2 重写相关函数

因为我们只需要修改`leakcanary-android/src/main/java/com/squareup/leakcanary/LeakCanary.java`中的一个函数，因此很容易想到继承+重写的方法。但是，LeakCanary 类是一个 final 类，无法被继承。既然这样，我们可以重新写一个类，内容和 LeakCanary 类保持一致，然后再修改上面提到的部分。之所以这样做，是因为 LeakCanary 类正是我们在接入时调用的类（`LeakCanary.install(this)`）。

此方法接入 LeakCanary 的方法与之前文章里提到的一样。但需要在项目的代码目录，新建`com/squareup/leakcanary`包，然后在这个包内新建一个类，名字随意，这里取`LeakCanaryWithoutDisplay.java`。直接将`LeakCanary.java`中的代码复制到这里，修改`public final class LeakCanary`为`public final class LeakCanaryWithoutDisplay`、`private LeakCanary()`为`private LeakCanaryWithoutDisplay()`，即：

``` java
/*
 * Copyright (C) 2015 Square, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.squareup.leakcanary;

import android.app.Application;
import android.content.Context;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.res.Resources;
import android.os.Build;
import android.util.Log;
import com.squareup.leakcanary.internal.DisplayLeakActivity;
import com.squareup.leakcanary.internal.HeapAnalyzerService;

import static android.text.format.Formatter.formatShortFileSize;
import static com.squareup.leakcanary.BuildConfig.GIT_SHA;
import static com.squareup.leakcanary.BuildConfig.LIBRARY_VERSION;
import static com.squareup.leakcanary.internal.LeakCanaryInternals.isInServiceProcess;
import static com.squareup.leakcanary.internal.LeakCanaryInternals.setEnabled;

public final class LeakCanaryWithoutDisplay {

  /**
   * Creates a {@link RefWatcher} that works out of the box, and starts watching activity
   * references (on ICS+).
   */
  public static RefWatcher install(Application application) {
    return install(application, DisplayLeakService.class,
        AndroidExcludedRefs.createAppDefaults().build());
  }

  /**
   * Creates a {@link RefWatcher} that reports results to the provided service, and starts watching
   * activity references (on ICS+).
   */
  public static RefWatcher install(Application application,
      Class<? extends AbstractAnalysisResultService> listenerServiceClass,
      ExcludedRefs excludedRefs) {
    if (isInAnalyzerProcess(application)) {
      return RefWatcher.DISABLED;
    }
    enableDisplayLeakActivity(application);
    HeapDump.Listener heapDumpListener =
        new ServiceHeapDumpListener(application, listenerServiceClass);
    RefWatcher refWatcher = androidWatcher(application, heapDumpListener, excludedRefs);
    ActivityRefWatcher.installOnIcsPlus(application, refWatcher);
    return refWatcher;
  }

  /**
   * Creates a {@link RefWatcher} with a default configuration suitable for Android.
   */
  public static RefWatcher androidWatcher(Context context, HeapDump.Listener heapDumpListener,
      ExcludedRefs excludedRefs) {
    LeakDirectoryProvider leakDirectoryProvider = new DefaultLeakDirectoryProvider(context);
    DebuggerControl debuggerControl = new AndroidDebuggerControl();
    AndroidHeapDumper heapDumper = new AndroidHeapDumper(context, leakDirectoryProvider);
    heapDumper.cleanup();
    Resources resources = context.getResources();
    int watchDelayMillis = resources.getInteger(R.integer.leak_canary_watch_delay_millis);
    AndroidWatchExecutor executor = new AndroidWatchExecutor(watchDelayMillis);
    return new RefWatcher(executor, debuggerControl, GcTrigger.DEFAULT, heapDumper,
        heapDumpListener, excludedRefs);
  }

  public static void enableDisplayLeakActivity(Context context) {
    setEnabled(context, DisplayLeakActivity.class, true);
  }

  public static void setDisplayLeakActivityDirectoryProvider(
      LeakDirectoryProvider leakDirectoryProvider) {
    DisplayLeakActivity.setLeakDirectoryProvider(leakDirectoryProvider);
  }

  /** Returns a string representation of the result of a heap analysis. */
  public static String leakInfo(Context context, HeapDump heapDump, AnalysisResult result,
      boolean detailed) {
    PackageManager packageManager = context.getPackageManager();
    String packageName = context.getPackageName();
    PackageInfo packageInfo;
    try {
      packageInfo = packageManager.getPackageInfo(packageName, 0);
    } catch (PackageManager.NameNotFoundException e) {
      throw new RuntimeException(e);
    }
    String versionName = packageInfo.versionName;
    int versionCode = packageInfo.versionCode;
    String info = "In " + packageName + ":" + versionName + ":" + versionCode + ".\n";
    String detailedString = "";
    if (result.leakFound) {
      if (result.excludedLeak) {
        info += "* EXCLUDED LEAK.\n";
      }
      info += "* " + result.className;
      if (!heapDump.referenceName.equals("")) {
        info += " (" + heapDump.referenceName + ")";
      }
      info += " has leaked:\n" + result.leakTrace.toString() + "\n";
      info += "* Retaining: " + formatShortFileSize(context, result.retainedHeapSize) + ".\n";
      if (detailed) {
        detailedString = "\n* Details:\n" + result.leakTrace.toDetailedString();
      }
    } else if (result.failure != null) {
      // We duplicate the library version & Sha information because bug reports often only contain
      // the stacktrace.
      info += "* FAILURE in " + LIBRARY_VERSION + " " + GIT_SHA + ":" + Log.getStackTraceString(
          result.failure) + "\n";
    } else {
      info += "* NO LEAK FOUND.\n\n";
    }
    if (detailed) {
      detailedString += "* Excluded Refs:\n" + heapDump.excludedRefs;
    }

    info += "* Reference Key: "
        + heapDump.referenceKey
        + "\n"
        + "* Device: "
        + Build.MANUFACTURER
        + " "
        + Build.BRAND
        + " "
        + Build.MODEL
        + " "
        + Build.PRODUCT
        + "\n"
        + "* Android Version: "
        + Build.VERSION.RELEASE
        + " API: "
        + Build.VERSION.SDK_INT
        + " LeakCanary: "
        + LIBRARY_VERSION
        + " "
        + GIT_SHA
        + "\n"
        + "* Durations: watch="
        + heapDump.watchDurationMs
        + "ms, gc="
        + heapDump.gcDurationMs
        + "ms, heap dump="
        + heapDump.heapDumpDurationMs
        + "ms, analysis="
        + result.analysisDurationMs
        + "ms"
        + "\n"
        + detailedString;

    return info;
  }

  /**
   * Whether the current process is the process running the {@link HeapAnalyzerService}, which is
   * a different process than the normal app process.
   */
  public static boolean isInAnalyzerProcess(Context context) {
    return isInServiceProcess(context, HeapAnalyzerService.class);
  }

  private LeakCanaryWithoutDisplay() {
    throw new AssertionError();
  }
}
```

这里的代码可能会随着 LeakCanary 版本的变化而变化，具体以 LeakCanary 源码中的为准。

最后，把软件入口类中的`LeakCanary.install()`改为`LeakCanaryWithoutDisplay.install()`即可。

# 跑 Monkey 过程中误触下拉通知栏导致 Wifi 被关闭

这个问题之前采用的方案是，每次跑 Monkey 前执行一次 UIAutomator 判断 Wifi 状态。但如果跑 Monkey 的过程中 Wifi 被关闭了，还是会导致部分泄漏信息无法上传数据库。

后来我无意间在 GitHub 上发现了这样一个项目：[https://github.com/Orange-OpenSource/simiasque](https://github.com/Orange-OpenSource/simiasque)。这个项目是一个 Android App，安装后点击“Hide status bar”就会在状态栏处生成一个全局遮罩，使状态栏无法下拉，从而屏蔽跑 Monkey 时下拉通知栏。你可以将这个项目拷贝下来自己打包，也可以直接下载 demo 文件夹里的 apk 文件。

该项目的 README 里也介绍了如何通过 adb shell 命令开启和关闭这个遮罩。

开启：

``` sh
adb shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable true
```

关闭：

``` sh
adb shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable false
```

接下来，就是在每次跑 Monkey 前，开启遮罩，跑完后再关闭遮罩即可。

<br><br>

到这里，这两个问题就算是解决了。实测解决这两个问题后，每天发现泄漏信息的效率比以前高了很多。

如果你有更好的方法，欢迎交流。