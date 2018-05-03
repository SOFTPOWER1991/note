### GT 是什么？

> GT 腾讯移动品质中心开源的一个工具集，是一个可以直接在手机上运行的“集成调测环境”。

### GT 用来解决什么问题？
> 
> 1. 对APP进行快速的性能测试，包括：CPU、内存、流量、电量、帧率、流畅度等等
> 2. 开发日志的查看
> 3. Crash日志查看
> 4. 网络数据包的抓取
> 5. APP内部参数的调试
> 6. 真机代码耗时统计
> 

**最爽的地方：代码都是开源的。如果感觉GT提供的功能不够，还可以利用GT提供的基础API开发满足我们需求的功能。**

### 支持的平台：

GT支持iOS和Android两个手机平台，其中：

* Android版由一个可直接安装的`GT控制台APP` 和 `GT SDK`组成,GT控制台可以独立安装使用，SDK需嵌入到被调试的应用、并利用GT控制台进行信息展示和参数修改。

* iOS版是一个Framework包，需要嵌入到APP工程，编译出带GT的APP才能使用；iPhone、iPad都能支持（我没验证过）


### GT 的使用步骤

1. 安装GT SDK：
	
	> * 将 `gt-sdk-3.1.0.jar` 以及 `jniLibs下的相应native`库引入到项目工程中
	> * 在Application中加入 `GTRController.init(getApplicationContext());`  就完成了GT SDK 的安装

2. 打开手机GT控制台，选择嵌入了GT SDK 的APP。
3. 启动嵌入了SDK的APP，进行一波操作之后，就可以看到相应的检测数据了。
4. 导出数据 data.js 在pc浏览器上可以生成一个详细的检测报告。【优测性能分析】



##  基础性能说明

Cpu: 表示进程、线程的繁忙程度； 
内存：当前进程内存的使用情况，内存占用过高可能会引起内存抖动、OutOfMemory异常； 
流量：表示当前进程网络的使用情况； 


### 一、cpu:

/proc 文件系统：
> /proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它以文件系统的方式为内核与进程提供通信的接口。用户和应用程序可以通过/proc得到系统的信息，并可以改变内核的某些参数。由于系统的信息，如进程，是动态改变的，所以用户或应用程序读取/proc目录中的文件时，proc文件系统是动态从系统内核读出所需信息并提交的。 从proc文件中可以获取系统、进程、线程的cpu时间片使用情况，所以两次采集时间片的数据就可以获取进程CPU占用率， CPU占用率 = (进程T2-进程T1)/(系统T2-系统T1) 的时间片比值。

1. 获取系统CPU时间片

获取系统CPU时间片使用情况：读取proc/stat，文件的内容如下：



```
cpu 2032004 102648 238344 167130733 758440 15159 17878 0
cpu0 1022597 63462 141826 83528451 366530 9362 15386 0
cpu1 1009407 39185 96518 83602282 391909 5796 2492 0
intr 303194010 212852371 3 0 0 11 0 0 2 1 1 0 0 3 0 11097365 0 72615114 6628960 0 179 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
ctxt 236095529
btime 1195210746
processes 401389
procs_running 1
procs_blocked 0

```

**问题： 如何读取proc/stat 下的文件？**

第一行各个字段的含义：

```
	user (14624) 从系统启动开始累计到当前时刻，处于用户态的运行时间，不包含 nice值为负进程。 
	nice (771) 从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间 
	system (8484) 从系统启动开始累计到当前时刻，处于核心态的运行时间 
	idle (283052) 从系统启动开始累计到当前时刻，除IO等待时间以外的其它等待时间 
	iowait (0) 从系统启动开始累计到当前时刻，IO等待时间(since 2.5.41) 
	irq (0) 从系统启动开始累计到当前时刻，硬中断时间(since 2.6.0-test4) 
	softirq (62) 从系统启动开始累计到当前时刻，软中断时间(since 2.6.0-test4) 
```

总的cpu时间: totalCpuTime = user + nice + system + idle + iowait + irq + softirq

2.获取进程和线程的CPU时间片

获取进程CPU时间片使用情况：读取proc/pid/stat，
获取线程CPU时间片使用情况：读取proc/pid/task/tid/stat，
这两个文件的内容相同，如下：

