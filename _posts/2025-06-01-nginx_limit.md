---
layout: post
title:  "Nginx限流及其原理"
description: Nginx限流及其原理
tag: [nginx,limit,java ,zf520,java思录]
date:   2025-06-01 08:50:35 +0800
categories: nginx
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---



# nginx限流及其原理（一）

## 前言

在开发的过程中，我们经常一些要限制用户访问速率或者限制IP同时在线的个数的问题。这样的需求，如果要自己手动开发，虽然还是比较简单，但还要浪费不少时间，且有一些问题在内。

不过，Nginx给我们提供了limit方面的指令，我们只要根据其配置，就能很方便的启用了限流的功能。

本文主要介绍指令的配置

## limit_req指令

`nginx_http_limit_req_module`模块在0.7.21中就提供了，主要是用来限制请求处理的速率，这个速率，还是要有一个参考值，比如是单个的IP，或者是单个用户，一般的情况，是用IP来处理的。

![image-20250531211540802](/assets/images/image-20250531211540802.png)

```
http{
	limit_req_zone $binary_remote_addr zone=perip:10m rate=2r/s;
	server {
		 location / {
            root   html;
            limit_req zone=perip burst=5 nodelay;
            index  index.html index.htm;
        }
	}
}
```

配置好之后，使用命令`nginx -s reload`重新加载nginx，此后，我们使用ab来对服务器做一个测试。

### ab压力测试

ab的命令就不多讲了，使用如下命令

`ab -c 10 -n 10 http://b.zf520.net.cn/` 

可以得到如下结果：

![image-20250531211959669](/assets/images/image-20250531211959669.png)

从上图中可以看到，并发10个用户，一共发送10个请求，其中，有4个请求失败了，返回了non-2XX的返回体。

再看下nginx的access.log日志：

![image-20250531212305128](/assets/images/image-20250531212305128.png)

确实是有4个返回结果为503，503的意思是服务不可用。

### 配置参数解析

#### `limit_req_zone`

先说下http块中的`limit_req_zone`，这个参数主要是定义共享内存和限制规则。其主要用法如下：

```
Syntax:	limit_req_zone key zone=name:size rate=rate [sync];
Default:	—
Context:	http
```

其中，

- `key` ： 限制的键，通常使用`$binary_remote_add`(客户端的IP)
- `zone=name:size` ： 定义共享内存区的名称和大小(如10m表示10MB)
- `rate=rate` ： 请求限制的速率如`10r/s`(每秒10次)或`30r/m`(每分钟30次)。r代表request

示例

```
limit_req_zone $binary_remote_addr zone=req_limit:10m rate=10r/s;
```

#### `limit_req`

这个参数主要是把限制的速率作用到哪个位置。其主要用法如下：

```
Syntax:	limit_req zone=name [burst=number] [nodelay | delay=number];
Default:	—
Context:	http, server, location
```

其中，

- `zone=name` ： 指定使用的共享内存区。
- `burst=number` : 突发请求缓存队列大小（可选）
- `nodelay` ： 是否立即处理突发请求（可选）
- `delay=number` ： 延迟处理的请求数（高级选项）

示例一：

```
limit_req zone=req_limit burst=20;
```

先解释下`burst`参数，`burst`表示突发请求的缓存队列大小，定义允许超过速率限制的突发请求数，这些请求会被放入到队列中延迟处理。

`nodelay` 参数，与`burst`配合使用，表示允许立即处理突发请求，而不是延迟处理，但如果超过了`burst + rate`的请求则会被拒绝。

`delay=number`：这个参数是nginx 1.15.7版本后增加的参数，主要是用来指定多少突发请求可以理解被处理，剩余的请求会被延迟处理。提供了比`nodelay`更精细化的控制。

总结下，就是：

当请求到达时，nginx检查对应客户端IP的请求计数，如果未超过速率限制（`rate`),请求就会被处理，如果超过了速率限制数，但还有`burst`空间，请求被排队处理或者立即处理（取决于是否参数`nodelay`)。如果超过了`rate + burst`，则返回错误状态码。这样，就可以平滑请求流量，防止服务器过载。

那么，上述示例就是，定义了一个20个的突发请求缓冲区，这20个请求将被延迟处理。

示例二：

```
limit_req zone=req_limit burst=20 nodelay;
```

这个示例中20个缓冲区的请求，会被立即处理，相当于为`limit_req_zone`中增加了20个冗余量。

示例三：

```
limit_req zone=req_limit burst=20 delay=10;
```

这个示例，缓冲区大小是20个，但只有10个可以被立即处理，剩余的10个要延迟处理。

最后，补充一点这个指令的作用范围

- http: 如果该指令配置再http块下，则对整个下面的server，location都有效果，即，差不多是全局起作用。
- server: 只对当前server下所有的location起作用。
- location: 只对当前请求有效，其他的请求不起作用。

#### `limit_req_status`

`limit_req_status`主要是指定当超过限制速率后的请求返回的状态码，默认是503。

```
Syntax:	limit_req_status code;
Default:	
limit_req_status 503;
Context:	http, server, location
```

示例：

```
limit_req_status 503;
```

#### `limit_req_log_level`

`limit_req_log_level`是定义限制触发时的日志级别，默认是`error`。

语法：

```
Syntax:	limit_req_log_level info | notice | warn | error;
Default:	
limit_req_log_level error;
Context:	http, server, location
```

示例：

```
limit_req_log_level warn;
```

日志级别的文章，可以参考笔者知乎上的一篇文章。







