---
title: MapReduce之倒排索引类应用
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 应用需求

通常在数据文件中包含大量的单词，每个单词可能会出现多次，需要根据单词查找文档，这时就需要用到倒排索引。

## 应用场景

在全文检索系统或搜索引擎中，经常会用到根据单词查找文档。

## 解决方案

通常在 Map 过程中，对文档进行切分，把单词和文档URL设置为 Key，单词为文档中的次数为 Value，使用 Combine 函数对文档中的词频进行统计，然后将 单词作为 Key，文档URL和词频作为 Value 输出到 Reduce 中，Reduce 函数以单词为 Key，生成倒排索引。

## 倒排索引

倒排索引主要是用来存储某个单词（或词组）在一个文档或一组文档中的存储位置的映射，即提供一种根据内容来查找文档的方式。由于不是根据文档来确定文档所包含的内容，而是进行相反操作，因而成为倒排索引。

通常情况下，倒排索引由一个单词（或词组）以及相关的文档列表组成，文档列表中的文档或者是表示文档的ID号，或者是指文档所在位置的URL。在实际应用中文档通常带有权重，即记录单词在文档中出现的次数。如下面所示。
有下面三个文档：
```
D0.txt:
MapReduce is simple is easy
```

```
D1.txt:
MapReduce is powerful is userful
```

```
D2.txt:
Hello MapReduce Bye MapReduce
```

则 D0.txt、D1.txt和 D2.txt的倒排索引文件，如下所示：

```
Bye	D2.txt:1;
Hello	D2.txt:1;
MapReduce	D2.txt:2;D1.txt:1;D0.txt:1;
easy	D0.txt:1;
is	D0.txt:2;D1.txt:2;
powerful	D1.txt:1;
simple	D0.txt:1;
userful	D1.txt:1;
```

如上面倒排索引所示，倒排索引文件中的 MapReduce 一行表示：MapReduce 这个单词在文本D0中出现1次，D1中出现1次，D2中出现2次。
当搜索条件为 MapReduce、is、Simple时，对应的集合为：{D0,D1,D2} && {D0,D1} && {D0} = {D0}，即文档 D0 包含了所要索引的单词，而且是连续的。

在实际的搜索引擎应用中，除了考虑词频外，还要考虑单词出现的位置，比如单词出现在标题和 URL 中就比出现在正文中的权重高。


## 分析与设计

### Map过程

首先使用默认的TextInputFormat类对输入文件进行处理，得到文本中每行的偏移量及其内容，Map过程首先必须分析输入的<key, value>对，得到倒排索引中需要的三个信息：单词、文档URI和词频，如图所示：

<center>![InvertedIndexMapper](http://oni2hc3a8.bkt.clouddn.com/bigdataInvertIndexMapper.jpg)</center>

存在两个问题，第一：<key, value>对只能有两个值，在不使用Hadoop自定义数据类型的情况下，需要根据情况将其中的两个值合并成一个值，作为value或key值；

第二，通过一个Reduce过程无法同时完成词频统计和生成文档列表，所以必须增加一个Combine过程完成词频统计.

这里将单词和文档URL组成 Key 值（如 MapReduce:D0.txt）,将词频作为 Value，这样做的目的是可以将同一文档的相同单词的词频组成列表，传递给 Combine 过程，就可以计算同一文档中单词的词频。

```java
package com.test.invertedindex;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class InvertedIndexMapper extends Mapper<LongWritable, Text, Text, Text>{

	private Text keyInfo = new Text();	//存储单词和文档URL组合
	private Text valueInfo = new Text();	//存储词频
	private FileSplit fileSplit;	//存储 FileSplit对象，目的为获取文档URL
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		//获取单词所属的 FileSplit 对象。
		fileSplit = (FileSplit) context.getInputSplit();
		
		String line = value.toString();
		StringTokenizer itr = new StringTokenizer(line);
		while(itr.hasMoreTokens()){
			//获取文件完整路径
			//keyInfo.set(itr.nextToken() + ":" + fileSplit.getPath().toString());
			//这里只获取文件的名称
			int splitIndex = fileSplit.getPath().toString().indexOf("D");
			//key值由单词和文档URL组成，如 "MapReduce:D0.txt"
			keyInfo.set(itr.nextToken() + ":" + fileSplit.getPath().toString().substring(splitIndex));
			
			//词频初始化为1
			valueInfo.set("1");
			
			context.write(keyInfo, valueInfo);
		}
	}

}

```

### Combine 过程

经过 Map 方法处理后，Combine过程将key值相同的value值累加，得到一个单词在文档中的词频，如图。

<center>![InvertedIndexCombiner](http://oni2hc3a8.bkt.clouddn.com/bigdataInvertedIndexCombiner.jpg)</center>

如果直接将上图所示的输出作为Reduce过程的输入，在Shuffle过程将面临一个问题：所有具有相同单词的记录（由单词、文档URL和词频组成）应该交由同一个Reducer处理，但当前的Key值无法保证这一点，所以必须修改Key值和Value值。这次将单词作为Key值，文档URL和词频组成Value值（如 D0.txt:1）.这样做的好处是可以利用MapReduce框架默认的HashPartitioner类完成Shuffle过程，将相同单词的所有记录发送给同一个Reducer进行处理。

```java 
package com.test.invertedindex;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

//计算每个单词所在文档的词频
public class InvertedIndexCombiner extends Reducer<Text, Text, Text, Text>{

	private Text info = new Text();
	
	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context)
			throws IOException, InterruptedException {
		//统计词频
		int sum = 0;
		for (Text val : values) {
			sum += Integer.parseInt(val.toString());
		}
		
		int splitIndex = key.toString().indexOf(":");
		
		//重新设置 value 的值为文档URL和词频组成。
		info.set(key.toString().substring(splitIndex + 1) + ":" + sum);
		//重新设置 key 的值为单词
		key.set(key.toString().substring(0, splitIndex));
		
		
		context.write(key, info);
	}

}
```

### Reduce 过程

经过上述两个过程后，Reduce过程只需将相同key值的value值组合成倒排索引文件所需的格式即可，剩下的事情就可以直接交给MapReduce框架进行处理。如图。
<center>![InvertedIndexReducer](http://oni2hc3a8.bkt.clouddn.com/bigdataInvertedIndexReducer.jpg)</center>


