想了好久终于想明白了，记下来！

先说下Android 消息机制涉及到的类：`Handler`、`Looper`、`MessageQueue`、`Message`、`ThreadLocal`。

我之前在想这个问题的时候想过的几个问题，这几个问题搞明白了，消息机制就搞定了：

> 1. 在哪儿发的消息？
> 2. 消息发到哪儿去了？
> 3. 消息最后怎么被处理的？
> 4. 怎么就把子线程发的消息变换到了主线程？
> 5. 哪儿来的Looper？
> 6. Looper的loop方法又是在哪儿调用的呢？
> 7. 上面各个类都是什么作用呢？
> 8. Handler在哪儿(主线程还是子线程)？
> 9. Looper在哪儿(主线程还是子线程)？
> 10. MessageQueue在哪儿(主线程还是子线程)？
> 11. ThreadLocal在哪儿(主线程还是子线程)？

本文主要涉及如下内容：

1. Android消息机制在实际开发中的应用；
2. 消息机制的源码分析；
3. 上述问题答案揭秘；

# Android消息机制在实际开发中的应用

正式开始之前，先看一个Handler在多线程中具体使用的案例：

> 一个很简单的需求：点击页面上的按钮后更新TextView的内容。

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



在主线程创建一个Handler，然后开启一个工作线程待工作线程休眠4000毫秒后，发出一个Message然后在主线程更新TextView的内容。

# 消息机制源码分析

结合上面的例子，下面就挨个把这些问题搞明白：

在子线程中发出的消息，通过如下代码：

```
Message message = Message.obtain();
message.what = 1;
updateHandler.sendMessage(message);
```
追踪sendMessage的源码：

> updateHandler.sendMessage(message);

发现经过一串的调用sendMessage->sendMessageDelayed->sendMessageAtTime，最后走到了sendMessageAtTime这个方法中。

下面一睹尊容：

```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

sendMessageAtTime中将Message入队，也就是放入到MessageQueue，MessageQueue是一个由链表实现的消息队列。

在sendMessageAtTime中调用了enqueueMessage（queue,msg,uptimeMillis）方法，在这个方法中进行了入队操作：

```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

而真正的入队操作则到了MessageQueue中，代码如下：

```
boolean enqueueMessage(Message msg, long when) {
        

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
看到prev=p、p = p.next、msg.next=p、prev.next=msg， 这不就是链表吗？但是感觉很奇怪的是，为什么名字是MessageQueue呢？

到此为止：

> 消息已经发出去了，也存到了MessageQueue中等待被处理了，那么问题来了：
> 1. MessageQueue是被谁处理的？
> 2. 被怎么处理的？

这个时候Looper就隆重登场了！

在主线程中创建了我们的Handler
>updateHandler = new UpdateHandler(MainActivity.this);

追踪源码，此处调用的是Handler默认的构造方法，源码如下：

```
/**
     * Default constructor associates this handler with the {@link Looper} for the
     * current thread.
     *
     * If this thread does not have a looper, this handler won't be able to receive messages
     * so an exception is thrown.
     */
    public Handler() {
        this(null, false);
    }
```
注释告诉我们:

> 1. 默认的构造方法已经关联了一个当前线程的Looper，当前线程是主线程，那么就使用的是主线程的Looper，在ActivityThread启动的时候，已经为我们准备好了一个mainthreadLooper.
> 2. 如果是我们自己启动的线程，不自己准备Looper的话就会抛出异常：Can't create handler inside thread that has not called Looper.prepare()
源码之下，了无秘密，这就是这个异常的出处,看代码：

```
public Handler(Callback callback, boolean async) {
        //删掉部分无关代码
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
       	......
    }
```

在主线程启动的时候，已经为我们准备了一个默认的主线程使用的Looper，在ActivityThread中，有如下代码：

```
public static void main(String[] args) {
        ......

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        ......
    }
```
其中的两句代码，就可以帮我们揭秘——上面提到的两个问题：

1. MessageQueue中的消息是被谁处理的？
2. 怎么被处理的？

其中的代码：
```
Looper.prepareMainLooper();
...
if (sMainThreadHandler == null) {
   sMainThreadHandler = thread.getHandler();
}
Looper.loop();
```
这就是为什么，我们在Activity中的OnCreate方法中创建了Handler但是并没有调用Looper.prepre（）和Looper.loop()但是没问题的原因，使用的是主线程Looper。

下面看看两个方法，先看看prepareMainLooper：

```
    //为当前线程创建一个Looper，Android环境已经为你准备好了，不需要再调用这个方法。 
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```
这个方法中调用了prepare(false)方法：
```
 private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        //创建一个Looper，并设置给ThreadLocal
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
这时候ThreadLocal出来了，在这儿我们提出问题：

