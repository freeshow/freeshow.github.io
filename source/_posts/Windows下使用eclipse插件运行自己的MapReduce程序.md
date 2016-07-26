---
title: Windows下使用Eclipse插件运行自己的MapReduce程序
date: {{ date }}
tags: Hadoop 
categories:
toc: false
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

在上一篇博客中：[Windows下使用eclipse编译打包运行自己的MapReduce程序](http://freeshow.github.io/2016/07/24/Windows下使用eclipse编译打包运行自己的MapReduce程序 Hadoop2.6.0/ ) 中，开发完成的jar包需要上传到集群并使用相应的命令才能执行，这对不熟悉Linux的用户仍具有一定困难，而使用Hadoop Eclipse插件能很好的解决这一问题。

Hadoop Eclipse插件不仅能让用户直接在本地(Windows下)提交任务到Hadoop集群上，还能调试代码、查看出错信息和结果、使用图形化的方式管理HDFS文件。

Hadoop Eclipse插件需要单独从网上获取，获取后，可以自己重编译，也可直接使用编译好的release版本，经过测试，笔者发现从网上获取的插件可以直接使用，下面介绍如何获取和简单使用Eclipse插件。

# 一、获取Hadoop Eclipse插件

获取Hadoop Eclipse插件的地址是：[https://github.com/winghc/hadoop2x-eclipse-plugin](https://github.com/winghc/hadoop2x-eclipse-plugin)

本文使用的是hadoop-2.6.4，所以使用的插件版本是`hadoop-eclipse-plugin-2.6.0.jar`。

# 二、使用Hadoop Eclipse插件

在Eclipse中使用插件非常简单，对于本文所使用的Eclipse Jee版本，只需要关闭Eclipse，把上述插件复制到 `Eclipse\dropins`目录下，在重新打开Eclipse即可。对于其他的Eclipse版本，可能需要复制到 `Eclipse\plugins` 目录下。

## 1.查找Eclipse插件

启动Eclipse后，依次点击 Windows -> Show View -> Other。在新弹出的选项框中找到 `Map/Reduce Locations`， 选中后单击 OK 按钮。如下图所示：

<center>![](http://i.imgur.com/cIfR0k2.png)</center>

单击 OK 后，在Eclipse下面的视图中会多出一栏 Map/Reduce Locations,如下图所示：

<center>![](http://i.imgur.com/Fue1psJ.png)</center>

然后单击 Windows -> Show View -> Project Expore,在Eclipse左侧视图中会显示项目浏览器，项目浏览器中最上面会出现 DFS Locations,如下图所示：

<center>![](http://i.imgur.com/rSfDgLQ.png)</center>

Map/Reduce Locations 用于建立连接到Hadoop 集群，当连接到Hadoop集群后，DFS Locations 则会显示相应集群 HDFS 中的文件。 Map/Reduce Locations 可以一次连接到多个Hadoop集群。

在 Map/Reduce Locations 下侧的空白处右击，在弹出的选项中选择 New Hadoop location,新建一个Hadoop连接，之后会弹出 Hadoop location 的详细设置窗口，如下图所示，各项解释如下。

- Location name: 当前建立的Hadoop location命名。
- Map/Reduce Host: 为集群nomenode的IP地址。
- Map/Reduce Port: MapReduce任务运行的通信端口号，客户端通过该地址向RM提交应用程序，杀死应用程序等。在yarn-site.xml中，默认值：${yarn.resourcemanager.hostname}:8032 
- DFS Master Use M/R Master host: 选中表示采用和 Map/Reduce Host一样的主机。
- DFS Naster Host： 为集群namenode的IP地址。
- DFS Master Port: HDFS端口号，对应core-site.xml中定义的fs.defaultFS参数中的端口号，一般为9000;
- User Name:设置访问集群的用户名，默认为本机的用户名。

<center>![](http://i.imgur.com/okOUyOd.png)</center>

配置完成后，单击Finish即可完成 Hadoop location的配置。在 Advanced parameters选项卡中还可以配置更多细节，但在实际使用中非常繁琐，<font color='red'>相应的设置在代码中也可以进行，或者将Hadoop集群的配置文件放到Eclipse目录下，自动完整配置。</font> 这里只需要配置General选项卡的内容即可。这是，右侧Project Expore中的DFS locations中会多出一个子栏，名字为上面设置的Hadoop location名称。

## 2.使用Eclipse插件管理HDFS

如果前面的配置参数没有问题，Hadoop集群也已经启动，那么Eclipse插件会自动连接Hadoop集群的HDFS，并获取HDFS的文件信息。变可以在上面操作HDFS。

需要注意的是，`Refresh`只对选中的项目有效，如果是文件，那么只刷新该文件的相关信息；如果是文件夹，则只刷新该文件夹下的内容。

还需要值的一提的是，为了安全，HDFS的权限检测机制默认是打开的，关闭之后，才能使用Eclipse插件上传文件到HDFS或者从HDFS中删除文件。

为了能在Windows上直接操作Hadoop集群中的HDFS:

第一步：
如果只在测试环境下，直接把Hadoop集群中的HDFS的权限检测关闭，可在hdfs-site.xml中添加如下变量，重启Hadoop集群即可。

```xml
<property>
	<name>dfs.permissions</name>
	<value>false</value>
</property>
```

第二步：
修改Windows本地主机名

首先，“右击”桌面上图标“我的电脑”，选择“管理”，接着选择“本地用户和组”，展开“用户”，找到当前系统用户，修改其为“hadoop"。

最后，把电脑进行“注销”或者“重启电脑”，这样修改的用户名才有效。

选用上述任意一种方法以后（如果只是学习用，推荐第一种方法），就可以用Hadoop Eclipse插件提供的图像化界面操作Hadoop集群中的HDFS了。

<center>![](http://i.imgur.com/OhPtnF6.png)</center>



# 三、在Eclipse中提交任务到Hadoop

使用Eclipse插件可以直接在Eclipse环境下采用图形操作的方式提交任务，可以极大的简化了开发人员提交任务的步骤。

## 1.配置本地Hadoop目录和输入输出目录

首先，需要在Eclipse中设置本地Hadoop目录，假设安装hadoop的压缩包解压到本地 `F:\hadoop-2.6.4`下，在Eclipse界面单击 Windows -> Preference 弹出设置界面，在设置界面找到 Hadoop Map/Reduce,在 Hadoop installation dierctory后面填上 `F:\hadoop-2.6.4`

此处，需要注意，解压的 `F:\hadoop-2.6.4`源码包是在Linux环境下的源码包，与Windows不兼容。
在Windows下提交任务是会出现`Failed to lacation the winutils binary in hadoop binary path`,需要使用如下操作进行修复：
（1） 下载Window下的运行包
 将 `https://github.com/steveloughran/winutils/`项目下的 `hadoop-2.6.4/bin`目录下的所有文件覆盖掉`F:\hadoop-2.6.4\bin`下的所有文件。
（2） 复制`F:\hadoop-2.6.4\bin\hadoop.dll`文件到`C:\Windows\System32`中。
（3）配置环境变量 HADOOP_HOME为F:\hadoop-2.6.4,并将`$HADOOP_HOME\bin`添加到PATH环境变量中去。
（4）重启电脑。

在向Hadoop集群提交任务时，还需要指定输入/输出目录，在Eclipse中，可按如下操作进行设置：双击打开工程的某代码文件，在代码编辑区 右键 -> Run As -> Run Configurations. 在弹出的窗口中找到 Java Application -> WordMain, 单击 WordMain 进入设置界面，单击 Arguments 切换选项卡，在 Program arguments 下的文本框中指定输入/输出目录。

第一行目录为输入目录，第二行目录为输出目录，格式为 `hdfs://[namenode_ip]:[端口号][路径]`。
注意，输出目录在HDFS中不能存在。如下图所示：

<center>![](http://i.imgur.com/4GjcRmn.png)</center>

注意：上图中的master我已经在本地主机的hosts文件中映射为namenode的IP地址了。

## 为WordCount添加配置信息。

本次演示，使用 [Windows下使用eclipse编译打包运行自己的MapReduce程序 Hadoop2.6.0](http://freeshow.github.io/2016/07/24/Windows下使用eclipse编译打包运行自己的MapReduce程序 Hadoop2.6.0/)中的例子。

添加配置信息有两种方法：
（1） 使用con.set()方法，设置配置信息。
（2） 将Hadoop集群中的修改过的配置文件，如`hdfs-site.xml`,`core-site.xml`,`mapred-site.xml`,`yarn-site.xml`,`log4j.properties`,复制到 WordCount 项目下的 src 文件夹下。

我比较喜欢用第二种方法。

<font color='red'>**注意：**</font>
配置信息完成后，在建立Job类对象后面也新增一行代码：
```java
job.setJar("wordcount.jar");
```
用户告诉hadoop集群所要运行的Jar文件，所以需要先导出WordCount项目为jar文件，位置位于项目根目录下，因为上面代码`job.setJar("wordcount.jar");`查找目标的相对路径为WordCount项目根目录。

<font color='red'>**运行时出现的一个问题:**</font>

在通过Windows客户端向Linux服务器提交Hadoop应用时，会提示如下错误：
```java
org.apache.hadoop.util.Shell$ExitCodeException: /bin/bash: line 0: fg: no job control

        at org.apache.hadoop.util.Shell.runCommand(Shell.java:505)
        at org.apache.hadoop.util.Shell.run(Shell.java:418)
        at org.apache.hadoop.util.Shell$ShellCommandExecutor.execute(Shell.java:650)
        at org.apache.hadoop.yarn.server.nodemanager.DefaultContainerExecutor.launchContainer(DefaultContainerExecutor.java:195)
        at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:300)
        at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:81)
        at java.util.concurrent.FutureTask.run(FutureTask.java:262)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
```

解决方法是：
在向项目文件夹下的src目录下添加的配置文件`mapred-site.xml`中添加如下信息：
```xml
<property>
        <name>mapreduce.app-submission.cross-platform</name>
        <value>true</value>
</property>
```
不需要向hadoop集群的`mapred-site.xml`配置文件中添加。

## 3.提交任务
提交任务非常简单，直接在代码编辑区 右键 -> Run As -> Run on Hadoop 即可。


# 四、建立Map/Reduce项目
在应用Hadoop Eclipse插件后，可以直接在Eclipse中建立Map/Reduce项目，该项目会自动引用相应的jar包，路径为设置的本地`F:\hadoop-2.6.4`目录，所以在项目中不用再进行建立lib文件夹，复制jar包等操作。

在Eclipse中一次单击 File -> New -> Project,弹出项目类型选择对话框，选择 Map/Reduce Project,单击 Next ,在新弹出的对话框中填上项目名称：wordcount2 ，单击 Finish 即可。

之后再项目中建立一个`WordCount2`类，插入如下代码：

```java
package wordcount2;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;


public class WordCount2 {
	public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable>{
		private final static IntWritable one = new IntWritable(1);
		private Text word = new Text();
		
		@Override
		protected void map(Object key, Text value, Context context)
				throws IOException, InterruptedException {
			StringTokenizer itr = new StringTokenizer(value.toString());
			while (itr.hasMoreTokens()) {
				word.set(itr.nextToken());
				context.write(word, one);
			}
		}
	}
	
	public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
		private IntWritable result = new IntWritable();
		
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,Context context)
				throws IOException, InterruptedException {
			int sum = 0;
			for(IntWritable val : values){
				sum += val.get();
			}
			result.set(sum);
			context.write(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		// Configuration 类: 读取hadoop的配置文件，如 site-core.xml...;
				//也可以用set方法重新设置(会覆盖): conf.set("fs.defaultFS","hdfs://master:9000")
				Configuration conf = new Configuration();
				
				//将命令行中的参数自动设置到变量conf中
				String[] otherArgs = new GenericOptionsParser(conf,args).getRemainingArgs();
				
				if (otherArgs.length != 2) {
					System.err.println("Usage: wordcount <in> <out>");
					System.exit(2);
				}
				
			    Job job = new Job(conf,"word count2");	//新建一个job,传入配置信息
				job.setJar("wordcount2.jar");
				job.setJarByClass(WordCount2.class);	//设置主类
				job.setMapperClass(TokenizerMapper.class);	//设置Mapper类
				job.setReducerClass(IntSumReducer.class);	//设置Reducer类
				job.setOutputKeyClass(Text.class);	//设置输出类型
				job.setOutputValueClass(IntWritable.class);	//设置输出类型
				FileInputFormat.addInputPath(job, new Path(otherArgs[0]));	//设置输入文件
				FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));	//设置输出文件
				
				boolean flag = job.waitForCompletion(true);
				System.out.println("SUCCEED!"+flag);	//任务完成提示
				System.exit(flag ? 0 : 1);
				System.out.println();
				
	}
}
```

之后：
（1）在Eclipse设置输入/输出目录
（2）导出项目jar包，存储到工程目录下，即可提交任务 Run on Hadoop.

注意，使用Map/Reduce插件建立的项目在运行时控制台并没有日志输出，所以在上面的代码最后添加一行输出 `System.out.println("SUCCEED!"+flag);`,当控制台最后输出`SUCCEED!true`时，表示任务运行成功，这是可以刷新 DFS Locations,会看到输出结果已经出来了。


本教程来自《Hadoop大数据处理技术基础与实战》--安俊秀 编著。

郑重声明：

在Windows下运行MapReduce程序，各种错误都有，如果有条件的话，建议在Linux下编程，Windows下实在麻烦。

















