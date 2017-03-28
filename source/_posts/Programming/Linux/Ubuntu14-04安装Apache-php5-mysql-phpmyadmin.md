---
title: Ubuntu14.04安装Apache+php5+mysql+phpmyadmin
date: 2016-07-24 00:39:41
tags: Linux
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、安装apache2
1. 打开终端：Ctrl+Alt+T,更新最新源：
	
	>sudo apt-get update

2. 通过apt-get方式安装Apache:
	>sudo apt-get install apache2//中途要输入y确认
	
　　检测安装是否成功: 在浏览器输入：127.0.0.1或locahost.浏览器上出现内容并有It works.说明安装成功！

注：

- Apache在Ubuntu中默认根目录为：/var/www/html
- 主配置文件目录为：/etc/apache2/apache2.conf
- 虚拟机配置目录为：/etc/apache2/sites-enabled

## 二、安装php5
1.安装php5和apachephp5模块
>sudo apt-get install php5 libapache2-mod-php5
>
>//其实libapache2-mod-php5可以不写，新版的源里已经自带了

2.安装好后，重启apache:
>/etc/init.d/apache2 restart

## 三、安装mysql
1.安装mysql
>sudo apt-get install mysql-server mysql-client

中途会出现以下提示：

<center>![](http://i.imgur.com/flBZohA.jpg)</center>

设置mysql密码。

2.测试mysql是否安装成功
>mysql -u root -p 

输入mysql root 密码即可

3.为PHP安装mysql扩展和其它扩展。

查看所有扩展：
>sudo apt-cache search php5

找到自己要安装的模块名字，按如下格式输入命令:
>sudo apt-get install php5-mysql php5-curl php5-gd

重启apache.

## 四、安装phpmyadmin
1.下载phpmyadmin
>sudo apt-get install phpmyadmin

注：
>phpmyadmin会自动安装在/usr/share/phpMyAdmin下，需要将 phpMyAdmin文件夹拷贝到/var/www/html目录下

使用命令：
>sudo ln -s /usr/share/phpmyadmin/ /var/www/html/

浏览器输入localhost/phpmyadmin就可以看到管理数据库的界面了。

配置完成！

