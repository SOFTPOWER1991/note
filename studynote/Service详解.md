# 什么是Service

# Service的生命周期

# 如何创建一个服务

# 如何启动一个服务

# Service的启动方式分那几种

# Service两种启动方式的区别

# 绑定本地服务和绑定远程服务的步骤

# 完成一个服务创建、启动、绑定、解绑、销毁的实例

# intentService的作用

如果有一个任务可以分成很多个子任务，需按照顺序完成。如果需放在一个service中完成，那么使用IntentService是最好的选择。

* Service在主线程当中使用，在Service中作耗时操作，会卡死主线程；
* IntentService的优点：
    
   1. 内置独立工作线程来处理一个个Intent;
   2. 创建了一个工作队列，来逐个发送Intent给onHandleIntent;
   3. 不需要主动调用stopSelf来结束服务，自己已经实现；
   4. 默认实现了onBind()返回null
   5. 默认实现了onStartCommand（）的目的是将intent插入到工作队列

   
总结：使用intentService的好处

1. 省去了手动开线程的麻烦
2. 执行完后，不用手动停止service
3. 设计了一个工作队列，可以启动多次——startSerice()，但只有一个Service实例和一个工作线程。



# 如何保证service在后台不被Kill （Service保活）

## 一、onStartCommand方法，返回START_STICKY
	
1.	START_STICKY 在运行onStartCommand后service进程被kill后，那将保留在开始状态，但是不保留那些传入的intent。不久后service就会再次尝试重新创建，因为保留在开始状态，在创建     service后将保证调用onstartCommand。如果没有传递任何开始命令给service，那将获取到null的intent。 
2.	START_NOT_STICKY 在运行onStartCommand后service进程被kill后，并且没有新的intent传递给它。Service将移出开始状态，并且直到新的明显的方法（startService）调用才重新创建。因为如果没有传递任何未决定的intent那么service是不会启动，也就是期间onstartCommand不会接收到任何null的intent。 
3.	START_REDELIVER_INTENT 在运行onStartCommand后service进程被kill后，系统将会再次启动service，并传入最后一个intent给onstartCommand。直到调用stopSelf(int)才停止传递intent。如果在被kill后还有未处理好的intent，那被kill后服务还是会自动启动。因此onstartCommand不会接收到任何null的intent。


## 二、提升service优先级

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

## 三、提升service进程优先级
Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收。Android将进程分为6个等级,它们按优先级顺序由高到低依次是:

1.	前台进程( FOREGROUND_APP)
2.	可视进程(VISIBLE_APP )
3.	次要服务进程(SECONDARY_SERVER )
4.	后台进程 (HIDDEN_APP)
5.	内容供应节点(CONTENT_PROVIDER)
6.	空进程(EMPTY_APP)

当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以使用startForeground 将service放到前台状态。这样在低内存时被kill的几率会低一些。

## 四、onDestroy方法里重启service
service +broadcast  方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

## 五、Application加上Persistent属性

## 六、监听系统广播判断Service状态

通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限啊。



 


