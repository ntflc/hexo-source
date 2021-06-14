title: NTFLC ROM N2.0 | 基于欧版2.10.401.1
date: 2015-07-20 19:09:16
categories:
- ROM
tags:
- ROM
- NTFLC ROM
- M9
---

# ROM引读 #
- 感谢大款小总、HTC-G11-black、溜达会儿、519ljh、Dave、久龙颜、Billgates-li、kufeen等人的支持与帮助。
- 本ROM部分刷机脚本、Aroma主题提取自MaximusHD，感谢MaximusHD原作者LlabTooFeR的付出。
- 感谢LlabTooFeR提供2.10.401.1的DUMP，此版本应该是近期欧版即将推送的版本。


# ROM信息 #
> ROM名称：NTFLC ROM N2.0
> ROM版本：Android 5.1 + Sense 7.0
> 基于版本：欧版最新版本 2.10.401.1
> 适配机型：HTC One M9 单卡国际版
> ROM作者：越昂超英(ntflc)
> 发布时间：2015年7月20日

<!-- more -->

# 支持版本 #
- **<font color=red>本ROM适用于HTC One M9单卡国际版(htc_himauhl)</font>**，包括美版(无锁版、开发者版、AT&T版、T-Mobile版)、欧版、亚太版、港版、台版、国行联通版等单卡通用版。本人测试机型为台版。
- **<font color=red>不支持Sprint版、Verizon版等非国际版本</font>**。

# 注意事项 #
- 1、请使用TWRP Recovery 2.8.6.1或2.8.7.0刷入本ROM。
- 2、第一次开机会执行升级，需要一定时间，请提前保证电量充足。
- 3、开机后，请进入Wi-Fi，点击右上角菜单键，选择高级，然后关闭始终扫描。
- 4、如果你没有选择清除Google服务相关组件，建议第一次开机时将SIM卡移除以跳过Google验证。

## 固件要求 ##
- **<font color=#4169e1>由于本ROM基于2.10.401.1，建议提前将固件版本升级至2.x.xxx.x。理论上，1.40.xxx.x的固件即可正常运行本ROM，但仍然建议升级到最新固件。</font>**
- **<font color=#4169e1>由于MaximusHD原作者LlabTooFeR只提供了未签名的2.10.401.1固件，只有S-OFF才能刷入此固件：[http://pan.baidu.com/s/1wMoOI](http://pan.baidu.com/s/1wMoOI)</font>**
- **<font color=#4169e1>刷固件的教程请看这里：[http://bbs.gfan.com/android-7943814-1-1.html](http://bbs.gfan.com/android-7943814-1-1.html)</font>**

