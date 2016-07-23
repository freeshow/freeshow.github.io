---
title: FreeSwitch安装文档
date: 2016-07-23 16:56:10
tags: VoIP
categories: 
---

## 编译安装FreeSWITCH所依赖的Linux软件包
    
    apt-get -y install build-essential automake autoconf git-core wget libtool  
    apt-get -y install libncurses5-dev libtiff-dev libjpeg-dev zlib1g-dev libssl-dev libsqlite3-dev  
    apt-get -y install libpcre3-dev libspeexdsp-dev libspeex-dev libcurl4-openssl-dev libopus-dev 

<!-- more -->

## 解压FreeSwitch源码包安装
	//下载
	wget http://files.freeswitch.org/freeswitch-1.4.0.beta6.tar.bz2
	//解压  
	tar xvjf freeswitch-1.4.0.beta6.tar.bz2
	//安装  
	cd freeswitch-1.4.0 
	./configure  
	make install

## 安装声音文件
在源代码目录中执行：


> make sounds-install  
>make moh-install

##  建立符号链接

为了便于使用，建议将这两个命令做符号链接放到你的搜索路径中，如：

> ln -sf /usr/local/freeswitch/bin/freeswitch /usr/bin/  
ln -sf /usr/local/freeswitch/bin/fs_cli /usr/bin/

这样启动FreeSWITCH时，就可以使用如下命令：
> sudo freeswitch

连接FreeSWITCH控制台：
>sudo fs_cli

 

