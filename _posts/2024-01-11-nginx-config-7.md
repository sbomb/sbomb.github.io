---
layout: post
title:  "nginx常用配置-VII"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-11 08:50:35 +0800
categories: Linux
author: ZF520

---

## 摘要

书接上回，本文介绍下静态网页的一些内容。

由于单独讲缓存部分的配置，文章较少，故增加了一节日志格式的解析和说明。

## http server部分

### 缓存静态资源

在http请求后，前端会有一些资源会缓存在用户的机器上，在一定的时间内，用户再次访问到相同的数据的时候，就可以从用户的前端资源中获取。此时，http请求返回的状态码是304，用户就可以在本地缓存中进行查找相应的数据。（从用户的内存中或者从硬盘中查找。）

那要如何配置一个资源什么过期。上配置。

``` nginx
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
     expires 10d;
}
```

上述配置，就是表示，以gif，jpg等结尾的图片文件，有效期是10天，访问一次后，10天内不需要再次从服务器上进行数据获取。

此配置比较简单，如果您不太理解，则可以直接拷贝当前的配置，当前配置主要是以对应后面配置的字符结尾的请求，比如http://b.zf520.net.cn/a.gif 或者http://b.zf520.net.cn/b.jpeg等等类似的请求。您可以增加其他类型的请求。比如js或css

```nginx
location ~ .*\.(js|css)$ {
     expires 10h;
}
```

### 日志格式配置

http请求到我们的服务器后，我们一般都想了解用户是从哪里访问到我们的机器，以及使用的是桌面应用，移动设备，用的是什么样的浏览器来对我们的系统进行的访问，方便我们对用户的地区，使用系统，使用的浏览器等等数据进行分析，当然，更多的分析的是访问的地址是什么。

那么，就需要我们来看以下日志的格式。

nginx默认的日志格式

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
```

这个格式默认的，不需要用户自己配置，默认存储的位置是nginx同级目录下logs/access.log文件下。

我从个人访问的nginx上拿一段日志来给大家看下。

 ```nginx
127.0.0.1 - - [29/Mar/2023:08:56:20 +0800] "GET / HTTP/1.1" 403 555 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"
 ```

可以和我们的配置对应下：

127.0.0.1 是$remote_addr字段，表示用户访问的ip地址。

两个 `-`直接的空格为$remote_user ，我们没有配置授权及用户信息，所以，该值为空。

`[]`之间的为本地时间， 具体值为  29/Mar/2023:08:56:20 +0800

`""` 为请求的URL地址以及请求的方法（POST/GET/DELETE/OPTIONS等）。 "GET  /  HTTP/1.1"

403 表示请求返回状态码，是$status。

555  是返回的body的发送字节数 $body_bytes_sent.

此后的部分是用户的请求头，关于用户的user_agent，一般是用户操作系统，浏览器核心，浏览器类型及版本$http_user_agent 。

最后的$http_x_forwarded_for之前的文章讲过，主要是代理IP的参数，本次请求无代理，所以，当前值为空。



## 后续

本文主要是介绍了nginx页面或数据的缓存。在系统运行期间，不仅要关注功能的运行，也要关注下系统的日志，以方便分析系统运行的情况或状态。敬请期待后续。



[个人博客](http://b.zf520.net)

