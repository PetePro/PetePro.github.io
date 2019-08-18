---
layout: post
title:  "Fixing 413 Request Entity Too Large Errors"
crawlertitle: "Fixing 413 Request Entity Too Large Errors"
summary: "Fixing 413 Request Entity Too Large Errors"
date:   2019-04-22 11:30:00 +0800
categories: posts
tags: 'HTTP'
author: WngLMng
---

![](/assets/images/201904/413-request-entity-too-large-lg.png)

## 413 Request Entity Too Large 是什么意思
当一个 request 请求内容太大，从客服端到服务端请求处理的时候，就可能会发生 413 request entity too large 这种错误。如果你的服务端设置了 HTTP 请求大小限制，客户端就会响应 413 request entity too large。举个例子，客户端尝试上传一个大文件到服务端。

这种情况依赖于你使用的服务器类型来确定你需要修改什么配置。无论你想显示用户上传文件的大小，或者还是增大上传文件的限制，看下面。

## 解决 413 Request Entity Too Large 的问题
### Nginx

对于 Nginx 用户来说，可以修改 HTTP 请求大小对应的配置 *client_max_body_size*。这个可能已经配置在 *nginx.conf* 中了，如果没有，你可以添加这个指令在 *http*，*server*，或者 *location* 代码块中。

```
server {
    client_max_body_size 100M;
    ...
}
```

*client_max_body_size* 的默认值是 1MB，你也可以修改成 0。

一旦修改成了你想要的值，保存后记得重启 Nginx。

### Apache

对于 Apache 服务，有一个指令是 *LimitRequestBody*，这条指令和 Nginx 中的 *client_max_body_size* 指令提供同样的功能。这条指令配置在 *http.conf* 文件中，默认值是 0，你可以修改成你希望的大小（单位字节）。
举个例子，你可以想下面这样，修改成 100MB 大小。

```
LimitRequestBody 104857600
```

同样，修改后记得重启 Apache 服务。

### PHP

对于 PHP 用户，可以尝试修改 *php.ini* 中的配置，*upload_max_filesize* 和 *post_max_size*。

* *uploa_max_filesize* 代表着允许上传文件的最大值，默认 2MB
* *post_max_size* 代表着 POST 数据 ，PHP 服务会接受的最大值，默认 8MB

同样，修改后记得重启 PHP 服务。
