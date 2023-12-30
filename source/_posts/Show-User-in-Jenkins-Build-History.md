---
title: 在 Jenkins 构建历史中显示启动人
date: 2018-10-10 16:49:25
categories:
- Test
tags:
- Test
- Jenkins
---

最近在使用 Jenkins 时，想在构建历史里显示启动人，网上搜了一圈，发现没有完整一些的教程，于是决定自己写一篇。

<!-- more -->

首先，需要在 Jenkins 中安装 [Groovy Postbuild](https://wiki.jenkins.io/display/JENKINS/Groovy+Postbuild+Plugin) 和 [user build vars plugin](https://wiki.jenkins.io/display/JENKINS/Build+User+Vars+Plugin) 这两个插件。前者是在构建后操作步骤中引入 Groovy Script，从而实现在构建历史中显示参数；后者是将启动人信息放入环境变量中，具体变量可参考：[https://wiki.jenkins.io/display/JENKINS/Build+User+Vars+Plugin](https://wiki.jenkins.io/display/JENKINS/Build+User+Vars+Plugin)。

插件准备完毕后，在项目配置的 `构建环境` 中勾选 `Set jenkins user build variables`，然后在 `构建后操作` 中添加 `Groovy Postbuild`，`Groovy Script`为：

``` groovy
manager.addShortText(manager.envVars['BUILD_USER'])
```

如此配置后，构建完成后，就会在对应构建历史后显示启动人，如下图所示（启动人为 ntflc）：

![](/images/Show-User-in-Jenkins-Build-History/1.jpg)

如果想在启动人前加上文字，`Groovy Script` 可以写成：

``` groovy
manager.addShortText("启动人：${manager.envVars['BUILD_USER']}")
```

效果如下图：

![](/images/Show-User-in-Jenkins-Build-History/2.jpg)

如果不想直接显示文字，而是显示图标，`Groovy Script` 可以写成：

``` groovy
manager.addInfoBadge("启动人：${manager.envVars['BUILD_USER']}")
```

效果如下图（鼠标放在图标上显示文字浮窗，点击弹窗显示文字）：

![](/images/Show-User-in-Jenkins-Build-History/3.jpg)

当然，这里只是最简单的几种情况，灵活运用 `Groovy Script` 可以实现更复杂的效果，具体可以参考：[https://wiki.jenkins.io/display/JENKINS/Groovy+Postbuild+Plugin](https://wiki.jenkins.io/display/JENKINS/Groovy+Postbuild+Plugin)
