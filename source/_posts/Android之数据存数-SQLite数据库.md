---
title: Android之数据存数--SQLite数据库
date: 2016-07-23 23:43:58
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载自http://liangruijun.blog.51cto.com/3061169/663686
《疯狂Android讲义》

## 一.SQLite的介绍 ##

 1. SQLite简介

	SQLite是一款轻型的数据库，是遵守ACID的关联式数据库管理系统，它的设计目标是嵌入 式的，而且目前已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。它能够支持 Windows/Linux/Unix等等主流的操作系统，同时能够跟很多程序语言相结合，比如Tcl、PHP、Java、C++、.Net等，还有ODBC接口，同样比起 Mysql、PostgreSQL这两款开源世界著名的数据库管理系统来讲，它的处理速度比他们都快。

	Android系统集成了一个轻量级的数据库：SQLite。SQLite只是一个嵌入式的数据库引擎，专门适用于资源有限的设备(如手机、PDA等)上适量数据存取。

	虽然SQL支持绝大部分SQL 92语法，也允许开发者使用SQL语句操作数据库中的数据，但SQLite并不像Oracle、MySQL数据库那样需要安装、启动服务器进程，SQLite数据库只是一个文件。

 2. SQLite的特点：
- 轻量级
SQLite和C/S模式的数据库软件不同，它是进程内的数据库引擎，因此不存在数据库的客户端和服务器。使用SQLite一般只需要带上它的一个动态  库，就可以享受它的全部功能。而且那个动态库的尺寸也挺小，以版本3.6.11为例，Windows下487KB、Linux下347KB。

- 不需要"安装"
SQLite的核心引擎本身不依赖第三方的软件，使用它也不需要"安装"。有点类似那种绿色软件。

- 单一文件
数据库中所有的信息（比如表、视图等）都包含在一个文件内。这个文件可以自由复制到其它目录或其它机器上。

- 跨平台/可移植性
除了主流操作系统 windows，linux之后，SQLite还支持其它一些不常用的操作系统。

- 弱类型的字段
同一列中的数据可以是不同类型

- 开源

 3. SQLite数据类型

一般数据采用的固定的静态数据类型，而SQLite采用的是动态数据类型，会根据存入值自动判断(即它允许把各种类型的数据保存到任意类型字段中)。SQLite具有以下五种常用的数据类型：

- NULL: 这个值为空值
- VARCHAR(n)：长度不固定且其最大长度为 n 的字串，n不能超过 4000。
- CHAR(n)：长度固定为n的字串，n不能超过 254。
- INTEGER: 值被标识为整数,依据值的大小可以依次被存储为1,2,3,4,5,6,7,8.
- REAL(浮点数): 所有值都是浮动的数值,被存储为8字节的IEEE浮动标记序号.
- TEXT:(文本) 值为文本字符串,使用数据库编码存储(TUTF-8, UTF-16BE or UTF-16-LE).
- BLOB(大二进制对象): 值是BLOB数据块，以输入的数据格式进行存储。如何输入就如何存储,不改  变格式。
- DATA ：包含了 年份、月份、日期。
- TIME： 包含了 小时、分钟、秒。



----------
## 二.SQLiteDatabase的介绍 ##

Android提供了创建和是用SQLite数据库的API。SQLiteDatabase代表一个数据库对象，提供了操作数据库的一些方法。在Android的SDK目录下有sqlite3工具，我们可以利用它创建数据库、创建表和执行一些SQL语句。下面是SQLiteDatabase的常用方法。

| 方法名称 | 方法描述 |
| ------------------------------------------------------------------------------ |------------------------------------------------------------| 
| openOrCreateDatabase(String path,SQLiteDatabase.CursorFactory factory)| 打开或创建数据库| 
|insert(String table,String nullColumnHack,ContentValues values) | 添加一条记录 | 
| delete(String table,String whereClause,String[] whereArgs) | 删除一条记录 | 
| update(String table,ContentValues values,String whereClause,String[] whereArgs) | 修改记录 |
| query(String table,String[] columns,String selection,String[] selectionArgs,String groupBy,String having,String orderBy) | 查询一条记录 |
| execSQL(String sql) | 执行一条SQL语句 |
| beginTransaction() | 开始事务 |
| endTransaction() | 结束事务 |
| close() | 关闭事务 |

