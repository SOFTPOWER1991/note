题目如下：

一个很简单的需求：点击页面上的按钮后更新TextView的内容

给出如下代码，请找出如下代码有那些不对的地方：

```
public class HandlerActivity extends AppCompatActivity {

    private TextView mTextMessage;
    private Button button;

    private UpdateHandler updateHandler;

    class UpdateHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

            if (msg.what == 1) {
                mTextMessage.setText("update====" );
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextMessage = (TextView) findViewById(R.id.message);

        button = (Button) findViewById(R.id.btn_start);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                updateTextViewContent();
            }
        });
    }


    private void updateTextViewContent() {
        new Thread() {
            @Override
            public void run() {
                super.run();
                try {
                    Thread.sleep(4000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                updateHandler = new UpdateHandler();
                Message message = new Message();
                message.what = 1;

                updateHandler.sendMessage(message);
            }
        }.start();
    }
}
```

一道很基础的面试题，感觉跟`大家一起来找茬`好相似！

这道题考察的地方大概有这么几个：

1. 不能在子线程更新UI；
2. Handler使用不当可能引起的内存泄露；
3. Message的优化！
4. 在子线程中创建Handler，需要为这个Handler准备Looper。
5. 可能出现异常：在Handler把消息处理完了，但是页面销毁了，这是可能就会出现各种异常；

下面来看详细的解析：

## 1. 不能在子线程更新UI

上面的代码中在子线程中创建了一个Handler，Handler内部有更新UI的动作，这其实到最后也还是在子线程更新UI。这样就会抛出如下错误：

```
android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
```

因此，需要将创建Handler的动作，移到主线程中去。

## 2. Handler使用不当可能引起的内存泄露

Handler 被作为 Activity 引用，如果为非静态内部类，则会引用外部类对象。当 Activity finish 时，Handler可能并未执行完，从而引起 Activity 的内存泄漏。在上面的例子中，就是点击按钮后，立即关闭页面这时候线程可能还没有跑完，但是handler却持有了activity的引用。这就造成了内存泄露，改进方法如下：

```
 static class UpdateHandler extends Handler {
        private final WeakReference<MainActivity> mActivity;

        public UpdateHandler(MainActivity activity) {
            mActivity = new WeakReference<MainActivity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

            MainActivity instance = mActivity.get();

            Log.e("handler", msg.what + "====");
            if (msg.what == 1) {
                instance.mTextMessage.setText("update====");
            }
        }
    }
```

## 3. Message的优化

在上面的代码中，Message message = new Message()。这样做是没有错误的，但是会涉及到性能的问题。在Message的源码开头这这段注释，下面我们读一下这段代码：

```
/**
 * 
 * <p class="note">While the constructor of Message is public, the best way to get
 * one of these is to call {@link #obtain Message.obtain()} or one of the
 * {@link Handler#obtainMessage Handler.obtainMessage()} methods, which will pull
 * them from a pool of recycled objects.</p>
 */
```

注意：尽管Message的构造方法是公开的，但是最好的使用方式是调用Message.obtain()或者Handler.obtainMessage()方法，这会从一个全局的对象池里拉出一个回收的消息来使用。避免分配对象。

优化方案：

```
Message message = Message.obtain();
```

# 4. 在子线程中创建Handler，需要为这个Handler准备Looper

上面的代码运行起来的话，其实会抛出如下异常：

```
java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()
```

在子线程中创建Handler是需要为Handler准备一个消息循环器Looper。因此，如果要在子线程中创建Handler的话，需要这样做：

```
Looper.prepare();

...

Handler handler = new Handler();

...

Looper.loop();
```

## 5. 可能出现异常

在Handler把消息处理完了后，但是页面销毁了，这个时候可能Handler会更新UI，但是比如TextView、ImageView之类的资源引用不见了，就会抛出异常。

解决方案：

在OnDestory的时候，移除消息队列中待处理的消息

```
@Override
protected void onDestroy() {
   super.onDestroy();
   //参数为null,表示移除所有回调和消息
   updateHandler.removeCallbacksAndMessages(null);
}
```

这就是上面的代码的优化过程！

综上所述，优化过的代码如下：

```
public class MainActivity extends AppCompatActivity {

    private  TextView mTextMessage;
    private Button button;

    private UpdateHandler updateHandler;

    static class UpdateHandler extends Handler {
        private final WeakReference<MainActivity> mActivity;

        public UpdateHandler(MainActivity activity) {
            mActivity = new WeakReference<MainActivity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

            MainActivity instance = mActivity.get();

            if (msg.what == 1) {
                instance.mTextMessage.setText("update====");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextMessage = (TextView) findViewById(R.id.message);

        updateHandler = new UpdateHandler(MainActivity.this);

        button = (Button) findViewById(R.id.btn_start);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            	updateTextViewContent()；
            }
        });
    }
    
    private void updateTextViewContent() {
        new Thread() {
            @Override
            public void run() {
                super.run();
                try {
                    Thread.sleep(4000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                Message message = Message.obtain();
                message.what = 1;
                updateHandler.sendMessage(message);
            }
        }.start();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        updateHandler.removeCallbacksAndMessages(null);
    }
}

```


