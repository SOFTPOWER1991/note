本文主要记录对易项优选APP进行Android 7.0 适配过程，其中包含如下内容:

1. 易项优选APP 在Android 7.0上碰到的问题
2. 适配7.0的步骤
		
	* 第一步，声明provider
	* 第二步，编写resource xml file
	* 第三步，使用FileProvider API
	
3. 这个特性将会影响到的使用场景以及解决方案
	* 自动安装文件
	* 调用系统拍照
	* 调用系统裁剪
4. 适配过程中遇到的问题

### 一、易项优选APP 在Android 7.0上碰到的问题

投资人在上传名片时，点击`拍照`，APP Crash，并抛出如下日志：

```
Process: com.capcom.app.android, PID: 656
android.os.FileUriExposedException: file:///storage/emulated/0/Android/data/com.capcom.app.android/files/capcom/IMG_20171026_172210.jpg exposed beyond app through ClipData.Item.getUri()
```

对于这个问题官网给出的解释：
> 对于面向 Android 7.0 的应用，Android 框架执行的 StrictMode API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常。

这个问题官网给出的解决方案：
> 要在应用间共享文件，您应发送一项 content:// URI，并授予 URI 临时访问权限。进行此授权的最简单方式是使用 FileProvider 类。

官网链接：https://developer.android.com/about/versions/nougat/android-7.0-changes.html#accessibility。

### 二、Android 7.0 适配步骤

如何使用FileProvider类来解决这个问题，官网也给了解决方案。传送门：[FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)


**第一步，声明provider**

在AndroidManifest.xml中的Application节点下声明provider

``` xml
 <provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.capcom.takephoto.fileprovider"
    android:grantUriPermissions="true"
    android:exported="false">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

其中需要注意的地方：

1. 配置authorities，android:authorities="com.capcom.takephoto.fileprovider" ，一般为applicationId+myfileprovider
2. exported:要求必须为false，为true则会报安全异常
3. grantUriPermissions: true，表示授予 URI 临时访问权限。

**第二步，编写resource xml file**

在res目录下新建xml文件夹，并命名为file_paths.xml(这个名字随便起，只要与第一步中meta-data节点下的android:resource的value保持一致即可）。

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path path="" name="camera_photos" />
</paths>
```

在paths节点内部支持以下几个子节点，分别为：

* &lt;root-path/&gt; 代表设备的根目录new File("/");
* &lt;files-path/&gt; 代表context.getFilesDir()
* &lt;cache-path/&gt; 代表context.getCacheDir()
* &lt;external-path/&gt; 代表Environment.getExternalStorageDirectory()
* &lt;external-files-path/&gt;  代表context.getExternalFilesDirs()
* &lt;external-cache-path/&gt;  代表getExternalCacheDirs()

在上面的xml中可以看到，每个节点包含两个属性：

* name
* path

path=""，这个得好好说说：
> 它代码根目录，意思是说你可以给别的应用共享根目录及其子目录下任何一个文件，如果你将path设为path="Download"，那么它代表着根目录下的Download目录(eg:/storage/emulated/0/Download)，如果你向其它应用分享Download目录范围之外的文件是不行的.


**第三步，使用FileProvider API**

接下来使用FileProvider类将file://URI转换为content://URI

代码如下：
``` java
/**
 * 打开相机
 *
 * @param file
 */
private void openCamera(File file) {
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    //兼容 Android 7.0
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        //通过FileProvider创建一个content类型的Uri
        Uri imageUri = FileProvider.getUriForFile(
                CertificateIdentityActivity.this,
                "com.capcom.takephoto.fileprovider",
                file);
        //添加这一句表示对目标应用临时授权该Uri所代表的文件
        intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        //将拍取的照片保存到指定URI
        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
    } else {
        intent.putExtra(
                MediaStore.EXTRA_OUTPUT,
                Uri.fromFile(new File(photoPathString)));
    }

    intent.putExtra(MediaStore.EXTRA_VIDEO_QUALITY, 1);
    startActivityForResult(intent, 1);
}
```

其中需要注意的地方：

1. 使用FileProvider创建content类型的uri时，其中的第二个参数：
	``` java
 		Uri imageUri = FileProvider.getUriForFile(
                CertificateIdentityActivity.this,
                "com.capcom.takephoto.fileprovider",
                file);
	```

	第二个参数就是我们配置的authorities，它将映射到确定的ContentProvider。
然后再看一眼我们生成的uri：

	`content://com.capcom.takephoto.fileprovider/camera_photos/Android/data/com.capcom.app.android/files/capcom/IMG_20171031_183008.jpg`可以看到格式为：content://authorities/定义的name属性/文件的相对路径，即name隐藏了可存储的文件夹路径。

2. 添加了intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);来对目标应用临时授权该Uri所代表的文件。


这样Android 7.0 的相机拍照就没问题了。

### 三、这个特性将会影响到的使用场景以及解决方案

**1. 自动安装文件**

APP更新时下载完新的apk后实现自动安装的功能，这是比较常见的场景。适配 Android 7.0 版本之前，我们代码可能是这样：