# ROM介绍 #
- **本ROM基于欧版官方(官方即将推送)2.10.401.1**
- 【系统】
- 系统deodex
- 加入root权限
- 加入busybox
- 加入init.d支持
- 去除安装apk时的签名校验
- 默认打开USB调试
- 默认允许安装来自未知源的应用
- 开启外置存储读写权限
- Aroma刷机选项中可选移除Google组件
- Aroma刷机选项中可选开启China Sense特性，具体如下：
- 拨号、相册、时钟等自带软件变为China Sense风格(短信由于暂不完善，暂时强制关闭了China Sense特性)
- 拨号支持公共黄页
- 桌面任意位置下滑显示下拉通知
- 下拉快速设置里增加亮度条
- 关闭下拉通知按键移到通知最下方
- 多任务界面显示RAM占用情况和一键清理按钮，支持任务锁定
- ……
- 【状态栏】
- 打开联通H+(HSPAP)网络显示(需在Aroma选项中选择开启)
- 更改4G网络图标为4GLTE(需在Aroma选项中选择开启)
- 【通话】
- 加入来去电归属地显示
- 归属地数据库为国行M9内置数据，在此基础上增加运营商和服务号显示(感谢@Dave 提供的修 改思路和服务号数据)
- 支持来电、去电自动录音(需在Aroma选项中选择开启)
- 支持IP拨号、黑名单
- 【桌面】
- 强制修改BlinkFeed新闻源为国内
- BlinkFeed图片铺满
- 主题直接用邮箱登陆即可下载，不再被墙
- 【音乐】
- 加入国行音乐组件，封面图、歌词自动连接酷我源
- 【其他】
- 更换天气定位为国行位置组件，解决Google定位无法使用的问题
- 由于国内天气源问题较多，故天气源还是默认的ACCU
- 修改GPS配置为国行默认配置
- 加入高级电源管理(APM)(需在Aroma选项中选择开启)
- 加入电源历史记录时间显示
- 加入国内主流邮件支持
- 加入滑动关屏，左滑或右滑底部虚拟按键即可关闭屏幕(需在Aroma选项中选择开启)
- 修改机型识别为HTC M9w，关于微博尾巴的问题请看这个帖子：[http://bbs.gfan.com/android-7964351-1-1.html](http://bbs.gfan.com/android-7964351-1-1.html)
- 加入国行微博、微信组件，自行安装微博、微信后可在BlinkFeed中显示
- 【语言】
- 精简语言选择[English (United States)、中文 (简体)、中文 繁体(台湾)]
- 精简键盘选择(English、拼音)
- 开机默认显示中文
- 【内置软件】
- 本ROM不含任何推广软件，真正纯净
- 由于M9多了一个apppreload分区，当数据清空时，有一定几率恢复该分区的应用。因此，刷完后本ROM后可能会出现部分第三方应用，但可以卸载，这类软件就是apppreload分区的，为HTC预装，可以用RE管理器进入/preload删除。(如：欧版为Facebook和HTC社区)

## 如何刷入 ##
- **首先确保已官解或S-OFF、固件版本已经升级为2.xx.xxx.x(具体看上方蓝字部分)**
- **刷入TWRP Recovery 2.7.0.0官方版**
- **将下载的ROM放入内置存储或外置存储**
- **进入TWRP Recovery**
- **无需进入Wipe选项执行清除数据，直接点击Install并选择ROM包开刷**
- **根据Aroma界面的提示一步一步操作(其中有一步是数据清除，建议选择，否则可能会导致各种问题)**
- **刷机完毕！**
- **(注意：如果非S-OFF用户刷完无法开机，请单刷boot.img)**

# BUG相关 #
- 开机向导可能会卡在获取联网信息那一步。如果卡住，请长按电源键，选择飞行模式，然后关闭飞行模式即可。

# 使用须知 #
- 本ROM基于HTC One M9单卡国际版制作，请核对您的机型后酌情刷入
- 本固件已经过本人实机测试，如您未注意本文中提到的相关注意事项，所造成的一切后果与作者无关
- 本固件为个人兴趣爱好所做，欢迎其他论坛注名转载，严禁不注明原作者的恶意转载
  **<font color=red>转载时请保留本人网盘地址(以防后续更新等问题)，禁止保存到自己网盘发布</font>**
  如有发现未经原作者许可盗用本固件进行非法的或其他盈利性质的用途，本人保留追究版权责任的权利
- 如果您有什么更好的建议或您在使用过程中遇到什么问题请通过新浪微博[@越昂超英](http://weibo.com/412070391)联系我
- 最后提醒大家：刷机有风险，选择需谨慎

# ROM下载 #
**<font color=red>请确认分享网盘的用户名为412070391。</font>**

[http://pan.baidu.com/s/1gd4dCpd](http://pan.baidu.com/s/1gd4dCpd)

下载后请注意校验文件信息，文件校验信息放在网盘里。

# ROM截图 #
![](http://i.imgur.com/YMf0pM3.jpg)
![](http://i.imgur.com/Yd09ouM.png)
![](http://i.imgur.com/d9odisz.png)
![](http://i.imgur.com/ruwRERW.jpg)
![](http://i.imgur.com/FJA1nE3.png)
![](http://i.imgur.com/YyubfsI.png)
![](http://i.imgur.com/cRmGgxk.png)
![](http://i.imgur.com/aKC0vop.png)
