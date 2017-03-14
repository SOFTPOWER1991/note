# 1. 功能介绍

EasyPermissions是简化 Android M (API >= 23)权限申请的一个库。

# 2. 流程图

以在App中调用打电话权限为例，流程图如下：
	
![runtime_permissions](https://github.com/SOFTPOWER1991/note/blob/master/raw/permisson.png)

# 3. 具体实现

涉及到的核心类：

> `EasyPermissions`: 为Android M (API >= 23)的App请求和检查系统权限

按照上面的流程图来走：

* 检查权限，涉及到的方法`hasPermissions`：

```
    /**
     * 检查传入的上下文是否有这一系列的权限
     *
     * @param context the calling context. 调用该方法的context
     * @param perms   一个或多个权限，比如：Manifest.permission.CAMERA
     * @return  如果所有的权限都被授予了，返回true;如果至少有一个权限没被授予，那么返回false;
     * @see Manifest.permission
     */
    public static boolean hasPermissions(@NonNull Context context, @NonNull String... perms) {
        //如果SDK版本小于M，总是返回true,让系统自己去决定需要的权限。
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            Log.w(TAG, "hasPermissions: API version < M, returning true by default");

            // DANGER ZONE!!! Changing this will break the library.
            return true;
        }

        for (String perm : perms) {
        		//ContextComat检查权限是否被授予
            boolean hasPerm = (ContextCompat.checkSelfPermission(context, perm) ==
                    PackageManager.PERMISSION_GRANTED);
            if (!hasPerm) {
                return false;
            }
        }

        return true;
    }
```

我们可以看到在里面调用了`ContextCompat.checkSelfPermission(context, perm) ` 来检查权限，如果等于`PackageManager.PERMISSION_GRANTED`，那么就返回true。

`PackageManager.PERMISSION_GRANTED`：如果给定的报名这个权限已经被授予成功了，那么就返回0；否则就是-1，即：`PERMISSION_DENIED`。


* 没有权限的情况下，请求权限，涉及到的代码 `requestPermissions`：

这个类中一共有6个`requestPermissions`的重载方法，我们只需要分析一个即可

```
	/**
     * 请求一系列的权限，并给出需要这个权限的理由。      *
     * @see #requestPermissions(Activity, String, int, int, int, String...)
     */
    public static void requestPermissions(@NonNull Activity activity,
                                          @NonNull String rationale,
                                          int requestCode,
                                          @NonNull String... perms) {
        requestPermissions(
                activity,
                rationale,
                android.R.string.ok,
                android.R.string.cancel,
                requestCode,
                perms);
    }
```

接着走到了`requestPermissions`的一个重载方法内：

```
/**
     *请求一系列的权限，并给出需要这个权限的理由。
     * @param activity      请求权限。应该实现ActivityCompat.OnRequestPermissionsResultCallback接口或者如果是继承自FragmentActivity那么就覆写FragmentActivity#onRequestPermissionsResult(int, String[], int[])方法。
     * @param rationale     解释为什么这个程序需要这个权限，如果用户在第一次拒绝这个请求的时候将会展示出来。
     * @param positiveButton 确定按钮的提示文字
     * @param negativeButton 取消按钮的提示文字
     * @param requestCode    请求码用来追踪这个请求，必须小于 256
     * @param perms          一系列需要被请求的权限
     * @see Manifest.permission
     */
    @SuppressLint("NewApi")
    public static void requestPermissions(@NonNull Activity activity,
                                          @NonNull String rationale,
                                          @StringRes int positiveButton,
                                          @StringRes int negativeButton,
                                          int requestCode,
                                          @NonNull String... perms) {
        //检查是否有权限，有提示已经有权限，返回；没有，往下进行；                                 
        if (hasPermissions(activity, perms)) {
            notifyAlreadyHasPermissions(activity, requestCode, perms);
            return;
        }
			
			//是否要展示一个对话框，来提示用户：申请这个权限的原因
        if (shouldShowRationale(activity, perms)) {
        		//展示申请权限的原因
            showRationaleDialogFragment(
                    activity.getFragmentManager(),
                    rationale,
                    positiveButton,
                    negativeButton,
                    requestCode,
                    perms);
        } else {
        		 //请求权限，请求方式分为：API >=23 和 低于23两种请求方式
            ActivityCompat.requestPermissions(activity, perms, requestCode);
        }
    }

```

上面最后的方法走到了真正的权限请求方法：

```
 ActivityCompat.requestPermissions(activity, perms, requestCode);
```

方法的具体实现如下：

```
 public static void requestPermissions(final @NonNull Activity activity,
            final @NonNull String[] permissions, final @IntRange(from = 0) int requestCode) {
        if (Build.VERSION.SDK_INT >= 23) {
            ActivityCompatApi23.requestPermissions(activity, permissions, requestCode);
        } else if (activity instanceof OnRequestPermissionsResultCallback) {
            Handler handler = new Handler(Looper.getMainLooper());
            handler.post(new Runnable() {
                @Override
                public void run() {
                    final int[] grantResults = new int[permissions.length];

                    PackageManager packageManager = activity.getPackageManager();
                    String packageName = activity.getPackageName();

                    final int permissionCount = permissions.length;
                    for (int i = 0; i < permissionCount; i++) {
                        grantResults[i] = packageManager.checkPermission(
                                permissions[i], packageName);
                    }
						// 具体的回调，我们在Activity中拿到的关于权限申请的结果就是从这儿返回的
                    ((OnRequestPermissionsResultCallback) activity).onRequestPermissionsResult(
                            requestCode, permissions, grantResults);
                }
            });
        }
    }
```


至此，完成了权限请求，接下来就要对权限的请求结果进行操作了，这时就要在我们的Activity中实现`onRequestPermissionsResult`这个方法，具体代码如下：

```
 @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        // 请求权限的回调
        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }
```

拿到请求结果后，我们就要进行具体的处理了，这就走到了`EasyPermissions.onRequestPermissionsResult`中，在这个方法中，对请求的权限结果进行了具体的分发，代码如下：

```
 /**
     * 处理权限请求的结果。必须在Activity中的onRequestPermissionsResult中进行调用。
     *
     * 如果有权限被授予或者禁止，调用者就要回调PermissionCallbacks定义的具体的方法如果合适的话会调用被AfterPermissionGranted注解的方法
     *
     * @param requestCode  权限请求码
     * @param permissions  要请求的一系列权限
     * @param grantResults 权限授予结果
     * @param receivers    具体的权限处理者，Activity或者fragment
     */
    public static void onRequestPermissionsResult(int requestCode,
                                                  @NonNull String[] permissions,
                                                  @NonNull int[] grantResults,
                                                  @NonNull Object... receivers) {
        // 从返回的结果中收集被禁止和授予的权限
        List<String> granted = new ArrayList<>();
        List<String> denied = new ArrayList<>();
        for (int i = 0; i < permissions.length; i++) {
            String perm = permissions[i];
            if (grantResults[i] == PackageManager.PERMISSION_GRANTED) {
                granted.add(perm);
            } else {
                denied.add(perm);
            }
        }

        // 迭代所有的接受者：Activity或者Fragment
        for (Object object : receivers) {
            // 如果有权限被授予，调用这个方法
            if (!granted.isEmpty()) {
                if (object instanceof PermissionCallbacks) {
                    ((PermissionCallbacks) object).onPermissionsGranted(requestCode, granted);
                }
            }

            // 如果有权限被禁止，调用这个方法
            if (!denied.isEmpty()) {
                if (object instanceof PermissionCallbacks) {
                    ((PermissionCallbacks) object).onPermissionsDenied(requestCode, denied);
                }
            }

            // 如果全部被授予的话，调用这个注解方法
            if (!granted.isEmpty() && denied.isEmpty()) {
                runAnnotatedMethods(object, requestCode);
            }
        }
    }
```

至此就很清晰了！

我们只要在自己的页面中调用具体的回调方法即可：

```
/**
     * 权限申请成功
     *
     * @param requestCode
     * @param perms
     */
    @Override
    public void onPermissionsGranted(int requestCode, List<String> perms) {
    	  //具体的调用动作
        call_action();
    }

    /**
     * 权限被禁止
     *
     * @param requestCode
     * @param perms
     */
    @Override
    public void onPermissionsDenied(int requestCode, List<String> perms) {
    		//如果权限被永久禁止，那么就让用户去设置里打开权限。
        if (EasyPermissions.somePermissionPermanentlyDenied(this, perms)) {
            new AppSettingsDialog.Builder(this).setRationale("请开启相应权限").build().show();

        }
    }
```

另外还需要注意的两个地方，我们需要弹出两个兑换框来：

> 1. 除此申请时，需要告诉用户为什么需要这个权限；
> 2. 当用户把这个权限禁止后，如果还要在用这个权限，就弹出对话框让用户去设置里开启相应权限。

至此就大体完成了权限申请库`EasyPermission`的源码解析。

另外，我有个想法，我可以自己写一个这样的库，因为弹出的对话框的样式，还有提示的文字需要自己定义！

这完全可行，因为我明白了这个库的原理！

