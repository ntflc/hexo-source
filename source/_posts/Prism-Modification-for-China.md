title: Sense 7.0 强制 BlinkFeed、主题为国内源
date: 2015-06-15 23:51:00
categories:
- Tutorial
tags:
- Tutorial
- BlinkFeed
- HTC Sense
---

在 Sense 6 里，强制中文源是通过改 `DeviceManagement.apk` 来实现的。
具体思路是修改 `smali\com\htc\cs\env\HepDeviceEnv.smali`，强制识别 `CID` 为 `HTCCN701`。

而 Sense 7 中，其实修改的方法不变，只是代码位置变了，改到 `Prism.apk` 里了。

而且这样修改后，主题源也变国内了（国内源主题独立于国外源，国外源主题需要翻墙才能连接）。

<!-- more -->

具体方法为：
反编译 `Prism.apk`，找到 `smali\com\htc\lib1\dm\env\DeviceEnv.smali`，搜索 
`.method public getCID()Ljava/lang/String;`，在此段代码的最后两行

``` smali
move-result-object v0
```

和

``` smali
return-object v0
```

中加一行

``` smali
const-string v0, "HTCCN701"
```

即可。