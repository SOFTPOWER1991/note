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

Manifest.permission: https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn

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

如果你的应用在其清单中列出正常权限（不会对用户隐私 或者 设备操作造成很大风险的权限），系统会自动授予这些权限。

如果你的应用在其清单中列出危险权限（会影响用户隐私、设备正常操作的权限），系统会要求用户明确授予这些权限。

更多详细的信息请参考：通用和危险权限，以及保护级别。

Android发出请求的方式取决于系统版本，而系统版本是应用的目标：

* 如果设备运行的是 Android 6.0（API 级别 23）或更高版本，并且应用的 targetSdkVersion 是 23 或更高版本，则应用在运行时向用户请求权限。用户可随时调用权限，因此应用在每次运行时均需检查自身是否具备所需的权限。 使用系统权限：https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn

> 第一次请求的时候，会弹出一个系统的dialog，包含：deny 、Allow按钮；
> 如果用户拒绝了权限请求，下次你的APP请求的时候会 包含一个checkbox，当勾选的时候，表示不想再接收这样的提示。
>  如果用户勾选了并且点击了Deny，当你再次尝试进行权限请求的时候，系统将不会进行任何提示。
> 即使用户当时给了你权限，但是这还是不可靠的。用户依旧可以在系统设置里，一个个的禁止掉。所以你依旧需要在运行时进行检查，防止SecurityException的发生。


### Android 5.1.1 and below，在安装时请求；

* 如果设备运行的是 Android 5.1（API 级别 22）或更低版本，并且应用的 targetSdkVersion 是 22 或更低版本，则系统会在用户安装应用时要求用户授予权限。如果将新权限添加到更新的应用版本，系统会在用户更新应用时要求授予该权限。用户一旦安装应用，他们撤销权限的唯一方式是卸载应用。只要用户安装了，那么所有的权限都给了。



通常，权限失效会导致 SecurityException 被扔回应用。但不能保证每个地方都是这样。例如，sendBroadcast(Intent) 方法在数据传递到每个接收者时会检查权限，在方法调用返回后，即使权限失效，您也不会收到异常。但在几乎所有情况下，权限失效会记入系统日志。

Android 系统提供的权限请参阅 Manifest.permission。此外，任何应用都可定义并实施自己的权限，因此这不是所有可能权限的详尽列表。

https://developer.android.com/reference/android/Manifest.permission.html?hl=zh-cn

可能在程序运行期间的多个位置实施特定权限：

* 在调用系统时，防止应用执行某些功能。
* 在启动 Activity 时，防止应用启动其他应用的 Activity。
* 在发送和接收广播时，控制谁可以接收您的广播，谁可以向您发送广播。
* 在访问和操作内容提供程序时。
* 绑定至服务或启动服务。

### 自动权限调整

随着时间的推移，平台中可能会加入新的限制，要想使用特定 API，您的应用可能必须请求之前不需要的权限。因为现有应用假设可随意获取这些 API 应用的访问权限，所以 Android 可能会将新的权限请求应用到应用清单，以免在新平台版本上中断应用。

Android 将根据为 targetSdkVersion 属性提供的值决定应用是否需要权限。如果该值低于在其中添加权限的版本，则 Android 会添加该权限。

例如，API 级别 4 中加入了 WRITE_EXTERNAL_STORAGE 权限，用以限制访问共享存储空间。如果您的 targetSdkVersion 为 3 或更低版本，则会向更新 Android 版本设备上的应用添加此权限。

权限保护级别：

权限被分为几个保护级别： normal , signature , dangerous permissions。


### 正常权限和 危险权限

系统权限分为几个保护级别。需要了解的两个最重要保护级别是正常权限和危险权限：

* 正常权限涵盖应用需要访问其沙盒外部数据或资源，但对用户隐私或其他应用操作风险很小的区域。例如，设置时区的权限就是正常权限。如果应用声明其需要正常权限，系统会自动向应用授予该权限。如需当前正常权限的完整列表，请参阅正常权限。https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#normal-dangerous
* 危险权限涵盖应用需要涉及用户隐私信息的数据或资源，或者可能对用户存储的数据或其他应用的操作产生影响的区域。例如，能够读取用户的联系人属于危险权限。如果应用声明其需要危险权限，则用户必须明确向应用授予该权限。

