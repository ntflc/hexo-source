title: HTC One M9 修改 MID 教程
date: 2015-12-25 23:18:24
categories:
- Tutorial
tags:
- Tutorial
- MID
- M9
---

之前玩过 M7、M8 的应该知道，修改 MID 比较复杂，涉及到修改分区文件。

而根据 xda-developers 上的说法，有 Download mode 的机型，均可以像修改 CID 那样修改 MID 了。

因此本人在使用该方法将手中的台版 M9 修改 MID 成功后，给出简单的教程，供需要的人修改。但是不建议连 adb 命令都不会使用用户尝试！

<!-- more -->

**具体步骤：**

0. **<font color=#4169e1>确保手机已经 S-OFF，没有 S-OFF 不能修改 MID！</font>**

1. 关闭手机，然后长按音量下+电源键，进入 Download mode。

2. 手机连接电脑，电脑端打开 CMD，cd 到 adb 所在目录。（不会 adb 操作不建议修改 MID！！！）

3. 执行：

``` sh
fastboot oem writemid XXXXXXXXX
```

其中 XXXXXXXXX 为 MID 值，9 位。下面给出部分机型对应的 MID：

港版、台版、欧版、联通版：`0PJA10000`

AT&T、开发者版：`0PJA11000`

T-Mobile 版：`0PJA12000`

国际版通用超级 MID：`0PJA1****`

**<font color=red>说明：</font>**

1. **<font color=red>超级 MID 必须保证前 5 位是确定值，即不能有『\*』。因此不存在 `0PJA*****` 这样的 MID！</font>**

2. **<font color=red>非国际版，即前五位不为 0PJA1 的版本，主要是 Sprint、Verizon 版，不建议修改 MID，因为即使修改 MID 刷了国际版资源，也可能导致无法开机，甚至变砖！（本条着重提醒，如果 Sprint、Verizon 等版本用户由于修改 MID 刷国际版资源导致手机变砖，本人概不负责！！！）</font>**

3. **<font color=red>请注意第一位是数字 0，不是字母 O！</font>**

4. 重启 Download mode 即可

<br>

查看 MID 方法：

``` sh
fastboot getvar mid
```
