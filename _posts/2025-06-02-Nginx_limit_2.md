---
layout: post
title:  "Nginx限流及其原理"
description: Nginx限流及其原理
tag: [nginx,limit,java ,zf520,java思录]
date:   2025-06-02 08:50:35 +0800
categories: nginx
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---

# nginx限流及其原理（二）

昨日写了关于限制速率的问题，今天，我们来接着聊一聊关于限制同时连接的问题。

## 实际配置

```nginx
 limit_conn_zone $binary_remote_addr zone=perip:10m;
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            limit_conn perip 2 ;
            index  index.html index.htm;
        }

```

![image-20250601192054724](/assets/images/image-20250601192054724.png)

### ab压力测试

![image-20250601192359012](/assets/images/image-20250601192359012.png)

再看下access.log中的日志

![image-20250601192525977](/assets/images/image-20250601192525977.png)

## `limit_conn`

### `limit_conn_zone`

该指令是和`limit_req_zone`的作用一致，定义一个共享的内存区域，用户存储客户端连接数统计信息，比如IP地址，用户标识等。

语法

```nginx
Syntax:	limit_conn_zone key zone=name:size;
Default:	—
Context:	http
```

- `key` ：用于标识客户端的键，比如可以是 `$binary_remote_addr`表示客户端IP，`$server_name` 表示虚拟主机名等等。
- `zone=name:size` ： 共享内存区域的名称和大小，`size`表示内存区域的大小。10M表示10MB。

示例：

```nginx
limit_conn_zone $binary_remote_addr zone=per_ip:10m;  # 按客户端 IP 限制连接数
limit_conn_zone $server_name zone=per_server:10m;  # 按虚拟主机限制连接数
```

### `limit_conn`

该指令主要作用是设置每个键值对应的最大连接数。

语法：

```nginx
Syntax:	limit_conn zone number;
Default:	—
Context:	http, server, location
```

- `zone` ： 引用`limit_conn_zone`定义的共享内存区域名称。
- `number`： 限制同时在线的连接。

示例：

```nginx
location / {
  limit_conn per_ip 1;  # 每个 IP 最多允许 1 个连接
}
```

### `limit_conn_log_level`

同`limit_req`中一样，设置日志触发级别。默认是`error`

语法：

```nginx
Syntax:	limit_conn_log_level info | notice | warn | error;
Default:	limit_conn_log_level error;
Context:	http, server, location
```

#### `limit_conn_status`

返回的状态码，默认是503

```nginx
Syntax:	limit_conn_status code;
Default:	
limit_conn_status 503;
Context:	http, server, location
```



## 关于缓存区

在`limit_req` 和`limit_conn`中，都提到了创建一个内存共享区域，并定义了一个内存的大小，这个区域是有限制的，并不是说有多少数据都能存储的下，这里，内存的大小，只能存储一定数量的数据，所以，不知道大家注意到了没，在key中，我们每次都是填写的`$binary_remote_addr`,这个参数是`$remote_addr`的二进制形式，是一个4字节的IPv4或者16字节的IPv6，比文本形式要节约内存空间。

## 最后

关于http limit方面的配置就这些，在`limit_conn`中需要注意，如果一些请求非常快，那么限制到数据可能就不太明显，我在使用`ab -c 10 -n 10 http://b.zf520.net.cn/index.html` 时就遇到了所有的请求都成功，都能正常获得结果，那时因为一个http请求非常快的完成了，当请求完成后，下一个请求就可以非常快速的接替上。所以，不会限制到同时在线的问题。