> 特殊权限
> 有许多权限其行为方式与正常权限及危险权限都不同。SYSTEM_ALERT_WINDOW 和 WRITE_SETTINGS 特别敏感，因此大多数应用不应该使用它们。如果某应用需要其中一种权限，必须在清单中声明该权限，并且发送请求用户授权的 intent。系统将向用户显示详细管理屏幕，以响应该 intent。

>  如需了解有关如何请求这些权限的详情，请参阅 SYSTEM_ALERT_WINDOW 和 WRITE_SETTINGS 参考条目。

### 权限组

所有危险的 Android 系统权限都属于权限组。如果设备运行的是 Android 6.0（API 级别 23），并且应用的 targetSdkVersion 是 23 或更高版本，则当用户请求危险权限时系统会发生以下行为：

* 如果应用请求其清单中列出的危险权限，而应用目前在权限组中没有任何权限，则系统会向用户显示一个对话框，描述应用要访问的权限组。对话框不描述该组内的具体权限。例如，如果应用请求 READ_CONTACTS 权限，系统对话框只说明该应用需要访问设备的联系信息。如果用户批准，系统将向应用授予其请求的权限。

* 如果应用请求其清单中列出的危险权限，而应用在同一权限组中已有另一项危险权限，则系统会立即授予该权限，而无需与用户进行任何交互。例如，如果某应用已经请求并且被授予了 READ_CONTACTS 权限，然后它又请求 WRITE_CONTACTS，系统将立即授予该权限。

任何权限都可属于一个权限组，包括正常权限和应用定义的权限。但权限组仅当权限危险时才影响用户体验。可以忽略正常权限的权限组。


Android 安全架构及权限控制机制剖析: https://www.ibm.com/developerworks/cn/opensource/os-cn-android-sec/index.html

如果设备运行的是 Android 5.1（API 级别 22）或更低版本，并且应用的 targetSdkVersion 是 22 或更低版本，则系统会在安装时要求用户授予权限。再次强调，系统只告诉用户应用需要的权限组，而不告知具体权限。

查看一个APP的权限：

$ adb shell pm list permissions -s 
$ adb shell install -g MyApp.apk （-g 授予所有权限）
### 自定义权限

要实施您自己的权限，必须先使用一个或多个 <permission> 元素在 AndroidManifest.xml 中声明它们。


**问题： 1. 为什么需要自定义权限？**
**问题： 2. 自定义权限用来解决什么问题？**
**问题： 3. 如何自定义权限？**
想要控制谁可以开始其中一个 Activity 的应用可如下所示声明此操作的权限：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp" >
    <permission android:name="com.example.myapp.permission.DEADLY_ACTIVITY"
        android:label="@string/permlab_deadlyActivity"
        android:description="@string/permdesc_deadlyActivity"
        android:permissionGroup="android.permission-group.COST_MONEY"
        android:protectionLevel="dangerous" />
    ...
