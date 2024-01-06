---
layout: post
title:  "nginx常用配置-IV"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-08 08:50:35 +0800
categories: Linux
author: ZF520
---

## 摘要

书接上回，本文主要介绍代理后端服务器之代理websocket的一些配置



## http server部分

上周的时候，我们介绍了http的请求配置，http请求是现在适用最广泛的一个协议，但一般的http请求都是短连接的，即：http请求后端服务，建立连接，发送数据，后端服务处理，返回数据，断开连接。此处应该有图。

不过，作为一个软件开发人员，我们应该明白，建立http请求是十分耗时的，如果一个页面频繁的与后端服务进行交互，每次建立连接都要浪费很多时间，这对用户不友好，也增加了后台服务的压力。这个时候，W3C组织提出了websocket，一个页面上的长连接，可以在页面的生命周期中都可以进行交互。那么，如何对websocket进行代理哪？

### websocket介绍

websocket是比较特殊的http连接，websocket会像简单的http请求一样，先使用普通的连接进行连接，但在请求头中增加了一个请求头： Connection:upgrade，表示升级协议，转为websocket协议，服务器接收到请求后，会解析Connection:upgrade请求头，发现该请求头后，在服务端就不会触发发送完数据后断开连接，也就是继续保持连接，等待后续数据的交互。数据返回到客户端，状态码是101，这个时候，客户端也明白了当前连接不应该触发断开的消息。

这样，服务器和客户端都不会主动触发websocket的断开，只有当客户端或者服务端明确的发出websocket.close()方法，才会关闭连接。

### websocket代理配置

配置websocket代理，和配置http代理类似。不过，需要注意的是，我们要先在server的同级配置一个这样的配置

```nginx
map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }
```

这段配置是什么意思哪？map指令主要是用来匹配自定义字段的数据。比如上述配置，其中http_upgrade是nginx内置的参数，当遇到当前参数后，默认（ default）数据给 connection_upgrade参数赋值为upgrade，当遇到为空字符串时，则给close值。这两个值一个是http传递的数据close，一个是websocket的传递的数据upgrade。有了这两个值，我们再对websocket进行配置。

```nginx
location /ws/ {
            proxy_pass http://192.168.2.19:81/ws/;
            keepalive_timeout  3600s;
            proxy_read_timeout  3600s;
            proxy_http_version	1.1;
            proxy_set_header Upgrade	$http_upgrade;
            proxy_set_header Connection	"upgrade";
            proxy_set_header Host	$host:$server_port;
            proxy_set_header X-Real-IP	$remote_addr;
            proxy_set_header	X-Forwarded-For	$proxy_add_x_forwarded_for;
}
```

仔细查看当前配置，和http的配置基本一致，尤其是proxy_pass上配置的是http开头的地址，虽然我们在测试的时候，都是用ws协议的地址（ws://192.168.2.19:81/ws/)，在上一节中，我们大概介绍了下websocket与http的关系，在nginx中，就是通过http的协议升级来实现的websocket，所以，这里，用http开头的地址进行配置，在下面的配置中，还有一些配置会升级当前的协议。

```nginx
proxy_set_header Upgrade	$http_upgrade;
proxy_set_header Connection	"upgrade";
```

上述两个配置就是告诉nginx要升级当前http协议为websocket协议，也是在请求头中增加相应的请求头，所以，nginx中就可以用相似的配置来处理websocket协议。

其中，还有一个配置，`proxy_http_version 1.1;` 由于http的协议之前还有一个http 1.0的规范，当时1.0的规范暂时还不支持长连接，所以，websocket的协议升级需要基于http的1.1协议规范。

### websocket配置需要注意的点

虽然http的协议升级为长连接的websocket，但在nginx中，当一个连接长时间没有数据交互的时候，nginx中会自动断开，虽然可以配置`keepalive_timeout  3600s;`，这个配置是告诉nginx，在这个时间内，要一直保持连接不要断开，但是，超过当前时间，连接还是会被断开，这个时候，我们可以在服务器和客户端直接增加一个心跳机制，保证服务器和客户端在一定时间内都会有数据进行交互，保证连接不会被nginx断开。

## 后续

本文主要是介绍了如何配置websocket的代理，在文中，我们简单介绍了下websocket与http的关系，没有深入，我们将在nginx配置完结后，把一些在配置中关联到的知识点再单独开一些篇章来仔细聊一聊。敬请期待后续。



[个人博客](http://b.zf520.net)

