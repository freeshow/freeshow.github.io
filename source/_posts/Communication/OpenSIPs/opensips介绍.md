---
title: opensips介绍
date: 2016-07-23 20:20:00
tags: SIP
categories: 网络通信
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、OpenSIPS简介
OpenSIPS是一个成熟的开源SIP服务器，除了提供基本的SIP代理及SIP路由功能外，还提供了一些应用级的功能。OpenSIPS的结构非常灵活，其核心路由功能完全通过脚本来实现，可灵活定制各种路由策略，可灵活应用于语音、视频通信、IM以及Presence等多种应用。同时OpenSIPS性能上是目前最快的SIP服务器之一，可用于电信级产品构建。凭借其可扩展、模块化的系统架构，OpenSIPS提供了一个高度灵活的、用户可配置的路由引擎，可以为voice、video、IM和 presence等服务提供强大高效的路由、鉴权、NAT、网关协议转化等功能。由于其稳定高效等特点，OpenSIPS已经被诸多电信运营商应用在自己的网络体系中。其 主要功能如下：

- SIP注册服务器/代理服务器（lcr、dynamic routing、dialplan）/重定向服务器
- SIP presence agent
- SIP B2BUA
- SIP IM Server
- SIP to SMS/XMPP网关
- SIP to XMPP网关
- SIP 负载均衡
- SIP NAT traversal


## 二、OpenSIPS开源官网
OpenSIPS开源网址：http://www.opensips.org/

## 三、OpenSIPS系统架构
OpenSIPS的架构开放灵活，其核心功能控制均可通过脚本控制实现，各个功能也通过模块加载的方式来构建。采用lex和yacc工具构建的配置文件分析器是其架构设计中的重要部分之一。通过这个分析器，opensips设计了自己的语法规则，使得我们可以适合SIP规范的语言来进行配置文件中的脚本编写，从而达到简化程序以及方便代码阅读的目的。同时这样的设计也使opensips.cfg配置文件的执行速度达到了C语言的级别。其体系结构大体如下图：

![这里写图片描述](http://img.blog.csdn.net/20160325102421018)

OpenSIPS框架的最上层是用于实现sip消息路由逻辑的opensips.cfg脚本配置，在配置文件中，可以使用Core提供的Parameter和Function，也可以使用众多Modules提供的Function。比如在之前的负载均衡示例中，is_method(“INVITE”)就属于textops模块提供的功能，src_ip和src_port都属于Core提供的参数。

下层，提供了网络传输、sip消息解析等基本功能。

在左侧，通过相应的数据库适配器，可是使用多种数据库存取数据。

在这样的体系结构下，我们就可以方便地通过增加功能module来添加我们需要的功能，而不会对原有系统造成影响。

除了以上所述的OpenSIPS的优点，OpenSIPS还提供了一系列的管理维护命令的接口。我们可以通过Core和Module提供的MI管理接口，方便的监控系统以及模块的状态。比如，通过Core的fifo ps命令，可以获取当前进程的状态；通过Core的fifo get_statistics命令，可以获得当前共享内存以及各进程私有内存的使用情况等等。通过MI管理接口，我们还可以方便地在运行时修改部分参数，比如，对于load_balancer模块，我们可以通过fifo lb_reload命令，更新目标组的配置信息，可以通过fifo lb_status命令激活或关闭某个目标，这些命令在实际应用中都非常实用。


转载自：[opensips](http://m.blog.csdn.net/article/details?id=48177299)