```
6873 (a.out) R 6723 6873 6723 34819 6873 8388608 77 0 0 0 41958 31 0 0 25 0 3 0 5882654 1409024 56 4294967295 134512640 134513720 3215579040 0 2097798 0 0 0 0 0 0 0 17 0 0 0
```
各个字段的含义：

```
pid=6873 进程(包括轻量级进程，即线程)号
	comm=a.out 应用程序或命令的名字
	task_state=R 任务的状态，R:runnign, S:sleeping (TASK_INTERRUPTIBLE), D:disk sleep (TASK_UNINTERRUPTIBLE), T: stopped, T:tracing stop,Z:zombie, X:dead
	ppid=6723 父进程ID
	pgid=6873 线程组号
	sid=6723 c该任务所在的会话组ID
	tty_nr=34819(pts/3) 该任务的tty终端的设备号，INT（34817/256）=主设备号，（34817-主设备号）=次设备号
	tty_pgrp=6873 终端的进程组号，当前运行在该任务所在终端的前台任务(包括shell 应用程序)的PID。
	task->flags=8388608 进程标志位，查看该任务的特性
	min_flt=77 该任务不需要从硬盘拷数据而发生的缺页（次缺页）的次数
	cmin_flt=0 累计的该任务的所有的waited-for进程曾经发生的次缺页的次数目
	maj_flt=0 该任务需要从硬盘拷数据而发生的缺页（主缺页）的次数
	cmaj_flt=0 累计的该任务的所有的waited-for进程曾经发生的主缺页的次数目
	utime=1587 该任务在用户态运行的时间，单位为jiffies
	stime=1 该任务在核心态运行的时间，单位为jiffies
	cutime=0 累计的该任务的所有的waited-for进程曾经在用户态运行的时间，单位为jiffies
	cstime=0 累计的该任务的所有的waited-for进程曾经在核心态运行的时间，单位为jiffies
	priority=25 任务的动态优先级
	nice=0 任务的静态优先级
	num_threads=3 该任务所在的线程组里线程的个数
	it_real_value=0 由于计时间隔导致的下一个 SIGALRM 发送进程的时延，以 jiffy 为单位.
	start_time=5882654 该任务启动的时间，单位为jiffies
	vsize=1409024（page） 该任务的虚拟地址空间大小
	rss=56(page) 该任务当前驻留物理地址空间的大小
	rlim=4294967295（bytes） 该任务能驻留物理地址空间的最大值
	start_code=134512640 该任务在虚拟地址空间的代码段的起始地址
	end_code=134513720 该任务在虚拟地址空间的代码段的结束地址
	start_stack=3215579040 该任务在虚拟地址空间的栈的结束地址
	kstkesp=0 esp(32 位堆栈指针) 的当前值, 与在进程的内核堆栈页得到的一致.
	kstkeip=2097798 指向将要执行的指令的指针, EIP(32 位指令指针)的当前值.
	pendingsig=0 待处理信号的位图，记录发送给进程的普通信号
	block_sig=0 阻塞信号的位图
	sigign=0 忽略的信号的位图
	sigcatch=082985 被俘获的信号的位图
	wchan=0 如果该进程是睡眠状态，该值给出调度的调用点
	nswap 被swapped的页数，当前没用
	cnswap 所有子进程被swapped的页数的和，当前没用
	exit_signal=17 该进程结束时，向父进程所发送的信号
	task_cpu(task)=0 运行在哪个CPU上
	task_rt_priority=0 实时进程的相对优先级别
	task_policy=0 进程的调度策略，0=非实时进程，1=FIFO实时进程；2=RR实时进程

```

进程的总Cpu时间 processCpuTime = utime + stime + cutime + cstime
线程的总Cpu时间 threadCpuTime = utime + stime + cutime + cstime

我们统计的CPU使用，已经将GT引入线程的损耗在总体的CPU使用中排除，因此结果可靠；

二，内存：

1. 系统内存

1) 系统内存总容量：只需要读取/proc/meminfo 文件的第一个字段MemTotal就可以了，文件内容如下：

