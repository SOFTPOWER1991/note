ConstraintLayout的性能优势

#### ConstraintLayout是什么？
> ConstraintLayotu是在Google I/O 大会上提出的一个可以灵活控制子控件的位置和大小的布局。可以把扁平化布局做到极致。用来解决之前==复杂布局==时RelativeLayout 和 LinearLayout带来的渲染性能问题。

当用户将某个 Android 视图作为焦点时，Android FrameWork会指示该视图进行自我绘制。

#### Android绘制过程包括 3 个阶段：

**1. 测量**

系统自顶向下遍历视图树，以确定每个 ViewGroup和View元素应当有多大。在测量ViewGroup的同时也会测量其子对象。

**2. 布局**

系统执行另一个自顶向下的遍历操作，每个ViewGroup都根据测量阶段中所确定的大小来确定其子对象的位置。

**3. 绘制**

系统再次执行一个自顶向下的遍历操作。对于视图树中的每个对象，系统都会为其创建一个Canvas对象，以便向GPU发送一个绘制命令列表。这些命令包含系统在前面2个阶段中确定的ViewGroup和View对象的大小和位置。

因为在绘制过程中的每个阶段都需要对视图树执行一次自顶向下的遍历操作。因此，视图层次结构中嵌入（或嵌套）的视图越多，设备绘制视图所需的时间和计算功耗也就越多。在Android应用布局中保持扁平的层次结构，可以为应用创建相应快速的界面。

#### 嵌套布局层次结构的开销

**Notice：布局的开销过大，仅仅会发生在布局过于复杂，需要使用各种布局嵌套时**

下面是Google Sample提供的是一个示例：
![屏幕快照 2018-09-02 17.09.11](http://7xkl0t.com1.z0.glb.clouddn.com/2018-09-02-屏幕快照 2018-09-02 17.09.11.png)

如果构想一个像上面那样的布局，如果使用传统布局来构建，布局层次是这样的：

```
<RelativeLayout>
  <ImageView />
  <ImageView />
  <RelativeLayout>
    <TextView />
    <LinearLayout>
      <TextView />
      <RelativeLayout>
        <EditText />
      </RelativeLayout>
    </LinearLayout>
    <LinearLayout>
      <TextView />
      <RelativeLayout>
        <EditText />
      </RelativeLayout>
    </LinearLayout>
    <TextView />
  </RelativeLayout>
  <LinearLayout >
    <Button />
    <Button />
  </LinearLayout>
</RelativeLayout>
```

而使用ConstraintLayout布局后的代码：

```
<android.support.constraint.ConstraintLayout>
  <ImageView />
  <ImageView />
  <TextView />
  <EditText />
  <TextView />
  <TextView />
  <EditText />
  <Button />
  <Button />
  <TextView />
</android.support.constraint.ConstraintLayout>
```

如何来测量两种布局方式的性能差异呢？借助HierarchyViewer来查看

先看下RealtiveLayout的布局层级：


![屏幕快照_2018-08-29_19_59_46](http://7xkl0t.com1.z0.glb.clouddn.com/2018-09-02-屏幕快照_2018-08-29_19_59_46.png)

看看ConstraintLayout的布局层级：
![屏幕快照_2018-08-29_19_56_42](http://7xkl0t.com1.z0.glb.clouddn.com/2018-09-02-屏幕快照_2018-08-29_19_56_42.png)

在HierarchyView中查看RealtiveLayout的测量、布局、绘制 所需要的时间：

![屏幕快照 2018-08-29 19.50.32](http://7xkl0t.com1.z0.glb.clouddn.com/2018-09-02-屏幕快照 2018-08-29 19.50.32.png)

其中一共使用了14个View，总共用时：46.431ms

看看ConstraintLayout的测量、布局、绘制所需要的时间：
![屏幕快照 2018-08-29 19.52.01](http://7xkl0t.com1.z0.glb.clouddn.com/2018-09-02-屏幕快照 2018-08-29 19.52.01.png)

其中一共使用了12个View，共用时：41.853ms

这里借助了HierarchyViewer来简单的分析了ConstraintLayout能够带来的改变，决定之后在项目中开始使用这种布局方式。

在下篇文章中将借助Android SDK提供的API来进行ConstraintLayout的性能分析：

1. [Systrace](https://developer.android.google.cn/studio/profile/systrace.html)
2. [FrameMetrics](https://developer.android.google.cn/reference/android/view/FrameMetrics.html)

附录链接：

1. 开发指南： [Build a Responsive UI with ConstraintLayout](https://developer.android.google.cn/training/constraint-layout/index.html)
2. API参考文档：[ConstraintLayout](https://developer.android.google.cn/reference/android/support/constraint/ConstraintLayout.html)