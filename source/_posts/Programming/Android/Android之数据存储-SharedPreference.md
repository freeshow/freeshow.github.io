---
title: Android之数据存储--SharedPreference
date: 2016-07-23 23:56:43
tags: Android
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

SharedPreference是Android提供的一种轻量级的数据存储方式，主要用来存储一些简单的配置信息，例如，默认欢迎语，登录用户名和密码等。其以键值对(key-value对)的方式存储,使得我们能很方便进行读取和存入。

1.读取Preferences数据：
SharedPreference接口主要负责读取应用程序的Preferences数据，它提供了如下常用的方法：

 - boolean contains(String key)：判断SharedPreferences是否包含特定key的数据。
 - abstract Map< String,?> getAll() : 获取SharedPreferences数据里全部的key-value对。
 - boolean getXxx(String key,Xxx defValue)：其中Xxx可以是boolean、float、int、long、String等各种基本类型的值。

2.写入Preferences数据：
SharedPreferences接口本身并没有提供写入数据的能力，而是通过SharedPreferences的内部接口，SharedPreferences调用edit()方法即可获取它所对应的Editor对象。Editor提供如下方法来向Preferences写入数据。

 - SharedPreferences.Editor clear()：清空SharedPreferences里所有数据。
 - SharedPreferences.Editor putXxx(String key,xxx value)：向SharedPreferences存入指定key对应的数据。
 - SharedPreferences.Editor remove(String key)：删除SharedPreferences里指定的key对应的数据项。
 - boolean commit()：当Editor编辑完成后，调用该方法提交修改。

从用法角度看，SharedPreferences和SharedPreferences.Editor组合起来非常像Map,其中SharedPreferences负责根据key读取数据，而SharedPreferences.Editor则用于写入数据。


----------
SharedPreferences本身是一个接口，程序无法直接创建SharedPreferences实例，只能通过Context提供的getSharedPreferences(String name,int mode)方法来获取SharedPreferences实例，该方法第一个参数为：保存键值对的文件名，第二个参数支持如下几个值。

 - Context.MODE_PRIVATE：指定SharedPreferences数据只能被本应用程序读写。
 - Context.MODE_WORLD_READABLE：指定SharedPreferences数据能被其他应用程序读，但不能写。
 -  Context.MODE_WORLD_WRITEABLE：指定SharedPreferences数据能被其他应用程序读写。
 - 使用0或者MODE_PRIVATE作为默认的操作权限模式。

提示：从Android 4.2 开始，Android不在推荐使用Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE，因为这两种模式允许其他应用程序来读或写本应用创建的数据，因此容易导致安全漏洞。如果应用程序确实需要把内部数据暴露出来提供给其他应用访问，则应该使用ContentProvider.

下面是一个保存用户名和密码的例子，先看效果：
![这里写图片描述](http://img.blog.csdn.net/20150826144313327)

刚开始输入用户名和密码，点击保存密码，登录后关闭程序，让后在启动程序，则自动保存了用户名和密码。

代码如下：

activity_login.xml

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".LoginActivity">
    <EditText
        android:id="@+id/userName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="UserName:"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="PassWord:"
        android:id="@+id/passWord"
        android:password="true"/>
    <CheckBox
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Save Password"
        android:id="@+id/savePassWord"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Login"
        android:textAllCaps="false"
        android:id="@+id/login"
        android:layout_gravity="center"/>
</LinearLayout>

```


LoginActivity.java

```
public class LoginActivity extends AppCompatActivity {
    private EditText userName;
    private EditText passWord;
    private CheckBox savePassword;
    private Button loginBtn;

    private SharedPreferences preferences;
    private SharedPreferences.Editor editor;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        userName = (EditText) findViewById(R.id.userName);
        passWord = (EditText) findViewById(R.id.passWord);
        savePassword = (CheckBox) findViewById(R.id.savePassWord);
        loginBtn = (Button) findViewById(R.id.login);

        preferences = getSharedPreferences("UserInfo",MODE_PRIVATE);
        editor = preferences.edit();

        savePassword.setChecked(preferences.getBoolean("savePassword", false));
        if (savePassword.isChecked())
        {
	        //读取数据
            userName.setText(preferences.getString("userName", null));
            passWord.setText(preferences.getString("passWord", null));
        }
        loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
		            //写入数据
	                editor.putString("userName",     userName.getText().toString());
                    editor.putString("passWord", passWord.getText().toString());
                    editor.putBoolean("savePassword", savePassword.isChecked());
                    //// 千万不要忘记commit。否则，如果文件存在，那么写入的内容不会保存。如果文件不存在，则不会创建文件。
                    editor.commit();
                Intent intent = new Intent(LoginActivity.this,HomeActivity.class);
                startActivity(intent);
            }
        });
    }
}

