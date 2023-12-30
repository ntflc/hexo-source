title: Sense 7 常用修改：CID 配置
date: 2015-12-27 21:34:21
categories:
- Tutorial
tags:
- Tutorial
- HTC Sense
---

上篇文章讲的是 `/system/customize/ACC` 下的修改，这篇文章主要是 `/system/customize/CID` 下的。

CID 目录下也是一个名为 `default.xml` 的文件，一般修改不多，主要是系统语言的设置。下面直接讲重点。

PS：部分地区 RUU 的 ACC、CID 目录下会有 `default.xml` 以外的 XML 文件，名字都是不同地区的 CID，比如 `HTC__001.xml`，系统会根据手机 CID 码（此 CID 非本篇提到的 CID 配置目录）匹配，如果匹配上就读取对应的 XML 文件，如果没有就读取 `default.xml`。一般做第三方 ROM 会考虑删除其他 XML 文件，只留 `default.xml`。

<!-- more -->

# 系统可选语言配置 #

搜索 `<module name="locale">`，找到类似于下面的一段代码：

``` xml
    <module name="locale">
      <function>
        <set name="single">
          <item name="total_list">ar_US;bg_US;cs_US;da_US;de_US;el_US;en_US;es_US;et_US;fa_US;fi_US;fr_US;hr_US;hu_US;in_US;it_US;iw_US;ja_US;kk_US;ko_US;lt_US;lv_US;nb_US;nl_US;pl_US;pt_US;ro_US;ru_US;sk_US;sl_US;sr_US;sv_US;th_US;tr_US;uk_US;vi_US;zh_CN;zh_TW;</item>
          <item type="boolean" name="ar_US">yes</item>
          <item type="boolean" name="bg_US">yes</item>
          <item type="boolean" name="cs_US">yes</item>
          <item type="boolean" name="da_US">yes</item>
          <item type="boolean" name="de_US">yes</item>
          <item type="boolean" name="el_US">yes</item>
          <item type="boolean" name="en_US">yes</item>
          <item type="boolean" name="es_US">yes</item>
          <item type="boolean" name="et_US">yes</item>
          <item type="boolean" name="fa_US">yes</item>
          <item type="boolean" name="fi_US">yes</item>
          <item type="boolean" name="fr_US">yes</item>
          <item type="boolean" name="hr_US">yes</item>
          <item type="boolean" name="hu_US">yes</item>
          <item type="boolean" name="in_US">yes</item>
          <item type="boolean" name="it_US">yes</item>
          <item type="boolean" name="iw_US">yes</item>
          <item type="boolean" name="ja_US">yes</item>
          <item type="boolean" name="kk_US">yes</item>
          <item type="boolean" name="ko_US">yes</item>
          <item type="boolean" name="lt_US">yes</item>
          <item type="boolean" name="lv_US">yes</item>
          <item type="boolean" name="nb_US">yes</item>
          <item type="boolean" name="nl_US">yes</item>
          <item type="boolean" name="pl_US">yes</item>
          <item type="boolean" name="pt_US">yes</item>
          <item type="boolean" name="ro_US">yes</item>
          <item type="boolean" name="ru_US">yes</item>
          <item type="boolean" name="sk_US">yes</item>
          <item type="boolean" name="sl_US">yes</item>
          <item type="boolean" name="sr_US">yes</item>
          <item type="boolean" name="sv_US">yes</item>
          <item type="boolean" name="th_US">yes</item>
          <item type="boolean" name="tr_US">yes</item>
          <item type="boolean" name="uk_US">yes</item>
          <item type="boolean" name="vi_US">yes</item>
          <item type="boolean" name="zh_CN">yes</item>
          <item type="boolean" name="zh_TW">yes</item>
        </set>
      </function>
    </module>
```

这一段就是用来配置系统可选语言的。其中 `total_list` 的值为所有语言，不同语言之间用 `;` 隔开。接来下数行就列出这些语言。

一般我做 ROM 只保留简体中文、繁体中文和英文，所以一般配置如下：

``` xml
    <module name="locale">
      <function>
        <set name="single">
          <item name="total_list">en_US;zh_CN;zh_TW;</item>
          <item type="boolean" name="en_US">yes</item>
          <item type="boolean" name="zh_CN">yes</item>
          <item type="boolean" name="zh_TW">yes</item>
        </set>
      </function>
    </module>
```

当然，这个因人而异，你也可以把所有语言都留着。不过这里的配置仅仅是提供设置中可选的语言，具体显示还得依赖各个 apk，如果 apk 本身没有这个语言支持，则显示默认语言。

# 强制软件版本信息显示 #

搜索 `<module name="locale">`，找到类似于下面的一段代码：

``` xml
    <module name="deviceData1">
      <function>
        <set name="single">
          <item name="sw_number">NA</item>
        </set>
      </function>
    </module>
```

如果 `sw_number` 值为 `NA`，则设置——关于——软件信息——软件版本显示的是 `build.prop` 中的值（具体后面文章会讲），如果不为 `NA`，则会强制显示为该值。强制显示是指，比如这里设为 NTFLC ROM 1.0，则下次不清数据刷值为 NTFLC ROM 2.0 的 ROM，设置里还是显示 NTFLC ROM 1.0。只有清除数据才会显示为新值。一般，国行官方系统会修改这里的值，比如 M9 国行联通版就设置其为 1.0.0.M9w。

一般还是建议将 ROM 版本信息写在 `build.prop` 中，否则用户不清数据升级到新版本，设置里还显示老版本号，会以为自己没升级成功。

# 默认允许安装未知源应用 #

搜索 `SettingsProvider`，找到类似于下面的一段代码：

``` xml
  <category name="SettingsProvider">
    <module name="res">
      <function name="values">
        <set name="single">
          <item name="def_htc_mobile_network_on">true</item>
          <item name="def_2g_on">3</item>
        </set>
      </function>
    </module>
  </category>
```

在

``` xml
        <set name="single">
          ...
        </set>
```

之间添加：

``` xml
          <item name="def_install_non_market_apps">1</item>
```

加上此项后，设置——安全——未知源默认勾选，这样安装非 Google Play 应用就不会提示去设置允许未知源了。
