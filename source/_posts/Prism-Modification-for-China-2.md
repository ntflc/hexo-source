title: Sense 7.0 Prism.apk 修改内容详解
date: 2015-09-24 22:18:34
categories:
- Tutorial
tags:
- Tutorial
- BlinkFeed
- HTC Sense
---

之前的一篇文章 [Sense 7.0 强制BlinkFeed、主题为国内源](/2015/06/15/Prism-Modification-for-China/) 已经讲了通过强制识别 `CID` 为 `HTCCN701` 来实现国内新闻源、主题源。而在今天，Sense 首页更新到 7.20.625874 后，代码结构发生了些许变化。借此机会，把我对 Prism.apk 的修改再讲一遍。

<!-- more -->

对于 Prism.apk 的修改，主要是两处。

第一处就是强制识别 `CID` 为 `HTCCN701` 的，最新版本起，代码位置从 `smali\com\htc\lib1\dm\env\DeviceEnv.smali` 移到了 `smali_classes2\com\htc\lib1\dm\env\DeviceEnv.smali`，也就是从 `classes.dex` 移到了 `classes2.dex` 里，`smali_classes2` 是 `classes2.dex` 反编译后对应的文件夹。

具体修改方法，还是搜索 `.method public getCID()Ljava/lang/String;`，在此段代码的最后两行

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

第二处是强制显示大图的，这个改不改因人而异，因为有些图强制放大会很模糊。

修改方法为，反编译 `Prism.apk`，找到 `smali_classes2\com\htc\libmosaicview\FeedGridLayoutHelper.smali`（之前的版本是在 `smali` 文件夹里，7.20.625874 版本起移到了 `smali_classes2`），搜索 `.method public static checkForNeedOverLay(Landroid/content/Context;Lcom/htc/libfeedframework/FeedData;I)Z`，将整段代码改为：

``` smali
.method public static checkForNeedOverLay(Landroid/content/Context;Lcom/htc/libfeedframework/FeedData;I)Z
    .locals 1

    const/4 v0, 0x1

    return v0
.end method
```

即可。

这段代码本来是根据一定的方法判断是否全图显示，现在直接返回 1，就可以做到强制大图了。

总的来说，相比之前版本的 Prism，修改内容其实没变，只是位置发生改变。今天我刚解开新版 Prism 的时候，发现之前改的文件都不见了，以为代码全部重写了，结果全局搜索发现，只是位置发生改变。不知道 HTC 搞的什么鬼……