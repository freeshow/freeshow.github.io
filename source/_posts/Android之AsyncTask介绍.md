---
title: Android之AsyncTask介绍
date: 2016-07-24 00:06:26
tags: Android
categories:
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## android AsyncTask介绍 ##
AsyncTask和Handler对比

 - AsyncTask实现的原理,和适用的优缺点
- AsyncTask是android提供的轻量级的异步类,可以直接继承AsyncTask,在类中实现异步操作,并提供接口反馈当前异步执行的程度(可以通过接口实现UI进度更新),最后反馈执行的结果给UI主线程.
  - 使用的优点: 简单,快捷，过程可控。
  - 使用的缺点: 在使用多个异步操作和并需要进行Ui变更时,就变得复杂起来.
 
 - Handler异步实现的原理和适用的优缺点
   - 在Handler 异步实现时,涉及到 Handler, Looper, Message,Thread四个对象，实现异步的流程是主线程启动Thread（子线程）àthread(子线程)运行并生成Message-àLooper获取Message并传递给HandleràHandler逐个获取Looper中的Message，并进行UI变更。
   - 使用的优点：(1)结构清晰，功能定义明确 (2)对于多个后台任务时，简单，清晰 
   - 使用的缺点：在单个后台异步处理时，显得代码过多，结构过于复杂（相对性）
   
**AsyncTask介绍**

Android的AsyncTask比Handler更轻量级一些，适用于简单的异步处理。
首先明确Android之所以有Handler和AsyncTask，都是为了不阻塞主线程（UI线程），且UI的更新只能在主线程中完成，因此异步处理是不可避免的。
 
Android为了降低这个开发难度，提供了AsyncTask。AsyncTask就是一个封装过的后台任务类，顾名思义就是异步任务。

AsyncTask直接继承于Object类，位置为android.os.AsyncTask。要使用AsyncTask工作我们要提供三个泛型参数，并重载几个方法(至少重载一个)。

AsyncTask定义了三种泛型类型 Params，Progress和Result。

- Params 启动任务执行的输入参数，比如HTTP请求的URL。
- Progress 后台任务执行的百分比。
- Result 后台执行任务最终返回的结果，比如String.
- 在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。

使用过AsyncTask 的同学都知道一个异步加载数据最少要重写以下这两个方法：

- doInBackground(Params…) 后台执行，比较耗时的操作都可以放在这里。注意这里不能直接操作UI。此方法在后台线程执行，完成任务的主要工作，通常需要较长的时间。在执行过程中可以调用publishProgress(Progress…)来更新任务的进度。publishProgress(Progress…)执行后会触发onProgressUpdate(Progress…) 方法，doInBackground(Params…)方法接受后会系统调用onPostExecute(Result) 方法。
- onPostExecute(Result)  相当于Handler 处理UI的方式，在这里面可以使用在doInBackground得到的结果处理操作UI。 此方法***在主线程执行***，任务执行的结果作为此方法的参数返回

有必要的话你还得重写以下这三个方法，但不是必须的：

- onProgressUpdate(Progress…)   可以使用进度条增加用户体验度。 此方法**在主线程执行**，用于显示任务执行的进度。
- onPreExecute()        这里是最终用户调用Excute时的接口，当任务执行之前开始调用此方法，可以在这里显示进度对话框。**Runs on the UI thread** before doInBackground.
- onCancelled()             用户调用取消时，要做的操作

一个异步任务的执行一般包括以下几个步骤：
1. execute(Params... params)，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。
 2. onPreExecute()，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。
 3. doInBackground(Params... params)，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。publishProgress(Progress... values)的任务是传递参数values到函数onProgressUpdate(Progress... values)的形参values中。
 4. onProgressUpdate(Progress... values)，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。
 5. onPostExecute(Result result) 接受doInBackground(Params... params)的函数返回结果传入到result中，当后台操作结束时，此方法将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。

使用AsyncTask类，以下是几条必须遵守的准则：

- Task的实例必须在UI thread中创建；
- execute方法必须在UI thread中调用；
- 不要手动调用onPreExecute(), onPostExecute(Result)，doInBackground(Params...), onProgressUpdate(Progress...)这几个方法；
- 该task只能被执行一次，否则多次调用时将会出现异常；

Demo: 是用异步任务下载网络资源。该Demo的界面布局很简单，只包含两个组件：一个文本框用于显示从网络下载的页面代码；一个按钮用于激发下载任务。

```
public class MainActivity extends Activity {
    private TextView textView;
    private Button downloadBtn;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        downloadBtn = (Button) findViewById(R.id.downloadBtn);

        downloadBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
	            //必须在UI线程中创建AsyncTask的实例
                DownTask downTask = new DownTask(MainActivity.this);
                try {
	                //必须在UI线程中调用AsyncTask实例的execute()方法
                    downTask.execute(new URL("http://www.csdn.net"));
                } catch (MalformedURLException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    class DownTask extends AsyncTask<URL,Integer,String>
    {
        ProgressDialog progressDialog;
        //记录已经读取行的数量
        int hasRead = 0;
        Context context;
		//构造函数
        public DownTask(Context context) {
            this.context = context;
        }
		//这里是最终用户调用Excute时的接口，当任务执行之前开始调用此方法，可以在这里显示进度对话框。在UI线程中执行,主要用于完成一些初始化工作。
        @Override
        protected void onPreExecute() {
	        //创建对话框
            progressDialog = new ProgressDialog(context);
            //设置对话框标题
            progressDialog.setTitle("任务正在进行中");
            //设置对话框显示的内容
            progressDialog.setMessage("任务正在执行中，敬请等待...");
            //设置对话框不能用取消按钮关闭
            progressDialog.setCancelable(false);
            //设置进度条的最大进度值
            progressDialog.setMax(100);
			//设置对话框的进度条风格 progressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
			//设置对话框的进度条是否显示进度
            progressDialog.setIndeterminate(false);
            //显示对话框
            progressDialog.show();
        }
		
		//后台耗时任务在这里执行，此处不可以更新UI界面
        @Override
        protected String doInBackground(URL... params) {
            StringBuilder sb = new StringBuilder();
            try {
                URLConnection conn = params[0].openConnection();
                BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                String line = null;
                while ((line = br.readLine()) != null)
                {
                    Thread.sleep(1000);
                    sb.append(line + "\n");
                    hasRead++;
	                //讲hasRead传到onProgressUpdate(Integer... values)函数中
                    publishProgress(hasRead);
                }
                //将返回值传递到onPostExecute(String s)函数中
                return sb.toString();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return null;
        }
		//此函数在UI线程中执行，用于更新进度条
        @Override
        protected void onProgressUpdate(Integer... values) {
            progressDialog.setProgress(values[0]);
            textView.setText("已经读取了【"+ values[0] +"】行!");
        }
		//在UI线程中执行，主要用于更新UI
        @Override
        protected void onPostExecute(String s) {
            textView.setText(s);
            progressDialog.dismiss();
        }
    }
}

```

效果：
![这里写图片描述](http://img.blog.csdn.net/20150821164158166)
下载完成后将下载的代码显示在TextView中