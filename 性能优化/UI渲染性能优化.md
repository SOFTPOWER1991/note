解决项目中的性能问题，有相应的套路，而这些套路也是解决所有问题的必要流程：

> 发现问题 ——> 定位问题——>改善问题——>解决问题

本文主要讲解：**Android开发中UI渲染方面的性能优化**

其中包括如下内容：

1. UI渲染机制
2. UI渲染所面临的问题
3. CPU、GPU各自所面临的问题
4. 借助相应的工具（GPU，Debug GPU OverDraw；CPU，CPU优化工具：Hierarchy Viewer）
5. 所出现的问题的相应解决手段。

# UI渲染机制

Android的显示过程可以简单概括为：Android应用程序把经过测量、布局、绘制后的Surface缓存数据，通过SurfaceFlinger把数据渲染到显示屏幕上，通过Android的刷新机制来刷新数据。也就是说：应用层负责绘制，系统层负责渲染，通过进程间通信把应用层需要绘制的数据传递到系统层服务，系统层通过刷新机制把数据更新到屏幕上。

UI渲染所涉及到的硬件：

**CPU:** 中央处理器,它集成了运算,缓冲,控制等单元,包括绘图功能.CPU将对象处理为多维图形polygons,纹理textures(Bitmaps、Drawables等都是一起打包到统一的纹理).
**GPU：** 类似于CPU的专门用来处理Graphics的处理器, 作用用来帮助加快栅格化操作,当然,也有相应的缓存数据(例如缓存已经光栅化过的bitmap等)机制。
**OpenGL ES：** 手持嵌入式设备的3DAPI,跨平台的、功能完善的2D和3D图形应用程序接口API,有一套固定渲染管线流程.
**DisplayList：** 在Android把XML布局文件转换成GPU能够识别并绘制的对象。这个操作是在DisplayList的帮助下完成的。DisplayList持有所有将要交给GPU绘制到屏幕上的数据信息。
**栅格化：** 将polygens，textures等资源,转化为一格格像素点的像素图,显示到屏幕上，由GPU来完成.
**垂直同步VSYNC：** 让显卡的运算和显示器刷新率一致以稳定输出的画面质量。它告知GPU在载入新帧之前，要等待屏幕绘制完成前一帧。

渲染机制流程：

UI对象---->CPU处理为多维图形,纹理 -----通过OpeGL ES接口调用GPU----> GPU对图进行光栅化(Frame Rate ) ---->硬件时钟(Refresh Rate)----垂直同步---->投射到屏幕。

如下图所示：

