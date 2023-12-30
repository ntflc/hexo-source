title: HTC 官方微博客户端，支持 M9 尾巴显示
date: 2015-06-15 11:24:41
categories:
- Resource
tags:
- Resource
- Weibo
- M9
---

不知道是什么原因，直接修改 `ro.product.model=HTC M9w` 并不能显示『来自HTC One (M9)』的尾巴，直到安装了 HTC 官方的微博客户端，才终于显示了 M9 的微博尾巴。
由于 HTC 官方的微博客户端非常老（4.3.0），所以这个客户端的意义仅仅是打开微博尾巴的显示，不建议日常使用（安装后覆盖安装最新版微博即可，尾巴依旧存在）。

<!-- more -->

**注意：**
1. **想要显示微博尾巴，需要提前修改 `/system/build.prop` 和 `/system/customize/clientid/default.prop`（主要是后者）内 `ro.product.model=HTC M9w`，注意大小写和空格。本人的 ROM 已经修改。**
2. **安装 HTC 官方微博客户端需要先卸载新浪官方微博，安装完后，可以覆盖安装新浪官方最新版的微博，并且尾巴还在。**
3. **安装完 HTC 官方微博后，不要用钛备份恢复微博数据，否则微博尾巴还是不会显示。**

下载地址：
[http://pan.baidu.com/s/1bJMge](http://pan.baidu.com/s/1bJMge)
