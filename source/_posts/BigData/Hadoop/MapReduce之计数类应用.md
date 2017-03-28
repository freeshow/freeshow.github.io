---
title: MapReduce之计数类应用
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 应用需求

在数据文件中包含大量的记录，每条记录中包含某类事物的若干属性，在实际应用中需要根据这类事物的某个属性进行数值计算，如求和、平均值等。

## 解决方案

针对这类应用，在 Map 函数中提取每条记录中这类事物的特定属性值，在 Reduce 函数中对所有相同事物属性值按照函数表达式进行计算。

## 应用案例

WordCount 就是经典的计数类应用中求和案例，下面通过另一案例讲解求平均值的方法。现在一个班级有 Rose、Andy、Tom、John、Michelle、Amy、Kim等同学，学习了 English、Math、Chinese 三门课程，一门课程是一个文本文件，通过运算求每个同学的平均成绩。文件内容如下。

```
English.txt:		
Rose		91	
Andy		87
Tom		 78
John		94
Michelle	74
Amy		 67
Kim		 71
```

```
Math.txt:		
Rose		83	
Andy		93
Tom		 67
John		92
Michelle	82
Amy		 85
Kim		 80
```

```
Chinese.txt:		
Rose		85	
Andy		84
Tom		 85
John		77
Michelle	93
Amy		 94
Kim		 83
```

## 程序代码

### AverageMapper

```java
package com.test.score;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class AverageMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

	private Text name = new Text();
	private IntWritable score = new IntWritable();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		StringTokenizer itr = new StringTokenizer(line);
		while(itr.hasMoreTokens()){
			name.set(itr.nextToken());
			score.set(Integer.parseInt(itr.nextToken()));
			context.write(name, score);
		}
	}

}

```

### AverageReducer

```java
package com.test.score;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class AverageReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context)
			throws IOException, InterruptedException {
		int sum = 0;
		int count = 0;
		for (IntWritable val : values) {
			sum += val.get();
			++count;
		}
		int avg = sum / count;
		context.write(key, new IntWritable(avg));
	}

}

```

### AverageRunner

```java
package com.test.score;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class AverageRunner extends Configured implements Tool{

	@Override
	public int run(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(AverageRunner.class);
		
		job.setMapperClass(AverageMapper.class);
		job.setReducerClass(AverageReducer.class);
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		return job.waitForCompletion(true) ? 0:1;
	}
	
	public static void main(String[] args) throws Exception {
		int res = ToolRunner.run(new Configuration(), new AverageRunner(), args);
		System.exit(res);
	}
}

```

### 运行结果

```
Amy	82
Andy	88
John	87
Kim	78
Michelle	83
Rose	86
Tom	76
```