```
MemTotal:          94096 kB
	MemFree:            1684 kB
	Buffers:              16 kB
	Cached:            27160 kB
	SwapCached:            0 kB
	Active:            35392 kB
	Inactive:          44180 kB
	Active(anon):      26540 kB
	Inactive(anon):    28244 kB
	Active(file):       8852 kB
	Inactive(file):    15936 kB
	Unevictable:         280 kB
	Mlocked:               0 kB
	SwapTotal:             0 kB
	SwapFree:              0 kB
	Dirty:                 0 kB
	Writeback:             0 kB
	AnonPages:         52688 kB
	Mapped:            17960 kB
	Slab:               3816 kB
	SReclaimable:        936 kB
	SUnreclaim:         2880 kB
	PageTables:         5260 kB
	NFS_Unstable:          0 kB
	Bounce:                0 kB
	WritebackTmp:          0 kB
	CommitLimit:       47048 kB
	Committed_AS:    1483784 kB
	VmallocTotal:     876544 kB
	VmallocUsed:       15456 kB
	VmallocChunk:     829444 kB

```
各个字段的含义:

```
	MemTotal: 所有可用RAM大小。
	MemFree: LowFree与HighFree的总和，被系统留着未使用的内存。
	Buffers: 用来给文件做缓冲大小。
	Cached: 被高速缓冲存储器（cache memory）用的内存的大小（等于diskcache minus SwapCache）。
	SwapCached:被高速缓冲存储器（cache memory）用的交换空间的大小。已经被交换出来的内存，仍然被存放在swapfile中，用来在需要的时候很快的被替换而不需要再次打开I/O端口。
	Active: 在活跃使用中的缓冲或高速缓冲存储器页面文件的大小，除非非常必要，否则不会被移作他用。
	Inactive: 在不经常使用中的缓冲或高速缓冲存储器页面文件的大小，可能被用于其他途径。
	SwapTotal: 交换空间的总大小。
	SwapFree: 未被使用交换空间的大小。
	Dirty: 等待被写回到磁盘的内存大小。
	Writeback: 正在被写回到磁盘的内存大小。
	AnonPages：未映射页的内存大小。
	Mapped: 设备和文件等映射的大小。
	Slab: 内核数据结构缓存的大小，可以减少申请和释放内存带来的消耗。
	SReclaimable:可收回Slab的大小。
	SUnreclaim：不可收回Slab的大小（SUnreclaim+SReclaimable＝Slab）。
	PageTables：管理内存分页页面的索引表的大小。
	NFS_Unstable:不稳定页表的大小。

```

2) 系统空闲的内存：需要通过ActivityManager即可获取：

```
//系统空闲内存
public static long getSysFreeMemory(Context context) {
   ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
   ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();
   am.getMemoryInfo(mi);
   return mi.availMem;
}
```

```
//进程内存上限
public static int getMemoryMax() {
   return (int) (Runtime.getRuntime().maxMemory()/1024);
}
```

3)进程总内存：

```
//进程总内存
public static int getPidMemorySize(int pid, Context context) {
   ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
   int[] myMempid = new int[] { pid };
   Debug.MemoryInfo[] memoryInfo = am.getProcessMemoryInfo(myMempid);
   int memSize = memoryInfo[0].getTotalPss();
	//        dalvikPrivateDirty： The private dirty pages used by dalvik。
	//        dalvikPss ：The proportional set size for dalvik.
	//        dalvikSharedDirty ：The shared dirty pages used by dalvik.
	//        nativePrivateDirty ：The private dirty pages used by the native heap.
	//        nativePss ：The proportional set size for the native heap.
	//        nativeSharedDirty ：The shared dirty pages used by the native heap.
	//        otherPrivateDirty ：The private dirty pages used by everything else.
	//        otherPss ：The proportional set size for everything else.
	//        otherSharedDirty ：The shared dirty pages used by everything else.
   return memSize;
}

```

**拿到这么多的信息，我怎么用？**

三、流量：

TrafficStats 类是又Android提供的一个从你的手机开机开始，累计到现在使用的流量总量，或者统计某个或多个进程或应用所使用的流量，这个流量包括的WiFi 和移动数据网GPRS。

