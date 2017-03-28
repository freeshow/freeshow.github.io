---
title: MapReduce之去重计数类应用
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 应用需求

在大数据文件中包含了大量的记录，每条记录记载了某事物的一些属性，需要根据某几个属性的组合，去除相同的重复组合，并统计其中某属性的统计值。

## 解决方法

在此类应用中，将计算过程分为两个步骤。
第一步，map 函数将每条记录中需要关注的属性组合作为关键字，将空字符串作为值，生成的<键-值>对作为中间值输出。
第二步，reduce 函数则将输入的中间结果的 key 作为新的 key,value仍然取空字符串，输出结果。
因为所有相同的 key 都被送到同一个 reducer，而 reducer 只输出了一个 key，这一过程实际上就是去重的过程。

## 应用案例

以下两个文件，文件中表示某天，某IP访问了系统这样一个日志。当时间和IP相同时，将这种相同的数据去掉，只留一个。

```
log1.txt:
2014-10-3	10.3.5.19
2014-10-3	10.3.3.19
2014-10-3	10.3.5.18
2014-10-3	10.3.51.19
2014-10-3	10.3.2.19
2014-10-4	10.3.2.5
2014-10-4	10.3.2.18
```

```
log2.txt
2014-10-3	10.3.5.19
2014-10-4	10.3.5.19
2014-10-3	10.3.5.18
2014-10-5	10.3.51.19
2014-10-4	10.3.2.5
2014-10-5	10.3.2.19
```

## 程序代码

### UniqMapper

```java
package com.test.uniq;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class UniqMapper extends Mapper<LongWritable, Text, Text, Text>{

	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		context.write(value, new Text(""));
	}

}

```

### UniqReducer

```java
package com.test.uniq;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class UniqReducer extends Reducer<Text, Text, Text, Text>{

	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context)
			throws IOException, InterruptedException {
		context.write(key, new Text(""));
	}
}

```

### UniqRunner

```java
package com.test.uniq;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class UniqRunner extends Configured implements Tool{

	@Override
	public int run(String[] args) throws Exception {
		
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(UniqRunner.class);
		
		job.setMapperClass(UniqMapper.class);
		job.setReducerClass(UniqReducer.class);
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		return job.waitForCompletion(true) ? 0:1;
	}
	
	public static void main(String[] args) throws Exception {
		int res = ToolRunner.run(new Configuration(),new UniqRunner(), args);
		System.exit(res);
	}
}

```

### 运行结果

```
2014-10-3	10.3.2.19	
2014-10-3	10.3.3.19	
2014-10-3	10.3.5.18	
2014-10-3	10.3.5.19	
2014-10-3	10.3.51.19	
2014-10-4	10.3.2.18	
2014-10-4	10.3.2.5	
2014-10-4	10.3.5.19	
2014-10-5	10.3.2.19	
2014-10-5	10.3.51.19	

```