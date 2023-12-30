---
title: 群晖安装 Home Assistant 及配置（接入米家及 WOL）
date: 2021-06-27 15:53:07
categories:
- NAS
tags:
- NAS
- Synology
- HomeAssistant
---

群晖 NAS 买了一年，主要就用来挂挂 PT 站，多少有些暴殄天物。恰好最近想将智能家居从米家迁移到 HomeKit，了解到了 Home Assistant，发现群晖也支持，于是折腾了一番。这里记录一下折腾过程，希望能给爱折腾的人一些帮助。

<!-- more -->

# 安装 Home Assistant

在套件中心安装 Docker 并启动。这一步很简单，就不多介绍了。

打开 Docker，在 `注册表` 中搜索 `homeassistant`，选择第一项 `homeassistant/home-assistant`，然后点击 `下载`，标签选择 `latest`。由于 Home Assistant 镜像较大，下载需要一段时间，请耐心等待。

![](/images/Synology-Install-Home-Assistant-and-Configure/1.png)

![](/images/Synology-Install-Home-Assistant-and-Configure/2.png)

点击左侧 `映像`，选中刚才下载的 `homeassistant/home-assistant:latest`，点击 `启动`。

![](/images/Synology-Install-Home-Assistant-and-Configure/3.png)

接下来的配置可参考如下：

1. 容器名称按个人喜好填写，这里以 `homeassistant` 为例。
![](/images/Synology-Install-Home-Assistant-and-Configure/4.png)
2. 点击 `高级设置`，勾选 `启用自动重新启动`。
![](/images/Synology-Install-Home-Assistant-and-Configure/5.png)
3. 点击顶部 `卷` 标签，然后点击 `添加文件夹`，选择 `docker/homeassistant`，装载路径填写 `/config`。
![](/images/Synology-Install-Home-Assistant-and-Configure/6.png)
4. 点击顶部 `网络` 标签，勾选 `使用与 Docker Host 相同的网络`。
![](/images/Synology-Install-Home-Assistant-and-Configure/7.png)
5. 点击顶部 `环境` 标签，然后点击 `新增`，可变填写 `TZ`，值填写 `Asia/Shanghai`。
![](/images/Synology-Install-Home-Assistant-and-Configure/8.png)

点击 `应用` 后会在 `容器` 中生成一条记录，点击右侧开关标签即可启动 Home Assistant。

![](/images/Synology-Install-Home-Assistant-and-Configure/9.png)

# 配置 Home Assistant

此时启动的 Home Assistant 没有任何设备，这里需要编辑 Home Assistant 配置文件，添加相应设备。

Home Assistant 配置文件的路径是 `/docker/homeassistant/congifuration.yaml`，建议在套件中心安装 `文本编辑器`，即可右击直接在 DSM 中编辑。

![](/images/Synology-Install-Home-Assistant-and-Configure/10.png)

配置文件修改后，可能需要重启 Home Assistant，只需在 Docker 的 `容器` 中点击开关即可。

## 接入米家设备

由于我的部分智能家居设备同时支持米家和 HomeKit，这里着重介绍不支持 HomeKit 的两个设备：米家智能插座和石头扫地机器人。

### 获取米家设备 token

想要通过 Home Assistant 控制米家设备，需要获取米家设备的 token。网上有很多方法介绍，大部分是使用 Android 手机或模拟器安装特定版本的米家，比较麻烦。

