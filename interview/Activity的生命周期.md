本文主要复习Activity的生命周期，这东西经常在用，但从未进行过一个系统的整理，借这个机会复习并总结一下！

Activity的生命周期大致可以分为两种情况：

* 第一种：典型情况下的生命周期，是指在用户参与的情况下，Activity所经过的生命周期的改变；
* 第二种，异常情况下的生命周期，是指Activity被系统回收或者由于当前设备的Configuration发生改变从而导致Activity被销毁重建。

# 正常情况下的生命周期
如下图所示：

![Activity生命周期图](https://developer.android.com/images/activity_lifecycle.png?hl=zh-cn)

正常情况下地址经历如下几个生命周期方法：

1. onCreate: 表示Activity正在被创建，可以在这个方法中做一些初始化工作，比如调用setContentView去加载界面布局资源、初始化Activity所需数据等。
2. onStart: 表示Activity正在被启动，即将开始，这时Activity已经可见了，但还没有出现在前台，还无法和用户进行交互。可以理解为Activity已经显示出来了，但是我们还看不到。
3. onResume: 表示Activity可见了，并且出现在前台并开始活动。**onStart和onResume都表示Activity已经可见，但是onStart的时候Activity还在后台，onResume的时候Activity才显示到前台。**
4. onPause: 表示Activity正在停止，正常情况下，onStop紧接着会被调用。**在onPause中可以做一些存储数据、停止动画等工作，但是不能太耗时，只有onPause方法执行完毕后，新Activity才能显示出来。**
5. onRestart:表示Activity正在重新启动。
6. onStop: 表示Activity即将停止，可以做一些轻量级的回收工作，不能太耗时
7. onDestory：表示Activity即将被销毁，这是Activity生命周期中的最后一个回调，在这里可以做一些回收工作和最终的资源释放。

# 异常情况下的生命周期

## 情况1：资源相关的系统配置发生改变导致Activity被杀死并重新创建

系统只在Activity异常终止的时候才会调用onSaveInstanceState和onRestoreInstance来存储和恢复数据，其它过程不会触发这个过程。

## 情况2：资源内存不足导致低优先级的Activity被杀死

Activity按照优先级可以从高到低，分为三类：

1. 前台Activity——正在和用户交互的Activity，优先级最高；
2. 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户直接交互。
3. 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。


