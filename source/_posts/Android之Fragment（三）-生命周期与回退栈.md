---
title: Android之Fragment（三）--生命周期与回退栈
date: 2016-07-24 00:02:48
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

本博客转载自：
[Android Fragment 真正的完全解析（上）](http://blog.csdn.net/lmj623565791/article/details/37970961)、
[Android Fragment 真正的完全解析（下）](http://blog.csdn.net/lmj623565791/article/details/37992017)

Fragment生命周期
------------

与Activity类似，Fragment也存在如下状态。

 - 运行状态：当前Fragment位于前台，用户可见，可以获得焦点。
 - 暂停：其他Activity位于前台，该Fragment依然可见，只是不能获得焦点。
 - 停止状态：该Fragment不可见，失去焦点。
 - 销毁状态：该Fragment完全被删除，或该Fragment所在的Activity被结束。

如下图官方给出的Fragment生命周期图：
![The lifecycle of a fragment (while its activity is running).](http://img.blog.csdn.net/20150823091935563)

 - onAttach():当该Fragment被添加到Activity时被调用。该方法只会被调用一次。
 - onCreate()：创建Fragment时被调用。该方法只会被调用一次。
 - onCreateView()：每次创建、绘制该Fragment的View组件时回调该方法，Fragment将会显示该方法返回的View组件。
 - onActivityCreated()：当Fragment所在的Activity被启动完成后回调该方法。
 - onStart()：启动Fragment时被回调。
 - onResume():恢复Fragment时被回调。
 - onPause()：暂停Fragment时被回调。
 - onStop():停止Fragment时被回调。
 - onDestroyView():销毁该Fragment所包含的View组件时被调用。
 - onDestroy():销毁Fragment时被回调。该方法之后被调用一次。
 - onDetach():将该Fragment从Activity中删除、替换完成时回调该方法，在onDestroy()方法后一定会回调onDetach()方法。该方法只会被调用一次。

从上图可以看出：
1. 当使用Activity加载Fragment时，执行的函数为：
    onAttach()
   onCreate()
   onCreateView()
   onActivityCreated()
   onStart()
   onResume()
   
   2.在Activity上启动一个对话框，Activity将转入暂停状态时，Fragment也会进入暂停状态。执行的函数为：onPause()
   
   3.关闭对话框，Activity进入运行状态，Fragment将会再次进入运行状态，执行的函数为：onResume()
   ***即由2到3，执行的函数为：onPause()--->onResume(),没有重构界面布局。因此，如果当按回退键，不想重构Fragment，要保存原来Fragment中的数据，可以将Fragment隐藏(使用hide()函数)，处于暂定状态***
   
   4.如上图左边那条流程线，替换Fragment不加入回退栈或按退出按钮，则Fragment实例会被销毁。执行的函数为：
   onPause()
   onStop()
   onDestroyView()
   onDestroy()
   onDetach()
   
   5.如上图右边的那条流程线，替换Fragment并加入回退栈，Fragment的实例不会销毁，只会销毁Fragment的界面布局。执行的函数为：
   onPause()
   onStop()
   onDestroyView()
   当按下手机的回退键时，Fragment将会再次显示出来，执行的函数为：
   onCreateView()
   onActivityCreated()
   onStart()
   onResume()


----------
***回退栈***

介绍回退栈之前先介绍下Fragment家族常用的API：
Fragment常用的三个类：

- android.app.Fragment 主要用于定义Fragment

- android.app.FragmentManager 主要用于在Activity中操作Fragment

- android.app.FragmentTransaction 保证一些列Fragment操作的原子性，熟悉事务这个词，一定能明白~

a、获取FragmentManage的方式：

getFragmentManager() // v4中，getSupportFragmentManager

b、主要的操作都是FragmentTransaction的方法

- FragmentTransaction transaction = fm.benginTransatcion();//开启一个事务

- transaction.add()： 往Activity中添加一个Fragment

- transaction.remove() ：从Activity中移除一个Fragment，如果被移除的Fragment没有添加到回退栈，这个Fragment实例将会被销毁。如果添加到回退栈，则这个Fragment的实例不会销毁，但是它的布局视图会被销毁。

- transaction.replace()：使用另一个Fragment替换当前的，实际上就是remove()然后add()的合体~

- transaction.hide()：隐藏当前的Fragment，仅仅是设为不可见，实例并不会销毁，Fragment的布局视图也不会销毁。

- transaction.show()：显示之前隐藏的Fragment

- detach()：会将view从UI中移除,和remove()不同,此时fragment的状态依然由FragmentManager维护。

- attach()：重建view视图，附加到UI上并显示。

- transatcion.commit()//提交一个事务

注意：常用Fragment的哥们，可能会经常遇到这样Activity状态不一致：State loss这样的错误。主要是因为：commit方法一定要在Activity.onSaveInstance()之前调用。

上述，基本是操作Fragment的所有的方式了，在一个事务开启到提交可以进行多个的添加、移除、替换等操作。

注：此处的FragmentTransaction(事务)是一个状态变化的过程，如

```
// Create new fragment and transaction
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

如上面代码，***transaction只是记录了从一个状态到另一个状态的变化过程，即比如从FragmentA替换到FragmentB的过程，当通过函数transaction.addToBackStack(null)将这个事务添加到回退栈，则会记录这个事务的状态变化过程，如从FragmentA --->FragmentB,当用户点击手机回退键时，因为transaction的状态变化过程被保存，则可以将事务的状态变化过程还原，即将FragmentB ---> FragmentA.***


添加到回退栈的函数：transaction.addToBackStack(null);
官方解释为：

> Before you call commit(), however, you might want to call addToBackStack(), in order to add the transaction to a back stack of fragment transactions. This back stack is managed by the activity and allows the user to return to the previous fragment state, by pressing the Back button.
> 
> For example, here's how you can replace one fragment with another, and preserve the previous state in the back stack:

```
// Create new fragment and transaction
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```
In this example, newFragment replaces whatever fragment (if any) is currently in the layout container identified by the R.id.fragment_container ID. By calling addToBackStack(), the replace transaction is saved to the back stack so the user can reverse the transaction and bring back the previous fragment by pressing the Back button.

当Activity的回退栈中没有事务时，在按返回键则会退出Activity。

类似与Android系统为Activity维护一个任务栈，我们也可以通过Activity维护一个回退栈来保存每次Fragment事务发生的变化。如果你将Fragment任务添加到回退栈，当用户点击后退按钮时，将看到上一次的保存的Fragment。一旦Fragment完全从后退栈中弹出，用户再次点击后退键，则退出当前Activity。

Demo:看如下效果
![这里写图片描述](http://img.blog.csdn.net/20150823103925121)

在文本框中输入"one",点击按钮，切换到第二个界面; 在文本框中输入"two",点击按钮，切换到第三个界面；点击按钮出现Toast提示消息；然后点击Back键依次回退到FragmentTwo,文本框中的数据"two"还存在，再次点击Back键会退到FragmentOne,文本框中的数据"one"不存在了。

代码：一共3个Fragment和一个Activity

先看Activity：activity_main.xml

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <FrameLayout
        android:id="@+id/frameLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></FrameLayout>
</RelativeLayout>

```
不同的Fragment就在这个FrameLayout中显示。

MainActivity.java

```
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FragmentManager fragmentManager = getFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

        FragmentOne fragmentOne = new FragmentOne();
        fragmentTransaction.add(R.id.frameLayout,fragmentOne,"One");
        fragmentTransaction.commit();
    }
}

```
很简单，直接将FragmentOne添加到布局文件中的FrameLayout中，注意这里并没有调用FragmentTransaction.addToBackStack(String)，因为我不喜欢在当前显示时，点击Back键出现白板。而正确的时点击相应Back键，即退出我们的Activity.

下面时3个Fragment的布局文件，大体相同，就只写一个，fragment_one.xml：

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.songxitang.fragmenttest.FragmentOne">

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/fragOneBtn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="FragmentOne"
        android:textAllCaps="false" />
</LinearLayout>

```

FragmentOne.java

```
public class FragmentOne extends Fragment {
    private Button fragOneBtn;

    public FragmentOne() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_one, container, false);
        fragOneBtn = (Button) view.findViewById(R.id.fragOneBtn);
        fragOneBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentTwo fragmentTwo = new FragmentTwo();
                FragmentManager fragmentManager = getFragmentManager();
                FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
                fragmentTransaction.replace(R.id.frameLayout,fragmentTwo,"Two");
                //将事务添加到回退栈中
                fragmentTransaction.addToBackStack("FragmentTwo");
                fragmentTransaction.commit();
            }
        });
        return view;
    }
}

