---
title: MapReduce之简单排序类应用
toc: false
date: 2017-03-27 22:23:13
tags: Hadoop
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 应用需求

通常在数据文件中包含大量的记录，每条记录中包含了这个事物的某个属性，需要根据这个属性对数据进行排序。

## 解决方案

map 函数对每条记录的事物和属性按照特定的规则进行计算，获得属性值，并以属性为 key,value为原数据值。reduce 函数对同组的排序值进行排序后按顺序输出。

## 应用案例

对输入文件中数据进行排序。输入文件中的每行内容均为一个数字，即一个数据。要求在输出中每行有两个间隔的数字，其中，第一个代表原始数据在数据集排序中的位次，第二个代表原始数据。

```
sort1.txt:
34
6543
12
-45
58
753
234
858
```

```
sort2.txt:
34
675
349
648
75
39
-7
```

```
sort3.txt:
34
76
236
2387
-497
45
34
```


## 程序代码

### SortMapper

```java
package com.test.sort;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class SortMapper extends Mapper<LongWritable, Text, IntWritable, IntWritable>{

	private static IntWritable data = new IntWritable();
	private static IntWritable one = new IntWritable(1);
	
	@Override
	protected void map(LongWritable key, Text value,Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		data.set(Integer.parseInt(line));
		context.write(data, one);
	}
}

```

### SortReducer

```java
package com.test.sort;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Reducer;

public class SortReducer extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable>{

	private static IntWritable lineNum = new IntWritable(1);
	
	@Override
	protected void reduce(IntWritable key, Iterable<IntWritable> values,Context context)
			throws IOException, InterruptedException {
		for (IntWritable val : values) {
			context.write(lineNum, key);
			lineNum = new IntWritable(lineNum.get() + 1);
		}
	}

}

```

### SortRunner

```java
package com.test.sort;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class SortRunner extends Configured implements Tool{

	@Override
	public int run(String[] args) throws Exception {
		
		Configuration  conf = new Configuration();
		Job job = Job.getInstance(conf, "Simple Sort");
		job.setJarByClass(SortRunner.class);
		
		job.setMapperClass(SortMapper.class);
		job.setReducerClass(SortReducer.class);
		
		job.setMapOutputKeyClass(IntWritable.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(IntWritable.class);
		job.setOutputValueClass(IntWritable.class);
		
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		return job.waitForCompletion(true) ? 0:1;
	}

	public static void main(String[] args) throws Exception {
		int res = ToolRunner.run(new Configuration(), new SortRunner(), args);
		System.exit(res);
	}
}

```

## 运行结果

```
1	-497
2	-45
3	-7
4	12
5	34
6	34
7	34
8	34
9	39
10	45
11	58
12	75
13	76
14	234
15	236
16	349
17	648
18	675
19	753
20	858
21	2387
22	6543

```



