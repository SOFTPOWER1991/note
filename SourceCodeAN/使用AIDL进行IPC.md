使用AIDL进行跨进程间通信


本文主要包含如下内容：

1. AIDL的介绍
2. AIDL的适用场景
3. 如何定义AIDL
4. 如何调用AIDL
5. 传递复杂对象
6. 调用IPC方法

# AIDL的介绍

AIDL（Android接口定义语言），可以利用它定义客户端与服务使用进程间通信IPC进行相互通信时都认可的编程接口。

# AIDL的适用场景

1. 只有允许不同应用的客户端用IPC方式访问服务，并且想要在服务中处理多线程时，才有必要适用AIDL。
2. 如果您不需要执行跨越不同应用的并发IPC，就应该通过实现一个Binder创建接口；或者，如果你想执行IPC，但根本不需要处理多线程，则适用Messenger类来实现接口。

# 如何定义AIDL接口

必须使用Java编程语法在.aidl文件中定义AIDL接口，然后将它保存在托管服务的应用以及任何其他绑定到服务的应用在源代码内。

您开发每一个包含.aidl文件的应用时，Android SDK工具都会生成一个基于该.aidl文件的IBinder接口，并将其保存在项目的gen/目录中。服务必须视情况实现IBinder接口。然后客户端应用便可绑定到该服务，并调用IBinder中的方法来执行IPC。 

使用AIDL创建绑定服务的步骤：

1. 创建.aidl文件
> 此文件定义带有方法签名的编程接口。
2. 实现接口
> Android SDK 工具基于您的 .aidl 文件，使用 Java 编程语言生成一个接口。此接口具有一个名为 Stub 的内部抽象类，用于扩展 Binder 类并实现 AIDL 接口中的方法。您必须扩展 Stub 类并实现方法。
3. 向客户端公开该接口
> 实现 Service 并重写 onBind() 以返回 Stub 类的实现。

下面对三个步骤逐一进行详细讲解：

## 1. 创建.aidl文件

先看实例代码：

```
// IRemoteService.aidl
package com.example.android;

// Declare any non-default types here with import statements

/** Example service interface */
interface IRemoteService {
    /** Request the process ID of this service, to do evil things with it. */
    int getPid();

    /** Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}

```

可以看到AIDL文件其实就是一个Java中的接口文件。那么定义这个接口文件有哪些注意事项呢：

* AIDL支持的数据类型：
	* Java中的所有原语类型（int, long, char, boolean等）
	* String
	* CharSequence
	* List,list中的数据也必须支持以上类型
	* Map，map中的数据也必须支持以上类型

* 如果你使用的数据类型是不是上述类型中的，你必须为每个附加类型加入一个import语句，即使这些类型是在与您的接口相同的软件包中定义。
* 方法可以附带零个或多个参数，返回值或空值。
* 所有非原语参数都需要指示数据走向的方向指标。可以是in,out或inout
* .aidl文件中包括的所有代码注释都包含在生成的IBinder接口中（import和package语句之前的注释除外）
* 支持方法，您不能公开AIDL中的静态字段。


## 2. 实现接口

先看一个实现接口的具体代码：

```
private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
    public int getPid(){
        return Process.myPid();
    }
    public void basicTypes(int anInt, long aLong, boolean aBoolean,
        float aFloat, double aDouble, String aString) {
        // Does nothing
    }
};
```

上述代码实现了一个mBinder它是Stub类的一个实例，用于定义服务的RPC接口。

在我们定义了IRmoteService后，，AndroidSDK工具会生成一个以.aidl文件命名的.java接口文件。生成的接口包括一个Stub的子类，用于声明.aidl文件中的所有方法。

如需实现.aidl生成的接口，请扩展生成的Binder接口并实现从.aidl文件继承的方法。

## 3. 向客户端公开该接口

你为服务实现该接口后，就要向客户端生成该几口，以便客户端进程绑定。要向您的服务公开该接口，请扩展Service并实现onBind()，以返回一个类实例，这个类实现了生成的Stub。

以下是一个向客户端公开IRmoteService示例接口的服务示例。

```
public class RemoteService extends Service {
    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // Return the interface
        return mBinder;
    }

    private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
        public int getPid(){
            return Process.myPid();
        }
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
            float aFloat, double aDouble, String aString) {
            // Does nothing
        }
    };
}

```

这个时候，当客户端（activity）调用bindService()以链接此服务时，客户端的onServiceConnected()回调会接受服务的onBind()方法赶回的mBinder实例。

