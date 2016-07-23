---
title: Android之列表视图（LitView）--SimpleAdapter
date: 2016-07-23 23:38:16
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

通过ArrayAdapter实现Adapter虽然简单、易用，但ArrayAdapter的功能比较有限，它的每个列表项只能是TextView。如果开发者需要实现更多复杂的列表项，则可以考虑使用SimpleAdapter.

不要被SimpleAdapter的名字欺骗了，SimpleAdapter并不简单，而且它的功能非常强大。ListView的大部分应用场景，都可以通过SimpleAdapter来提供列表项。

下面先定义界面布局文件。
activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>

```
上面的界面布局文件定义了一个ListView，该ListView将会显示有SimpleAdapter提供的列表项。

下面是Activity代码：
MainActivity.java

```
package com.example.simpleadaptertest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ListView;
import android.widget.SimpleAdapter;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MainActivity extends AppCompatActivity {
    //定义所有列表项中的数据：头像，名字，描述
    private int[] imageIds = new int[]
            {R.drawable.tiger,R.drawable.nongyu
                    ,R.drawable.qingzhao,R.drawable.libai};
    private String[] names = new String[]
            {"虎头","弄玉","李清照","李白"};
    private String[] descs = new String[]
            {"可爱的小孩","一个擅长音乐的女孩",
            "一个擅长文学的女性","浪漫主义诗人"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //创建一个List集合(表示所有列表项)，List集合的元素是Map(代表一个列表项)
        List<Map<String,Object>> listItems = new ArrayList<>();
        for (int i=0; i<names.length; i++)
        {
            Map<String,Object> listItem = new HashMap<>();
            listItem.put("header",imageIds[i]);
            listItem.put("personName",names[i]);
            listItem.put("desc",descs[i]);
            listItems.add(listItem);
        }
        //创建一个SimpleAdapter
        SimpleAdapter simpleAdapter = new SimpleAdapter(this,listItems,//所有列表项。
                R.layout.simple_item,//布局文件。
                new String[]{"header","personName","desc"},//上面Map对象中的Key。
                new int[]{R.id.imageview_header,R.id.textview_name,
                        R.id.textview_desc});//列表项中要填充的组件ID。
        ListView listView = (ListView) findViewById(R.id.listview);
        //为ListView设置Adapter
        listView.setAdapter(simpleAdapter);
    }
}

```
使用SimpleAdapter的最大难点在于创建SimpleAdapter对象，它需要5个参数，其中后面4个参数十分关键：

 - 第2个参数：该参数应该是一个List< ?extends Map< String,?>>类型的集合对象，该集合中的每个Map< String,?>对象生成一个列表。
 - 第3个参数：该参数指定一个界面布局的ID。该界面布局文件作为列表项组件。
 - 第4个参数：该参数应该是一个String[ ]类型的参数，该参数决定提取Map< String,?>对象中那些key对应的value来生成列表项。
 - 第5个参数：该参数决定填充那些组件。
从上面程序看，listIterms是一个长度为4的集合，这意味着它生成的ListView将会包含4个列表项，每个列表项都是R.layout.simple_item对相的组件(也就是一个LinearLayout组件)。LinearLayout中包含了3个组件：ID为R.id.imageview_header的ImageView组件、ID为R.id.textview_name和R.id.textview_desc的TextView组件，这些组件的内容有listItems集合提供。

R.laylout.simple_item对应的布局文件代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    <!--定义一个ImageView，用于作为列表项的一部分-->
    <ImageView
        android:id="@+id/imageview_header"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingLeft="10dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <!--定义一个TextView，用作列表项的一部分-->
        <TextView
            android:id="@+id/textview_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingLeft="10dp"
            android:textColor="#f0f"
            android:textSize="20dp" />
        <!--定义一个TextView，用作列表项的一部分-->
        <TextView
            android:id="@+id/textview_desc"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingLeft="10dp"
            android:textSize="14dp" />
    </LinearLayout>
</LinearLayout>
```
运行效果如下：

![这里写图片描述](http://img.blog.csdn.net/20151130182613938)


SimpleAdapter同样可以作为ListActivity的内容Adapter，这样可以让用户方面的定制ListActivity所显示的列表项。

如果需要监听用户单击、选中某个列表项的事件，则可以通过AdapterView的setOnItemClickListener()方法为单击事件添加监听器，或者通过setOnItemSelectedListener()方法为列表项的选中事件添加监听器。

**总结**：
ArrayAdapter与SimpleAdapter的区别：

 - ArrayAdapter中的每个列表项的组件只能是TextView且每个列表项中只能添冲一个值。故不能设计复杂的列表项。
 - SimpleAdapter中每个列表项中可包含多个不同的组件，故每个列表项中可添加多个值，故可设计复杂的列表项。

参考自：《疯狂Android讲义》