这里介绍一个开源项目 [PiotrMachowski/Xiaomi-cloud-tokens-extractor](https://github.com/PiotrMachowski/Xiaomi-cloud-tokens-extractor)，是由 Python 编写的，支持多平台。只需要输入米家账号密码，即可获取所有设备的 ip 和 token。可能有人会担心安全问题，由于项目是开源的，可以自行查看源码。该项目主要是调用米家的接口登录并获取设备的 ip 和 token。如果不放心，也可以根据源码中的接口自行调用。

具体操作就不详述了，总之获取到 token 后，就可以继续添加设备了。

### 添加米家智能插座

在 `congifuration.yaml` 中添加如下配置信息：

``` yaml
switch:
  - platform: xiaomi_miio
    name: "插座"
    host: 192.168.2.10
    token: 1234567890abcdef1234567890abcdef
```

这里配置信息的 `host` 和 `token` 只是示例，实际配置时需改为自己设备的相应信息。说明如下：

- `switch` 为设备类型，这里是开关
- `platform` 为平台，这里需为 `xiaomi_miio`
- `name` 为设备名称，可以自定义，建议简单明了，以便后续 Hey Siri 指令更方便顺口
- `host` 为设备 IP
- `token` 为设备 token

### 添加石头扫地机器人

在 `congifuration.yaml` 中添加如下配置信息：

``` yaml
vacuum:
  - platform: xiaomi_miio
    name: "扫地机器人"
    host: 192.168.2.11
    token: 1234567890abcdef1234567890abcdef
```

同样，这里配置信息的 `host` 和 `token` 只是示例，实际配置时需改为自己设备的相应信息。说明如下：

- `vacuum` 为设备类型，这里是扫地机
- `platform` 为平台，这里需为 `xiaomi_miio`
- `name` 为设备名称，可以自定义，建议简单明了，以便后续 Hey Siri 指令更方便顺口
- `host` 为设备 IP
- `token` 为设备 token

## 接入 WOL

Wake-on-LAN 简称 WOL 或 WoL，中文多译为“网络唤醒”、“远程唤醒”技术。WOL 是一种技术，同时也是该技术的规范标准，它的功效在于让休眠状态或关机状态的电脑，透过局域网的另一台电脑对其发令，使其唤醒、恢复成运作状态，或从关机状态转成引导状态。（本段摘自维基百科）

### 开启 WOL

WOL 的配置主要分为两部分：主板 BIOS 和网卡。

主板 BIOS 由于不同品牌的配置不同，这里不做过多介绍，大家可以根据自己主板的型号搜索。BIOS 配置的关键词是 `WOL` 或 `网络唤醒`，设为开启即可。

网卡配置这里以 Windows 10 系统为例：

1. 右击 `此电脑`，点击 `管理`。
2. 在 `设备管理器` 的 `网络适配器` 中找到网卡设备，右击点击 `属性`。
![](/images/Synology-Install-Home-Assistant-and-Configure/11.png)
3. 点击顶部 `高级` 标签，将属性 `Shutdown Wake-On-Lan` 和 `Wake on Magic Packet` 的值设为 `Enable`。
![](/images/Synology-Install-Home-Assistant-and-Configure/12.png)
4. 点击顶部 `电源管理` 标签，勾选 `允许此设备唤醒计算机`。
![](/images/Synology-Install-Home-Assistant-and-Configure/13.png)

这样 WOL 就配置完了。

### 获取网卡 MAC 地址

获取网卡 MAC 地址的方法很多，可以在路由器、系统中获取，这里以 Windows 10 设置为例。

1. 打开 Windows 10 的系统设置，点击 `网络和 Internet`。
![](/images/Synology-Install-Home-Assistant-and-Configure/14.png)
2. 点击已连接到 Internet 设备的 `属性`。
![](/images/Synology-Install-Home-Assistant-and-Configure/15.png)
3. 滑动到底部，获取 `物理地址(MAC)`。
![](/images/Synology-Install-Home-Assistant-and-Configure/16.png)

### 添加 WOL

在 `congifuration.yaml` 中添加如下配置信息：

``` yaml
switch:
  - platform: wake_on_lan
    name: "电脑"
    mac: "0A:1B:3C:4D:5E:6F"
```

如果之前已经配置过 `switch`，可以直接在之前的 `switch` 里添加第 2 至 4 行。配置信息的 `mac`，只是示例，实际配置时需改为自己设备的相应信息。说明如下：

- `switch` 为设备类型，这里是开关（实际只有开是有效的，关并不会关闭设备）
- `platform` 为平台，这里需为 `wake_on_lan`
- `name` 为设备名称，可以自定义，建议简单明了，以便后续 Hey Siri 指令更方便顺口
- `mac` 为网卡 MAC 地址（注意格式）

---

至此，米家设备和 WOL 都接入成功。此时在 Home Assistant 的概览中就可以看到这些设备了。

![](/images/Synology-Install-Home-Assistant-and-Configure/17.jpg)

# 接入 HomeKit

在 Home Assistant 的 `配置`-`集成` 中，默认会添加 HomeKit。如果没有自动添加，点击右下角 `添加集成`，选择 `HomeKit` 并勾选要包含的域（全选即可）即可。

此时打开 iOS 设备的家庭 App，在 `家庭设置` 的 `中枢与桥接` 中可以看到 Home Assistant 中的设备。在家庭 App 里也可以直接控制这些设备。

{% gp 2-2 %}
![](/images/Synology-Install-Home-Assistant-and-Configure/18.jpg)
![](/images/Synology-Install-Home-Assistant-and-Configure/19.jpg)
{% endgp %}

---

至此，本文就告一段落了。如果家里有 HomePod 设备，此时就可以通过 Hey Siri 控制这些设备了。
