---
title: Android之Fragment（四）--Fragment与Activity通讯
date: 2016-07-24 00:00:41
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

本博客参考自：
[Android Fragment详解(五)：Fragment与Activity通讯](http://blog.csdn.net/t12x3456/article/details/8119607)

将Fragment添加到Activity之后，Fragment必须与Activity交互信息，这就需要Fragment获取它所在的Activity，Activity也能获取它所包含的任意的Fragment。可按如下方式进行：

 - Fragment获取它所在的Activity：调用Fragment的getActivity()方法即可返回它所在的Activity。
 - Activity获取它包含的Fragment：调用Activity关联的FragmentManager的findFragmentById(int id)或findFragmentByTag(String tag)方法即可获取指定的Fragment。

提示：在界面布局文件中使用fragment元素添加Fragment时，可以为fragment元素指定android:id或android:tag属性，这两个属性都可以用于标识该Fragment，接下来Activity经可以通过findFragmentById(int id)或findFragmentByTag(String tag)方法来获取该Fragment。

除此之外，Fragment与Activity可能还需要相互传递数据，可按如下方式进行。

 - Activity向Fragment传递数据：在Activity中创建Bundle数据包，并调用Fragment的setArguments(Bundle bundle)方法即可将Bundle数据包传给Fragment。
 例如：
 

```
//创建Bundle，准备向Fragment传入参数
Bundle arguments = new Bundle();
arguments.putInt("index",1);
ExampleFragment fragment = new ExampleFragment();
//向Fragment传入参数
fragment.setArguments(arguments);
getFragmentManager().beginTransaction().
	replace(R.id.container,fragment).commit();
```

 - Fragment向Activity传递数据或Activity需要在Fragment运行中进行实时通信：在Fragment中定义一个内部回调接口，在让包含该Fragment的Activity实现该回调接口，这样Fragment即可调用该回调方法将数据传递给Activity。(***仔细想想，其实就是Java中利用接口实现多态，不了解多态的请看这篇博客：***[Java运行时多态性：继承和接口的实现](http://blog.csdn.net/u011026329/article/details/47945829))


请看下面的Demo:
演示效果：
![这里写图片描述](http://img.blog.csdn.net/20150824143658019)
它有一个activity，activity中含有两个Fragment。LeftFragment显示3个Button，RightFragment显示当点击LeftFragment中的Button时显示的对应内容。LeftFragment必须在用户点击了某个Button时告诉activity，然后activity再告诉RightFragment，RightFragment就显示出对应的内容（为什么这么麻烦？直接LeftFragment告诉RightFragment不就行了？也可以啊，但是你的Fragment就减少了可重用的能力。现在我只需把我的事件告诉宿主，由宿主决定如何处置，这样是不是重用性更好呢？）。如下例，OnFragmentInteractionListener接口在LeftFragment中定义：

```
public class LeftFragment extends Fragment implements View.OnClickListener
{
    public interface OnFragmentInteractionListener
    {
        public void showMessage(int index);
    }
    ...
 }
```
然后activity实现接口OnFragmentInteractionListener，在方法showMessage()中通知RightFragment。当LeftFragment添加到activity中时，会调用Fragment的方法onAttach()，这个方法中适合检查activity是否实现了OnArticleSelectedListener接口，检查方法就是对传入的activity的实例进行类型转换，如下所示：

```
public class LeftFragment extends Fragment implements View.OnClickListener
{
    public interface OnFragmentInteractionListener
    {
        public void showMessage(int index);
    }

    private OnFragmentInteractionListener mListener;

    public LeftFragment() {
        // Required empty public constructor
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnFragmentInteractionListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnFragmentInteractionListener");
        }
    }
}
```

由于在android的实现机制中fragment和activity会被分别实例化为两个不相干的对象,他们之间的联系由activity的一个成员对象fragmentmanager来维护.fragment实例化后会到activity中的fragmentmanager去注册一下,这个动作封装在fragment对象的onAttach中,所以你可以在fragment中声明一些回调接口,当fragment调用onAttach时,将这些回调接口实例化,这样fragment就能调用各个activity的成员函数了,当然activity必须implements这些接口,否则会包classcasterror

如果activity没有实现那个接口，fragment抛出ClassCastException异常。如果成功了，mListener成员变量保存OnFragmentInteractionListener的实例。于是LeftFragment就可以调用mListener的方法来与activity共享事件。例如，如果单击LeftFragment中的按钮，就会调用LeftFragment的onClick(View v)方法，在这个方法中调用mListener.showMessage()来与activity共享事件，如下：

```
@Override
    public void onClick(View v) 
    {
        switch (v.getId())
        {
            case R.id.firstBtn:
                mListener.showMessage(1);
                break;
            case R.id.secondBtn:
                mListener.showMessage(2);
                break;
            case R.id.thirdBtn:
                mListener.showMessage(3);
                break;
        }
    }
```


整体代码如下：
Activity与Fragment的布局文件：
activity_main.xml

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <LinearLayout
        android:id="@+id/left_layout"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:orientation="vertical"></LinearLayout>

    <LinearLayout
        android:id="@+id/right_layout"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_weight="10"
        android:orientation="vertical"></LinearLayout>
</LinearLayout>

```
fragment_left.xml

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.songxitang.fragmentcommunication.LeftFragment">

    <Button
        android:id="@+id/firstBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="FirstButton"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/secondBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="SecondBtn"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/thirdBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ThirdButton"
        android:textAllCaps="false" />
</LinearLayout>

```
fragment_right.xml

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.songxitang.fragmentcommunication.RightFragment">

    <TextView
        android:id="@+id/right_show_message"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/holo_orange_dark"
        android:textColor="@android:color/white"
        android:textSize="30dp"/>

</LinearLayout>

```

然后就是Activity与Fragment的java文件：
LeftFragment.java

```
public class LeftFragment extends Fragment implements View.OnClickListener
{
    public interface OnFragmentInteractionListener
    {
        public void showMessage(int index);
    }
    private Button buttonOne;
    private Button buttonTwo;
    private Button buttonThree;

    private OnFragmentInteractionListener mListener;

    public LeftFragment() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        View view = inflater.inflate(R.layout.fragment_left, container, false);
        buttonOne = (Button) view.findViewById(R.id.firstBtn);
        buttonTwo = (Button) view.findViewById(R.id.secondBtn);
        buttonThree = (Button) view.findViewById(R.id.thirdBtn);
        buttonOne.setOnClickListener(this);
        buttonTwo.setOnClickListener(this);
        buttonThree.setOnClickListener(this);
        return view;
    }


    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnFragmentInteractionListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnFragmentInteractionListener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        mListener = null;
    }

    @Override
    public void onClick(View v) {
        switch (v.getId())
        {
            case R.id.firstBtn:
                mListener.showMessage(1);
                break;
            case R.id.secondBtn:
                mListener.showMessage(2);
                break;
            case R.id.thirdBtn:
                mListener.showMessage(3);
                break;
        }
    }
}

```

MainActivity.java

```
public class MainActivity extends Activity implements LeftFragment.OnFragmentInteractionListener{
    private TextView showMessageView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        System.out.println("Activity--->onCreate");

        FragmentManager manager = getFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();

        LeftFragment leftFragment = new LeftFragment();
        RightFragment rightFragment = new RightFragment();
        transaction.add(R.id.left_layout, leftFragment, "leftFragment");
        transaction.add(R.id.right_layout, rightFragment, "rightFragment");
        transaction.commit();
    }

    @Override
    protected void onResume() {
        super.onResume();
        System.out.println("Activity--->onResume");
        showMessageView = (TextView) findViewById(R.id.right_show_message);
    }

    @Override
    public void showMessage(int index) {
        switch (index)
        {
            case 1:
                showMessageView.setText("First Button Show Content!");
                break;
            case 2:
                showMessageView.setText("Second Button Show Content!");
                break;
            case 3:
                showMessageView.setText("Third Button Show Content!");
        }
    }
}

```

RightFragment.java

```
public class RightFragment extends Fragment {


    public RightFragment() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_right, container, false);
    }
}
```


最后说一下，其实Android Studio已经帮我们实现好了带有回调函数的Fragment模本，创建方法如下：
![这里写图片描述](http://img.blog.csdn.net/20150824111637710)
![这里写图片描述](http://img.blog.csdn.net/20150824111714183)