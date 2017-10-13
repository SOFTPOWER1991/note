RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一。所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。

常见操作符如下：

![](http://7xkl0t.com1.z0.glb.clouddn.com/17-10-12/53828412.jpg)

下文就这些常见操作符挨个解释，并给出具体使用案例：

### map 操作符

map(): 对事件对象的直接变换，比如输入的事件是Integer类型的，变换后为Boolean类型的。

示例如下：

```
private Integer[] number = {1, 2, 3, 4, 5, 6};

Observable
    .from(number)  //之前提到的创建Observable方法
    .map(new Func1<Integer, Boolean>() {

        @Override
        public Boolean call(Integer integer) {
            return (integer < 3);
        }
    })
    .subscribe(new Action1<Boolean>() {
        @Override
        public void call(Boolean aBoolean) {
            mText.append("\n观察到输出结果：\n");
            mText.append(aBoolean.toString());
        }
    });
```

可以看到，map()方法将参数中的String对象转换成一个Bitmap对象后返回，而在经过map()方法后，事件的参数类型也由Integer转为了Boolean类型。这种直接变换对象并返回的，是最常见的变换。

在map中出现了一个类Func1。它和Action1很相似，都是RxJava的接口，用于包装含有一个参数的方法。他们的区别在于，Func1包装的是有返回值的方法，而Action1则没有返回值。

而且可以看到：Func1中的泛型参数，第一个是输入的事件类型，第二个是变换后的事件类型。最终这个变换之后的事件类型交给了Action1中的泛型参数来进行处理。

### flatmap 操作符

flatmap操作符完成一对多的转化。传入一个参数，可以转换出很多参数。

下面看具体实例：

```
SchoolClass[] mSchoolClasses=new SchoolClass[2];

Student[] student=new Student[5];
for(int i=0;i<5;i++){
    Student s=new Student("二狗"+i,"17");
    student[i]=s;
}
mSchoolClasses[0]=new SchoolClass(student);

Student[] student2=new Student[5];
for(int i=0;i<5;i++){
    Student s=new Student("小明"+i,"27");
    student2[i]=s;
}
mSchoolClasses[1]=new SchoolClass(student2);

public SchoolClass[] getSchoolClass(){
        return  mSchoolClasses;
}

class SchoolClass{
    Student[] stud;
    public SchoolClass(Student[] s){
        this.stud=s;
    }
    public Student[] getStudents(){
        return  stud;
    }
}

class Student{
    String name;
    String age;
    public Student(String name,String age){
        this.name=name;
        this.age=age;
    }
}

Observable.from(getSchoolClass())
        .flatMap(new Func1<SchoolClass, Observable<Student>>() {
            @Override
            public Observable<Student> call(SchoolClass schoolClass) {
                //将Student列表使用from方法一个一个发出去
                return Observable.from(schoolClass.getStudents());
            }
        })
        .subscribe(new Action1<Student>() {
            @Override
            public void call(Student student) {
                mText.append("打印单个学生信息：\n");
                mText.append("name:"+student.name+"    age: "+student.age+"\n");
            }
        });
```

从上面的例子中可以看出，flatmap把传入的参数转化为另一个对象。但和map不同的是，flatmap返回的是Observable对象，并且这个Observable对象并不是被直接发送到了Subscriber的回调中。

flatMap()的原理：
> 1. 使用传入的事件对象，创建一个Observable对象；
> 2. 并不发送这个Observable，而是将它激活，于是它开始发送事件；
> 3. 每一个创建出来的Observable发送的事件，都被汇入同一个Observale，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。

这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。

至此，我有个问题：
> 我如何才能识别出在合适的场景使用合适的操作符？

### filter操作符

filter操作符：输出过滤条件后的结果项。

具体案例：

```
Integer[] integers={1,2,3,4,5,6,7,8,9,10};
Observable.from(integers)
        .filter(new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                Log.e("obs", (integer % 2 != 0) + "");
                return integer%2!=0;
            }
        })
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                mText.append(integer.toString()+",");
            }
        });
```

上面的例子，借助filter操作符，过滤出不能被2整除的数字。


### merge操作符

merge操作符用来合并多个Observable。Merge可能会让合并的Observables发射的数据交错（有一个类似的操作符Concat不会让数据交错，它会按顺序一个接着一个发射多个Observables的发射物）。任何一个原始Observable的onError通知会被立即传递给观察者，而且会终止合并后的Observable。

还有一个叫MergeDelayError的操作符，它的行为有一点不同，它会保留onError通知直到合并后的Observable所有的数据发射完成，在那时它才会把onError传递给观察者。
RxJava将它实现为merge, mergeWith和mergeDelayError。


```
Observable obs1=Observable.create(new Observable.OnSubscribe<String>(){

    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            Thread.sleep(500);
            subscriber.onNext(" aaa");
            subscriber.onCompleted();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).subscribeOn(Schedulers.newThread());

Observable obs2=Observable.create(new Observable.OnSubscribe<String>(){

    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            Thread.sleep(1500);
            subscriber.onNext("bbb");
            subscriber.onCompleted();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).subscribeOn(Schedulers.newThread());

Observable.merge(obs1,obs2)
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<String>() {
            StringBuffer sb=new StringBuffer();
            @Override
            public void onCompleted() {
                mText.append("两个任务都处理完毕！！\n");
                mText.append("更新数据："+sb+"\n");
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                sb.append( s+",");
                mText.append("得到一个数据："+s+"\n");
            }
        });
```

这个操作符适合在进入一个页面有多个请求同时发出的情况下使用。

### toSortedList

对给定的List进行排序。

```
private Integer [] words={1,3,5,2,34,7,5,86,23,43};

Observable.from(words)
      .toSortedList()
       .flatMap(new Func1<List<Integer>, Observable<Integer>>() {
           @Override
           public Observable<Integer> call(List<Integer> strings) {
               return Observable.from(strings);
           }
       })
      .subscribe(new Action1<Integer>() {
          @Override
          public void call(Integer strings) {
              mText.append(strings+"\n");
          }
      });
```

### take 和 takeLast 操作符

take操作符，让你可以修改Observable的行为，只返回前面的N项数据，然后发射完成通知，忽略剩余的数据。

takeLast操作符，发射Observable发射的最后N项数据。使用TakeLast操作符修改原始Observable，你可以只发射Observable'发射的后N项数据，忽略前面的数据。


```
Observable.from(number)
  .filter(new Func1<Integer, Boolean>() {
      @Override
      public Boolean call(Integer integer) {
          return integer%2!=0;
      }
  })
    //取前四个
    .take(4)
    //取前四个中的后两个
    .takeLast(2)
    .doOnNext(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            mText.append("before onNext（）\n");
        }
    })
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            mText.append("onNext()--->"+integer+"\n");
        }
    });
```
                                                                                                                                                                                                                                          
### interval 操作符

interval操作符，返回一个Observable，它按固定的时间间隔发射一个无限递增的整数序列。它接受一个表示时间间隔的参数和一个表示时间单位的参数。

```
//interval（）是运行在computation Scheduler线程中的，因此需要转到主线程
mSubscription = Observable.interval(1, TimeUnit.SECONDS)
        .takeUntil(new Func1<Long, Boolean>() {
            @Override
            public Boolean call(Long aLong) {
                return aLong == 10;
            }
        })
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<Long>() {
            @Override
            public void call(Long aLong) {
                mText.setText(aLong + "");

                Toast.makeText(RxTimerActivity.this, "====" + aLong, Toast.LENGTH_SHORT).show();
            }
        });
```

上面的例子每个一秒钟发送一个递增的数字，takeUtil直到10的时候，停止。

### connect 操作符

让一个可连接的Observable开始发射数据给订阅者。

可连接的Observable (connectable Observable)与普通的Observable差不多，不过它并不会在被订阅时开始发射数据，而是直到使用了Connect操作符时才会开始。用这个方法，你可以等待所有的观察者都订阅了Observable之后再开始发射数据。

```
ConnectableObservable observable = Observable.range(1, 1000000).sample(10, TimeUnit.MILLISECONDS).publish();

observable.subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        System.out.println("onCompleted1.");
    }

    @Override
    public void onError(Throwable e) {
        System.out.println("onError1: " + e.getMessage());
    }

    @Override
    public void onNext(Integer integer) {
        System.out.println("onNext1: " + integer);
    }
});

observable.subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        System.out.println("onCompleted2.");
    }

    @Override
    public void onError(Throwable e) {
        System.out.println("onError2: " + e.getMessage());
    }

    @Override
    public void onNext(Integer integer) {
        System.out.println("onNext2: " + integer);
    }
});

observable.connect();

```

在订阅的时候，并不会开始发射数据，只有等到connect连接后，才开始发射数据，所以两个观察者接收到的数据是一样的。

### sort 






