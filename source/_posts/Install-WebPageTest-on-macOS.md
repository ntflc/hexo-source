---
title: 在 macOS 下搭建 WebPageTest 私有实例
date: 2017-09-27 10:31:24
categories:
- 测试
tags:
- 测试
- WebPageTest
---

WebPageTest 最新版本（[17.08](https://github.com/WPO-Foundation/webpagetest/releases/tag/WebPageTest-17.08)）的一个新功能，是增加了一个新的支持多平台的 Agent 端：[wptagent](https://github.com/WPO-Foundation/wptagent)。

> The biggest change by far is a new cross-platform agent (wptagent) that supports Linux, Windows, Mac and Android testing. Eventually wptdriver will be deprecated and all testing will be moved to the new agent.

得益于此，我们终于可以在 macOS 系统下搭建 WebPageTest 环境，不用再依赖 Windows 虚拟机了。而且，新的 wptagent 还支持 Android 设备（无需 root）。本文将介绍 macOS 系统下如何搭建 WebPageTest 服务端、测试端，以及如何使用 Android 设备进行测试。

虽然 macOS 自带 Apache 和 PHP，但我还是习惯于自己配置一套新环境，因此以下教程基于非自带 Apache 和 PHP。

<!-- more -->

---

# 安装 brew

brew（又叫 Homebrew）是 macOS 平台最有名的第三方包管理工具。由于 brew 几乎人人必备，因此这里不做过多介绍。没有用过的可以查看官网：[https://brew.sh/](https://brew.sh/)，里面有中文教程。

# 安装 Apache 2.4

## 添加第三方仓库

由于 Apache 不在 brew 官方仓库中，因此需要添加第三方仓库。终端执行：

``` bash
brew tap homebrew/apache
```

## 关闭并移除自带 Apache 自动加载脚本

终端执行：

``` bash
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

## 安装 Apache

终端执行：

``` bash
brew install httpd24 --with-privileged-ports --with-http2
```

安装需要一段时间，大概1-3分钟，安装后会看到这样的提示：

```
/usr/local/Cellar/httpd24/2.4.27: 212 files, 4.4M, built in 1 minute 45 seconds
```

这里的路径（此处是`/usr/local/Cellar/httpd24/2.4.27`）需要记住，如果你的路径与此不同，则下面的操作需要自行修改路径：

``` bash
sudo cp -v /usr/local/Cellar/httpd24/2.4.27/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
```

## 配置 Apache

安装后，配置文件在`/usr/local/etc/apache2/2.4/httpd.conf`下。修改该文件：

- 174行左右，将`#LoadModule rewrite_module libexec/mod_rewrite.so`取消注释
- 185行左右，将`User daemon`改为`User your_user_name`（`your_user_name`为用户名）
- 186行左右，将`Group daemon`改为 `Group staff`
- 216行左右，将`#ServerName www.example.com:80`改为`ServerName 0.0.0.0:80`
- 240行左右，将`DocumentRoot "/usr/local/var/www/htdocs"`改为`DocumentRoot "/Users/your_user_name/WebPageTest/www"`（`your_user_name` 为用户名，此处路径可自行确定）
- 241行左右，将`<Directory "/usr/local/var/www/htdocs">`改为`<Directory "/Users/your_user_name/WebPageTest/www">`（与上一条的路径一致）
- 261行左右，将`AllowOverride None`改为`AllowOverride All`

## 启动 Apache

1. 创建路径`/Users/your_user_name/WebPageTest/www`
2. 创建`index.html`文件，内容随意，如`<p>Hello World<p>`
3. 在终端中输入`sudo apachectl -k restart`重启 Apache 服务
4. 打开浏览器，输入`localhost`，显示`Hello World`（即第2步的`index.html`）说明 Apache 启动成功

# 安装 PHP 7

## 添加第三方仓库

由于 PHP 不在 brew 官方仓库中，因此需要添加第三方仓库。终端执行：

``` bash
brew tap homebrew/php
```

## 安装依赖

安装 Xcode 的 Command Line Tool。终端执行：

``` bash
xcode-select --install
```

根据提示安装即可。

## 安装 PHP

这里以 PHP 7.1 为例。终端执行：

``` bash
brew install php71 --with-httpd24
```

安装后建立链接：

``` bash
brew link php71
```

## 配置 PHP

安装后，配置文件在`/usr/local/etc/php/7.1/php.ini`下。修改该文件：

- 404行左右，将`memory_limit = 128M`改为`memory_limit = 256M`
- 671行左右，将`post_max_size = 8M`改为`post_max_size = 10M`
- 824行左右，将`upload_max_filesize = 2M`改为`upload_max_filesize = 10M`

以上修改主要是为了满足 WebPageTest 的推荐配置，可根据实际情况修改。

# Apache 支持 PHP

## 添加 PHP 模块

在 Apache 配置文件`/usr/local/etc/apache2/2.4/httpd.conf`的175行左右（即一堆`#LoadModule xxx`后）添加：

``` conf
LoadModule php7_module /usr/local/opt/php71/libexec/apache2/libphp7.so
```

## 添加主页 index.php

在 Apache 配置文件`/usr/local/etc/apache2/2.4/httpd.conf`的273行左右，即：

``` conf
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

中，在`index.html`前添加`index.php`，即：

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

## 添加 PHP 支持

在 Apache 配置文件`/usr/local/etc/apache2/2.4/httpd.conf`的277行左右（即上一步的位置后面）添加：

```
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

## 测试效果

在`/Users/your_user_name/WebPageTest/www`下创建`index.php`，代码为：

``` php
<?php
    phpinfo();
?>
```

重启 Apache，浏览器访问`localhost`看到下图即为配置成功：

![](/images/Install-WebPageTest-on-macOS/1.jpg)

# 下载并启动 WebPageTest

WebPageTest GitHub：[https://github.com/WPO-Foundation/webpagetest](https://github.com/WPO-Foundation/webpagetest)

下载地址：[https://github.com/WPO-Foundation/webpagetest/releases](https://github.com/WPO-Foundation/webpagetest/releases)

下载最新版本即可，这里以 [webpagetest_17.08.zip](https://github.com/WPO-Foundation/webpagetest/releases/download/WebPageTest-17.08/webpagetest_17.08.zip) 为例。

下载后解压，得到`www`、`agent`、`wptagent`等目录，其中`www`为服务端，`agent`为 Windows 测试端（本次不使用），`wptagent`为最新的多平台测试端（也就是本次使用的测试端）。将 `www`和`wptagent`文件夹（前者是服务端，后者是测试端）移动到`/Users/your_user_name/WebPageTest`下，重启 Apache。打开浏览器，访问`localhost/install`如下图：

![](/images/Install-WebPageTest-on-macOS/2.jpg)

接来下就是根据提示信息安装相关依赖。

# 安装 WebPageTest 依赖

## 安装 APC

终端执行：

``` bash
brew install php71-apcu
```

## 安装 FFmpeg

终端执行：

``` bash
brew install ffmpeg
```

如果提示已经安装某个版本，需升级（否则可能导致录视频失败）：

``` bash
brew upgrade ffmpeg
```

## 安装 ImageMagick

终端执行：

``` bash
brew install imagemagick
```

如果提示已经安装某个版本，需升级（否则可能导致截图失败）：

``` bash
brew upgrade imagemagick
```

## 安装 ExifTool

终端执行：

``` bash
brew install exiftool
```

## 安装 Python 2.7 及相关依赖包

macOS 已经自带 Python 2.7.10。但由于自带 Python 默认没有 pip，且为了避免影响系统，这里自行安装 Python 2.7。

### 使用 brew 安装 Python 2.7

1、终端执行：

``` bash
brew install python
```

2、安装完毕后，执行：

``` bash
python --version
```

查看版本是否为2.7.14（即新安装的版本是否替代了系统自带版本）。如果还是2.7.10则需要继续以下步骤

3、终端执行：

``` bash
sudo vim /etc/paths
```

查看`/usr/local/bin`是否在`/usr/bin`前面。如果不在，将`/usr/local/bin`移到`/usr/bin`前面

4、终端执行：

``` bash
ls -l /usr/local/bin | grep python

```

查看是否存在 pip 和 python。如果不存在，记录 pip2 和 python2 的软链接路径，执行：

``` bash
cd /usr/local/bin
ln -s ../Cellar/python/2.7.14/bin/pip2 pip
ln -s ../Cellar/python/2.7.14/bin/python2 python
```

注意，这里的路径一定要以实际情况（即`pip2`和`python2`的软链接路径）为准

5、再次执行：

``` bash
python --version

```
此时版本应该是新安装的版本了。

### 使用 pip 安装依赖包

终端执行：

```
pip install convert compare Pillow pyssim dateutils dnspython monotonic psutil requests ujson
```

# 配置 WebPageTest 服务端

## 设置配置文件

将`www/settings`目录和`www/settings/custom_metrics`目录下`.sample`复制一份，并删除`.sample`后缀。

由于使用非 macOS 自带 Apache 且路径为用户路径，因此无需对 Installation Check 页面 Filesystem 下的文件夹进行权限修改。

## 修改 location.ini

此处提供一个例子，以两个测试端为例：测试端1（Test_PC）为 macOS 平台，包括 Chrome 和 Firefox 浏览器；测试端2（Test_Mobile）为 Android 平台，包括 Chrome 浏览器。

``` ini
; 以下为测试地点，即 WebPageTest 首页 Test Location 中的选项
; key 包括 1、2、3... 和 default
; 1、2、3... 的 value 为对应地点名（不能重复）
; default 的 value 对应默认地点名
[locations]
1=Test_PC
2=Test_Mobile
default=Test_PC

; 以下为测试地点的详情项，locations 中每一个 value 都应该对应这里的一个 section
; key 包括 1、2、3... 和 label、group，label 为该地点的标签名，group 为组信息
; 1、2、3... 的 value 为对应浏览器节点名（不建议直接用浏览器名，以免多个地点的浏览器名重复）
; label 的 value 随意填写，只要不与其他地点的值重复就行（对应 WebPageTest 首页 Test Location 下拉框中每个组下的选项）
; group 的 value 随意填写，可与其他地点的值重复（对应 WebPageTest 首页 Test Location 下拉框中的组）
; 这里 Test_PC 以 Firefox 和 Chrome 为例
[Test_PC]
1=WPT_Firefox
2=WPT_Chrome
label=macOS
group=Desktop

[Test_Mobile]
1=Android_Chrome
label=Android Chrome
group=Mobile

; 以下为浏览器的详情项，测试地点详细项中的每一个浏览器节点名都应该对应这里的一个 section
; key 包括 browser 和 label
; browser 的 value 为浏览器名，同一个浏览器节点下不得重复（对应 WebPageTest 首页 Browser 下拉框中的选项）
; label 的 value 随意填写，只要不与其他 label 重复就行
[WPT_Firefox]
browser=Firefox
label="macOS - Firefox"

[WPT_Chrome]
browser=Chrome
label="macOS - Chrome"

[Android_Chrome]
browser=Chrome
label="Android - HTC One M9"
type=nodejs,mobile
connectivity="WiFi"
```

# 运行 WebPageTest 测试端

## 准备工作

终端执行：

``` bash
sudo visudo
```

此时进入 vim 编辑模式（等同于`sudo vim /etc/sudoers`，但后者即使 sudo 也会提示无编辑权限），找到：

```
root        ALL = (ALL) ALL
%admin      ALL = (ALL) ALL
```

在下面添加一行：

```
your_user_name ALL = (ALL) NOPASSWD:ALL
```

此操作完成后，再执行 sudo 命令就不需要输入密码了。此操作为官网文档推荐操作：

> It is recommended that the agent itself not run as admin/root but that it can elevate without prompting which means disabling UAC on windows or adding the user account to the sudoers file on Linux and OSX (NOPASSWD in visudo).

## macOS 测试端

终端执行：

``` bash
cd ~/WebPageTest/wptagent
python wptagent.py -vvvv --server http://127.0.0.1/work/ --location WPT_Chrome
```

**说明：**

1. wptagent 依赖于 Python 2.7，不支持 Python 3.x
2. `-vvvv`是日志的显示级别，可根据实际情况添加该参数
3. `--server`是必填项，后面跟服务端地址（需要加`http://`且加`/work/`）
4. `--location`是必填项，后面跟服务单配置的地点名（此处应该为浏览器节点名）
5. 执行此命令后，如果开启最高日志级别，会先看到`Waiting for Idle...`，稍等一段时间即可。如果等待时间过长，可以 Ctrl+C 停止，重新执行一次
6. Chrome 和 Firefox 正常安装（不更改名称），无需做任何配置（无需修改`browsers.ini`），wptagent 会自动去相应路径调起浏览器

**注意：**

1. 测试时，如果已经运行了 Chrome 或 Firefox 浏览器，已经打开的标签页会被关闭，请提前做好准备
2. 只有 Chrome 浏览器支持修改 Headers（包括 Script 中`setCookie`命令）

## Android 测试端

终端执行：

``` bash
cd ~/WebPageTest/wptagent
python wptagent.py -vvvv --server http://127.0.0.1/work/ --location Android_Chrome --android
```

**说明：**

1. wptagent 依赖于 Python 2.7，不支持 Python 3.x；Android 设备作为测试端还依赖于 adb，这里不赘述
2. `-vvvv`是日志的显示级别，可根据实际情况添加该参数
3. `--server`是必填项，后面跟服务端地址（需要加`http://`且加`/work/`）
4. `--location`是必填项，后面跟服务单配置的地点名（此处应该为浏览器节点名）
5. `--android`代表测试端是 Android 设备
6. 如果有多台设备（`adb devices`有多个结果），需要带上`--device`参数，后面跟 Android SN 码（即`adb devices`的中的设备编码）
7. Android 端仅 Chrome 支持全功能，需提前安装好 Chrome 浏览器

**注意：**

1、如果报错`AttributeError: 'Process' object has no attribute 'cpu_affinity'`，编辑`wptagent/internal/adb.py`，搜索`proc.cpu_affinity([0])`，将`proc.cpu_affinity([0])`注释，并添加`pass`（注意缩进）

``` python
def start(self):
        """ Do some startup check to make sure adb is installed"""
        import psutil
        ret = False
        out = self.run(self.build_adb_command(['devices']))
        if out is not None:
            ret = True
            # Set the CPU affinity for adb which helps avoid hangs
            for proc in psutil.process_iter():
                if proc.name() == "adb.exe" or proc.name() == "adb" or proc.name() == "adb-arm":
                    # proc.cpu_affinity([0])
                    pass
```

2、如果提示`Device not ready, high temperature`，编辑`wptagent/internal/adb.py`，搜索`if 'temp' in battery and battery['temp'] >`，将`> 36.0`改大，如`> 40.0`

``` python
if 'level' in battery and battery['level'] < 50:
    logging.info("Device not ready, low battery: %d %%", battery['level'])
    is_ready = False
if 'temp' in battery and battery['temp'] > 36.0:
    logging.info("Device not ready, high temperature: %0.1f degrees", battery['temp'])
    is_ready = False
```

同理，如果提示`Device not ready, low battery`，将上面`< 50`改低，如`< 20`

---

参考文档：[在 MacOS Sierra 上安装 Apache 和多个版本的 PHP](https://github.com/nodejh/nodejh.github.io/issues/25)
