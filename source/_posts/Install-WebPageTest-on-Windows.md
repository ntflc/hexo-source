---
title: 在 Windows 下搭建 WebPageTest 私有实例
date: 2017-06-04 17:06:13
categories:
- Test
tags:
- Test
- WebPageTest
---

上篇文章 [在 Windows 下安装、配置 Apache 2.4 和 PHP 7](/2017/06/04/Install-Apache-and-PHP-on-Windows/) 讲了在 Windows 下配置 PHP 环境，主要是为了搭建 WebPageTest 而准备。

[WebPageTest](https://www.webpagetest.org/) 是一项由 Google 开发、支持的开源项目，最初是 [AOL](http://dev.aol.com/) 内部使用的工具，后于 2008 年开源。这是一款用于测试网页性能的工具，简单易用，并且可以自己部署私有实例。这篇文章就简单讲一下私有实例的部署过程。

<!-- more -->

# 下载并启动 WebPageTest

WebPageTest GitHub：[https://github.com/WPO-Foundation/webpagetest](https://github.com/WPO-Foundation/webpagetest)

下载地址：[https://github.com/WPO-Foundation/webpagetest/releases](https://github.com/WPO-Foundation/webpagetest/releases)

下载最新版本即可，这里以 `webpagetest_3.0.zip` 为例。

下载后解压，得到 `www`、`agent` 等目录，其中 `www` 为服务端，`agent` 为测试端。将 `www` 移动到 `D:\xamp` 下，重启 Apache。打开浏览器，访问 `localhost/install` 如下图：

![](http://i1.piimg.com/1949/a067934c6a7504f2.png)

接来下就是根据提示信息安装相关依赖。

# 安装 WebPageTest 依赖

## 安装 FFmpeg

下载地址：[http://ffmpeg.zeranoe.com/builds/](http://ffmpeg.zeranoe.com/builds/)

选择最新版本、64-bit、Static，点击 Download FFmpeg。

解压下载的文件到任意目录，如 `D:\ffmpeg`，将其中的 `bin` 文件夹（即 `D:\ffmpeg\bin`）添加到系统 Path 中。

## 安装 ImageMagick

下载地址：[http://www.imagemagick.org/script/download.php](http://www.imagemagick.org/script/download.php)

下载 Windows Binary Release 下的 `ImageMagick-x.x.x-x-Q16-x64-dll.exe`。

双击安装，安装时注意选中 Add application directory to your system path 和 Install legacy utilities (e.g. convert)。

![](http://i1.piimg.com/1949/16ae3e2db82ef67e.jpg)

## 安装 jpegtran

下载地址：[http://jpegclub.org/jpegtran/](http://jpegclub.org/jpegtran/)

将下载得到的 exe 文件放到任意目录，然后将该目录添加到系统 Path 中。

## 安装 ExifTool

下载地址：[http://www.sno.phy.queensu.ca/~phil/exiftool/](http://www.sno.phy.queensu.ca/~phil/exiftool/)

将下载得到的 exe 文件放到任意目录，然后将该目录添加到系统 Path 中。

注意：不要直接放到 `C:\Windows\system32` 中。

## 安装 Python 2.7

下载地址：[https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)

下载 Latest Python 2 Release - Python 2.7.x 下的 Windows x86-64 MSI installer 版本。

双击安装，安装时注意选择添加系统 Path。

安装后，在CMD中输入 `python`，显示版本信息和 `>>> ` 即为正确。

## 安装 Python 包 Pillow、SSIM

### Pillow

CMD执行：

``` bash
pip install Pillow
```

### SSIM

先下载 numpy（[http://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy](http://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy)）和 scipy（[http://www.lfd.uci.edu/~gohlke/pythonlibs/#scipy](http://www.lfd.uci.edu/~gohlke/pythonlibs/#scipy)），分别选择 `numpy‑x.x.x+mkl‑cp27‑cp27m‑win_amd64.whl` 和 `scipy‑x.x.x‑cp27‑cp27m‑win_amd64.whl`。

然后在 whl 文件路径执行CMD命令安装，注意先后顺序：

``` bash
pip install numpy‑x.x.x+mkl‑cp27‑cp27m‑win_amd64.whl
pip install scipy‑x.x.x‑cp27‑cp27m‑win_amd64.whl
```

安装完后，CMD执行：

``` bash
pip install pyssim
```

----------

以上操作全部完成后，建议重启系统。然后再次通过浏览器访问 `localhost/install`，应该每一项都是勾了。
