---
title: Python开发简单爬虫之爬虫介绍
date: 2016-07-24 16:00:38
tags: Crawler
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

本博客来自慕课网---[Python开发简单爬虫](http://www.imooc.com/view/563)

爬虫主要场景:
- 不需要登录的静态网页
- 使用Ajax异步加载的内容
- 需要用户登录才可以访问的网页

以下主要介绍 `不需要登录的静态网页`。

# 一、爬虫简介以及爬虫的技术价值

## 1. 爬虫是什么

<center>![](http://i.imgur.com/3yzbpF9.jpg)</center>

----------


## 2. 爬虫技术的价值

<center>![](http://i.imgur.com/6QRgBJU.jpg)</center>

# 二、简单爬虫架构

## 1. 简单爬虫架构

<center>![](http://i.imgur.com/mNcxZ03.jpg)</center>

- 爬虫调度端：开启爬虫、终止爬虫、监视爬虫的运行情况
- URL管理器：将要爬取的URL和已经爬取过的URL进行管理
- 网页下载器：从URL管理器中的URL中下载网页，并生成字符串
- 网页解析器：解析出网页下载器的中内容。一方面解析出有用的价值数据；一方面，网页中都存在链接，将解析出的链接，有送回到URL管理器。

URL管理器、网页下载器、网页解析器，就形成了一个循环，只要有URL就会一直运行下去。就会将互联网上所有相关联的网页都爬取下来。


## 2.简单爬虫架构的动态运行流程

<center>![](http://i.imgur.com/mg9CLgo.jpg)</center>

# 三、URL管理器和实现方法

## 1.URL管理器

<center>![](http://i.imgur.com/L2U9sbj.jpg)</center>

- 添加新URL到待爬取URL集合中，首先判断待添加的URL是否在URL管理器中，如果在，则不添加；如果不在，则可以添加。
- 获取待爬取URL，首先判断URL管理器中是否有带爬取的URL，如果有，则获取待爬取URL，并将URL从待爬取URL移动到已爬取URL。如果没有，则说明爬取结束。

## 2. URL管理器的实现方法

3中实现方式：

<center>![](http://i.imgur.com/8v4Ofpy.jpg)</center>

# 四、网页下载器和urllib2模块

## 1. 网页下载器简介

<center>![](http://i.imgur.com/QMYXrh9.jpg)</center>

<center>![](http://i.imgur.com/Gg4MHBi.jpg)</center>


## 2.urlib2下载器网页的三种方法
### 方法一
<center>![](http://i.imgur.com/fFsvtzA.jpg)</center>

<center>![](http://i.imgur.com/b6weheu.jpg)</center>

### 方法二
<center>![](http://i.imgur.com/GIAZbrZ.jpg)</center>

<center>![](http://i.imgur.com/9oBkc8a.jpg)</center>

### 方法三
<center>![](http://i.imgur.com/4Wp3rnN.jpg)</center>
<center>![](http://i.imgur.com/kS3Yzxo.jpg)</center>

----------

## 3.urlib2实例代码演示
<center>![](http://i.imgur.com/uXJ8QbO.jpg)</center>

# 五、网页解析器和BeautifulSoup第三方库

## 1. 网页解析器

<center>![](http://i.imgur.com/Z3eDGvg.jpg)</center>

<center>![](http://i.imgur.com/Nr2s3ht.jpg)</center>

<center>![](http://i.imgur.com/spEByBx.jpg)</center>

## 2. BeautifulSoup模块介绍和安装

<center>![](http://i.imgur.com/CZG0YAp.jpg)</center>

## 3. BeautifulSoup的语法
### BeautifulSoup语法
<center>![](http://i.imgur.com/MvgIxph.jpg)</center>

<center>![](http://i.imgur.com/zSemshi.jpg)</center>

### 创建BeautifulSoup对象
<center>![](http://i.imgur.com/32VBhPh.jpg)</center>

### 搜索节点find_all、find
<center>![](http://i.imgur.com/hS3Sjlg.jpg)</center>

python中已经存在关键字class,故当属性为class时，用class_代替。

### 访问节点信息
<center>![](http://i.imgur.com/osC8d44.jpg)</center>

## 4. BeautifulSoup实例测试
	
	# -*- coding: utf-8 -*-
	import re
	from bs4 import BeautifulSoup

	html_doc = """
	<html><head><title>The Dormouse's story</title></head>
	<body>
	<p class="title"><b>The Dormouse's story</b></p>

	<p class="story">Once upon a time there were three little sisters; and their names were
	<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
	<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
	<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
	and they lived at the bottom of a well.</p>

	<p class="story">...</p>
	"""
	def run_demo():
    	soup = BeautifulSoup(html_doc,'html.parser',from_encoding='utf-8')

    	print '获取所有的链接：'
    	links = soup.find_all('a')
    	for link in links:
        	print link.name,link['href'],link.get_text()

    	print '获取lacie的链接'
    	link_node = soup.find('a',href='http://example.com/lacie')
    	print link_node.name, link_node['href'], link_node.get_text()

    	print '获取正则匹配'
    	link_node = soup.find('a', href=re.compile(r'ill'))
    	print link_node.name, link_node['href'], link_node.get_text()
    	#a http://example.com/tillie Tillie

    	print '获取p段落文字'
    	p_node = soup.find('p', class_='title')
    	print p_node.name,p_node.get_text()


	if __name__ == '__main__':
    	run_demo()