***1.*** ***打开或者创建数据库***

在Android 中以使用SQLiteDatabase的静态方法openOrCreateDatabase(String path,SQLiteDatabae.CursorFactory factory)打开或者创建一个数据库。它会自动去检测是否存在这个数据库，如果存在则打开，不存在则创建一个数据库；创建成功则返回一个SQLiteDatabase对象，否则抛出异常FileNotFoundException。

下面是创建名为“stu.db”数据库的代码：

```
SQLiteDatabase db;
db = SQLiteDatabase.openOrCreateDatabase(this.getFilesDir().toString()+"/stu.db3",null);
```
上面代码没有指定SQLiteDatabase.CursorFactory参数，该参数是一个用于返回Cursor的工厂，如果指定该参数为null,则意味着使用默认的工厂。

上面代码即可返回一个SQLiteDatabase对象，该对象的execSQL()方法可以执行任意的SQL语句。

***2.*** ***创建表***

创建一张表很简单。首先，编写创建表的SQL语句，然后，调用SQLiteDatabase的execSQL()方法来执行SQL语句便可以创建一张表了。

下面的代码创建了一张学生表，属性列为：id（主键并且自动增加）、sname（学生姓名）、snumber（学号）

```
 private void createTable(SQLiteDatabase db) {
        String stu_table = "create table StuTable(id integer primary key autoincrement,sname text,snumber text)";
        db.execSQL(stu_table);
    }
```


***3.插入数据***

从上面SQLiteDatabase常用的API中可以看出，SQLiteDatabase提供了对数据库操作的的insert(增)、delete(删)、update(改)、query(查)的方法，其实这些方法完全可以通过执行SQL语句来完成，但Android考虑到部分开发者对SQL语法不熟悉，所以提供这种方法帮助开发者以更简单的方式来操作数据库表中的数据。

因此，SQLiteDatabase对数据库的增、删、改、查等操作，有两种方法可已完成。
第一种是：使用SQLiteDatabase提供的insert、delete、update、query方法操作数据库。
第二种是：使用SQL语句操作SQLite数据库。

第一种方法：
SQLiteDatabase的insert(String table,String nullColumnHack,ContentValues values)方法，参数一是表名称，参数二是空列的默认值，参数三是ContentValues类型的一个封装了列名称和列值的Map；

```
private void insertDB(SQLiteDatabase db,String name,String number) 
{
		//实例化常量值
        ContentValues values = new ContentValues();
        //插入姓名
        values.put("sname",name);
        //插入学号
        values.put("snumber",number);
        //调用insert插入数据
        db.insert("stuTable", null, values);
}

insertDB(db, "xiaoming", "00001");
insertDB(db, "xiaohong", "00002");
insertDB(db, "xiaoqiang", "00003");
```

第二种方法：

```
private void insert(SQLiteDatabase db)
{   
 
     //插入数据SQL语句  
     String stu_sql="insert into stu_table(sname,snumber) values('xiaoming','00001')";  
   
    //执行SQL语句  
     db.execSQL(sql);  
}  
```

***4.删除数据***

第一种方法：
调用SQLiteDatabase的delete(String table,String whereClause,String[] whereArgs)方法，参数一是表名称，参数二是删除条件，参数三是删除条件值数组；

```
private void deleteDB(SQLiteDatabase db)
 {
		//删除条件
        String whereCause = "sname=?";
        //删除条件参数
        String[] whereArgs = {"xiaoming"};
        //执行删除
        db.delete("stuTable",whereCause,whereArgs);
 }
```

第二种方法：

```
private void deleteDB(SQLiteDatabase db)
 {  
   
   //删除SQL语句  
   String sql = "delete from stuTable where sname = xiaoming";  
   
   //执行SQL语句  
   db.execSQL(sql);  
}
```

***5.修改数据***

第一种方法：

调用SQLiteDatabase的update(String table,ContentValues values,String whereClause, String[] whereArgs)方法。参数是表名称，参数是更行列ContentValues类型的键值对（Map），参数是更新条件（where字句），参数是更新条件数组。

```
 private void updateDB(SQLiteDatabase db)
 {
     ContentValues values = new ContentValues();
     values.put("snumber","10010");
     String whereCause = "sname=?";
     String[] whereArgs = {"xiaohong"};
     db.update("stuTable",values,whereCause,whereArgs);
 }
```

