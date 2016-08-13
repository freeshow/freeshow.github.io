---
title: Hive应用实例：WordCount
toc: false
date: 2016-08-13 20:21:47
tags: Hadoop
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

出自《大数据原理与应用》一书。

# 词频统计任务要求：
首先，需要创建一个需要分析的输入数据文件
然后，编写HiveQL语句实现WordCount算法

#具体步骤如下：
## （1）创建input目录，其中input为输入目录。命令如下：
```
$ cd /home/hadoop
$ mkdir input
```
##（2）在input文件夹中创建两个测试文件file1.txt和file2.txt，命令如下：
```
$ cd  /home/hadoop/input
$ echo "hello world" > file1.txt
$ echo "hello hadoop" > file2.txt
```

##（3）进入hive命令行界面，编写HiveQL语句实现WordCount算法，命令如下：
```
$ hive
hive> create table docs(line string);
hive> load data inpath 'input' overwrite into table docs;
hive>create table word_count as 
     select word, count(1) as count from
     (select explode(split(line,' '))as word from docs) w
     group by word
     order by word;
```

<center>![](http://i.imgur.com/nwNRG38.png)</center>
<center>![](http://i.imgur.com/gup2ShW.png)</center>

**执行完成后，用select语句查看运行结果如下：**
```
hive> select * from word_count;
OK
hadoop  1
hello   2
world   1
Time taken: 0.111 seconds, Fetched: 3 row(s)
hive> 

```