
# 简介

Android 6.0引入了新的权限模式——用户直接在运行时管理应用权限。此版本引入了一种新的权限模式，如今，用户可直接在运行时管理应用权限。这种模式让用户能够更好地了解和控制权限，同时为应用开发者精简了安装和自动更新过程。用户可为所安装的各个应用分别授予或撤销权限。

> 1. 要确定您的应用是否已被授予权限，请调用新增的 checkSelfPermission() 方法。
> 2. 要请求权限，请调用新增的 requestPermissions() 方法。

请注意：即使您的应用并不以 Android 6.0（API 级别 23）为目标平台，您也应该在新权限模式下测试您的应用。

这种请求方式很是繁琐，在googlesamples 提供了一种更简单的方式 [easypermissions
](https://github.com/googlesamples/easypermissions)，而且我的项目一直在使用这种方式！我想总结下这种使用方式，并且读一读源码，了解下这个easypermission的运行原理。


# 实例

以Android中打电话为例来说：

```
Intent phoneIntent = new Intent(
                "android.intent.action.CALL", Uri.parse("tel:"
                + "15525723303"));
        startActivity(phoneIntent);
```

* 在Android6.0以前，我们只需要把需要的权限都配置的清单文件里即可。
* 在Android6.0的时候及以后，如果直接运行程序就会Crash。这就要在运行时动态申请，用到哪个权限申请哪个。

我们按照如下流程图来走：

![permission](https://github.com/SOFTPOWER1991/note/blob/master/raw/permisson.png)

## 1. 在build.gradle文件中引入

```
compile 'pub.devrel:easypermissions:0.3.0'
```

## 2. implements EasyPermissions.PermissionCallbacks 

## 3. 检查是否有权限

```
//1. 检查是否有权限
if (EasyPermissions.hasPermissions(this, Manifest.permission.CALL_PHONE)) {
    // 有权限，直接打电话
    Toast.makeText(this, "TODO: Camera things", Toast.LENGTH_LONG).show();
    call_action();
} else {
    //没权限，申请权限
    EasyPermissions.requestPermissions(this, "请开启打电话权限",
            RC_CALL_PERM, Manifest.permission.CAMERA);
}
```

## 4. 权限申请结果回调

```
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        // 请求权限的回调
        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }
```

## 5. 是否成功

覆写：

```
    /**
     * 权限申请成功
     * @param requestCode
     * @param perms
     */
    @Override
    public void onPermissionsGranted(int requestCode, List<String> perms) {
        call_action();
    }

    /**
     * 权限被禁止
     * @param requestCode
     * @param perms
     */
    @Override
    public void onPermissionsDenied(int requestCode, List<String> perms) {
        if (EasyPermissions.somePermissionPermanentlyDenied(this, perms)) {
            new AppSettingsDialog.Builder(this).build().show();
        }
    }
```

这样很简单就完成了一次权限申请！

这只是对单一的权限！那么如果是很多个权限该怎么办呢？

# 多个权限的申请

只需要在申请的时候多加个权限即可：

```
@AfterPermissionGranted(RC_LOCATION_CONTACTS_PERM)
    public void locationAndContactsTask() {
        String[] perms = { Manifest.permission.ACCESS_FINE_LOCATION, Manifest.permission.READ_CONTACTS };
        if (EasyPermissions.hasPermissions(this, perms)) {
            // Have permissions, do the thing!
            Toast.makeText(this, "TODO: Location and Contacts things", Toast.LENGTH_LONG).show();
        } else {
            // Ask for both permissions
            EasyPermissions.requestPermissions(this, getString(R.string.rationale_location_contacts),
                    RC_LOCATION_CONTACTS_PERM, perms);
        }
    }
```

这是因为,hasPermissions的参数允许传递一个变长参数进入！哈哈，这下搞明白了！

```
    /**
     * Check if the calling context has a set of permissions.
     *
     * @param context the calling context.
     * @param perms   one ore more permissions, such as {@link Manifest.permission#CAMERA}.
     * @return true if all permissions are already granted, false if at least one permission is not
     * yet granted.
     * @see Manifest.permission
     */
    public static boolean hasPermissions(@NonNull Context context, @NonNull String... perms) {
        // Always return true for SDK < M, let the system deal with the permissions
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            Log.w(TAG, "hasPermissions: API version < M, returning true by default");

            // DANGER ZONE!!! Changing this will break the library.
            return true;
        }

        for (String perm : perms) {
            boolean hasPerm = (ContextCompat.checkSelfPermission(context, perm) ==
                    PackageManager.PERMISSION_GRANTED);
            if (!hasPerm) {
                return false;
            }
        }

        return true;
    }
```

以上便是对6.0动态运行时权限的一个使用介绍！

然而，不能就此为止，接下来要读一读源码，研究一下这个库的原理！