```
//系统流量统计：
	TrafficStats.getTotalRxBytes() ——获取从此次开机起总接受流量（流量是分为上传与下载两类的，当然其实这里还有本地文件之间数据交换的流量，这个暂且不说，等下说明一下我遇到的问题）；
	TrafficStats.getTotalTxBytes()——获取从此次开机起总发送流量；
	TrafficStats.getMobileRxBytes()——获取从此次开机起不包括Wifi的接受流量，即只统计数据网Gprs接受的流量；
	TrafficStats.getMobileTxBytes()——获取从此次开机起不包括Wifi的发送流量，即只统计数据网Gprs发送的流量；
	//进程流量统计：
	TrafficStats.getUidRxBytes(mUid)
    TrafficStats.getUidTxBytes(mUid)	
``` 

获取进程流量的方法：

```
public  static TrafficInfo collect(int mUid) {
   long upload  = TrafficStats.getUidRxBytes(mUid);
   long download = TrafficStats.getUidTxBytes(mUid);

   TrafficInfo trafficInfo = new TrafficInfo();
   trafficInfo.upload = upload;
   trafficInfo.download = download;

  return  trafficInfo;
}
```

## 布局检测原理以及规则

View构建时长：
> View在使用之前需要进行inflate操作，此操作在主线程执行且耗时严重，通常是造成卡顿的直接原因。

View绘制深度：
> View的绘制深度决定着当前试图的复杂度，复杂度越高，越容易引起卡顿。**复杂度高的视图是优化的重点。**

一、 View构建时长：

View 构建时通过调用inflate函数实现的，setContentView的原理也是通过inflate函数构建View，最佳Hook节点。

```
//hook的函数:
	hook android.view.LayoutInflater.inflate		:inflate
	hook android.app.Activity.setContentView		:setContentView

    //举个例子：
    @HookAnnotation(className = "android.view.LayoutInflater")
    public View inflate(int resource, ViewGroup root) {
        LogUtil.e(TAG,"LayoutInflater.inflate");
        String resourceName="未知";
        if (PDM.getContext()!=null){
            try{
                resourceName = PDM.getContext().getResources().getResourceName(resource);
            }catch (Exception e){ }
        }
        long start = System.currentTimeMillis();
        View  view = KHookManager.getInstance().callOriginalMethod("android.view.LayoutInflater.inflate", this, resource,root);
        long end = System.currentTimeMillis();
        ViewBuildCollector.onLayoutInflater_inflate(resourceName,start,end);
        return view;
    }

```





## 流畅性检测原理以及规则

流畅性检测：简单的来讲就是APP的流畅程度，可以用来衡量的维度有：流畅度检测、页面启动时长、Fragment启动时长，这里主要说流畅度检测。

Android系统在流畅的情况下绘帧速度是60帧/s(即：16.7ms一帧)。当绘帧间隔超过一定时长，我们就可以说此时掉帧了，也就会造成用户直接感官的卡顿。此模块可以统计一秒内绘帧次数（即：流畅度SM），并对丢帧的原因进行代码定位。 


一 流畅度检测：

首先Android的帧绘制流程是：

> CPU主线程图像处理->GPU进行光栅化->显示帧。APP产生掉帧的情况大多是由“CPU主线程图像处理”这一步超负载引起的，所以我们思考如何去监控主线程绘制情况。要检测CPU绘制帧的时间，就必须找到那个调用View.dispatchDraw的类，Choreographer类就是那个接受系统垂直同步信号，在每次接受时同步信号触发View的Input、Animation、Draw等操作。


所以我们可以向Choreographer类中加入自己的Callback来冒充View的Callback,通过此Callback我们可以获得View绘制的时间、可以统计一秒内帧绘制的能力（后面把此值称作“流畅值SM”，它能直观的代表当前时间段的流畅度）。之所以不用FPS来代表当前流畅度，是因为Android系统默认在前台页面静止时，FPS可能为0，FPS低无法直接代表当前处于卡顿。


第一步，代码实现:

