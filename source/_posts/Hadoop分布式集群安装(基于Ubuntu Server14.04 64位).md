---
title: {{ title }}
date: {{ date }}
tags: Hadoop 
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载声明：
本教程转载自：厦门大学数据库实验室 blog
[Hadoop安装教程_单机/伪分布式配置_Hadoop2.6.0/Ubuntu14.04](http://dblab.xmu.edu.cn/blog/install-hadoop/)
[Hadoop集群安装配置教程_Hadoop2.6.0_Ubuntu/CentOS](http://dblab.xmu.edu.cn/blog/install-hadoop-cluster/)

# 一、环境搭建

## 1.使用VirtualBox创建三台Ubuntu Server 14.04(64 位)虚拟机
首先，使用VirtualBox安装3台Ubuntu Server，主机名分别为master,slave1,slave2，各用户名均为hadoop. 
如图所示：

![](http://i.imgur.com/T4lUiC7.png)

![](http://i.imgur.com/RLAaMQc.png)

## 2.配置网络

使用虚拟机安装的系统，需要更改网络连接方式为桥接（Bridge）模式，才能实现多个节点互连。
例如在VirturalBox中的设置如下图。此外，如果节点的系统是在虚拟机中直接复制的，要确保各个节点的Mac地址不同（可以点右边的按钮随机生成MAC地址，否则IP会冲突）：

![](http://i.imgur.com/9bXwfCr.png)

## 3.添加主机名与IP的对应关系

### (1)各节点与IP对应关系：

| 用户名  | 主机名 | 		 IP       |
| :----: |:------:| :------------:|
| hadoop | master | 192.168.1.104 |
| hadoop | slave1 | 192.168.1.105 |
| hadoop | slave2 | 192.168.1.106 |

### (2)添加对应关系
在 `/etc/hosts` 中将该映射关系填写上去即可，如下图所示：
>一般该文件中只有一个 127.0.0.1，其对应名为 localhost，如果有多余的应删除，特别是不能有 “127.0.0.1 master” 这样的记录

```
sudo vi /etc/hosts
```
修改如下图：
![](http://i.imgur.com/ENKuoXG.png)

>需要在所有节点上完成上述配置，
>如上面讲的是master节点的配置，而在其他的slave节点上，也要对`/etc/hosts`（跟 master 的配置一样）文件进行修改！

### (3)测试各节点的连通性
配置好后需要在各个节点上执行如下命令，测试是否相互 ping 得通，如果 ping 不通，后面就无法顺利配置成功：
以master节点为例：
```
ping slave1 -c 3		# 只ping 3次，否则要按 Ctrl+c 中断
ping slave2 -c 3
```

如下图所示：
![](http://i.imgur.com/AUPaYbF.png)

# 二、安装SSH、配置SSH无密码登陆

**这个操作是要让 master 节点可以无密码 SSH 登陆到各个 slave 节点上。**

## 1. 安装SSH
集群、单节点模式都需要用到 SSH 登陆（类似于远程登陆，你可以登录某台 Linux 主机，并且在上面运行命令）。
因此，需要安装 SSH client 和 SSH server：

```
sudo apt-get install openssh-server openssh-client
```

>安装完成后，就可以在Windows下使用SSH工具登录，上面三台虚拟机了。

## 2. master免密码登录本机
安装完成后，可以使用如下命令登陆本机:
```
ssh localhost
```


此时会有如下提示(SSH首次登陆提示)，输入 yes 。然后按提示输入密码 hadoop，这样就登陆到本机了。如下图所示：

![](http://i.imgur.com/hLkCf8M.png)


>此时，在~/下，如果没有`.ssh`文件夹，则会自动创建`~/.ssh`文件夹。

但这样登陆是需要每次输入密码的，我们需要配置成SSH无密码登陆比较方便。

首先退出刚才的 ssh，就回到了我们原先的终端窗口，然后利用 ssh-keygen 生成密钥，并将密钥加入到授权中：

```
exit                           # 退出刚才的 ssh localhost
cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat ./id_rsa.pub >> ./authorized_keys  # 加入授权
```
此时再用 ssh localhost 命令，无需输入密码就可以直接登陆了.

>只在master节点上配置ssh免密码登录本机即可，其它slave节点不需要配置。

## 3. master免密码登录slaves
在 master 节点将上公匙传输到 slave1 节点：
```
scp ~/.ssh/id_rsa.pub hadoop@slave1:~
```
>scp 是 secure copy 的简写，用于在 Linux 下进行远程拷贝文件，类似于 cp 命令，不过 cp 只能在本机中拷贝。
>执行 scp 时会要求输入 slave1 上 hadoop 用户的密码(hadoop)，输入完成后会提示传输完毕

接着在 slave1 节点上，将 ssh 公匙加入授权：
```
mkdir ~/.ssh       # 如果不存在该文件夹需先创建，若已存在则忽略
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
rm ~/id_rsa.pub    # 用完就可以删掉了
```
如果有其他 slave 节点，也要执行将 master 公匙传输到 slave 节点、在 slave 节点上加入授权这两步。

这样，在 master 节点上就可以无密码 SSH 到各个 slave 节点了，可在 master 节点上执行如下命令进行检验，如下图所示：

![](http://i.imgur.com/no5qaV8.png)


# 三、安装Java环境

查看我的博客：Ubuntu14.04安装JDK与配置环境变量

在各节点上都需要安装java环境。

# 四、安装hadoop集群

在master节点上：

## 1.下载hadoop压缩包
首先，去apache hadoop官网下载，hadoop压缩包，我下载的为：`hadoop-2.6.4.tar.gz`

## 2.解压

```
sudo tar -zxvf hadoop-2.6.4.tar.gz -C /opt	#解压到/opt下
sudo mv hadoop-2.6.4 hadoop		#重命名为hadoop
```
此时，压缩包被解压到/opt/hadoop下

## 3.修改配置文件
集群/分布式模式需要修改 /opt/hadoop/etc/hadoop 中的6个配置文件，更多设置项可点击查看官方说明，这里仅设置了正常启动所必须的设置项： hadoop-env.sh、slaves、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml 。

(1)hadoop-env.sh

修改JAVA_HOME,改为绝对路径，hadoop有时不能读取`$JAVA_HOME`的值.
```
# The java implementation to use.
export JAVA_HOME=/opt/java

```
(2)slaves
文件 slaves，将作为 DataNode 的主机名写入该文件，每行一个，默认为 localhost，所以在伪分布式配置时，节点即作为 NameNode 也作为 DataNode。分布式配置可以保留 localhost，也可以删掉，让 Master 节点仅作为 NameNode 使用。

本教程让 master 节点仅作为 NameNode 使用，因此将文件中原来的 localhost 删除，添加两行内容：
```
slave1
slave2
```

(3)core-site.xml

```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/opt/hadoop/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
</configuration>
```

(4)hdfs-site.xml

文件 hdfs-site.xml，dfs.replication 一般设为 3，但我们有两个 slave 节点，所以 dfs.replication 的值还是设为 2：

```
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>Master:50090</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/opt/hadoop/tmp/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/opt/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
```

(5)mapred-site.xml

文件 mapred-site.xml （可能需要先重命名，默认文件名为 mapred-site.xml.template），然后配置修改如下：

```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>Master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>Master:19888</value>
        </property>
</configuration>
```

(6)yarn-site.xml

```
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>Master</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
```
## 4.配置好后，将master上的`/opt/Hadoop`文件夹复制到各个节点上。

在master节点上执行：

```
tar -zcf ~/hadoop.master.tar.gz /opt/hadoop   # 先压缩再复制
cd ~
scp ./hadoop.master.tar.gz slave1:/home/hadoop	#复制到slave1上
scp ./hadoop.master.tar.gz slave2:/home/hadoop	#复制到slave2上
```

在slave1节点上执行：
```
sudo tar -zxf ~/hadoop.master.tar.gz -C /opt
sudo chown -R hadoop:hadoop /opt/hadoop
```

在slave2节点上执行的与在slave1节点上执行的相同。


注意：
>需要保证/opt/hadoop权限属于hadoop:hadoop,如果不是执行：

```
sudo chown -R hadoop:hadoop /opt/hadoop
```

如下图所示：

![](http://i.imgur.com/i2txxn9.png)

## 五、运行hadoop集群

### 1.配置hadoop环境变量

```
sudo vi /etc/profile
```

在最后添加如下内容：
```
#set hadoop env
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

### 2.启动hadoop集群

首次启动需要先在 master 节点执行 NameNode 的格式化：
```
hdfs namenode -format       # 首次运行需要执行初始化，之后不需要。
```

接着，可以启动 hadoop 了，启动需要在 master 节点上进行：
```
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
```
通过命令 `jps` 可以查看各个节点所启动的进程。正确的话，在 master 节点上可以看到 NameNode、ResourceManager、SecondrryNameNode、JobHistoryServer 进程，如下图所示：

![通过jps查看master的Hadoop进程](http://i.imgur.com/VBZrruo.png)

在 slave 节点可以看到 DataNode 和 NodeManager 进程，如下图所示：

![通过jps查看slave的Hadoop进程](http://i.imgur.com/mZCwp1f.png)

缺少任一进程都表示出错。另外还需要在 master 节点上通过命令 hdfs dfsadmin -report 查看 DataNode 是否正常启动，如果 Live datanodes 不为 0 ，则说明集群启动成功。例如我这边一共有 2 个 Datanodes：

![](http://i.imgur.com/K25fzmw.png)

也可以通过 Web 页面看到查看 DataNode 和 NameNode 的状态：[http://master:50070/](http://master:50070/)。如果不成功，可以通过启动日志排查原因。

由于本教程是在Windows上使用VirtualBox开启了master、slave1、slave2,3个虚拟机，如果要访问[http://master:50070](http://master:50070),需要将Windows中的hosts文件中添加一行映射地址：
```
192.168.1.104	master
```

笔者用的是Win10 64位系统，hosts文件在`C:\Windows\System32\drivers\etc`下，用notepad++打开，添加上面一行即可。

配置完hosts文件后，打开浏览器，输入网址：[http://master:50070](http://master:50070 "http://master:50070")，可以看到如下图效果：

![](http://i.imgur.com/E7vpxN6.png)

## 六、在hadoop集群上，执行分布式实例

首先，在 HDFS 上创建用户目录：

```
hdfs dfs -mkdir -p /user/hadoop
```

将 /etc/hadoop/etc/hadoop 中的配置文件作为输入文件复制到分布式文件系统中：

```
hdfs dfs -mkdir input
hdfs dfs -put /opt/hadoop/etc/hadoop/*.xml input
```

通过查看 DataNode 的状态（占用大小有改变），输入文件确实复制到了 DataNode 中，如下图所示:

![通过Web页面查看DataNode的状态](http://i.imgur.com/ISH8yzd.png)

接着就可以运行 MapReduce 作业了：

```
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.4.jar grep input output 'dfs[a-z.]+'
```

运行时的输出信息，会显示 Job 的进度,如下图所示：

![](http://i.imgur.com/Rdi4gJN.png)

同样可以通过 Web 界面查看任务进度 [http://master:8088/cluster](http://master:8088/cluster)，在 Web 界面点击 “Tracking UI” 这一列的 History 连接，可以看到任务的运行信息，如下图所示：

![](http://i.imgur.com/F2MIQSP.png)

执行完毕后的输出结果：
```
hdfs dfs -cat output/*
```
结果，如下图所示：

![](http://i.imgur.com/2O4Gudh.png)


##最后，注意：

如果，上面配置教程中用到的安装包或文件，是从Windows上传到虚拟机的，需要更改上传文件的权限：

```
sudo chown -R hadoop:hadoop 上传文件
```




