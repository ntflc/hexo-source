title: Sense 7 常用修改：ACC配置
date: 2015-12-27 15:04:07
categories:
- 教程
tags:
- 教程
- Sense
---

HTC Sense 将很多选项放在`/system/customize`下，这样就可以不用修改apk来修改很多东西了。

这里主要谈谈`/system/customize/ACC`下的修改。这个目录下，均为xml文件，主要是default.xml。接下来我会列举经常需要修改的项，只能想到哪个罗列哪个，其实ACC下很多都可以根据变量名来判断的。

<br>

<!-- more -->

# nfc_show_icon #

``` xml
<item type="integer" name="nfc_show_icon">1</item>
```

功能：状态是否显示NFC图标

取值：1为开启NFC时显示NFC图标，0为开启NFC时不显示NFC图标。（不开启NFC时都不显示）

建议：一般推荐修改为0，我是一直开启NFC的，如果一直显示NFC图标很占地方

<br>

# enablePinyin、supportBlacklist、supportIpDial #

``` xml
<item type="boolean" name="enablePinyin">false</item>
<item type="boolean" name="supportBlacklist">false</item>
<item type="boolean" name="supportIpDial">false</item>
```

功能：电话是否支持拼音、黑名单、IP拨号

取值：false为不支持，true为支持

建议：随便，我一般会改，虽然并用不到

<br>

# cityidinfo_support_type #

``` xml
<item type="integer" name="cityidinfo_support_type">0</item>
```

功能：是否开启归属地显示

取值：1为开启归属地显示，其他值为不开启

建议：建议设为1，同时还需要配合CallerLocation.apk，这个国行系统里会有，也可以采用我ROM里的修改版

<br>

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
……
```

建议：根据个人情况修改，考虑到大多数用户喜欢隐藏虚拟按键按键，所以建议添加hide_nav这一项

<br>

# region、sku_id #

``` xml
<item type="integer" name="region">6</item>
<item type="integer" name="sku_id">42</item>
```

功能：region为区域码，sku_id为运营商码

取值：见附件（两个Excel文件）：[http://pan.baidu.com/s/1sj2mazN](http://pan.baidu.com/s/1sj2mazN)

建议：一般我都是按照欧洲或台湾地区的值来取，即6、42或9、73，其实除了设为国行地区码和运营商sku_id，其他都差不多。如果地区码设为3，则会触发国行特性，比如国内天气源、HTC登录提供QQ、微博登陆、H Plus网络显示H+、不联网时不隐藏网络图标而且底下多一个勾等。如果运营商码设为26则为中国移动，4G是双信号，对应SGLTE；如果设为27则为中国电信，默认双信号，对应SVLTE；如果设为29则为中国联通，4G图标为4GLTE。

<br>

# quicklaunch_flashlight #

``` xml
<item type="boolean" name="quicklaunch_flashlight">false</item>
```

功能：长按电源键出来的电源选项里是否显示手电筒

取值：true为显示手电筒，false为不显示

建议：根据个人情况设置

<br>

# allapps_style_enabled_deafult #

``` xml
<item type="boolean" name="allapps_style_enabled_deafult">true</item>
```

功能：默认桌面格式，即是否显示所有应用页（应用抽屉）

取值：true为默认显示所有应用页，false为默认不显示所有应用页。无论选择什么，均可长按桌面空白处切换。但如果手动开启China Sense，会根据此值重新确定是否显示所有应用页。

建议：反正我是设为true，没有所有应用页不能忍

<br>

# show_4g_for_lte #

``` xml
<item type="boolean" name="show_4g_for_lte">false</item>
```

功能：4G图标显示为4G或LTE

取值：true为4G，false为LTE

建议：根据个人喜好选择

<br>

# china_sense #

``` xml
<item type="boolean" name="support_china_sense_feature">true</item>
<item type="boolean" name="support_china_sense_typeb_feature">true</item>
<item type="boolean" name="support_china_sense_typeb_new_feature">true</item>
```

功能：是否开启 China Sense

取值：true为开启，false为不开启

建议：China Sense 还是建议开启的

<br>

# vibration_duration #

``` xml
<item type="integer" name="vibration_duration">20</item>
```

功能：单次振动时间，单位毫秒

取值：数值，越大振感越明显

建议：默认即可