```
long lastTime = 0;
    long thisTime = 0;
    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    void startMonitor(){
        Choreographer.FrameCallback frameCallback = new Choreographer.FrameCallback() {//系统绘帧回调
            public void doFrame(long frameTimeNanos) {

                thisTime = System.currentTimeMillis();
                //累计流畅值
                plusSM(thisTime);//当前秒的SM+1，如果当前秒数已经到下一秒，则将此SM值写入文件
                //判别超时：
                if (thisTime - lastTime > 40 && lastTime!=0){
                    Log.e("DDDD","frame超时"+(thisTime - lastTime)+"ms: "+ lastTime +"-"+ thisTime);
                    //saveBlockInfo(lastTime, thisTime);//此处保存卡顿信息
                }
                //设置下一帧绘制的回调
                Choreographer.getInstance().postFrameCallback(this);//设置下次系统绘帧回调
                lastTime = thisTime;
            }
        };
        Choreographer.getInstance().postFrameCallback(frameCallback);
    }

    //保存当前SM值
    private long nowTime =1;//当前的时间（ms）
    private int sm = 1;
    private void plusSM(long t){
        if (nowTime ==1){
            nowTime = t;
        }
        if (nowTime/1000 == t/1000){
            sm++;
        }else if (t/1000 - nowTime/1000 >=1){
            //saveSMInfo(sm,t);//此处保存此时的流畅值SM
            Log.e("DDDD","sm："+sm);
            sm=1;
            nowTime = t;
        }
    }

```
第二步，运行结果，可以在消息执行的过程中进行拉栈操作，用于耗时代码的定位；

```
07-02 22:32:51.721 15931-15931/com.utest.elvistestapplication E/DDDD: frame超时113ms: 1499005971618-1499005971731
	07-02 22:32:51.781 15931-15931/com.utest.elvistestapplication E/DDDD: frame超时55ms: 1499005971731-1499005971786
	07-02 22:32:52.001 15931-15931/com.utest.elvistestapplication E/DDDD: sm：17
	07-02 22:32:53.011 15931-15931/com.utest.elvistestapplication E/DDDD: sm：60
	07-02 22:32:53.991 15931-15931/com.utest.elvistestapplication E/DDDD: sm：59
	07-02 22:32:54.991 15931-15931/com.utest.elvistestapplication E/DDDD: sm：60
	07-02 22:32:56.001 15931-15931/com.utest.elvistestapplication E/DDDD: sm：60
	......

```

评判规则:

1. 单次大卡顿：当两次绘帧间隔大于70ms，相当于丢了4帧以上，建议开发人员对耗时的代码进行异步或拆分。
2. 低流畅值区间：流畅值低于40帧/s的区间，导致低流畅值区间出现的原因有两类：“单次大卡顿”“连续小卡顿”，建议开发人员针对不同的场景进行优化。



## 页面启动时长检测原理以及规则

Activity启动时长就是唤醒Activityy到Activity在前台进行第一次绘制的时间，配合“绘帧检测”中定位的掉帧区间，可以直观的展示卡顿问题。

1. 实现原理

a: 对Activity的生命周期监控：

Android 4.0以上的版本可以利用ActivityLifecycleCallbacks来实现对生命周期的监听，但此方法无法得到每个生命周期函数的执行时长。因此我们采用Hook的方式来监控Activity生命周期，这里介绍一下最佳Hook节点：

```
所有Hook节点：
hook android.app.Instrumentation.execStartActivity函数 :startActivity函数
hook android.app.Instrumentation.callActivityOnCreate函数:onCreate函数
hook android.app.Instrumentation.callActivityOnStart函数:onStart函数
hook android.app.Instrumentation.callActivityOnResume函数:onResume函数
hook android.app.Instrumentation.callActivityOnPause函数:onPause函数
hook android.app.Instrumentation.callActivityOnStop函数:onStop函数

举个例子：
@HookAnnotation(className = "android.app.Instrumentation")
public void callActivityOnStart(Activity activity) {
   LogUtil.e(TAG,"Instrumentation.callActivityOnStart");
   long start = System.currentTimeMillis();
   KHookManager.getInstance().callOriginalMethod("android.app.Instrumentation.callActivityOnStart", this, activity);
   long end = System.currentTimeMillis();
   ActivityCollector.onInstrumentation_callActivityOnStart(activity,start,end);//PageLoad模块
}
```
Hook数据的结果：

