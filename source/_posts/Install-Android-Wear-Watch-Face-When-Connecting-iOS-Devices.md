title: 在与 iOS 连接的 Android Wear 设备上安装第三方表盘
date: 2015-12-29 23:27:34
categories:
- 教程
tags:
- 教程
- iOS
- Android Wear
---

几个月前，Google宣布Android Wear支持iOS。看似很美好，但是连接iOS设备后，手表仅仅只能接收推送，无法像Android手机那样，在手机上安装软件，然后同步手表端apk到手表上。这也导致我之前购买的一款表盘无法使用，加之默认表盘丑陋无比，于是想到一个点子——强行安装表盘apk。

于是尝试直接通过`adb install`安装表盘apk，但并没有效果。突然想起，这个apk是手机端的，我应该获取手表端的apk再安装。

<!-- more -->

我购买的表盘叫GlowSticks Analog Watch，在Google Play商城卖0.99美元。这里以这款表盘为例。

具体步骤：

1、将Android Wear连接Android手机，并在手机上安装GlowSticks Analog Watch。

2、手表连接电脑，打开USB调试。

3、电脑端运行CMD，执行：

``` sh
adb shell

pm list packages

exit
```

获取手表上安装的所有应用。

这里我这个表盘应用名为`au.dx.watchface.neonone`

4、电脑端继续执行：

``` sh
adb pull /data/app/au.dx.watchface.neonone-1/base.apk d:/
```

将手表端的表盘apk拉到电脑端。

5、手表恢复出厂设置后，连接iOS设备。

6、手表连接电脑，打开USB调试

7、电脑端执行：

``` sh
adb install base.apk
```

到此大功告成！此时长按手表，你就可以看到新的表盘了！
