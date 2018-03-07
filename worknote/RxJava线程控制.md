本文主要包含如下内容：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-11/4248639.jpg)

在RxJava中，默认事件的发出和消费都是在同一个线程中的。如果要实现异步机制，就要使用到RxJava中的Scheduler。

### Scheduler涉及的API

在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：

* `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
* `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
* `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
* `Schedulers.computation()`: 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
*  Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

### 对线程进行控制的方法

`subscribeOn()`: 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。

`observeOn()`: 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

以上一篇文章中显示图片的案例为例：

```
final Drawable imageId = getResources().getDrawable(R.mipmap.gril);
Observable.create(new Observable.OnSubscribe<Drawable>() {
    @Override
    public void call(Subscriber<? super Drawable> subscriber) {
        subscriber.onNext(imageId);
        subscriber.onCompleted();
    }
})
.subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
.observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
.subscribe(new Observer<Drawable>() {
    @Override
    public void onNext(Drawable drawable) {
        imageView.setImageDrawable(drawable);
    }

    @Override
    public void onCompleted() {
    }

    @Override
    public void onError(Throwable e) {
        Toast.makeText(NormalRxActivity.this, "Error!", Toast.LENGTH_SHORT).show();
    }
});
```

这样操作之后：加载图片将会发生在 IO 线程，而设置图片则被设定在了主线程。这就意味着，即使加载图片耗费了几十甚至几百毫秒的时间，也不会造成丝毫界面的卡顿。

### Scheduler的原理

subscribeOn() 和 observeOn() 的内部实现，也是用的 lift()。

subscribeOn() 原理图：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-11/57429996.jpg)

observeOn() 原理图：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-11/45855116.jpg)

从图中可以看出，subscribeOn() 和 observeOn() 都做了线程切换的工作（图中的 "schedule..." 部位）。不同的是， subscribeOn()的线程切换发生在 OnSubscribe 中，即在它通知上一级 OnSubscribe 时，这时事件还没有开始发送，因此 subscribeOn() 的线程控制可以从事件发出的开端就造成影响；而 observeOn() 的线程切换则发生在它内建的 Subscriber 中，即发生在它即将给下一级 Subscriber 发送事件时，因此 observeOn() 控制的是它后面的线程。

最后，我用一张图来解释当多个 subscribeOn() 和 observeOn() 混合使用时，线程调度是怎么发生的（由于图中对象较多，相对于上面的图对结构做了一些简化调整）：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-11/96254421.jpg)

图中共有 5 处含有对事件的操作。由图中可以看出，①和②两处受第一个 subscribeOn() 影响，运行在红色线程；③和④处受第一个 observeOn() 的影响，运行在绿色线程；⑤处受第二个 onserveOn() 影响，运行在紫色线程；而第二个 subscribeOn() ，由于在通知过程中线程就被第一个 subscribeOn() 截断，因此对整个流程并没有任何影响。这里也就回答了前面的问题：当使用了多个 subscribeOn() 的时候，只有第一个 subscribeOn() 起作用。

以上便是RxJava线程变换部分的内容。


