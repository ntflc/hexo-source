title: HTC One M9 修改 MID 教程
date: 2015-12-25 23:18:24
categories:
- 教程
tags:
- 教程
- MID
- M9
---

之前玩过M7、M8的应该知道，修改MID比较复杂，涉及到修改分区文件。

而根据xda-developers上的说法，有Download mode的机型，均可以像修改CID那样修改MID了。

因此本人在使用该方法将手中的台版M9修改MID成功后，给出简单的教程，供需要的人修改。但是不建议连adb命令都不会使用用户尝试！

<br>

**具体步骤**

**<font color=#4169e1>0、确保手机已经S-OFF，没有S-OFF不能修改MID！</font>**

1、关闭手机，然后长按音量下+电源键，进入Download mode。

2、手机连接电脑，电脑端打开CMD，cd到adb所在目录。（不会adb操作不建议修改MID！！！）

<!-- more -->

3、执行：

``` sh
fastboot oem writemid XXXXXXXXX
```

其中XXXXXXXXX为MID值，9位。下面给出部分机型对应的MID：

港版、台版、欧版、联通版：0PJA10000

AT&T、开发者版：0PJA11000

T-Mobile版：0PJA12000

国际版通用超级MID：0PJA1\*\*\*\*

**<font color=red>说明：</font>**

**<font color=red>1)超级MID必须保证前5位是确定值，即不能有“\*”。因此不存在0PJA\*\*\*\*\*这样的MID！</font>**

**<font color=red>2)非国际版，即前五位不为0PJA1的版本，主要是Sprint、Verizon版，不建议修改MID，因为即使修改MID刷了国际版资源，也可能导致无法开机，甚至变砖！（本条着重提醒，如果Sprint、Verizon等版本用户由于修改MID刷国际版资源导致手机变砖，本人概不负责！！！）</font>**

**<font color=red>3)请注意第一位是数字0，不是字母O！</font>**

4、重启Download mode即可

<br>

查看MID方法：

``` sh
fastboot getvar mid
```
