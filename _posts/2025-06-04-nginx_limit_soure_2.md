---
layout: post
title:  "nginx limit req 模块从源码分析(二)"
description: nginx limit req 模块从源码分析
tag: [nginx,limit,java ,zf520,java思录]
date:   2025-06-04 08:50:35 +0800
categories: nginx
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---

# Nginx limit req 模块从源码分析(二)

上文说了`limit req`的整体流程，今天笔者就限流的具体流程来说一下nginx的实现。

## `ngx_http_limit_req_lookup`

`ngx_http_limit_req_lookup`是nginx用于实现请求限速(rate limiting)的核心函数之一，负责查找、更新或者插入一个请求键(key)，并根据配置的速率和时间间隔判断是否允许该请求通过。

首先，nginx先将当前时间、上下文等数据进行初始化。

```c
    # 获取当前时间(以毫秒为单位)
	now = ngx_current_msec;
	# 指向共享内存区域中的上下文结构体，保存了限流相关的数据。
    ctx = limit->shm_zone->data;
	# 红黑树的根节点
    node = ctx->sh->rbtree.root;
	# 代表红黑树中的 "空节点",用于标记叶子节点的结束。
    sentinel = ctx->sh->rbtree.sentinel;
```

初始化完成后，就开始对红黑树进行循环遍历。上文的时候说过，key是一个经过hash计算的hash值，所以，再结合红黑树的特点，大的值在右边，小的值在左边。所以，nginx对红黑树进行遍历的代码如下

```c
    while (node != sentinel) {

        if (hash < node->key) {
            node = node->left;
            continue;
        }

        if (hash > node->key) {
            node = node->right;
            continue;
        }
```

当查找到相应的值后，nginx就会更新队列位置，将当前节点从队列中移除，并插入到对头，这样是为了维护最近使用的顺序（LRU算法的一部分）。

```c
        /* hash == node->key */

        lr = (ngx_http_limit_req_node_t *) &node->color;

        rc = ngx_memn2cmp(key->data, lr->data, key->len, (size_t) lr->len);

        if (rc == 0) {
            ngx_queue_remove(&lr->queue);
            ngx_queue_insert_head(&ctx->sh->queue, &lr->queue);
```

下面就是限流和限速的具体代码实现，笔者先把代码放在这里，下面将详细说一下代码的逻辑。

```c 
            ms = (ngx_msec_int_t) (now - lr->last);

            if (ms < -60000) {
                ms = 1;

            } else if (ms < 0) {
                ms = 0;
            }

            excess = lr->excess - ctx->rate * ms / 1000 + 1000;

            if (excess < 0) {
                excess = 0;
            }

            *ep = excess;

            if ((ngx_uint_t) excess > limit->burst) {
                return NGX_BUSY;
            }

            if (account) {
                lr->excess = excess;

                if (ms) {
                    lr->last = now;
                }

                return NGX_OK;
            }

            lr->count++;

            ctx->node = lr;
```

在这段代码中，首先计算出上一次访问到现在的时间差now为开始初始化中的当前时间，`lr->last`,最后一次访问的时间，这个如果当前访问通过的话，则会把该`now`值赋给`lr->last`。限流是一段时间内的流量，所以，是有一个时间段的。至于为什么会有负值，这种情况主要是由于系统时间被回退了，比如手动修改或者NTP时间同步导致时间倒退，所以，会有当前时间比最后一次访问时间小的问题产生。在这样的情况下，nginx处理分成两种情况来处理。一是当时间回退超过1分钟，则视为刚刚发生，当时间处于-60000 <= ms < 0，则表示时间倒退了，但差异不大，按“无间隔”处理。这样，就避免了由于系统时间同步或bug导致负数时间差破坏了限速逻辑，也能确保即使时间倒退了，也不会让限速算法误判为“很久没请求”，从而允许大量突发流量通过。

通过刚才的时间段，这个时候来计算超额请求数（excess）。先根据上次的超额数、时间差和配置的速率重新计算新的`excess`.如果结果小于0，则就设置为0，否则就把结果写入到输出参数`*ep`种。

计算出来的超额数(excess)与突发限制的数值进行比较，看看是否超过了限制的`burst`值(最大突发请求数)，如果超过了， 则直接返回`NGX_BUSY`来拒绝请求。

接下来，判断当前的限速配置是否是最后一个，如果是的话，则记录相应的数据，如果`ms`值（表示时间差大于0），就记录下当前时间为最后的请求时间。

