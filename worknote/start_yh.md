二话不说，先上图：

图一：未优化前APP启动的样子：

<img src="http://7xkl0t.com1.z0.glb.clouddn.com/17-11-6/46997386.jpg" width = "30%" />

图二：优化后APP启动的样子：

<img src="http://7xkl0t.com1.z0.glb.clouddn.com/17-11-6/27100468.jpg" width = "30%" />

对比这两幅图可以看到什么：

1. 在图一中，APP的启动经历如下过程：点击桌面APP icon——>经历黑屏页面——>跳转到Splash页面；
2. 在图二中，APP的启动经历如下过程：点击桌面APP icon——>跳转到Splash页面。

因此，这次的优化主要目标就是：
> **秒开，一触即发，不给用户任何等待的时间。**

本文主要记录这次优化过程，其中包含如下内容：

0. APP启动秒开的意义所在
1. Android APP启动的时候都做了哪些事儿？
2. 为什么有的APP启动的时候会看到一个黑屏（易项优选）or 白屏界面？
3. 如何优化，让易项优选的Android APP做到秒开？

下面开始正文：

# 一、APP启动秒开的意义所在？

这个问题的答案我只想说一个字儿——**不能忍**。

仔细点来说，是这样的：
> APP启动时间越短，用户才有耐心等待这个APP打开，然后使用。如果每次打开都很慢，用户可能都来不及等待直接切换到其他的APP了，最严重的后果，可能被用户抛弃掉，我们的APP被卸载掉。

# 二、Android APP启动的时候都做了哪些事儿？
 
APP启动的三种方式：冷启动、热启动和温启动。
 
**冷启动：**

APP在冷启动的时候，做的事儿比较多。其它两种情况，系统只需要将APP从后台切到前台即可。因此，这里着重分析APP在冷启动的时候经历的过程。
 
在冷启动时，系统会执行三个任务：
> 1. 加载并启动APP
> 2. 在APP启动后，立即展示空白的window
> 3. 创建APP进程

当系统创建了APP进程之后，那么APP进程就会执行如下步骤：

>1. 创建APP对象
2. 启动main thread
3. 创建MainActivity(APP要启动的第一个页面，也不一定是MainActivity)
4. inflate views
5. 布置屏幕
6. 进行首次绘制

一旦app进程完成了第一次绘制，系统进程就会用main activity替换已经展示的background window之后用户才可以使用app。

下图，展示了系统和APP进程的交互工作过程，展示了APP启动创建时期的几个重要部分，在创建APP和Main Activity之间，我们可以提升性能问题的地方：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-11-6/16249652.jpg)

**Application的创建：**

当应用启动的时候，空白的window在app第一次完成绘制之前都会存在。在那之后，系统进程才会替换启动窗口，允许用户开始和app交互。

如果你覆写了 Application.oncreate() 方法，app启动的时候，会调用该方法。之后，app会孵化主线程（UI线程），并通过它来创建main activity。

**Activity的创建：**

在app进程创建了Activity之后，Activity将会执行以下操作

> 1. 初始化值
2. 调用构造函数
3. 调用回调方法，比如Activity.onCreate()。

通常，onCreate方法会对加载时间有比较大的影响。因为它将执行繁重的工作：加载和填充view，并初始化Activity运行期间需要用的对象。

**热启动：**

相对于冷启动，热启动会简单的多。如果app的所有Activities还存在内存中，那么系统需要做的就是将activity切换到前台。这样app会避免进行对象初始化，布局填充和渲染。
但是，如果一些内存在触发内存回调方法的时候被回收了，比如onTrimMemory()，那么这些对象就需要重新创建。

热启动会和冷启动有相同的行为。系统也会展示一个空白的window，知道app完成Activity的渲染。

**温启动：**

温启动做的工作介于冷热启动之间。这里列举几种可能被认为是温启动的状态：

1. 用户离开了app，然后重新启动它。这时进程还在继续运行，但是Activity被回收了，app需要重新创建activity。

2. 系统将你的app回收了，然后用户重新启动app。进程和Activity都需要重新启动，但它们可以从onCreate方法保存的bundle中恢复。

# 三、为什么有的APP启动的时候会看到一个黑屏（易项优选）or 白屏界面？

有了前置知识之后，下面看我们APP中的启动黑屏问题：

为什么在Activity的onCreate()方法中已经调用setContentView(View)设置该Activity的显示布局，而刚启动的时候会显示一个黑屏而不是我设置的布局呢？ 下面仔细说道说道：

> 当打开一个Activity时，如果这个Activity所属Application还没有在运行，系统会为这个Activity的创建一个进程，但进程的创建与初始化是需要时间的，在这个动作完成之前，如果初始化的时间过长，屏幕上可能没有任何动静，用户会以为没有点到屏幕上的icon。那该怎么办呢，这就有了StartingWindow（也称之为PreviewWindow，也就是上文中提到的空白window）的出现，这样看起来就像Activity已经启动起来了，只是数据内容还没有初始化好。

> StartingWindow 一般出现在应用程序进程创建并初始化成功前，所以它是个临时窗口，对应的WindowType是TYPE_APPLICATION_STARTING。目的是告诉用户，系统已经接受到操作，正在响应，在程序初始化完成后实现目的UI，同时移除这个窗口。

> 而这个StartingWindow就是造成我们APP启动时黑屏的“元凶”，Google给出的解决方案是：给Application和Activity设置Theme，系统会根据设置的Theme初始化StartingWindow。Window布局的顶层是DecorView，StartingWindow显示一个空DecorView，但是会给这个DecorView应用这个Activity指定的Theme，如果这个Activity没有指定Theme就用Application的（Application系统要求必须设置Theme）。

在Theme中可以指定窗口的背景，Activity的ICON，APP整体文字颜色等，如果说没有指定任何属性，就会用默认的属性，也就是上文中提到的空DecorView，所以我们的黑屏和空DecorView息息相关，我们给APP设置的Style就决定了是白屏还是黑屏。

1. 如果选择了Black的系列的主题那么Activity跳转的时候就是黑屏
2. 如果选择了Light的系列的主题那么Activity跳转的时候就是白屏

# 四. 如何优化让易项优选的Android APP做到秒开？

这个问题，Google已经给出了我们具体的解决方案，其原理就是将预览的黑屏页面替换成我们启动页面的图片。

第一步，在res/drawable下新建一个layer-list，命名为：splash.xml

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 背景颜色 -->
    <item android:drawable="@color/white" />

    <item>
        <!-- 图片 -->
        <bitmap
            android:gravity="center_horizontal"
            android:src="@mipmap/splash_img" />
    </item>

</layer-list>
```

第二步：使用第一步设置的layer-list作为背景

```
<!-- Base application theme. -->
<style name="AppTheme" parent="@style/Theme.AppCompat"></style>

<style name="SplashTheme" parent="AppTheme">
    <!-- 欢迎页背景引用刚才写好的 -->
    <item name="android:windowBackground">@drawable/splash</item>
    <item name="android:windowFullscreen">true</item>
</style>
```

第三步：在AndroidManifest.xml中定义BootActivity（第一个启动的页面）的theme为SplashTheme：

```
<activity
    android:name=".activity.SplashActivity"
    android:theme="@style/SplashTheme">
</activity>
```

这样就可以做到在手机屏幕上点击APP icon之后做到秒开效果了。

### 参考文档：

1. [Launch-Time Performance](https://developer.android.com/topic/performance/launch-time.html)
2. [Splash Screens the Right Way](https://www.bignerdranch.com/blog/splash-screens-the-right-way/)


