最早接触RxJava是在2015年的时候，那时候在项目中一个分支上进行了测试单并没有进行实际的应用。来到以太后，发现项目中已经实际运用了RxJava,RxAndroid, RxBinding这一系列的Rx全家桶。因此决定花一些时间来学习Rx全家桶，跟项目中使用的技术保持同步。

以下是主要参考文档：

* ReactiveX/RxJava wiki  https://github.com/ReactiveX/RxJava/wiki
* ReactiveX/RxJava Github https://github.com/ReactiveX/RxJava
* 给Android开发者的RxJava详解 https://gank.io/post/560e15be2dca930e00da1083#toc_25
* 关于RxJava最友好的文章 : http://www.jianshu.com/p/6fd8640046f1

下面是我对RxJava掌握前进地图：

![rxjava](http://7xkl0t.com1.z0.glb.clouddn.com/17-9-30/25953479.jpg)

### RxJava是什么

官方文档如是说：
> RxJava is a Java VM implementation of ReactiveX (Reactive Extensions): a library for composing asynchronous and event-based programs by using observable sequences.

翻译成中文如是：
> RxJava 是一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库。

对于我真正理解RxJava然并卵。

下面以一个实际案例：**按下开关，台灯灯亮** 来讲解RxJava是什么，话不多说，反手就是一个图：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-9-30/19143076.jpg)

这个图说了什么？

* 开关（被观察者）作为事件的产生方（生产“开”和“关”这两个事件），是主动的，是整个开灯事理流程的起点。
* 台灯（观察者）作为事件的处理方（处理“灯亮”和“灯灭”这两个事件），是被动的，是整个开灯事件流程的终点。
* 在起点和终点之间，即事件传递的过程中是可以被加工，过滤，转换，合并等等方式处理的（上图没有体现，后面对会讲到）

这就引出了RxJava中非常重要的几个概念：被观察者Observable、观察者Observer、事件、subscribe (订阅)。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。


### RxJava的原理和API

### RxJava有什么特点

### RxJava用来解决什么问题

### RxJava的使用场景


