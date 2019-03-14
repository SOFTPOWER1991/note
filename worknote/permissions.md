Links: https://developer.android.com/guide/topics/security/permissions?hl=zh-cn

Android 是一个权限分隔的操作系统，其中每个应用都有其独特的系统标识（Linux 用户ID 和 组ID）。系统各部分也分隔为不同的标识。Linux据此将不同的应用以及应用与系统分隔开来。


其他更详细的安全功能通过“权限”机制提供。此机制会限定特定进程可以执行的具体操作。并且根据URI权限授权临时访问特定的数据段。

### 安全架构

Android安全架构的中心设计点是：在默认情况下任何应用都没有权限执行对其它应用、操作系统或用户有不利影响的任何操作。这包括**读取或写入用户的私有数据（例如联系人或电子邮件）、读取或写入其他应用程序的文件、执行网络访问、使设备保持唤醒状态等。**

由于每个Android应用都是在进程沙盒中运行，因此应用必须显示共享资源和数据。它们的方法是声明需要哪些权限来获取基本沙盒未提供的额外功能。应用以静态方式声明它们需要的权限，然后Android系统提示用户同意。

应用沙盒不依赖用于开发应用的技术。特别是，Dalvik VM 不是安全边界，任何应用都可运行原生代码。各类应用 — Java、原生和混合 — 以同样的方式放在沙盒中，彼此采用相同程度的安全防护。

### 用户ID 和 文件访问

在安装时，Android 为每个软件包提供唯一的 Linux 用户 ID。此 ID 在软件包在该设备上的使用寿命期间保持不变。在不同设备上，相同软件包可能有不同的 UID；重要的是每个软件包在指定设备上的 UID 是唯一的。

由于在进程级实施安全性，因此任何两个软件包的代码通常都不能在同一进程中运行，因为它们需要作为不同的 Linux 用户运行。您可以在每个软件包的 AndroidManifest.xml 的 manifest 标记中使用 sharedUserId 属性，为它们分配相同的用户 ID。这样做以后，出于安全目的，两个软件包将被视为同一个应用，具有相同的用户 ID 和文件权限。请注意，为保持安全性，只有两个签署了相同签名（并且请求相同的 sharedUserId）的应用才被分配同一用户 ID。

应用存储的任何数据都会被分配该应用的用户 ID，并且其他软件包通常无法访问这些数据。使用 getSharedPreferences(String, int)、openFileOutput(String, int) 或 openOrCreateDatabase(String, int, SQLiteDatabase.CursorFactory) 创建新文件时，可以使用 MODE_WORLD_READABLE 和/或 MODE_WORLD_WRITEABLE 标记允许任何其他软件包读取/写入文件。设置这些标记时，文件仍归您的应用所有，但其全局读取和/或写入权限已适当设置，使任何其他应用都可看见它。

### 使用权限

Manifest.permission:https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn

基本 Android 应用默认情况下未关联权限，这意味着它无法执行对用户体验或设备上任何数据产生不利影响的任何操作。要利用受保护的设备功能，必须在应用清单中包含一个或多个 <uses-permission> 标记。

例如，需要监控传入的短信的应用要指定：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.app.myapp" >
    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    ...
</manifest>
```

权限级别：https://developer.android.com/guide/topics/security/permissions?hl=zh-cn#normal-dangerous