客户端还必须具有对 interface 类的访问权限，因此如果客户端和服务在不同的应用内，则客户端的应用 src/ 目录内必须包含 .aidl 文件（它生成 android.os.Binder 接口 — 为客户端提供对 AIDL 方法的访问权限）的副本。
当客户端在 onServiceConnected() 回调中收到 IBinder 时，它必须调用YourServiceInterface.Stub.asInterface(service) 以将返回的参数转换成 YourServiceInterface 类型。例如：

具体示例代码：

```
IRemoteService mIRemoteService;
private ServiceConnection mConnection = new ServiceConnection() {
    // Called when the connection with the service is established
    public void onServiceConnected(ComponentName className, IBinder service) {
        // Following the example above for an AIDL interface,
        // this gets an instance of the IRemoteInterface, which we can use to call on the service
        mIRemoteService = IRemoteService.Stub.asInterface(service);
    }

    // Called when the connection with the service disconnects unexpectedly
    public void onServiceDisconnected(ComponentName className) {
        Log.e(TAG, "Service has unexpectedly disconnected");
        mIRemoteService = null;
    }
};

```

# 通过IPC传递对象

我们可以通过IPC接口把某个类从一个进程发送到另一个进程。但是，这个被传递的对象必须实现Parcelable接口，因为Android系统可通过它将对象分解成可编组到各进程的原语。

通过IPC传递对象，你需要做如下操作：

1. 让该对象实现Parcelable接口

代码如下：

```
import android.os.Parcel;
import android.os.Parcelable;

public final class Rect implements Parcelable {
    public int left;
    public int top;
    public int right;
    public int bottom;

    public static final Parcelable.Creator<Rect> CREATOR = new
Parcelable.Creator<Rect>() {
        public Rect createFromParcel(Parcel in) {
            return new Rect(in);
        }

        public Rect[] newArray(int size) {
            return new Rect[size];
        }
    };

    public Rect() {
    }

    private Rect(Parcel in) {
        readFromParcel(in);
    }

    public void writeToParcel(Parcel out) {
        out.writeInt(left);
        out.writeInt(top);
        out.writeInt(right);
        out.writeInt(bottom);
    }

    public void readFromParcel(Parcel in) {
        left = in.readInt();
        top = in.readInt();
        right = in.readInt();
        bottom = in.readInt();
    }
}

```

2. 创建一个声明可打包类的.aidl文件
代码如下：

```
package android.graphics;

// Declare Rect so AIDL can find it and knows that it implements
// the parcelable protocol.
parcelable Rect;

```

# 调用IPC方法

调用类必须执行如下步骤，才能调用使用AIDL定义的远程接口：

1. 在项目src/目录加入.aidl文件
2. 声明一个IBinder接口实例（基于AIDL生成）
3. 实现ServiceConnection
4. 调用Context.bindService(),以传入您的ServiceConnection实现。
5. 在您的 onServiceConnected() 实现中，您将收到一个 IBinder 实例（名为 service）。调用YourInterfaceName.Stub.asInterface((IBinder)service)，以将返回的参数转换为 YourInterface 类型。
6. 调用您在接口上定义的方法。您应该始终捕获 DeadObjectException 异常，它们是在连接中断时引发的；这将是远程方法引发的唯一异常。
7. 如需断开连接，请使用您的接口实例调用 Context.unbindService()。

具体示例代码如下：

