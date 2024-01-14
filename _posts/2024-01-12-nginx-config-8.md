---
layout: post
title:  "nginx常用配置-VIII"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-12 08:50:35 +0800
categories: Linux
author: ZF520

---

## 摘要

书接上回，本文介绍下常用的配置，行文比较散，笔者尽量按问题的需要进行文章梳理。



## http server部分



### 跨域处理

当我们的页面或者数据被嵌入其他的页面时，会容易出现跨域问题，这个时候，我们就需要在我们的配置中增加相应的跨域设置。

```
add_header Access-Control-Allow-Credentials true;
add_header Access-Control-Allow-Origin * always;
add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

if ($request_method = 'OPTIONS') {
    return 204;
}
```

add_header 是对数据增加返回头的，此配置就是增加一些返回头，根据浏览器的规则，

增加allow-origin * 或者允许的IP地址，都是可以的，*为通配符，标识所有的refer的地址都可以用。refer的地址一般为请求地址。即页面在192.168.2.10 上，访问的地址是192.168.2.10，访问的数据或者部分页面是192.168.2.11上的，那么，refer为192.168.2.10，那么，origin后面的地址就要填写192.168.2.10。一般情况下，默认为 * 即可。

allow-Methods，允许的方法，可选的方法有 GET、POST、PUT、DELETE、OPTIONS等。

下面的if条件，是当请求的Method为OPTIONS时，直接返回204状态码。为什么要如此设置哪。这个时候，就需要简单讲下跨域时，HTTP的请求流程。

### 页面跨域访问

当跨域访问时，如果请求为非简单请求（什么是简单请求，简单请求就是如果请求方法为POST，且请求类型为application/text时，即为简单请求。有朋友就问了，那application/json是简单请求吗？答案是否。所以，一般请求application/json时，也为非简单请求），那么浏览器就会有一次试探性的请求访问，看看浏览器是否支持。如果后端只设置了POST方法，那么OPTIONS请求就会报错，提示不支持的OPTIONS方法请求，一样会导致请求失败，如果支持了OPTIONS，那么就会跑一遍应用逻辑，导致系统业务复杂度增高。

本文简单介绍下OPTIONS请求的逻辑。还请期待后续详细介绍。

### try_files 指令

从几年前开始，现在的web应用程序都是单页面应用，什么意思那，就是只有一个页面，所有的内容都在一个页面上进行展示，符合数据与界面双向绑定的概念，其所有的样式，js文件，都会被主页面进行包含，主页面也只有一个空的div来接收后续所有的请求。

那用什么来标识请求的不同的页面内容那，在vue、react等框架中，大家都使用一个叫路由的东西来定位到不同的页面内容，在浏览器上的表现如URL类似http://192.168.2.10:8080/#/home 这个地址的请求地址就是根目录，路由为home的页面内容，VUE等框架就会在js中加载这些数据及界面内容。（本人主要研发后端，对前端只有大概的了解，如果您有比较好的解释，还请[联系我](mailto:zengfei@zf520.net.cn)，或者关注我，给我私信。)

由于地址栏上很难体现我们要访问的资源名称，尤其是类似的index.html资源，nginx如何才能找到当前地址下路由信息那？这个时候，我们就可以用try_files指令了。

配置如下

```nginx
try_files $uri $uri/ /index.html
```

 这样，就可以解决vue、react中路由找不到的问题了。

## 后续

本文主要是介绍了nginx配置跨域问题及try_files指令的问题。敬请期待后续。



[个人博客](http://b.zf520.net)

