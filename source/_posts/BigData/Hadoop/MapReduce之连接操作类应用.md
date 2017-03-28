---
title: MapReduce之连接操作类应用
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 用MapReduce实现关系的自然连接

<center>![join](http://oni2hc3a8.bkt.clouddn.com/bigdatajoin.png)</center>

- 假设有关系R(A，B)和S(B,C)，对二者进行自然连接操作
- 使用Map过程，把来自R的每个元组`<a,b>`转换成一个键值对`<b, <R,a>>`，其中的键就是属性B的值。把关系R包含到值中，这样做使得我们可以在Reduce阶段，只把那些来自R的元组和来自S的元组进行匹配。类似地，使用Map过程，把来自S的每个元组`<b,c>`，转换成一个键值对`<b,<S,c>>`
- 所有具有相同B值的元组被发送到同一个Reduce进程中，Reduce进程的任务是，把来自关系R和S的、具有相同属性B值的元组进行合并
- Reduce进程的输出则是连接后的元组<a,b,c>，输出被写到一个单独的输出文件中


## 自然连接过程

<center>![joinProcess](http://oni2hc3a8.bkt.clouddn.com/bigdatajoinProcess.png)</center>


## 应用示例

在HDFS中有两个文件，一个记录了学生的基本信息，包含了姓名和学号信息，名为student_info.txt,内容为：

```
Jenny	00001
Hardy	00002
Bradley	00003
```

还有一个文件记录了学生的选课信息表，包括了学号和课程名，名为student_class_info.txt,内容为：

```
00001	Chinese
00001	Math
00002	Music
00002	Math
00003	Physic
```

现在经join操作后，得出的结果为：

```
Jenny	Chinese
Jenny	Math
Hardy	Music
Hardy	Math
Bradley	Physic
```

## 程序代码

### JoinMapper

```java
package com.test.join;

import java.io.IOException;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class JoinMapper extends Mapper<LongWritable, Text, Text, Text>{
	private static final String STUDENT_FILENAME = "student_info.txt";
	private	static final String STUDENT_CLASS_FILENAME = "student_class_info.txt";
	private	static final String STUDENT_FLAG = "student";
	private	static final String STUDENT_CLASS_FLAG = "student_class";
	
	private FileSplit fileSplit;
	private Text outKey = new Text();
	private Text outValue = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		fileSplit = (FileSplit) context.getInputSplit();
		String filePath = fileSplit.getPath().toString();
		
		String line = value.toString();
		String[] fields = StringUtils.split(line,"\t");
		
		//判断记录来自哪个文件
		if (filePath.contains(STUDENT_FILENAME)) {
			outKey.set(fields[1]);
			outValue.set(STUDENT_FLAG + "\t" + fields[0]);
		}
		else if (filePath.contains(STUDENT_CLASS_FILENAME)) {
			outKey.set(fields[0]);
			outValue.set(STUDENT_CLASS_FLAG + "\t" + fields[1]);
		}
		
		context.write(outKey, outValue);
	}
}

```

### JoinReducer

```java
package com.test.join;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class JoinReducer extends Reducer<Text, Text, Text, Text>{
	private	static final String STUDENT_FLAG = "student";
	private	static final String STUDENT_CLASS_FLAG = "student_class";
	
	private String fileFlag = null;
	private String stuName = null;
	private List<String> stuClassNames;
	
	private Text outKey = new Text();
	private Text outValue = new Text();

	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context)
			throws IOException, InterruptedException {
		stuClassNames = new ArrayList<>();
		
		for (Text val : values) {
			String[] fields = StringUtils.split(val.toString(),"\t");
			fileFlag = fields[0];
			//判断记录来自哪个文件，并根据文件格式解析记录。
			if (fileFlag.equals(STUDENT_FLAG)) {
				stuName = fields[1];
				outKey.set(stuName);
			}
			else if (fileFlag.equals(STUDENT_CLASS_FLAG)) {
				stuClassNames.add(fields[1]);
			}
		}
		
		//求笛卡尔积
		for (String stuClassName : stuClassNames) {
			outValue.set(stuClassName);
			context.write(outKey, outValue);
		}
	}

}

```

### JoinRunner

```java
package com.test.join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JoinRunner extends Configured implements Tool{

	@Override
	public int run(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf, "Join");
		job.setJarByClass(JoinRunner.class);
		
		job.setMapperClass(JoinMapper.class);
		job.setReducerClass(JoinReducer.class);
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		return job.waitForCompletion(true) ? 0:1;
	}
	
	public static void main(String[] args) throws Exception {
		int res = ToolRunner.run(new Configuration(), new JoinRunner(), args);
		System.exit(res);
	}
}

```

### 运行结果

```
Jenny	Math
Jenny	Chinese
Hardy	Math
Hardy	Music
Bradley	Physic

```
