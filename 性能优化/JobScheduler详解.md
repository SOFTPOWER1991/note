第一次听说JobScheduler是在2016年上Google Developer Day上听到的！之后在学性能优化的时候碰到过几次，最近碰到这个东西的频率越来越高，因此决定写下来，一探究竟！

说来惭愧，这东西早在Android5.0的时候就已经引入了，然而我却后知后觉！

关于JobScheduler我们思考如下几个问题，文本也将依次解答如下几个问题：

1. JobScheduler 是什么？
2. JobScheduler 的出现是为了解决什么问题？
3. JobScheduler 的使用场景是什么？
4. 如何使用JobScheduler？（这一部分将会通过一个具体的实例来进行讲解！）
5. 深入理解JobScheduler！这一部分将通过源码阅读的形式深入探究JobScheduler的运作原理，知其然并知其所以然！

下面我将依次解答上面提出来的几个问题：

## 1. JobScheduler 是什么？

JobScheduler是Android 5.0 即API 21引入的一个新增的Api。允许您定义一些系统在稍后或指定条件下（如设备充电时、连接WiFi时）以异步方式运行的作业，从而节省电量的消耗，优化电池寿命。

## 2. JobScheduler 的出现是为了解决什么问题？

减少系统和程序的电量消耗，从而提高电池续航。为此Google 经过测试发现，每次唤醒设备，1-2秒的时候，都会消耗2分钟的待机电量，可见每次唤醒设备的时候，不仅仅是点亮了屏幕，系统也在后台处理很多事情。使用了一个新的API JobScheduler ，这个东西可以让系统批处理一些不重要的APP 请求，比如清理数据库和上传日志等等。研发人员也可以使用这个API减少自己APP 的不必要操作。

## 3. JobScheduler 的使用场景是什么？

如果APP有符合如下情况的任务, 可以尝试使用JobScheduler机制执行任务:

> 1. 可以推迟的非面向用户的任务(如定期数据库数据更新、上传日志比如Crash日志)
> 2. 当充电时才希望执行的工作(如备份数据)
> 3. 需要访问网络或 Wi-Fi 连接的任务(如向服务器拉取内置数据)
> 4. 希望作为一个批次定期运行的许多任务

## 4. 如何使用JobScheduler？

打开Android的官方Api文档可以看到包`android.app.job`下一共有如下几个类