```
07-03 10:39:39.922 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnCreate
07-03 10:39:40.092 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnStart
07-03 10:39:40.102 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnResume
07-03 10:39:51.192 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.execStartActivity
07-03 10:39:51.222 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnPause
07-03 10:39:51.242 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnCreate
07-03 10:39:51.652 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnStart
07-03 10:39:51.652 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnResume
07-03 10:39:52.042 28363-28363/com.utest.pdm.example E/_PDM_HookList_activity: Instrumentation.callActivityOnStop
......
```

如何进行数据的整理：

> 页面分为冷启动和热启动（页面从startActivity开始则是冷启动，如果从onStart或onResume开始，则是热启动），我们可以维护一个页面列表pageList,然后通过hashCode和生命周期函数的执行时间来归类数据，并可以对页面的冷热启动进行分析。

对View绘制的监控：

对View绘制的监控， 只需要hook ViewGroup的dispatchDraw方法：

```
@HookAnnotation(className = "android.view.ViewGroup")
protected void dispatchDraw(Canvas canvas) {
   LogUtil.e(TAG,"ViewGroup.dispatchDraw");
   Object view = this;
   ViewDrawCollector.onViewGroup_dispatchDraw_before((View) view);//PageLoad模块
   KHookManager.getInstance().callOriginalMethod("android.view.ViewGroup.dispatchDraw", this, canvas);
   ViewDrawCollector.onViewGroup_dispatchDraw_after((View) view);//PageLoad模块
}

```

首先是对View绘制数据是杂乱无章的，由于ViewGroup的执行是递归的，所以我们发明了一种递归压栈归类法（将当前绘制节点进行压栈和弹栈操作），而且通过最大栈深可以得知View的绘制深度：

```
private static int stackSize = 0;//dispatchDraw的栈深，每当栈弹光，说明一次绘制完成
public static DrawInfo drawInfo;//代表一次绘制对象

public static synchronized void onViewGroup_dispatchDraw_before(View view) {
	//绘制开始--创建对象
   if (stackSize == 0) {
       drawInfo = new DrawInfo();
	}
	//将draw数据保存在drawInfo中
	//...省略n行.... 
   stackSize++;
	if(stackSize>drawInfo.drawDeep){
       drawInfo.drawDeep = stackSize;//保存最大绘制深度
   }
}

public static synchronized void onViewGroup_dispatchDraw_after(View view) {
   stackSize--;
	//绘制栈弹光，则说明当前进行了一次完整的递归绘制，保存drawInfo数据
   if (stackSize == 0 && drawInfo != null) {
       drawInfo.drawEnd = System.currentTimeMillis();
       for (CallBack callBack : callBackArrayList){
           callBack.onDrawFinish(drawInfo);
       }
   }
}

```

其次是将viewDraw信息匹配给Activity：

```
//原理：前面说过，将页面对象数据存成一个pageList,当一次绘制完成后，我们先检查此绘制是否为前
	//一个页面的绘制信息，是则将此绘制数据add到之前页面对象中，否则该绘制信息是新页面的绘制信息。
    public void onDrawFinish(DrawInfo drawInfo) {
        synchronized (lock){
            //检查此绘制是否为前一个页面的绘制信息
            if (pageList.size()>=2 && pageList.get(pageList.size()-2).isMyDraw(drawInfo)){
                pageList.get(pageList.size()-2).addDrawInfo(drawInfo);
                LogUtil.e("Draw","绘制赋值:"+drawInfo.drawClassName +" -> "+pageList.get(pageList.size()-2).activityClassName);
            }else if(pageList.size()>=1){
                pageList.get(pageList.size()-1).addDrawInfo(drawInfo);
                LogUtil.e("Draw","绘制赋值:"+drawInfo.drawClassName +" -> "+ pageList.get(pageList.size()-1).activityClassName);
            }
        }
    }
	//判别是否属于前一个页面的绘制信息：判断此View的hashCode是否在前一个页面显示的时候绘制过。
    public boolean isMyDraw(DrawInfo drawInfo){
        for (DrawInfo d:drawInfoList){
            if (d.objectHashCode.equals(drawInfo.objectHashCode)){
                return true;
            }
        }
        return false;
    }

```

