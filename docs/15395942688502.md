官方文档链接： 

https://developer.android.com/reference/android/view/ViewTreeObserver


final ViewTreeObserver viewTreeObserver = view.getViewTreeObserver(); 


ViewTreeObserver 注册一个观察者来监听视图树，当视图树的布局、视图树的焦点、视图树将要绘制、视图树滚动等发生改变时，ViewTreeObserver都会收到通知，ViewTreeObserver不能被实例化，因为它是由当前View层级提供的，可以调用View.getViewTreeObserver()来获得。 

ViewTreeObserver继承关系： 

```
	public final class ViewTreeObserver 
	
	extends Object 
	
	java.lang.Object
	   ↳ 
	android.view.ViewTreeObserver 
```

ViewTreeObserver的源码： 

```
	// 
	// Source code recreated from a .class file by IntelliJ IDEA 
	// (powered by Fernflower decompiler) 
	// 
	
	package android.view; 
	
	public final class ViewTreeObserver { 
	    ViewTreeObserver() { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnWindowAttachListener(ViewTreeObserver.OnWindowAttachListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnWindowAttachListener(ViewTreeObserver.OnWindowAttachListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnWindowFocusChangeListener(ViewTreeObserver.OnWindowFocusChangeListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnWindowFocusChangeListener(ViewTreeObserver.OnWindowFocusChangeListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnGlobalFocusChangeListener(ViewTreeObserver.OnGlobalFocusChangeListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnGlobalFocusChangeListener(ViewTreeObserver.OnGlobalFocusChangeListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnGlobalLayoutListener(ViewTreeObserver.OnGlobalLayoutListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    /** @deprecated */ 
	    @Deprecated 
	    public void removeGlobalOnLayoutListener(ViewTreeObserver.OnGlobalLayoutListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnGlobalLayoutListener(ViewTreeObserver.OnGlobalLayoutListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnPreDrawListener(ViewTreeObserver.OnPreDrawListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnPreDrawListener(ViewTreeObserver.OnPreDrawListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnDrawListener(ViewTreeObserver.OnDrawListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnDrawListener(ViewTreeObserver.OnDrawListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnScrollChangedListener(ViewTreeObserver.OnScrollChangedListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnScrollChangedListener(ViewTreeObserver.OnScrollChangedListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void addOnTouchModeChangeListener(ViewTreeObserver.OnTouchModeChangeListener listener) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public void removeOnTouchModeChangeListener(ViewTreeObserver.OnTouchModeChangeListener victim) { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public boolean isAlive() { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public final void dispatchOnGlobalLayout() { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public final boolean dispatchOnPreDraw() { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public final void dispatchOnDraw() { 
	        throw new RuntimeException("Stub!"); 
	    } 
	
	    public interface OnScrollChangedListener { 
	        void onScrollChanged(); 
	    } 
	
	    public interface OnTouchModeChangeListener { 
	        void onTouchModeChanged(boolean var1); 
	    } 
	
	    public interface OnDrawListener { 
	        void onDraw(); 
	    } 
	
	    public interface OnPreDrawListener { 
	        boolean onPreDraw(); 
	    } 
	
	    public interface OnGlobalLayoutListener { 
	        void onGlobalLayout(); 
	    } 
	
	    public interface OnGlobalFocusChangeListener { 
	        void onGlobalFocusChanged(View var1, View var2); 
	    } 
	
	    public interface OnWindowFocusChangeListener { 
	        void onWindowFocusChanged(boolean var1); 
	    } 
	
	    public interface OnWindowAttachListener { 
	        void onWindowAttached(); 
	
	        void onWindowDetached(); 
	    } 
	} 

```

ViewTreeObserver提供了View的多种监听，每一种监听都有一个内部类接口与之对应，内部类接口全部保存在CopyOnWriteArrayList中，通过ViewTreeObserver.addXXXListener()来添加这些监听，源码如下：

```
// Recursive listeners use CopyOnWriteArrayList
private CopyOnWriteArrayList<OnWindowFocusChangeListener> mOnWindowFocusListeners;
private CopyOnWriteArrayList<OnWindowAttachListener> mOnWindowAttachListeners;
private CopyOnWriteArrayList<OnGlobalFocusChangeListener> mOnGlobalFocusListeners;
private CopyOnWriteArrayList<OnTouchModeChangeListener> mOnTouchModeChangeListeners;
private CopyOnWriteArrayList<OnEnterAnimationCompleteListener>
        mOnEnterAnimationCompleteListeners;

// Non-recursive listeners use CopyOnWriteArray
// Any listener invoked from ViewRootImpl.performTraversals() should not be recursive
private CopyOnWriteArray<OnGlobalLayoutListener> mOnGlobalLayoutListeners;
private CopyOnWriteArray<OnComputeInternalInsetsListener> mOnComputeInternalInsetsListeners;
private CopyOnWriteArray<OnScrollChangedListener> mOnScrollChangedListeners;
private CopyOnWriteArray<OnPreDrawListener> mOnPreDrawListeners;
private CopyOnWriteArray<OnWindowShownListener> mOnWindowShownListeners;

// These listeners cannot be mutated during dispatch
private ArrayList<OnDrawListener> mOnDrawListeners;

```

