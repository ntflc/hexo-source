title: 在与 iOS 连接的 Android Wear 设备上安装第三方表盘
date: 2015-12-29 23:27:34
categories:
- Tutorial
tags:
- Tutorial
- iOS
- Android Wear
---

几个月前，Google 宣布 Android Wear 支持 iOS。看似很美好，但是连接 iOS 设备后，手表仅仅只能接收推送，无法像 Android 手机那样，在手机上安装软件，然后同步手表端 apk 到手表上。这也导致我之前购买的一款表盘无法使用，加之默认表盘丑陋无比，于是想到一个点子——强行安装表盘 apk。

<!-- more -->

于是尝试直接通过 `adb install` 安装表盘 apk，但并没有效果。突然想起，这个 apk 是手机端的，我应该获取手表端的 apk 再安装。

我购买的表盘叫 GlowSticks Analog Watch，在 Google Play 商城卖 0.99 美元。这里以这款表盘为例。

具体步骤：

1. 将 Android Wear 连接 Android 手机，并在手机上安装 GlowSticks Analog Watch。

2. 手表连接电脑，打开 USB 调试。

3. 电脑端运行 `CMD`，执行：

   ``` sh
   adb shell
   pm list packages
   exit
   ```

   获取手表上安装的所有应用。

   这里我这个表盘应用名为 `au.dx.watchface.neonone`

4. 电脑端继续执行：

   ``` sh
   adb pull /data/app/au.dx.watchface.neonone-1/base.apk d:/
   ```

   将手表端的表盘 apk 拉到电脑端。

5. 手表恢复出厂设置后，连接 iOS 设备。

6. 手表连接电脑，打开 USB 调试

7. 电脑端执行：

   ``` sh
   adb install base.apk
   ```

到此大功告成。此时长按手表，你就可以看到新的表盘了。
