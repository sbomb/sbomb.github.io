---
layout: post
title:  "nginx常用配置-V"
description: nginx常用配置
tag: [nginx,config,zf520,java思录]
date:   2024-01-09 08:50:35 +0800
categories: Linux
author: ZF520

---

## 摘要

书接上回，本文主要介绍代理https的配置。



## http server部分

互联网发展至今，人们对安全的关注越来越高，http的协议，在网络上传输，实际上是明文传输，任何人获取到数据后，就很容易看懂数据传输的是什么，于是，安全专家们就又制订了一个新的标准--https标准，https标准也是在http协议的基础上实现的，与上文讲的websocket类似，不同的是，websocket升级的是网络连接上的机制，而https协议是对传输数据做了一个新的规定。

### ssl配置

https的配置，是针对当前server下的所有请求数据都是有效的，所以，ssl的配置信息，是在server下级，与location指令同级。

一个标准的ssl配置如下：

```nginx
        ssl_certificate public.crt; 
        ssl_certificate_key private.key; 
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
        ssl_prefer_server_ciphers on;
```

ssl_certificate是公钥，是提供给用户的文件，后面配置的是相应文件路径即文件名，可以写绝对地址，也可以写相对位置。

ssl_certificate_key 为私钥文件，是服务器用来加密和解密数据的文件。这个文件是隐私的，不能对外提供。

本文讲的公钥和私钥，是非对称加密的两个加密文件，用公钥加密，用私钥解密，用私钥加密，需要用公钥解密，公钥和私钥是一对数据，要同时生成，如果不匹配的公钥和私钥数据，是无法进行解密数据的。一般的非对称加密算法有RSA、AES128、DES等等算法。

ssl_session_timeout	5m; 表示ssl的对话时间是5分钟。

ssl_protocols：代表支持的https的协议，可选的有TLSv1  、TLSv1.1  、 TLSv1.2等。

ssl_ciphers： 表示可以使用的加密算法。

### SSL证书

ssl证书，是需要向机构进行申请的，虽然也可以自生成一个证书，但此证书并不会被windows系统或者终端系统认可，需要用户手动确认是否认可该证书。

SSL证书是有颁发机构的，只有被终端系统认可的颁发结构才能颁发证书，在浏览器上，可以看到办法机构，也可以看到颁发对象，只有域名与颁发对象一致的域名才能显示绿色的安全的连接。

SSL证书一般是颁发对象对应的证书，但当你有很多子域名证书的时候，就需要通配符SSL证书，即可以保护主域名下的二级域名证书。

如果你有很多域名，那么可以申请多域名证书。一般是可以输入250个左右的域名。

如果您需要购买SSL证书，而您不太熟悉的话，可以联系笔者。

### 自生成SSL证书

如果只是想测试下SSL证书及https的可用性，那么我们可以自己生成一个SSL证书。

命令如下：

```shell
$ openssl req -newkey rsa:2048 -nodes -keyout rsa_private.key -x509 -days 365 -out cert.crt
Generating a RSA private key
............................................................+++++
..........................+++++
writing new private key to 'rsa_private.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Hubei
Locality Name (eg, city) []:WH
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ZFSoft.ltd
Organizational Unit Name (eg, section) []:zfsoft
Common Name (e.g. server FQDN or YOUR name) []:zf520.net.cn
Email Address []:zengfei@zf520.net.cn
```

上面的一些数据主要是填写国家、省份、城市、组织机构等等数据，而后面的Common Name，则是输入您的域名，Email Address则是联系人的邮件地址（当日，您也可以通过这个邮箱地址联系到我。）。

上述命令输入后，并输入您的一些基本信息后，就会在当前目录下生成rsa_private.key 和cert.cert两个文件，配置到您的nginx配置中即可。

PS：友情提醒，自签名证书，虽然可以在系统中使用，但会操作系统会对用户进行提示，告知用户这是一个不安全的连接，需要用户手动确认继续连接。此方法可以用于测试，但不建议用于生产环境。

## 后续

本文主要是介绍了如何配置SSL连接，由于https是后续互联网的基本配置，所以，我们简单的介绍了下如何申请SSL证书，及自签名证书。敬请期待后续。



[个人博客](http://b.zf520.net)

