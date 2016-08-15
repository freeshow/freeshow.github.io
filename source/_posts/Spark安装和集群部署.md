---
title: Spark安装和集群部署
toc: false
date: 2016-08-15 11:47:41
tags: Spark
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

# 一、搭建Hadoop分布式集群

参考 [Hadoop分布式集群安装](http://freeshow.github.io/2016/07/24/Hadoop分布式集群安装/) 进行搭建

# 二、Spark安装和集群部署

## 1.安装Scala

Spark对配套的Scala版本有规定，所以要根据自己的实际情况来选择Scala的版本。

如下图所示：
<center>![](http://i.imgur.com/YYVi9s9.png)</center>

由于Hadoop我们安装的是2.6.4，故我们选择上图中与Hadoop配套的Spark，因而选择Scala的版本为2.11。

我下载的Scala为`scala-2.11.8.tgz`

(1)解压并放到相应的目录

```
tar -zxvf scala-2.11.8.tgz -C /opt/		#解压到/opt/目录下
cd /opt/
mv scala-2.11.8.tgz scala	#重名为scala
```

(2)配置环境变量

```
sudo vi /etc/profile
```
在文件最后添加如下内容：

```
# set scala env
export SCALA_HOME=/opt/scala
export PATH=$PATH:$SCALA_HOME/bin
```

(3)在终端输入`scala`，进入Scala的命令交互式界面，则安装成功。


注意：
>由于Spark需要运行在三台机器上，另外两台同样需要安装Scala。

## 2.安装Spark和集群部署

Spark需要运行在三台机器上，这里先安装 Spark 到 master 这台机器上，另外两台的安装方法一致，也可以使用SSH的`scp`命令把master机器上安装好的Spark目录复制到另外两台机器相同目录下。

(1)下载并解压

从Spark官网下载Spark安装包，我下载的是`spark-2.0.0-bin-hadoop2.6.tgz`。下载完后解压，并存放到自己指定的存储目录下：

```
sudo tar -zxvf spark-2.0.0-bin-hadoop2.6.tgz -C /opt
cd /opt
mv spark-2.0.0-bin-hadoop2.6 spark
```

(2)配置环境变量

```
sudo vi /etc/profile
```

在文件末尾添加：
```
# set spark env
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin
```
并使配置文件生效：
```
source /etc/profile
```

(3)配置Spark,需要配置`spark-env.sh`和`slaves`文件。

配置`spark-env.sh`文件：
```
cd /opt/spark/conf
cp spark-defaults.conf.template spark-env.sh
vi spark-env.sh
```
配置内容如下：

```
export JAVA_HOME=/opt/java
export SCALA_HOME=/opt/scala
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
export SPARK_MASTER_IP=master
```
>SPARK_MASTER_IP: Spark集群的Master节点的IP地址。


配置slaves文件：

```
cp slaves.template slaves
vi slaves
```
把Worker节点的主机名都添加进去，修改后的内容为：
```
slave1
slave2
```

(4) 按照上述配置，将Spark安装到另外两台机器上(slave1、slave2)

(5) 启动并测试集群的情况

 1)当前我们只使用Hadoop的HDFS文件系统，所以可以只启动Hadoop的HDFS文件系统。
```
start-dfs.sh
```
 2)用Spark的sbin目录下的`start-all.sh`命令启动Spark集群，这里需要注意的是，在命令终端必须写成`./start-all.sh`，因为在Hadoop的sbin目录下也有一个`start-all.sh`可执行文件。

 3)此时使用JPS在master节点、slave1节点和slave2节点分别可以查看到新开启的Master和Worker进程。

```
hadoop@master:~$ jps
3570 NameNode
3908 Master
3978 Jps
3790 SecondaryNameNode
```

```
hadoop@slave1:~$ jps
1826 Worker
1939 Jps
1689 DataNode
```

```
hadoop@slave2:~$ jps
1826 Worker
1939 Jps
1689 DataNode
```

 4)可以进入Spark的WebUI页面，访问`master:8080`,如下如所示(8080为Spark的WebUI监听端口，7077为Spark集群的Master内部监听端口)。

<center>![](http://i.imgur.com/NzIiwOC.png)</center>

 5)进入Spark的bin目录，使用`spark-shell`命令可以进入spark - shell控制台：

```
hadoop@master:~$ spark-shell 
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel).
16/08/15 13:18:38 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
16/08/15 13:18:40 WARN spark.SparkContext: Use an existing SparkContext, some configuration may not take effect.
Spark context Web UI available at http://192.168.1.104:4040
Spark context available as 'sc' (master = local[*], app id = local-1471238319919).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.0.0
      /_/
         
Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_91)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 

```

我们也可以在WebUI页面输入`http://master:4040`从Web的角度了解Spark-Shell。


这时，Spark集群部署成功。