</manifest>
```

> 注：系统不允许多个软件包使用同一名称声明权限，除非所有软件包都使用同一证书签署。如果软件包声明权限，则系统不允许用户安装具有相同权限名称的其他软件包，除非这些软件包使用与第一个软件包相同的证书签署。为避免命名冲突，建议对自定义权限使用相反域名样式命名，例如 com.example.myapp.ENGAGE_HYPERSPACE。

查看权限：

1. adb shell pm list permissions -s

#### 自定义权限建议 

应用可以定义自己的自定义权限，并通过定义 <uses-permission> 元素请求其他应用的自定义权限。不过，您应该仔细评估您的应用是否有必要这样做。

* 如果要设计一套向彼此显示功能的应用，请尽可能将应用设计为每个权限只定义一次。如果所有应用并非使用同一证书签署，则必须这样做。即使所有应用使用同一证书签署，最佳做法也是每个权限只定义一次。
* 如果功能仅适用于使用与提供应用相同的签名所签署的应用，您可能可以使用签名检查避免定义自定义权限。当一个应用向另一个应用发出请求时，第二个应用可在遵从该请求之前验证这两个应用是否使用同一证书签署。
* 如果您要开发一套只在您自己的设备上运行的应用，则应开发并安装管理该套件中所有应用权限的软件包。此软件包本身无需提供任何服务。它只是声明所有权限，然后套件中的其他应用通过 <uses-permission> 元素请求这些权限。


### 实施 AndroidManifest.xml 中的权限
### 发送广播时实施权限
### 其他权限实施
### URI 权限



=====

### 在运行时请求权限：

在所有版本的 Android 中，您的应用都需要在其应用清单中同时声明它需要的正常权限和危险权限，如声明权限中所述。不过，该声明的影响因系统版本和应用的目标 SDK 级别的不同而有所差异：

* 如果设备运行的是 Android 5.1 或更低版本，或者应用的目标 SDK 为 22 或更低：如果您在清单中列出了危险权限，则用户必须在安装应用时授予此权限；如果他们不授予此权限，系统根本不会安装应用。
* 如果设备运行的是 Android 6.0 或更高版本，或者应用的目标 SDK 为 23 或更高：应用必须在清单中列出权限，并且它必须在运行时请求其需要的每项危险权限。用户可以授予或拒绝每项权限，且即使用户拒绝权限请求，应用仍可以继续运行有限的功能。

#### 检查权限：

要检查您是否具有某项权限，请调用 ContextCompat.checkSelfPermission() 方法。例如，以下代码段显示了如何检查 Activity 是否具有在日历中进行写入的权限：

```
// Assume thisActivity is the current activity
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.permission.WRITE_CALENDAR);
```

如果应用具有此权限，方法将返回 PackageManager.PERMISSION_GRANTED，并且应用可以继续操作。如果应用不具有此权限，方法将返回 PERMISSION_DENIED，且应用必须明确向用户要求权限。

#### 请求权限

如果您的应用需要应用清单中列出的危险权限，那么，它必须要求用户授予该权限。Android 为您提供了多种权限请求方式。调用这些方法将显示一个标准的 Android 对话框，不过，您不能对它们进行自定义。

##### 解释应用为什么需要权限

在某些情况下，您可能需要帮助用户了解您的应用为什么需要某项权限。例如，如果用户启动一个摄影应用，用户对应用要求使用相机的权限可能不会感到吃惊，但用户可能无法理解为什么此应用想要访问用户的位置或联系人。在请求权限之前，不妨为用户提供一个解释。请记住，您不需要通过解释来说服用户；如果您提供太多解释，用户可能发现应用令人失望并将其移除。

您可以采用的一个方法是仅在用户已拒绝某项权限请求时提供解释。如果用户继续尝试使用需要某项权限的功能，但继续拒绝权限请求，则可能表明用户不理解应用为什么需要此权限才能提供相关功能。对于这种情况，比较好的做法是显示解释。

为了帮助查找用户可能需要解释的情形，Android 提供了一个实用程序方法，即 shouldShowRequestPermissionRationale()。如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 true。

> 如果用户在过去拒绝了权限请求，并在权限请求系统对话框中选择了 Don't ask again 选项，此方法将返回 false。如果设备规范禁止应用具有该权限，此方法也会返回 false。

### 请求您需要的权限

如果应用尚无所需的权限，则应用必须调用一个 requestPermissions() 方法，以请求适当的权限。应用将传递其所需的权限，以及您指定用于识别此权限请求的整型请求代码。此方法异步运行：它会立即返回，并且在用户响应对话框之后，系统会使用结果调用应用的回调方法，将应用传递的相同请求代码传递到 requestPermissions()。

```
// Here, thisActivity is the current activity
if (ContextCompat.checkSelfPermission(thisActivity,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {

    // Should we show an explanation?
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.READ_CONTACTS)) {

        // Show an expanation to the user *asynchronously* -- don't block
        // this thread waiting for the user's response! After the user
        // sees the explanation, try again to request the permission.

    } else {

        // No explanation needed, we can request the permission.

        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);

        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
    }
}

```

### 处理权限请求响应

```
@Override
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted, yay! Do the
                // contacts-related task you need to do.

            } else {

                // permission denied, boo! Disable the
                // functionality that depends on this permission.
            }
            return;
        }

        // other 'case' lines to check for other
        // permissions this app might request
    }
}
```


### 自定义权限