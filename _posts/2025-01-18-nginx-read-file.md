---
layout: post
title:  "石头侃Nginx之配置本地文件访问"
description: 石头侃Nginx之配置本地文件访问
tag: [nginx,本地文件访问,zf520,java思录]
date:   2025-01-18 08:50:35 +0800
categories: Linux
author: ZF520
---
nginx用途非常多，但一般都是用在HTTP上，今天，笔者又发现了一个新用法。

### **背景**

最近，需要和三方进行对接，接口需要联调，因为双方的服务，都在自己的公司内网，而且两个公司的内网都不能进行[互联网访问](https://zhida.zhihu.com/search?content_id=250953508&content_type=Article&match_order=1&q=互联网访问&zhida_source=entity)，开始开发的时候，笔者使用接口模拟的方式，直接绕过了请求的访问，系统逻辑跑通了，但一直没有对接口进行访问，还是有点心慌慌。

于是想到了nginx既然能作为一个静态服务器，那么，也应该可以访问到本地的json文件（对接格式使用的json格式），于是，笔者找到三方的研发人员，让对方每个请求都用[postman](https://zhida.zhihu.com/search?content_id=250953508&content_type=Article&match_order=1&q=postman&zhida_source=entity)来请求了一遍，把每个请求的返回数据给保存了下来。

### **先说坑**

想到了解决方案，当然心情非常舒畅，于是，修改配置文件，先来测试一个简单的json吧。于是，手写了一个index.json，就返回了`{"key":"value"}`的数据，嗯。请求路径是对方写死了的，还需要模拟一个请求URL，于是，下面的配置文件出来了。

```nginx
location /abcd/efg/ab {
    alias /home/source/;
    index index.json;
    default_type application/json;
}
```

把index.json文件放到目录下，使用url进行请求，返回404，嗯？错误了？不应该啊？查看错误日志。

如图：

![img](https://pica.zhimg.com/v2-21c877f8115cabbf0863910f6ff5729e_1440w.jpg)

再查，再改，一直报这个错误，为什么有这个问题啊，不应该啊，这个目录是错误的啊，我明明配置了对应的目录了啊。找不到这个文件？why。

此处省略半小时的重复验证，试错。还是不行。有点没头绪。嗯。不用目录了，我直接走代理看一下，于是，我把配置修改成了

```nginx
location /abcd/efg/ab {
    proxy_pass http://127.0.0.1/json/index.json;
}
```

再来访问，好像也不行？嗯？？？？有点怀疑人生了。

嗯。站起来走走，哦。好像想起来了，下午的时候，因为[部署测试](https://zhida.zhihu.com/search?content_id=250953508&content_type=Article&match_order=1&q=部署测试&zhida_source=entity)的事情，在本机上用命令行启动了一次nginx，下班的时候，直接把窗口关掉就走了。于是，把当前进程给关闭，使用了nginx -s stop，刚刚启动的nginx关掉了，检查进程中的数据，嗯。还在，确实是这个问题，原有的进程没有关闭，一直在接收请求。

![img](https://pic3.zhimg.com/v2-6fd835b22f9dd5d85f455dfdb4a055e8_1440w.jpg)

也许有人会问，从笔者的截图中，可以看到有nginx -s reload 成功的信息啊。这个地方要说明下，-s 的命令，查找的进程，都是 log目录下的nginx.pid文件中的进程ID，因为下班后，又启动了一次nginx，所以，把原有的nginx.pid给覆盖掉了，导致nginx -s 的命令，并不能重新加载原来的进程了。

![img](https://picx.zhimg.com/v2-eea691bb42fd534ebf1f33fd921e7275_1440w.jpg)



还有一点点疑问，就是，因为端口被占用了，此时，nginx应该启动不起来才是，但第二个进程还是正常启动了。这个是点小疑问，笔者后续再来排查。

我们言归正传，再来说回这次想要解决的方案。

### **配置**

嗯。既然已经配置成代理了的方式了，那就用这种方式吧。于是，配置如下：

```nginx
location /a/test1 {
    proxy_pass http://127.0.0.1/json/test1.json;
}
location /a/test2 {
    proxy_pass http://127.0.0.1/test2.json;
}

location /json/ {
    alias /home/soft/;
    default_type application/json;
}
```

再次来访问，`curl http://127.0.0.1/a/test1` ,嗯。能正常返回json的数据了。

（PS：补充一点问题，在浏览器上以及curl使用的都是get请求，所以，nginx没问题，但如果使用了post方法，就会出现405请求，则，可以在json配置的地方，增加一行配置 error_page 405 =200 $uri ; 则可解决这个问题。）

### **总结**

配置nginx来访问json数据，这个方案并不是多大的难题，但在配置过程中，由于笔者自己的粗心，导致了本次故障的多次排查和[配置文件](https://zhida.zhihu.com/search?content_id=250953508&content_type=Article&match_order=3&q=配置文件&zhida_source=entity)的修改不生效问题，才是本次处理解决方案最大的收获。