```
我们在点击FragmentOne中的按钮时，使用了replace方法，replace是remove和add的合体，并且如果不添加事务到回退栈，前一个Fragment实例会被销毁。这里很明显，我们调用tx.addToBackStack(null);将当前的事务添加到了回退栈，**所以FragmentOne实例不会被销毁，但是视图层次依然会被销毁，即会调用onDestoryView和onCreateView，**证据就是：仔细看上面的效果图，我们在跳转前在文本框输入的内容，在用户Back得到第一个界面的时候不见了。

FragmentTwo.java

```
public class FragmentTwo extends Fragment {
    private Button fragTwoBtn;

    public FragmentTwo() {
        // Required empty public constructor
    }
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_two, container, false);
        fragTwoBtn = (Button) view.findViewById(R.id.fragTwoBtn);
        fragTwoBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentThree fragmentThree = new FragmentThree();
                FragmentManager fragmentManager = getFragmentManager();
                FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
                fragmentTransaction.hide(FragmentTwo.this);
                fragmentTransaction.add(R.id.frameLayout,fragmentThree,"Three");
                fragmentTransaction.addToBackStack(null);
                fragmentTransaction.commit();
            }
        });
        return view;
    }
}

```
这里点击时，我们没有使用replace，而是先隐藏了当前的Fragment，然后添加了FragmentThree的实例，最后将事务添加到回退栈。这样做的目的是为了给大家提供一种方案：如果不希望视图重绘该怎么做，请再次仔细看效果图，我们在FragmentTwo的EditText填写的内容，用户Back回来时，数据还在~~~

最后FragmentThree就是简单的Toast了,FragmentThree.java:

```
public class FragmentThree extends Fragment {
    private Button fragThreeBtn;

