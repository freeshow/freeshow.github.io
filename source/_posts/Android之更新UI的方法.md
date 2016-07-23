---
title: Android之更新UI的方法
date: 2016-07-24 00:11:20
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

出于性能优化考虑，Android的UI操作并不是线程安全的，这意味着如果有多个线程并发操作UI组件，则可能导致线程安全问题。为了解决这个问题，Android制定了一条简单的规则：只允许UI线程修改Activity里的UI组件。
如果不明白的，可以看这篇博客：
[http://www.cnblogs.com/qingblog/archive/2012/07/18/2597802.html](http://www.cnblogs.com/qingblog/archive/2012/07/18/2597802.html)
## 方法一：使用Handler实现线程之间的通信##
此方法在在这篇博客中已经将的很清楚了：
[http://blog.csdn.net/u011026329/article/details/47785701](http://blog.csdn.net/u011026329/article/details/47785701)

Demo:

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button updateBtn;
    private Handler handler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        updateBtn = (Button) findViewById(R.id.updateBtn);
        handler = new Handler();

        updateBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
	                    //耗时操作，完成之后发送Runnable对象
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        handler.post(new Runnable() {
                            @Override
                            public void run() {
	                            //更新UI
                                textView.setText("After Update");     
                            }
                        });
                    }
                }).start();
            }
        });
    }
}

```
此方法Handler必须关联到UI线程。

## 方法二：View.post(Runnable) ##

Demo:

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button updateBtn;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        updateBtn = (Button) findViewById(R.id.updateBtn);

        updateBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
	                     //函数调用
                        textView.post(new Runnable() {
                            @Override
                            public void run() {
                                textView.setText("After Update");
                            }
                        })
                    }
                }).start();
            }
        });
    }
}

```
View.post(Runnable)方法。在post(Runnable run)方法里，View获得当前线程（即UI线程）的Handler，然后将run对象post到Handler里。
即view.post()方法内部会创建一个handler，并调用handler.post()。故其本质是Hander方法的实现。

**这种方法更简单，但需要传递要更新的View过去；**

## 方法三：Activity.runOnUiThread(Runnable) ##
利用Activity.runOnUiThread(Runnable)把更新ui的代码创建在Runnable中，然后在需要更新ui时，把这个Runnable对象传给Activity.runOnUiThread(Runnable)。 这样Runnable对像就能在ui程序中被调用。如果当前线程是UI线程,那么行动是立即执行。如果当前线程不是UI线程,操作是发布到事件队列的UI线程

Demo:

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button updateBtn;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        updateBtn = (Button) findViewById(R.id.updateBtn);

        updateBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        //函数调用
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {
                                textView.setText("After Update");
                            }
                        });
                    }
                }).start();
            }
        });
    }
}

```
如果在非上下文类中（Activity），可以通过传递上下文实现调用；

```
		Activity activity = (Activity) textView.getContext();
        activity.runOnUiThread(new Runnable() 
        {
            @Override
            public void run() 
            {
                textView.setText("After Update");
            }
        });
```

这种方法使用比较灵活，但如果Thread定义在其他地方，需要传递Activity对象；

## 方法四：AsyncTask ##
此方法在下一篇博客中单独列出来讲解。


更多关于Android更新UI的方法请看这篇博客：
[探讨android更新UI的几种方法](http://www.cnblogs.com/wenjiang/p/3180324.html#undefined)
