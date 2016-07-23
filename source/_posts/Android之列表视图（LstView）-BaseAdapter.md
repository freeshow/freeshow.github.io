---
title: Android之列表视图（LstView）--BaseAdapter
date: 2016-07-23 23:36:31
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

通过扩展BaseAdapter可以取得Adapter最大的控制权：程序要创建多少个列表项，每个列表项的组件都由开发者来决定。

本实例的布局文件非常简单，布局文件中只包含一个简单的ListView，布局文件代码如下：
activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
   >
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </ListView>
</LinearLayout>

```

该实例的Activity将会扩展BaseAdapter来实现Adapter对象，Activity代码如下：
MainActivity.java

```
package com.example.baseadaptertest;

import android.graphics.Color;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    ListView listView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listView = (ListView) findViewById(R.id.listview);

        BaseAdapter baseAdapter = new BaseAdapter() {
            @Override
            public int getCount() {
                //指定一共包含20个选项。
                return 20;
            }

            @Override
            public Object getItem(int position) {
                return null;
            }

            //重写该方法，该方法返回值将作为列表项的ID
            @Override
            public long getItemId(int position) {
                //position从0开始
                return position+1;
            }

            //重写该方法，该方法返回的View将作为列表项
            @Override
            public View getView(int position, View convertView, ViewGroup parent) {
                //创建一个LinearLayout，并向其中添加两个组件
                LinearLayout linearLayout = new LinearLayout(MainActivity.this);
                linearLayout.setOrientation(LinearLayout.HORIZONTAL);

                ImageView imageView = new ImageView(MainActivity.this);
                imageView.setImageResource(R.drawable.ic_launcher);

                TextView textView = new TextView(MainActivity.this);
//                textView.setText("第"+(position+1)+"个列表项");
                textView.setText("第"+getItemId(position)+"个列表项");
                textView.setTextSize(20);
                textView.setTextColor(Color.RED);

                linearLayout.addView(imageView);
                linearLayout.addView(textView);

                //返回LinearLayout实例
                return linearLayout;
            }
        };

        listView.setAdapter(baseAdapter);
    }
}

```
上面程序创建了一个BaseAdapter对象，扩展该对象需要重写如下4个方法：

 - getCount：该方法的返回值控制该Adapter将会包含多少个列表项。
 - getItem(int position)：该方法的返回值决定第position处的列表项的内容(position从0开始)。
 - getItemId(int position)：该方法的返回值决定第position处列表项的ID(position从0开始)。
 - getView(int positon,View convertView,ViewGroup parent)：该方法的返回值决定第position处的列表项组件(position从0开始)。

上面方法中最重要的是第1个和第4个。

运行上面程序效果如下：

![这里写图片描述](http://img.blog.csdn.net/20151130192310420)


可能你还没认识到BaseAdapter的强大，现在让我们用扩展的BaseAdapter实现上一节[列表视图(ListView)——SimpleAdapter](http://blog.csdn.net/u011026329/article/details/50115613)中的Demo：

主布局文件(activity_main.xml)：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </ListView>
</LinearLayout>

```

扩展的BaseAdapter--TestAdapter：

```
package com.example.baseadapterdemo.Adapter;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import com.example.baseadapterdemo.R;

/**
 * Created by songxitang on 2015/11/30.
 */
public class TestAdapter extends BaseAdapter {
    protected Context mContext;
    protected LayoutInflater mLayoutInfater;

    //定义所有列表项中的数据：头像，名字，描述
    private int[] imageIds = new int[]
            {R.drawable.tiger,R.drawable.nongyu
                    ,R.drawable.qingzhao,R.drawable.libai};
    private String[] names = new String[]
            {"虎头","弄玉","李清照","李白"};
    private String[] descs = new String[]
            {"可爱的小孩","一个擅长音乐的女孩",
                    "一个擅长文学的女性","浪漫主义诗人"};

    public TestAdapter(Context mContext) {
        this.mContext = mContext;
        mLayoutInfater = (LayoutInflater) mContext.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    }
    @Override
    public int getCount() {
        return 4;
    }

    @Override
    public Object getItem(int position) {
        return null;
    }

    @Override
    public long getItemId(int position) {
        return position+1;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null)
        {
            convertView = mLayoutInfater.inflate(R.layout.list_item,null);

            ImageView imageView = (ImageView) convertView.findViewById(R.id.imageview_header);
            imageView.setImageResource(imageIds[position]);

            TextView textViewName = (TextView) convertView.findViewById(R.id.textview_name);
            textViewName.setText(names[position]);

            TextView textViewDesc = (TextView) convertView.findViewById(R.id.textview_desc);
            textViewDesc.setText(descs[position]);
        }
        return convertView;
    }
}

```

列表项的布局文件(list_item.xml):

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

MainActivity.java:

```
package com.example.baseadapterdemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ListAdapter;
import android.widget.ListView;

import com.example.baseadapterdemo.Adapter.TestAdapter;

public class MainActivity extends AppCompatActivity {
    private ListView listView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listView = (ListView) findViewById(R.id.listview);

        TestAdapter testAdapter = new TestAdapter(this);

        listView.setAdapter(testAdapter);

    }
}

```

运行效果如下图：

![这里写图片描述](http://img.blog.csdn.net/20151130195523669)

总结：其实SimpleAdapter可以算作是扩展的BaseAdapter的一个特例，扩展的BaseAdapter可以最大限度的操作列表项。

到目前为止，我们通过ListView介绍了4中实现Adapter的方法。从表面上看，此处只是在介绍ListView，但实际上这里介绍的知识完全适用于AdapterView的其他子类：GradView、Spinner、Gallery、AdapterViewFlipper等。因此，后面介绍这些组件的步骤依然是如下两步：

 1. 采用4中方式(1.ArrayAdapter 2.ListActivity 3.SimpleAdapter 4.BaseAdapter)之一创建Adapter。
 2. 调用AdapterView的setAdapter(Adapter)方法设置Adapter即可。

参考自：《疯狂Android讲义》