title: MaximusHD 5.0 本地化
date: 2015-06-24 20:18:26
categories:
- ROM
tags:
- ROM
- MaximusHD
- M9
---

# ROM 引读 #
- 感谢 大款小总、HTC-G11-black、溜达会儿、519ljh、Dave、久龙颜、Billgates-li、kufeen 等人的支持与帮助。
- 感谢 MaximusHD 原作者 LlabTooFeR。
- MaximusHD 原版地址：[http://forum.xda-developers.com/one-m9/development/rom-maximushd-1-0-0-android-5-0-2-sense-t3076909](http://forum.xda-developers.com/one-m9/development/rom-maximushd-1-0-0-android-5-0-2-sense-t3076909)

# ROM 信息 #
> ROM 名称：MaximusHD 5.0.0 本地化
> ROM 版本：Android 5.1 + Sense 7.0
> 基于版本：MaximusHD 5.0.0
> 适配机型：HTC One M9 单卡国际版
> ROM 作者：越昂超英（ntflc）
> 发布时间：2015 年 6 月 24 日

<!-- more -->

# 支持版本 #
- <strong><font color=red>本 ROM 适用于 HTC One M9 单卡国际版（htc_himauhl）</font></strong>，包括美版（无锁版、开发者版、AT&T 版、T-Mobile 版）、欧版、亚太版、港版、台版、国行联通版等单卡通用版。本人测试机型为台版。
- **<font color=red>不支持 Sprint 版、Verizon 版等非国际版本。</font>**

# 注意事项 #
1. 请使用 TWRP Recovery 2.8.6.1 或 2.8.7.0 刷入本 ROM。
2. 第一次开机会执行升级，需要一定时间，请提前保证电量充足。
3. 开机后，请进入 Wi-Fi，点击右上角菜单键，选择高级，然后关闭始终扫描。
4. 如果你没有选择清除 Google 服务相关组件，建议第一次开机时将 SIM 卡移除以跳过 Google 验证。

## 固件要求 ##
- **<font color=#4169e1>由于本 ROM 基于 2.7.401.1，所以需要提前将固件版本升级至 2.7.xxx.x。不升级固件可能会导致各种问题！</font>**
- **<font color=#4169e1>由于 MaximusHD 原作者 LlabTooFeR 只提供了未签名的 2.7.401.1 固件，所以只有 S-OFF才 能刷入：[http://pan.baidu.com/s/1jGJ4Nzw](http://pan.baidu.com/s/1jGJ4Nzw)</font>**
- **<font color=#4169e1>刷固件的教程请看这里：[http://bbs.gfan.com/android-7943814-1-1.html](http://bbs.gfan.com/android-7943814-1-1.html)。</font>**

# ROM 介绍 #
## 6 月 24 日更新 MaximusHD 5.0.0 整包 ##
- **<font color=red>原版更新内容：</font>**
- <font color=red>基于 Android 5.1，部分新 UI</font>
- <font color=red>加入色温调节（设置—显示和手势—色温）</font>
- <font color=red>加入多用户选项</font>
- <font color=red>加入手电筒</font>
- **<font color=red>本地化更新内容：</font>**
- <font color=red>Aroma 刷机选项里提供更多选择：</font>
- <font color=red>是否移除 Google 组件</font>
- <font color=red>是否开启 China Sense</font>
- <font color=red>是否开启来电、去电自动录音</font>
- <font color=red>是否显示 H+（HSPAP）</font>
- <font color=red>是否显示 4GLTE 代替 4G</font>
## 本地化修改内容 ##
- 【系统】
- 去除安装 apk 时的签名校验
- 默认允许安装来自未知源的应用
- Aroma 刷机选项中可选移除 Google 组件
- Aroma 刷机选项中可选开启 China Sense 特性，具体如下：
- 拨号、相册、时钟等自带软件变为 China Sense 风格（短信由于暂不完善，暂时强制关闭了 China Sense 特性）
- 拨号支持公共黄页
- 桌面任意位置下滑显示下拉通知
- 下拉快速设置里增加亮度条
- 关闭下拉通知按键移到通知最下方
- 多任务界面显示 RAM 占用情况和一键清理按钮，支持任务锁定
- ……
- *注意：如果开启 China Sense，所有应用默认全部显示在桌面，如果不喜欢这种样式，请长按桌面空白处或者进入设置—个性化，选择更改全部应用程序样式*
- 【状态栏】
- 去除不联网时小对勾
- 打开联通 H+（HSPAP）显示（需在 Aroma 选项中选择开启）
- 更改 4G 图标为 4GLTE（需在 Aroma 选项中选择开启）
- 【通话】
- 加入来去电归属地显示
- 归属地数据库为国行 M9 内置数据，在此基础上增加运营商和服务号显示（感谢 @Dave 提供的修改思路和服务号数据）
- 支持来电、去电自动录音（需在 Aroma 选项中选择开启）
- 支持 IP 拨号、黑名单
- 【桌面】
- 强制修改 BlinkFeed 新闻源为国内
- BlinkFeed 图片铺满
- 主题直接用邮箱登陆即可下载，不再被墙
- 【音乐】
- 加入国行音乐组件，封面图、歌词自动连接酷我源
- 【其他】
- 更换天气定位为国行位置组件，解决 Google 定位无法使用的问题
- 由于国内天气源问题较多，故天气源还是默认的 ACCU
- 修改 GPS 配置为国行默认配置
- 加入高级电源管理（APM）
- 加入国内主流邮件支持
- 修改机型识别为 HTC M9w，关于微博尾巴的问题请看这个帖子：[http://bbs.gfan.com/android-7964351-1-1.html](http://bbs.gfan.com/android-7964351-1-1.html)
- 加入国行微博、微信组件，自行安装微博、微信后可在 BlinkFeed 中显示
- 【语言】
- 精简语言选择 [English (United States)、中文（简体）、中文 繁体（台湾）]
- 精简键盘选择（English、拼音）
- 开机默认显示中文
- 【内置软件】
- data/app 下共有 4 个程序，为推广软件，不喜可直接卸载
- 四个推广软件分别为猎豹清理大师、百度手机助手、搜狗输入法、UC 浏览器，如果支持我，请至少插卡联网运行一次
- 由于 M9 多了一个 apppreload 分区，当数据清空时，有一定几率恢复该分区的应用。因此，刷完后本 ROM 后可能会出现以上没有列出的应用，但可以卸载，这类软件就是 apppreload 分区的，为 HTC 预装。（如：欧版为 Facebook 和 HTC 社区）
## 如何刷入 ##
- **首先确保已官解或 S-OFF、固件版本已经升级为 2.17.xxx.x（具体看上方蓝字部分）**
- **刷入 TWRP Recovery 2.8.7.0**
- **将下载的 ROM 放入内置存储或外置存储**
- **进入 TWRP Recovery**
- **无需进入 Wipe 选项执行清除数据，直接点击 Install 并选择 ROM 包开刷**
- **根据 Aroma 界面的提示一步一步操作（其中有一步是数据清除，建议选择，否则可能会导致各种问题）**
- **刷机完毕！**
- **（注意：如果非 S-OFF 用户刷完无法开机，请单刷 boot.img）**

# BUG 相关 #
- 暂未发现

# 使用须知 #
- 本 ROM 基于 HTC One M9 单卡国际版制作，请核对您的机型后酌情刷入
- 本固件已经过本人实机测试，如您未注意本文中提到的相关注意事项，所造成的一切后果与作者无关
- 本固件为个人兴趣爱好所做，欢迎其他论坛注名转载，严禁不注明原作者的恶意转载
  **<font color=red>转载时请保留本人网盘地址（以防后续更新等问题），禁止保存到自己网盘发布</font>**
  如有发现未经原作者许可盗用本固件进行非法的或其他盈利性质的用途，本人保留追究版权责任的权利
- 如果您有什么更好的建议或您在使用过程中遇到什么问题请通过新浪微博 [@越昂超英](http://weibo.com/412070391) 联系我
- 最后提醒大家：刷机有风险，选择需谨慎

# ROM 下载 #
**<font color=red>请确认分享网盘的用户名为 412070391。</font>**

[http://pan.baidu.com/s/1Hdmhw](http://pan.baidu.com/s/1Hdmhw)

下载后请注意校验文件信息，文件校验信息放在网盘里。

# ROM 截图 #
![](http://i.imgur.com/hUFh9xF.png)
![](http://i.imgur.com/durHOBG.png)
![](http://i.imgur.com/wsYX8r9.png)
![](http://i.imgur.com/EF39U60.jpg)
![](http://i.imgur.com/jqc5e8M.png)
![](http://i.imgur.com/ErKyNtu.png)
![](http://i.imgur.com/YvfkV2D.png)
![](http://i.imgur.com/b2wQte1.png)