``` java
File apkFile = new File(getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS), "app_capcom.apk");

Intent installIntent = new Intent(Intent.ACTION_VIEW);
installIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
installIntent.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");
startActivity(installIntent);
```
适配 7.0 及以上版本的系统，必须使用 Content URI 代替 File URI了，代码如下：

在 res/xml 目录下新建一个 file_provider_paths.xml 文件（文件名自由定义），并添加子目录路径信息：
``` xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path name="my_download" path="Download"/>
</paths>

```
然后在 Manifest 文件中注册 FileProvider 对象，并链接上面的 path 路径文件：
``` xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.capcom.capcoms.myprovider"
    android:exported="false"
    android:grantUriPermissions="true">

    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_provider_paths"/>

</provider>
```
修改 java 代码，根据 File 对象生成 Content URI 对象，并授权访问：
``` java
File apkFile = new File(getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS), "app_capcom.apk");
Uri apkUri = FileProvider.getUriForFile(this,
        BuildConfig.APPLICATION_ID+".myprovider", apkFile);
Intent installIntent = new Intent(Intent.ACTION_VIEW);
installIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
installIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
installIntent.setDataAndType(apkUri, "application/vnd.android.package-archive");
startActivity(installIntent);
```
如此，便完成了应用中调用系统功能安装apk文件的适配工作。

**2. 调用系统拍照**

> 调用系统拍照，这里就不再赘述了，上文已经详细说明了。

**3. 调用系统裁剪**

在Android7.0之前，你可以通过如下方法来裁切照片

``` java
File file=new File(Environment.getExternalStorageDirectory(), "/temp/"+System.currentTimeMillis() + ".jpg");
if (!file.getParentFile().exists())file.getParentFile().mkdirs();
Uri outputUri = Uri.fromFile(file);
Uri imageUri=Uri.fromFile(new File("/storage/emulated/0/temp/1474960080319.jpg"));
Intent intent = new Intent("com.android.camera.action.CROP");
intent.setDataAndType(imageUri, "image/*");
intent.putExtra("crop", "true");
intent.putExtra("aspectX", 1);
intent.putExtra("aspectY", 1);
intent.putExtra("scale", true);
intent.putExtra(MediaStore.EXTRA_OUTPUT, outputUri);
intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
intent.putExtra("noFaceDetection", true); // no face detection
startActivityForResult(intent,1008);
```
和拍照一样，上述代码在Android7.0上同样会引起android.os.FileUriExposedException异常，解决办法就是上文说说的使用FileProvider。

``` java
File file=new File(Environment.getExternalStorageDirectory(), "/temp/"+System.currentTimeMillis() + ".jpg");
if (!file.getParentFile().exists())file.getParentFile().mkdirs();
Uri outputUri = FileProvider.getUriForFile(context, "com.capcom.takephoto.fileprovider",file);
Uri imageUri=FileProvider.getUriForFile(context, "com.capcom.takephoto.fileprovider", new File("/storage/emulated/0/temp/1474960080319.jpg");//通过FileProvider创建一个content类型的Uri
Intent intent = new Intent("com.android.camera.action.CROP");
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
intent.setDataAndType(imageUri, "image/*");
intent.putExtra("crop", "true");
intent.putExtra("aspectX", 1);
intent.putExtra("aspectY", 1);
intent.putExtra("scale", true);
intent.putExtra(MediaStore.EXTRA_OUTPUT, outputUri);
intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
intent.putExtra("noFaceDetection", true); // no face detection
startActivityForResult(intent,1008);
```
### 四、适配过程中碰到的问题

易项优化APP的下载、安装都是交给了Bugly去做的，因此需要对Bugly的7.0进行兼容性适配。

按照上面的步骤step by step，做完跑起来后运行APP直接crash了，赶紧追踪日志。log如下：
```
java.lang.NullPointerException: Attempt to invoke virtual method 'android.content.res.XmlResourceParser android.content.pm.PackageItemInfo.loadXmlMetaData(android.content.pm.PackageManager, java.lang.String)' on a null object reference
```

查看代码的各个地方都是没问题的，最后注意到了Bugly文档上的一句话：
> 如果你使用的第三方库也配置了同样的FileProvider, 可以通过继承FileProvider类来解决合并冲突的问题。

感觉可能是因为我的两个FileProvider用了相同的名字引起的。于是继承FileProvider给两个不同的FileProvider设置不同的name，问题就解决了。代码如下：
```java
/**
 * 通过继承FileProvider类来解决合并冲突的问题
 *
 * Created by zg on 2017/10/30.
 */

public class BuglyFileProvider extends FileProvider {
    public BuglyFileProvider() {
    }
}
```
修改后的Provider配置为：
```xml
<provider
    android:name=".utils.BuglyFileProvider"
    android:authorities="com.capcom.app.android.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true"
    tools:replace="name,authorities,exported,grantUriPermissions">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"
        tools:replace="name,resource"/>
</provider>
```


以上便是对Android 7.0已知问题的适配过程。