第二种方法：

```
private void updateDB(SQLiteDatabase db)
{  
   
    //修改SQL语句  
    String sql = "update stuTable set snumber = 10010 where sname = xiaohong";  
   
    //执行SQL  
    db.execSQL(sql);   
}
```

***6.查询数据***

在Android中查询数据是通过Cursor类来实现的，当我们使用SQLiteDatabase.query()方法时，会得到一个Cursor对象，Cursor指向的就是每一条数据。它提供了很多有关查询的方法，具体方法如下：
public Cursor query(String table,String[] columns,String selection,String[] selectionArgs,String groupBy,String having,String orderBy,String limit);
各个参数的意义说明：
①table:表名称
②columns:列名称数组
③selection:条件字句，相当于where
④selectionArgs:条件字句，参数数组
⑤groupBy:分组列
⑥having:分组条件
⑦orderBy:排序列
⑧limit:分页查询限制
⑨Cursor:返回值，相当于结果集ResultSet

Cursor是一个游标接口，提供了遍历查询结果的方法，如移动指针方法move()，获得列值方法getString()等.
Cursor游标常用方法:
| 方法名称 | 方法描述 | 
| ------------- |:-------------:| 
| getCount() | 获得总的数据项数 | 
| isFirst() | 判断是否第一条记录 | 
| isLast() | 判断是否最后一条记录 |
|moveToFirst()|移动到第一条记录|
|moveToLast()|移动到最后一条记录|
|moveToPrevious()|移动到上一条记录|
|moveToNext()|移动到下一条记录|
|move(int offset)|将记录指针移动指定的行数，offset为正就向下移动，offset为负就向上移动。|
|getColumnIndexOrThrow(String columnName)|根据列名称获得列索引|
|getInt(int columnIndex)|获得指定列索引的int类型值|
|getString(int columnIndex)|获得指定列索引的String类型值|

第一种方法：

```
private void queryDB(SQLiteDatabase db)
    {
        Cursor cursor = db.query("stuTable",null,null,null,null,null,null);
        if (cursor.moveToFirst())
        {
            for (int i = 0;i < cursor.getCount();i++)
            {
                cursor.move(i);
                int id = cursor.getInt(0);
                String name = cursor.getString(1);
                String number = cursor.getString(2);
                System.out.println(id+":"+name+":"+number);
            }
        }
    }
```

第二种方法：
SQLiteDatabase的execSQL()方法可执行任意的SQL语句，包括带占位符的SQL语句。但由于该方法没有返回值，因此一般用于执行DDL语句或DML语句；如果需要执行查询语句，则可调用SQLiteDatabase的rawQuery(String sql,String[] selectionArgs)方法。

***7.删除指定表***

编写插入数据的SQL语句，直接调用SQLiteDatabase的execSQL()方法来执行
```
private void dropTable(SQLiteDatabase db) 
{
        String sql = "DROP TABLE stuTable";
        db.execSQL(sql);
}

```

***提示：***虽然Android提供了这些所谓的“便捷”方法来操作SQLite数据库，但在笔者看来这些方法纯属“鸡肋”，对于一个程序员而言，SQL语法可以说是基本功中的基本功---你见过不会1+1=2的数学工作者吗？

***8.事务***
SQLiteDatabase中包含如下两个方法来控制事务。

 - beginTransaction():开始事务
 - endTransaction():结束事务

除此之外，SQLiteDatabase还提供了如下方法来判断当前上下文是否处于事务环境中。

 - inTransaction():如果当前上下文处于事务中，则返回true;否则返回false。

当程序执行endTransaction()方法时将会结束事务——到底是提交事务，还是回滚事务呢？这取决于SQLiteDatabase是否调用了setTransactionSuccessful()方法来设置事务标志，如果程序在事务执行中调用该方法设置了事务成功则提交事务；否则程序将会回滚事务。

```
//开始事务
db.beginTrainsaction();
try
{
	//指定DML语句
	......
	//调用该方法设置事务成功；否则endTransaction()方法将回滚事务。
	db.setTransactionSuccessful();
}
finally
{
	//由事务的标志决定是提交事务还是回滚事务
	db.endTransaction();
}
```

