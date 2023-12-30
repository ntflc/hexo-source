---
title: 使用 NGINX 为网站做反向代理
date: 2016-10-25 21:56:13
categories:
- Tutorial
tags:
- Tutorial
- NGINX
---

# 背景

我在我的 VPS 上搭了一个 Jenkins 服务，然后将二级域名 [jenkins.ntflc.com](http://jenkins.ntflc.com) 解析到 VPS 的 ip 上。但由于 Jenkins 服务的默认端口是 8080，而访问 HTTP 的默认端口是 80，因此直接访问 jenkins.ntflc.com 并不会进入 Jenkins 页面，只能通过 jenkins.ntflc.com:8080 访问。

为了省去每次输入端口号，我一开始使用的是 [rinetd](https://boutell.com/rinetd/)，一个端口转发工具。但是由于种种原因，后来放弃了这个工具，所以准备使用 NGINX 来做反向代理。

<!-- more -->

# 安装 NGINX

我的 VPS 安装的是 Ubuntu 14.04，所以这里以 Ubuntu 14.04 为例。

具体方法，NGINX 官网有具体的说明，可以到这里来看：[https://nginx.org/en/linux_packages.html](https://nginx.org/en/linux_packages.html)

首先，下载 `nginx_signing.key`：

``` bash
wget https://nginx.org/keys/nginx_signing.key
```

然后：

``` bash
sudo apt-key add nginx_signing.key
```

这样就可以使用 `apt-get` 命令安装了：

``` bash
apt-get update
apt-get install nginx
```

# 配置反向代理

在 `/etc/nginx/conf.d/` 下新建 `.conf` 文件，文件名随意，然后将以下配置复制到 `.conf` 文件中：

```
server {
    listen 80;
    server_name jenkins.ntflc.com;
    access_log off;

    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8080;
    }
}
```

其中 `server_name jenkins.ntflc.com;` 是我想要反向代理的域名，`listen 80;` 是监听的端口（HTTP 默认为 80 端口），`proxy_pass http://127.0.0.1:8080;` 是我希望访问的端口地址。

配置完毕后，运行：

``` bash
nginx
```

即可。

好了，现在访问 jenkins.ntflc.com 就会自动请求 8080 端口了。

# 后续操作

如果需要添加开机启动项，编辑 `/etc/rc.local` 文件，添加：

``` bash
sudo nginx
```

即可。
