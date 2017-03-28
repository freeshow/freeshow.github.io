---
title: Kafka集群安装
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

# 一、安装ZooKeeper集群

可以参考我的博客： [ZooKeeper集群安装](http://freeshow.github.io/2016/07/26/ZooKeeper集群安装/)

# 二、 安装Kafka集群

## 1. 解压

在master节点上：

去Apache Kafka官网下载压缩包，我下载的是 `kafka_2.11-0.10.0.0.tgz`

解压到`/opt`目录下，并重命名为kafka
```
sudo tar -zxvf kafka_2.11-0.10.0.0.tgz -C /opt
cd /opt
sudo mv kafka_2.11-0.10.0.0 kafka
```

## 2.修改配置文件

修改`server.properties`配置文件：

```
broker.id=1		# 其他节点分别为broker.id=2和broker.id=3
zookeeper.connect=master:2181,slave1:2181,slave2:2181    #设置各zookeeper地址
```

将master节点上`/opt/kafka`发送到其他节点上去，并修改配置文件`server.properties`中broker.id的值。

# 三、操作Kafka集群

## 1.在各节点上启动ZooKeeper
```
zkServer.sh start
```

## 2.在各节点上启动Kafka：
```
bin/kafka-server-start.sh config/server.properties
```

## 3.在kafka集群中创建一个topic

以master节点为例，在其他节点上也可以：
```
bin/kafka-topics.sh --create --zookeeper master:2181 --replication-factor 3 --partitions 1 --topic test		#创建topic: test
```

## 4.用一个producer向某一个topic中写入消息
```
bin/kafka-console-producer.sh --broker-list master:9092 --topic test
```

## 5.用一个comsumer从某一个topic中读取信息
```
bin/kafka-console-consumer.sh --zookeeper master:2181 --from-beginning --topic test
```

此次，如果producer输入消息，则comsumer就是收到消息。

当然，这只是命令行的形式，实际开发中一般用 Java API编写。


## 6.查看一个topic的分区及副本状态信息

```
bin/kafka-topics.sh --describe --zookeeper master:2181 --topic test
```