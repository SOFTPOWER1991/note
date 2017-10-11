本文主要涉及如下内容：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-11/91724581.jpg)

RxJava中涉及到的4个概念：

* Observable (可观察者，即被观察者)
* Observer (观察者)
* subscribe (订阅)
* 事件

其中：Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

# **1. 创建被观察者 Observable**

创建观察者的一共有三种方式：

第一种方式：
```
Observable observable= Observable.create(new Observable.OnSubscribe<String>(){

    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("一二三四五");
        subscriber.onNext("上山打老虎");
        subscriber.onNext("老虎一发威");
        subscriber.onNext("武松就发怵");
        subscriber.onCompleted();

    }
});

```

第二种方式：
```
String [] kk={"一二三四五","上山打老虎","老虎一发威","武松就发怵"};
Observable observable=Observable.from(kk);
```

第三种方式：
```
Observable observable = Observable.just(
	"一二三四五====just",
	"上山打老虎=======just",
	"老虎一发威=======just",
	"武松就发怵=======just");
```

# **2. 创建观察者 Subscriber**

```
Subscriber subscriber=new Subscriber<String>() {
    @Override
    public void onCompleted() {
        mText.append("执行观察者中的onCompleted()...\n");
        mText.append("订阅完毕，结束观察...\n");
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onNext(String s) {
        mText.append("执行观察者中的onNext()...\n");
        mText.append(s+"...\n");
    }

    @Override
    public void onStart() {
        super.onStart();
        mText.append("on subscriber start.....\n");
    }
};
```

# **3. 将Observable 和 Subscriber 建立订阅关系**

第一种订阅方式：

```
observable.subscribe(subscriber);
```

第二种订阅方式：使用不完整定义的回调方式。

```
Action1<String> onNextAction = new Action1<String>() {
    // onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    // onError()
    @Override
    public void call(Throwable throwable) {
        // Error handling
    }
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {
        Log.d(tag, "completed");
    }
};

// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);

// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()

observable.subscribe(onNextAction, onErrorAction);

// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()

observable.subscribe(onNextAction, onErrorAction, onCompletedAction);

```

以上便完成了被观察者和观察者的创建、以及他们建立订阅的过程。

**Notice:注意事项**

1. 创建观察者其实有两种方式：Observer 和 Subscriber。这里只看了Subscriber创建观察者的方式。其中有两个注意事项：

	> * 	**onStart()**: 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法，具体可以在后面的文中看到。
	> * **unsubscribe()**: 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 onPause() onStop() 等方法中）调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。

2. 在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。
3. 将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe().


## 使用案例：


**案例1：将字符串数组 names 中的所有字符串依次打印出来**

```
String[] names = {"我们", "你好", "哈哈", "天哪"};
Observable.from(names)
        .subscribe(new Action1<String>() {
            @Override
            public void call(String name) {
                Log.d("subscribe", name);
            }
        });
```



**案例2：由指定的一个 drawable 文件 id drawableRes 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错**

```
final Drawable imageId = getResources().getDrawable(R.mipmap.ic_launcher);
Observable.create(new Observable.OnSubscribe<Drawable>() {
    @Override
    public void call(Subscriber<? super Drawable> subscriber) {
        subscriber.onNext(imageId);
        subscriber.onCompleted();
    }
}).subscribe(new Observer<Drawable>() {
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


以上便是RxJava的基本使用了。

