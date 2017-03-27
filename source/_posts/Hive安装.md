---
title: Hive安装
toc: false
date: 2016-08-13 09:05:58
tags: Hadoop
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

因为Hive是构建在Hadoop之上的，所以在安装Hive前，我们需要安装Hadoop环境。
Hadoop的安装可以参考[Hadoop分布式集群安装](http://freeshow.github.io/2016/07/24/Hadoop分布式集群安装/)

本教程使用Hive的本地模式进行安装，本地模式下Hive使用MySQL作为作为元数据库。

# 一、安装MySQL

## 1.安装MySQL

```
sudo apt-get install mysql-server mysql-client
```

## 2.允许MySQL远程连接

默认情况下，MySQL只允许本地登录，所以需要修改 my.cnf 配置。

```
sudo vi /etc/mysql/my.cnf
#bind-address=127.0.0.1
```
注释掉上一句即可在任意位置登录MySQL，然后重启MySQL：

```
sudo service mysql restart
```

## 3.创建MySQL用户和数据库

```
#登录MySQL
mysql -u root -p

#创建hive数据库
create database hive;

#创建MySQL用户hive
grant all on hive.* to hive@'%' identified by 'hive';
grant all on hive.* to hive@'localhost' identified by 'hive'; 
flush privileges;
exit                   #退出mysql

#验证hive用户
mysql -u hive -p hive        
show databases;    
```

# 二、安装Hive

## 1.解压软件包

```
sudo tar -zxvf apache-hive-2.1.0-bin.tar.gz -C /opt/ #解压到/opt目录下
cd /opt
sudo mv apache-hive-2.1.0 hive	#重名名为hive
sudo chown -R hadoop:hadoop hive #修改hive目录拥有者
```

## 2.配置Hive的环境变量

在 `/etc/profile`文件名末尾添加如下内容

```
# set hive
export HIVE_HOME=/opt/hive
export PATH=$PATH:$HIVE_HOME/bin

```

使profile发挥作用：
```
source /etc/profile
```

## 3.修改Hive配置文件

（1）修改hive-env.sh

```
cd /opt/hive/config
cp hive-env.sh.template hive-env.sh
sudo vi hive-env.sh
```
在`hive-env.sh`文件末尾添加变量指向Hadoop的安装路径
```
HADOOP_HOME=/opt/hadoop
```

（2）修改hive-site.xml
```
cp hive-default.xml.template hive-site.xml
sudo vi hive-site.xml
```
修改`hive-site.xml`的主要内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

<property>
    <name>hive.exec.scratchdir</name>
    <value>/tmp/hive</value>
 </property>

<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://master:9000/hive/warehouse</value>
    <description>location of default database for the warehouse</description>
</property>

<property>
    <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://127.0.0.1:3306/hive?createDatebaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
</property>

<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
</property>

<property>
   <name>javax.jdo.option.ConnectionPassword </name>
   <value>hive</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
  <description>Username to use against metastore database</description>
</property>

<property>
    <name>javax.jdo.option.Multithreaded</name>
    <value>true</value>
</property>

</configuration>

```

上面配置的说明：
- hive.exec.scratchdir: 执行Hive操作访问HDFS时用于存储临时数据的目录，默认为/tmp/目录，通常设置为/tmp/hive/,目录权限设置为733.
- hive.metastore.warehouse.dir: 执行Hive数据仓库操作的数据存储目录，设置为HDFS存储路径`hdfs://master_hostname:port/hive/warehouse`。
- javax.jdo.option.ConnectionURL: 设置Hive通过JDBC模式连接MySQL数据库存储metastore内容。

创建上述配置中的目录：
```
hdfs dfs -mkdir /tmp/hive
hdfs dfs -mkdir /hive/warehouse

#分别对刚创建的目录添加组可写权限，允许同组用户进行数据分析操作
hdfs dfs -chmod g+w /tmp
hdfs dfs -chmod g+w /hive/warehouse
```

## 4.添加MySQL JDBC驱动

下载MySQL JDBC驱动，并放在 `$HIVE_HOME/lib`目录下

---

经过上述Hive的基本安装和配置步骤后，在Linux命令提示符下输入hive命令即可进入Hive Shell交互模式环境中进行Hive相关的操作。

```
$ hive
hive>
```

如果执行hive命令出现如下错误：
```
Exception in thread "main" java.lang.RuntimeException: 
Hive metastore database is not initialized. 
Please use schematool (e.g. ./schematool -initSchema -dbType ...) to create the schema. 
If needed, don't forget to include the option to auto-create the underlying database in your JDBC connection string (e.g. ?createDatabaseIfNotExist=true for mysql)
```
可执行如下命令解决：
```
schematool -dbType mysql -initSchema 
```