![android_app_job](https://github.com/SOFTPOWER1991/note/blob/master/raw/android_app_job.png)
* JobInfo : 传递给JobScheduler的数据容器完全封装了根据调用应用程序调度工作所需的参数。

* JobInfo.Builder : Builder类用来创建JobInfo对象
* JobInfo.TriggerContentUri：内容UI的修改将会触发一个Job作业
* JobParameters： 包含用于配置/识别作业的参数。
* JobScheduler：这是一个API，依靠Framework来调度各种不同类型的任务。这个动作在你自己的应用程序进程中执行。
* JobService： 从JobScheduler回调的入口点

Google为我们提供了使用JobScheduler的示例：[android-JobScheduler
](https://github.com/googlesamples/android-JobScheduler)。可以参见这个示例，JobScheduler怎么使用。

接下来主要通过这篇博客[How to use Android's Job Scheduler
](http://toastdroid.com/2015/02/21/how-to-use-androids-job-scheduler/)，来了解JobScheduler的使用方法：

Lollipop非常酷的地方在于它引入了全新的JobScheduler API。这不仅对于开发这来说是非常兴奋的事儿对于用户来说也应该高兴。非常让人出乎意料的是你可以非常容易的使用API通过传递一系列的参数来让你的App调度一些任务。这不仅可以帮你的用户延长电池的使用寿命，同时让用户减少卸载你的应用的机会。这个API主要包含3部分：JobInfo,JobService 和 JobScheduler。

### JobInfo 和 可用的参数

通过JobInfo来定义一些参数，然后使用JobInfo.Builder来构建这个JobInfo。下面是你可以设置的参数：

* 	`setBackoffCriteria(long initialBackoffMillis, int backoffPolicy)`。设置重试、退避策略，当一个任务调度失败的时候执行什么样的测量采取重试。`initialBackoffMillis`:第一次尝试重试的等待时间间隔ms数；`backoffPolicy`:对应的退避策略。比如等待的间隔呈指数增长。默认是30 sec 和 指数的。
*  `setExtras(PersistableBundle extras)`:一个附属信息的Bundle。这让你发送一个具体的数据给你的任务。
*  `setMinimumLatency(long minLatencyMillis)`:你的任务最小的延迟时间。
*  `setOverrideDeadline(long maxExecutionDelayMillis)`: 最大的等待执行你的任务的时间。如果一到时间就立马执行忽略你的其它参数。
*  ` setPeriodic(long intervalMillis)`：设置执行周期，每隔一段时间间隔任务最多可以执行一次。
*  ` setPersisted(boolean isPersisted)`：设置设备重启后这个任务是否还要保留。需要 RECEIVE_BOOT_COMPLETED权限。
*  `setRequiredNetworkType(int networkType)`：你想让你的任务在什么网络类型的时候执行。
*  `setRequiresCharging(boolean requiresCharging)`：设备是否需要在充电
*  ` setRequiresDeviceIdle(boolean requiresDeviceIdle)`：是否需要在设备闲置的时候执行任务。这是处理繁重的任务的绝佳时机。

如果我们有一个应用商店我们可能想在如下情况时更新App：

* 在WiFi状况下，来节省用户的流量
* 设备处于空闲状态这样我们就不会让用户感觉到慢。不在屏幕电量的时候进行大量的更新工作。
* 设备正在充电，允许我们帮助用户来延长用户的电池寿命和续航时间。

```
ComponentName serviceName = new ComponentName(context, MyJobService.class);
JobInfo jobInfo = new JobInfo.Builder(JOB_ID, serviceName)
        .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
        .setRequiresDeviceIdle(true)
        .setRequiresCharging(true)
        .build();
```

MyJobService在哪儿？不慌，接下来我们讨论它。

### JobService

JobService是真正执行你的任务的service。它和普通service实现的方法不同。第一个方法是`onStartJob(JobParameters params)`。当JobScheduler决定基于其参数运行作业时，调用此方法。你可以从JobParameters中获得jobId接下来你需要持有这些参数知道任务执行完毕。下一个方法是onStopJob(JobParameters params)。当您的参数不再被满足时，这将被调用。在我们的上一个示例中，当用户关闭WiFi，拔掉或打开设备上的屏幕时，会发生这种情况。这意味着我们要尽快停止我们的工作;在这种情况下，我们不会更新任何更多的应用程序。

在使用JobServices时有3个非常重要的事情需要知道：

* JobService是运行在主线程。你需要在别的线程来处理一些繁重的任务。如果当一个任务正在主线程执行的时候你的用户尝试打开App，可能会得到一个ANR。
* 在你的任务完成的时候你需要结束你的job。因为JobScheduler为你的job持有了一个wake lock。如果你没有调用jobFinished(JobParameters params, boolean needsReschedule) 其中的JobParameters是需要从onStartJob（JobParameters params）中传过来的。JobScheduler将会为你的App持有一个wake lock这会严重消耗设备的电量。最糟糕的情况下，用户可能卸载你的App。
* 你需要在AndroidManifest文件中注册你的job service。如果没有这样做，系统会找不到你的service不会开启你的任务。

```
<application
    .... stuff ....
    >

    <service
        android:name=".MyJobService"
        android:permission="android.permission.BIND_JOB_SERVICE"
        android:exported="true"/>
</application>

```

```
public class MyJobService extends JobService {

    private UpdateAppsAsyncTask updateTask = new UpdateAppsAsyncTask();

    @Override
    public boolean onStartJob(JobParameters params) {
        // Note: this is preformed on the main thread.

        updateTask.execute(params);

        return true;
    }

    // Stopping jobs if our job requires change.

    @Override
    public boolean onStopJob(JobParameters params) {
        // Note: return true to reschedule this job.

        boolean shouldReschedule = updateTask.stopJob(params);

        return shouldReschedule;
    }

    private class UpdateAppsAsyncTask extends AsyncTask<JobParameters, Void, JobParameters[]> {


        @Override
        protected JobParameters[] doInBackground(JobParameters... params) {

          // Do updating and stopping logical here.
          return params;
        }

        @Override
        protected void onPostExecute(JobParameters[] result) {
            for (JobParameters params : result) {
                if (!hasJobBeenStopped(params)) {
                    jobFinished(params, false);
                }
            }
        }

        private boolean hasJobBeenStopped(JobParameters params) {
            // Logic for checking stop.
        }

        public boolean stopJob(JobParameters params) {
            // Logic for stopping a job. return true if job should be rescheduled.
        }

    }
}

```

注意：无论如何在任务结束的时候要调用`jobFinished(JobParameters params, boolean needsReschedule) `。

### JobScheduler

接下来获取JobScheduler来执行JobInfo

```
JobScheduler scheduler = (JobScheduler) context.getSystemService(Context.JOB_SCHEDULER_SERVICE);
int result = scheduler.schedule(jobInfo);
if (result == JobScheduler.RESULT_SUCCESS) Log.d(TAG, "Job scheduled successfully!");

```

## 5. 深入理解JobScheduler

这部分将从源码的角度来解析JobScheduler的工作流程。这个涉及的内容有点懂，将在下一篇博客中介绍！







