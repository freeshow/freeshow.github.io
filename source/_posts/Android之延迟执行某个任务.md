---
title: Android之延迟执行某个任务
date: 2016-07-24 00:10:31
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

本博客转载自[android中延迟执行某个任务](http://blog.csdn.net/qinde025/article/details/6828723)
android中延迟执行某个任务
android App开发在某些情况下需要有延时功能，比如说App首页显示定格3秒，然后自动跳到登录页的情况，这就好比是一个预加载，但是这个预加载可能瞬间就完成了，撑不到3秒钟，这是就要求你做延时处理。

下面是三种方法：
一、线程

```
new Thread(new Runnable()
{
	public void run()
	{    
    	Thread.sleep(XXXX);    
    	handler.sendMessage();----告诉主线程执行任务    
     }    
}).start
```

二、延时器

```
TimerTask task = new TimerTask()
{    
    public void run()
	{    
    	//execute the task     
    }    
};    
Timer timer = new Timer();  
timer.schedule(task, delay);
```

三、android消息处理

```
new Handler().postDelayed(new Runnable(){    
    public void run() 
	{    
    	//execute the task    
    }    
 }, delay);
```
推荐使用第三种