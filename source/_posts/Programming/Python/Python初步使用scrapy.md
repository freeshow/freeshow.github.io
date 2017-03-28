---
title: Python初步使用Scrapy
date: 2016-07-24 16:31:35
tags: 
- Crawler
- Python
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Scrapy at a glance

### 爬取[http://stackoverflow.com/questions?sort=votes]('http://stackoverflow.com/questions?sort=votes')页面的每个问题link,以及每个问题的第一答案的title,body,votes,tags.

### link对应的数据格式：

>response.css('.question-summary h3 a::attr(href)')

<center>![](http://i.imgur.com/Gnr3aKW.png)</center>

### title对应的数据格式

>response.css('h1 a::text')

<center>![](http://i.imgur.com/Nv6YohK.png)</center>

### body对应的数据格式

>response.css('.question .post-text')

<center>![](http://i.imgur.com/Wek8xhr.png)</center>

### tags对应的数据格式

>response.css('.question .post-tag::text')

<center>![](http://i.imgur.com/tvGkwrv.png)</center>

### votes对应的数据格式

>response.css('.question .vote-count-post::text')

<center>![](http://i.imgur.com/e3nPFYR.png)</center>


### 代码示例

	# -*- coding: utf-8 -*-
	import scrapy

	class StackOverflowSpider(scrapy.Spider):
    	# 爬虫名称
    	name = 'stackoverflow'
    	#url入口
    	#start_urls = ['http://stackoverflow.com/questions?sort=votes']

    	#解析start_urls页面的所有questions的url.
    	def parse(self, response):
        	#获取每个link
        	for href in response.css('.question-summary h3 a::attr(href)'):
            	full_url = response.urljoin(href.extract())
            	#使用回调函数callback=self.parse_question,
            	# 解析questions中的每个question的url页面
            	#将link页面的response传给parse_question()
           		yield scrapy.Request(full_url, callback=self.parse_question)

    	#解析每个link的response.
    	def parse_question(self, response):
        	#获取每个question页面中的title,votes,body,tags,link
        	yield {
				#因为有好几个answer，因此只去第一个answer.
            	'title': response.css('h1 a::text').extract_first(),
            	'votes': response.css('.question .vote-count-post::text').extract_first(),
            	'body': response.css('.question .post-text').extract_first(),
            	'tags': response.css('.question .post-tag::text').extract(),
            	'link': response.url,
        	}


### run the spider

Put this in a file, name it to something like stackoverflow_spider.py and run the spider using the runspider command:

	scrapy runspider stackoverflow_spider.py -o top-stackoverflow-questions.json

When this finishes you will have in the top-stackoverflow-questions.json file a list of the most upvoted questions in StackOverflow in JSON format, containing the title, link, number of upvotes, a list of the tags and the question content in HTML, looking like this (reformatted for easier reading):

<center>![](http://i.imgur.com/xYwzioY.png)</center>


## 原理讲解

scrapy runspider somefile.py -o xx.csv

1. 在somefile.py文件中找到已定义的爬虫，然后通过抓取引擎运行爬虫。
2. 具体的抓取过程
  1. 使用start_urls作为初始url生成Request，并默认把parse作为它的回调函数。
  2. 在parse中采用css选择器获得目标URL,并注册parser_question作为目标URL的回调函数。

背后的处理：
1. 请求被异步调度、处理
2. 有一些参数可以控制过程，比如每个域名/ip的并发请求数、请求之间的下载延迟(或者自动调节)

## 高级特性
1. 内置的数据抽取器css/xpath/re
2. (scrapy shell)交互式控制台用于调试数据抽取方法
3. 内置对结果输出的支持，可以保存为json,csv,xml等
4. 自动处理编码
5. 支持自定义扩展
6. 丰富的内置扩展，可用于处理：
	1. cookies and session
	2. Http features like comparession,authentication,caching
	3. user-agent spoofing
	4. robots.txt
	5. craw depth restriction
7. 远程调试scrapy
8. 更多的支持，比如可抓取xml,csv,可自动下载图片等等。 

http://www.cnblogs.com/python-life/articles/4511314.html