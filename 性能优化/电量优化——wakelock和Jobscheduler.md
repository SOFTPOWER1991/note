
我们对Android引用进行电量优化的目的：延长用户设备的电池使用寿命，减少电池的耗电！提供更好的用户体验。

那么在Android设备中都有哪些地方耗电呢？

# 哪些地方耗电量大

* 屏幕
* 蜂窝网络
* WIFI
* 唤醒CPU、GPU进行工作
* 闹钟

# 电量优化用到的检测工具

* battery Historian 电量记录分析工具

github主页：[battery-historian](https://github.com/google/battery-historian)

通过ADB获取的数据，通过使用Battery Historian工具分析处理后，得到的html结果文件，用浏览器可以直接查看的。

# 常用手段

## 手段一：为了省电，有些工作可以放当手机插上电源的时候去做。往往这样的情况非常多。

像这些不需要及时地和用户交互的操作可以放到后面处理。比如：360手机助手，当充上电的时候，才会自动清理手机垃圾，自动备份上传图片、联系人等到云端。
总之，一些任务可以放到设备充电的时候执行.

## 手段二：向服务器同步大量数据时，在设备电量的情况下可以等到连接了WiFi后再去做相应的操作


## 手段三： wake_lock

系统为了节省电量，CPU在没有任务忙的时候就会自动进入休眠。有任务需要唤醒CPU高效执行的时候，就会给CPU加wake_lock锁。

wake_lock锁主要是相对系统的休眠而言的，意思就是我的程序给CPU加了这个锁那系统就不会休眠了，这样做的目的是为了全力配合我们程序的运行。有的情况如果不这么做就会出现一些问题，比如微信等及时通讯的心跳包会在熄屏不久后停止网络访问等问题。所以微信里面是有大量使用到了wake_lock锁。

这块儿经常会犯得错误：

我们很容易去唤醒CPU来干货，但是很容易忘记释放wake_lock。

```
mWakelock.acquire();//唤醒CPU
mWakelock.release();//记得释放CPU锁
```

解决方案：使用PowerManager的API

第一步：

```
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
<uses-permission android:name="android.permission.WAKE_LOCK"></uses-permission>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"></uses-permission>
```

第二步：

```
public class MainActivity extends AppCompatActivity {
    TextView wakelock_text ;
    PowerManager pw;
    PowerManager.WakeLock mWakelock;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        wakelock_text = (TextView)findViewById(R.id.wakelock_text);
        pw = (PowerManager) getSystemService(POWER_SERVICE);
        mWakelock = pw.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,"mywakelock");

    }
    public void execut(View view){
        wakelock_text.setText("正在下载....");
        IBinder b;b.get
        for(int i=0;i<10;i++){
            mWakelock.acquire();//唤醒CPU
            wakelock_text.append("连接中……");
//            wakelock_text.append("");
            //下载
            if(isNetWorkConnected()) {
                new SimpleDownloadTask().execute();
            }else{
                wakelock_text.append("没有网络连接。");
            }
        }
    }

    private boolean isNetWorkConnected() {
        ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
        return (activeNetworkInfo!=null&&activeNetworkInfo.isConnected());
    }

    /**
     *  Uses AsyncTask to create a task away from the main UI thread. This task creates a
     *  HTTPUrlConnection, and then downloads the contents of the webpage as an InputStream.
     *  The InputStream is then converted to a String, which is displayed in the UI by the
     *  onPostExecute() method.
     */
    private static final String LOG_TAG = "ricky";
    private class SimpleDownloadTask extends AsyncTask<Void, Void, String> {

        @Override
        protected String doInBackground(Void... params) {
            try {
                // Only display the first 50 characters of the retrieved web page content.
                int len = 50;

                URL url = new URL("https://www.baidu.com");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setReadTimeout(10000); // 10 seconds
                conn.setConnectTimeout(15000); // 15 seconds
                conn.setRequestMethod("GET");
                //Starts the query
                conn.connect();
                int response = conn.getResponseCode();
                Log.d(LOG_TAG, "The response is: " + response);
                InputStream is = conn.getInputStream();

                // Convert the input stream to a string
                Reader reader = new InputStreamReader(is, "UTF-8");
                char[] buffer = new char[len];
                reader.read(buffer);
                return new String(buffer);

            } catch (IOException e) {
                return "Unable to retrieve web page.";
            }
        }

        @Override
        protected void onPostExecute(String result) {
            wakelock_text.append("\n" + result + "\n");
            releaseWakeLock();
        }
    }

    private void releaseWakeLock(){
        if(mWakelock.isHeld()){
            mWakelock.release();//记得释放CPU锁
            wakelock_text.append("释放锁！");
        }
    }

}
```

有一些意外的情况，比如小米手机是做了同步心跳包（心跳对齐）(如果超过了这个同步的频率就会被屏蔽掉或者降频)，所有的app后台唤醒频率不能太高，比如每隔2S中去请求。

使用：AlarmManager，

AlarmManager ,Android上实现轮询机制的方法,AlarmManager在Android中主要用来定时处理一个事件或是定期处理一个事件。

总之:

1.	关键逻辑的执行过程，就需要Wake Lock来保护。如断线重连重新登陆
2.	休眠的情况下如何唤醒来执行任务？用AlarmManager。如推送消息的获取

## 手段四：大量高频次的CPU唤醒及操作，我们最好把这些操作集中处理。我们可以采取一些算法来解决。可以借鉴谷歌的精髓，JobScheduler/GCM

JobScheduler的使用，我写了另一篇博客来讲解：[JobScheduler详解](JobScheduler详解.md)