Demo:
该程序提供两个文本框，用户可以在这两个文本框中输入内容，当用户点击“插入”按钮时这两个文本框的内容将会被插入数据库。

效果：
![这里写图片描述](http://img.blog.csdn.net/20150914092224095)

代码：
**main.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
<EditText 
	android:id="@+id/title"
	android:layout_width="match_parent" 
	android:layout_height="wrap_content" 
	/>
<EditText  
	android:id="@+id/content"
	android:layout_width="match_parent" 
	android:layout_height="wrap_content" 
	android:lines="2"
	/>	
<Button  
	android:id="@+id/ok"
	android:layout_width="wrap_content" 
	android:layout_height="wrap_content" 
	android:text="@string/insert"
	/>
<ListView  
	android:id="@+id/show"
	android:layout_width="match_parent" 
	android:layout_height="match_parent" 
	/>			
</LinearLayout>

```


**line.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
<EditText 
	android:id="@+id/my_title"
	android:layout_width="wrap_content" 
	android:layout_height="wrap_content" 
	android:width="120dp"
	/>
<EditText  
	android:id="@+id/my_content"
	android:layout_width="match_parent" 
	android:layout_height="wrap_content" 
	/>	
</LinearLayout>

```

MainActivity.java

```
package org.crazyit.db;

import android.app.Activity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteException;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.CursorAdapter;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.SimpleCursorAdapter;


public class MainActivity extends Activity
{
	SQLiteDatabase db;
	Button bn = null;
	ListView listView;
	@Override
	public void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		// 创建或打开数据库（此处需要使用绝对路径）
		db = SQLiteDatabase.openOrCreateDatabase(
				this.getFilesDir().toString()
						+ "/my.db3", null); // ①
		listView = (ListView) findViewById(R.id.show);
		bn = (Button) findViewById(R.id.ok);
		bn.setOnClickListener(new OnClickListener()
		{
			@Override
			public void onClick(View source)
			{
				// 获取用户输入
				String title = ((EditText) findViewById(
						R.id.title)).getText().toString();
				String content = ((EditText) findViewById(R.id.content))
						.getText().toString();
				try
				{
					insertData(db, title, content);
					Cursor cursor = db.rawQuery("select * from news_inf"
							, null);
					inflateList(cursor);
				}
				catch (SQLiteException se)
				{
					//如果数据库中不包含数据表，则创建数据库表。
					// 执行DDL创建数据表
					db.execSQL("create table news_inf(_id integer"
							+ " primary key autoincrement,"
							+ " news_title varchar(50),"
							+ " news_content varchar(255))");
					// 执行insert语句插入数据
					insertData(db, title, content);
					// 执行查询
					Cursor cursor = db.rawQuery("select * from news_inf"
							, null);
					//使用ListView将查询结果显示出来。
					inflateList(cursor);
				}
			}
		});
	}
	private void insertData(SQLiteDatabase db
			, String title, String content)  // ②
	{
		// 执行插入语句
		db.execSQL("insert into news_inf values(null , ? , ?)"
				, new String[] {title, content });
	}
	private void inflateList(Cursor cursor)
	{
		// 填充SimpleCursorAdapter
		SimpleCursorAdapter adapter = new SimpleCursorAdapter(
				MainActivity.this,
				R.layout.line, cursor,
				new String[] { "news_title", "news_content" }
				, new int[] {R.id.my_title, R.id.my_content },
				// ③
				//将cousor封装成SimpleCursorAdapter
				CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);  
		// 显示数据
		listView.setAdapter(adapter);
	}
	@Override
	public void onDestroy()
	{
		super.onDestroy();
		// 退出程序时关闭SQLiteDatabase
		if (db != null && db.isOpen())
		{
			db.close();
		}
	}
}



```

程序中③号粗体字代码用于将Cursor封装成SimpleCursorAdapter，这个SimpleCursorAdapter实现了Adapter接口，因此可以作为ListView或GridView的内容适配器。

SimpleCursorAdapter的构造器参数与SimpleAdapter的构造器参数大致相同，区别是SimpleAdapter负责封装集合元素的为Map的List，而SimpleCursorAdapter负责封装Cursor——如果我们把Cursor里的结果集当成List集合，Cursor里的每一行当成Map处理(以数据列的列名为key,数据列的值为value),那么SimpleCursorAdapter与SimpleAdapter就统一起来了。



运行上面程序即可生成数据库文件，如下图：
![这里写图片描述](http://img.blog.csdn.net/20150914104224522)

将my.db3数据库文件导出，用SQLite管理工具打开即可。
[5 个免费的受欢迎的 SQLite 管理工具](http://www.oschina.net/news/43608/5-popular-and-free-sqlite-management-tools)

打开my.db3数据库，如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20150914104507518)


----------
## 三.SQLiteOpenHelper类 ##
在上面的Demo程序中，我们为了判断底层数据库是否包含news_inf数据表，采用的处理方法十分繁琐：程序先尝试向news_inf数据表中插入记录，如果程序抛出异常，在异常捕获的catch块中创建news_inf数据表，然后在插入记录，那么是否有一种更优雅的方式来处理这种问题呢？有Android提供了SQLiteOpenHelper类来处理这种问题。

在实际项目中很少使用SQLiteDatabase的方法来打开数据库，通常会继承SQLiteOpenHelper开发子类，并通过该子类的getReadableDatabase()、getWritableDatabase()方法打开数据库。

SQLiteOpenHelper是Android提供的一个管理数据库的工具类，可用于管理数据库的创建和版本更新。一般用法是创建SQLiteOpenHelper的子类，并扩展它的onCreate(SQLiteDatabase db)、onUpgrade(SQLiteDatabase db,int oldVersion,int newVersion)方法。

SQLiteOpenHelper包含如下常用方法。

 - synchronized SQLiteDatabase getReadableDatabase():以读写的方式打开数据库对应的SQLiteDatabase对象。
 - synchronized SQLiteDatabase getWritableDatabase():以写的方式打开数据库对应的SQLiteDatabase对象。
 - abstract void onCreate(SQLiteDatabase db):用于初次使用软件时生成数据库表。当调用SQLiteOpenHelper的getReadableDatabase()或getWritableDatabase()方法用于获取操作数据库的SQLiteDatabase实例时，如果数据库不存在，Android系统会自动生成一个数据库，接着调用onCreate()方法，onCreate()方法在初次生成数据库时才会调用。重写onCreate()方法时，可以生成数据库表结构，以添加应用使用到的一些初始化数据。
 - abstract void onUpgrade(SQLiteDatabase db,int oldVersion,int newVersion):用于升级软件时更新数据库表结构，此方法在数据库的版本发生变化时会调用，该方法调用时oldVersion代表之前的版本号，newVersion代表数据库当前的版本号。那么在哪里指定数据库的版本号呢？当程序创建SQLiteOpenHelper对象时，必须执行一个version参数，该参数就决定了所使用的版本号——也就是说，数据库的版本号是由程序员控制的。只要某次创建SQLiteOpenHelper时指定的数据库的版本号高于之前指定的版本号，系统会自动触发onUpgrade()方法，程序就可以在onUpgrade()方法里面根据原版本号和目标版本号进行判断，即可根据版本号进行必须的表结构更新。
 - synchronized void close():关闭所有打开的SQLiteDatabase对象。

一旦获得了SQLiteOpenHelper对象之后，程序无需使用SQLiteDatabase的静态方法创建SQLiteDatabase实例，而且可以使用getWritableDatabase()或getReadableDatabase()方法来获取一个用于操作数据库的SQLiteDatabase实例。

***Demo:英文生词本***
允许用户将自己不熟悉的单词添加到系统数据库中，当用户需要查询某个单词或解释时，只要在程序中输入相应的关键词，程序中相应的条目就会显示出来。

效果：
![这里写图片描述](http://img.blog.csdn.net/20150914165817870)

代码：
界面布局文件我就不写了。
***MyDatabaseHelper.java***

```
package com.example.sqliteopenhelper;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

/**
 * Created by songxitang on 2015/9/14.
 */
public class MyDatabaseHelper extends SQLiteOpenHelper 
{
    final String CREATE_TABLE_SQL = "create table dict(_id integer primary "+
            "key autoincrement,word,detail)";

    public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) 
    {
        super(context, name, factory, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db)
    {
	    //第一次使用数据库时自动建表。
        db.execSQL(CREATE_TABLE_SQL);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) 
    {
        System.out.println("---onUpgrade Called---: "+oldVersion+"--->"+newVersion);
    }
}

```

***MainActivity.java***

```
package com.example.sqliteopenhelper;

import android.app.Activity;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class MainActivity extends Activity
{
    MyDatabaseHelper dbHelper;
    Button insert;
    Button search;
    EditText word;
    EditText detail;
    EditText key;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		
		//创建MyDatabaseHelper对象，指定数据库的版本为1，此处使用相对路径即可
		//数据库文件会自动保存在程序的数据文件夹的databases目录下。
        dbHelper = new MyDatabaseHelper(this,"myDict.db3",null,1);
        insert = (Button) findViewById(R.id.insert);
        search = (Button) findViewById(R.id.search);
        word = (EditText) findViewById(R.id.word);
        detail = (EditText) findViewById(R.id.detail);
        key = (EditText) findViewById(R.id.key);

        insert.setOnClickListener(new View.OnClickListener()
        {
            @Override
            public void onClick(View v)
            {
	            //获取用户输入
                String sWord = word.getText().toString();
                String sDetail = detail.getText().toString();
//插入生词记录                insertData(dbHelper.getReadableDatabase(),sWord,sDetail);
//显示提示信息              Toast.makeText(getApplicationContext(),"Insert Word Successful",
                        Toast.LENGTH_SHORT).show();
            }
        });

        search.setOnClickListener(new View.OnClickListener()
        {
            @Override
            public void onClick(View v)
            {
	            //获取用户输入
                String sKey = key.getText().toString();
                //执行查询
                Cursor cursor = dbHelper.getReadableDatabase().rawQuery("" +
                                "select * from dict where word like ? or detail like ?",
                        new String[]{"%" + sKey + "%", "%" + key + "%"});
                //创建一个Bundle对象
                Bundle data = new Bundle();
                data.putSerializable("data",converCursorToList(cursor));
                //创建一个intent
                Intent intent = new Intent(MainActivity.this,ResultActivity.class);
                intent.putExtras(data);
                //启动Activity
                startActivity(intent);
            }
        });
    }

    private ArrayList<Map<String, String>> converCursorToList(Cursor cursor)
    {
        ArrayList<Map<String,String>> result = new ArrayList<>();
        //遍历Cursor结果集
        while (cursor.moveToNext())
        {
	        //将结果集中的数据存入ArrayList中
            Map<String,String> map = new HashMap<>();
            //取出查询记录中第2列和第3列的值
            map.put("word",cursor.getString(1));
            map.put("detail",cursor.getString(2));
            result.add(map);
        }
        return result;
    }

    private void insertData(SQLiteDatabase db, String word, String detail)
    {
	    //执行插入语句
        db.execSQL("insert into dict values(null,?,?)"
                ,new String[]{word,detail});
    }

    @Override
    protected void onDestroy()
    {
        super.onDestroy();
        //退出程序是关闭数据库
        if (dbHelper != null)
        {
            dbHelper.close();
        }
    }
}

```

***ResultActivity.java***

```
package com.example.sqliteopenhelper;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.ListView;
import android.widget.SimpleAdapter;

import java.util.List;
import java.util.Map;

public class ResultActivity extends Activity
{

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result);

        ListView listView = (ListView) findViewById(R.id.show);
        Intent intent = getIntent();
        //获取该intent所携带的数据
        Bundle data = intent.getExtras();
        //从Bundle数据包中取出数据
        List<Map<String,String>> list = (List<Map<String, String>>) data.getSerializable("data");
        //将List封装成SimpleAdapter
        SimpleAdapter adapter = new SimpleAdapter(ResultActivity.this,list,R.layout.line,
                new String[]{"word","detail"},new int[]{R.id.word,R.id.detail});
        //填充ListView
        listView.setAdapter(adapter);
    }
}

```

上面的ResultActivity只是一个普通的Activity，但是我们在AndroidManifest.xml文件中将该Activity设为对话框风格的Activity，这样就可让应用程序已对话框来显示查询结构。

```
 <activity
         android:name=".ResultActivity"
         android:label="@string/title_activity_result"
		//以对话框模式来显示Activity。
         android:theme="@android:style/Theme.Material.Dialog">
 </activity>
```


默认生成的数据库文件存放位置：
![这里写图片描述](http://img.blog.csdn.net/20150914183228976)

数据库文件内容：
![这里写图片描述](http://img.blog.csdn.net/20150914183604507)


