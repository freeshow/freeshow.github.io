---
title: MapReduce之二次排序类应用
date: 2017-03-27 12:22:51
tags: Hadoop 
categories: 大数据
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## 应用需求

在某些应用场合中，需要对数据文件中的大量记录某个属性进行排序，可是这个属性的记录太多，需要根据其他属性在排序。这种应用称为“二次排序”。

## 应用场景

在对大数据进行分析时，常采用排序的方式，排序后，发现数据量太大，具有相同关键值的记录也非常多，这是，就需要对第二属性进行排序。

## 解决方案

默认情况下，Map 输出的结果会对 Key 进行默认排序，但是“二次排序”中除了对 Key 进行排序外，还需要对位于 value 值中的另外一个属性进行排序，而 MapReduce 框架并没有提供对 value 值进行排序的方法。怎么实现对 value 的排序呢？ 这就需要变通的去实现这个需求。

变通手段：

可以把 key 和 value 联合起来作为新的 Key,记作 Newkey。这时，NewKey含有两个字段，假设分别为k,v.这里的k和v是原来的 key 和 value。原来的value还是不变。这样，value就同时在NewKey和value的位置。再实现NewKey的比较规则，先按照key排序，在key相同的基础上再按照value排序。在分组时，在按照原来的key进行分组，就不会影响原有的分组逻辑了。最后在输出时，只把原有的key、value输出，就可以变通的实现了二次排序的需求。

## 应用案例

现有一输入文件，包含两列数据，要求先按照第一列整数大小排序，如果第一列相同，按照第二列整数大小排序。

```
secondrysort.txt:
20 21		70 56		60 56
50 51		70 57		60 57
50 52		70 58		740 58
50 53		5  6		63 61
50 54		7  82		730 54
60 51		203 21		71 55
60 53		50 512		71 56
60 52		50 522		73 57
60 56		50 53		74 58
60 57		530 54		12 211
70 58		40 511		31 42
60 61		20 53		50 62
70 54		20 522		7  8
70 55
```

## 程序代码

### SecondrySortPair

```java
package com.test.secondrysort;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

/**
 * 创建主键类 SecondrySortPair,把第一列整数和第二列整数作为类的属性。
 * @author hadoop
 *
 */
public class SecondrySortPair implements WritableComparable<SecondrySortPair>{
	private int first = 0;
	private int second = 0;

	public void set(int first, int second) {
		this.first = first;
		this.second = second;
	}
	
	public int getFirst() {
		return first;
	}

	public int getSecond() {
		return second;
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		first = in.readInt();
		second = in.readInt();
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeInt(first);
		out.writeInt(second);
	}

	//这里的代码是关键，因为对key排序时，调用的就是这个compareTo方法
	@Override
	public int compareTo(SecondrySortPair o) {
		if (first != o.first) {
			return first - o.first;
		}
		else if (second != o.second) {
			return second - o.second;
		}
		else {
			return 0;
		}
	}
	
	@Override
	public boolean equals(Object obj) {
		if (obj instanceof SecondrySortPair) {
			SecondrySortPair o = (SecondrySortPair) obj;
			return first == o.first && second == o.second;
		}
		else {
			return false;
		}
	}
	
	@Override
	public int hashCode() {
		return first + "".hashCode() + second + "".hashCode();
	}
}

```

### GroupingComparator

```java
package com.test.secondrysort;

import org.apache.hadoop.io.RawComparator;
import org.apache.hadoop.io.WritableComparator;

/**
 * 在分组比较时，只比较原来的key,而不是组合key
 * @author hadoop
 *
 */
public class GroupingComparator implements RawComparator<SecondrySortPair>{

	@Override
	public int compare(SecondrySortPair o1, SecondrySortPair o2) {
		int first1 = o1.getFirst();
		int first2 = o2.getFirst();
		return first1 - first2;
	}

	@Override
	public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {
		return WritableComparator.compareBytes(b1, s1, Integer.SIZE/8, b2, s2, Integer.SIZE/8);
	}

}

```

### SecondarySortMapper

