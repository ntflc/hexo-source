---
title: 修改微博尾巴为“来自 HTC 10”
date: 2016-04-08 22:17:26
categories:
- 教程
tags:
- 教程
- 微博
---

之前发过一篇文章“[HTC官方微博客户端，支持M9尾巴显示](http://ntflc.com/2015/06/15/HTC-Weibo-App/)”，里面教大家如何更改微博尾巴为“来自 HTC One (M9)”。今天教大家如何修改微博尾巴为“来自 HTC 10”（算是正式确定下一代旗舰叫“HTC 10”而不是“HTC One M10”了）。

前提条件：
手机已ROOT。

<!-- more -->

首先，需要安装HTC官方微博，下载地址在上面那篇文章里有。该版本微博比较老，需要先卸载已有微博。待下面操作完成后，可以覆盖安装最新版微博。

然后，使用Root Explorer等软件，编辑`/system/build.prop`和`/system/customize/clientid/default.prop`（如果有`/system/customize/clientid/default.prop`，必须修改此文件），找到

```
ro.product.model=XXX
```

将其值改为`HTC M10u`，即

```
ro.product.model=HTC M10u
```

修改完后重启手机。

最后登录微博，发一条微博确认尾巴正确后，即可覆盖安装最新版微博了。

![](http://ww4.sinaimg.cn/large/68e1aca9jw1f2po6hskooj20u01hcn0w.jpg)

<br>

**常见问题：**

Q：为什么我修改了`ro.product.model`，微博尾巴还是“来自 Android 客户端”？

A：有两种可能，一是没有使用HTC官方微博，直接使用了官方版本微博；二是修改的`ro.product.model`值有问题，必须是`HTC M10u`，注意大小写、空格等。
