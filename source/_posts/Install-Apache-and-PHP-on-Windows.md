---
title: 在 Windows 下安装、配置 Apache 2.4 和 PHP 7
date: 2017-06-04 16:15:45
categories:
- Test
tags:
- Test
- Apache
- PHP
---

最近在调研 H5 性能测试，接触到 [WebPageTest](https://www.webpagetest.org/)。由于搭建 WebPageTest 私有实例需要 Windows 环境的测试 Agent，于是琢磨着干脆服务端也部署在 Windows 上。WebPageTest 是 PHP 编写的，因此要搭建 Apache + PHP 环境，这里以 Apache 2.4 和 PHP 7 为例，简单讲一下 Windows 部署 Apache + PHP。

注：下文使用 Windows 10。

<!-- more -->

# 安装 Apache 2.4

## 安装 Visual C++ Redistributable for Visual Studio 2015

下载地址：[https://www.microsoft.com/en-us/download/details.aspx?id=48145](https://www.microsoft.com/en-us/download/details.aspx?id=48145)

下载后安装即可。

## 下载 Apache

Apache 官网：[https://httpd.apache.org/](https://httpd.apache.org/)

由于官网不提供编译好的安装文件，因此在 [https://httpd.apache.org/docs/current/platform/windows.html#down](https://httpd.apache.org/docs/current/platform/windows.html#down) 页面列出了几个提供编译好安装文件的源，这里建议使用 Apache Lounge。

下载地址：[https://www.apachelounge.com/download/](https://www.apachelounge.com/download/)

下载文件名为 `httpd-2.4.x-win64-VC14.zip` 的文件。

## 配置 Apache

在 D 盘新建文件夹，名为 `wamp`。

解压下载好的 Apache 文件得到 `Apache24` 文件夹，将其放到 `D:\wamp` 下，即 `D:\wamp\Apache24`。

修改 Apache 配置文件，路径为 `D:\wamp\Apache24\conf\httpd.conf`，修改如下内容：

- 37 行左右，修改 `ServerRoot "c:/Apache24"` 为 `ServerRoot "D:/wamp/Apache24"`
- 245 行左右，修改 `DocumentRoot "c:/Apache24/htdocs"` 为 `DocumentRoot "D:/wamp/www"`
- 246 行左右，修改 `<Directory "c:/Apache24/htdocs">` 为 `<Directory "D:/wamp/www">`
- 362 行左右，修改 `ScriptAlias /cgi-bin/ "c:/Apache24/cgi-bin/"` 为 `ScriptAlias /cgi-bin/ "D:/wamp/Apache24/cgi-bin/"`
- 378 行左右，修改 `<Directory "c:/Apache24/cgi-bin">` 为 `<Directory "D:/wamp/Apache24/cgi-bin">`

主要修改内容是将默认配置的 Apache 路径从 `c:/Apache24` 改成实际路径，这里统一以 `D:/wamp/Apache24` 为准。

注意：Windows 正确的路径格式使用的是 `\`，但这里需要使用 `/`。

## 启动 Apache

1. 将 Apache 路径 `D:\wamp\Apache24` 添加到系统 Path 中。
2. 创建路径 `D:\wamp\www`，并创建 `index.html` 文件，内容随意，如 `<p>Hello<p>`。
3. 通过管理员身份打开 `CMD`，输入 `httpd -k install` 安装 Windows 服务。
4. 继续在 `CMD` 中输入 `httpd -k start` 启动 Apache。
5. 打开浏览器，输入 `localhost`，显示 Hello（即第 2 步的 `index.html`）说明 Apache 启动成功。

# 安装 PHP 7

## 下载 PHP

下载地址：[http://windows.php.net/download/](http://windows.php.net/download/)

下载 VC14 x64 Thread Safe 版本。

## 解压 PHP

将下载好的 PHP 文件解压到 `D:\wamp\php7` 下。

## 配置 PHP

将 PHP 路径 `D:\wamp\php7` 添加到系统 Path 中。

复制 `D:\wamp\php7\php.ini-development` 并重命名为 `php.ini`，修改如下内容：

- 738 行左右，修改 `; extension_dir = "ext"` 为 `extension_dir = "D:/wamp/php7/ext"`
- 892 行左右，将需要的扩展 `; extension` 前的 `;` 去掉

这里列出了部分可能会用到的扩展：

```
extension=php_bz2.dll
extension=php_curl.dll
extension=php_fileinfo.dll
extension=php_ftp.dll
extension=php_gd2.dll
extension=php_gettext.dll
extension=php_gmp.dll
extension=php_intl.dll
extension=php_imap.dll
extension=php_openssl.dll
extension=php_sqlite3.dll
```

由于后续 WebPageTest 的需要，这里还需要下载 APC：[http://pecl.php.net/package/APCu](http://pecl.php.net/package/APCu)。

将下载得到的 `php_apcu.dll` 放到 `D:\wamp\php7\ext` 下，然后在 `D:\wamp\php7\php.ini` 中添加：

```
extension=php_apcu.dll
```

# Apache 支持 PHP

## 添加 PHP 模块

在 Apache 配置文件 `D:\wamp\Apache24\conf\httpd.conf` 的 180 行左右（即一堆 `#LoadModule xxx` 后）添加：

```
PHPIniDir "D:/wamp/php7"
LoadModule php7_module "D:/wamp/php7/php7apache2_4.dll"
```

## 添加 PHP 文件后缀

在 Apache 配置文件 `D:\wamp\Apache24\conf\httpd.conf` 的 393 行左右，即：

```
<IfModule mime_module>
    TypesConfig conf/mime.types
    ....
</IfModule>
```

之间，添加 `AddType application/x-httpd-php .php`。即：

```
<IfModule mime_module>
    TypesConfig conf/mime.types
    ....
    AddType application/x-httpd-php .php
</IfModule>
```

## 添加主页 index.php

在 Apache 配置文件 `D:\wamp\Apache24\conf\httpd.conf` 的 278 行左右，即：

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

中，在 `index.html` 前添加 `index.php`。即：

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

## 测试效果

在 `D:\wamp\www` 下创建 `index.php`，代码为：

``` php
<?php
    phpinfo();
?>
```

重启 Apache（管理员身份运行 `CMD`，输入 `httpd -k restart`），浏览器访问 `localhost` 看到下图即为配置成功：

![](http://i4.buimg.com/1949/c17c955e90d20dbf.png)
