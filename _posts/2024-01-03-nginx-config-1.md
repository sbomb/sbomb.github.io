---
layout: post
title:  "nginx常用配置-I"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2023-12-26 08:50:35 +0800
categories: Linux
author: ZF520
---

## 摘要

主要介绍一些常用的配置选项，以及一些可能遇到坑。



## http server部分

http的server部分，是一个监听的重要部分，server是http节点下的一个重要的部分，server定义了http监听的端口，访问路径及规则。

### 配置一个静态服务器

我们来看一下一个简单的配置

```nginx
http {
	server {
		listen		81;
		location / {
			root	/opt/zfsoft/front/;
			index	index.html;
		}
	}
}
```

上面就配置了一个简单的静态服务器，监听的端口是81，访问的根路径会指向root下的目录文件，当直接使用 / 访问时，默认查找root下的index.html文件，其他的访问路径，都会去查找当前路径下相应的文件名。

比如 访问 /a.html ，则会去/opt/zfsoft/front/下查找 a.html文件，找到a.html文件，则返回  a.html文件的内容，若未找到，则返回404错误，并显示一个错误信息（由nginx定义）。

如果要自定义显示404或者500信息，则可以增加如下配置

```nginx
error_page  404              /404.html;
```

当前配置与location平级，合起来的配置信息如下：

```nginx
http {
	server {
		listen		81;
		location / {
			root	/opt/zfsoft/front/;
			index	index.html;
		}
		error_page 404	/404.html;
	}
}
```

同样的道理，我们也可以配置相似的500错误。可以使用如下方式`error_page 500 502 503  	/50x.html` 进行配置，同样的是在根目录下找相应的文件。

此处，可以这样理解，在触发了404或者500 502 503 等错误码时，nginx会自动重定向到相应的页面，比如/404.html  /50x.html的访问，再次通过location指令找到相应的文件来返回给前端进行展示。

### 配置一个反向代理

说完了一个静态页面的配置，毕竟我们的系统不是一个简单的html页面的系统，还有一些后台需要进行交互的操作或者数据，那么，我们要访问一些动态数据，虽然可以在前端页面写死一些url请求的地址，但这样，不就把我们的后台交互地址给暴露出去了吗，还有就是，如果我们把相同的程序给其他人用，或者需要迁移到其他的后台服务上，难道又要把所有的地址替换一遍。

不不，不要那么粗鲁，我们可以使用反向代理的方式来解决这个问题，可以直接在页面上写根据当前路径或者从根路径进行访问的地址，如/hello请求，当请求到达服务器端时，由于nginx监听着相应的端口，nginx就接收到了当前的访问，当去location指令下的根目录去查找hello文件时，发现没有当前文件，就会抛出404，噢，不对，我们忘记加配置信息了。

那么，由于/hello请求不是一个静态的地址，而是另一个服务器的URL请求，需要转发到相应的服务器上，那么，为了不让nginx抛出404错误，那么，我们就要增加相应的/hello的请求路径，让它能找到相应的服务。

```nginx
location /hello{
	proxy_pass http://192.168.2.12:8080/;
}
```

有了如上的配置，proxy_pass 指令就把当前的请求通过http请求的方式去请求http://192.168.2.12:8080/hello。

## 后续

先介绍些简单的nginx配置，敬请期待后续。



[个人博客](http://b.zf520.net)

