7.0 适配：


1. 在7.0上我们的APP遇到了什么问题；
2. 7.0 compile sdk ,target sdk , 
3. 7.0 的新特性，可能会对APP造成什么影响？查阅官方文档。


1. 多窗口支持；
2. 通知
3. 后台优化
4. WebView
5. 作用域目录访问
6. 权限更改，在应用间共享文件
7. 



本文主要包含如下内容：
	1.	易项优选在Android 7.0 上遇到的问题
	2.	Android 7.0版本新特性
	3.	新特性的适配难度

	
易项优选在Android 7.0 上遇到的问题


# Android 7.0版本新特性

* 多窗口支持
>  Android的画中画模式，不是强需求场景。对应文档：https://developer.android.com/guide/topics/ui/multi-window.html?hl=zh-cn

* 通知增强功能
> App中好像没用到Notification相关的内容。对应文档：https://developer.android.com/guide/topics/ui/notifiers/notifications.html?hl=zh-cn

* 低电耗模式限制,将应用调整到低电耗模式
	> 在低电耗模式下，应用会受到以下限制：
	
	>	* 暂停访问网络。
>	*	系统将忽略 wake locks。
>	*	标准 AlarmManager 闹铃（包括 setExact() 和 setWindow()）推迟到下一维护时段。
>	*	如果您需要设置在低电耗模式下触发的闹铃，请使用 setAndAllowWhileIdle() 或 setExactAndAllowWhileIdle()。
>	*	一般情况下，使用 setAlarmClock() 设置的闹铃将继续触发 — 但系统会在这些闹铃触发之前不久退出低电耗模式。
>	*	系统不执行 Wi-Fi 扫描。
>	*	系统不允许运行同步适配器。
>	*	系统不允许运行 JobScheduler
>
>
> 对应文档：https://developer.android.com/training/monitoring-device-state/doze-standby.html?hl=zh-cn#understand_app_standby

* 默认受信任的证书颁发机构，网络安全性配置
> 面向 Android 7.0 的应用仅信任系统提供的证书，且不再信任用户添加的证书颁发机构 (CA)。如果面向 Android N 的应用希望信任用户添加的 CA，则应使用网络安全性配置以指定信任用户 CA 的方式。
> 
> 网络安全性配置文档：https://developer.android.com/training/articles/security-config.html?hl=zh-cn

* 作用域目录访问
> 在 Android 7.0 中，应用可以使用新的 API 请求访问特定的外部存储目录，包括可移动媒体上的目录，如 SD 卡。新 API 大大简化了应用访问标准外部存储目录的方式，如 Pictures 目录。应用（如照片应用）可以使用这些 API（而不是使用 READ_EXTERNAL_STORAGE），其授予所有存储目录的访问权限或存储访问框架，从而让用户可以导航到目录。
> 
> 此外，新的 API 简化了用户向应用授予外部存储访问权限的步骤。当您使用新的 API 时，系统使用一个简单的权限 UI，其清楚地详细介绍应用正在请求访问的目录。
> 
> 对应文档：https://developer.android.com/training/articles/scoped-directory-access.html?hl=zh-cn



* FrameMetricsListener API
> FrameMetricsListener API 允许应用监测它的 UI 渲染性能。API 通过公开流式传输 Pub/Sub API 来提供此能力，以传递应用当前窗口的帧计时信息。返回的数据相当于 adb shell dumpsys gfxinfo framestats 显示的数据，但不限定于在过去的 120 帧内。
> 
> 
> 以使用 FrameMetricsListener 来衡量生产中的交互级 UI 性能，无需 USB 连接。此 API 允许在比 adb shell dumpsys gfxinfo 更高的粒度上收集数据。因为系统可以从应用中的特定交互中收集数据，因此更高的粒度变得可行；系统不需要采集关于完整应用性能的全局概要或清除任何全局状态。您可以使用这种能力来针对应用的真实使用案例收集性能数据和捕捉 UI 性能回归。
> 
要监测一个窗口，实现 FrameMetricsListener.onMetricsAvailable() 回调方法，并在窗口上注册。


# 对易项优选Android APP影响最大的新特性
	
对易项优选Android APP影响最大的新特性是——作用域目录访问。

> 影响：**在未做适配的情况下，投资人在拍照并提交名片信息的时候，应用直接Crash掉。**