页面启动时长就是唤醒Activity到Activity在前台进行第一次绘制的时间。以下三种情况则可认为页面启动卡顿或启动超时：

1. 启动时长超过250ms
2. 页面1秒内卡顿超过300ms
3. 页面5秒内卡顿超过500ms


##  Fragment启动时长：

Fragment启动时长就是唤醒Fragment到Fragment执行onResume的完成时间。

1. 实现原理:

对Fragment生命周期的监控： 我们同样采用Hook的方式来监控Fragment生命周期，这里介绍一下最佳Hook节点：

```
//android.app.Fragment包:
hook android.app.Fragment.onAttach					:onAttach
hook android.app.Fragment.performCreate				:onCreate
hook android.app.Fragment.performCreateView			:onCreateView
hook android.app.Fragment.performActivityCreated	:onActivityCreated
hook android.app.Fragment.performStart				:onStart
hook android.app.Fragment.performResume				:onResume
hook android.app.Fragment.performPause				:onPause
hook android.app.Fragment.performStop				:onStop
hook android.app.Fragment.performDestroyView		:onDestoryView
hook android.app.Fragment.performDestroy			:onDestory
hook android.app.Fragment.performDetach				:onDetach
hook android.app.Fragment.onHiddenChanged			:onHiddenChanged
hook android.app.Fragment.setUserVisibleHint		:setUserVisibleHint


//android.support.v4.app.Fragment包：
hook android.support.v4.app.Fragment.onAttach					:onAttach
hook android.support.v4.app.Fragment.performCreate				:onCreate
hook android.support.v4.app.Fragment.performCreateView			:onCreateView
hook android.support.v4.app.Fragment.performActivityCreated		:onActivityCreated
hook android.support.v4.app.Fragment.performStart				:onStart
hook android.support.v4.app.Fragment.performResume				:onResume
hook android.support.v4.app.Fragment.performPause				:onPause
hook android.support.v4.app.Fragment.performStop				:onStop
hook android.support.v4.app.Fragment.performDestroyView			:onDestoryView
hook android.support.v4.app.Fragment.performDestroy				:onDestory
hook android.support.v4.app.Fragment.performDetach				:onDetach
hook android.support.v4.app.Fragment.onHiddenChanged			:onHiddenChanged
hook android.support.v4.app.Fragment.setUserVisibleHint			:setUserVisibleHint

//举个例子：	
@HookAnnotation(className = "android.app.Fragment")
void performCreate(Bundle savedInstanceState) {
   LogUtil.e(TAG,"performCreate");
   long start = System.currentTimeMillis();
   KHookManager.getInstance().callOriginalMethod("android.app.Fragment.performCreate", this, savedInstanceState);
   long end = System.currentTimeMillis();
   String activityClassName = "";
   String activityHashCode = "";
   String fragmentClassName = "";
   String fragmentHashCode = "";
   Object fragment = this;
   if (fragment instanceof android.app.Fragment){
       fragmentClassName =  ((android.app.Fragment)fragment).getClass().getName();
       fragmentHashCode = ""+this.hashCode();
       Activity activity = ((android.app.Fragment) fragment).getActivity();
       if (activity!=null){
           activityClassName  = activity.getClass().getName();
           activityHashCode = ""+activity.hashCode();
       }
   }
   FragmentCollector.onFragment_performCreate(activityClassName, activityHashCode,fragmentClassName,fragmentHashCode,start,end);//Fragment模块
}

```

Hook数据的结果：

```
07-03 15:21:02.831 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: onAttach
	07-03 15:21:02.831 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: performCreate
	07-03 15:21:02.831 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: performCreateView
	07-03 15:21:02.841 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: performActivityCreated
	07-03 15:21:02.841 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: performStart
	07-03 15:21:02.841 22305-22305/com.utest.pdm.example E/_PDM_PDMHookList_Fragment: performResume

```

评判规则:

```
每个页面详细的启动数据，包含了Fragment生命周期、卡顿信息、页面平均流畅值，启动时长Fragment启动时长就是唤醒Fragment到Fragment执行onResume完成的时间。启动时长超过150ms则可认为页面启动卡顿或启动超时
```

