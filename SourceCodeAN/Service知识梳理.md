今日主题：

> Service知识最强梳理

本文主要包含如下主题：

1. Service是什么？
2. Service可以做什么？
3. 如何创建一个Service？
4. Service的启动方式探究
5. Service与用户交互的反馈方式
6. Service的使用注意事项
7. Service的工作原理

下面逐个讨论上面的问题：

# 1. Service是什么？

Service 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。

# 2. Service可以做什么？

服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。

# 3. Service的具体使用？

服务的启动方式一共两种：启动服务和绑定服务。

## 3.0 使用清单文件声明服务

如同Activity（以及其他组件）一样，必须在应用的清单文件中声明所有服务。 

如下：

```
<application
    ......
    android:theme="@style/AppTheme">
    ......

    <service
        android:name=".CommonService"
        android:enabled="true"
        android:exported="true" />
    <service
        android:name=".MyIntentService"
        android:exported="false"></service>
</application>
```

## 3.1 创建启动服务

启动服务由另一个组件通过调用startService()启动，这会导致调用服务的onStartCommand()方法。

服务启动之后，其生命周期即独立于启动它的组件，并且可以在后台无限期地运行，即使启动服务的组件已被销毁也不受影响。 因此，服务应通过调用 stopSelf() 结束工作来自行停止运行，或者由另一个组件通过调用stopService() 来停止它。

应用组件（如 Activity）可以通过调用 startService() 方法并传递 Intent 对象（指定服务并包含待使用服务的所有数据）来启动服务。服务通过 onStartCommand() 方法接收此 Intent。

> 注意：默认情况下，服务与服务声明所在的应用运行于同一进程，而且运行于该应用的主线程中。 因此，如果服务在用户与来自同一应用的 Activity 进行交互时执行密集型或阻止性操作，则会降低 Activity 性能。 为了避免影响应用性能，您应在服务内启动新线程。

可以扩展两个类来创建启动服务：

### Service

这是适用于所有服务的基类。扩展此类时，必须创建一个用于执行所有服务工作的新线程，因为默认情况下，服务将使用应用的主线程，这会降低应用正在运行的所有 Activity 的性能。

代码如下：

```
public class HelloService extends Service {
  private Looper mServiceLooper;
  private ServiceHandler mServiceHandler;

  // Handler that receives messages from the thread
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
      @Override
      public void handleMessage(Message msg) {
          // Normally we would do some work here, like download a file.
          // For our sample, we just sleep for 5 seconds.
          try {
              Thread.sleep(5000);
          } catch (InterruptedException e) {
              // Restore interrupt status.
              Thread.currentThread().interrupt();
          }
          // Stop the service using the startId, so that we don't stop
          // the service in the middle of handling another job
          stopSelf(msg.arg1);
      }
  }

  @Override
  public void onCreate() {
    // Start up the thread running the service.  Note that we create a
    // separate thread because the service normally runs in the process's
    // main thread, which we don't want to block.  We also make it
    // background priority so CPU-intensive work will not disrupt our UI.
    HandlerThread thread = new HandlerThread("ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND);
    thread.start();

    // Get the HandlerThread's Looper and use it for our Handler
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
  }

  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();

      // For each start request, send a message to start a job and deliver the
      // start ID so we know which request we're stopping when we finish the job
      Message msg = mServiceHandler.obtainMessage();
      msg.arg1 = startId;
      mServiceHandler.sendMessage(msg);

      // If we get killed, after returning from here, restart
      return START_STICKY;
  }

  @Override
  public IBinder onBind(Intent intent) {
      // We don't provide binding, so return null
      return null;
  }

  @Override
  public void onDestroy() {
    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show();
  }
}

```
### IntentService

这是 Service 的子类，它使用工作线程逐一处理所有启动请求。如果您不要求服务同时处理多个请求，这是最好的选择。 您只需实现 onHandleIntent() 方法即可，该方法会接收每个启动请求的 Intent，使您能够执行后台工作。

```
public class HelloIntentService extends IntentService {

  /**
   * A constructor is required, and must call the super IntentService(String)
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      try {
          Thread.sleep(5000);
      } catch (InterruptedException e) {
          // Restore interrupt status.
          Thread.currentThread().interrupt();
      }
  }
}

```
### 启动服务

