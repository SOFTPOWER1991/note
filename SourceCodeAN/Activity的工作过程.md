在我们启动Activity的时候，可以通过如下代码操作：

```
Intent intent = new Intent(this , ActivityTest.class);
startActivity(intent);
```
我之前在学习的时候，想过的几个问题：

1. 系统内部到底是如何启动一个Activity的呢？
2. 新启动的Activity是在何时创建的？
3. Activity的各个回调方法是在什么时候被系统回调的？

本文主要讲解Activity的工作过程。

在启动Activity的时候，我们都会调用startActivity方法，那就从这个方法入手，看看startActivity的代码：

```
@Override
public void startActivity(Intent intent, @Nullable Bundle options) {
    if (options != null) {
        startActivityForResult(intent, -1, options);
    } else {
        // Note we want to go through this call for compatibility with
        // applications that may have overridden the method.
        startActivityForResult(intent, -1);
    }
}
```

可以看到startActivity最终都会调用startActivityForResult方法，接下来看startActivityForResult方法，代码如下:
```
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
    if (mParent == null) {
        options = transferSpringboardActivityOptions(options);
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, this,
                intent, requestCode, options);
        if (ar != null) {
            mMainThread.sendActivityResult(
                mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                ar.getResultData());
        }
        if (requestCode >= 0) {
            // If this start is requesting a result, we can avoid making
            // the activity visible until the result is received.  Setting
            // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
            // activity hidden during this time, to avoid flickering.
            // This can only be done when a result is requested because
            // that guarantees we will get information back when the
            // activity is finished, no matter what happens to it.
            mStartedActivity = true;
        }

        cancelInputsAndStartExitTransition(options);
        // TODO Consider clearing/flushing other event sources and events for child windows.
    } else {
        if (options != null) {
            mParent.startActivityFromChild(this, intent, requestCode, options);
        } else {
            // Note we want to go through this method for compatibility with
            // existing applications that may have overridden it.
            mParent.startActivityFromChild(this, intent, requestCode);
        }
    }
}
```

这里的mParent是指Activity，当Activity为null时，变开始进行正常逻辑的执行。在上面的代码中可以看到，调用了Instrumentation的execStartActivity方法来执行Activity的启动，其中有个参数mMainThread.getApplicationThread()这个参数，它的类型是ApplicationThread，它是ActivityThread的一个内部类，以后会用到。

**问题：Instrumentation有什么作用？**
> 真正初始化一个Activity的时候，由它来完成的。

接下来看Instrumentation的execStartActivity方法，代码如下：
```
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    if (am.match(who, null, intent)) {
                        am.mHits++;
                        if (am.isBlocking()) {
                            return requestCode >= 0 ? am.getResult() : null;
                        }
                        break;
                    }
                }
            }
        }
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            //检查启动Activity的结果，当无法启动时，会抛出异常。常见原因：待启动Activity没有在AndroidManifest中注册。
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

从上面代码中可以看到，启动Activity真正的实现由ActivityManagerNative.getDefault()的startActivity方法完成。**ActivityManagerService**继承自ActivityManagerNative，而ActivityManagerNative继承自Binder并实现了IActivityManager这个Binder接口，因此AMS也是一个Binder，它是IActivityManager的具体实现。因为ActivityManagerNative.getDefault()是一个IActivityManger类型的Binder对象，因此它的具体实例是AMS。在ActivityManagerNative中，AMS这个Binder对象采用单利模式，第一次调用它的get方法它会通过create方法来初始化AMS这个Binder 对象，在后续调用中则直接返回之前创建的对象，代码如下：

```
 /**
     * Retrieve the system's default/global activity manager.
     */
    static public IActivityManager getDefault() {
        return gDefault.get();
    }

 private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity");
            if (false) {
                Log.v("ActivityManager", "default service binder = " + b);
            }
            IActivityManager am = asInterface(b);
            if (false) {
                Log.v("ActivityManager", "default service = " + am);
            }
            return am;
        }
    };
```

从上面代码可以看到，Activity由ActivityManagerNative.getDefault()来启动，而ActivityManagerNative.getDefault()实际上是AMS，因此Activity的启动过程又转移到了AMS中，接下来看AMS中startActivity方法，代码如下：

```
   @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }
    
	@Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        enforceNotIsolatedCaller("startActivity");
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(),
                userId, false, ALLOW_FULL_ONLY, "startActivity", null);
        // TODO: Switch to user app stacks here.
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, null);
    }
```

Activity的启动过程转移到了ActivityStarter的startActivityMayWait方法中，在startActivityMayWait方法中又调用了startActivityLocked方法，

**卡住的地方：**

> 1. P324中，突然就漂移到了ActivityStackSupervisor中，我在源码中发现事实并不是这样的，不知道哪儿出了问题，进展不下去了？
> 2. ActivityStackSupervisor 和ActivityStack是如何关联起来的?
> 
> **不同SDK的源码很多地方都不一样了。**

接下来往后进行至IApplicationThread，它继承自IInterface，是一个Binder类型的接口。它内部包含了大量启动、停止Activity的接口，此外还包含了启动和停止服务的接口。它实现了大量和Activity以及Service启动停止相关的功能。

那IApplicationThread的实现者到底是谁呢？
> ActivityThread中的内部类ApplicationThread，ApplicationThread的定义，如下：

```
private class ApplicationThread extends ApplicationThreadNative {}

