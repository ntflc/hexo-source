title: Sense 7.0 强制打开H+和4GLTE联网图标显示
date: 2015-06-15 23:23:46
categories:
- 教程
tags:
- 教程
- 联网图标
- Sense
---

用联通的用户应该知道，联通3G主要有3种模式，UMTS、HSPA、HSPAP(HSPA+)，显示在顶栏图标上分别是3G、H、H+。

![](http://i.imgur.com/6AoN2zS.jpg)

而国行手机联通4G则显示的一般不是4G，也不是LTE，而是4GLTE(LTE逆时针旋转90度)。

![](http://i.imgur.com/PvkznyZ.jpg)

而在M8、M9上，我们经常刷xda-developers上的ROM，这些都是针对欧美地区优化的，所以显示HSPAP时，一般都只显示H而不是H+，4G也只是显示4G或LTE。

但有些人偏偏就是喜欢H+和4GLTE标志怎么办？今天讲两个方法，一个很简单，但有副作用；一个稍显复杂，但没有副作用。

<!-- more -->

# 方法一：直接修改region和sku_id #

打开`system\customize\ACC\default.xml`

搜索`region`和`sku_id`

找到

``` xml
<item type="integer" name="region">6</item>
```

和

``` xml
<item type="integer" name="sku_id">42</item>
```

(默认数值不一定相同)

`region`是HTC用于判断国家、地区的数值，`sku_id`是HTC用于判断运营商的数值，这里有个`region`和`sku_id`列表，感兴趣的可以下了研究下：[http://pan.baidu.com/s/1sj2mazN](http://pan.baidu.com/s/1sj2mazN)

一般xda-developers上针对国际版的ROM默认都是`region=6`，`sku_id=42`，因为欧洲的`region`为6，欧洲通用`sku_id`为42(HTC_EUROPE)。而中国的`region`为3，国行联通的`sku_id`为29(HTCCN_CHS_CU)。

因此，我们只要把这里的值改成3和29即可，即：

``` xml
<item type="integer" name="region">3</item>
```

和

``` xml
<item type="integer" name="sku_id">29</item>
```

然后保存，替换相应文件并重启即可。

这个方法修改很简单，甚至在手机上就可以完成(用R.E.管理器编辑)。但这样会出现一个问题，就是国行特性被打开了，比如不联网时，联网图标并不消失，而且底部变成勾；天气源也变成国内源(国内源经常出问题)。因此这个方法是有副作用的。

# 方法二：修改SystemUI.apk文件 #

此方法的前提是，

反编译`SystemUI.apk`

定位到`smali\com\android\systemui\statusbar\policy\HtcGenericNetworkController.smali`

编辑此文件，搜索`isChina`，一般在第四处，找到这样一段代码：

``` smali
:cond_41
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is46sku()Z

move-result v9

if-nez v9, :cond_43

invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->isChina()Z

move-result v9

if-eqz v9, :cond_42

invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is26sku()Z

move-result v9

if-eqz v9, :cond_43

:cond_42
const/16 v9, 0x27

sget v10, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->SKU_ID:I

if-eq v9, v10, :cond_43

invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is40sku()Z

move-result v9

if-eqz v9, :cond_44

.line 1262
:cond_43
sget-object v8, Lcom/android/systemui/statusbar/policy/HtcIcons;->HTC_DATA_HPLUS:[I
```

将

``` smali
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->isChina()Z

move-result v9

if-eqz v9, :cond_42
```

中的`if-eqz`改为`if-nez`，即

``` smali
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->isChina()Z

move-result v9

if-nez v9, :cond_42
```

这样，H+就可以正常显示了。

再搜索`is29sku`，一般在第五处，找到这样一段代码：

``` smali
:cond_5a
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is29sku()Z

move-result v8

if-nez v8, :cond_5b

invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->isLATAM()Z

move-result v8

if-eqz v8, :cond_5c

.line 1343
:cond_5b
sget-object v8, Lcom/android/systemui/statusbar/policy/HtcIcons;->HTC_DATA_4GLTE:[I
```

将

``` smali
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is29sku()Z

move-result v8

if-nez v8, :cond_5b
```

中的`if-nez`改为`if-eqz`，即

``` smali
invoke-static {}, Lcom/android/systemui/statusbar/policy/HtcGenericNetworkController;->is29sku()Z

move-result v8

if-eqz v8, :cond_5b
```

这样，4GLTE就可以正常显示了。

但这个方法也有个限制，就是`region`和`sku_id`分别不能是3和29。原因是，这两处其实是修改HTC_DATA_HPLUS和HTC_DATA_4GLTE的触发条件，本来识别到`region==3`时显示H+，现在改成`region!=3`时显示H+。同理，本来识别到`sku_id==29`时显示4GLTE，现在改成`sku_id!=29`时显示4GLTE。

总之，前一种适合喜欢国行特性的人使用，后一种方法适合不喜欢国行特性且会反编译的人使用。

