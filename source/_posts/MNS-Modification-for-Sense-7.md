title: Sense 7 常用修改：MNS 配置
date: 2015-12-27 22:01:45
categories:
- Tutorial
tags:
- Tutorial
- HTC Sense
---

上篇文章讲的是 `/system/customize/CID` 下的修改，这篇文章主要是 `/system/customize/MNS` 下的。

MNS 目录下也是一个名为 `default.xml` 的文件，主要是一些杂项的修改，比如默认系统地区、时钟地区、浏览器书签、系统输入法设置、邮件配置、时间格式、温度制式等。

下面直接切入重点。

<!-- more -->

# 默认温度制式 #

搜索 `temprature_unit`，找到如下代码：

``` xml
<item name="temprature_unit">f</item>
```

`f` 为华氏度，`c` 为摄氏度。

由于国内使用摄氏度，因此这里一般改为 `c`。

# 世界时钟地区 #

搜索 `weather_provider`，找到类似于下面的代码：

``` xml
    <module name="weather_provider">
      <function name="default_city">
        <set name="plenty" max="15">
          <item name="app">com.htc.elroy.Weather</item>
          <item name="type">1</item>
          <item name="name">Taipei</item>
          <item name="country">Taiwan</item>
          <item name="code">ASI|TW|TW018|TAIPEI</item>
          <item name="timezone">-480</item>
          <item name="timezoneid">Asia/Taipei</item>
        </set>
        <set name="plenty" max="15">
          <item name="app">com.htc.elroy.Weather</item>
          <item name="type">1</item>
          <item name="name">Seattle</item>
          <item name="state">WA</item>
          <item name="country">United States</item>
          <item name="code">NAM|US|WA|SEATTLE</item>
          <item name="timezone">480</item>
          <item name="timezoneid">America/Los_Angeles</item>
        </set>
        <set name="plenty" max="15">
          <item name="app">com.htc.elroy.Weather</item>
          <item name="type">1</item>
          <item name="name">London</item>
          <item name="country">United Kingdom</item>
          <item name="code">EUR|UK|UK124|LONDON</item>
          <item name="timezone">0</item>
          <item name="timezoneid">Europe/London</item>
        </set>
      </function>
    </module>
```

这里每一个 `function` 为一个地区，即时钟应用里，世界时钟选项的显示项。

考虑到一般人不需要默认显示其他地区时间，建议将整段代码删除。或者修改为你希望的默认值。

# 系统自带邮件配置 #

搜索 `name="Mail"`，找到类似于下面的代码：

``` xml
    <module name="Mail">
      <function name="provider">
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Microsoft Exchange ActiveSync</item>
          <item name="inprotocol">EAS</item>
          <item name="description">exchangeAccountDesc</item>
          <item name="resid">icon_launcher_active_sync</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Gmail</item>
          <item name="domain">gmail.com</item>
          <item name="inprotocol">IMAP</item>
          <item name="description">googleAccountDesc</item>
          <item name="resid">icon_launcher_gmail</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Yahoo</item>
          <item name="domain">yahoo.com</item>
          <item name="inprotocol">IMAP</item>
          <item name="description">yahooAccountDesc</item>
          <item name="resid">icon_launcher_yahoo</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Hotmail</item>
          <item name="domain">outlook.com</item>
          <item name="inprotocol">EAS</item>
          <item name="description">outlookcomAccountDesc</item>
          <item name="resid">icon_launcher_outlook</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Other</item>
          <item name="description">otherAccountDesc</item>
          <item name="resid">icon_launcher_mail</item>
        </set>
      </function>
      <function name="provider_setting">
        <set name="plenty" max="10">
          <item name="provider">YahooTW</item>
          <item name="domain">yahoo.com.tw</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Yahoo</item>
          <item name="domain">yahoo.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooHK</item>
          <item name="domain">yahoo.com.hk</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooUK</item>
          <item name="domain">yahoo.co.uk</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooFR</item>
          <item name="domain">yahoo.fr</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooIT</item>
          <item name="domain">yahoo.it</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Yahoo</item>
          <item name="domain">yahoo.de</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooES</item>
          <item name="domain">yahoo.es</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Ymail</item>
          <item name="domain">ymail.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">RocketMail</item>
          <item name="domain">rocketmail.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Ovi</item>
          <item name="domain">ovi.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Rogers</item>
          <item name="domain">rogers.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
      </function>
    </module>
```

