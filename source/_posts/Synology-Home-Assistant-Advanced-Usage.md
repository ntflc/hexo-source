---
title: 群晖 Home Assistant 进阶指南
date: 2023-12-31 13:00:35
categories:
- NAS
tags:
- NAS
- Synology
- HomeAssistant
---

21 年的时候，给群晖装了 Home Assistant，接入了几个米家设备和台式机 WOL，当时还写了篇博客 [群晖安装 Home Assistant 及配置（接入米家及 WOL）](/2021/06/27/Synology-Install-Home-Assistant-and-Configure/)。两年过去了，Home Assistant 升级了不少内容（比如内置了 Xiaomi Miio，不再需要手动改配置文件），我也住进了新家，有不少新设备需要接入。借着最近再次折腾，再来写篇进阶指南。

<!-- more -->

# 升级 Home Assistant 版本

之前在群晖上安装的是 Docker 版的 Home Assistant，无法使用自带的版本升级。但其实通过 Docker 升级也不麻烦：

1. 打开 Container Manager，点击左侧 Image。如果有新版本，直接点击 Update 下载最新镜像。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/1.png)
   如果你的群晖或 Container Manager 版本较低，不支持 Image 更新功能，也可以点击左侧 Registry，搜索 `homeassistant`，重新下载最新镜像。
2. 点击左侧 Container，右击 Home Assistant 对应容器，点击 Stop 停止容器。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/2.png)
3. 待容器停止后，点击 Reset 重置容器。重置容器并不会丢失 Home Assistant 的数据。
4. 待重置完成，点击 Start 启动容器。

至此，Home Assistant 就升级完毕了。

# 安装 HACS

HACS (Home Assistant Community Store)，是 Home Assistant 的第三方应用商店，支持大量第三方插件。官网是 [https://hacs.xyz/](https://hacs.xyz/)，上面也有安装教程。我这里采用的是手动安装，你也可以按照官网的文档直接运行 SSH 脚本安装。

1. 在 Home Assistant 配置根目录（`/docker/homeassistant`）创建 `custom_components` 目录。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/3.png)
2. 在 [https://github.com/hacs/integration/releases](https://github.com/hacs/integration/releases) 下载最新版本的 `hacs.zip`，将解压后的 `hacs` 文件夹移动到 `/docker/homeassistant/custom_components` 下。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/4.png)
3. 进入 Home Assistant，点击左下角 Settings，进入 Devices & services，点击右下角 ADD INTEGRATION，搜索 `HACS` 并点击。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/5.png)
4. 勾选前 4 项，第 5 项根据个人情况勾选。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/6.png)
5. 点击链接，输入 8 位验证码，授权 GitHub。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/7.png)
   ![](/images/Synology-Home-Assistant-Advanced-Usage/8.png)
6. 完成后看到如下画面，HACS 就安装完毕了。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/9.png)
7. 刷新页面，左侧应该能看到 HACS 入口了。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/10.png)

# 安装 Xiaomi Miot Auto

虽然 Home Assistant 已经内置了 Xiaomi Miio，但支持的设备有限。Xiaomi Miot Auto 是一款第三方插件，基于 miot 协议，可以将大部分米家设备（不局限于小米）自动接入 Home Assistant。该项目的 GitHub 地址：[https://github.com/al-one/hass-xiaomi-miot](https://github.com/al-one/hass-xiaomi-miot)，里面有中文文档。这里简单讲下如何通过 HACS 安装。

1. 进入 Home Assistant，点击左侧 HACS，进入 Integrations，点击右下角 EXPLORE & DOWNLOAD REPOSITORIES，搜索 `Xiaomi Miot Auto` 并点击，最后点击右下角 DOWNLOAD 下载。
2. 点击左下角 Settings，进入 Devices & services，点击右下角 ADD INTEGRATION，搜索 `Xiaomi Miot Auto` 并点击。
3. Action 建议选择第一项 `Add devices using Mi Account (账号集成)`。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/11.png)
4. 输入米家账号、密码，选择对应区域，Connection mode 选择 `Automatic (自动模式)`，并提交。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/12.png)
5. Filter cloud devices 可根据个人情况选择。如果想接入大部分设备，可选择 `Exclude (排除)` 并勾选排除掉的设备；如果只想接入个别设备，可选择 `Include (包含)` 并勾选对应设备。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/13.png)
6. 回到 Integrations，可以在 Configured 中看到 Xiaomi Miot Auto 模块和对应设备了。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/14.png)

# 更新 HomeKit 设备

Home Assistant 新增或删除设备后，HomeKit 中的设备并不会自动更新。这里需要手动更新下 HomeKit Bridge 的配置。

1. 进入 Home Assistant，点击左下角 Settings，进入 Devices & services，点击 HomeKit Bridge。
2. 点击 CONFIGURE，HomeKit Mode 选择 `bridge`，Inclusion Mode 根据个人情况选择。
   ![](/images/Synology-Home-Assistant-Advanced-Usage/15.png)
   ![](/images/Synology-Home-Assistant-Advanced-Usage/16.png)
3. 选择需要接入或排除的设备，点击 SUBMIT。
4. 打开 iOS 的 Home App，此时设备已更新为上一步选择的设备。

---

至此，本文告一段落。折腾 Home Assistant 还是挺有乐趣的，但如果不想折腾，还是尽量买支持 HomeKit 的设备吧。这里列一下本人购买的智能家居设备：

- 支持 HomeKit 的：
  - Aqara 空调伴侣 P3
  - Aqara 智能窗帘电机 Zigbee
  - Aqara 智能管状电机
  - 小米智能门锁 X
  - Yeelight 光璨智能吸顶灯
- 不支持 HomeKit 但可以通过 Home Assistant 接入的：
  - 石头扫拖机器人 G20
  - Yeelight 智能浴霸 Pro（S20）
  - Yeelight 智能凉霸
  - Yeelight 皓白智能 LED 面板灯
  - 米家智能晾衣机 1S

{% gp 2-2 %}
![](/images/Synology-Home-Assistant-Advanced-Usage/17.jpg)
![](/images/Synology-Home-Assistant-Advanced-Usage/18.jpg)
{% endgp %}
