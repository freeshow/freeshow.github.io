---
title: {{ title }}
date: {{ date }}
tags: Hadoop 
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>


# 一、相关文件准备
## 1. java JDK for Windows
## 2. hadoop-2.6.4.tar.gz 就是安装hadoop时使用的文件
## 3. Eclipse JEE版本

![](http://i.imgur.com/dbjp1oz.png)

# 二、环境准备

## 1.安装java并配置环境
自己百度

## 2.解压hadoop-2.6.4.tar.gz源文件
Hadoop源文件在整个开发过程中都会用到，因为很多依赖包都出自里面，用户可按自己的喜好选择位置，但路径层次最好不要太多，本文选在解压到E盘根目录下，即`E:\hadoop-2.6.4`

## 3.安装Eclipse
自己百度

# 三、使用Eclipse创建一个Java工程
使用Eclipse创建一个名为`wordcound`的Java工程

# 四、导入Hadoop的相关jar包

在编写MapReduce代码时，需要用到Hadoop源文件中的部分Jar包，就像在编写纯Java代码时需要使用Java自带的依赖包一样，所以这里需要把相应的Hadoop依赖包导入工程。

现在工程 wordcount上右键，在弹出的菜单中选择第一个 New（新建），在选择Folder(文件),名称填上lib; 然后在把下面目录下的jar包复制到lib文件夹下(之前把Hadoop源文件解压到E盘根目录下)。

```
E:\hadoop-2.6.4\share\hadoop\common
E:\hadoop-2.6.4\share\hadoop\common\lib
E:\hadoop-2.6.4\share\hadoop\common\lib\hadoop-hdfs-2.6.4.jar
E:\hadoop-2.6.4\share\hadoop\mapreduce
E:\hadoop-2.6.4\share\hadoop\yarn
```

导入Jar包后，还需要把这些jar包添加到工程的构建路径，否则工程并不能识别。选中所有的jar包然后单击右键，选择Build Path -> Add to Build Path.

上面就是Eclipse导入jar包的其中一种方法，其他方法也可以，只要让Eclipse程序能够引用上面的Jar包即可。

# 五、 MapReduce 代码实现

本代码演示 wordcount程序。

MapReduce代码实现并不难，这里要编写3个类，分别是WordMapper类、WordReducer类和WordMain驱动类，前面两个类分别实现相应的 Map 和 Reduce 方法，后面一个则是对任务的创建进行部署。

分别创建这3个类，并放入wordcount package下，目录结构如下：

![](http://i.imgur.com/w9gmeGt.png)

## WordMapper.java

```java
package wordcount;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

//创建一个 WordMapper 类继承与 Mapper 抽象类
public class WordMapper extends Mapper<Object, Text, Text, IntWritable>{
	private final static IntWritable one = new IntWritable(1);
	private Text word = new Text();
	
	//Mapper 抽象类的核心方法，三个参数
	@Override
	protected void map(Object key,	//首字符偏移量	 
					  Text value, 	//文件的一行内容
					  Context context)	//Mapper端的上下文
			throws IOException, InterruptedException {
		//默认使用空格分隔
		StringTokenizer itr = new StringTokenizer(value.toString());
		while(itr.hasMoreTokens()){
			word.set(itr.nextToken());
			context.write(word, one);
		}
	}
}
```

map函数实现了对传入值的解析，将value解析成`<key, value>`的形式，然后使用`context.write(word, one)`进行输出。

## WordReducer.java

```java
package wordcount;

import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

//创建一个 WordReducer 类继承与 Reducer 抽象类
public class WordReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
	private IntWritable result = new IntWritable();	//记录词频
	
	// Reducer 抽象类的核心方法，3个参数
	@Override
	protected void reduce(Text key,	//Map 端输出的 key 值
						Iterable<IntWritable> values,	//Map 端输出的 Value 集合
						Context context)	//Reducer端上下文
						throws IOException, InterruptedException {
		int sum = 0;
		
		for (IntWritable var : values) {	//遍历 values 集合，并把值相加
			sum += var.get();
		}
		
		result.set(sum);	//得到最终词频数
		context.write(key, result);		//写入结果
		
	}
}

```
reduce方法中，将获取的values进行遍历累加，得到相应的key出现的次数，最后将结果写入HDFS。

## WordMain.java

WordMain驱动类主要是在Job中设定相应的Mapper类和Reducer类(用户编写的类)，这样任务运行时才知道使用相应的类进行处理；WordMain驱动类还可以对MapReducer程序进行相应配置，让任务在Hadoop集群运行时按所定义的配置进行。其代码如下：

```java
package wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordMain {
	
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
		
		Job job = new Job(conf,"word count");	//新建一个job,传入配置信息
		job.setJarByClass(WordMain.class);	//设置主类
		job.setMapperClass(WordMapper.class);	//设置Mapper类
		job.setReducerClass(WordReducer.class);	//设置Reducer类
		job.setOutputKeyClass(Text.class);	//设置输出类型
		job.setOutputValueClass(IntWritable.class);	//设置输出类型
		FileInputFormat.addInputPath(job, new Path(otherArgs[0]));	//设置输入文件
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));	//设置输出文件
		System.exit(job.waitForCompletion(true) ? 0 : 1);	//等待完成退出
		
	}
}

```

该类中的main方法就是MapReduce程序的入口，在main方法中，首先创建一个Configuration类对象conf用于保存所有的配置信息，该对象在创建时会读取所需要配置文件如 site-core.xml、hdfs-site.xml等，根据配置文件中的变量信息进行初始化，当然配置文件中的配置有时候并不是人们想要的，这时候可以调用Configuration类中的set方法进行覆盖，如想要修改Reducer的数量，可以使用如下方法：

```java
conf.set("mapreduce.job.reduces","2");
```

也不是所有的变量都可以修改，有时候集群管理员并不希望用户在应用程序中修改某变量的值，这时候会在相应变量后面添加final属性：

```xml
<property>
	<name>mapreduce.task.io.sort.factor</name>
	<value>10</value>
	<final>true</final>
</property>
```
这时候，Configuration类中在set上面的属性将不再起左右。

最后，main方法中创建一个Job类对象job,并传入配置信息conf和作业名称。之后对job对象进行相关设置，如Mapper类、Reducer类等。job对象就是最终的作业对象，它里面包含一个作业所需的所有信息。

至此，一个MapReduce程序便开发完成了。

# 六、打包工程为jar包

WordCount代码完成后，并不能直接在hadoop中运行，还需要将其打包成jvm所能执行的二进制文件，即打包成.jar文件，才能被hadoop所有。

在WordCount项目上右击，选择Export(导出),在弹出的对话框中选择 JAR file，如下图所示，然后单击Next。之后会进入JAR依赖包过滤对话框，这里只选择src即可，把lib文件夹前的勾选去掉，因为lib中的依赖包本来就是复制的hadoop的源文件，在集群中已经包含了。之后选择一个保存位置，单击Finish即可。

![选择Jar file](http://i.imgur.com/UGQVGbE.png)

![jar依赖包过滤](http://i.imgur.com/sMsTRJx.png)

打包成wordcount.jar

WordMain驱动类为wordcount.WordMain。

# 七、部署并运行

部署其实就把前面打包生成的wordcount.jar包放入集群中运行。hadoop一般会有多个节点，一个namenode节点和多个datanode节点，这里只需要把jar放入namenode中，并使用相应的hadoop命令即可，hadoop集群会把任务传送给需要运行任务的节点。wordcount.jar运行时需要有输入文本。

## 1.创建测试文本并上传相关文件到namenode中

为了方便，在桌面上创建测试文本file1.txt、file2.txt。内容分别为

```
File: file1.txt					File:file2.txt
hadoop is very good				hadoop is very good
mapreduce is very good			 mapreduce is very good
```
然后使用WinSCP工具把上述txt文件和wordcount.jar文件一起上传到namenode节点的hadoop用户目录下，hadoop用户指的是安装运行hadoop集群的用户，本文的用户名就为hadoop.

注意：
上传结束后，需要查看上传文件的权限是否为hadoop:hadoop(hadoop用户和hadoop组)，如果不是则需要将上传文件的权限改为hadoop:hadoop,命令为：
```
sudo chown -R hadoop:hadoop file1.txt
sudo chown -R hadoop:hadoop file2.txt
sudo chown -R hadoop:hadoop wordcount.jar
```
如下图所示：

![](http://i.imgur.com/qlFHGQP.png)

## 2.上传测试文件到HDFS

```
hdfs dfs -mkdir input	//创建输入文件夹input
hdfs dfa -put file* input	//将file1.txt file2.txt放入input文件夹中
```

## 3.在hadoop集群中运行WordCount

测试文件已经准备完毕，现在要做的就是把任务提交到hadoop集群中。
在hadoop中运行jar任务需要使用的命令：
```
hadoop jar [jar文件位置] [jar 主类] [HDFS输入位置] [HDFS输出位置]
```
- hadoop: hadoop脚本命令，如果要直接使用，必须添加相应bin路径到环境变量PATH中。
- jar: 表示要运行的是一个基于Java的任务。
- jar文件位置： 提供所要运行任务的jar文件位置，如果在当前操作目录下，可直接使用文件名。
- jar主类： 提供入口函数所在的类，格式为[包名.]类名
- HDFS输入位置： 指定输入文件在HDFS中的位置。
- HDFS输出位置： 执行输出文件在HDFS中的存储位置，该位置必须不存在，否则任务不会运行，该机制就是为了防止文件被覆盖出现意外丢失。

本例的操作命令如下：
```
hadoor jar wordcount.jar wordcount.WordMain input output
```

提交任务后，hadoop集群便会开始执行任务，在任务的执行过程中，会出现一系列任务提示或信息进度，如下所示：

![](http://i.imgur.com/IaQSzgL.png)

## 4.查看任务结果

任务结束保存在设定的输出目录中，如下图所示：

![](http://i.imgur.com/3MBdhW4.png)

- _SUCCESS： 该文件中无任何内容，生成它主要是为了使hadoop集群检测并停止任务。
- part-r-00000： 由Reducer生成的结果文件，一般来说一个Reducer生成一个，本例中只有一个Reducer运行，所以结果文件只有一个。


可以使用hdfs dfs中的-cat命令查看结果：

```
hdfs dfs -cat output/*
```
结果如下图所示：

![](http://i.imgur.com/hooLx5f.png)

至此，一个MapReducer程序的开发过程就结束了。

## 本文转自：《Hadoop大数据处理技术基础与实践》--安俊秀 编著








