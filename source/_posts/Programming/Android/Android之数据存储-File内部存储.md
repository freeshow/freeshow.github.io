---
title: Android之数据存储--File内部存储
date: 2016-07-23 23:55:46
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

Java提供了一套完整的IO流体系，包括FileInputStream、FileOutputStream等，通过这些IO流可以非常方便的访问磁盘上的内容。Android同样支持以这种方式来访问手机存储器上的文件。

Context提供了如下两种方法来打开应用程序的数据文件夹里的文件IO流。

 - FileInputStream openFileInput(String name)：打开应用程序的数据文件夹下的name文件对应的输入流。
 - FileOutputStream openFileOutput(String name，int mode)：打开应用程序的数据文件夹下的name文件对应的输入流。

以上两个方法默认打开的文件路径为：/data/data/< package name>/files目录下的name文件。即应用程序的数据文件夹是/data/data/< package name>/files,故此时File存储的数据位于内部存储器上。

上面两种方法分别用于打开文件输入流、输出流，其中第二个参数mode指定打开文件的模，该模式支持如下值：

 - MODE_PRIVATE参数：指示要创建这个文件（或者，如果有同名文件存在，则会替换旧文件），并且让这个文件是应用程序的私有文件。
 - MODE_APPEND：如果文件已经存在，则在后面追加数据
 - MODE_WORLD_READABLE：让其他应用有读的权限
 - MODE_WORLD_WRITEABLE：让其他应用有写的权限

注：MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE已经在Android 4.2（API level 17）废弃了，因为这样危险，安全性不高。

1.**往内部存储内写文件（写入到/data/data/com.xxx.xxx/files目录下，com.xxx.xxx为应用程序包名）：**

```
        String FILE_NAME = "hello_file";
        String FILE_TEXT = "hello world!!!";
        FileOutputStream fos = openFileOutput(FILE_NAME, Context.MODE_PRIVATE);
        fos.write(FILE_TEXT.getBytes());
        fos.close();
```

运行上面代码，发现/data/data/com.xxx.xxx/files目录下多了个hello_file文件。

2.**读取内部存储内的文件（也就是从/data/data/com.xxx.xxx/files目录下读取文件）:**

```
        String FILE_NAME = "hello_file";
        byte[] b = new byte[1024];
        StringBuffer sb = new StringBuffer();
        FileInputStream fis = openFileInput(FILE_NAME);
        int num;
        while ((num = fis.read(b)) != -1) {
            sb.append(new String(b, 0, num));
        }
        fis.close();
        Log.d("xxx", sb.toString());
```
运行一下，成功打印hello world!!!

除此之外，Context还提供了如下几个方法来访问应用程序的数据文件夹。

 - getDir(String name,int mode)：在应用程序数据文件夹下获取或创建name对应的子目录。
 - File getFileDir()：获取应用程序的数据文件夹的绝对路径。
 - String[] fileList()：返回应用程序数据文件夹下的全部文件。
 - deleteFile(String name)：删除应用程序数据文件夹下的制定文件。




----------
下面示范如何读取应用程序数据文件夹得内容。给程序界面布局只有两个文本框和两个按钮，其中第一组文本框和按钮用于处理写入，文本框用于接收用户输入，当用户单击"写入"按钮时，程序会把文本框中的数据写入文件；第二组文本框和按钮用于处理读取，当用户点击“读取”按钮时，该文本框显示文件中的数据，效果如下：
![这里写图片描述](http://img.blog.csdn.net/20150826212853365)

代码如下：

activity_main.xml

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/writeEditText"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/writeButton"
        android:text="Write File"
        android:textAllCaps="false"/>

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/readButton"
        android:text="Read File"
        android:textAllCaps="false"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/readEditText"/>
</LinearLayout>

```


MainActivity.java

```
public class MainActivity extends Activity {
    final String FILE_NAME = "file.test";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        System.out.println(new StringBuffer("a").append("b").append("c"));

        Button writeButton = (Button) findViewById(R.id.writeButton);
        Button readButton = (Button) findViewById(R.id.readButton);

        final EditText writeEditText = (EditText) findViewById(R.id.writeEditText);
        final EditText readEditText = (EditText) findViewById(R.id.readEditText);
        Log.i("FILE",getFilesDir().toString());
        writeButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                write(writeEditText.getText().toString());
                writeEditText.setText("");
            }
        });

        readButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                readEditText.setText(read());
            }
        });
    }

    private String read() {
        try {
	        //从文件读取数据
            FileInputStream fileInputStream = openFileInput(FILE_NAME);
            byte[] buffer = new byte[1024];
            int hasRead = 0;
            StringBuilder stringBuilder = new StringBuilder("");
            while ((hasRead = fileInputStream.read(buffer)) > 0 )
            {
                stringBuilder.append(new String(buffer,0,hasRead));
            }
            fileInputStream.close();
            return stringBuilder.toString();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private void write(String s) {
        try {
	        //向文件写入数据
            FileOutputStream fileOutputStream = openFileOutput(FILE_NAME,MODE_APPEND);
            PrintStream printStream = new PrintStream(fileOutputStream);
            printStream.println(s);
            printStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```


----------
3.**在内存存储内保存缓存文件(也就是将数据保存在/data/data/< package name>/cache目录下的文件中)**

如果只是要缓存一些数据，而不是要持久的保存它，那么应该使用getCacheDir()方法来打开一个File对象，它代表了应用程序要保存临时缓存文件的内部目录。

当设备的内部存储空间不足的时候，Android可能会删除这些缓存文件来回收存储空间。但是，不应该依赖系统来给你清理这些文件，应该始终自己来维护缓存文件，并且要把存储空间的耗费限定在合理的范围内，如1MB。当用户卸载应用程序时，这些文件会被删除。

getCacheDir()：获取应用程序的缓存目录文件夹，即/data/data/< package name>/cache目录。

将上面的Demo改为向应用程序的缓存目录下下程序只需：

//向文件写入数据：
            FileOutputStream fileOutputStream = openFileOutput(FILE_NAME,MODE_APPEND);
改为：
FileOutputStream fileOutputStream = 
new FileOutputStream(getCacheDir()+ File.separator+FILE_NAME);

//从文件读取数据：
FileInputStream fileInputStream = openFileInput(FILE_NAME);
改为：
FileInputStream fileInputStream =
 new FileInputStream(getCacheDir()+ File.separator+FILE_NAME);

注：

 - openFileOutput()和openFileInput()函数属于Android中定义的函数，文件默认保存的路径为/data/data/< package name>/files目录下。
 - new FileOutputStream()和new FileInputStream()函数属于Java中定义的函数，没有默认的文件保存路径，故需要制定文件路径。
 

转载自：http://blog.csdn.net/tianjf0514/article/details/8271114#
参考自：《疯狂Java讲义》
