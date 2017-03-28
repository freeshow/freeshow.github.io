---
title: Python爬虫实例：唐诗三百首
date: 2016-07-24 16:31:35
tags: 
- Crawler
- Python
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 介绍

唐诗三百首网址：http://www.gushiwen.org/gushi/tangshi.aspx

- 爬取诗词标题
- 爬取诗词作者
- 爬取诗词的网页地址
- 根据诗的网页地址爬取诗的正文。

## 步骤

### 分析数据格式

**诗的标题，作者，url的数据格式如下**：

<center>![](http://i.imgur.com/LQZ5D5l.png)</center>


>如上图所示，所要提取的数据在`<a>`标签中，但是光靠`<a>`标签，不能确定这一行，
>
>故需要借助`<a>`标签的父标签`<div>`来锁定`<a>`标签

**诗的正文数据格式如下**：
<center>![](http://i.imgur.com/FJlnhJu.png)</center>

>根据`<p>`标签和其属性align即可确定诗的正文。



### 编写代码



	# -*- coding: utf-8 -*-
	import requests
	from HTMLParser import HTMLParser
	import re

	def _attr(attrs,attrname):
    	for attr in attrs:
        	if attr[0] == attrname:
            	return attr[1]
    	return None

	#解析诗的标题，作者，url
	class PoemParser(HTMLParser):
    	def __init__(self):
        	HTMLParser.__init__(self)
        	self.tangshi_list = []
        	self.in_div = False
        	self.in_a = False
			#行宫(元稹)
        	self.pattern = re.compile(r'''(.+) #匹配标题 group(1)
                                        	\(  #匹配作者左边的括号
                                        	(.+) #匹配作者 group(2)
                                        	\) #匹配作者右边的括号
                                        ''',re.VERBOSE)
        	self.current_poem = {}

    	def handle_starttag(self, tag, attrs):
        	#找到数据所在的<div>标签
        	if tag == 'div' and _attr(attrs,'class') == 'guwencont2':
            	self.in_div = True

        	#找到<div>标签下的<a>标签
        	if self.in_div and tag == 'a':
            	self.in_a = True
            	self.current_poem['url'] = _attr(attrs,'href')

    	def handle_endtag(self, tag):
        	if tag == 'div':
            	self.in_div = False

        	if tag == 'a':
            	self.in_a = False

    	def handle_data(self, data):
        	#找到查找的数据
        	if self.in_a:
            	print data
            	m = self.pattern.match(data)
            	if m:
                	self.current_poem['title'] = m.group(1)
                	self.current_poem['author'] = m.group(2)
                	self.tangshi_list.append(self.current_poem)
                	self.current_poem = {}

	#解析诗的正文
	class PoemContentParser(HTMLParser):
    	def __init__(self):
        	HTMLParser.__init__(self)
        	self.content = []
        	self.in_p = False

    	def handle_starttag(self, tag, attrs):
        	if tag == 'p' and _attr(attrs,'align') == 'center':
            	self.in_p = True

    	def handle_endtag(self, tag):
        	if tag == 'p':
            	self.in_p = False

    	def handle_data(self, data):
        	if self.in_p:
            	self.content.append(data)

	
	def retrive_tangshi_300():
    	url = 'http://www.gushiwen.org/gushi/tangshi.aspx'
    	r = requests.get(url)
    	p = PoemParser()
    	p.feed(r.content)
    	return p.tangshi_list

	#根据poem的url并把诗的正文爬取出来
	def down_poem(poem):
    	url = poem['url']
    	r = requests.get(url)
    	p = PoemContentParser()
    	p.feed(r.content)
    	poem['content'] = '\n'.join(p.content)

	if __name__ == '__main__':
    	l = retrive_tangshi_300()
    	print 'total %d poems.' % len(l)
    	for i in range(10):
        #print l[i]
        print ('标题:%(title)s\t作者:%(author)s\tURL:%(url)s' % (l[i]))

    	#download all poem
    	for i in range(len(l)):
        	print '%d downloading poem from: %s' % (i,l[i]['url'])
        	down_poem(l[i])
        	print ('标题:%(title)s\t作者:%(author)s\tURL:%(url)s\n%(content)s' % (l[i]))



