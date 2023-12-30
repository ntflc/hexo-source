---
title: 使用 Jenkins Pipeline 为接入 LeakCanary 的软件做持续集成
date: 2016-08-23 11:04:52
categories:
- Test
tags:
- Test
- Jenkins
- LeakCanary
---

上一篇文章 [为 Android 软件接入 LeakCanary 实现内存泄漏检测](/2016/08/22/LeakCanary-for-Android-App/) 讲了如何为软件接入 LeakCanary 内存泄漏检测，这篇文章着重讲讲如何通过 Jenkins Pipeline 来做持续集成。

<!-- more -->

# Jenkins

Jenkins（[https://jenkins.io](https://jenkins.io)）是一个用 Java 编写的开源的持续集成工具，前身是 Hudson。在与 Oracle 发生争执后，项目从 Hudson 项目复刻。

Jenkins 提供了软件开发的持续集成服务。它运行在 Servlet 容器中（例如 Apache Tomcat），支持软件配置管理（SCM）工具（包括 AccuRev SCM、CVS、Subversion、Git、Perforce、Clearcase 和 RTC），可以执行基于 Apache Ant 和 Apache Maven 的项目，以及任意的 Shell 脚本和 Windows 批处理命令。Jenkins 的主要开发者是川口耕介，是在 MIT 许可证下发布的自由软件。

Jenkins 的扩展插件已经发布，能使非 Java 语言编写的项目也使用 Jenkins。对于大多数的版本控制系统和大的数据库，有与 Jenkins 集成的插件可用。许多构建（build）工具都是通过他们各自的插件提供支持。插件还可以改变 Jenkins 的外观，或添加新的功能。

构建时可以生成各种格式的测试报告（JUnit 是被内建支持的，别的格式则需通过插件）。Jenkins 可以显示报表，生成趋势图，并在图形化界面中呈现它们。

（以上内容来自维基百科词条：[Jenkins (软件)](https://zh.wikipedia.org/wiki/Jenkins_(%E8%BD%AF%E4%BB%B6))）

# Pipeline

对于常规的 Jenkins 任务，一般只能在一个节点完成一件事情。 Pipeline 的加入，让 Jenkins 从单一任务变成了流水线式的任务。Pipeline 是 Jenkins 的一个插件，需要自行安装。安装后，新建任务的时候就会看见多了一个 Pipeline 的选项。

Pipeline 和主要构造内容是 Pipeline script，语法很简单，并且可以使用 Groovy 沙盒。常用命令有选择工作节点 `node('name_of_your_node')`，步骤 `stage 'name_of_your_stage'`，构建已有的 Jenkins 任务 `build 'name_of_jenkins_job'`，定义变量 `def str="Hello World"`，执行 Shell 脚本 `sh "shell_script"`，输出提示语 `echo 'Hello World'` 等等。下面给出一个简单的 Pipeline 的例子：

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
}
```

这个例子共有个 3 步骤：第 1 步，构建项目 Job1；第 2 步，将第 1 步生成 apk 文件拷贝到 Linux-for-stf 节点。前两步均在 Jenkins 的 master 节点执行。第 3 步，运行节点不再是 master 节点，而是 Linux-for-stf 节点，给指定手机安装刚才拷贝过来的 apk 文件。

# STF (Smartphone Test Farm)

STF (Smartphone Test Farm) 是 GitHub 上的一个开源项目（[https://github.com/openstf/stf](https://github.com/openstf/stf)），是一个通过浏览器来控制、调试手机的 WEB 软件。只需要一台 Linux 服务器，和数台手机，就可以通过浏览器远程操作这些手机了。对于一个公司的测试部门来说，STF 可以有效解决手机多、难以管理的问题。这里只是简单介绍一下，之后会单独写一篇关于 STF 的文章。

在这里，STF 并不是必须的，只是因为我们公司的测试机全部连在 STF 服务器上，所以这里提一下。对于本篇文章，你完全可以将测试手机直接连在 Jenkins 服务器上，或者连在任意一个 Jenkins 节点上，反正后面涉及到的都是 `adb shell` 命令。

# 使用 Jenkins Pipeline 为接入 LeakCanary 的软件做持续集成

由于我在公司负责各产品的 LeakCanary 接入，同时也要负责接入后的管理，那么持续集成肯定是必不可少的环节。使用 Pipeline，我就可以将打包、安装、跑 Monkey 等步骤串在一起。

我使用到的 Jenkins 节点有 2 个，一个是 master 主节点，一个是 Linux-for-stf 节点。前者是 Jenkins 的主服务器，负责打包等工作；后者是 STF 所在的服务器，连接有数台测试机。

## 思路

这里主要分为 5 个步骤：

1. 在 master 节点构建接入了 LeakCanary 的 apk 包
2. 将 apk 包拷贝到 Linux-for-stf 节点
3. 在 Linux-for-stf 节点安装 apk 包
4. 选择 STF 中的一台手机跑 Monkey（用来发现泄漏信息，发现后自动上传到数据库）
5. 发送邮件

其中，跑 Monkey 会遇到一些问题，后面会提到。

## 过程

最初，Pipeline script 大概如下：

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

然后发现个问题，一旦中间某个过程失败了，就不会发出邮件了。于是查阅了资料，发现可以直接用 `try catch`，然后就诞生了下面这个版本：

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
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
}
```

这样，无论成功失败，都会有邮件提醒，并且通过标题就能知道是否成功了。

## 新的问题

### 1. Leaks 进程的干扰

在跑 Monkey 的时候，我发现了一个新的问题。因为接入了 LeakCanary 的软件，会生成一个新的应用叫 Leaks，且包名与原软件包名一致，因此跑 Monkey 的时候，经常会进入 Leaks 里，甚至起始就进入。这个问题暂时还没有彻底解决，暂时想到的解决方法是，先使用 `adb shell am start` 开启软件本身的进程（进程名可以通过 `adb logcat` 获取，具体方法是，打开软件，然后在 logcat 里搜索 `cmp=`，然后自己识别下就能找到对应进程了），这样跑 Monkey 时即使进入 Leaks，还是会返回到软件本身的进程。因此产生了下面的版本：

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
        sh "adb -s $device shell am start -n com.example.ExampleApp/.MainActivity"
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
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
}
```

但这终究只是个妥协了的办法。其中也尝试过 CrashMonkey4Android（[https://testerhome.com/topics/2698](https://testerhome.com/topics/2698)），这是 TesterHome 上一位大神做的基于 CTS 框架的改良版 Monkey，可以限制只在某个进程中跑 Monkey，这样就可以避免进入 Leaks 的进程。但由于各种各样的问题，这个方案被停掉了。如果你有其他解决方案，欢迎留言讨论。

### 2、跑 Monkey 过程中，WiFi 被关闭

因为接入 LeakCanary 时，我们在发现泄漏信息后，会自动将其上传到数据库。如果 WiFi 被关闭，就没法上传了。现在的解决方案也只是临时性的，就是在每次跑 Monkey 前，使用 UIAutomator 先判断 WiFi 状态，如果关闭则开启 WiFi。对于已经 Root 的手机，这一步可以直接用 `adb shell svc wifi enable` 解决。但考虑到我们公司的大部分测试机没有 Root，因此只是使用 UIAutomator 模拟打开 WiFi。

关于 UIAutomator，这里不做介绍，感兴趣的可以自行搜索。这里提供我写的打开 WiFi 的 UIAutomator 代码：

``` java
import com.android.uiautomator.core.UiObject;
import com.android.uiautomator.core.UiObjectNotFoundException;
import com.android.uiautomator.core.UiScrollable;
import com.android.uiautomator.core.UiSelector;
import com.android.uiautomator.testrunner.UiAutomatorTestCase;

