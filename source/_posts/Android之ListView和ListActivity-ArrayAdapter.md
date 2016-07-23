---
title: Android之ListView和ListActivity--ArrayAdapter
date: 2016-07-23 23:41:00
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

ListView以垂直列表的形式显示所有的列表项。
生成列表视图有如下两种方式：

 - 直接使用ListView进行创建。
 - 让Activity继承ListActivity(相当于该Activity显示的)。

一旦程序获得了ListView之后，接下来就需要为ListView设置它要显示的列表项了。在这一点上ListView显示出AdapterView的特征：通过setAdapter(Adapter)方法为之提供Adapter，并由Adapter提供列表项即可。

***提示***：ListView、GridView、Spinner、Gallery等都只是容器，而Adapter负责提供每个“列表项”的组件，AdapterView则负责采用合适的方式显示这些列表项。

AbsListView提供的常用XML属性及相关方法如下表所示：
![AbsListView提供的常用XML属性及相关方法](http://img.blog.csdn.net/20151130095751962)

ListView提供的常用XML属性如下表所示：
![ListView提供的常用XML属性](http://img.blog.csdn.net/20151130095907921)

Demo：改变分隔条、基于数组的ListView。
程序清单：
activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--直接使用数组资源给出列表项-->
    <!--设置使用红色的分隔符-->
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:divider="#f00"
        android:dividerHeight="2px"
        android:headerDividersEnabled="false"
        android:entries="@array/books"></ListView>
</LinearLayout>

```

arrays.xml

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="books">
        <item>Hello Java</item>
        <item>Hello JavaWeb</item>
        <item>Hello Android</item>
        <item>Hello Python</item>
    </string-array>
</resources>

```
MainActivity.java

```
package com.example.simplelistviewtest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

```

显示效果如下图所示：

![这里写图片描述](http://img.blog.csdn.net/20151130102752157)

使用数组创建ListView十分简单，但这种ListView能定制的内容很少，甚至连每个列表项的字号大小、颜色都不能改变。

如果想对ListView的外观、行为进行定制，就需要把ListView作为AdapterVeiw使用，通过Adapter控制每个列表项的外观和行为。


----------

## Adapter接口及实现类 ##

Adapter本身只是一个接口，它派生了ListAdapter和SpinnerAdapter两个子接口，其中ListAdapter为AbsListView提供列表项，而SpinnerAdapter为AbsSpinner提供列表项。Adapter接口及其实现类的继承关系图如下：

![这里写图片描述](http://img.blog.csdn.net/20151130110705121)

从图中可以看出，BaseAdapter同时实现了ListAdapter、SpinnerAdapter两个接口，因此BaseAdapter及其子类可以同时为AbsListView、AbsSpinner提供列表项。

Adapter常用的实现类如下：

 - ArrayAdapter：简单、易用的Adapter，通常用于将数组或List集合的多个**值**包装成列表项。
 - SimpleAdapter：并不简单、功能强大的Adapter，可以用于将List集合的多个**对象**包装成多个列表项。
 - SimpleCursorAdapter：与SimpleAdapter基本相似，只是用于包装Cursor提供的数据。
 - BaseAdapter：通常用于被扩展。扩展BaseAdapter可以对各列表项进行最大限度的定制。

Demo：使用ArrayAdapter创建ListView。

在界面布局文件中定义两个ListView。
activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!--使用红色的分隔条-->
    <ListView
        android:id="@+id/listview_one"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:divider="#f00"
        android:dividerHeight="2px"
        android:headerDividersEnabled="false">
    </ListView>

    <!--使用绿色的分隔条-->
    <ListView
        android:id="@+id/listview_two"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:divider="#0f0"
        android:dividerHeight="2px"
        android:headerDividersEnabled="false">
    </ListView>
</LinearLayout>

```

上面的界面布局文件中定义了两个ListView，但这两个ListView都没有指定android:entries属性，这意味着它们都需要通过Adapter来提供列表项。

接下来Activity为两个ListView提供Adapter，Adapter决定ListView所显示的列表项。程序如下。
MainActivity.java

```
package com.example.arrayadaptertest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ListView listViewOne = (ListView) findViewById(R.id.listview_one);
        //定义一个数组
        String[] arr1 = {"孙悟空","猪八戒","牛魔王"};
        //将数组包装为ArrayAdapter
        ArrayAdapter<String> arrayAdapterOne = new ArrayAdapter<String>
                (this,R.layout.array_item,arr1);
        //为ListView设置Adapter
        listViewOne.setAdapter(arrayAdapterOne);

        ListView listViewTwo = (ListView) findViewById(R.id.listview_two);
        String[] arr2 = {"Java","Android","Python","Scala"};
        ArrayAdapter<String> arrayAdapterTwo = new ArrayAdapter<String>
                (this,R.layout.array_item,arr2);
        listViewTwo.setAdapter(arrayAdapterTwo);
    }
}

```
创建ArrayAdapter时必须指定如下三个参数：

 - Context：这个参数无需多说，它代表了访问整个Android应用的接口。几乎创建所有组件都需要传入Context对象。
 - textViewResourceId：一个资源ID，该资源ID代表一个TextView，该TextView组件将作为ArrayAdapter的列表项组件。
 - 数组或List：该数组或List将负责为多个列表项提供数据。


不难看出，创建ArrayAdapter时传入的第二个参数控制每个列表项的组件，第三个参数则负责为列表项提供数据。该数组或List包含多少个元素，就将生成多少个列表项，每个列表项都是TextView组件，TextView组件显示的文本由数组或List元素提供。

以第一个ArrayAdapter为例，该ArrayAdapter对应的数组为{"孙悟空","猪八戒","牛魔王"}，该数组将会负责生成一个包含三个列表项的ArrayAdapter，每个列表项的组件外观由R.layout.array_item布局文件(该布局文件只是一个TextView组件)控制，第一个TextView列表项显示的文本为“孙悟空”，第二个列表项显示的文本为“猪八戒”......

array_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/textview"
    android:textSize="24dp"
    android:padding="10px"
    android:shadowColor="#f0f"
    android:shadowDx="4"
    android:shadowDy="4"
    android:shadowRadius="2">
</TextView>
```

运行效果：

![这里写图片描述](http://img.blog.csdn.net/20151130151849614)


***Demo:基于ListActivity实现列表。***

如果程序窗口仅仅需要显示一个列表，则可以直接让Activity继承ListActivity类实现，ListActivity的子类无须调用setContentView()方法来显示某个界面，而是可以直接传入一个内容Adapter，ListActivity的子类就呈现出一个列表。
例如如下程序：
MainActivity.java

```
package com.example.listactivitytest;

import android.app.ListActivity;
import android.os.Bundle;
import android.widget.ArrayAdapter;

public class MainActivity extends ListActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //无须使用布局文件
        String[] arr = {"孙悟空","猪八戒","唐僧"};
        //创建ArrayAdapter对象
        ArrayAdapter<String> arrayAdapter = new ArrayAdapter<String>
                (this,android.R.layout.simple_list_item_multiple_choice,arr);
        //设置该窗口显示列表
        setListAdapter(arrayAdapter);
    }
}

```

上面程序的Activity继承了ListActivity，ListActivity无须界面布局文件——相当于它的布局文件只有一个ListView，因此，只要为ListActivity设置Adapter即可。上面程序使用Android默认提供的android.R.layout.simple_list_item_multiple_choice布局文件作为列表项组件，当然，也可以自定义这个布局文件。

运行效果如下：

![这里写图片描述](http://img.blog.csdn.net/20151130153450688)