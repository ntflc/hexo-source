---
title: LeakCanary+Jenkins 内存泄漏监控实践
date: 2016-09-26 15:57:27
categories:
- Test
tags:
- Test
- LeakCanary
- Jenkins
- Monkey
---

# 前言

本文是之前几篇文章 [为 Android 软件接入 LeakCanary 实现内存泄漏检测](/2016/08/22/LeakCanary-for-Android-App/)、[使用 Jenkins Pipeline 为接入 LeakCanary 的软件做持续集成](/2016/08/23/Jenkins-Pipeline-for-Leakcanary/)、[解决 LeakCanary 和 Jenkins Pipeline 中遇到的问题](/2016/09/20/Fix-Problems-for-LeakCanary-and-Jenkins-Pipeline/) 的总结，为了发布到 [TesterHome](https://testerhome.com)。

<!-- more -->

# 背景

公司 Android 产品的 OOM 崩溃率持续增长，为了检测出内存泄漏问题，决定使用 [LeakCanary](https://github.com/square/leakcanary)。为了持续发现内存泄漏问题，尝试将 LeakCanary 与 Jenkins 相结合。本文着重于 LeakCanary 与 Jenkins 的结合，不会对 LeakCanary 和 Jenkins 本身做过多介绍，敬请谅解。

# 思路

- 将 LeakCanary 接入 Android 产品
  - 在 Jenkins 平台完成 LeakCanary 代码接入和 Debug 包的构建
  - 通过 Shell 脚本实现代码修改
  - 发现泄漏信息后自动上传到数据库
- 使用 Monkey 进行随机操作触发泄漏
- 使用 Jenkins Pipeline 实现持续集成流程

# LeakCanary 接入方法

关于 LeakCanary 的详细信息可以看这里：[LeakCanary 中文使用说明](http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)。

主要修改点有两处：一处是在项目的 `build.gradle` 中加入 LeakCanary 的引用：

``` gradle
dependencies {
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4'
    testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4'
}
```

另一处是在项目的主 Application 类（即 `AndroidManifest.xml` 内 `<application>` 标签中 `android:name` 的值）中安装 LeakCanary：

``` java
import com.squareup.leakcanary.*

public class YourApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // 安装LeakCanary
        LeakCanary.install(this);
    }
}
```

按照最基础的 `LeakCanary.install(this);` 方式接入 LeakCanary 后，发现泄漏都会产生一个通知，点击可以查看具体的 leak trace。考虑到持续集成，这里希望每次发现泄漏后，将相关信息自动上传到数据库。并且由于查看 leak trace 的界面是一个新的 Activity，在跑 Monkey 的过程中也会存在干扰，导致相当一段时间不在产品本身的界面中操作。因此，这里需要做一些修改。

## 修改点 1：发现泄漏后自动上传到数据库

### 新建 `LeakUploadService` 类

在 Application 类所在的 package 内新建一个 `LeakUploadService` 类，继承 `DisplayLeakService` 类：

``` java
import com.squareup.leakcanary.*

public class LeakUploadService extends DisplayLeakService {

    @Override
    protected void afterDefaultHandling(HeapDump heapDump, AnalysisResult result, String leakInfo) {
        if (!result.leakFound || result.excludedLeak){
            return;
        }
        // 下面是处理泄漏数据和上传数据库的代码
    }
}
```

其中，发生泄漏的类名为 `result.className.toString();`，其余信息诸如软件包名、软件版本号、leak trace 等，均在 `leakInfo` 中，形如：

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

一般将软件包名、版本号、leak trace（第一行下面，`* Retaining: 131 KB.` 之上的部分）、泄漏大小等信息上传到数据库即可，有了这些信息，研发就可以定位问题了。

处理方法也很简单，就是对 String 对象进行一些操作。这里我是将发送泄漏的类名、软件包名、软件版本号、整个泄漏信息（除了 Details 部分）上传到数据库，因此代码如下：

``` java
String className = result.className.toString();
String pkgName = leakInfo.trim().split(":")[0].split(" ")[1]
String pkgVer = leakInfo.trim().split(":")[1]
String leakDetail = leakInfo.split("\n\n")[0] + "\n\n" + leakInfo.split("\n\n")[1];
```

至于上传数据库的代码，这里就不提供了（这个是之前同事写的，我直接拿来用了）。

### 注册 Service

接下来需要在 `AndroidManifest.xml` 中注册 Service，即在 `<application android:name=...>` 和 `</application>` 之间添加 `<service android:name="xxx.LeakUploadService" android:exported="false"/>`，其中 `xxx` 为 `LeakUploadService` 所在的 package。

### 修改安装方式

最后修改主 Application 类中的安装方式，改为：


``` java
public class ExampleApplication extends Application {

    private RefWatcher refWatcher;
    protected RefWatcher installLeakCanary(){return LeakCanary.install(this, LeakUploadService.class, AndroidExcludedRefs.createAppDefaults().build());}
    @Override
    public void onCreate() {
        super.onCreate();
        // 安装LeakCanary
        refWatcher = installLeakCanary();        
    }
}
```

做了如上操作后，每次发送泄漏，都会自动将发送泄漏的类名、软件包名、软件版本号、泄漏信息等上传到数据库了。

## 修改点 2：屏蔽 `DisplayLeakActivity` 类

`DisplayLeakActivity` 类是 LeakCanary 展示 leak trace 的类，这个类的存在，会导致跑 Monkey 的过程中，多次进入这个 Activity，在其中操作，从而减少在软件本身界面中的操作时间。屏蔽这个类后，不会再在应用列表中生成一个名为 Leaks 的软件，发送泄漏后，点击通知栏里的泄漏提醒，也不会进入 leak trace 的展示页面。但考虑到泄漏的信息已经上传到数据库，所以这么做也无可厚非。

通过查阅 LeakCanary 源码可以发现，是否开启 `DisplayLeakActivity`，是由 `leakcanary-android/src/main/java/com/squareup/leakcanary/LeakCanary.java` 中的 `enableDisplayLeakActivity()` 函数决定的，该函数为：

``` java
public static void enableDisplayLeakActivity(Context context) {
    setEnabled(context, DisplayLeakActivity.class, true);
}
```

只需要把 `true` 改成 `false` 即可屏蔽 `DisplayLeakActivity` 类。

但我采用的接入方法是直接在 `build.gradle` 中添加远程依赖，并且我也不想修改成本地依赖。因此我首先想到的方法是新建一个类，让其继承 `LeakCanary` 类，然后重写 `enableDisplayLeakActivity()` 函数。但是 `LeakCanary` 类是一个 final 类，无法被继承。

由于 `LeakCanary` 类正好是在前面接入 LeakCanary 时添加代码 `LeakCanary.install(this);` 用到的类，因此我想到的方法是自己创建一个新的类 `LeakCanaryWithoutDisplay`，位置在 `com/squareup/leakcanary` 下（需要自己新建这个 package），里面的代码直接复制 `LeakCanary` 类，然后做如下修改：

- 把 `public final class LeakCanary {` 改为 `public final class LeakCanaryWithoutDisplay {`
- 把 `private LeakCanary() {` 改为 `private LeakCanaryWithoutDisplay() {`
- 修改 `enableDisplayLeakActivity()` 函数，将 `true` 改为 `false`
- 将主 Application 类中安装 LeakCanary 的代码 `LeakCanary.install();` 改为 `LeakCanaryWithoutDisplay.install();`

这样就 OK 了。

# 将接入 LeakCanary 的所有修改整理成 Shell 脚本

由于 Jenkins 打包每次都会拉取最新代码，因此需要将接入 LeakCanary 的修改整理成 Shell 脚本，这样每次拉取最新代码后，执行这个 Shell 脚本，然后再打出的包才是后面流程所需要的包。

编写 Shell 脚本主要使用到的命令是 `sed` 和 `cp`，`sed` 用来修改代码，`cp` 用来复制文件。其中，`LeakUploadService.java` 和 `LeakCanaryWithoutDisplay.java` 是固定的，因此可以提前写好备用，然后直接 `cp` 到相应目录。

Shell 脚本主要分为以下几个部分：

- 修改 `build.gradle`，添加 LeakCanary 依赖
- 修改 `AndroidManifest.xml`，注册 Service
- 修改主 Application 类，安装 LeakCanary
- 复制 `LeakUploadService.java`（以及上传数据库可能需要用到的其他 Java 文件）到主 Application 类所在的包下
- 复制 `LeakCanaryWithoutDisplay.java` 到 `com/squareup/leakcanary` 下

由于不同项目的脚本存在一定差异，这里就不给出具体的 Shell 脚本了。

# 新建 Jenkins 项目，用于构建包含 LeakCanary 的 Debug 包

在 Jenkins 中新建项目，配置源码管理。

将编写的 Shell 脚本和相关文件拷贝到 Jenkins 项目文件夹中。

添加构建步骤，选择 Execute shell，执行 Shell 脚本，命令为：

``` bash
cd $WORKSPACE
sh ../your_shell_file_name.sh
```

添加构建步骤，选择 Invoke Gradle script，生成 Debug 包，配置如下图：

![](/images/LeakCanary-Jenkins-Practice/1.jpg)

# 新建 Jenkins Pipeline 项目，用于实现完整流程

结合我司实际环境，Jenkins 项目的打包是在 master 节点上进行的，而跑 Monkey 的操作是在另一台服务器上进行的（这台服务器是 Jenkins 的一个从节点，搭建了 [STF](https://github.com/openstf/stf) 服务，连有多台测试机），记这个从节点的标签为 Linux-for-stf。

Pipeline 的步骤如下图所示：

![](/images/LeakCanary-Jenkins-Practice/2.jpg)

包括 5 个步骤：

1. 首先在 master 节点构建接入了 LeakCanary 的 apk 包
2. 然后将 apk 包拷贝到 STF 节点
3. 接着在 STF 节点安装 apk 包
4. 其次在 STF 节点上选择一台手机跑 Monkey 触发泄漏
5. 最后发送邮件提醒（项目是否构建成功）

对应 Pipeline script 可以简化为：

``` groovy
node('master') {
    stage 'build Job1'
    build 'Job1'

    stage 'scp apk to stf node'
    def apkDir="/home/test/.jenkins/jobs/Job1/workspace/app/build/outputs/apk"
    def destDir="stf@linux-for-stf:/home/stf/jenkins/apks"
    sh "scp $apkDir/test.apk $destDir"
}
node('Linux-for-stf') {
    stage 'install apk'
    def device="ABCD1234"
    def apkFile="/home/stf/jenkins/apks/test.apk"
    sh "adb -s $device install -r $apkFile"

    stage 'run monkey'
    sh "adb -s $device shell monkey -p com.example.ExampleApp -s 100 --ignore-crashes --ignore-timeouts --throttle 700 -v 10000"
}
node('master') {
    stage 'send email'
    mail to: 'test@gmail.com',
    subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) finished",
    body: "Please go to ${env.BUILD_URL} and verify the build"
}
```

这里会遇到一个问题：这个 Pipeline 项目完成后会发送一封邮件，但如果中途某一步出了问题，就直接结束运行，从而导致成功可以收到邮件、失败却收不到邮件的情况。解决方法是，使用 `try catch`。改进的 Pipeline script 为：

``` groovy
try {
    node('master') {
        stage 'build Job1'
        build 'Job1'

        stage 'scp apk to stf node'
        def apkDir="/home/test/.jenkins/jobs/Job1/workspace/app/build/outputs/apk"
        def destDir="stf@linux-for-stf:/home/stf/jenkins/apks"
        sh "scp $apkDir/test.apk $destDir"
    }
    node('Linux-for-stf') {
        stage 'install apk'
        def device="ABCD1234"
        def apkFile="/home/stf/jenkins/apks/test.apk"
        sh "adb -s $device install -r $apkFile"

        stage 'run monkey'
        sh "adb -s $device shell monkey -p com.example.ExampleApp -s 100 --ignore-crashes --ignore-timeouts --throttle 700 -v 10000"
    }
    node('master') {
        stage 'send email'
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) succeeded",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
} catch (Exception e) {
    node('master') {
        stage 'send email'
        echo '$e'
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
}
```

这样持续了一段时间后发现，跑 Monkey 过程中经常会下拉状态栏，然后点击里面的快捷按钮，而且经常会点击到 WiFi 按钮，从而导致 WiFi 被关闭。WiFi 被关闭，意味着发现的泄漏信息无法上传到数据库，因此这个问题必须要解决。然而从网上搜索的结果来看并不理想，针对这个问题，大部分人都表示没有什么好的方法。

好在有一个 GitHub 项目被我发现了，叫 [simiasque](https://github.com/Orange-OpenSource/simiasque)。这是一款 Android 软件，通过全局遮罩遮住状态栏位置来防止 Monkey 下拉状态栏，测试效果非常好。使用方法也很简单，安装 demo 下的 apk 文件，打开软件，点击 Hide status bar 按钮即可。更方便的是，作者也提供了启动和关闭的命令，分别是：

开启：

``` bash
adb shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable true
```

关闭：

``` bash
adb shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable false
```

接下来要做的，就是在 Pipeline script 跑 Monkey 的命令之前加上安装和开启 simiasque 的命令，跑完 Monkey 后再加上关闭 simiasque 的命令即可。

最终的 Pipeline script 大致是这样的：

``` groovy
try {
    node('master') {
        stage 'build Job1'
        build 'Job1'

        stage 'scp apk to stf node'
        def apkDir="/home/test/.jenkins/jobs/Job1/workspace/app/build/outputs/apk"
        def destDir="stf@linux-for-stf:/home/stf/jenkins/apks"
        sh "scp $apkDir/test.apk $destDir"
    }
    node('Linux-for-stf') {
        stage 'install apk'
        def device="ABCD1234"
        def apkFile="/home/stf/jenkins/apks/test.apk"
        sh "adb -s $device install -r $apkFile"

        stage 'run monkey'
        sh "adb -s $device install -r /home/stf/jenkins/simiasque-debug.apk"
        sh "adb -s $device shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable true"
        sh "adb -s $device shell monkey -p com.example.ExampleApp -s 100 --ignore-crashes --ignore-timeouts --throttle 700 -v 10000"
        sh "adb -s $device shell am broadcast -a org.thisisafactory.simiasque.SET_OVERLAY --ez enable false"
    }
    node('master') {
        stage 'send email'
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) succeeded",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
} catch (Exception e) {
    node('master') {
        stage 'send email'
        echo '$e'
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
}
```

到此为止，Jenkins Pipeline 的步骤基本是完成了。接下来，设置好每天运行 1 次，泄漏信息尽收数据库之中。

# 后续流程

以上流程完成后，LeakCanary+Jenkins 的内存泄漏监控实践基本上是完成了。但在公司的实际项目中，这样仅仅是发现和收集了泄漏信息，最关键的还是让研发修改这些问题。由于不同公司采用的 Bug 平台不尽相同，不同 Bug 平台的区别可能也很大，因此这里就不具体展开后面的步骤了。大体思路就是，定期将数据库中的泄漏信息提交到 Bug 平台上。至于实现方法，如果提供接口当然是直接用调用接口；如果不提供接口，也可以尝试直接向数据库添加信息。

总之，发现内存泄漏的关键还是解决，如果不解决，上面所做的一切都是白费。

当然，上面的一些方法可能并不完美，不过都是我在最近一段时间的工作中慢慢摸索得到的。如果你有更好的解决方案，欢迎交流。