public abstract class ApplicationThreadNative extends Binder  implements IApplicationThread {}
```

可以看出ApplicationThread继承了ApplicationThreadNative，而ApplicationThreadNative实现了IApplicationThread接口。在ApplicationThreadNative内部有一个ApplicationThreadProxy类，最终Activity的启动过程最终回到了ApplicationThread中，在ApplicationThread通过scheduleLaunchActivity方法来启动Activity，代码如下：

```
 // we use token to identify this activity without having to send the
        // activity itself back to the activity manager. (matters more with ipc)
        @Override
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
                ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
                CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
                int procState, Bundle state, PersistableBundle persistentState,
                List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
                boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

            updateProcessState(procState, false);

            ActivityClientRecord r = new ActivityClientRecord();

            r.token = token;
            r.ident = ident;
            r.intent = intent;
            r.referrer = referrer;
            r.voiceInteractor = voiceInteractor;
            r.activityInfo = info;
            r.compatInfo = compatInfo;
            r.state = state;
            r.persistentState = persistentState;

            r.pendingResults = pendingResults;
            r.pendingIntents = pendingNewIntents;

            r.startsNotResumed = notResumed;
            r.isForward = isForward;

            r.profilerInfo = profilerInfo;

            r.overrideConfig = overrideConfig;
            updatePendingConfiguration(curConfig);

            sendMessage(H.LAUNCH_ACTIVITY, r);
        }
```
在上面代码中，发送了一个启动Activity的消息交给Handler处理H。sendMessage的作用是发送一个消息给H处理，代码如下：
```
private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        if (DEBUG_MESSAGES) Slog.v(
            TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
            + ": " + arg1 + " / " + obj);
        Message msg = Message.obtain();
        msg.what = what;
        msg.obj = obj;
        msg.arg1 = arg1;
        msg.arg2 = arg2;
        if (async) {
            msg.setAsynchronous(true);
        }
        mH.sendMessage(msg);
    }
```

接着看Handler H对消息的处理：
```
private class H extends Handler {
        public static final int LAUNCH_ACTIVITY         = 100;
        ......

        String codeToString(int code) {
           
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
               ......
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }
}
```
从上述代码可以看到，在代码中调用了handleLaunchActivity来处理Activity的启动过程，源码如下：
```
 private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
        // If we are getting ready to gc after going to the background, well
        // we are back active so skip it.
        unscheduleGcIdler();
        mSomeActivitiesChanged = true;

        if (r.profilerInfo != null) {
            mProfiler.setProfiler(r.profilerInfo);
            mProfiler.startProfiling();
        }

        // Make sure we are running with the most recent config.
        handleConfigurationChanged(null, null);

        if (localLOGV) Slog.v(
            TAG, "Handling launch of " + r);

        // Initialize before creating the activity
        WindowManagerGlobal.initialize();

        Activity a = performLaunchActivity(r, customIntent);

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            Bundle oldState = r.state;
            handleResumeActivity(r.token, false, r.isForward,
                    !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason);

            if (!r.activity.mFinished && r.startsNotResumed) {
                // The activity manager actually wants this one to start out paused, because it
                // needs to be visible but isn't in the foreground. We accomplish this by going
                // through the normal startup (because activities expect to go through onResume()
                // the first time they run, before their window is displayed), and then pausing it.
                // However, in this case we do -not- need to do the full pause cycle (of freezing
                // and such) because the activity manager assumes it can just retain the current
                // state it has.
                performPauseActivityIfNeeded(r, reason);

                // We need to keep around the original state, in case we need to be created again.
                // But we only do this for pre-Honeycomb apps, which always save their state when
                // pausing, so we can not have them save their state when restarting from a paused
                // state. For HC and later, we want to (and can) let the state be saved as the
                // normal part of stopping the activity.
                if (r.isPreHoneycomb()) {
                    r.state = oldState;
                }
            }
        } else {
            // If there was an error, for any reason, tell the activity manager to stop us.
            try {
                ActivityManagerNative.getDefault()
                    .finishActivity(r.token, Activity.RESULT_CANCELED, null,
                            Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
    }
```
最终走到了performLaunchActivity方法完成了Activity对象的创建和启动过程，它主要完成了如下几个操作：

1. 从ActivityClientRecord中获取待启动的Activity的组件信息；
```
ActivityInfo aInfo = r.activityInfo;
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);
        }

        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }
```

2. 通过Instrumentation的newActivity方法使用类加载器创建Activity对象；

	```
	 Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }
	```

3. 通过LoadApk的makeApplication方法来尝试创建Application对象；
	```
	Application app = r.packageInfo.makeApplication(false, mInstrumentation);
	```
4. 通过ContextImpl对象并通过Activity的attach方法来完成一些重要数据的初始化；
	```
	Context appContext = createBaseContextForActivity(r, activity);
	CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
	Configuration config = new Configuration(mCompatConfiguration);
	if (r.overrideConfig != null) {
	    config.updateFrom(r.overrideConfig);
	}
	if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
	        + r.activityInfo.name + " with config " + config);
	Window window = null;
	if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
	    window = r.mPendingRemoveWindow;
	    r.mPendingRemoveWindow = null;
	    r.mPendingRemoveWindowManager = null;
	}
	activity.attach(appContext, this, getInstrumentation(), r.token,
	        r.ident, app, r.intent, r.activityInfo, title, r.parent,
	        r.embeddedID, r.lastNonConfigurationInstances, config,
	        r.referrer, r.voiceInteractor, window);
	
	if (customIntent != null) {
	    activity.mIntent = customIntent;
	}
	r.lastNonConfigurationInstances = null;
	activity.mStartedActivity = false;
	int theme = r.activityInfo.getThemeResource();
	if (theme != 0) {
	    activity.setTheme(theme);
	}
```
5. 调用Activity的onCreate方法

mInstrumentation.callActivityOnCreate(activity , r.state)，由于Activity的onCreate已经被调用了，这就意味着Activity已经完成了整个启动过程。




