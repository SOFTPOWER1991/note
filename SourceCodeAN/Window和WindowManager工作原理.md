本文主要讲解Window和WindowManger的工作原理，其中包含如下内容：

1. Window和WindowManger简介
2. 通过WindowManager添加一个简单的Window以及Window的Flags属性
3. Window的内部机制
	* 3.1 Window的添加过程
	* 3.2 Window的删除过程
	* 3.3 Window的更新过程
4. Window的创建过程
	* 4.1 Activity的Window创建过程
	* 4.2 Dialog的Window创建过程
	* 4.3 Toast的Window创建过程

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





