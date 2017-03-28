---
title: Python开发简单爬虫之实战演练
date: 2016-07-24 16:31:35
tags: 
- Crawler
- Python
categories:
- 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

本博客来自慕课网---[Python开发简单爬虫](http://www.imooc.com/view/563)

接上一篇博客：[Python开发简单爬虫之爬虫介绍](https://freeshow.github.io/2016/07/24/Python%E5%BC%80%E5%8F%91%E7%AE%80%E5%8D%95%E7%88%AC%E8%99%AB%E4%B9%8B%E7%88%AC%E8%99%AB%E4%BB%8B%E7%BB%8D%EF%BC%88%E4%B8%80%EF%BC%89/)

# 一、爬虫实例-分析目标

<center>![](http://i.imgur.com/YpnnJ8O.jpg)</center>

<center>![](http://i.imgur.com/TwsuxIR.jpg)</center>

## 项目结构：

<center>![](http://i.imgur.com/nMd2DG9.png)</center>

# 二、调度程序

<center>![](http://i.imgur.com/fjADhzH.jpg)</center>

## spider_main.py

	# -*- coding: utf-8 -*-

	# 爬虫总调度程序，会以一个入口的url作为参数，爬取相关页面
	from baike_spider import url_manager
	from baike_spider import html_downloader
	from baike_spider import html_parser
	from baike_spider import html_outputer


	class SpiderMain(object):
    	def __init__(self):
        	self.urls = url_manager.UrlManager()
        	self.downloader = html_downloader.HtmlDownloader()
        	self.parser = html_parser.HtmlParser()
        	self.outputer = html_outputer.OutPuter()

    	def craw(self, root_url):
        	count = 1 #爬取的url个数
        	self.urls.add_new_url(root_url)
        
       		while self.urls.has_new_url():
            	try:
                	new_url = self.urls.get_new_url()
                	print 'craw %s: %s' %(count,new_url)
                	html_cont = self.downloader.download(new_url)
                	new_urls,new_data = self.parser.parse(new_url,html_cont)
                	self.urls.add_new_urls(new_urls)
                	self.outputer.collect_data(new_data)
                	count = count + 1
           		except:
                	print 'craw failed'
                
        	#将结果显示在html页面中
        	self.outputer.output_html()

	if __name__ == '__main__':
    	# 入口url
    	root_url = 'http://baike.baidu.com/view/21087.htm'
    	obj_spider = SpiderMain()
    	# 爬虫开始
    	obj_spider.craw(root_url)



# 三、URL管理器

## url_manager.py

	# -*- coding: utf-8 -*-
	class UrlManager(object):
    	def __init__(self):
        	self.new_urls = set()
        	self.old_urls = set()

    	def add_new_url(self, url):
        	if url is None:
            	return
        	if url not in self.new_urls and url not in self.old_urls:
            	self.new_urls.add(url)

    	def add_new_urls(self, urls):
        	if urls is None or len(urls) == 0:
            	return
        	for url in urls:
            	self.add_new_url(url)

    	def get_new_url(self):
        	new_url = self.new_urls.pop()
        	self.old_urls.add(new_url)
        	return new_url

    	def has_new_url(self):
        	return len(self.new_urls) != 0


# 四、HTML下载器html_downloader

## html_downloader.py

	# -*- coding: utf-8 -*-
	import urllib2

	class HtmlDownloader(object):
    	def download(self, url):
        	if url is None:
            	return
        
        	response = urllib2.urlopen(url)

        	if response.getcode() != 200:
            	return None

        	return response.read()

# 五、HTML解析器html_parser

## html_parser.py

	# -*- coding: utf-8 -*-
	from bs4 import BeautifulSoup
	import re
	import urlparse

	class HtmlParser(object):
    	def parse(self, page_url,html_cont):
        	if page_url is None or html_cont is None:
            	return

        	soup = BeautifulSoup(html_cont,'html.parser',from_encoding='utf-8')
        	new_urls = self._get_new_urls(page_url,soup)
        	new_data = self._get_new_data(page_url,soup)

        	return new_urls,new_data

    	def _get_new_urls(self, page_url, soup):
       		new_urls = set()
        	#/view/123.htm
        	links = soup.find_all('a',href = re.compile(r'/view/\d+\.htm'))
        	for link in links:
            	#page_url:http://baike.baidu.com/view/21087.htm
            	#new_url: /view/10812319.htm
            	#new_full_url: http://baike.baidu.com/view/10812319.htm
            	new_url = link['href']
            	new_full_url = urlparse.urljoin(page_url,new_url)
            	new_urls.add(new_full_url)
        	return new_urls

    	#获取标题和简介
    	def _get_new_data(self, page_url, soup):
        	res_data = {}

        	#url
        	res_data['url'] = page_url

        	#获取词条标题
        	#<dd class="lemmaWgt-lemmaTitle-title">
        	#    <h1>Python</h1>
        	title_node = soup.find('dd',class_='lemmaWgt-lemmaTitle-title').find('h1')
        	res_data['title'] = title_node.get_text()

        	#获取词条简介
        	#<div class="lemma-summary" label-module="lemmaSummary">
        	summary_node = soup.find('div',class_='lemma-summary')
        	res_data['summary'] = summary_node.get_text()

        	return res_data

# 六、HTML输出器

## html_outputer.py

	# -*- coding: utf-8 -*-
	class OutPuter(object):
    	def __init__(self):
        	self.datas = []

    	def collect_data(self, data):
        	if data is None:
            	return
        	#print data
        	self.datas.append(data)

    	def output_html(self):
        	with open('output.html','w') as fout:
            	fout.write('<html>')
            	fout.write('<head>')
            	#指定网页的编码格式
            	fout.write('<meta charset="UTF-8">')
            	fout.write('</head>')
            	fout.write('<body>')
            	fout.write('<table>')

            	#python的默认编码是ascii,需要转码为utf-8
            	for data in self.datas:
                	fout.write('<tr>')
                	fout.write('<td>%s</td>' % data['url'])
                	fout.write('<td>%s</td>' % data['title'].encode('UTF-8'))
                	fout.write('<td>%s</td>' % data['summary'].encode('UTF-8'))
                	fout.write('</tr>')

            	fout.write('</table>')
            	fout.write('</body>')
            	fout.write('</html>')



# 七、开始运行爬虫和爬取结果展示 

## 运行spider_main.py

## 效果如下：

<center>![](http://i.imgur.com/bHEyJqH.png)</center>





