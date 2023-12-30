title: Sense 7 常用修改：build.prop 及 clientid 配置
date: 2015-12-27 22:36:04
categories:
- Tutorial
tags:
- Tutorial
- HTC Sense
---

`build.prop` 是 Android 系统中一个类似于 Windows 注册表的配置项，在 `/system` 目录下。

说到 `build.prop`，还得提到另一个文件，即 `/system/customize/clientid/default.prop`，这是 Sense 7 开始才有的配置项，其实内容和 `build.prop` 前若干行一致，但是系统会默认读取这部分的代码。也就是说，如果你只修改了 `build.prop` 前若干行的内容，没有修改这里，系统是不会读取 `build.prop` 内对应内容的。当然，你也可以直接删除 `/system/customize/clientid/default.prop`，这样系统就只读取 `build.prop` 的配置项了。

下面只说 `build.prop` 的修改项，请自行对比 `/system/customize/clientid/default.prop` 与 `build.prop` 的重合部分，将这部分里需要修改的内容在 `/system/customize/clientid/default.prop` 中修改，或者干脆删掉它。

<!-- more -->

# 开机系统语言 #

```
ro.product.locale.language=en
ro.product.locale.region=US
```

将 `en` 和 `US` 分别改为 `zh` 和 `CN`，则第一次开机系统语言为简体中文。

# 机型信息 #

```
ro.product.model=HTC One M9
```

`ro.product.model` 是机型唯一识别信息，微博尾巴、部分软件识别机型都是依据这个值。

对于微博尾巴，像 M8 和之前大部分机型，会直接读取这里的值，匹配对应尾巴。但对于 M9 及之后机型，除了修改这里的值，还需要安装 HTC 官方微博客户端，才能显示正确的尾巴。具体可以看这篇文章：[HTC官方微博客户端，支持M9尾巴显示](/2015/06/15/HTC-Weibo-App/)

建议将这里的值修改为国行机型值，M9 为 `HTC M9w`，注意大小写区别。即：

```
ro.product.model=HTC M9w
```

# 软件版本信息 #

```
ro.product.version=2.11.1405.5
```

前面 CID 配置修改里提到软件版本信息的强制显示修改，这里是可变显示。即如果 CID 配置里 `deviceData1` 下 `sw_number` 的值为 `NA`，则软件版本信息显示等号后面的值；如果不为 `NA`，则显示 `sw_number` 的值。且如果显示此处值，修改后设置里的软件版本信息也会改变。此处值可以为任意信息，不局限于 x.xx.xxx.x，比如：

```
ro.product.version=NTFLC ROM 2.0
```

注意：部分第三方 ROM 通过修改系统代码，禁止用户修改此处值，比如毒蛇 ROM，一旦修改，开机后就无法正常使用，只能重刷 ROM。官方系统没有限制。

# 默认开启 USB 调试等 #

添加：

```
ro.adb.secure=0
ro.secure=0
ro.allow.mock.location=0
ro.debuggable=0
persist.service.adb.enable=1
persist.sys.usb.config=adb
```

# 强制显示/不显示虚拟按键 #

```
qemu.hw.mainkeys=1
```

这一项一般是没有的，需要自行添加。值为 `1` 是强制不显示虚拟按键，值 `0` 为是强制显示虚拟按键。

对于 Sense 7，由于系统提供隐藏虚拟按键按钮，不建议强制隐藏。