以OnGlobalLayoutListener为例，首先是定义接口：

```
/**
 * Interface definition for a callback to be invoked when the global layout state
 * or the visibility of views within the view tree changes.
 */
public interface OnGlobalLayoutListener {
    /**
     * Callback method to be invoked when the global layout state or the visibility of views
     * within the view tree changes
     */
    public void onGlobalLayout();
}

```
将OnGlobalLayoutListener 添加到CopyOnWriteArray数组中：
```
/**
 * Register a callback to be invoked when the focus state within the view tree changes.
 *
 * @param listener The callback to add
 *
 * @throws IllegalStateException If {@link #isAlive()} returns false
 */
public void addOnGlobalFocusChangeListener(OnGlobalFocusChangeListener listener) {
    checkIsAlive();

    if (mOnGlobalFocusListeners == null) {
        mOnGlobalFocusListeners = new CopyOnWriteArrayList<OnGlobalFocusChangeListener>();
    }

    mOnGlobalFocusListeners.add(listener);
}

```

移除OnGlobalLayoutListener，当视图布局发生变化时不会再收到通知了：

```
/**
 * Remove a previously installed global layout callback
 *
 * @param victim The callback to remove
 *
 * @throws IllegalStateException If {@link #isAlive()} returns false
 * 
 * @see #addOnGlobalLayoutListener(OnGlobalLayoutListener)
 */
public void removeOnGlobalLayoutListener(OnGlobalLayoutListener victim) {
    checkIsAlive();
    if (mOnGlobalLayoutListeners == null) {
        return;
    }
    mOnGlobalLayoutListeners.remove(victim);
}

```

### ViewTreeObserver常用内部类：

* ViewTreeObserver.OnPreDrawListener: 当视图树将要被绘制时，会调用的接口；
* ViewTreeObserver.OnGlobalLayoutListener: 当视图树的布局发生改变或者View在视图树的可见状态发生改变时会调用的接口；
* ViewTreeObserver.OnGlobalFocusChangeListener: 当一个视图树的焦点状态改变时，会调用的接口；
* ViewTreeObserver.OnScrollChangeListener: 当视图树的一些组件发生滚动时会调用的接口；
* ViewTreeObsserver.OnTouchModeChangeListener: 当视图树的触摸模式发生改变时，会调用的接口；

### 获取View高度的几种方式:

在OnCreate()中View没有完成绘制时，获取View的宽、高都是0，怎么才能获取到宽、高呢？

######1. 通过设置View的MeasureSpec.UNSPECIFIED来测量：

	```
	int w = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED); 
	
	int h = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED); view.measure(w, h); //获得宽高 
	
	int viewWidth=view.getMeasuredWidth(); 
	int viewHeight=view.getMeasuredHeight();
	
	```
	设置我们的SpecMode为UNSPECIFIED，然后去调用onMeasure测量宽高，就可以得到宽高。
	
######2. 通过ViewTreeObserver .addOnGlobalLayoutListener来获得宽高，当获得正确的宽高后，请移除这个观察者，否则回调会多次执行：

	```
	//获得ViewTreeObserver 
	ViewTreeObserver observer=view.getViewTreeObserver();
	//注册观察者，监听变化
	observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
	     @Override
	     public void onGlobalLayout() {
	            //判断ViewTreeObserver 是否alive，如果存活的话移除这个观察者
	           if(observer.isAlive()){
	             observer.removeGlobalOnLayoutListener(this);
	             //获得宽高
	             int viewWidth=view.getMeasuredWidth();
	             int viewHeight=view.getMeasuredHeight();
	           }
	        }
	   });
	```
	
######3. 通过ViewTreeObserver .addOnPreDrawListener来获得宽高，在执行onDraw之前已经执行了onLayout()和onMeasure()，可以得到宽高了，当获得正确的宽高后，请移除这个观察者，否则回调会多次执行

```
//获得ViewTreeObserver 
ViewTreeObserver observer=view.getViewTreeObserver();
//注册观察者，监听变化
observer.addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
       @Override
       public boolean onPreDraw() {
          if(observer.isAlive()){
            observer.removeOnDrawListener(this);
             }
          //获得宽高
           int viewWidth=view.getMeasuredWidth();
           int viewHeight=view.getMeasuredHeight();
           return true;
     }
   });

```

