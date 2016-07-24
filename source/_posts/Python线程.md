---
title: Python线程
date: {{ date }}
tags: Python
categories:
toc: false
---

<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>


## 介绍

### 金库例子

#### 条件：

- 采购需要进去拿钱买东西
- 销售会把卖东西得到的钱放入金库中
- 进门需要带一把很贵的纯金打造的钥匙

#### 锁
- aquire: 上锁，获得金库钥匙
- release: 解锁，把钥匙放回
- threading.Lock: 钥匙带着进门的人身上，任何人要进去必须等待里面的人出来才可以进入。
- threading.RLock: 钥匙放在部门经理那，同一个部门的人可以一起进来。

#### 信号量

- threading.Semaphore: 多配了几把钥匙，每个钥匙都带着进去的人身上。
- init_value: 有多少把钥匙

#### threading.Condition 

金库里的钱花光了，采购拿到钥匙后，也拿不到钱。这个时候可以使用条件来实现。当销售部门卖完产品拿到钱后，会把钱放进金库，才通知采购去取钱。

- wait,notify,notify_all
- 调用wait前需要获得资源锁。

wait前，先需要acquire获得锁，进去查看有没有钱，如果没有钱，则wait.wait会自动释放锁，以便销售进入仓库放钱。当放入钱后，销售调用notify/notify_all通知采购，可以拿钱了。

#### 事件

- 和条件类似，不同的是不能像条件那样只通知一个人，对于金库的例子，当销售把钱放进仓库时，如果使用事件机制，那么所有排除的人都会知道金库有钱了，它们可以进去取钱。
- 另一个不同点是事件没有锁，只是单纯等待事件的发生。而条件是有锁机制的。


［免费福利1枚］领极客学院30天的VIP，平时30元，现在免费。可以看全站7500节视频课程，想学编程的小伙伴速来。时间有限：http://e.jikexueyuan.com/invite/index.html?ZnJvbV9jb2RlPTZTSDM0SiZ1bmFtZT1DbG91ZFN0eWxlJmNoYW5uZWw9aW52aXRlX3NoYXJlYnV0dG9uX2RpcmVjdDA0



## Usage

	# -*- coding: utf-8 -*-
	import threading
    import time
    import random

    def worker_func():
    	#print threading.current_thread()
    	print('worker thread is started at %s' % (threading.current_thread()))
    	random.seed()
    	time.sleep(random.random())
    	print('worker thread is finished at %s' % (threading.current_thread()))

	def worker_func_lock(lock):
    	lock.acquire()
    	worker_func()
    	lock.release()

	#普通线程例子
	def simple_thread_demo():
    	for i in range(10):
        	th = threading.Thread(target=worker_func)
        	th.start()

	#互斥锁例子
	def thread_demo_lock():
   		for i in range(10):
        	th = threading.Thread(target=worker_func_lock,args=[gLock])
        	th.start()

	#信号量例子
	def thread_demo_sem():
    	for i in range(10):
        	th = threading.Thread(target=worker_func_lock,args=[gSem])
        	th.start()

	#生成者
	class Producer(threading.Thread):
    	def run(self):
        	print('%s is starting' % (threading.current_thread()))
        	while True:
            	global  gMoney
            	global gCondition

            	gCondition.acquire()
            	random.seed
            	p = random.randint(500,1000)
            	gMoney += p
            	print('%s: Produce %s: Left %s' % (threading.currentThread(),p,gMoney))
            	time.sleep(random.random())

            	gCondition.notify_all()
            	gCondition.release()

	#消费者
	class Consumer(threading.Thread):
    	def run(self):
        	print('%s is starting' % (threading.current_thread()))
        	while True:
            	global gMoney
            	global gCondition

            	gCondition.acquire()
            	random.seed()
            	c = random.randint(100, 200)
            	print('%s: Trying to use %s: Left %s' % (threading.currentThread(),c, gMoney))
            	while gMoney < c:
                	gCondition.wait()
            	gMoney -= c
            	print('%s: Consume %s: Left %s' % (threading.currentThread(),c, gMoney))
            	time.sleep(random.random())
            	gCondition.release()

	def consumer_producer_demo():
    	for i in range(10):
        	Consumer().start()

    	for i in range(1):
        	Producer().start()

	#互斥锁
	gLock = threading.Lock()
	#信号量：一次最多执行3个线程
	gSem = threading.Semaphore(3)

	#条件变量：生成者--消费者例子
	gMoney = 1000
	gCondition = threading.Condition()

	if __name__ == '__main__':
    	#simple_thread_demo()
    	#thread_demo_lock()
    	#thread_demo_sem()
    	consumer_producer_demo()