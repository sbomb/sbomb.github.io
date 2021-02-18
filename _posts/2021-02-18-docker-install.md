---
layout: post
title:  "docker安装"
description: docker安装
tag: [docker安装,zf520,java思录]
date:   2021-02-18 08:50:35 +0800
categories: docker
author: ZF520
---

# 安装docker  
## 卸载旧docker  
首先卸载原有的docker 
{% highlight shell %}yum remove docker{% endhighlight %}
![docker remove ](https://img.zf520.net.cn/remove-docker.PNG)  
由于有一些docker的其他的包，先使用
{% highlight shell %}yum list installed | grep docker{% endhighlight %}  
查询一下，把docker的依赖也移除掉  
如果没有移除掉旧的版本，容易出现以下错误
![alt docker install error](https://img.zf520.net.cn/docker-install-error.PNG)

## 安装docker  
前提操作  
安装yum-util
{% highlight shell %}
yum install -y yum-utils \ 
    device-mapper-persistent-data \ 
    lvm2
{% endhighlight %}

增加docker-ce的yum镜像文件
{% highlight shell %}
yum-config-manager \ 
    --add-repo \ 
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
{% endhighlight %}
安装最新版本docker  
{% highlight shell %}
yum install docker-ce docker-ce-cli containerd.io{% endhighlight %}

如果需要安装特定版本的docker （docker-ce-18.09.1）  
{% highlight shell %}
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io    
{% endhighlight %}

# 安装docker-compose  
{% highlight shell %}
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
{% endhighlight %}
