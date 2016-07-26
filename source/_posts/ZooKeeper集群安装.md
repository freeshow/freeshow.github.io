---
title: ZooKeeper集群安装
toc: false
date: 2016-07-26 18:35:17
tags: Hadoop 
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

环境基于上篇博客的搭建环境： [Hadoop分布式集群安装](http://freeshow.github.io/2016/07/24/Hadoop分布式集群安装/)

有三台虚拟机：master,slave1,slave2

# 一、安装步骤

1.下载ZooKeeper: 去 Apache ZooKeeper官网下载，我下载的为 `zookeeper-3.4.8.tar.gz`.

2.解压：

```
sudo tar -zxf zookeeper-3.4.8.tar.gz -C /opt 	#解压到/opt目录下
cd /opt
sudo mv zookeeper-3.4.8 zookeeper		#重命名为zookeeper
```
3.创建ZooKeeper的data目录

为了便于管理我创建了ZooKeeper安装目录下
```
mkdir /opt/zookeeper/data
```
4.创建myid文件

在`/opt/zookeeper/data`文件夹下创建文件`myid`,并输入内容 1。其余节点分别为2和3。

```
cd /opt/zookeeper/data
vi myid 	#输入1,其他节点分别输入2和3
```
5.配置文件

在conf目录下创建一个配置文件zoo.cfg:
```
cp zoo_sample.cfg zoo.cfg
sudo vi zoo.cfg
```

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/opt/zookeeper/data/
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

# 二、最低配置要求中必须配置的参数
1. clent: 监听客户端连接的端口
2. tickTime: 基本时间单元，这个时间作为ZooKeeper服务器之间或客户端与服务器之间维持心跳的时间间隔。
3. dataDir: 存储内存中数据库快照的位置。

# 三、集群配置

1. initLimit: 此配置表示，允许follower(相对于Leader而言的“客户端”)连接并同步到Leader的初始化连接时间。
2. syncLimit: 此配置项表示Leader和Follower之间发送消息时，请求和应答得时间长度。
3. server.A=B: C: D。 其中A是一个数字，表示这个是服务器的编号(myid文件中的数字)；B是这个服务器的IP地址；C是Leader选举的端口；D是ZooKeeper服务器之间的通信端口。
4. myid和zoo.cfg。除了zoo.cfg配置文件外，集群模式下还要配置一个文件myid,这个文件在dataDir目录下，这个文件里就有一个数据就是A的值，ZooKeeper启动时会读取这个文件，拿到里面的数据和zoo.cfg里面的配置信息做比较，从而判断是哪个server.

# 四、启动ZooKeeper

将 `/opt/zookeeper/bin`加入到Path文件路径中。

配置好之后，可以通过下面命令对ZooKeeper进行操作。
1. 在3个节点上分别执行命令 zkServer.sh start 启动 ZooKeeper。
2. 在3个节点上分别执行命令 zkServer.sh status 检查节点状态。
3. 在3个节点上分别执行命令 zkServer.sh stop 关闭 ZooKeeper。

