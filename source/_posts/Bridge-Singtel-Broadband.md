---
title: 桥接 Singtel 宽带
date: 2026-08-31 22:09:49
categories:
- Tutorial
tags:
- Singapore
- Singtel
- Broadband
---

太久没有更新博客了（年度总结除外）。最近看了不少别人的博客，不少都是 00 后，感觉年轻真好。想了想其实这些年也折腾了不少东西，接下来应该会陆续分享一些。

今天先从如何桥接新加坡 Singtel 宽带开始。

<!-- more -->

# 新加坡宽带选择

到了 2026 年，新加坡宽带运营商选择变得愈加丰富，除了 Singtel、StarHub、M1，还有 MyRepublic、ViewQwest、Whizcomms、SIMBA，甚至 Eight 和 GOMO 也加入了战局。而且目前新办宽带，3Gbps 已经是入门选择，像 SIMBA、Eight、GOMO 都只提供 10Gbps 了。

我 24 年来新加坡时，Singtel 还是第一大宽带运营商。到了 26 年，这个名头已经被 StarHub 抢过去了。

关于宽带运营商的选择，建议参考本地论坛 HardwareZone 的[这篇](https://forums.hardwarezone.com.sg/threads/official-readme-first-2026-sg-isp-comparison-latest-promo-deals.6665380/)帖子。

# ONT 和 ONR

光纤宽带接入家庭后，需要通过光网络设备将光纤信号转为普通以太网信号。在国内，这个设备是光猫；在新加坡，这个设备是 ONT 和 ONR。

ONT（Optical Network Terminal）主要负责完成光纤网络和以太网之间的转换，本身通常不承担家庭网络的路由功能。可以简单理解为：

`光纤 → ONT → 自有路由器`

在这种情况下，公网连接通常直接交给后面的路由器处理，因此用户可以比较自由地使用自己的路由器。

ONR（Optical Network Router）则是在光网络终端的基础上进一步集成了路由功能。除了完成光电转换之外，它通常还负责 NAT、DHCP、防火墙等功能，有些型号甚至直接集成 Wi-Fi。

典型结构变成：

`光纤 → ONR → 自有路由器 / 其他设备`

如果直接把自己的路由器连接在 ONR 后面，两台设备都进行路由和 NAT，就可能形成 Double NAT。对于普通上网通常影响不大，但可能给端口转发、远程访问、游戏主机、VPN 以及部分高级网络配置带来额外麻烦。

因此，希望完全使用自己的路由器管理家庭网络时，通常需要将 ONR 配置为 Bridge Mode。桥接后，ONR 会尽量只承担光网络接入和数据转发功能，将路由工作交给用户自己的路由器。

# Singtel 宽带现状

24 年我刚到新加坡时，Singtel 已经基本只提供 ONR 且不提供桥接服务了，不过还是有一些老套餐可以提供 ONT。到了 26 年，新套餐已经全部变为 ONR，所以如果不想折腾，还是建议选择 StarHub 或其他运营商。

然而，虽然官方不提供桥接服务，民间还是有不少研究桥接的讨论，只要稍微有点动手能力，还是比较简单的。

# Singtel 桥接方法

由于 Singtel 提供的 ONR 设备有好几款，不同设备的桥接方法差别很大，以下仅以 Nokia XS-240X-A 为例。

## 以 support 账号登录 ONR 管理页面

在浏览器输入 [http://192.168.1.254/](http://192.168.1.254/) ，用户名输入 `support`，密码是 ONR 设备上贴的密码的倒置（如标签上的密码是 `12345678`，则输入 `87654321`）。

## 创建 ONR 网络连接

1. 进入 Network - WAN 页面。
2. 点击 `WAN Connection List` 下拉框，选择 `Create One New Connection`。
3. 将 `Connection mode` 的值改为 `Bridge mode`。
4. 勾选 `Enable/Disable`。
5. 勾选 `Enable VLAN`，`VLAN ID` 和 `VLAN PRI` 分别填入 `10` 和 `0`。
6. `LAN Port binding` 勾选 `LAN1`。
7. 点击 `Save`。

![](/images/Bridge-Singtel-Broadband/1.png)

## 设置路由器网络连接

1. 根据提示，断开路由器和 ONR 的连接，重新将 ONR 的 LAN1 连到路由器的 WAN 口。
2. 在路由器 WAN 设置里，将网络类型改为 Automatic IP (DHCP)。

至此，路由器应该能获取到 IPv4 地址了。理论上，如果路由器开启了 IPv6，也能获取到 IPv6 地址。

## 一些额外说明

理论上，也可以不创建新网络连接，而是直接改 `1_TR069_INTERNET_R_VID_10`。但这里建议还是不要修改，而是直接创建新网络连接。

笔者在改桥接后，虽然 IPv4 正常，但 IPv6 一直无法使用（能获取到地址，但连不通）。后来联系 Singtel 客服，甚至专门有技术人员跟进，也一直没解决。过了几个月，突然又能连通了，应该是 Singtel 后台配置的问题。