import android.os.RemoteException;

public class MainTest extends UiAutomatorTestCase {

	public void testCase() throws RemoteException, UiObjectNotFoundException {
		wakeUpDevice();
		getUiDevice().pressHome();
		openSettings();
		turnOnWifi();
		getUiDevice().pressHome();
	}
	
	private void wakeUpDevice() throws RemoteException {
		if (!getUiDevice().isScreenOn()) {
			getUiDevice().wakeUp();
			getUiDevice().swipe(100, 1500, 980, 1500, 20);
		}
	}
	
	private void openSettings() throws UiObjectNotFoundException {
		// 下拉快速设置
		getUiDevice().openQuickSettings();
		sleep(1000);
		UiSelector settingsSelector = new UiSelector().resourceId("com.android.systemui:id/settings_button");
		UiObject settingsButton = new UiObject(settingsSelector);
		settingsButton.click();
	}
	
	private void turnOnWifi() throws UiObjectNotFoundException {
		// 新建UiScrollable类
		UiScrollable scroll = new UiScrollable(new UiSelector().scrollable(true));
		// 滑动选择并点击WLAN
		UiObject wifiItem = scroll.getChildByText(new UiSelector().className("android.widget.TextView"), "WLAN");
		wifiItem.clickAndWaitForNewWindow();
		// 等待3秒以防止页面未加载完成
		sleep(3000);
		// 选择Wifi开关
		UiSelector wifiSwitchSelector = new UiSelector().resourceId("com.android.settings:id/switch_widget");
		UiObject wifiSwitchButton = new UiObject(wifiSwitchSelector);
		// 判断Wifi是否开启，如果未开启则开启
		if (!wifiSwitchButton.isChecked()) {
			wifiSwitchButton.click();
		}
	}
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub

	}

}
```

由于不同 Android 手机的下拉栏不同，所以此代码并不通用。

然后，将打包好的 jar 包放到 Linux-for-stf 节点，再在 Pipeline script 中加入 UIAutomator 的部分，即变成下面这个版本：

``` groovy
try {
    node('master') {
        stage 'build Job1'
        build 'Job1'

        stage 'scp files to stf node'
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
        sh "adb -s $device push /home/stf/jenkins/TurnOnWifi.jar /data/local/tmp"
        sh "adb -s $device shell uiautomator runtest TurnOnWifi.jar -c com.ntflc.MainTest"
        sleep 3
        sh "adb -s $device shell am start -n com.example.ExampleApp/.MainActivity"
        sleep 3
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
        mail to: 'test@gmail.com',
        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
        body: "Please go to ${env.BUILD_URL} and verify the build"
    }
}
```

### 3. 一些特殊软件

由于有些软件的特殊性质，比如锁屏类、桌面类软件，直接用 Monkey 会有各种各样的问题。对于这类软件，没有统一的解决方法，只能具体问题具体分析。这里我就不提供具体方案了。
