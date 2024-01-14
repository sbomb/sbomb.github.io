---
layout: post
title:  "nginx常用配置-IX"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-15 08:50:35 +0800
categories: Linux
author: ZF520

---

## 摘要

书接上回， 本讲主要是讲一讲nginx代理tcp/udp流量。



## stream部分

### 前言

此前的所有的配置，基本上都是配置的是http的部分，这部分流量也都算是tcp的流量，不过，这些都是应用层上的数据，我们可以通过应用层上的应用，比如http协议来解析数据，根据协议中的比如http的URL请求来分发请求，也可以根据HTTP的header的数据来进行数据的代理和处理，或根据请求的对端如请求用户的IP地址来转发到系统不同服务器上。

在互联网上，还有大部分的流量是tcp和udp的流量，这些流量，有的可能是自定义的协议，也有其他的比如FTP或者DNS等协议的流量，这些流量并没有HTTP协议那么常用，使用url、header或者ip等来进行代理或者处理比较复杂。但使用nginx同样可以代理这些流量到后端服务器。

### 前提

使用stream的配置，必须要nginx启用--with-stream模块，之前的时候，我们提到过，要如何查看nginx编译时所有启用的模块。  `nginx -V`  ，命令仅做参考，注意V是大写，如果使用的是小写的v，那么输出的数据就是nginx的版本号。-V的输出中如果含有--with-stream模板，则您可以继续想下看，如果您的输出中，没有--with-stream模块，则请参照之前的文章[知乎-- nginx源码安装](https://zhuanlan.zhihu.com/p/674919774) , [个人博客--nginx源码安装](http://b.zf520.net.cn/2022/11/nginx%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85.html)。

此处需要增加一个提醒，在现有的版本中，windows版本的nginx暂不支持--with-stream模块，意思就是，即便是在windows的版本中看到了--with-stream的模块，也不会支持，增加相应的配置后，使用nginx -t的命令来测试配置文件后，会提示如下：

```shell
E:\soft\nginx-1.21.6>nginx.exe -t
nginx: [emerg] "stream" directive is not allowed here in *****/conf/nginx.conf:141
nginx: configuration file ******/conf/nginx.conf test failed
```

### stream指令

先把配置放上来，再进行解释。

```nginx

stream {
    server {
        listen 53 udp ;
        proxy_timeout 60s;
        proxy_pass 192.168.1.2;
    }
}
```

请注意，stream是与http同级的，也是在根目录下的。

listen 指令，是要开启监听的端口，默认是tcp协议，如果要开启udp协议，那么需要在后面进行指明协议。

proxy_timeout  表明代理的超时时间，当链接在60s内，如果双方没有数据交互，那么，链接就会被断开。

proxy_pass和http的代理是一样的，表示要把请求转发到的服务器上。



上面介绍的是常用的tcp或udp链接，如果，要使用集群的话，配置和http中的配置基本是类似的。

```nginx
stream {
	upstream mysql {
		192.168.1.2
		192.168.1.3
	}
	server {
		listen 3306;
		proxy_timeout 60s;
		proxy_pass mysql;
	}
}
```

在upstream中，我们也可以做一些简单的负载均衡策略，比如轮询，加权轮询等，但似请求url的一些负载策略则无法使用，在tcp和udp协议中，一些http协议中的数据是无法获取到的。

### 案例

在一个局域网内有一台mysql的服务器，我们需要在另一局域网内访问当前的mysql，由于网络不能直通，但有一台服务器中有双网卡服务器，双网卡服务器链接在两个局域网中。那么，我们就可以在当前的双网卡服务器中架设一个nginx服务(linux版本)。

通过使用nginx的tcp代理，就可以访问到mysql数据，即可以使用mysql的协议来访问nginx所在的服务，并代理到实际的mysql服务器上。

```nginx
stream {
	server {
		listen 3306;
		proxy_timeout 60s;
		proxy_pass 192.168.1.2;
	}
}
```



#### 扩展

讲到使用nginx服务来代理服务，实际上，使用putty之类的ssh服务登陆软件也可以解决类似的问题，很多事情，一些公司会提供给外网的访问者一个堡垒机器，此时，很多人想要使用本地的mysql客户端来链接数据库，（主要是不想使用命令行来访问数据库，虽然对一些人员来讲，命令行也可以使用，但对数据筛选来讲，还是有点不太容易查看。）那么，在putty的connection-ssh-tunnels中进行配置source port 就是配置listen中的参数，destination的就是目的ip与端口，add后，就类似我们上述的配置。

## 后续

本文主要是介绍了nginx代理tcp与udp协议的通用服务器，最后，我们通过一个mysql的案例来讲述了下如何进行配置。在最后的最后，我们做了下扩展，有很多朋友应该会遇到类似的问题，毕竟，能对外少开一些端口，就会少很多风险。敬请期待后续。



[个人博客](http://b.zf520.net)