这里是系统自带的邮件应用的默认配置，可以参考国行的配置文件修改，这里提供 M9 国行 2.11.1405.5 的配置信息：

``` xml
    <module name="Mail">
      <function name="mail_signature_localization">
        <set name="plenty" max="10">
          <item name="locale">en_US</item>
          <item name="signature">Sent from my HTC Phone</item>
        </set>
        <set name="plenty" max="10">
          <item name="locale">zh_CN</item>
          <item name="signature">发送自我的HTC Phone</item>
        </set>
        <set name="plenty" max="10">
          <item name="locale">zh_TW</item>
          <item name="signature">發送自我的HTC Phone</item>
        </set>
      </function>
      <function name="provider">
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Microsoft Exchange ActiveSync</item>
          <item name="inprotocol">EAS</item>
          <item name="description">exchangeAccountDesc</item>
          <item name="resid">icon_launcher_active_sync</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Sina</item>
          <item name="domain">sina.com</item>
          <item name="inprotocol">POP</item>
          <item name="resid">icon_launcher_mail</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">QQ</item>
          <item name="domain">qq.com</item>
          <item name="inprotocol">POP</item>
          <item name="resid">icon_launcher_mail</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">163</item>
          <item name="domain">163.com</item>
          <item name="inprotocol">POP</item>
          <item name="resid">icon_launcher_mail</item>
        </set>
        <set name="plenty" max="10">
          <!-- %%UAND000308_SAND000313_File Pool%% -->
          <item name="provider">Other</item>
          <item name="description">otherAccountDesc</item>
          <item name="resid">icon_launcher_mail</item>
        </set>
      </function>
      <function name="provider_setting">
        <set name="plenty" max="10">
          <item name="provider">Sina</item>
          <item name="domain">sina.com</item>
          <item name="inserver">pop.sina.com</item>
          <item name="inport">110</item>
          <item name="outserver">smtp.sina.com</item>
          <item name="outport">25</item>
          <item name="inprotocol">POP</item>
          <item name="useSSLin">0</item>
          <item name="useSSLout">0</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">QQ</item>
          <item name="domain">qq.com</item>
          <item name="inserver">pop.qq.com</item>
          <item name="inport">995</item>
          <item name="outserver">smtp.qq.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">POP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">QQ</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">163</item>
          <item name="domain">163.com</item>
          <item name="inserver">pop.163.com</item>
          <item name="inport">110</item>
          <item name="outserver">smtp.163.com</item>
          <item name="outport">25</item>
          <item name="inprotocol">POP</item>
          <item name="useSSLin">0</item>
          <item name="useSSLout">0</item>
          <item name="providerGroup">163</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooTW</item>
          <item name="domain">yahoo.com.tw</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Yahoo</item>
          <item name="domain">yahoo.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooHK</item>
          <item name="domain">yahoo.com.hk</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooUK</item>
          <item name="domain">yahoo.co.uk</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooFR</item>
          <item name="domain">yahoo.fr</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooIT</item>
          <item name="domain">yahoo.it</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Yahoo</item>
          <item name="domain">yahoo.de</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">YahooES</item>
          <item name="domain">yahoo.es</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Ymail</item>
          <item name="domain">ymail.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">RocketMail</item>
          <item name="domain">rocketmail.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Ovi</item>
          <item name="domain">ovi.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
        <set name="plenty" max="10">
          <item name="provider">Rogers</item>
          <item name="domain">rogers.com</item>
          <item name="inserver">htc.imap.mail.yahoo.com</item>
          <item name="inport">993</item>
          <item name="outserver">htc.smtp.mail.yahoo.com</item>
          <item name="outport">465</item>
          <item name="inprotocol">IMAP</item>
          <item name="useSSLin">1</item>
          <item name="useSSLout">1</item>
          <item name="providerGroup">Yahoo</item>
          <item name="htcInternal">1</item>
        </set>
      </function>
      <function name="eas_sync_option">
        <set name="plenty" max="10">
          <item name="deviceModel">HTC M9w</item>
          <item name="deviceFriendName">HTC M9w</item>
          <item name="deviceName">HTCM9w</item>
        </set>
      </function>
    </module>
```

