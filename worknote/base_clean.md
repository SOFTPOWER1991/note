
* base清理
   > 1. adapter之类的替换为MultiType（这个可能要替换所有ListView实现的地方）
   移动至 需要使用的地方
   > 2. 下拉刷新、上拉加载之类删掉旧的资源，替换为现在在使用的SmartRefreshLayout（这个需要找到项目中的）
   > 3. indicator 的整理删除；
   
**时间：1 和 2  移动至 需要使用的地方，3移动至需要使用的地方  半天**
   
* extension 扩充  (我来做)
  > 1. Context 相关扩充：
        >  Toast、startActivity、获取PackageInfo（CommonUtils）、获取屏幕宽、高
  
  > 2. 将StringUtils 这些工具类，统一至StringExtengManager.kt中；
          android:background="@drawable/top_yellow">

  > 工时：2个小时
  
* 图片加载的整理: 图片加载全部改为Glide
  > 1. 移除项目中的Fresco 相关的东西
  > 2. 移除picasso 相关的东西；
  > 3. 图片加载工具类的统一；
  
  工作量在于：找到并替换原有的地方 1.5天；


* 工具类的统一

  > 1. 启动Activity的类有3个：UIHelper类 有两个（）、UILinkHelper、UILinkUtils。
  > 2. Log工具类统一: LogUtils 和 Logger库 、EtherLogHelper的统一、'com.squareup.okhttp3:logging-interceptor:3.10.0'
  > 3. 状态栏、导航栏管理类的统一；
  > 4. 文件管理类的统一；
  > 5. 网络状态管理：NetUtils 和 CommonUtils
  > 6. CommonUtils的拆解：里面包含了各种不同功能的方法，拆解，分离。
  > 7. Toast相关的整理：ToastUitls。
 
 
 启动Activity类：统一使用UIHelper；
 
 Log工具类统一使用：LogUtils；
 
 文件管理类统一使用:FileUtils；
 
 网络状态管理统一使用: NetUtils；
 
 Toast 统一使用ToastUtils；
 
 图片加载统一使用EtherImageLoader;
 
 dp2px, getScreenHeigth，统一使用EtherDisplayHelper;
 
 ViewAnimationUtils
 > 工时： 2天
  
判断app版本信息，APP是否存活 ，使用ContextExtendManagerKt

* Http请求的整理
> 1. 请求出错的时候，能提示出具体的错误，打印出错误码。
> 2. 出错时，不要全部弹出个“网络错误，请稍后重试”，能打印出具体的出错信息(Exception，能打印出Exception信息)
> 3. 能控制是否弹出toast

> 翠翠在搞===== 3天

* 权限的整理
    > easypermission 
    > BaseActivity
    > PermissionUitls

> 工作量在于： 1. 把现有的权限方案整合；2. 并把原来的使用地方替换掉；
> 半天 

* Dialog整理(找到的Dialog、)
   
    > 替换掉零零散散到处new的dialog,统一改为之前封装的dialog;
    > 目前发现的地儿：
    > 1. CommonUtils
    > 2. base/ui/dialog
    > 3. DialogUitls
    > 4. IosDialog
    > 5. 引用的第三方库： com.afollestad.material-dialogs:core:0.9.1.0
    > 6. Activity中有的地方也在创建Dialog

> 工作量： 
> 1. 替换各种地方使用的dialog,纯体力活。
> 2.5 天时间

问题： 
项目中有很多material-dialogs的地方，旧版本和新版本差距较大。


====


* model 的整理：
    > 将model移动至具体的业务处，不要放在base中；
    
    2天
    
* 目录整理:
    > 
    
    
    ====
    
    1. 设备兼容性：https://developer.android.com/guide/practices/compatibility?hl=zh-cn
    2. 
