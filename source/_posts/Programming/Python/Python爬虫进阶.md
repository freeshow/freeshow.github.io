---
title: Python爬虫进阶
date: 2016-07-24 16:31:35
tags: 
- Crawler
- Python
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Python爬虫架构选择

### HTML解析器：
HTMLParser,BeautifulSoup4,XPath的lxml.
选择：XPath > BeautifulSoup4 > HTMLParser

### HTTP请求：
urllib,urllib2,requests
选择：requsets >> urllib2,urllib

### 爬虫框架：
Scrapy


## Python爬虫进阶内容

- Scrapy爬虫框架
- beautifulsoup解析器
- Selector/XPath -> Scrapy
- 并发
	- twisted
	- gevent
- 分布式爬虫
	- 任务队列：[https://github.com/nvie/rq](https://github.com/nvie/rq)
	- 任务队列与存储结合：[https://github.com/rolando/scrapy-redis](https://github.com/rolando/scrapy-redis)
- 数据处理：[https://www.github.com/grangier/python-goose](https://www.github.com/grangier/python-goose)


## 不知道用爬虫来做什么？

知乎搜索一下：何明科

[https://www.zhihu.com/people/he-ming-ke](https://www.zhihu.com/people/he-ming-ke)

刷一下他的高票回答，你就可以知道原来用爬虫可以做这么酷的事情，顺便还把钱赚了。