---
layout: post
title:  "Nginx限流模块limit_req详解"
description: Nginx限流模块limit_req详解
tag: [ Nginx,limit_req,zf520,java思录]
date:   2025-06-07 08:50:35 +0800
categories: java
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---



# Nginx限流模块limit_req详解

在Web服务器管理的复杂生态中，限流（Rate Limiting）犹如一道坚固的防线，其重要性不言而喻。它能够精准地调控客户端的请求频率，有效抵御突发流量的冲击，防范恶意攻击对服务器造成的损害。Nginx作为一款卓越的高性能反向代理和Web服务器，凭借其强大的`ngx_http_limit_req_module`模块，为限流功能提供了坚实的支持，而`limit_req`指令则是该模块中最为常用的核心指令。本文将深入剖析`limit_req`模块的使用方法、配置技巧以及实际应用场景，助力读者全面掌握这一关键技术。

## 一、什么是limit_req？

`limit_req`是Nginx提供的一个限流机制，主要用于限制单位时间内客户端的请求频率。它基于**漏桶算法（Leaky Bucket Algorithm）**，通过定义固定的处理速率，将请求像水一样注入桶中，桶以恒定速率将请求“泄漏”出去。如果请求过多导致桶满，则后续请求会被丢弃或排队等待。

该模块包含两个核心指令：

- `limit_req_zone`：此指令用于定义一个共享内存区域，该区域专门用于存储会话状态信息。通过合理配置，可以确保不同请求之间的限流状态能够被准确记录和共享。该指令主要是在http区块进行定义。（前文[nginx限流及其原理](https://zhuanlan.zhihu.com/p/1912272233407186898)有更详细的说明）
- `limit_req`：该指令在location或server块中启用限流策略，实现对特定路径或整个服务器的请求频率限制。

## 二、基本配置语法

### 1. 定义限流区域

```nginx
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    
    server {
        listen 80;
 
        location / {
            limit_req zone=one burst=5;
            proxy_pass http://backend;
        }
    }
}
```

- `$binary_remote_addr`：作为限流的Key，它表示客户端的IP地址。使用二进制形式的IP地址可以节省内存空间，提高处理效率。
- `zone=one:10m`：定义了一个名为`one`的共享内存区域，其大小为10MB。这个区域将用于存储与该限流策略相关的状态信息。
- `rate=1r/s`：此参数表示每秒最多允许1个请求通过。通过调整该参数，可以灵活控制请求的频率限制。
- `burst=5`：该参数定义了突发请求的最大数量。在短时间内，允许超过限流速率的请求数量最多为5个。超过这个数量后，后续请求将被拒绝或延迟处理。

### 2. 更多参数说明

- `nodelay`：当启用此参数时，突发请求将不会被延迟处理，而是立即得到处理，但前提是突发请求的数量不超过`burst`值。这一参数在某些对实时性要求较高的场景中非常有用。
- `limit_req_status`：该参数用于自定义返回的HTTP状态码，默认情况下为503（Service Temporarily Unavailable）。通过自定义状态码，可以更灵活地向客户端传达请求被限制的原因。

示例：

```nginx
limit_req zone=one burst=5 nodelay;
limit_req_status 429;
```

## 三、限流策略的应用场景

### 1. 防止DDoS攻击

DDoS攻击是一种常见的网络攻击手段，它通过大量请求淹没服务器，导致服务器无法正常响应合法请求。通过限制单个IP的请求频率，可以有效抵御一些简单的DDoS攻击。例如，可以设置每个IP每秒只能发送10个请求，一旦超过这个限制，请求将被拒绝，从而保护服务器免受攻击。

PS:需要注意的是，Nginx是从应用层进行拦截和处理的，如果采用的是半连接的DDos攻击方式，则需要从更底层，比如网络层进行处理。这个可以参考之前的关于[网络方面-石头侃网络](https://zhuanlan.zhihu.com/column/c_1854623471000240128)的文章。

### 2. API接口限流

对于提供RESTful API的服务来说，API接口的调用频率限制至关重要。如果不加以限制，API可能会被滥用，导致服务器资源耗尽。通过为不同的API路径设置不同的限流策略，可以确保每个API接口的调用频率在合理范围内。例如，对于一些核心API接口，可以设置更严格的限流策略，而对于一些辅助API接口，可以适当放宽限制。

```nginx
location /api/v1/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://api_server;
}
```

### 3. 控制爬虫访问频率

搜索引擎爬虫在抓取网站内容时，有时会对网站造成较大的访问压力。如果爬虫的访问频率过高，可能会导致服务器性能下降，影响正常用户的访问体验。通过限流技术，可以精确控制爬虫的访问频率。例如，可以根据爬虫的User-Agent信息，识别出常见的搜索引擎爬虫，并为其设置特定的限流策略。(知名公司的爬虫，都会在UA(User_Agent)中标明，也会遵守协议，但一些恶意爬虫，会伪装这个值，还需要根据日志进行分析，判断出一些恶意爬虫来进行处理和规则调整)

```nginx
map $http_user_agent $limit_bots {
    default         "";
    ~*(googlebot|Baiduspider)   $binary_remote_addr;
}
 
limit_req_zone $limit_bots zone=bots:10m rate=1r/s;
 
location / {
    limit_req zone=bots burst=5;
    ...
}
```

## 四、高级配置与注意事项

### 1. 多级限流

在实际应用中，单一的限流策略可能无法满足复杂的业务需求。通过结合多个`limit_req`指令，可以实现多级限流。例如，可以先按照IP地址进行限流，然后再按照用户ID或其他变量进行限流。这种多级限流策略可以更加精细地控制请求频率，提高系统的安全性和稳定性。

```nginx
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;
limit_req_zone $uid zone=user_limit:10m rate=5r/s;
 
location /api/ {
    limit_req zone=ip_limit burst=20;
    limit_req zone=user_limit burst=10;
    ...
}
```

### 2. 动态限流

虽然Nginx本身并不直接支持动态修改限流参数，但可以通过Lua脚本结合OpenResty扩展来实现更灵活的限流逻辑。OpenResty是一个基于Nginx和Lua的高性能Web平台，它提供了丰富的Lua API，可以方便地实现动态限流功能。例如，可以根据服务器的实时负载情况，动态调整限流速率，以适应不同的业务场景。

### 3. 性能与内存占用

- 每个`limit_req_zone`都需要占用一定的内存空间。一般来说，1MB内存大约可以存储64,000个IP地址（使用二进制的方式，如果是字符串，则可能会更少，当然，也可以增大共享区域，但内存大了，也容易造成系统管理困难）的状态信息。因此，在定义限流区域时，需要根据实际需求合理设置内存大小，避免内存浪费或不足。
- `rate`参数的设置需要谨慎。如果设置过低，可能会误伤正常用户，导致用户体验下降；如果设置过高，则可能失去限流的意义，无法有效保护服务器。因此，需要根据业务特点和服务器性能，进行合理的参数调整。