将启动服务的Intent传递给startService()，从Activity或者其他应用组件启动服务。

```
Intent intent = new Intent(); intent.setClass(MainActivity.this , CommonService.class); startService(intent);
```

### 停止服务

启动服务必须管理自己的生命周期。除非系统回收内存资源，否则系统不会停止或销毁服务，而且服务在onStartCommand()返回后会继续运行。因此，服务必须通过调用stopSelf()自行停止运行，或者由另一个组件通过调用stopService()来停止它。

一旦请求使用stopSelf()或stopService()停止服务，系统就会尽快销毁服务。

如果服务同时处理多个 onStartCommand() 请求，则您不应在处理完一个启动请求之后停止服务，因为您可能已经收到了新的启动请求（在第一个请求结束时停止服务会终止第二个请求）。为了避免这一问题，您可以使用stopSelf(int) 确保服务停止请求始终基于最近的启动请求。也就说，在调用 stopSelf(int) 时，传递与停止请求的 ID 对应的启动请求的 ID（传递给 onStartCommand() 的 startId）。然后，如果在您能够调用 stopSelf(int)之前服务收到了新的启动请求，ID 就不匹配，服务也就不会停止。

> 为了避免浪费系统资源和消耗电池电量，应用必须在工作完成之后停止其服务。 如有必要，其他组件可以通过调用 stopService() 来停止服务。即使为服务启用了绑定，一旦服务收到对 onStartCommand() 的调用，您始终仍须亲自停止服务。

## 3.2 创建绑定服务

绑定服务允许应用组件通过调用 bindService() 与其绑定，以便创建长期连接（通常不允许组件通过调用startService() 来启动它）。

要创建绑定服务，必须实现 onBind() 回调方法以返回 IBinder，用于定义与服务通信的接口。然后，其他应用组件可以调用 bindService() 来检索该接口，并开始对服务调用方法。服务只用于与其绑定的应用组件，因此如果没有组件绑定到服务，则系统会销毁服务（您不必按通过 onStartCommand() 启动的服务那样来停止绑定服务）。

要创建绑定服务，首先必须定义指定客户端如何与服务通信的接口。 服务与客户端之间的这个接口必须是 IBinder的实现，并且服务必须从 onBind() 回调方法返回它。一旦客户端收到 IBinder，即可开始通过该接口与服务进行交互。

多个客户端可以同时绑定到服务。客户端完成与服务的交互后，会调用 unbindService() 取消绑定。一旦没有客户端绑定到该服务，系统就会销毁它。

# 4. 服务的生命周期探究

服务生命周期根据启动方式分为两种：

## 4.1 启动服务的生命周期

startService方式启动的服务，其生命周期如下：

startService()--> onCreate()-->onStartCommand()-->Service running-->onDestory()



## 4.2 绑定服务的生命周期

bindService方式启动的服务，其生命周期如下：

bindService()-->onCreate()-->onBind()-->onUnbind()-->onDestory()

# 5. 绑定服务

绑定服务是客户端-服务器接口中的服务器。绑定服务可让组件（例如 Activity）绑定到服务、发送请求、接收响应，甚至执行进程间通信 (IPC)。 绑定服务通常只在为其他应用组件服务时处于活动状态，不会无限期在后台运行。

绑定服务是 Service 类的实现，可让其他应用与其绑定和交互。要提供服务绑定，您必须实现 onBind()回调方法。该方法返回的 IBinder 对象定义了客户端用来与服务进行交互的编程接口。

客户端可通过调用 bindService() 绑定到服务。调用时，它必须提供 ServiceConnection 的实现，后者会监控与服务的连接。bindService() 方法会立即无值返回，但当 Android 系统创建客户端与服务之间的连接时，会对 ServiceConnection 调用 onServiceConnected()，向客户端传递用来与服务通信的IBinder。

多个客户端可同时连接到一个服务。不过，只有在第一个客户端绑定时，系统才会调用服务的 onBind() 方法来检索 IBinder。系统随后无需再次调用 onBind()，便可将同一 IBinder 传递至任何其他绑定的客户端。

当最后一个客户端取消与服务的绑定时，系统会将服务销毁（除非 startService() 也启动了该服务）。

