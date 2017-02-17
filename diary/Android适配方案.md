Android适配包括：

* 屏幕适配：多分辨率适配
* 不同版本的适配Android 5.0 ， 6.0 之间的版本兼容性适配

屏幕适配和版本兼容适配在不同的业务场景，这些适配方法都会用到！没有说是，我们只用一种适配方案！

# 屏幕适配方案（分辨率适配）


* 资源适配

为不同的分辨率提供不同大小的图片资源;
.9图片的使用，保证背景不被拉伸

* 布局适配：百分比适配，只需要支持控件能够参考屏幕的百分比去计算宽高。

	根据设计采用的设计标准比如720*1080来做的设计图。会将宽、高进行平分
	
	a. 宽度为720份，将任何分辨率的宽度分为720份，取值为x1-x720
	b. 高度为1080份，将任何分辨率的高度分为1080份，取值为y1-y1080
	
* 为不同屏幕提供不同的布局：比如App要兼容手机和平板的时候，就需要提供两套不同的布局
* 提供多套dimens文件夹，保证在不同分辨率屏幕上使用的是相对应平台的大小

	



# Android不同版本兼容性适配

* 指定最小和目标API级别 

为App指定支持的最小版本和最大兼容的android版本，比如最小支持Android 4.0 ，最高兼容Android 7.0

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >
    <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
    ...
</manifest>

```

* 运行时检查系统版本

Android在Build常量类中提供了对每一个版本的唯一代号，在我们的app中使用这些代号可以建立条件，保证依赖于高级别的API的代码，只会在这些API在当前系统中可用时，才会执行。

```
private void setUpActionBar() {
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        ActionBar actionBar = getActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
    }
}
```

* 不同版本的权限适配

android6.0 的动态运行时权限检查；