# 浏览器主页和书签 #

这里的浏览器，指的是 HTC 自带浏览器（名为互联网那个），现在台版、开发者版默认已经不带这个应用，改为 Chrome 替代了。

提供国行 2.11.1405.5 的配置信息：

``` xml
    <module name="Browser">
      <function name="bookmark">
        <set name="plenty">
          <item name="title">HTC</item>
          <item name="url">http://www.htc.com</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">百度</item>
          <item name="url">http://m.baidu.com/?from=1526a</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">新浪</item>
          <item name="url">http://sina.cn/?wm=9016_0002</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">腾讯网</item>
          <item name="url">http://3g.qq.com</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">新浪微博</item>
          <item name="url">http://m.weibo.cn/?wm=9016_0002</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">搜狐新闻</item>
          <item name="url">http://m.k.sohu.com</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
        <set name="plenty">
          <item name="title">淘宝</item>
          <item name="url">http://r.m.taobao.com/m3?p=mm_32961662_3375812_10890786&amp;c=1002</item>
          <!--%%UAND000554_SAND000556_Bookmark thumbnail%%-->
        </set>
      </function>
      <function name="homepage">
        <set name="single" max="15">
          <item name="url">http://www.baidu.com</item>
        </set>
      </function>
    </module>
```

# 默认系统时间格式 #

搜索 `defaultTimeFormatSetting`，找到如下代码：

``` xml
    <module name="defaultTimeFormatSetting">
      <function>
        <set name="single">
          <item name="default">MM/d/yyyy</item>
        </set>
      </function>
    </module>
    <module name="defaultTimeFormatSettingShort">
      <function>
        <set name="single">
          <item name="default">MM/d</item>
        </set>
      </function>
    </module>
```

将其改为：

``` xml
    <module name="defaultTimeFormatSetting">
      <function>
        <set name="single">
          <item name="default">yyyy MMM d, EE</item>
        </set>
      </function>
    </module>
    <module name="defaultTimeFormatSettingShort">
      <function>
        <set name="single">
          <item name="default">MMM d, EE</item>
        </set>
      </function>
    </module>
```

# 默认输入法配置 #

搜索 `defaultInputMethod`，找到如下代码：

``` xml
    <module name="defaultInputMethod">
      <function name="default_IME_prediction_set">
        <set name="single">
          <item name="qwerty">yes</item>
        </set>
      </function>
      <function name="default_IME_keyboard_type">
        <set name="single">
          <item name="default">2</item>
        </set>
      </function>
      <function name="default_IKB_settings_checked">
        <set name="single">
          <item name="checked_list_latin">English;France;Spanish;</item>
        </set>
      </function>
      <function name="default_IKB_init_value">
        <set name="single">
          <item name="default">English</item>
        </set>
      </function>
    </module>
```

将其改为：

``` xml
    <module name="defaultInputMethod">
      <function name="default_IME_prediction_set">
        <set name="single">
          <item name="qwerty">yes</item>
        </set>
      </function>
      <function name="default_IME_keyboard_type">
        <set name="single">
          <item name="default">2</item>
        </set>
      </function>
      <function name="default_IKB_settings_checked">
        <set name="single">
          <item name="checked_list">PinYin;Handwriting;Stroke;</item>
          <item name="checked_list_latin">English;</item>
        </set>
      </function>
      <function name="default_IKB_init_value">
        <set name="single">
          <item name="default">PinYin</item>
          <item name="enable_download">no</item>
        </set>
      </function>
    </module>
```

这样，默认提供拼音、手写和笔画三个选择，且默认为拼音输入法。

# 默认系统地区 #

搜索 `defaultLocale`，找到如下代码：

``` xml
<item name="defeault">en_US</item>
```

将值改为 `zh_CN`，则地区默认为中国大陆。