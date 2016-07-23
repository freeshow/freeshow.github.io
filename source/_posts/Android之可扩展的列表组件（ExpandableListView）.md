---
title: Android之可扩展的列表组件（ExpandableListView）
date: 2016-07-23 23:31:29
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

ExpandableListView是ListView的子类，它在普通ListView的基础上进行了扩展，它把应用中的列表项分为几组，每组里又包含多个列表项。

ExpandableListView的用法与普通ListView的用法非常相似，只是ExpandableListView所显示的列表项应该由ExpandableListAdapter提供。ExpandableListView也是一个接口。

与Adapter类似的是，实现ExpandableListAdapter也有如下三种常用方式：

 - 扩展BaseExpandableListAdapter，实现ExpandableListAdapter。
 - 使用SimpleExpandableListAdapter将两个List集合包装成ExpandableListAdapter。
 - 使用SimpleCursorTreeAdapter将Cursor中的数据包装成SimpleCursorTreeAdapter。

下图显示ExpandableListView额外支持的常用XML属性：

![这里写图片描述](http://img.blog.csdn.net/20151201160347205)

Demo：

布局文件：activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ExpandableListView
        android:id="@+id/expandListView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
</LinearLayout>

```

主Activity：MainActivity.java

```
package com.example.expandablelistviewtest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ExpandableListView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ExpandableListView expandableListView = (ExpandableListView) findViewById(R.id.expandListView);
        MainAdapter mainAdapter = new MainAdapter(this);
        expandableListView.setAdapter(mainAdapter);
    }
}

```

Adapter(扩展BaseExpandableListAdapter)：MainAdapter.java

```
package com.example.expandablelistviewtest;

import android.content.Context;
import android.view.Gravity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AbsListView;
import android.widget.BaseExpandableListAdapter;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;

/**
 * Created by songxitang on 2015/12/1.
 */
public class MainAdapter extends BaseExpandableListAdapter {
    private Context context;

    int[] logos = new int[]
            {
                    R.drawable.p,
                    R.drawable.z,
                    R.drawable.t
            };
    private String[] armTypes = new String[]
            {"神族兵种", "虫族兵种", "人族兵种"};
    private String[][] arms = new String[][]
            {
                    {"狂战士", "龙骑士", "黑暗圣堂", "电兵"},
                    {"小狗", "刺蛇", "飞龙", "自爆飞机"},
                    {"机枪兵", "护士MM", "幽灵"}
            };

    public MainAdapter(Context context) {
        this.context = context;
    }

    @Override
    public int getGroupCount() {
        return armTypes.length;
    }

    @Override
    public int getChildrenCount(int groupPosition) {
        return arms[groupPosition].length;
    }

    @Override
    public Object getGroup(int groupPosition) {
        return armTypes[groupPosition];
    }

    @Override
    public Object getChild(int groupPosition, int childPosition) {
        return arms[groupPosition][childPosition];
    }

    @Override
    public long getGroupId(int groupPosition) {
        return groupPosition;
    }

    @Override
    public long getChildId(int groupPosition, int childPosition) {
        return childPosition;
    }

    @Override
    public boolean hasStableIds() {
        return false;
    }

    @Override
    public View getGroupView(int groupPosition, boolean isExpanded, View convertView, ViewGroup parent) {
	    //此处不建议用这种方式添加布局，最好使用布局文件添加布局。
        LinearLayout linearLayout = new LinearLayout(context);
        linearLayout.setOrientation(LinearLayout.HORIZONTAL);

        ImageView imageView = new ImageView(context);
        imageView.setImageResource(logos[groupPosition]);

        TextView textView = new TextView(context);
        textView.setText(getGroup(groupPosition).toString());

        linearLayout.addView(imageView);
        linearLayout.addView(textView);

        return linearLayout;
    }

    private TextView getTextView()
    {
        AbsListView.LayoutParams layoutParams = new AbsListView.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,64);
        TextView textView = new TextView(context);
        textView.setLayoutParams(layoutParams);
        textView.setGravity(Gravity.CENTER_VERTICAL|Gravity.LEFT);
        textView.setPadding(36,0,0,0);
        textView.setTextSize(20);

        return textView;
    }

    @Override
    public View getChildView(int groupPosition, int childPosition, boolean isLastChild, View convertView, ViewGroup parent) {
        TextView textView = getTextView();
        textView.setText(getChild(groupPosition,childPosition).toString());

        return textView;
    }

    @Override
    public boolean isChildSelectable(int groupPosition, int childPosition) {
        return false;
    }
}

```

运行效果：

![这里写图片描述](http://img.blog.csdn.net/20151201160909756)

上面程序使用扩展BaseExpandableListAdapter来实现ExpandableListAdapter，当扩展BaseExpandableListAdapter时，关键是实现如下4个方法：

 - getGroupCount()：该方法返回包含的组列表项的数量。
 - getGroupView()：该方法返回的View对象将作为组列表项。
 - getChildrenCount()：该方法返回特定组所包含的子列表项的数量。
 - getChildView()：该方法返回的View对象将作为特定组、特定位置的子列表项。

上面程序中的getChildView()方法返回一个普通TextView，因为每个自列表项都是一个普通文本框。而getGroupView()方法则返回一个LinearLayout对象，该LinearLayout对象里包含一个ImageView和一个TextView。因此，每个组列表项都由图片和文本组成。


参考自：《疯狂Android讲义》