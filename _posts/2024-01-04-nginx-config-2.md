---
layout: post
title:  "nginx常用配置-II"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-04 08:50:35 +0800
categories: Linux
author: ZF520
---

## 摘要

书接上回，本文主要介绍代理后端服务器的一些配置



## http server部分

### 反向代理需要关注的点

上篇文章，我们介绍了如何配置一个代理，这样，我们就可以通过nginx来访问我们的后台服务器了。不过，这个时候，你去后台服务器上检查每个IP访问的URL信息，或者在程序中获取访问者的IP时，你就犯迷糊了，怎么全是nginx所在机器的IP，甚至，如果两者是在一起的时候，你可能发现所有的IP都是127.0.0.1的访问IP。这样的话，怎么才能区分用户登陆的IP啊。你别着急，下面我们就介绍下，怎么把前端的访问IP传递到后端的服务器上去。

还是要把原来的配置项拿过来，我们在其中增加

```nginx
location /hello{
	proxy_pass http://192.168.2.12:8080/;
    proxy_set_header X-Real-IP	$remote_addr;
    proxy_set_header X-Forwarded-For	$proxy_add_x_forwarded_for;
}
```

其中，remote_addr是nginx中内置的变量，表示上一个前一层前端用户访问的ip，此IP就是上一层访问的真实IP，注意，当前的IP有可能并不是真实的终端用户IP，如果是经过了多次的代理，那么，此remote_addr表示访问当前nginx的用户的IP。这个时候，我们就需要下面这个变量了。

proxy_add_x_forwarded_for 如果当前访问的X-Forwarded-For没有值，那么，就获取上面讲的$remote_addr值，并增加当前代理的IP，以逗号进行连接，如果有值，则直接增加代理的IP。

增加了相应的配置后，我们在服务端获取值的时候，也需要注意。我们以java为例。

直连时，我们可以通过reuqest.getRemoteAddr()来获取客户端的IP，时间上，该方法获取到的和nginx中的remote_addr中的值是相似的，都是上一层访问得到的IP，所以，我们看到的ip都是一个ip的原因了。我们需要改变一下策略，通过获取请求头X-Forwarded-For，来检查该请求头中是否有值，如果有值，则该数据中的第一个IP就是我们要找的用户的IP。

### 代理超时配置

配置上上述的配置后，一般简单的请求及网页都可以正常访问了。虽然主要的请求都是文字型的文本文件传输，上述配置就足以满足大部分用户的需求了，下面要说转折了，但，现在进入了5G时代，进入了图片与视频的年代，大家可能要传输图片或者文档文件到服务器上进行识别或者解析，尤其是当网络进入弱网环境下，文件传输的时间比较长的时候，就会收到timeout的错误。

这个时候，就需要请出我们的配置了。

```nginx
location /hello{
	proxy_pass http://192.168.2.12:8080/;
    proxy_set_header X-Real-IP	$remote_addr;
    proxy_set_header X-Forwarded-For	$proxy_add_x_forwarded_for;
    proxy_send_timeout	600s;
    proxy_read_timeout	600s;
}
```

解释下新增的两个参数配置。

proxy_send_timeout 表示nginx服务器发送数据到后端服务器上的时间，由于文件传输到nginx的时间较长，导致时间超出了默认的send_timeout的60s时间，这个时候，nginx就直接返回了错误信息，导致服务不能正常执行。

proxy_read_timeout表示后端服务器返回给nginx的时间，当我们的服务需要长时间处理一个逻辑的时候，如果超出了默认的60s的时间，同样nginx也会直接端口与后端的连接，返回前端一个超时的设置。



## 后续

本文主要介绍代理的一些常用配置，敬请期待后续。



[个人博客](http://b.zf520.net)

