---
title: HDFS Java API
toc: false
date: 2017-03-27 12:22:51
tags:
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>


- 通常MapReduce会把一个文件数据块处理成一个Map任务。
- HDFS默认工作目录为/user/${USER},${USER}是当前的登录用户名。


## HDFS中的 Java API 的使用

- 文件在 Hadoop 中表示一个Path对象，通常封装一个URI，如HDFS上有个test文件，URI表示成hdfs://master:9000/test。
- Hadoop 中关于文件操作类基本上全部是在"org.apache.hadoop.fs"包中，这些 API 能够支持的操作包含打开文件、读写文件、删除文件等。


Hadoop 类库中最终面向用户提供的接口类是 FileSystem，该类是个抽象类，只能通过类的 get 方法得到具体的类。get 方法存在几个重载版本，常用的是 static FileSystem get(Configuration conf); 该类封装了几乎所有的文件操作，如 mkdir、delete等。综上基本上可以得出操作文件的程序库框架：
```java
operator()
{
	得到Configuration对象
	得到FileSystem对象
	进行文件操作
}

```

### 上传文件

通过 “FileSystem.copyFromLocalFile(Path src,Path dst)” 可将本地文件上传到HDFS指定的位置上，其中 src 和 dst 均为文件完整路径，具体示例如下。

```java 
package com.test.hdfs;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

//文件上传至HDFS
public class PutFile {

	public static void main(String[] args) throws URISyntaxException, IOException {
		// TODO Auto-generated method stub
		Configuration con = new Configuration();
		
		URI uri = new URI("hdfs://master:9000");
		FileSystem fs = FileSystem.get(uri, con);

		//本地文件
		Path src = new Path("D:\\test.txt");
		//HDFS存放文件
		Path dst = new Path("/");
		//上传文件
		fs.copyFromLocalFile(src, dst);
		System.out.println("Upload to "+con.get("fs.defaultFS"));
                                   
		//以下相当于执行hdfs dfs -ls /
		FileStatus[] files = fs.listStatus(dst);
		for (FileStatus file : files) {
			System.out.println(file.getPath());
		}
	}

}

```

### 新建文件

通过 "FileSystem.create(Path f, Boolean b)" 可在 HDFS 上创建文件，其中 f 为文件的完整路径， b 为判断是否覆盖，具体实现如下。

```java 
package com.test.hdfs;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class CreateFile {

	public static void main(String[] args) throws URISyntaxException, IOException {
		// TODO Auto-generated method stub
		Configuration conf = new Configuration();
		URI uri = new URI("hdfs://master:9000");
		FileSystem fs = FileSystem.get(uri, conf);
		
		//定义新文件
		Path dfs = new Path("/hdfsfile");
		//创建新文件，如果有则覆盖（true）
		FSDataOutputStream create = fs.create(dfs, true);
		//创建目录为：fs.mkdirs()

		//向新创建的文件中写入数据
		create.writeBytes("Hello,HDFS!");
	}

}

```

### 查看文件详细信息

通过 “Class FileStatus” 可查找指定文件在 HDFS 集群上的具体信息，包括文件路径、访问时间、修改时间、文件长度、所占块大小、文件拥有者、文件用户组和文件复制数等信息，具体实现如下。

