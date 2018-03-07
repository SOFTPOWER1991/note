# 一、配置

**1. Android Studio 配置**

app/build.gradle 下（用可以通过引入aar、jar那种形式）：

```
android {
        defaultConfig {
          ndk {
            //设置支持的SO库架构
            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
          }
        }
      }
      dependencies {
          //注释掉原有bugly的仓库
          //compile 'com.tencent.bugly:crashreport:latest.release'//其中latest.release指代最新版本号，也可以指定明确的版本号，例如2.3.2
          compile 'com.tencent.bugly:crashreport_upgrade:latest.release'//其中latest.release指代最新版本号，也可以指定明确的版本号，例如1.2.0
          compile 'com.tencent.bugly:nativecrashreport:latest.release' //其中latest.release指代最新版本号，也可以指定明确的版本号，例如2.2.0
      }
```

**2. AndroidManifest.xml 中的配置**

* 权限
	
	```
	<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_LOGS" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	```
* Activity配置

```
<activity
    android:name="com.tencent.bugly.beta.ui.BetaActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|locale"
    android:theme="@android:style/Theme.Translucent" />
```

* 配置FileProvider

> 注意：如果您想兼容Android N或者以上的设备，必须要在AndroidManifest.xml文件中配置FileProvider来访问共享路径的文件。

```
 <provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>
```

如果你使用的第三方库也配置了同样的FileProvider, 可以通过继承FileProvider类来解决合并冲突的问题，示例如下：

```
<provider
    android:name=".utils.BuglyFileProvider"
    android:authorities="${applicationId}.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true"
    tools:replace="name,authorities,exported,grantUriPermissions">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"
        tools:replace="name,resource"/>
</provider>
```

这里要注意一下，FileProvider类是在support-v4包中的，检查你的工程是否引入该类库。
在res目录新建xml文件夹，创建provider_paths.xml文件如下：

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- /storage/emulated/0/Download/${applicationId}/.beta/apk-->
    <external-path name="beta_external_path" path="Download/"/>
    <!--/storage/emulated/0/Android/data/${applicationId}/files/apk/-->
    <external-path name="beta_external_files_path" path="Android/data/"/>