    public FragmentThree() {
        // Required empty public constructor
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_three, container, false);
        fragThreeBtn = (Button) view.findViewById(R.id.fragThreeBtn);
        fragThreeBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(getActivity(),"I am a Button in FragmentThree!"
                ,Toast.LENGTH_LONG).show();
            }
        });
        return view;
    }
}

```

值得注意的是：如果你喜欢使用Fragment，一定要清楚这些方法，哪个会销毁视图，哪个会销毁实例，哪个仅仅只是隐藏，这样才能更好的使用它们。

a、比如：我在FragmentA中的EditText填了一些数据，当切换到FragmentB时，如果希望会到A还能看到数据，则适合你的就是hide和show；也就是说，希望保留用户操作的面板，你可以使用hide和show，当然了不要使劲在那new实例，进行下非null判断。

b、再比如：我不希望保留用户操作，你可以使用remove()，然后add()；或者使用replace()这个和remove,add是相同的效果。

c、remove和detach有一点细微的区别，在不考虑回退栈的情况下，remove会销毁整个Fragment实例，而detach则只是销毁其视图结构，实例并不会被销毁。那么二者怎么取舍使用呢？如果你的当前Activity一直存在，那么在不希望保留用户操作的时候，你可以优先使用detach。

[代码下载](http://download.csdn.net/detail/u011026329/9037503)