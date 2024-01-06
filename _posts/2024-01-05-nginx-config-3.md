---
layout: post
title:  "nginx常用配置-III"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-05 08:50:35 +0800
categories: Linux
author: ZF520
---

## 摘要

书接上回，本文主要介绍代理后端服务器的一些配置



## http server部分

### 反向代理之负载均衡

上文讲过了代理后端服务器，我们可以通过nginx来访问后端服务，也能把一些前端的信息通过参数的形式给传递到服务器上。似乎代理服务器配置就成功了，万事大吉了。我们的行文要结束了吗？这个时候，有同学可能要疑惑了，如果我的后台服务器上有多台机器，每台机器都可以接收前端的请求，而我们之前配置的时候，好像只配置了一个ip和端口，那一台机器要如何配置。

问题是个好问题，看来这个同学遇到了单机性能问题了，想通过太多机器来分担单机的压力，也有可能是怕一台机器出现问题，另一台机器也可以做一个备用。

下面，我们先请出来我们原来的配置项

```nginx
location /hello{
	proxy_pass http://192.168.2.12:8080/;
    proxy_set_header X-Real-IP	$remote_addr;
    proxy_set_header X-Forwarded-For	$proxy_add_x_forwarded_for;
}
```

现在，我们来看下这个配置，我们的IP是写在了proxy_pass后面，那么，我们如果有多台机器的时候，要如何修改哪？想一想，如果有一个标签可以表示多个机器的信息，那我们就可以在proxy_pass后直接写这个标签就可以了。

是的，nginx中还真的有这么一个标签，这个标签是叫upstream的指令，这个指令是和server指令同级，配置如下：

```nginx
upstream backendServer{
	server 192.168.2.12:8080;
    server 192.168.2.13:8081;
}
```

在server同级上配置了upstream后，我们就可以在location中进行如下的配置：

```nginx
proxy_pass http://backendServer/;
```

如此，就可以轮流对两台机器进行访问。

### 负载均衡策略

讲到了多台机器参与服务器访问，那么，我们就不得不说一下负载均衡的策略了。类似我们刚才配置的两台机器，这个策略就是默认的轮询策略。

#### 轮询

多台机器进行依次访问，从第一台开始，下一次访问再请求下一台，一直请求到最后一台机器，再从从第一台机器进行请求。

配置比较简单，和上文中的一致

```nginx
upstream backendServer{
	server 192.168.2.12:8080;
    server 192.168.2.13:8081;
}
```



#### 加权轮询

通过给机器增加不同的权重，权重高的机器，可以增加被请求的几率。

配置如下：

```nginx
upstream backendServer{
	server 192.168.2.12:8080	weight=2;
    server 192.168.2.13:8081	weight=3;
}
```

#### IP hash

通过对请求IP使用hash算法，来决定访问那台机器。适用于上次请求到某台机器上，而下一次的请求也希在当前机器上进行处理。

配置如下：

```nginx
upstream backendServer{
	server 192.168.2.12:8080;
    server 192.168.2.13:8081;
    ip_hash;
}
```

#### URL hash

对请求不同的URL进行hash计算，决定访问请求的机器。

配置如下：

```nginx
upstream backendServer{
	server 192.168.2.12:8080;
    server 192.168.2.13:8081;
    hash $request_uri;
}
```

#### fair 

按照后端服务器的响应时间来决定下一次访问哪台机器。

配置如下：

```nginx
upstream backendServer{
	server 192.168.2.12:8080;
    server 192.168.2.13:8081;
    fair;
}
```



## 后续

本文主要是介绍了负载均衡的部分配置，通过讲述不同的负载均衡策略，来看nginx的不同配置，在其他类型的负载均衡软件中，还有其他类型的负载策略，本文主要是讲nginx的配置，其他的负载策略就不再展开来讲。敬请期待后续。



[个人博客](http://b.zf520.net)

