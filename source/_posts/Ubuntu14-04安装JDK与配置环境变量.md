---
title: Ubuntu14.04安装JDK与配置环境变量
date: 2016-07-23 15:37:35
tags: Linux
categories:
---

# 一、下载JDK
先从Oracle官网下载JDK。先选择同意按钮，然后根据自己的系统下载相应版本。我的系统是Ubuntu14.04 64位的，所以我下载的是`jdk-8u91-linux-x64.tar.gz`

<!-- more -->

# 二、解压到合适的目录下
```
sudo tar -zxvf jdk-8u91-linux-x64.tar.gz -C /opt/  #解压到/opt目录下
cd /opt
sudo mv jdk1.8.0_91 java	#将解压后的文件夹重命名为java,方便配置环境变量。
```
故，jdk解压的文件现在在`/opt/java`文件夹下。

# 三、配置环境变量
有两种配置方式：
1. 修改/etc/profile(对所有用户有效)
2. 修改~/.bashrc(对当前用户有效)

此处以修改`/etc/profile`文件为例。

```
sudo vi /etc/profile
```

在文件末尾添加如下配置：
```
# set java env
export JAVA_HOME=/opt/java
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin

```

最后一定要执行，下面命令（要不然，配置文件不生效）：
```
source /etc/profile
```
# 四、测试

```
java -version
```

显示如下：
```
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)

```
则证明配置成功。