</paths>
```

这两个路径是干什么的？
> 这里配置的两个外部存储路径是升级SDK下载的文件可能存在的路径，一定要按照上面格式配置，不然可能会出现错误。

**3.混淆配置**

为了避免混淆SDK，在Proguard混淆文件中增加以下配置。

```
-dontwarn com.tencent.bugly.**
-keep public class com.tencent.bugly.**{*;}
-keep class android.support.**{*;}
```

# 二、基本使用：初始化

```
Bugly.init(getApplicationContext(), "注册时申请的APPID", false);
```

* 参数1： 上下文；
* 参数2：注册时申请的AppleID
* 参数3：是否开启debug模式，true表示打开debug模式，false表示关闭调试模式

**文档如是说：**
>  init方法会自动检测更新，不需要再手动调用`Beta.checkUpgrade()`, 如需增加自动检查时机可以使用`Beta.checkUpgrade(false,false)`;
> 
> 参数1：isManual 用户手动点击检查，非用户点击操作请传false
> 参数2：isSilence 是否显示弹窗等交互，[true:没有弹窗和toast] [false:有弹窗或toast]

# 三、高级使用

Bugly提供了Beta类，它的作用：
> 作为Bugly的初始化扩展，通过Beta类可以修改升级的检测时机，界面元素以及自定义升级行为

* 自动初始化开关

```
Beta.autoInit = true;
```
> 1. true表示app启动自动初始化升级模块; false不会自动初始化; 
> 2. 如果担心sdk初始化影响app启动速度，可以设置为false，在后面某个时刻手动调用Beta.init(getApplicationContext(),false);
> (其中debug表示：是否开启debug模式)

* 自动检查更新开关

```
Beta.autoCheckUpgrade = true;
```
> 1. true表示初始化时自动检查升级; 
> 2. false表示不会自动检查升级,需要手动调用Beta.checkUpgrade()方法;

* 升级检查周期设置

```
Beta.upgradeCheckPeriod = 60 * 1000;
```
> 设置升级检查周期为60s(默认检查周期为0s)，60s内SDK不重复向后台请求策略);

* 延迟初始化

```
Beta.initDelay = 1 * 1000;
```
> 设置启动延时为1s（默认延时3s），APP启动1s后初始化SDK，避免影响APP启动速度;

* 设置通知栏大图标

```
Beta.largeIconId = R.drawable.ic_launcher;
```
> largeIconId为项目中的图片资源;

* 设置状态栏小图标

```
Beta.smallIconId = R.drawable.ic_launcher;
```
> smallIconId为项目中的图片资源id

* 设置更新弹窗默认展示的banner

```
Beta.defaultBannerId = R.drawable.ic_launcher;
```
> defaultBannerId为项目中的图片资源Id; 当后台配置的banner拉取失败时显示此banner，默认不设置则展示“loading...“;

* 设置sd卡的Download为更新资源存储目录

```
Beta.storageDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS);
```
> 后续更新资源会保存在此目录，需要在manifest中添加WRITE_EXTERNAL_STORAGE权限;

* 添加可显示弹窗的Activity

```
Beta.canShowUpgradeActs.add(MainActivity.class);
```
> 只允许在MainActivity上显示更新弹窗，其他activity上不显示弹窗; 如果不设置默认所有activity都可以显示弹窗。

* 设置开启显示打断策略

```
Beta.showInterruptedStrategy = true;
```
> 设置点击过确认的弹窗在App下次启动自动检查更新时会再次显示。

* 设置自定义升级对话框UI布局

```
Beta.upgradeDialogLayoutId = R.layout.upgrade_dialog;
```
> upgrade_dialog为项目的布局资源。 注意：因为要保持接口统一，需要用户在指定控件按照以下方式设置tag，否则会影响您的正常使用： - 特性图片：beta_upgrade_banner，如：android:tag="beta_upgrade_banner"
		标题：beta_title，如：android:tag="beta_title" 
		升级信息：beta_upgrade_info 如： android:tag="beta_upgrade_info" 
		更新属性：beta_upgrade_feature 如： android:tag="beta_upgrade_feature" 
		取消按钮：beta_cancel_button 如：android:tag="beta_cancel_button" 
		确定按钮：beta_confirm_button 如：android:tag="beta_confirm_button" 


* 设置自定义tip弹窗UI布局

```
Beta.tipsDialogLayoutId = R.layout.tips_dialog;
```
> 注意：因为要保持接口统一，需要用户在指定控件按照以下方式设置tag，否则会影响您的正常使用：
		标题：beta_title，如：android:tag="beta_title" 
		提示信息：beta_tip_message 如： android:tag="beta_tip_message" 
		取消按钮：beta_cancel_button 如：android:tag="beta_cancel_button" 
		确定按钮：beta_confirm_button 如：android:tag="beta_confirm_button" 

* 设置升级对话框生命周期回调接口

```
Beta.upgradeDialogLifecycleListener = new UILifecycleListener<UpgradeInfo>() {
            @Override
            public void onCreate(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onCreate");

            }

            @Override
            public void onStart(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onStart");
            }

            @Override
            public void onResume(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onResume");
                // 注：可通过这个回调方式获取布局的控件，如果设置了id，可通过findViewById方式获取，如果设置了tag，可以通过findViewWithTag，具体参考下面例子:

                // 通过id方式获取控件，并更改imageview图片
                ImageView imageView = (ImageView) view.findViewById(R.id.icon);
                imageView.setImageResource(R.mipmap.ic_launcher);

                // 通过tag方式获取控件，并更改布局内容
                TextView textView = (TextView) view.findViewWithTag("textview");
                textView.setText("my custom text");

                // 更多的操作：比如设置控件的点击事件
                imageView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Intent intent = new Intent(getApplicationContext(), OtherActivity.class);
                        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                        startActivity(intent);
                    }
                });
            }

            @Override
            public void onPause(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onPause");
            }

            @Override
            public void onStop(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onStop");
            }

            @Override
            public void onDestroy(Context context, View view, UpgradeInfo upgradeInfo) {
                Log.d(TAG, "onDestory");
            }

        };
```
> 如果想监听升级对话框的生命周期事件，可以通过设置OnUILifecycleListener接口 回调参数解释： 
> - context - 当前弹窗上下文对象 
> - view - 升级对话框的根布局视图，可通过这个对象查找指定view控件 
> - upgradeInfo - 升级信息


* 设置是否显示消息通知

```
Beta.enableNotification = true;
```
> 如果你不想在通知栏显示下载进度，你可以将这个接口设置为false，默认值为true。

* 设置Wifi下自动下载

```
Beta.autoDownloadOnWifi = false;
```
> 如果你想在Wifi网络下自动下载，可以将这个接口设置为true，默认值为false。

* 设置是否显示弹窗中的apk信息

```
Beta.canShowApkInfo = true;
```
> 如果你使用我们默认弹窗是会显示apk信息的，如果你不想显示可以将这个接口设置为false。

* 关闭热更新能力

```
Beta.enableHotfix = true;
```
> 升级SDK默认是开启热更新能力的，如果你不需要使用热更新，可以将这个接口设置为false。

* 自定义UI

SDK给我们提供两种自定义UI的方式：固定控件id方式和自定义activity两种方式；

1. 固定控件id

	> 使用固定控件id方式，弹窗的生命周期仍然由更新sdk管理，用户可以自定义升级弹窗的布局和网络弹窗布局，仅需要将自定义布局中的控件tag设置为指定tag，并在在初始化时设置自定义布局的资源id与回调。


2. 使用自定义activity的方式
> 使用自定义activity的方式，弹窗界面的绘制与生命周期均由用户自己维护，sdk负责在收到策略,下载时回调与事件上报，并提供相关接口控制任务下载，获取任务状态。


