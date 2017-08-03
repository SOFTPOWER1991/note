本文主要讲解Window和WindowManger的工作原理，其中包含如下内容：

1. Window和WindowManger简介
2. 通过WindowManager添加一个简单的Window以及Window的Flags属性
3. Window的内部机制
	* 3.1 Window的添加过程
	* 3.2 Window的删除过程
	* 3.3 Window的更新过程

# 1. Window和WindowManager简介

Window和WindowManager工作原理涉及到的类有：
Window、PhoneWindow、WindowManager、WindowManagerService。

在之前学习Android的View事件分发机制的时候，了解到：
> 单击事件由Window传递给DecorView，然后再由DecorView传递给我们的View。我们在Activity中设置的视图的方法setContentView在底层也是通过Window来完成的。

其实在Android中所有的视图都是通过Window来呈现的，不管是Activity、Dialog还是Toast，他们的视图实际都是附加在Window上来呈现，因此Window实际是View的直接管理者。

从源码中可以看到Window是一个抽象类，它的具体实现是PhoneWindow。创建一个Window是很简单的事儿，只需要通过WindowManager即可完成。WindowManager是外接访问Window的入口，Window的具体实现位于WindowManagerService中，WindowManger和WindowManagerService的交互是一个IPC的过程。

# 2. 通过WindowManager添加一个简单的Window以及Window的Flags属性

```
wManager = (WindowManager) getApplicationContext().getSystemService(
                Context.WINDOW_SERVICE);
        mParams = new WindowManager.LayoutParams();
        mParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ALERT;// 系统提示window
        mParams.format = PixelFormat.TRANSLUCENT;// 支持透明
        //mParams.format = PixelFormat.RGBA_8888;
        mParams.flags |= WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;// 焦点
        mParams.width = 490;//窗口的宽和高
        mParams.height = 160;
        mParams.x = 100;//窗口位置的偏移量
        mParams.y = 0;
        //mParams.alpha = 0.1f;//窗口的透明度
        myView = new MyView(this);
        myView.setOnClickListener(this);
			wManager.addView(myView);
```

上述代码将一个View添加到屏幕的（100，300）的位置。WindowManger.LayoutParams中的flags和type这两个参数很重要，下面进行说明:

Flags参数表示Window的属性，有很多选项，他们可以控制Window的显示特性，有如下几个常用的：

* FLAG_NOT_FOCUSABLE
> 表示Window不需要获取焦点，也不需要接收各种输入事件，此标记会同时启用FLAG——NOT_TOUCH_MODAL,最终事件会直接传递给下层的具有焦点的Window。 

* FLAG_NOT_TOUCH_MODAL
> 系统会将当前Window区域以外的单击事件传递给底层的Window，当前Window区域以内的单击事件则自己处理。一般都需要开启此标记，否则其他Window将无法收到单击事件。
* FLAG_SHOW_WHEN_LOCKED
> 开启此模式可以让Window显示在锁屏的界面上。

Type表示Window的类型，Window有三种类型，分别是应用Window、子Window和系统Window。

* 应用类Window对应着一个Activity。
* 子Window不能单独存在，需要附属在特定的父Window之中，比如常见的一个Dialog就是一个子Window。
* 系统Window需要声明权限才能创建Window，比如Toast和系统状态栏这些都是系统Window.

Window的分层：
> 每个Window都有对应的z-ordered，层级大的会覆盖在层级小的Window上面。
> 在三类Window中：
> 1. 应用层Window的层级范围是 1~99;
> 2. 子Window的层级范围是1000~1999；
> 3. 系统Window的层级范围是2000~2999；
这些层级范围对应着WindowManger.LayoutParams的type参数。如果想要Window位于所有Window的最顶层，那么采用较大的层级即可。很显然系统Window的层级是最大的，而且系统层级有很多值，一般我们可以选择TYPE_SYSTEM_OVERLAY或者TYPE_SYSTEM_ERROR。

如果要使用TYPE_SYSTEM_ERROR，只需要为type参数指定这个层级即可：mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR；同时声明权限：
\<users-permission android:name="android.permission.SYSTEM_ALERT_WINDOW">.因为系统类型的Window是需要检查权限的，如果不在AndroidManifest中使用相应的权限，那么创建Window时就会报错。

Window所能提供的功能很简单，常用的只有三个方法，即：添加View、更新View、删除View。这三个方法定义在ViewManager中，而WindowManger继承自ViewManger。

