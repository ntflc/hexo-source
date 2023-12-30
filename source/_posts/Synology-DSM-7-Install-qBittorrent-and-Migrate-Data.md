---
title: 群晖 DSM 7.0 安装 qBittorrent 及迁移数据
date: 2021-06-14 22:00:42
categories:
- NAS
tags:
- NAS
- Synology
- qBittorrent
---

端午节闲来无事，想着把 DS920+ 升级到 DSM 7.0 RC 版。由于 DSM 7.0 第三方套件需要重新适配才能支持，而我用 NAS 主要是为了 PT 站挂种，最常用套件 qBittorrent Asuka 版（[http://www.gebi1.com/thread-295399-1-1.html](http://www.gebi1.com/thread-295399-1-1.html)）还不支持 DSM 7.0，因此之前一直没升。最近正好看到另一位作者 zhangbo8418 编译了支持 DSM 7.0 的 qBittorrent 套件（[http://www.gebi1.com/thread-297854-1-1.html](http://www.gebi1.com/thread-297854-1-1.html)），才终于想着折腾下 DSM 7.0。之前 Asuka 编译的版本要求必须启用 admin 用户和家目录，配置和数据都在 admin 用户目录下，而 zhangbo8418 编译的版本则是将配置和数据放在套件目录下，因此这里还涉及到数据迁移。

<!-- more -->

# 安装 qBittorrent

安装 qBittorrent 和安装其他第三方套件一样，下载 spk 文件并手动安装即可。

套件地址：[http://www.gebi1.com/thread-297854-1-1.html](http://www.gebi1.com/thread-297854-1-1.html)，下载的时候选 DSM7 目录下的。

下载后，打开套件中心，点击手动安装，选择下载的 spk 文件即可。

# 配置 qBittorrent

qBittorrent 本身的配置这里就不赘述了，网上有很多。这里着重讲一下下载目录的问题。

zhangbo8418 编译版本的 qBittorrent 默认下载地址是共享文件夹的 Download 目录，如果之前下载的文件在其他共享文件夹中，需要手动配置权限，否则 qBittorrent 无法访问。

打开控制面板，点击共享文件夹，右击指定共享文件夹后点击编辑，切换到权限标签页，再将“本地用户”切换为“系统内部用户账号”，找到 qBittorrent，勾选“可读写”，最后保存。

![](/images/Synology-DSM-7-Install-qBittorrent-and-Migrate-Data/1.png)

# 数据迁移

Asuka 编译版本的 qBittorrent 配置和数据均在 `/volume1/homes/admin` 目录下。其中配置的路径是 `/volume1/homes/admin/.config/qBittorrent`，主要包括 `qBittorrent.conf` 文件和 `qBittorrent-data.conf` 文件。数据的路径是 `/volume1/homes/admin/.local/share/data/qBittorrent/BT_backup`，包括种子文件（.torrent）和信息文件（.fastresume，包括 tracker、上传下载量等信息）。

zhangbo8418 编译版本的 qBittorrent 配置和数据均在 `/volume1/@appstore/qBittorrent` 目录下。其中配置的路径是 `/volume1/@appstore/qBittorrent/qBittorrent_conf/config`，数据的路径是 `/volume1/@appstore/qBittorrent/qBittorrent_conf/data/BT_backup`。

路径关系有了后，只需要将配置和数据复制或移动到对应目录即可。但这里会遇到一个问题，群晖套件目录是无法直接在 File Station 中看到的，这里需要用到 SSH 来操作文件。

首先开启 SSH。打开控制面板，点击终端机和 SNMP，勾选“启动 SSH 功能”。

![](/images/Synology-DSM-7-Install-qBittorrent-and-Migrate-Data/2.png)

接着就可以通过 SSH 来访问和操作 NAS 上的目录了。接下来是具体步骤：

1. 在套件中心停用 qBittorrent
2. SSH 到 NAS 上，切换到 ROOT 用户：`sudo -iu root`，不使用 ROOT 用户无法访问套件目录（套件目录的 owner 是套件本身）
3. 复制配置文件：
   ```
   cp /volume1/homes/admin/.config/qBittorrent/*.conf /volume1/@appstore/qBittorrent/qBittorrent_conf/config
   ```
4. 复制数据文件：
   ```
   cp -r /volume1/homes/admin/.local/share/data/qBittorrent/BT_backup/* /volume1/@appstore/qBittorrent/qBittorrent_conf/data/BT_backup
   ```
5. 将上述复制后文件的 owner 改为 qBittorrent：
   ```
   chown qBittorrent:qBittorrent /volume1/@appstore/qBittorrent/qBittorrent_conf/config/*
   chown qBittorrent:qBittorrent /volume1/@appstore/qBittorrent/qBittorrent_conf/data/BT_backup/*
   ```

最后一步是最重要的，如果不修改 owner 信息，启用 qBittorrent 会报错。所有操作完成后，启动 qBittorrent 即可。

好了，到此为止 DSM 7.0 就可以正常使用 qBittorrent 了。其实安装本身很容易，主要是数据迁移（新用户就没这个烦恼了）。对于数据迁移这块，感觉做得有点复杂了，如果有更好的办法欢迎交流。
