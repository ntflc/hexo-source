---
title: 在 Ubuntu 下安装 STF
date: 2016-08-23 14:02:57
categories:
- 测试
tags:
- 测试
- STF
---

![](/images/Install-STF-in-Ubuntu/1.png)

STF (Smartphone Test Farm) 是 GitHub 上的一个开源项目（[https://github.com/openstf/stf](https://github.com/openstf/stf)），是一个通过浏览器来控制、调试手机的 WEB 软件。只需要一台 Linux 服务器和数台手机，就可以通过浏览器远程操作这些手机了。对于一个公司的测试部门来说，STF 可以有效解决手机多、难以管理的问题。

<!-- more -->

![](/images/Install-STF-in-Ubuntu/2.jpg)

![](/images/Install-STF-in-Ubuntu/3.gif)

STF 可以直接通过 NPM 安装，或者也可以通过 Docker 来安装。本文只提供 NPM 安装的方法，关于 Docker 的，可以参考这篇文章《[STF 开发环境搭建与制作 docker 镜像过程](https://testerhome.com/topics/5206)》。本文使用的Ubuntu 版本为 14.04 LTS。

# 具体步骤

## 环境要求

根据 STF 官方文档，环境需求如下：

- [Node.js](https://nodejs.org/) >= 0.12
- [ADB](http://developer.android.com/tools/help/adb.html) properly set up
- [RethinkDB](http://rethinkdb.com/) >= 2.2
- [GraphicsMagick](http://www.graphicsmagick.org/) (for resizing screenshots)
- [ZeroMQ](http://zeromq.org/) libraries installed
- [Protocol Buffers](https://github.com/google/protobuf) libraries installed
- [yasm](http://yasm.tortall.net/) installed (for compiling embedded [libjpeg-turbo](https://github.com/sorccu/node-jpeg-turbo))
- [pkg-config](http://www.freedesktop.org/wiki/Software/pkg-config/) so that Node.js can find the libraries

下面会介绍下每个组件的安装方法。为了方便，这里全部使用`apt-get install`命令安装，而不使用自编译的方法。

### 1、Node.js

Ubuntu 14.04 下直接用`sudo apt-get install nodejs`获取的 Node.js 版本较低，因此这里使用 Node.js 官方提供的方法安装：

``` sh
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
```

这样安装完后，NPM 也一并安装了。

### 2、ADB

adb 可以将 Android SDK 的路径加到 PATH 中，也可以直接安装：

``` sh
sudo apt-get install android-tools-adb
```

### 3、RethinkDB

RethinkDB 是 STF 服务的数据库，安装方法为：

``` sh
source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install rethinkdb
```

### 4、GraphicsMagick

这个直接`apt-get install`就行了：

``` sh
sudo apt-get install graphicsmagick
```

### 5、ZeroMQ

这个也是直接`apt-get install`就行了：

``` sh
sudo apt-get install libzmq3-dev
```

### 6、Protocol Buffers

这个也是直接`apt-get install`就行了：

``` sh
sudo apt-get install libprotobuf-dev
```

### 7、yasm

这个也是直接`apt-get install`就行了：

``` sh
sudo apt-get install yasm
```

### 8、pkg-config

这个 Ubuntu 14.04 已经自带了，如果需要手动安装，还是`apt-get install`就行了：

``` sh
sudo apt-get install pkg-config
```

### 9、g++

这个是 STF 文档里没有提到的，Ubuntu 14.04 也不自带，但没有这个，后面安装 STF 会出问题：

``` sh
sudo apt-get install g++
```

## 安装

当上面的环境全部配置完成后，就可以开始安装 STF 了：

``` sh
sudo npm install -g stf
```

默认安装位置是`/bin/lib/node_modules/stf`或`/bin/local/lib/node_modules/stf`。

## 运行

首先运行 RethinkDB:

``` sh
rethinkdb
```

然后运行 STF：

``` sh
stf local
```

或

``` sh
stf local --public-ip <your_internal_network_ip_here>
```

之后就可以通过`http://localhost:7100`或`http://<your_internal_network_ip>:7100`来访问了。

当然，你也可以使用`nohup &`的方式运行，这样就不用一直开着终端了。

## 更新

再次执行安装命令即可：

``` sh
sudo npm install -g stf
```

如果存在问题，把 stf 目录删除再执行上面命令即可。