ViewManager代码如下：
```
/** Interface to let you add and remove child views to an Activity. To get an instance
  * of this class, call {@link android.content.Context#getSystemService(java.lang.String) Context.getSystemService()}.
  */
public interface ViewManager
{
    /**
     * Assign the passed LayoutParams to the passed View and add the view to the window.
     * <p>Throws {@link android.view.WindowManager.BadTokenException} for certain programming
     * errors, such as adding a second view to a window without removing the first view.
     * <p>Throws {@link android.view.WindowManager.InvalidDisplayException} if the window is on a
     * secondary {@link Display} and the specified display can't be found
     * (see {@link android.app.Presentation}).
     * @param view The view to be added to this window.
     * @param params The LayoutParams to assign to view.
     */
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```

# 3. Window的内部机制
Window是一个抽象的概念，每一个Window都对应着一个View和一个ViewRootImpl，Window和View通过ViewRootImpl来建立联系，因此Window并不是实际存在的，它是以View的形式存在的。这点从WindowManager的定义看一看出来，它提供三个接口方法addView、updateViewLayout以及removeView都是针对View的，这说明View才是Window存在的实体。在实际使用中也是无法直接访问Window的，必须通过WindowManager。而Window的内部机制，其实也就是Window的添加、删除、更新过程。

## 3.1 Window的添加过程

Window的添加过程需要通过WindowManger的addView来实现，WindowManger是一个接口，它的真正实现是WindowMangerImpl类。在WindowMangerImpl中Window的三个操作实现如下：

```
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view, false);
}

```

在上述代码中发现，这三个操作又转交给了mGlobal 一个叫做WindowManagerGlobal的家伙处理，WindowManagerGlobal以工厂的形式向外提供自己的实例。代码如下：

```
private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
```
下面看看WindowManagerGlobal添加View的过程,大致会经历如下三步：

**1. 检查参数是否合法，如果是子Window那么还需要调整一些布局参数**
**2. 创建ViewRootImpl并将View添加到列表中**
**3. 通过ViewRootImpl来更新界面并完成Window的添加过程**

代码如下：

**1. 检查参数是否合法，如果是子Window那么还需要调整一些布局参数**

```
 if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        if (parentWindow != null) {
            parentWindow.adjustLayoutParamsForSubWindow(wparams);
        } else {
            // If there's no parent, then hardware acceleration for this view is
            // set from the application's hardware acceleration setting.
            final Context context = view.getContext();
            if (context != null
                    && (context.getApplicationInfo().flags
                            & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
                wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
            }
        }
```
**2. 创建ViewRootImpl并将View添加到列表中**

在WindowMangerGlobal内部有如下几个列表：
```
private final ArrayList<View> mViews = new ArrayList<View>();
private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
private final ArrayList<WindowManager.LayoutParams> mParams =
       new ArrayList<WindowManager.LayoutParams>();
private final ArraySet<View> mDyingViews = new ArraySet<View>();
```
* mViews：存储的是所有Window所对应的View
* mRoots：存储的是所有Window所对相应的ViewRootImpl
* mParams：存储的是所有Window所对应的布局参数，
* mDyingViews：存储了那些正在被删除的View对象，或者说是那些已经调用removeView方法但是删除操作还未完成的Window对象。

在addView中通过如下方式将Window的一系列对象添加到列表中：

```
root = new ViewRootImpl(view.getContext(), display);

view.setLayoutParams(wparams);

mViews.add(view);
mRoots.add(root);
mParams.add(wparams);
```
**3. 通过ViewRootImpl来更新界面并完成Window的添加过程**
这个步骤由ViewRootImpl的setView方法来完成，在View绘制流程了解到，View的绘制过程是由ViewRootImpl来完成的，这里当然也不例外，在setView内部会通过requestLayout来完成异步刷新请求。而真正的View绘制入口在scheduleTraversals中，代码如下：

```
@Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }
```

之后在setView中通过WindowSession来完成Window的添加过程。看代码：
```
try {
        mOrigWindowType = mWindowAttributes.type;
        mAttachInfo.mRecomputeGlobalAttributes = true;
        collectViewAttributes();
        res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                getHostVisibility(), mDisplay.getDisplayId(),
                mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                mAttachInfo.mOutsets, mInputChannel);
    } catch (RemoteException e) {
        mAdded = false;
        mView = null;
        mAttachInfo.mRootView = null;
        mInputChannel = null;
        mFallbackEventHandler.setView(null);
        unscheduleTraversals();
        setAccessibilityFocus(null, null);
        throw new RuntimeException("Adding window failed", e);
    } finally {
        if (restore) {
            attrs.restore();
        }
    }
```
mWindowSession类型是IWindowSession，它是一个Binder对象，真正的实现类是Session，也就是Window的添加过程是一次IPC调用。在Session内部会通过WindowMangerService来实现Window的添加。

