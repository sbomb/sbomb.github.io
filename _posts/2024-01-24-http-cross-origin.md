---
layout: post
title:  "http跨域原理及解决方案"
description: 跨域原理
tag: [nginx,config,跨域,CORS,zf520,java思录]
date:   2024-01-15 08:50:35 +0800
categories: Linux
author: ZF520
---



## 同源

浏览器的同源策略是一个重要的安全策略，它主要用于限制一个源的文档或者加载它的脚本如何能与另一个源的资源进行交互。

它能帮助阻隔恶意文档，减少可能被攻击的媒介。例如：它可以防止互联网上的恶意网站在浏览器中运行js脚本，从第三方网络服务或公司内网读取数据，并将这些数据转发给攻击者。

### 源的定义

如果两个URL的协议（scheme）、端口（port，如果没有指明端口号，则默认http为80，https为443端口）和主机（host）都相同的话，则两个URL是同源的。我们大致举例说明下：

是否与http://zf520.net.cn/zf520/index.html

| URL                                        | 是否同源 | 原因                               |
| ------------------------------------------ | -------- | ---------------------------------- |
| http://zf520.net.cn/test/test.html         | 同源     | 只有路径不同，符合上述三元组       |
| http://zf520.net.cn/zf520/test/test.html   | 同源     | 只有路径不同                       |
| https://zf520.net.cn/zf520/index.html      | 否       | 协议不同，http与https协议不同      |
| http://zf520.net.cn：8080/zf520/index.html | 否       | 端口不同，80（http默认端口）与8080 |
| http://b.zf520.net.cn/zf520/index.html     | 否       | 主机不同，（即子域名与主域名也为不 |

## 跨域

上面介绍了同源及同源策略，理解了同源，那么，跨域就是一个非常容易理解了，上述不同源的数据中，如何从一个源访问到另一个源，这就是我们经常说的跨域。

同源策略控制不同源之间的交互，例如使用XMLHttpRequest或者<img>标签时则受到同源策略的约束。

- 跨域写操作（cross-origin writes) 一般是允许的，例如连接、重定向及表单提交。（非简单HTTP请求需要先发送OPTIONS请求）
- 跨域资源嵌入（cross-origin embedding) 一般是允许的。
- 跨域读操作(cross-origin reads）一般是不允许的，但可以通过其他方式来解决，（后续的JSONP就是一种解决方案



## 跨域方案

### CORS跨域

可以使用CORS来允许跨域访问。CORS是HTTP的一部分，它允许服务端来指定哪些主机跨域从这个服务端加载资源。 

比如在nginx的配置中增加如下配置：

```
add_header Access-Control-Allow-Credentials true;
add_header Access-Control-Allow-Origin * always;
add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

if ($request_method = 'OPTIONS') {
    return 204;
}
```

[sbomb：nginx常用配置-VIII](https://zhuanlan.zhihu.com/p/677387556)

即在请求头上会有  Allow-Origin: * 或者配置的ip : port 等。

### JSONP跨域

jsonp进行跨域，实际是通过script标签不受同源策略的影响，通过使用回调的方式来处理请求的返回。

具体可以参考：

[JSONP 什么是JSONP？JSONP的实现原理？实现JSONP](https://juejin.cn/post/6982517005974765599)



参考资料：

1. [浏览器的同源策略 - Web 安全 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
2. [JSONP 什么是JSONP？JSONP的实现原理？实现JSONP](https://juejin.cn/post/6982517005974765599)
3. [sbomb：nginx常用配置-VIII](https://zhuanlan.zhihu.com/p/677387556)


