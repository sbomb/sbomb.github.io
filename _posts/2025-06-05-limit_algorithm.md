---
layout: post
title:  "限流机制的常用算法"
description: 限流机制的常用算法
tag: [nginx,limit,令牌桶算法,漏斗算法,java ,zf520,java思录]
date:   2025-06-05 08:50:35 +0800
categories: nginx
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---

# 限流机制的常用算法

前两天，我们从Nginx的配置以及源码角度来讨论了下关于nginx中限流机制，今日，笔者将从更宏观的限流算法方向来说一下限流有哪些算法。

## 固定窗口计数器算法

固定窗口计数器算法：在固定时间窗口（如1秒）内统计请求数，超过阈值就拒绝请求。因为时间窗口是固定的，所以，如果某一时间有大量的突发流量就只能拒绝请求，同样，如果当前窗口的流量较低，也无法平摊下之前突发的流量。这种算法比较适合请求分布比较均匀的场景，每个时间段都比较平滑，没有太多的突发流量。

![限流之固定窗口/滑动窗口计数法理解-CSDN博客](/assets/images/ef4133d159daeda8aad124d272495c81.png)

算法实现：

```java
// 伪代码示例
int counter = 0;
long windowStart = System.currentTimeMillis();
if (System.currentTimeMillis() - windowStart < 1000) {
    if (counter < limit) {
        counter++;
        // 处理请求
    } else {
        // 限流
    }
} else {
    windowStart = System.currentTimeMillis();
    counter = 0;
}
```

## 滑动窗口计数算法

滑动窗口计数算法，是对固定时间窗口算法的改进，通过多个子窗口来平滑统计请求，避免瞬时的高峰。在网络的发送控制中，就是使用滑动窗口算法。该算法适用于需要精确控制流量的场景，如API网关、网络发送控制。

![简单介绍4种限流算法！（固定窗口计数器算法、滑动窗口计数器算法、漏桶算法、令牌桶算法） - 21ic电子网](/assets/images/2e98ce6f9d1da7ea88594c12d11ed03b.png)

算法实现：

```java
import java.util.*;

class SlidingWindow {
    private final int maxRequests;   // 窗口内最大请求数
    private final long windowSizeMs; // 窗口大小(毫秒)
    private final Map<Long, Integer> slots = new HashMap<>(); // 时间槽->计数
    private int totalCount = 0;      // 当前窗口总请求数

    public SlidingWindow(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
    }

    public synchronized boolean allowRequest() {
        long now = System.currentTimeMillis();
        long currentSlot = now / 1000; // 每秒为一个槽位
        
        // 清理过期槽位(1秒前的槽位)
        long oldestValidSlot = currentSlot - (windowSizeMs / 1000 - 1);
        Iterator<Map.Entry<Long, Integer>> it = slots.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<Long, Integer> entry = it.next();
            if (entry.getKey() < oldestValidSlot) {
                totalCount -= entry.getValue();
                it.remove();
            }
        }

        // 检查是否允许请求
        if (totalCount < maxRequests) {
            slots.put(currentSlot, slots.getOrDefault(currentSlot, 0) + 1);
            totalCount++;
            return true;
        }
        return false;
    }
}
```

## 漏桶算法（Leaky Bucket）

漏桶算法，以恒定的速率处理请求，超出容量的请求将被丢弃或者进行排队。作为中国的小学生，大家应该都会被一个数学题给折磨过，有一个浴缸，下面以恒定的速率出水，上面有水龙头以一个速率进水，问什么时候水会满？而我们的请求则不是恒定速率进来的，而是有时快，有时慢的进入，所以，利用漏桶算法的，可以很好的平衡服务器利用率。适合强制固定速率处理，比如网络流量的控制等等。

算法实现：

```java
// 伪代码
Queue<Request> bucket = new LinkedList<>();
void processRequest(Request req) {
    if (bucket.size() < capacity) {
        bucket.add(req);
    } else {
        // 限流
    }
}
// 定时以固定速率处理bucket中的请求
```

## 令牌桶算法（Token Bucket）

令牌桶算法，则是以固定的速率生成令牌，请求获取到令牌才能执行，允许有突发的流量。该算法比较适合有短期突发请求的场景。

![解剖常见的限流算法_固定窗口滑动窗口-CSDN博客](/assets/images/52b675e0e81c4e9a9ca17716a3963e97.png)

算法实现：

```java
RateLimiter limiter = RateLimiter.create(100); // 每秒100个令牌
if (limiter.tryAcquire()) {
    // 处理请求
} else {
    // 限流
}
```

## Nginx中的限流算法

上面的限流算法使用的是Java实现，而Nginx是使用c的实现，不过，从前面的几篇文章中可以看出，nginx的限流主要是基于`漏桶算法(Leaky Bucket)`实现，虽然Nginx的实现的时候，基于时间窗口进行请求统计，但还是通过漏桶算法的平滑处理避免了滑动窗口的临界突发问题。