```
public int addToDisplay(IWindow window, 
								int seq , 
								WindowManger.LayoutParams attrs, 
								int viewVisibility, 
								int displayId, 
								Rect outContentInsets, 
								InputChannel outInputChannel){
 return 
 	mService.addWindow(this , window, seq , attrs, viewVisibility, displayId, outContentInsets, outInputChannel);
}
```

## 3.2 Window的删除过程

Window的删除过程和添加过程一样，都是先通过WindowMangerImpl后，再进一步通过WindowMangerGlobal来实现的。下面是WindowManagerGlobal的removeView的实现：
```
 public void removeView(View view, boolean immediate) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }

        synchronized (mLock) {
            int index = findViewLocked(view, true);
            View curView = mRoots.get(index).getView();
            removeViewLocked(index, immediate);
            if (curView == view) {
                return;
            }

            throw new IllegalStateException("Calling with view " + view
                    + " but the ViewAncestor is attached to " + curView);
        }
    }
```

removeView方法中，首先通过findViewLocked来查找待删除的View的索引，这个查找过程就是建立的数组遍历，然后再调用removeViewLocked来做进一步的删除，代码如下：
```

    private void removeViewLocked(int index, boolean immediate) {
        ViewRootImpl root = mRoots.get(index);
        View view = root.getView();

        if (view != null) {
            InputMethodManager imm = InputMethodManager.getInstance();
            if (imm != null) {
                imm.windowDismissed(mViews.get(index).getWindowToken());
            }
        }
        boolean deferred = root.die(immediate);
        if (view != null) {
            view.assignParent(null);
            if (deferred) {
                mDyingViews.add(view);
            }
        }
    }
```
removeViewLocked是通过ViewRootImpl来完成删除操作的。在WindowManager中提供了两种删除接口removeView和removeViewImmediate，分别表示异步删除和同步删除。
在异步删除的过程中，由ViewRootImpl调用die方法来完成，die方法只是发送了一个请求删除的消息后就立刻返回了，这时候View并没有完成删除操作，所以最后会将其添加到mDyingViews中，mDyingViews表示待删除的View列表。代码如下：
```
boolean die(boolean immediate) {
        // Make sure we do execute immediately if we are in the middle of a traversal or the damage
        // done by dispatchDetachedFromWindow will cause havoc on return.
        if (immediate && !mIsInTraversal) {
            doDie();
            return false;
        }

        if (!mIsDrawing) {
            destroyHardwareRenderer();
        } else {
            Log.e(mTag, "Attempting to destroy the window while drawing!\n" +
                    "  window=" + this + ", title=" + mWindowAttributes.getTitle());
        }
        mHandler.sendEmptyMessage(MSG_DIE);
        return true;
    }
```
在die方法中，只是做了简单的判断，如果是异步删除，就发送一个MSG_DIE的消息，ViewRootImpl中的Handler会处理此消息并调用doDie方法。在doDie内部会调用dispatchDetachedFromWindow方法，也就是真正删除View的逻辑所在。

## 3.3 Window的更新过程

Window的更新过程，还是要看WindowMangerGlobal的updateViewLayout方法，代码如下：
```
public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;

        view.setLayoutParams(wparams);

        synchronized (mLock) {
            int index = findViewLocked(view, true);
            ViewRootImpl root = mRoots.get(index);
            mParams.remove(index);
            mParams.add(index, wparams);
            root.setLayoutParams(wparams, false);
        }
    }
```
updateViewLayout方法，做了如下操作：
1. 首先需要更新View的LayoutParams并替换掉老的LayoutParams
2. 接着更新ViewRootImpl的LayoutParams，这一步是通过ViewRootImpl的setLayoutParams方法来实现的。
3. 在ViewRootImpl中会通过scheduleTraversals方法来对View重新布局，包括:测量、布局、重绘三个过程。

除了View本身的重绘以外，ViewRootImpl还会通过WindowSession来更新Window的视图，这个过程最终是由WindowManagerService的relayoutWindow()来具体实现的，也是一个IPC过程。

