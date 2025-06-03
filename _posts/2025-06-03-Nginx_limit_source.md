---
layout: post
title:  "nginx limit req 模块从源码分析"
description: nginx limit req 模块从源码分析
tag: [nginx,limit,java ,zf520,java思录]
date:   2025-06-03 08:50:35 +0800
categories: nginx
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---

# nginx limit req 模块从源码分析

今天分析下nginx http的limit req module模块，这个module主要是提供http访问速率限制的一个模块，以防止服务过载或者被滥用，该模块主要代码在`ngx_http_limit_req_module.c`文件中。

## 数据结构定义

介绍一个算法，应该要先了解这个算法的数据结构

- `ngx_http_limit_req_node_t`: 用于存储每个客户端请求的信息，比如key，时间戳，超额请求数等。
- `ngx_http_limit_req_shctx_t`: 用于共享内存中的红黑树和队列，用于管理所有请求节点。
- `ngx_http_limit_req_ctx_t`: 每个zone的上下文信息，包括共享内存指针、速率、key等。
- `ngx_http_limit_req_limit_t`: 定义一个限流规则，包括burst、delay和对应的共享内存区域。
- `ngx_http_limit_req_conf_t`: 配置结构体，包含多个限流规则、日志级别、状态码等。

![image-20250602211848406](/assets/images/image-20250602211848406.png)

请注意，今天主要分析的是限流的整体框架，所以，我们没有具体说每个参数的含义，大家可以根据前面的文章中的配置，来参考相应的数据。

## 限流处理主函数`ngx_http_limit_req_handler(r)`

nginx数据到达限流模块后，就会进入到`ngx_http_limit_req_handler`方法中来进行处理，负责根据配置的限流规则判断请求通过、延迟还是拒绝。

```c
    if (r->main->limit_req_status) {
        return NGX_DECLINED;
    }

```

如果请求的`limit_req_status`已经设置过了，就直接返回`NGX_DECLINED`，避免应用重复检查。

```c
    lrcf = ngx_http_get_module_loc_conf(r, ngx_http_limit_req_module);
    limits = lrcf->limits.elts;

    excess = 0;
```

从当前的location中获取限流配置`lrcf`. 获取到的是一个数组，此处，`ngx_http_limit_req_limit_t  *limit, *limits;`  limits 是一个数组，因为在c语言中，数组也可以退化为指针。

```c
    for (n = 0; n < lrcf->limits.nelts; n++) {

        limit = &limits[n];

        ctx = limit->shm_zone->data;

        if (ngx_http_complex_value(r, &ctx->key, &key) != NGX_OK) {
            ngx_http_limit_req_unlock(limits, n);
            return NGX_HTTP_INTERNAL_SERVER_ERROR;
        }

        if (key.len == 0) {
            continue;
        }

        if (key.len > 65535) {
            ngx_log_error(NGX_LOG_ERR, r->connection->log, 0,
                          "the value of the \"%V\" key "
                          "is more than 65535 bytes: \"%V\"",
                          &ctx->key.value, &key);
            continue;
        }
        # 计算key的hash数值
        hash = ngx_crc32_short(key.data, key.len);

        ngx_shmtx_lock(&ctx->shpool->mutex);
        # 限流主要的处理逻辑
        rc = ngx_http_limit_req_lookup(limit, hash, &key, &excess,
                                       (n == lrcf->limits.nelts - 1));

        ngx_shmtx_unlock(&ctx->shpool->mutex);

        ngx_log_debug4(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
                       "limit_req[%ui]: %i %ui.%03ui",
                       n, rc, excess / 1000, excess % 1000);

        if (rc != NGX_AGAIN) {
            break;
        }
    }
```

遍历所有的限流规则，对每个限流规则`limit`,计算key的hash值，并在共享内存中查找对应的请求节点，然后，调用`ngx_http_limit_req_lookup`判断是否超过限流阈值。

```c
 if (rc == NGX_BUSY || rc == NGX_ERROR) {
```

通过从`ngx_http_limit_req_lookup`得到的结果来判断后续是否继续。

- 如果返回结果`NGX_BUSY`或`NGX_ERROR`,表示请求被拒绝。
- 如果返回`NGX_AGAIN`或`NGX_OK` ，调用`ngx_http_limit_req_account()`方法来计算延迟时间。

```c
if (rc == NGX_AGAIN) {
        excess = 0;
    }

    delay = ngx_http_limit_req_account(limits, n, &excess, &limit);
```

如果返回值是`NGX_AGAIN`话，则超额请求数则会被置为0，说明系统还有余量来执行，有可能要被延迟，也有可能会立即执行。此时的行为，则有`ngx_http_limit_req_account()`方法来决定。如果延迟为0，请求通过，如果有延迟，则设置写时间延迟处理，并挂起请求。