```java
package com.test.hdfs;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.BlockLocation;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class FileLocation {

	public static void main(String[] args) throws URISyntaxException, IOException {
		// TODO Auto-generated method stub
		Configuration conf = new Configuration();
		URI uri = new URI("hdfs://master:9000");
		FileSystem fs = FileSystem.get(uri, conf);
		
		Path fPath = new Path("/hdfsfile");
		FileStatus fileStatus = fs.getFileStatus(fPath);
		
		/*获取文件在  HDFS 集群位置
		 *FileSystem.getFileBlockLocations(FileStatus file, long start, long len)
		 *可查找指定文件在 HDFS 集群上的位置 ，其中 file 为文件的完整路径， start 和 len 来标识查找文件的路径
		 * */
		BlockLocation[] blockLocations = fs.getFileBlockLocations(fileStatus, 0, 
				fileStatus.getLen());
		for (int i = 0; i < blockLocations.length; i++) {
			String[] hosts = blockLocations[i].getHosts();
			System.out.println("block_"+i+"_location:"+hosts[0]);
		}
		
		//格式化日期输出
		SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		
		//获取文件访问时间，返回long
		long accessTime = fileStatus.getAccessTime();
		System.out.println("access:"+formatter.format(new Date(accessTime)));
		
		//获取文件修改时间，返回long
		long modificationTime = fileStatus.getModificationTime();
		System.out.println("modification:"+formatter.format(new Date(modificationTime)));
		
		//获取块大小，单位B
		long blockSize = fileStatus.getBlockSize();
		System.out.println("blockSize:"+blockSize);
		
		//获取文件大小，单位B
		long len = fileStatus.getLen();
		System.out.println("length:"+len);
		
		//获取文件拥有者
		String ower = fileStatus.getOwner();
		System.out.println("owner:"+ower);
		
		//获取文件所在用户组
		String group = fileStatus.getGroup();
		System.out.println("group:"+group);
		
		//获取文件拷贝数
		short replication = fileStatus.getReplication();
		System.out.println("replication:"+replication);
		
	}

}

```

### 下载文件

从 HDFS 下载文件到本地非常简单，直接调用 FileSystem.copyToLocalFile(Path src, Path dst)即可。其中 src 为 HDFS 上的文件， dst 为要下载到本地的文件名，示例如下。

```java
package com.test.hdfs;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class GetFile {

	public static void main(String[] args) throws URISyntaxException, IOException {
		// TODO Auto-generated method stub
		Configuration conf = new Configuration();
		URI uri = new URI("hdfs://master:9000");
		FileSystem fs = FileSystem.get(uri, conf);
		
		//hdfs上的文件
		Path src = new Path("/file");
		//下载到本地的文件名
		Path dst = new Path("F:/newfile");
		//下载文件
		fs.copyToLocalFile(src, dst);
	}

}

```

结果中会出现一个crc文件，里面保存了对 file 文件的循环校验信息，如下图所示。

