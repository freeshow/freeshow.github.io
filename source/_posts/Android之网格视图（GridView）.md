---
title: Android之网格视图（GridView）
date: 2016-07-23 23:32:45
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

GridView用于在界面上按行、列分布的方式显示多个组件。

GridView和ListView有共同的父类：AbsListView，因此GridView和ListView具有很高的相似性，它们都是列表项。它们的唯一区别是：ListView只显示一列；而GridView可以显示多列。从这个角度来看，ListView相当于一种特殊的GridView，如果让GridView只显示一列，那么该GridView就变成了ListView。

与ListView类似的是，GridView也需要通过Adapter类提供显示的数据。

GridView提供的常用XML属性及相关方法如下表：

![这里写图片描述](http://img.blog.csdn.net/20151201103242374)

注意：使用GridView时一般都应该制定numColumns大于1；否则该属性默认值为1，如果将该属性设为1，则意味着该GridView只有一列，那么GridView就变成了ListView。

上表中android:stretchMode属性支持如下几个属性：

 - NO_STRETCH：不拉伸。
 - STRETCH_SPACING：仅拉伸元素之间的距离。
 - STRETCH_SPACING_UNIFORM：表格元素本身、元素之间的距离一起拉伸。
 - STRETCH_COLUMN_WIDTH：仅拉伸表格元素本身。

下面通过一个实例来介绍GridView的用法，本实例采用SimpleAdapter为GridView提供数据。

Demo：带预览的图片浏览器
本实例将采用一个GridView以行、列的形式组织所有图片的预览视图。然后程序用一个ImageView来显示图片。

activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical">
    <!--定义一个GridView组件-->
    <GridView
        android:id="@+id/gridView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:horizontalSpacing="1pt"
        android:numColumns="4"
        android:verticalSpacing="1pt">
    </GridView>
    <!--定义一个ImageView组件-->
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="240dp"
        android:layout_height="240dp"
        android:layout_gravity="center_horizontal" />
</LinearLayout>

```

上面的界面布局只是简单的定义了一个GridView、一个ImageView。定义GridView时指定了android:numColumns="4",这意味着该网格包含4列。那么该GridView多少行呢？这是动态改变的——正如ListView到底包含多少行由该ListView对应的Adapter所决定的，GridView到底包含多少行也是由Adapter决定的。

主程序代码：MainActivity.java

```
package com.example.gridviewtest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.GridView;
import android.widget.ImageView;
import android.widget.SimpleAdapter;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MainActivity extends AppCompatActivity {
    private GridView gridView;
    private ImageView imageView;
    int[] imageIds = new int[]{
            R.drawable.bomb1,R.drawable.bomb2,R.drawable.bomb3,R.drawable.bomb4,
            R.drawable.bomb5,R.drawable.bomb6,R.drawable.bomb7,R.drawable.bomb8,
            R.drawable.bomb9,R.drawable.bomb10,R.drawable.bomb11,R.drawable.bomb12
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        gridView = (GridView) findViewById(R.id.gridView);
        imageView = (ImageView) findViewById(R.id.imageView);

        List<Map<String,Object>> listItems = new ArrayList<>();
        for (int i=0; i<imageIds.length; i++)
        {
            Map<String,Object> listItem = new HashMap<>();
            listItem.put("image",imageIds[i]);
            listItems.add(listItem);
        }
        SimpleAdapter simpleAdapter = new SimpleAdapter(this,listItems,
                R.layout.cell,
                new String[]{"image"},
                new int[]{R.id.image1});

        gridView.setAdapter(simpleAdapter);

        gridView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                imageView.setImageResource(imageIds[position]);
            }
        });

        gridView.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                imageView.setImageResource(imageIds[position]);
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });
    }
}

```


布局文件cell.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:gravity="center_horizontal"
	android:padding="2pt">
<ImageView
	android:id="@+id/image1"
	android:layout_width="50dp" 
	android:layout_height="50dp" 
	/>	
</LinearLayout>
```

上面程序使用了SimpleAdapter对象作为GridView的Adapter，这个SimpleAdapter底层保证了一个长度为12的List集合——这意味着GridView一共需要12个组件，GridView总共有4列，因此，GridView一共包含3行。

运行效果如下：

![这里写图片描述](http://img.blog.csdn.net/20151201105327921)


总结：
GridView每行中的各个组件都相当于ListView中的一个组件(一行)。

用BaseAdapter重写上面的Demo：

activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical">
    <!--定义一个GridView组件-->
    <GridView
        android:id="@+id/gridView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:horizontalSpacing="1pt"
        android:numColumns="4"
        android:verticalSpacing="1pt">
    </GridView>
    <!--定义一个ImageView组件-->
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="240dp"
        android:layout_height="240dp"
        android:layout_gravity="center_horizontal" />
</LinearLayout>

```


扩展的BaseAdapter：ImageAdapter.java

```
package com.example.gridviewwithbaseadapter;

import android.content.Context;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;

import java.lang.reflect.Field;

/**
 * Created by songxitang on 2015/12/1.
 */
public class ImageAdapter extends BaseAdapter {
    private static final String FACE_PREFIX = "bomb";
    private Context context;
    private int imageCount;

    public ImageAdapter(Context context) {
        this.context = context;
        this.imageCount = calculateImageCount();
    }

    //计算图片的数量
    private int calculateImageCount() {
        int i = 0;
        while (true) {
            i++;
            String imageName = FACE_PREFIX + i;

            try {
                //Java反射: 查看资源文件中是否有名为faceName的资源。
                R.drawable.class.getField(imageName);
            } catch (Exception e) {
                break;
            }
        }
        return i - 1;
    }

    @Override
    public int getCount() {
        return imageCount;
    }

    @Override
    public Object getItem(int position) {
        return null;
    }

    @Override
    public long getItemId(int position) {
        return position + 1;
    }

    //根据资源名获取资源ID
    public static int getResourceIDFromName(Class c, String name) {
        int resID = -1;
        try {
            //利用Java反射
            Field field = c.getField(name);
            resID = field.getInt(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return resID;
    }

    //根据position获取资源ID
    public static int getImageResourceID(int position) {
        //position是从0开始的。
        position++;

        try {
            return getResourceIDFromName(R.drawable.class, FACE_PREFIX + position);
        } catch (Exception e) {
            e.printStackTrace();
            return -1;
        }
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ImageView imageView = new ImageView(context);
        imageView.setImageResource(getImageResourceID(position));

        return imageView;
    }
}

```

MainActivity.java

```
package com.example.gridviewwithbaseadapter;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.GridView;
import android.widget.ImageView;
import android.widget.SimpleAdapter;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MainActivity extends AppCompatActivity {
    private GridView gridView;
    private ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        gridView = (GridView) findViewById(R.id.gridView);
        imageView = (ImageView) findViewById(R.id.imageView);

        ImageAdapter imageAdapter = new ImageAdapter(this);
        gridView.setAdapter(imageAdapter);

        gridView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                imageView.setImageResource(ImageAdapter.getImageResourceID(position));
            }
        });

        gridView.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                imageView.setImageResource(ImageAdapter.getImageResourceID(position));
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });
    }
}

```

目录结构如下图：

![这里写图片描述](http://img.blog.csdn.net/20151201144835224)

运行效果与上面的Demo相同。


参考自：《疯狂Android讲义》