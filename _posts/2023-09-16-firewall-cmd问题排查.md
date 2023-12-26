---
layout: post
title:  "firewall-cmd转发"
description: 负载均衡
tag: [负载均衡,网络转发,网络流量,七层负载,四层负载,zf520,java思录]
date:   2023-09-14 11:58:00 +0800
categories: 负载均衡
author: ZF520
---
# 摘要


# 正文
前期在现场遇到一起关于firewall端口转发的问题，特做如下记录。

系统需要对客户端的IP进行解析，以便在websocket上进行限制，限制了一个IP只能有一路websocket的访问，现场使用了firewall-cmd的端口转发，网络情况是有两个网络，两个网络之间通过一个中转机器进行互联，虽然网络之间由于某中转机器限制导致网络有可能拥堵，但至少解决了两个网络之间不能互通的问题。

表象：系统管理平台上显示多个用户访问后，只有一个websocket的连接，且前端表现为用户中前边登陆的显示websocket断开。

问题很明显，就是通过网络转发时，网关解析到数据时中转机器的IP，由于所有用户都是通过中转机器进行转发的网络流量。直接找到网络转发的配置人员，问清楚了是在中转机器上使用了firewall-cmd进行的端口转发。

解决：在中转机器上直接使用nginx进行转发，相当于内外网隔离，由于nginx是7层负载，可以在应用层上进行数据处理，而且前期有经验处理这个事情（由于其他现场也有用apache做代理转发，也是导致后端网关解析的IP都是代理的IP，于是经过多次协商后修改为nginx）。直接配置nginx（中转机器上也有nginx，故无需安装等）。

教训：由于对firewall的端口转发并不理解，并没有意识到端口转发的时候，会把source ip进行改写，导致问题的出现，需要再次加深对网络以及各防火墙，负载的理解和学习。



反思： 虽然问题解决了，但我不喜欢没有找到问题缘由就放过问题，于是，在网络上找了些资料，发现centos系统在使用firewall-cmd进行端口转发的时候，都需要先开启 ` firewall-cmd --add-masquerade` ，这是做什么用的？开启ip伪装，



firewall-cmd --add-masquerade  ## 开启ip伪装



firewall-cmd 增加转发规则

firewall-cmd --add-forwarded-port=port=80:proto=tcp:toport=8080:toaddr=192.168.4.121

firewall-cmd --reload (# 如果是使用的--permanent , 则需要使用reload，如果没有则不能使用reload，否则会把当前配置移除掉)