```java
package com.test.secondrysort;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class SecondrySortMapper extends Mapper<LongWritable, Text, SecondrySortPair, IntWritable>{
	private SecondrySortPair newKey = new SecondrySortPair();
	private IntWritable outValue = new IntWritable();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		
		int first = 0, second = 0;
		String line = value.toString();
		StringTokenizer itr = new StringTokenizer(line);
		while(itr.hasMoreTokens()){
			first = Integer.parseInt(itr.nextToken());
			if (itr.hasMoreTokens()) {
				second = Integer.parseInt(itr.nextToken());
			}
			newKey.set(first, second);
			outValue.set(second);
			context.write(newKey, outValue);
		}
	}

}

```

### SecondarySortReducer

```java
package com.test.secondrysort;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class SecondrySortReducer extends Reducer<SecondrySortPair, IntWritable, Text, IntWritable>{
	private static final Text SEPARATOR = new Text("----------");
	private Text outKey = new Text();

	@Override
	protected void reduce(SecondrySortPair key, Iterable<IntWritable> values, Context context)
			throws IOException, InterruptedException {
		context.write(SEPARATOR, null);
		outKey.set(Integer.toString(key.getFirst()));
		for (IntWritable val : values) {
			context.write(outKey, val);
		}
	}	
}

```

### SecondarySortRunner

```java
package com.test.secondrysort;

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

public class SecondrySortRunner extends Configured implements Tool{

	@Override
	public int run(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf, "SecondrySort");
		job.setJarByClass(SecondrySortRunner.class);
		
		job.setMapperClass(SecondrySortMapper.class);
		job.setReducerClass(SecondrySortReducer.class);
		
		//设置分组函数类，对二次排序非常关键。
		job.setGroupingComparatorClass(GroupingComparator.class);
		
		//设置Map的输出 key value类，对二次排序非常重要。
		job.setMapOutputKeyClass(SecondrySortPair.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		return job.waitForCompletion(true) ? 0:1;
	}

	public static void main(String[] args) throws Exception {
		int res = ToolRunner.run(new Configuration(), new SecondrySortRunner(), args);
		System.exit(res);
	}
}

```

### 运行结果

```
----------
5	6
----------
7	8
7	82
----------
12	211
----------
20	21
20	53
20	522
----------
31	42
----------
40	511
----------
50	51
50	52
50	53
50	53
50	54
50	62
50	512
50	522
----------
60	51
60	52
60	53
60	56
60	56
60	57
60	57
60	61
----------
63	61
----------
70	54
70	55
70	56
70	57
70	58
70	58
----------
71	55
71	56
----------
73	57
----------
74	58
----------
203	21
----------
530	54
----------
730	54
----------
740	58

```

## 程序分析

先对现在第一列和第二列整数创建一个新的类，作为NewKey，这里的NewKey类型为SecondrySortPair,对NewKey的比较有两种方法。
- 在Map阶段的最后，会先调用job.setPartitionerClass 对输出的List进行分区，每个分区映射到一个Reducer，每个分区又调用job.setSortComparatorClass设置Key比较函数类进行排序。
- 如果没有通过job.setSortComparatorClass设置Key比较类，则使用Key实现的 compareTo方法排序，本例代码就使用了 compareTo方法排序。

在Reduce阶段，Reduce接收到所有映射到这个Reduce的Map输出后，也会调用job.setSortComparatorClass设置的Key比较函数类对所有数据进行排序。<font color=red>**然后开始构建一个Key对应的Value迭代器。这是就要用到分组，使用job.setGroupingComparatorClass设置的分组函数。只要这个比较器比较的两个Key相同，它们就属于同一个组，它们的Value就在一个Value迭代器**</font>，而这个迭代器的Key使用属于同一组的所有Key的第一个Key。

最后就是进入Reducer的reduce方法，reduce方法的输入是所有的Key和它的Value迭代器。


## 不添加分组的结果

```
----------
5	6
----------
7	8
----------
7	82
----------
12	211
----------
20	21
----------
20	53
----------
20	522
----------
31	42
----------
40	511
----------
50	51
----------
50	52
----------
50	53
50	53
----------
50	54
----------
50	62
----------
50	512
----------
50	522
----------
60	51
----------
60	52
----------
60	53
----------
60	56
60	56
----------
60	57
60	57
----------
60	61
----------
63	61
----------
70	54
----------
70	55
----------
70	56
----------
70	57
----------
70	58
70	58
----------
71	55
----------
71	56
----------
73	57
----------
74	58
----------
203	21
----------
530	54
----------
730	54
----------
740	58

```
