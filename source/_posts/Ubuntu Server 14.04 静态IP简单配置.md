---
title: Ubuntu Server 14.04 静态IP简单配置
date: {{ date }}
tags: Crawler
categories:
toc: false
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

# 一、配置静态IP地址

```
vi /etc/network/interfaces
```

原内容有如下4行：

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

以上表示默认使用DHCP分配IP，修改为如下：

```
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
#iface eth0 inet dhcp

iface eth0 inet static
address 192.168.80.129
netmask 255.255.255.0
gateway 192.168.80.2
```
保存退出。
>注意：只需要设置address（IP地址）、netmask（子网掩码）、gateway（网关）这三项就OK，network和broadcast这两项参数是可以不写的。

>如果使用的是VMWare或Virtual Box虚拟机搭建的Ubuntu Server虚拟机，网关地址与本机电脑的网关相同即可。
>IP跟本机电脑的IP在相同的地址段即可。

<center>![](http://i.imgur.com/2eYCCJw.jpg)</center>

# 二、手动设置DNS服务器

```
vim /etc/resolv.conf
```

添加如下内容（这点所有Linux发行版都通用）：

```
nameserver 192.168.80.2
nameserver 8.8.8.8
```
保存退出。

注意：重启Ubuntu后发现又不能上网了，问题出在/etc/resolv.conf。重启后，此文件配置的dns又被自动修改为默认值。所以需要永久性修改DNS。方法如下：

```
vim /etc/resolvconf/resolv.conf.d/base
```

```
nameserver 192.168.80.2
nameserver 8.8.8.8
```
# 三、重启networking服务使其生效

```
/etc/init.d/networking restart
```

这样网络配置就永久生效。

然后重启电脑即可。