**操作方式**
> 1. 找一个未认证的账号，然后登录；
> 2. 跳转到认证上传名片的页面，点击拍照，应用直接Crash。

Crash后抛出如下异常：

```

10-26 17:22:10.456   656   656 E AndroidRuntime: Process: com.ethercap.app.android, PID: 656
10-26 17:22:10.456   656   656 E AndroidRuntime: android.os.FileUriExposedException: file:///storage/emulated/0/Android/data/com.ethercap.app.android/files/ethercap/IMG_20171026_172210.jpg exposed beyond app through ClipData.Item.getUri()
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.StrictMode.onFileUriExposed(StrictMode.java:1816)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.net.Uri.checkFileUriExposed(Uri.java:2350)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.ClipData.prepareToLeaveProcess(ClipData.java:832)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.Intent.prepareToLeaveProcess(Intent.java:9052)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.Intent.prepareToLeaveProcess(Intent.java:9037)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Instrumentation.execStartActivity(Instrumentation.java:1530)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Activity.startActivityForResult(Activity.java:4391)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.BaseFragmentActivityJB.startActivityForResult(BaseFragmentActivityJB.java:48)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.FragmentActivity.startActivityForResult(FragmentActivity.java:77)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.base.android.BaseActivity.startActivityForResult(BaseActivity.java:558)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Activity.startActivityForResult(Activity.java:4335)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.FragmentActivity.startActivityForResult(FragmentActivity.java:859)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.base.android.BaseActivity.startActivityForResult(BaseActivity.java:565)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.app.android.logincertificate.CertificateIdentityActivity$7.onClick(CertificateIdentityActivity.java:183)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.view.View.performClick(View.java:5646)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.view.View$PerformClick.run(View.java:22450)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Handler.handleCallback(Handler.java:755)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Handler.dispatchMessage(Handler.java:95)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Looper.loop(Looper.java:156)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.ActivityThread.main(ActivityThread.java:6524)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at java.lang.reflect.Method.invoke(Native Method)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:941)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:831)
10-26 17:22:10.468  1022  5023 E ReportTools: This is not beta user build
10-26 17:22:10.456   656   656 E AndroidRuntime: Process: com.ethercap.app.android, PID: 656
10-26 17:22:10.456   656   656 E AndroidRuntime: android.os.FileUriExposedException: file:///storage/emulated/0/Android/data/com.ethercap.app.android/files/ethercap/IMG_20171026_172210.jpg exposed beyond app through ClipData.Item.getUri()
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.StrictMode.onFileUriExposed(StrictMode.java:1816)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.net.Uri.checkFileUriExposed(Uri.java:2350)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.ClipData.prepareToLeaveProcess(ClipData.java:832)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.Intent.prepareToLeaveProcess(Intent.java:9052)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.content.Intent.prepareToLeaveProcess(Intent.java:9037)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Instrumentation.execStartActivity(Instrumentation.java:1530)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Activity.startActivityForResult(Activity.java:4391)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.BaseFragmentActivityJB.startActivityForResult(BaseFragmentActivityJB.java:48)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.FragmentActivity.startActivityForResult(FragmentActivity.java:77)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.base.android.BaseActivity.startActivityForResult(BaseActivity.java:558)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.Activity.startActivityForResult(Activity.java:4335)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.support.v4.app.FragmentActivity.startActivityForResult(FragmentActivity.java:859)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.base.android.BaseActivity.startActivityForResult(BaseActivity.java:565)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.ethercap.app.android.logincertificate.CertificateIdentityActivity$7.onClick(CertificateIdentityActivity.java:183)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.view.View.performClick(View.java:5646)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.view.View$PerformClick.run(View.java:22450)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Handler.handleCallback(Handler.java:755)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Handler.dispatchMessage(Handler.java:95)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.os.Looper.loop(Looper.java:156)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at android.app.ActivityThread.main(ActivityThread.java:6524)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at java.lang.reflect.Method.invoke(Native Method)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:941)
10-26 17:22:10.456   656   656 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:831)
```

# Android 7.0 适配调研总结：
> 对于Android 7.0 的适配还是非常有必要的，主要适配的新特性：**作用于目录访问的适配**，其余的新特性并没什么影响。