```
public static class Binding extends Activity {
    /** The primary interface we will be calling on the service. */
    IRemoteService mService = null;
    /** Another interface we use on the service. */
    ISecondary mSecondaryService = null;

    Button mKillButton;
    TextView mCallbackText;

    private boolean mIsBound;

    /**
     * Standard initialization of this activity.  Set up the UI, then wait
     * for the user to poke it before doing anything.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.remote_service_binding);

        // Watch for button clicks.
        Button button = (Button)findViewById(R.id.bind);
        button.setOnClickListener(mBindListener);
        button = (Button)findViewById(R.id.unbind);
        button.setOnClickListener(mUnbindListener);
        mKillButton = (Button)findViewById(R.id.kill);
        mKillButton.setOnClickListener(mKillListener);
        mKillButton.setEnabled(false);

        mCallbackText = (TextView)findViewById(R.id.callback);
        mCallbackText.setText("Not attached.");
    }

    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the service object we can use to
            // interact with the service.  We are communicating with our
            // service through an IDL interface, so get a client-side
            // representation of that from the raw service object.
            mService = IRemoteService.Stub.asInterface(service);
            mKillButton.setEnabled(true);
            mCallbackText.setText("Attached.");

            // We want to monitor the service for as long as we are
            // connected to it.
            try {
                mService.registerCallback(mCallback);
            } catch (RemoteException e) {
                // In this case the service has crashed before we could even
                // do anything with it; we can count on soon being
                // disconnected (and then reconnected if it can be restarted)
                // so there is no need to do anything here.
            }

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_connected,
                    Toast.LENGTH_SHORT).show();
        }

        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mKillButton.setEnabled(false);
            mCallbackText.setText("Disconnected.");

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                    Toast.LENGTH_SHORT).show();
        }
    };

    /**
     * Class for interacting with the secondary interface of the service.
     */
    private ServiceConnection mSecondaryConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // Connecting to a secondary interface is the same as any
            // other interface.
            mSecondaryService = ISecondary.Stub.asInterface(service);
            mKillButton.setEnabled(true);
        }

        public void onServiceDisconnected(ComponentName className) {
            mSecondaryService = null;
            mKillButton.setEnabled(false);
        }
    };

    private OnClickListener mBindListener = new OnClickListener() {
        public void onClick(View v) {
            // Establish a couple connections with the service, binding
            // by interface names.  This allows other applications to be
            // installed that replace the remote service by implementing
            // the same interface.
            Intent intent = new Intent(Binding.this, RemoteService.class);
            intent.setAction(IRemoteService.class.getName());
            bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
            intent.setAction(ISecondary.class.getName());
            bindService(intent, mSecondaryConnection, Context.BIND_AUTO_CREATE);
            mIsBound = true;
            mCallbackText.setText("Binding.");
        }
    };

    private OnClickListener mUnbindListener = new OnClickListener() {
        public void onClick(View v) {
            if (mIsBound) {
                // If we have received the service, and hence registered with
                // it, then now is the time to unregister.
                if (mService != null) {
                    try {
                        mService.unregisterCallback(mCallback);
                    } catch (RemoteException e) {
                        // There is nothing special we need to do if the service
                        // has crashed.
                    }
                }

                // Detach our existing connection.
                unbindService(mConnection);
                unbindService(mSecondaryConnection);
                mKillButton.setEnabled(false);
                mIsBound = false;
                mCallbackText.setText("Unbinding.");
            }
        }
    };

    private OnClickListener mKillListener = new OnClickListener() {
        public void onClick(View v) {
            // To kill the process hosting our service, we need to know its
            // PID.  Conveniently our service has a call that will return
            // to us that information.
            if (mSecondaryService != null) {
                try {
                    int pid = mSecondaryService.getPid();
                    // Note that, though this API allows us to request to
                    // kill any process based on its PID, the kernel will
                    // still impose standard restrictions on which PIDs you
                    // are actually able to kill.  Typically this means only
                    // the process running your application and any additional
                    // processes created by that app as shown here; packages
                    // sharing a common UID will also be able to kill each
                    // other's processes.
                    Process.killProcess(pid);
                    mCallbackText.setText("Killed service process.");
                } catch (RemoteException ex) {
                    // Recover gracefully from the process hosting the
                    // server dying.
                    // Just for purposes of the sample, put up a notification.
                    Toast.makeText(Binding.this,
                            R.string.remote_call_failed,
                            Toast.LENGTH_SHORT).show();
                }
            }
        }
    };

    // ----------------------------------------------------------------------
    // Code showing how to deal with callbacks.
    // ----------------------------------------------------------------------

    /**
     * This implementation is used to receive callbacks from the remote
     * service.
     */
    private IRemoteServiceCallback mCallback = new IRemoteServiceCallback.Stub() {
        /**
         * This is called by the remote service regularly to tell us about
         * new values.  Note that IPC calls are dispatched through a thread
         * pool running in each process, so the code executing here will
         * NOT be running in our main thread like most other things -- so,
         * to update the UI, we need to use a Handler to hop over there.
         */
        public void valueChanged(int value) {
            mHandler.sendMessage(mHandler.obtainMessage(BUMP_MSG, value, 0));
        }
    };

    private static final int BUMP_MSG = 1;

    private Handler mHandler = new Handler() {
        @Override public void handleMessage(Message msg) {
            switch (msg.what) {
                case BUMP_MSG:
                    mCallbackText.setText("Received from service: " + msg.arg1);
                    break;
                default:
                    super.handleMessage(msg);
            }
        }

    };
}

```




