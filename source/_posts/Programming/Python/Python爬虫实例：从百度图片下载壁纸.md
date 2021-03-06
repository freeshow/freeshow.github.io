---
title: Python爬虫实例：从百度图片下载壁纸
date: 2016-07-24 20:49:06
tags: Crawler
categories: 编程语言

---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 一、数据分析
百度图片壁纸网址：[http://image.baidu.com/channel/wallpaper](http://image.baidu.com/channel/wallpaper)

### 1.打开网址，点击`国家地理`,打开Chrom浏览器的开发者工具，选中图片图片元素。
<center>![](http://i.imgur.com/8oPqY6E.png)</center>

获得第一张图片的url为:
`http://b.hiphotos.baidu.com/image/w%3D400/sign=937884d0d5ca7bcb7d7bc62f8e086b3f/64380cd7912397dd403225175b82b2b7d0a2875e.jpg`

### 2.右键-->查看网页源代码，在网页源代码中查找第一张图片的url
<center>![](http://i.imgur.com/Pv4tcEz.png)</center>

发现在网页源代码中没有找到第一张图片的url，说明图片的加载是通过Ajax实现的。即网页运行时加载的。故需要通过开发者工具的`Network`查找。

### 3.打开开发者工具的`Network`,抓取网络请求。
找到一条网络请求的`Preview`，如下图：
<center>![](http://i.imgur.com/8Nof5qO.png)</center>

里面的downloadUrl即为所要下载图片的url.**即找到了请求图片的接口。**

查看其`Headers`,如下图：
<center>![](http://i.imgur.com/bAcfM1A.png)</center>

**模仿其请求的url,params，返回的为json数据，里面包含图片的url.**

## 二、爬虫实现

	# -*- coding: utf-8 -*-
	import requests
	import urllib
	import os

	#下载图片
	def _download_image(url,floder='image'):
    	print('downloading %s' % url)
    	if not os.path.isdir(floder):
       	 os.mkdir(floder)

    	#若url为http://b.hiphotos.baidu.com/image/pic/item/cefc1e178a82b90133597e20718da9773912ef29.jpg
    	#返回为：cefc1e178a82b90133597e20718da9773912ef29.jpg
    	def _fname(s):
        	return os.path.join(floder,os.path.split(url)[1])

    	fname = _fname(url)
    	#下载图片
    	urllib.urlretrieve(url,fname)

	#获取所有图片的url,并下载图片。
	def download_wallpaper():
    	url = 'http://image.baidu.com/data/imgs'
    	params ={
        	'pn': 0,
        	'rn': 18,
        	'col': '壁纸',
        	'tag': '国家地理',
        	'tag3': '',
        	'width':1440,
        	'height':900,
        	'ic':0,
        	'ie':'utf8',
        	'oe':'utf-8',
        	'image_id': '',
        	'fr':'channel',
        	'p':'channel',
        	'from':1,
        	'app':'img.browse.channel.wallpaper',
        	't':0.884265049666543
    	}
    	r = requests.get(url,params=params)
    	#print(r.json())
    	imgs = r.json()['imgs']
    	print('totally %d images.' % len(imgs))
    	for img in imgs:
        	if 'downloadUrl' in img:
            	_download_image(img['downloadUrl'])

	if __name__ == '__main__':
    	download_wallpaper()

## 多线程爬虫

	# -*- coding: utf-8 -*-
	import requests
	import urllib
	import os
	import threading

	#所有下载图片的url
	gImageList = []
	gCondition = threading.Condition()

	#生产者从网站上获取图片url放进gImageList变量中
	class Producer(threading.Thread):
    	def run(self):
        	global gImageList
        	global gCondition

        	print('%s: started.' % threading.currentThread())

        	imgs = download_wallpaper_list()
        	gCondition.acquire()
        	for img in imgs:
            	if 'downloadUrl' in img:
                	gImageList.append(img['downloadUrl'])
                	print('%s: produce finished. Left: %s' % (threading.currentThread(),len(gImageList)))

        	gCondition.notify_all()
        	gCondition.release()

	#消费者从下载图片Url列表中下载图片。
	class Consumer(threading.Thread):
    	def run(self):
        	print('%s: started.' % threading.currentThread())
        	while True:
            	global gImageList
            	global gCondition

            	gCondition.acquire()
            	print('%s: trying to download from pool,pool size is %s' % \
                  (threading.currentThread(),len(gImageList)))
            	while len(gImageList) == 0:
                	gCondition.wait()
            	print('%s: waken up. pool size is %d.' % \
                      (threading.currentThread(),len(gImageList)))
            	url = gImageList.pop()
            	gCondition.release()

            	_download_image(url)


	def _download_image(url,floder='image'):
    	print('downloading %s' % url)
    	if not os.path.isdir(floder):
        	os.mkdir(floder)

    	#若url为http://b.hiphotos.baidu.com/image/pic/item/cefc1e178a82b90133597e20718da9773912ef29.jpg
    	#返回为：cefc1e178a82b90133597e20718da9773912ef29.jpg
    	def _fname(s):
        	return os.path.join(floder,os.path.split(url)[1])

    	fname = _fname(url)
    	#下载图片
    	urllib.urlretrieve(url,fname)

	#获取图片的url
	def download_wallpaper_list():
    	url = 'http://image.baidu.com/data/imgs'
    	params ={
        	'pn': 0,
        	'rn': 18,
        	'col': '壁纸',
        	'tag': '国家地理',
        	'tag3': '',
        	'width':1440,
        	'height':900,
        	'ic':0,
        	'ie':'utf8',
        	'oe':'utf-8',
        	'image_id': '',
        	'fr':'channel',
        	'p':'channel',
        	'from':1,
        	'app':'img.browse.channel.wallpaper',
        	't':0.884265049666543
    	}
    	r = requests.get(url,params=params)
    	#print(r.json())
    	imgs = r.json()['imgs']
    	print('%s: totally %d images.' % (threading.currentThread(),len(imgs)))
   		return imgs

	if __name__ == '__main__':
    	#1个生产者，即一个线程获取下载图片的url列表。
    	Producer().start()

    	#5个消费者：即5个线程下载图片。
    	for i in range(5):
        	Consumer().start()

## 多线程爬虫的问题

- python多线程的限制条件：python解析器的全局锁导致即使在多核CUP中，一个时间片内也只能有一个python程序在执行。
- 多线程只适合有IO等待的场景。如果是纯计算的场景，多线程无法优化性能。
- 使用多进程 multiprocessing
- 使用分布式
- 关于并发，有并发控制。关注twisted/gevent等基于事件的框架。