<center>![](http://i.imgur.com/SzawpvH.png)</center>


### 删除文件

从　HDFS 上删除文件非常简单，直接调用 FileSystem.delete(Path path, Boolean b)即可。其中 path 为要删除的文件，示例如下。

```java
package com.test.hdfs;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class DeleteFile {

	public static void main(String[] args) throws URISyntaxException, IOException {
		// TODO Auto-generated method stub
		Configuration conf = new Configuration();
		URI uri = new URI("hdfs://master:9000");
		FileSystem fs = FileSystem.get(uri, conf);
		
		//HDFS上删除的文件
		Path delPath = new Path("/file");
		if (fs.exists(delPath)) {
			fs.delete(delPath);
			System.out.println(delPath+" has been deleted sucessfully.");
		}
		else {
			System.out.println(delPath + " deleted failed.");
		}
	}

}

```

### java.net.URL

```java
package com.test.hdfs;

import java.io.IOException;
import java.io.InputStream;
import java.net.MalformedURLException;
import java.net.URL;

import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.apache.hadoop.io.IOUtils;

/*该程序是从HDSF中读取文件最简单的方式，
 * 即用java.net.URL对象打开数据流。
 * */
public class DescURL {

	//让Java程序识别Hadoop的HDFS url
	static{
		URL.setURLStreamHandlerFactory(
				new FsUrlStreamHandlerFactory());
	}
	
	public static void main(String[] args) 
			throws MalformedURLException, IOException {
		InputStream in = null;
		try {
			in = new URL(args[0]).openStream();
			IOUtils.copyBytes(in, System.out, 4096,false);
		} finally {
			IOUtils.closeStream(in);
		}
	}

}

```


### 示例

```java
package cn.itcast.hadoop.hdfs;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.URI;

import org.apache.commons.io.IOUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.LocatedFileStatus;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.RemoteIterator;
import org.junit.Before;
import org.junit.Test;

public class HdfsUtil {
	
	FileSystem fs = null;

	
	@Before
	public void init() throws Exception{
		
		//读取classpath下的xxx-site.xml 配置文件，并解析其内容，封装到conf对象中
		Configuration conf = new Configuration();
		
		//也可以在代码中对conf中的配置信息进行手动设置，会覆盖掉配置文件中的读取的值
		conf.set("fs.defaultFS", "hdfs://weekend110:9000/");
		
		//根据配置信息，去获取一个具体文件系统的客户端操作实例对象
		fs = FileSystem.get(new URI("hdfs://weekend110:9000/"),conf,"hadoop");
		
		
	}
	
	
	
	/**
	 * 上传文件，比较底层的写法
	 * 
	 * @throws Exception
	 */
	@Test
	public void upload() throws Exception {

		Configuration conf = new Configuration();
		conf.set("fs.defaultFS", "hdfs://weekend110:9000/");
		
		FileSystem fs = FileSystem.get(conf);
		
		Path dst = new Path("hdfs://weekend110:9000/aa/qingshu.txt");
		
		FSDataOutputStream os = fs.create(dst);
		
		FileInputStream is = new FileInputStream("c:/qingshu.txt");
		
		IOUtils.copy(is, os);
		

	}

	/**
	 * 上传文件，封装好的写法
	 * @throws Exception
	 * @throws IOException
	 */
	@Test
	public void upload2() throws Exception, IOException{
		
		fs.copyFromLocalFile(new Path("c:/qingshu.txt"), new Path("hdfs://weekend110:9000/aaa/bbb/ccc/qingshu2.txt"));
		
	}
	
	
	/**
	 * 下载文件
	 * @throws Exception 
	 * @throws IllegalArgumentException 
	 */
	@Test
	public void download() throws Exception {
		
		fs.copyToLocalFile(new Path("hdfs://weekend110:9000/aa/qingshu2.txt"), new Path("c:/qingshu2.txt"));

	}

	/**
	 * 查看文件信息
	 * @throws IOException 
	 * @throws IllegalArgumentException 
	 * @throws FileNotFoundException 
	 * 
	 */
	@Test
	public void listFiles() throws FileNotFoundException, IllegalArgumentException, IOException {

		// listFiles列出的是文件信息，而且提供递归遍历
		RemoteIterator<LocatedFileStatus> files = fs.listFiles(new Path("/"), true);
		
		while(files.hasNext()){
			
			LocatedFileStatus file = files.next();
			Path filePath = file.getPath();
			String fileName = filePath.getName();
			System.out.println(fileName);
			
		}
		
		System.out.println("---------------------------------");
		
		//listStatus 可以列出文件和文件夹的信息，但是不提供自带的递归遍历
		FileStatus[] listStatus = fs.listStatus(new Path("/"));
		for(FileStatus status: listStatus){
			
			String name = status.getPath().getName();
			System.out.println(name + (status.isDirectory()?" is dir":" is file"));
			
		}
		
	}

	/**
	 * 创建文件夹
	 * @throws Exception 
	 * @throws IllegalArgumentException 
	 */
	@Test
	public void mkdir() throws IllegalArgumentException, Exception {

		fs.mkdirs(new Path("/aaa/bbb/ccc"));
		
		
	}

	/**
	 * 删除文件或文件夹
	 * @throws IOException 
	 * @throws IllegalArgumentException 
	 */
	@Test
	public void rm() throws IllegalArgumentException, IOException {

		fs.delete(new Path("/aa"), true);
		
	}

	
	public static void main(String[] args) throws Exception {

		Configuration conf = new Configuration();
		conf.set("fs.defaultFS", "hdfs://weekend110:9000/");
		
		FileSystem fs = FileSystem.get(conf);
		
		FSDataInputStream is = fs.open(new Path("/jdk-7u65-linux-i586.tar.gz"));
		
		FileOutputStream os = new FileOutputStream("c:/jdk7.tgz");
		
		IOUtils.copy(is, os);
	}
	
}

```
