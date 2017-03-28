---
title: Android之Fragment（二）--使用
date: 2016-07-24 00:04:09
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

转载自：[Android Fragment 基本介绍](http://www.cnblogs.com/mengdd/archive/2013/01/08/2851368.html)
## 创建Fragment ##
与创建Activity类似，开发者实现的Fragment必须继承Fragment基类，Android提供了如下图所示的Fragment继承体系。

![这里写图片描述](http://img.blog.csdn.net/20150822100223601)

开发者实现的Fragment可以根据需要继承上图所示的Fragment基类或它的任意子类。接下来实现Fragment与实现Activity非常相似，它们都需要实现与Activity类似的回调方法，如onCreate()、onCreateView()、onStart()、onResume()、onPause()、onStop()等。

通常来说，创建Fragment需要实现如下三个方法：

 - onCreate()：系统创建Fragment对象后回调该方法，在实现代码中只初始化想要在Fragment中保持的必要组件，当Fragment被暂停或者停止后可以恢复。
 - onCreateView()：当Fragment绘制界面时回调该方法。该方法必须返回一个View,该View也就是该Fragment所显示的View.
 - onPause():当用户离开该Fragment时将回调该方法。
 
对于大部分Fragment而言，通常都可以重写上面的这三个方法。但实际开发者可以根据需要重写Fragment的任意回调方法。

***实现Fragment的UI***

提供Fragment的UI，必须实现onCreateView()方法。
假设Fragment的布局设置写在example_fragment.xml资源文件中，那么onCreateView()方法可以如下写：

```
public static class ExampleFragment extends Fragment
{
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
　　Bundle savedInstanceState)
    {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}
```

 onCreateView()中container参数代表该Fragment在Activity中的父控件；savedInstanceState提供了上一个实例的数据。

inflate()方法的三个参数：

- 第一个是resource ID，指明了当前的Fragment对应的资源文件；
- 第二个参数是父容器控件；
- 第三个布尔值参数表明是否连接该布局和其父容器控件，在这里的情况设置为false，因为系统已经插入了这个布局到父控件，设置为true将会产生多余的一个View Group。

把Fragment加入Activity
-------------------
当Fragment被加入Activity中时，它会处在对应的View Group中。


 - 一种是在Activity的layout中使用标签<fragment>声明；
 - 另一种方法是在代码中把它加入到一个指定的ViewGroup中。
 - 另外，Fragment它可以并不是Activity布局中的任何一部分，它可以是一个不可见的部分。这部分内容先略过。
 


----------


***静态的使用Fragment***
这是使用Fragment最简单的一种方式，把Fragment当成普通的控件，直接写在Activity的布局文件中。

在Activity的布局文件中，将Fragment作为一个子标签加入即可。
如：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment android:name="com.example.news.ArticleListFragment"
            android:id="@+id/list"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
    <fragment android:name="com.example.news.ArticleReaderFragment"
            android:id="@+id/viewer"
            android:layout_weight="2"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
</LinearLayout>
```

其中android:name属性填上你自己创建的fragment的完整类名。

当系统创建这个Activity的布局文件时，系统会实例化每一个fragment，并且调用它们的onCreateView()方法，来获得相应fragment的布局，并将返回值插入fragment标签所在的地方。

有三种方法为Fragment提供ID：

1. android:id属性：唯一的id
2. android:tag属性：唯一的字符串
3. 如果上面两个都没提供，系统使用容器view的ID。

Demo:
1. 首先创建Fragment及其布局文件，ExampleFragment.java:

```
public class ExampleFragment extends Fragment {
    private TextView textView;
    private Button showBtn;

    public ExampleFragment() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_example, container, false);
        textView = (TextView) view.findViewById(R.id.textView);
        showBtn = (Button) view.findViewById(R.id.showBtn);

        showBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView.setText("This is a Fragment!");
            }
        });

        return view;
    }
}

```
布局文件，fragment_example.xml:

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center|top"
    android:orientation="vertical"
    tools:context="com.example.songxitang.fragmentexample.ExampleFragment">

    <!-- TODO: Update blank fragment layout -->
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="Hello Fragment"

        />

    <Button
        android:id="@+id/showBtn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="Click" />
</LinearLayout>

```
2.将Fragment添加到Activity中，MainActivity.java:

```
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```
及其布局文件，activity_main.xml:

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/exampleFragment"
        android:name="com.example.songxitang.fragmentexample.ExampleFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></fragment>
</LinearLayout>
```
效果：
![这里写图片描述](http://img.blog.csdn.net/20150822125224775)
点击CLICK按钮后：
![这里写图片描述](http://img.blog.csdn.net/20150822125321653)

所有事件及绘图都在Fragment中实现，Activity中看起来非常干净。


----------


***动态的使用Fragment***

通过编程的方式将Fragment加入到一个ViewGroup中

当Activity处于Running状态下的时候，可以在Activity的布局中动态地加入Fragment，只需要指定加入这个Fragment的父View Group即可。因此Activity中的布局文件需要将静态加载Fragment的fragment元素替换为可以容纳组件的容器，如LineLayout、Include、FragmentLayout等，将Fragment对象放入容器中。

首先，需要一个FragmentTransaction实例：

> FragmentManager fragmentManager = getFragmentManager()

> FragmentTransaction fragmentTransaction
 = fragmentManager.beginTransaction();

(注,如果import android.support.v4.app.FragmentManager;那么使用的是：FragmentManager fragmentManager = getSupportFragmentManager();）

之后，用add()方法加上Fragment的对象：

> ExampleFragment fragment = new ExampleFragment();

>fragmentTransaction.add(R.id.fragment_container, fragment);

>fragmentTransaction.commit();

其中add()函数第一个参数是这个fragment的容器，即父控件组。
最后需要调用commit()方法使得FragmentTransaction实例的改变生效。

Demo:
动态使用Fragment修改上面的静态使用Fragment的Demo

ExampleFragment.java及其布局文件不变。
修改Activity的布局文件，activity_main.xml:

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <FrameLayout
        android:id="@+id/fragmLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></FrameLayout>
</LinearLayout>
```
修改MainActivity.java:

```
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FragmentManager fragmentManager = getFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

        ExampleFragment exampleFragment = new ExampleFragment();
        fragmentTransaction.add(R.id.fragmLayout,exampleFragment);
        fragmentTransaction.commit();
    }
}

```
效果和上面的一样。