```

点击登录按钮跳转到主界面的代码就不写了，主界面显示“Login Sucess”.

运行完上面程序，点击Android Studio的Android Device Monitor中的DDMS的 File Explorer面板，然后展开浏览树，如图：
![这里写图片描述](http://img.blog.csdn.net/20150826150453984)

发现SharedPreferences数据总是保存在/data/data/< package name>/shared_prefs目录下，因此SharedPreferences数据保存在内部存储器中，SharedPreferences数据总是以XML格式保存。通过File Explorer面板的导出文件按钮导出该XML文档，打开该XML文档卡伊看到如下文件内容：

```
<?xml version="1.0" encoding="UTF-8" standalone="true"?>
-<map>
	<boolean value="true" name="savePassword"/>
	<string name="userName">Android</string>
	<string name="passWord">123456</string>
</map>
```

几点说明：

- SharedPreferences的获取有两种方法：一是上面提到的通过Activity自带（本质来讲是Context的）的getSharedPreferences方法，可以得到SharedPreferences对象。这种方法的好处是可以指定保存的xml文件名。另一种是通过PreferenceManager.getSharedPreferences(Context)获取SharedPreferences对象。这种方法不能指定保存的xml文件名，文件名使用默认的：<package name>+"_preferences.xml"的形式，不过如果在一个包里面采用这种方式需要保存多个这样的xml文件，可能会乱掉。建议采用第一种指定xml文件名的形式。

- 数据的存入必须通过SharedPreferences对象的编辑器对象Editor来实现，存入（put）之后与写入数据库类似一定要commit。

----------
记录应用程序的使用次数Demo:

```
public class MainActivity extends Activity
{
	SharedPreferences preferences;
	@Override
	public void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		preferences = getSharedPreferences("count"
			, MODE_PRIVATE);
		// 读取SharedPreferences里的count数据
		int count = preferences.getInt("count", 0);
		// 显示程序以前使用的次数
		Toast.makeText(this, "程序以前被使用了" + count + "次。"
				, Toast.LENGTH_LONG).show();
		SharedPreferences.Editor editor = preferences.edit();
		// 存入数据
		editor.putInt("count", ++count);
		// 提交修改
		editor.commit();
	}
}
```


----------
为SharedPreferences添加监听器：

```
// 如果test.xml文件不存在，则会在editor.commit()时创建。
    	// 如果没有调用commit()方法，写入的内容不会保存，而且不会创建文件。
    	SharedPreferences prefs = getSharedPreferences("test", MODE_PRIVATE);
    	SharedPreferences.Editor editor = prefs.edit();
    	editor.putBoolean("bVal", true);
    	editor.putString("sVal", "allei");
    	editor.putFloat("fVal", 10.12f);
    	editor.putInt("iVal", 1000);
    	editor.putLong("lVal", 100l);
    	editor.commit(); // 千万不要忘记commit。否则，如果文件存在，那么写入的内容不会保存。如果文件不存在，则不会创建文件。
    	
    	Log.i("xxxxxxxxxx", "bVal: " + prefs.getBoolean("bVal", false));
    	Log.i("xxxxxxxxxx", "sVal: " + prefs.getString("sVal", ""));
    	Log.i("xxxxxxxxxx", "fVal: " + prefs.getFloat("fVal", 0.0f));
    	Log.i("xxxxxxxxxx", "iVal: " + prefs.getInt("iVal", 0));
    	Log.i("xxxxxxxxxx", "lVal: " + prefs.getLong("lVal", 0l));
    	
    	prefs.registerOnSharedPreferenceChangeListener(myListener); // 注册
    	
    	editor = prefs.edit();
    	editor.putString("sVal", "boss");
    	editor.commit();  // 不要忘记commit。否则不会触发监听器。
    	
    	prefs.unregisterOnSharedPreferenceChangeListener(myListener);  // 解除注册，不会再相应改变
    	editor = prefs.edit();
    	editor.putString("sVal", "bosssss");
    	editor.commit(); // 因为监听器已经解除注册，所以不会再触发监听器。
    	
```

监听器定义：

```
private OnSharedPreferenceChangeListener myListener = new OnSharedPreferenceChangeListener() {
		@Override
		public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {
			Log.i("xxxxxxxx", "key: " + key + " = " + sharedPreferences.getString(key, "default"));
		}
	};
```

参考自：http://blog.csdn.net/ydpl2007/article/details/7590349
			 http://blog.csdn.net/zhangqijie001/article/details/5838213