---
title: XPath与多线程爬虫
date: {{ date }}
tags: Crawler
categories:
toc: false
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

# 一、神器XPath的介绍和配置

## 1.XPath是什么

- XPath 是一门语言
- XPath可以在XML文档中查找信息
- XPath支持HTML
- XPath通过元素和属性进行导航
- XPath可以用来提取信息
- XPath比正则表达式厉害
- XPath比正则表达式简单

## 2.如何安装使用XPath

- 安装lxml库
- from lxml import etree
- Selector = etree.HTML(网页源代码)
- Selector.xpath(一段神奇的符号)


# 二、神器XPath的使用

## 1.XPath与HTML结构

- 树状结构
- 逐层展开
- 逐层定位
- 寻找独立节点


## 2.获取网页元素的Xpath

- 手动分析法
- Chrome生成法


## 3.应用XPath提取内容

- //定位根节点
- /往下层寻找
- 提取文本内容：/text()
- 提取属性内容: /@xxxx

## 4.应用实例

```python
# -*- coding: utf-8 -*-

from lxml import etree

html = '''
<!DOCTYPE html>
<html>
	<head lang="en">
    	<meta charset="UTF-8">
    	<title>测试-常规用法</title>
	</head>
	<body>
		<div id="content">
    		<ul id="useful">
        		<li>这是第一条信息</li>
        		<li>这是第二条信息</li>
        		<li>这是第三条信息</li>
    		</ul>
    		<ul id="useless">
        		<li>不需要的信息1</li>
        		<li>不需要的信息2</li>
        		<li>不需要的信息3</li>
    		</ul>

    		<div id="url">
        		<a href="http://jikexueyuan.com">极客学院</a>
        		<a href="http://jikexueyuan.com/course/" title="极客学院课程库">点我打开课程库</a>
    		</div>
		</div>

	</body>
</html>
'''

selector = etree.HTML(html)

#selector.xpath()返回列表。

#提取文本
content = selector.xpath('//ul[@id="useful"]/li/text()')
for each in content:
    print each
    
# 这是第一条信息
# 这是第二条信息
# 这是第三条信息

#提取属性
link = selector.xpath('//a/@href')
for each in link:
    print each
    
# http://jikexueyuan.com
# http://jikexueyuan.com/course/

title = selector.xpath('//a/@title')
print title[0]

#极客学院课程库
```





# 三、神器XPath的特殊用法

## 1.以相同的字符开头

- starts-with(@属性名称, 属性字符相同部分)
	
```html	
<div id="test-1">需要的内容1</div>
<div id="test-2">需要的内容2</div>
<div id="testfault">需要的内容3</div>
```	

- 应用实例


```python

# -*- coding: utf-8 -*-

from lxml import etree

html1 = '''
<!DOCTYPE html>
<html>
    <head lang="en">
        <meta charset="UTF-8">
        <title></title>
    </head>
    <body>
        <div id="test-1">需要的内容1</div>
        <div id="test-2">需要的内容2</div>
        <div id="testfault">需要的内容3</div>
    </body>
</html>
'''
selector = etree.HTML(html1)

content = selector.xpath('//div[starts-with(@id,"test")]/text()')

for each in content:
    print each

# 需要的内容1
# 需要的内容2
# 需要的内容3
```

## 2.标签套标签

- string(.)

```html
<div id=“class3”>美女，
	<font color=red>你的微信是多少？</font>
</div>
```

- 应用实例

```python
# -*- coding: utf-8 -*-

from lxml import etree

html = '''
<!DOCTYPE html>
<html>
    <head lang="en">
        <meta charset="UTF-8">
        <title></title>
    </head>
    <body>
        <div id="test3">
            我左青龙，
            <span id="tiger">
                右白虎，
                <ul>上朱雀，
                    <li>下玄武。</li>
                </ul>
                老牛在当中，
            </span>
            龙头在胸口。
        </div>
    </body>
</html>
'''
selector = etree.HTML(html)

content_1 = selector.xpath('//div[@id="test3"]/text()')
for each in content_1:
    print each

#显示效果
#    我左青龙，


#    龙头在胸口。

data = selector.xpath('//div[@id="test3"]')[0]
info = data.xpath('string(.)')
content_2 = info.replace('\n','').replace(' ','')
print content_2

#显示效果
#我左青龙，右白虎，上朱雀，下玄武。老牛在当中，龙头在胸口。
```


# 四、Python并行化介绍与演示

## 1.Python并行化介绍

- 多个线程同时处理任务
- 高效
- 快速


## 2.Map的使用

