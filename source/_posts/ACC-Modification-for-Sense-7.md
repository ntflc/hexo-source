title: Sense 7 常用修改：ACC 配置
date: 2015-12-27 15:04:07
categories:
- Tutorial
tags:
- Tutorial
- HTC Sense
---

HTC Sense 将很多选项放在 `/system/customize` 下，这样就可以不用修改 apk 来修改很多东西了。

这里主要谈谈 `/system/customize/ACC` 下的修改。这个目录下，均为 XML 文件，主要是 `default.xml`。接下来我会列举经常需要修改的项，只能想到哪个罗列哪个，其实 ACC 下很多都可以根据变量名来判断的。

<!-- more -->

# nfc_show_icon #

``` xml
<item type="integer" name="nfc_show_icon">1</item>
```

功能：状态是否显示 NFC 图标

取值：`1` 为开启 NFC 时显示 NFC 图标，`0` 为开启 NFC 时不显示 NFC 图标。（不开启 NFC 时都不显示）

建议：一般推荐修改为 `0`，我是一直开启 NFC 的，如果一直显示 NFC 图标很占地方

# enablePinyin / supportBlacklist / supportIpDial #

``` xml
<item type="boolean" name="enablePinyin">false</item>
<item type="boolean" name="supportBlacklist">false</item>
<item type="boolean" name="supportIpDial">false</item>
```

功能：电话是否支持拼音、黑名单、IP 拨号

取值：`false` 为不支持，`true` 为支持

建议：随便，我一般会改，虽然并用不到

# cityidinfo_support_type #

``` xml
<item type="integer" name="cityidinfo_support_type">0</item>
```

功能：是否开启归属地显示

取值：`1` 为开启归属地显示，其他值为不开启

建议：建议设为 `1`，同时还需要配合 `CallerLocation.apk`，这个国行系统里会有，也可以采用我 ROM 里的修改版

# navigation_items #

``` xml
<item type="string-array" name="navigation_items">
  <string>back</string>
  <string>home</string>
  <string>recent</string>
</item>
```

功能：默认（即第一次开机）虚拟按键样式，顺序为从左到右，上面这个即为三个按键：返回、主页、多任务

取值：

``` xml
<string>back</string>
<string>home</string>
<string>recent</string>
<string>hide_nav</string>
...
```

建议：根据个人情况修改，考虑到大多数用户喜欢隐藏虚拟按键按键，所以建议添加 `hide_nav` 这一项

# region / sku_id #

``` xml
<item type="integer" name="region">6</item>
<item type="integer" name="sku_id">42</item>
```

功能：`region` 为区域码，`sku_id` 为运营商码

取值：见附件（两个 Excel 文件）：[http://pan.baidu.com/s/1sj2mazN](http://pan.baidu.com/s/1sj2mazN)

建议：一般我都是按照欧洲或台湾地区的值来取，即 `6`、`42` 或 `9`、`73`，其实除了设为国行地区码和运营商 `sku_id`，其他都差不多。如果地区码设为 `3`，则会触发国行特性，比如国内天气源、HTC 账号登录提供 QQ、微博登录、H Plus 网络显示 H+、不联网时不隐藏网络图标而且底下多一个勾等。如果运营商码设为 `26` 则为中国移动，4G 是双信号，对应 SGLTE；如果设为 `27` 则为中国电信，默认双信号，对应 SVLTE；如果设为 `29` 则为中国联通，4G 图标为 4GLTE。

# quicklaunch_flashlight #

``` xml
<item type="boolean" name="quicklaunch_flashlight">false</item>
```

功能：长按电源键出来的电源选项里是否显示手电筒

取值：`true` 为显示手电筒，`false` 为不显示

建议：根据个人情况设置

# allapps_style_enabled_deafult #

``` xml
<item type="boolean" name="allapps_style_enabled_deafult">true</item>
```

功能：默认桌面格式，即是否显示所有应用页（应用抽屉）

取值：`true` 为默认显示所有应用页，`false` 为默认不显示所有应用页。无论选择什么，均可长按桌面空白处切换。但如果手动开启 China Sense，会根据此值重新确定是否显示所有应用页。

建议：反正我是设为 `true`，没有所有应用页不能忍

# show_4g_for_lte #

``` xml
<item type="boolean" name="show_4g_for_lte">false</item>
```

功能：4G 图标显示为 4G 或 LTE

取值：`true` 为 4G，`false` 为 LTE

建议：根据个人喜好选择

# china_sense #

``` xml
<item type="boolean" name="support_china_sense_feature">true</item>
<item type="boolean" name="support_china_sense_typeb_feature">true</item>
<item type="boolean" name="support_china_sense_typeb_new_feature">true</item>
```

功能：是否开启 China Sense

取值：`true` 为开启，`false` 为不开启

建议：China Sense 还是建议开启的

# vibration_duration #

``` xml
<item type="integer" name="vibration_duration">20</item>
```

功能：单次振动时间，单位毫秒

取值：数值，越大振感越明显

建议：默认即可
