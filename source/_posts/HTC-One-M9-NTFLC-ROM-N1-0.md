title: NTFLC ROM N1.0 | 基于台版 1.40.709.8
date: 2015-06-17 23:08:01
categories:
- ROM
tags:
- ROM
- NTFLC ROM
- M9
---

# ROM 引读 #
- 感谢 大款小总、HTC-G11-black、溜达会儿、519ljh、Dave、久龙颜、Billgates-li、kufeen 等人的支持与帮助。
- 本 ROM 部分刷机脚本、Aroma 主题提取自 MaximusHD，感谢 MaximusHD 原作者 LlabTooFeR 的付出。
- 这是我第一个基于 RUU 的 ROM，我将其命名为 NTFLC ROM。去年，我还是毒蛇 ROM 的重度使用者，因此去年我基本都是以毒蛇本地化为主的。然而，今年开始，我慢慢不再喜欢毒蛇那种繁杂的工具箱，反倒是觉得 Sense 7.0 本身已经足够好了，不想再做过多修改，因此 M9 开始我都是做的 MaximusHD 本地化，而 MaximusHD 本身是基于欧版 RUU 的。后来想了下，为何非要用别人的底包？于是终于决定基于官方 RUU 做 ROM 了。

# ROM 信息 #
> ROM 名称：NTFLC ROM N1.0
> ROM 版本：Android 5.0.2 + Sense 7.0
> 基于版本：台版官方 RUU 1.40.709.8
> 适配机型：HTC One M9 单卡国际版
> ROM 作者：越昂超英（ntflc）
> 发布时间：2015 年 6 月 17 日

<!-- more -->

# 支持版本 #
- <strong><font color=red>本 ROM 适用于 HTC One M9 单卡国际版（htc_himauhl）</font></strong>，包括美版（无锁版、开发者版、AT&T 版、T-Mobile 版）、欧版、亚太版、港版、台版、国行联通版等单卡通用版。本人测试机型为台版。
- <strong><font color=red>不支持 Sprint 版、Verizon 版等非国际版本</font></strong>。

# 注意事项 #
1. 请使用 TWRP Recovery 2.8.6.4 CT 版或 2.8.6.1 官方版刷入本 ROM。
2. 第一次开机会执行升级，需要一定时间，请提前保证电量充足。
3. 开机后，请进入 Wi-Fi，点击右上角菜单键，选择高级，然后关闭始终扫描。
4. 如果你没有选择清除 Google 服务相关组件，建议第一次开机时将 SIM 卡移除以跳过 Google 验证。

## 固件要求 ##
- **<font color=#4169e1>由于本 ROM 基于 1.40.709.8，所以需要提前将固件版本升级至 1.40.xxx.x。不升级固件可能会导致各种问题！</font>**
- **<font color=#4169e1>如果已经 OTA 到 1.40.xxx.x 或者已经刷过 1.40.xxx.x 固件，可直接刷本 ROM。</font>**
- **<font color=#4169e1>台版 1.40.709.4 固件：[http://bbs.gfan.com/android-7967148-1-1.html](http://bbs.gfan.com/android-7967148-1-1.html)</font>**
- **<font color=#4169e1>台版 1.40.709.8 固件：[http://pan.baidu.com/s/1i3s1ZaX](http://pan.baidu.com/s/1i3s1ZaX)</font>**
- **<font color=#4169e1>欧版 1.40.401.8 固件：[http://pan.baidu.com/s/1jGgojrw](http://pan.baidu.com/s/1jGgojrw)</font>**
- **<font color=#4169e1>刷固件的教程请看这里：[http://bbs.gfan.com/android-7943814-1-1.html](http://bbs.gfan.com/android-7943814-1-1.html)</font>**

# ROM 介绍 #
- **本 ROM 基于台版官方 RUU 1.40.709.8**
- 【系统】
- 加入 busybox
- 加入 init.d 支持
- 加入 root 权限
- 默认打开 USB 调试
- 默认允许安装来自未知源的应用
- 开启外置存储读写权限
- Aroma 刷机选项中可选开启 China Sense 特性，具体如下：
- 拨号、相册、时钟等自带软件变为 China Sense 风格（短信由于暂不完善，暂时强制关闭了 China Sense 特性）
- 桌面任意位置下滑显示下拉通知
- 下拉快速设置里增加亮度条
- 关闭下拉通知按键移到通知最下方
- 多任务界面显示 RAM 占用情况和一键清理按钮，支持任务锁定
- ……
- 【状态栏】
- 加入下拉农历显示，支持年份显示
- 打开联通 H+（HSPAP）显示，去除不联网时小对勾
- 更改 4G 图标为 4GLTE
- 【通话】
- 加入国行最新拨号组件，支持黄页
- 加入来去电归属地显示
- 归属地数据库为国行 M9 内置数据，在此基础上增加运营商和服务号显示（感谢 @Dave 提供的修改思路和服务号数据）
- 支持通话手动或自动录音（手动录音方法：通话时点击菜单—开始录制，其中自动录音需在 Aroma 刷机选项中选择开启）
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
- 加入电源历史记录时间显示
- 加入国内主流邮件支持
- 加入滑动关屏，左滑或右滑底部虚拟按键即可关闭屏幕（需在 Aroma 选项中选择开启）
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
- **首先确保已官解或 S-OFF、固件版本已经升级为 1.40.xxx.x（具体看上方蓝字部分）**
- **刷入 TWRP Recovery 2.8.6.1 官方版**
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

[http://pan.baidu.com/s/1bn11KFd](http://pan.baidu.com/s/1bn11KFd)

下载后请注意校验文件信息，文件校验信息放在网盘里。

# ROM 截图 #
![](http://i.imgur.com/8KZNmoF.png)
![](http://i.imgur.com/76F5OWD.png)
![](http://i.imgur.com/927x6J3.jpg)
![](http://i.imgur.com/KhfFMrv.png)
![](http://i.imgur.com/6F0pxkz.png)
![](http://i.imgur.com/4WOA9Sl.png)
![](http://i.imgur.com/TG0DSzV.png)