1. ThreadLocal是什么？
2. 他有什么作用？
暂且按下不表，待后文见分晓！

很好奇new Looper(quitAllowed)这句代码做了什么，点击去看看：

```
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

一个private方法，我们无法new出一个Looper来，其中包含一个MessageQueue和当前的线程！因为当前线程是主线程，所以MessageQueue，Looper都是在主线程中。

在prepareMainLooper方法的最后，调用了myLooper()方法：

```
/**
     * 返回当前线程关联的Looper对象，如果当前线程没有关联Looper对象那么就返回null.
     */
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```
至此，Looper.prepare()方法分析完毕，已经为主线程准备好了Looper。

接下来看，Looper.loop()方法：
```
 /**
     *  轮询当前线程的消息队列。在轮询完成之后确保调用了quit()方法。
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

       	......
       	//死循环
        for (;;) {
            Message msg = queue.next(); // 可能会阻塞，从MessageQueue中获取下一个消息
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

           ......
            try {
            	//msg.target 就是Handler，从MessageQueue中获取到的Message，被分发给了Handler中的dispatchMessage方法
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

           ......
        }
    }
```

在Looper.loop()方法中，我们找到了答案：

> 阻塞式的获取消息队列中的Message。然后，调用Handler中的dispatchMessage进行Message的分发。

下面看看dispatchMessage的代码：

```
	/**
     * 在这儿处理系统Message
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
哈哈，HandlerMessage好眼熟，此处应该有掌声！

```
	/**
     * 子类应该实现这个方法，来接收消息
     */
    public void handleMessage(Message msg) {
    }
```
最后消息，在子类的handlerMessage方法中被处理了！

这样Android中的消息机制的源码就分析完毕了，下面来回答刚开始提出的问题！

# 问题揭秘

## 1. Message在哪儿发出的

在子线程中发出的消息，通过如下代码：

```
Message message = Message.obtain();
message.what = 1;
updateHandler.sendMessage(message);
```

## 2. Message发到哪儿去了？

Message发送到了MessageQueue中，一个消息队列中！

## 3. Message最后怎么被处理的？

Message被Looper的Loop方法从MessageQueue中借助next()方法取出，分发给Handler中的dispatchMessage方法，进行处理。这是一个典型的生产者、消费者模型！

## 4. 怎么就把子线程发出的消息变换到了主线程中？
子线程把Message发送到了MessageQueue中，MessageQueue在Looper中初始化，Looper在主线程中，所以MessageQueue也在主线程中，消息就这样被变换到了主线程中。

## 5. 哪儿来的Looper？

主线程中Looper是在ActivityTread创建的时候，Looper.preperMainLooper()创建的

## 6. Looper的loop方法在哪儿调用的呢？

主线程中的Loop方法在ActivityThread的main方法中调用，如果是自己的线程中的loop方法，需要自己手动调用。 

## 7. 上面各个类都是什么作用呢？
* Handler，Android中的消息处理机制
* Looper，消息泵，用来从MessageQueue中读取消息
* MessageQueue，消息队列，内部用链表实现
* Message,主线程和子线程之间的通信载体
* ThreadLocal: 每一个线程可以直接把数据存取到ThreadLocal中，而数据是隔离的互不干扰，用作线程的数据隔离。

## 8. Handler在哪儿(主线程还是子线程)？

Handler在主线程中创建，因此Handler在主线程中

## 9.Looper在哪儿(主线程还是子线程)？

Looper在主线程中创建，因此Looper在主线程中

## 10. MessageQueue在哪儿(主线程还是子线程)？

MessageQueue在Looper的构造方法中初始化，因此MessageQueue也在主线程中

## 11. ThreadLocal在哪儿(主线程还是子线程)？

在Looper中，用来存取Looper，因此也在主线程中

至此，Android消息机制的源码就分析完毕！

源码之下，了无秘密！一个字儿，了然！


