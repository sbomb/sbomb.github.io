---
layout: post
title:  "nginx常用配置-VI"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-10 08:50:35 +0800
categories: Linux
author: ZF520

---

## 摘要

书接上回，本文介绍下静态网页的一些内容。

虽然nginx对静态资源的访问很快，但一些数据还是可以再次加快访问速度的，有哪些方面的优化是我们可以在nginx中进行的处理的那，且听笔者和你慢慢道来。

## http 部分

### 压缩

今天所讲部分还是http部分。第一部分就是要对页面的传输进行压缩。压缩是有前提条件，如果您是自己源码编译的nginx，需要查看您的模块是否包含gzip模块，您可以使用如下命令进行查看

```shell
$ nginx -V
nginx version: nginx/1.21.6
......
configure arguments: --with-http_gunzip_module --with-http_gzip_static_module
```

如果您的输出中含有这些模块，那么就可以使用压缩指令。一般编译nginx是默认开启这些模块的，如果您在windows上编译，可能会提示您找不到gzip库，让您用--without-gzip编译参数忽略压缩。所以，一般不需要担心压缩没有启用。

压缩部分配置如下：

```nginx
gzip on; #开启gzip压缩输出
gzip_min_length 1k;
gzip_http_version 1.1;
gzip_comp_level 2;
gzip_types text/plain application/x-javascript text/css application/xml;
```

先讲解下压缩的指令

首先要让nginx明白，我要启用压缩，所以，第一步就要配置一个gzip on;的配置，只有启用了压缩，下面的配置选项才能生效。

启用了压缩，那要对哪些文件进行压缩哪？我们需要配置gzip_types ，后面跟着的就是返回的头中的类型content-type为 text/plain (纯文本) ， applicaiton/x-javascrip （js脚本） ， text/css（css样式文件），applicaiton/json（json格式文件) ， applicaiton/xml （xml格式文件）等等，nginx就会对我们配置的请求类型的返回数据进行压缩。

这时，可能有朋友就要问了，如果我没有配置这些类型，nginx会不会把所有的返回数据都压缩了？抱歉，不会，nginx会有一个默认的配置text/html，html页面的会被默认压缩。也不一定，还有一个条件，就是下面我们要讲的gzip_min_length 配置项。

gzip_min_lenght配置项，后面跟着的是多大的文件我们才需要压缩，如果文件太小，使用数据压缩的话，可以压缩的时间都要比传输的时间还要长，有点得不偿失，所以，小与一定大小的文件，nginx是不会进行压缩的，我们可以设置1k，1M等等，默认值为0，无论页面多大，都会被压缩；建议设置大于1k。

既然是压缩要浪费CPU的，是不是有压缩的级别可以配置，是的，gzip_comp_level就是可以设置压缩的级别，可以平衡压缩与数据大小的关系，1级压缩比最小,处理速度快，压缩比较小，数据包比较大，9级压缩比最大,比较消耗cpu资源,处理速度最慢,但是因为压缩比最大,所以包最小,传输速度快。

比较完整的nginx配置

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;



    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    
    server {
        listen       81;
        server_name  localhost;

        location / {
            root   /opt/zfsoft/front/;
            index  index.html index.htm;
        }
    }
}
```



PS：小知识点，如果nginx在配置过程中发现一些指令配置错误，可以看看该指令在那个模块，如果nginx编译过程中没有编译该模块，则指令则无法使用。

使用nginx -V可以查看nginx中含有的模块，看看是否有 --with--(模块名)。如果没有，则可以重新编译nginx，增加相应的模块。



## 后续

本文主要是介绍了nginx对页面或数据的压缩，配置压缩的一些指令。敬请期待后续。



[个人博客](http://b.zf520.net)