![UI渲染](http://upload-images.jianshu.io/upload_images/1848340-0224bc9dde7b163e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**渲染时间线：16ms**

绘制一个单元到底要多长时间合适？这里涉及到一个概念：FPS，每秒传递的帧数。在理想情况下，60FPS就感觉不到卡顿，因此必须在16ms内完成渲染，这就是所谓的渲染时间线。

Android系统每隔16ms发出VSYNC信号(1000ms/60=16.66ms)，触发对UI进行渲染， 如果每次渲染都成功，这样就能够达到流畅的画面所需要的60fps，为了能够实现60fps，这意味着计算渲染的大多数操作都必须在16ms内完成。

**正常情况：**

![正常情况渲染](http://upload-images.jianshu.io/upload_images/1848340-d5f27647c1cd1f99?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**渲染超时,计算渲染时间超过16ms：**

当这一帧画面渲染时间超过16ms的时候,垂直同步机制会让显示器硬件 等待GPU完成栅格化渲染操作,
这样会让这上一帧画面,多停留了16ms,甚至更多.这样就这造成了 用户看起来 画面停顿.

![渲染超时](http://upload-images.jianshu.io/upload_images/1848340-ef60409448b8395e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当GPU渲染速度过慢,就会导致如下情况,某些帧显示的画面内容就会与上一帧的画面相同：

![](http://upload-images.jianshu.io/upload_images/1848340-65876173c9401296.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# UI渲染所面临的问题

在UI渲染机制中，有三大过程：
1. CPU负责计算的过程；
2. OpenGL ES负责将CPU计算好的数据传给GPU
3. GPU负责栅格化操作的过程；

而UI渲染时所面临的问题，也主要在这三个过程中。

**CPU所面临的问题：**

CPU将View对象处理为多维图形polygons,纹理textures(Bitmaps、Drawables等都是一起打包到统一的纹理).

我们知道在Android中每个View绘制都有三个核心步骤，通过Measure、Layout来确定当前需要绘制的View所在的大小和位置，通过绘制（Draw）到Surface，在Android系统中整体绘制流程是从ViewRootImpl的performTraversals（）方法开始，通过这个方法可以看出**Measure和Layout都是递归来获取View的大小和位置，并且以深度作为优先级。层级越深，元素越多，耗时越多。**

因此，CPU所面临的问题是：

* Layout太复杂，无法在16ms内完成渲染；
* UI有层叠太多的绘制单元；
* 动画执行的次数过多；

**GPU所面临的问题**

GPU的绘制过程,就跟刷墙一样,一层层的进行,16ms刷一次.这样就会造成,图层覆盖的现象,即无用的图层还被绘制在底层,造成不必要的浪费.

因此，GPU所面临的问题是：

* 过度绘制

# 性能问题检测工具

**Profile GPU Rendering**
> 功能特点如下：
> 
> * 是一个图形化监测工具，能实时反映当前绘制时间；
> * 横轴表示时间，纵轴表示每一帧的耗时（单位为ms）;
> * 随着时间推移，从左到右的舒心呈现；
> * 提供了一个标准的耗时，如果高于标准耗时，表示当前一帧丢失。

使用要点：

每一条柱状图都有4中颜色，红、橙、蓝、紫，对应每一帧不同阶段的实际耗时情况。

* 红色，表示执行的时间，Android进行2D渲染Display List的时间，为了绘制到屏幕上，Android需要使用OpenGL ES 的API接口来绘制Display List,这些API有效的将数据发送到GPU，最终显示在屏幕上。当红色线非常高时可能是由于重新提交了视图导致的。
* 橙色，表示处理的时间，或者是CPU告诉GPU渲染一帧的地方，这是一个阻塞调用，因为CPU会一直等待GPU发出接到命令的回复，如果柱状图很高，说明GPU繁忙。
* 蓝色，表示测量绘制的时间，代表需要多长时间去创建和更新DisplayList。蓝色越高，可能是因为需要重新绘制，或者自定义视图的onDraw函数处理事情太多了。
* 紫色，表示资源转移到渲染线程的时间。

在实际开发中，可以通过命令：**adb shell dumpsys gfxinfo XXXXX(包名)** 把具体的耗时输出到日志中来分析。

任何时候超过绿线，就可能丢失一帧的内容。GPU Profile工具可以帮助发现渲染相关的问题，需要借助另外的工具来定位问题所在，比如：Hierarchy Viewer、TraceView

文档：
[Profile GPU Rendering Walkthrough
](https://developer.android.com/studio/profile/dev-options-rendering.html)


**Hierarchy Viewer**
> Android SDK自带的一款可视化调试工具。用来检查Layout嵌套及绘制时间。

使用要点：

在检测的过程中会出现三个指示灯，分别代表：Measure、Layout，draw这三个过程耗时情况。

绿色，代表良好；黄色，有问题；红色，有严重问题；

文档：
[Profile Your Layout with Hierarchy Viewer
](https://developer.android.com/studio/profile/hierarchy-viewer.html)

**Debug GPU Overdraw**
> 通过Android设备打开Debug GPU Overdraw可以方便的看到界面渲染情况

使用要点：

* 无色，没有过度绘制，每个像素绘制了一次。
* 蓝色，每个像素多绘制了一次。大片的蓝色还是可以接受的。如果整个窗口是蓝色的，可以尝试优化减少一次绘制。
* 绿色，每个像素多绘制了2次。
* 淡红，每个像素多绘制了3次。一般来说，这个区域不超过屏幕的四分之一是可以接受的。
* 深红，每个像素多绘制了4次或者更多，严重影响性能，需要优化，避免出现深红色区域。

我们优化的目标就是尽量减少红色OverDraw，看到更多的蓝色区域。

文档：
[Debug GPU Overdraw Walkthrough](https://developer.android.com/studio/profile/dev-options-overdraw.html)


# 常见性能问题解决方案

### 过度绘制解决方案

一般在XML布局和自定义控件中绘制，导致过度绘制的主要原因是：

* XML布局——>控件有重叠切都有设置背景
* View自绘——>View.onDraw里面同一个区域被绘制多次

### 布局优化

* **减少Layout层级，合理使用布局**

RelativeLayout存在性能低的问题，因为RelativeLayout会对子View做两次测量，在RelativeLayout中子View的排列方式是彼此依赖的，，在确定每个自View的时候需要，在横向、纵向分别进行一次测量。

LinearLayout只需要对一个方向上进行测量。但是，如果使用了Weight属性，也需要进行两次测量，因为没有过多的依赖关系，所以效率比RelativeLayout要高处很多。

如果布局嵌套层级太深，推荐使用RelativeLayout减少布局本身层次。


*  **合理使用Merge**

Merge的使用场合有两处：
1. 在自定义View中使用，父元素尽量是FrameLayout和LinearLayout
2. 在Activity中整体布局，根元素需要是FrameLayout

Merge标签的原理
> 在Android布局的源码中，如果是Merge标签，那么直接将其中的子元素添加到Merge标签Parent中，这样就保证了不会引入额外的层级。

Merge不是所有地方都可以使用的，使用要求：

1. Merge只能用在布局XML文件的根元素上。
2. 使用Merge加载一个布局时，必须制定一个ViewGroup作为其父元素，并且要设置加载的attachToRoot参数为true（inflate（int，ViewGroup，boolean））
3. 不能在ViewStub中使用Merge标签，原因是：ViewStub的infalte方法根本没有attachToRoot的设置


* **提高显示速度，合理使用ViewStub**

	ViewStub是一个轻量级的View，它是一个看不见的，并且不占布局位置，占用资源非常小的视图对象。可以为ViewStub指定一个布局，加载布局时，只有ViewStub会被初始化，然后当ViewStub被设置为可见时，或是调用了ViewStub.inflate()时，ViewStub所指向的布局会被加载和实例化，然后ViewStub的布局属性都会传给它指向的布局。这样，就可以使用ViewStub来设置是否显示某个布局。

	ViewStub使用注意事项：
	
	1. ViewStub只能加载一次，之后ViewStub对象会被置为空。也就是说，某个被ViewStub指定的布局被加载后，就不能再通过ViewStub来控制它了。
	2. ViewStub只能用来加载一个布局文件，而不是某个具体的View，当然也可以把View写在某个布局文件中。如果想操作单独的View，还是使用visibility属性。
	3. ViewStub中不能嵌套Merge标签。

	ViewStub使用场景：
	
	1. 在程序运行期间，某个布局在加载后，就不会有变化，除非销毁页面再重新加载。
	2. 想要控制显示与隐藏的是一个布局文件，而非某个View
	
* **布局复用，合理使用include**
	
	使用include标签，提取代码公用部分，在编写Android布局文件时，可以将相同的部分提取出来。






