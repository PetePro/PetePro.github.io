---
layout: post
title:  "Shadowsocks 搭建 VPN"
crawlertitle: "Shadowsocks 搭建 VPN"
summary: "Shadowsocks 搭建 VPN"
date:   2017-05-23 23:09:47 +0800
categories: posts
tags: 'VPN'
author: WngLMng
---

## Shadowsocks 服务端安装

### Debian / Ubuntu 安装

```shell
$ apt-get install python-pip
$ pip install shadowsocks
```

### CentOS 安装

```shell
$ yum install python-setuptools && easy_install pip
$ pip install shadowsocks
```

## Shadowsocks 配置

创建 */etc/shadowsocks.json* 文件，添加如下内容

```vim
vim /etc/shadowsocks.json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"your_password",
    "timeout":600,
    "method":"aes-256-cfb",
    "auth": true
}
```

## Shadowsocks 命令

```shell
$ ssserver -c /etc/shadowsocks.json -d start // 启动
$ ssserver -c /etc/shadowsocks.json -d restart // 重启
$ ssserver -c /etc/shadowsocks.json -d stop // 关闭
```

最后记得开放 8388 端口

```shell
$ /sbin/iptables -I INPUT -p tcp --dport 8388 -j ACCEPT
```

## 客户端配置

到 https://shadowsocks.org/en/download/clients.html 下载客户端，然后输入你的配置即可

![](/assets/images/201705/shadowsocks.jpg)

**注意** 如果希望服务一直在后台中跑下去，可以在 screen 中执行以上操作，这样即使关闭 SSH 连接，服务也会一直在服务器中运行。

## Iterm 代理

* [使用shadowsocks及 ProxyChains-NG 实现终端(iterm)下代理](https://segmentfault.com/a/1190000004607285)
* [MacOS 让终端命令使用全局代理](http://www.jianshu.com/p/bee7c63c3d50)
