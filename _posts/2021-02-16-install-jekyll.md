---
layout: post
title:  "jekyll 搭建博客：安装过程及安装中的问题排除"
description: jekyll , github pages,博客搭建，问题排查，安装过程
tag: [jekyll,github pages,github,blog,博客,ruby,搭建过程,搭建问题,zf520,java思录]
date:   2021-02-16 08:50:35 +0800
categories: Java
author: ZF520
---
笔者最近开始想着记录一些工作过程中解决问题的过程，之前也经常记录一些文档，但都十分零散，没有系统性的进行整理，正好就着本次机会，搭建了一套以[github pages](https://pages.github.com/)，[jekyll](https://jekyllrb.com/)(中文网站：[https://jekyllrb.com/](http://jekyllcn.com/)) 为基础的博客网站。
其中，[github pages](https://pages.github.com/)作为发布存储位置，提供了类似ftp的空间存储，其利用[github](https://github.com)作为代码仓库或者文件仓库的功能，提供文件存储功能;[jekyll](http://jekyllcn.com/)为github提供的基于[ruby](https://www.ruby-lang.org/en/)开发的静态网站文件生成工具，该工具可以解析mardown文件为html文件；本文就是在此场景下进行编写的，下文的行文主要是以搭建过程的一些问题和解决思路为写作思路。主要工作环境是在centos7的环境下进行处理的。
首先，[Jekyll](https://jekyllrb.com/)是基于ruby开发的，需要首先安装ruby程序，使用yum安装ruby 
{% highlight shell %}yum install ruby-full build-essential zlib1g-dev {% endhighlight  %}
安装完[ruby](https://www.ruby-lang.org/en/)后，使用
{% highlight shell %}ruby -v {% endhighlight %}
查看ruby的版本，发现centos7默认安装的版本不符合[Jekyll](https://jekyllrb.com/)的要求，为2.0.0的版本，而[Jekyll](https://jekyllrb.com/)（版本为4.2.0）要求的版本为2.4.0及以上，还要包含development headers的（即开发版）。
如果不升级[ruby](https://www.ruby-lang.org/en/)的版本，就要安装[Jekyll](https://jekyllrb.com/)的低版本程序，使用命令
{% highlight shell %}gem install jekyll -v '2.0.0'{% endhighlight  %}
而在安装过程中，发现2.0.0的版本安装出现错误，

笔者也没有仔细去搜该版本错误信息要如何解决，既然低版本的jekyll安装不成功，那就直接用高版本，或者最新的版本。既然要用最新的版本，就要升级[ruby](https://www.ruby-lang.org/en/)的版本。
升级ruby，离线安装等其他方式，十分容易出问题，需要使用rvm的方式对ruby进行升级。升级前，首先把gem的源修改为国内的源，
{% highlight shell %}
gem sources --查看当前使用的源地址
gem sources -a http://mirrors.aliyun.com/rubygems/ 　　 --添加阿里云镜像地址
gem sources -r https://rubygems.org/ 　　 --删除默认的源地址
gem sources -u 　　 --更新源的缓存
{% endhighlight  %}
安装RVM：
{% highlight shell %}curl -sSL https://get.rvm.io | bash -s stable --ruby{% endhighlight  %}
国内使用该地址进行安装rvm，会发现 地址无法访问，打开浏览器，复制该地址到地址栏中直接输入，发现该地址进行了重定向，且指向了github的仓库，地址为https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer , 此地址为github的预览的地址，在国内查看的话，要看github的心情，笔者连续几天的查看，都无法打开该地址（同样的情况，查看其他的文件或者下载程序，也无法进行）。既然是[github](https://github.com)的地址，那我们可以直接找到该文件的仓库，打开[github](https://github.com)，搜索rvm包，找到binscripts目录下rvm-installer文件，直接打开，可以预览到具体的文件内容，拷贝出来，在本地新建一个文件，粘贴保存（wq），修改该文件的权限
{% highlight shell %}
sudo chomod u+x file.sh --改变文件的权限  
./file.sh  -- 执行该文件
source ~/.bashrc
source ~/.bash_profile -- 刷新环境变量
{% endhighlight  %}
列出已知的ruby版本
{% highlight shell %}rvm list known{% endhighlight  %}
安装ruby 
{% highlight shell %}rvm instal 2.7{% endhighlight  %}
安装成功。
此时，
{% highlight shell %}ruby -v{% endhighlight  %}
展示的版本为2.7版本。
既然ruby的版本已经适配了，继续执行
{% highlight shell %}gem install jekyll {% endhighlight  %}
等待安装，发现也可以执行成功了。
使用
{% highlight shell %}jekyll -v{% endhighlight  %}
jekyll的版本为4.2.0，已经安装完成了。
初始化一个博客
{% highlight shell %}jekyll new myblog{% endhighlight  %}
预览初始化的博客
{% highlight shell %}jekyll serve {% endhighlight  %}
默认的是本地预览，即只能用127.0.0.1来进行访问，端口为4000，
如果需要其他的机器来访问，可以指定--host 和--port（ip地址和端口）
{% highlight shell %}jekyll serve --host 0.0.0.0 --port 8080{% endhighlight  %}
PS:由于笔者是在linux下进行的安装，而在windows下进行的访问，而linux默认是开启着防火墙的，其他机器是访问不到的，需要关闭防火墙
{% highlight shell %}
systemctl stop firewalld --  关闭防火墙
systemctl disable firewalld -- 禁止开机自启动防火墙
{% endhighlight  %}
行文至此，安装过程已完成，其中主要的问题载域centos7默认安装的[ruby](https://www.ruby-lang.org/en/)版本与最新的[Jekyll](https://jekyllrb.com/)版本不一致导致的一些问题，只要解决了ruby的升级问题，本文也算完成任务了。
此后文章，会再介绍一些[Jekyll](https://jekyllrb.com/)写作博客或者是设置上的一些问题。期待下次再见。