- map 函数一手包办了序列操作、参数传递和结果保存等一系列的操作。
- from multiprocessing.dummy import Pool
- pool = Pool(4)
- results = pool.map(爬取函数, 网址列表)

## 3.使用实例

```python
# -*- coding: utf-8 -*-

from multiprocessing.dummy import Pool as ThreadPool
import requests
import time

def getsource(url):
    html = requests.get(url)

urls = []

for i in range(1,21):
    newpage = 'http://tieba.baidu.com/p/3522395718?pn=' + str(i)
    urls.append(newpage)

time1 = time.time()
for i in urls:
    print i
    getsource(i)
time2 = time.time()
print u'单线程耗时：' + str(time2-time1)

pool = ThreadPool(4)
time3 = time.time()
results = pool.map(getsource, urls)
pool.close()
pool.join()
time4 = time.time()
print u'并行耗时：' + str(time4-time3)
```
运行效果：
```
http://tieba.baidu.com/p/3522395718?pn=1
http://tieba.baidu.com/p/3522395718?pn=2
http://tieba.baidu.com/p/3522395718?pn=3
http://tieba.baidu.com/p/3522395718?pn=4
http://tieba.baidu.com/p/3522395718?pn=5
http://tieba.baidu.com/p/3522395718?pn=6
http://tieba.baidu.com/p/3522395718?pn=7
http://tieba.baidu.com/p/3522395718?pn=8
http://tieba.baidu.com/p/3522395718?pn=9
http://tieba.baidu.com/p/3522395718?pn=10
http://tieba.baidu.com/p/3522395718?pn=11
http://tieba.baidu.com/p/3522395718?pn=12
http://tieba.baidu.com/p/3522395718?pn=13
http://tieba.baidu.com/p/3522395718?pn=14
http://tieba.baidu.com/p/3522395718?pn=15
http://tieba.baidu.com/p/3522395718?pn=16
http://tieba.baidu.com/p/3522395718?pn=17
http://tieba.baidu.com/p/3522395718?pn=18
http://tieba.baidu.com/p/3522395718?pn=19
http://tieba.baidu.com/p/3522395718?pn=20
单线程耗时：12.4940001965
并行耗时：3.63199996948

```


# 五、实战--百度贴吧爬虫

- 目标网站：http://tieba.baidu.com/p/3522395718
- 目标内容：跟帖用户名，跟帖内容，跟帖时间
- 涉及知识：	
	- Requests获取网页
	- XPath提取内容
	- map实现多线程爬虫


## 应用实例

```python
# -*- coding: utf-8 -*-

from lxml import etree
from multiprocessing.dummy import Pool as ThreadPool
import requests
import json
import sys

reload(sys)
sys.setdefaultencoding('utf-8')

def towrite(contentdict):
    f.write(u'回帖时间:' + str(contentdict['topic_reply_time']) + '\n')
    f.write(u'回帖内容:' + unicode(contentdict['topic_reply_content']) + '\n')
    f.write(u'回帖人:' + contentdict['user_name'] + '\n\n')


def spider(url):
    html = requests.get(url)
    selector = etree.HTML(html.text)
    content_field = selector.xpath('//*[@id="j_p_postlist"]/div')
    item = {}
    for each in content_field:
        reply_info = json.loads(each.xpath('@data-field')[0].replace('&quot', ''))
        author = reply_info['author']['user_name']
        reply_time = reply_info['content']['date']

        content = each.xpath('div[@class="d_post_content_main"]/div/cc/div/text()')[0]

        print content
        print reply_time
        print author
        item['user_name'] = author
        item['topic_reply_content'] = content
        item['topic_reply_time'] = reply_time
        towrite(item)


if __name__ == '__main__':
    pool = ThreadPool(4)
    f = open('content.txt', 'wb')
    page = []
    for i in range(1, 21):
        newpage = 'http://tieba.baidu.com/p/3522395718?pn=' + str(i)
        page.append(newpage)
        print newpage

    results = pool.map(spider, page)
    pool.close()
    pool.join()
    f.close()
```

效果如下：

```
回帖时间:2015-01-11 20:49
回帖内容:            大神在不？我这是啥情况啊
回帖人:闫烁楠

回帖时间:2015-01-11 20:49
回帖内容:            我朋友可以拿国行5s黑色16g 2880 能想信么
回帖人:超级恶魔的左手

回帖时间:2015-01-11 20:51
回帖内容:            桌面图标随意摆放 必须要越狱吗？
回帖人:不足五分钟

回帖时间:2015-01-11 20:52
回帖内容:            这个是真的吗
回帖人:莪嫒迩德莼
......
```