当您实现绑定服务时，最重要的环节是定义您的 onBind() 回调方法返回的接口。您可以通过几种不同的方法定义服务的 IBinder 接口：

## 扩展Binder类

如果服务是供您的自有应用专用，并且在与客户端相同的进程中运行（常见情况），则应通过扩展Binder 类并从 onBind() 返回它的一个实例来创建接口。客户端收到 Binder 后，可利用它直接访问 Binder 实现中乃至 Service 中可用的公共方法。
如果服务只是您的自有应用的后台工作线程，则优先采用这种方法。 不以这种方式创建接口的唯一原因是，您的服务被其他应用或不同的进程占用。

设置方法：

1. 在您的服务中，创建一个可满足下列任一要求的Binder实例：
	* 包含客户端课调用的公共方法
	* 返回当前Service实例，其中包含客户端可调用的公共方法
	* 或返回由服务承载的其他类的实例，
2. 从onBind()回调方法返回此Binder实例
3. 在客户端中，从onServiceConnected()回调方法接受Binder，并使用提供的方法调用绑定服务。

具体实例：

LocalBinder 为客户端提供 getService() 方法，以检索 LocalService 的当前实例。这样，客户端便可调用服务中的公共方法。 例如，客户端可调用服务中的 getRandomNumber()。

```
public class LocalService extends Service {
    // Binder given to clients
    private final IBinder mBinder = new LocalBinder();
    // Random number generator
    private final Random mGenerator = new Random();

    /**
     * Class used for the client Binder.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    public class LocalBinder extends Binder {
        LocalService getService() {
            // Return this instance of LocalService so clients can call public methods
            return LocalService.this;
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    /** method for clients */
    public int getRandomNumber() {
      return mGenerator.nextInt(100);
    }
}
```

点击按钮时，以下这个 Activity 会绑定到 LocalService 并调用 getRandomNumber() ：

public class BindingActivity extends Activity {
    LocalService mService;
    boolean mBound = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Bind to LocalService
        Intent intent = new Intent(this, LocalService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }

    /** Called when a button is clicked (the button in the layout file attaches to
      * this method with the android:onClick attribute) */
    public void onButtonClick(View v) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call were something that might hang, then this request should
            // occur in a separate thread to avoid slowing down the activity performance.
            int num = mService.getRandomNumber();
            Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
        }
    }

    /** Defines callbacks for service binding, passed to bindService() */
    private ServiceConnection mConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            LocalBinder binder = (LocalBinder) service;
            mService = binder.getService();
            mBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            mBound = false;
        }
    };
}

上例说明了客户端如何使用 ServiceConnection 的实现和 onServiceConnected() 回调绑定到服务。下文更详细介绍了绑定到服务的过程。

> 注：在上例中，onStop() 方法将客户端与服务取消绑定。 客户端应在适当时机与服务取消绑定，如附加说明中所述。
> 
## 使用Messenger

如需让接口跨不同的进程工作，则可使用 Messenger 为服务创建接口。服务可以这种方式定义对应于不同类型 Message 对象的 Handler。此 Handler 是 Messenger 的基础，后者随后可与客户端分享一个 IBinder，从而让客户端能利用 Message 对象向服务发送命令。此外，客户端还可定义自有Messenger，以便服务回传消息。
这是执行进程间通信 (IPC) 的最简单方法，因为 Messenger 会在单一线程中创建包含所有请求的队列，这样您就不必对服务进行线程安全设计。




## 使用AIDL 

之前采用 Messenger 的方法实际上是以 AIDL 作为其底层结构。 如上所述，Messenger 会在单一线程中创建包含所有客户端请求的队列，以便服务一次接收一个请求。 不过，如果您想让服务同时处理多个请求，则可直接使用 AIDL。 在此情况下，您的服务必须具备多线程处理能力，并采用线程安全式设计。
如需直接使用 AIDL，您必须创建一个定义编程接口的 .aidl 文件。Android SDK 工具利用该文件生成一个实现接口并处理 IPC 的抽象类，您随后可在服务内对其进行扩展。

# 6. Service与用户交互的反馈方式

##  6.1 向用户发送通知
##  6.2 在前台运行服务

# 7. Service的使用注意事项
# 8. Service的工作原理





