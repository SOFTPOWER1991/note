0. 命名空间使用错误引起的血案；
1. 命名空间；
2. 作用，用来解决什么问题；
3. Android中的命名空间问题；

最近在做投资标的改版的时候有一个项目图片的地方，设计稿上的图片是圆形的，使用的FaceBook 的图片加载框架 fresco，搞成圆形只需要配置一个属性：
```
fresco:roundAsCircle="true"
```

然而，怎么都不管用。后来对比了下官方文档上的案例，原来在导入命名空间时倒错了：
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:fresco="http://schemas.android.com/apk/tools"
    android:id="@+id/rl_tem_info"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <com.facebook.drawee.view.SimpleDraweeView
        android:id="@+id/img_user_header"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_marginLeft="12dp"
        android:layout_marginStart="12dp"
        fresco:roundAsCircle="true"
        fresco:failureImage="@mipmap/failure_image"
        fresco:placeholderImage="@mipmap/failure_image"
        fresco:placeholderImageScaleType="centerCrop"
       />
       ......

</RelativeLayout>
```

排查了好久，最终发现原来是命名空间的问题。

问题既然能来了，就要搞明白。如果说，只是解决当前这个问题，那么so easy ，只需要把
```
xmlns:fresco="http://schemas.android.com/apk/tools"
```

改成：
```
xmlns:fresco="http://schemas.android.com/apk/res-auto"
```

便可以解决这个问题。

然而，反思一下对于这类问题真的能彻底搞明白了，貌似只是经常在用，但貌似从来没总结过这块儿的内容，借这个机会，总结下Android中命名空间相关的内容：
从以下几个问题入手：
1. Android中的命名空间有几种
2. 都是怎么用的
3. 有什么区别？

先来想个问题:

什么是命名空间？命名空间用来解决什么问题？

[W3School](http://www.w3school.com.cn/xml/xml_namespaces.asp) 如是说：

> XML 命名空间提供避免元素命名冲突的方法。

命名冲突如何来理解呢？
> 举个生活中的例子： 学校里有2个班级，每个班级里都有一个叫小明的同学。两个班级在一起开会，校长说，小明同学请站出来，这时候，1班 和 2班的小明都一脸懵逼的样子，谁该站出去，后来一个老师告诉了校长，有两个小明。
> 
> 这时候，校长说：“1班的小明 请站出来？”

这个场景中 1班 就相当于是命名空间。

在考虑下这种场景：

> 两个班级中都有3个小明，1班一个，2班有两个，一个男的，一个女的。
> 
> 如果这时候想要找2班的女的小明，就要这样称呼：
> **2班的女 小明 请站出来**

这种场景中2班的女 就相当于是命名空间。

Android中常见的命名空间：

1. Android自带的：xmlns:android=”http://schemas.android.com/apk/res/android”
2. Android自带的：xmlns:tools=”http://schemas.android.com/tools”
3. 自定义控件的，比如Fresco的：xmlns:fresco="http://schemas.android.com/apk/res-auto"

可以看到他们的格式遵循如下规范：

xmlns:namespace-prefix="namespaceURI"

以xmlns:android=”http://schemas.android.com/apk/res/android”为例：

* xmlns: xml namespace,声明我们要开始定义一个命名空间。
* namespace-prefix: 代表赋予命名空间一个名字，如上文所说”2班的小明“ 中的”2班“就相当于一个命名空间。
* namespaceURI : 是一个不可访问的地址，这是一个URI（统一资源标识符），是一个固定的常量。

在上面的布局代码中 只要以Android：开头的属性便是引用了命名空间中的属性，上面已经提到过，Android便是赋予命名空间一个名字，跟我们平时定义变量一样，


刚开始我导入错误的namespaceURI "http://schemas.android.com/tools" 这又是什么鬼，这就引出了Android中的另一个命名空间tools。

tools的命名空间，是帮助开发人员的工具，它的作用只在于开发阶段发挥作用，当APP被打包后，所有关于tools属性将会被摒弃掉。



使用技巧：

1. tools:text="Mastering ToolsNs" 代替硬编码，在开发阶段进行预览，线上版本将会被摸出掉。 
2. android: tools 的使用阶段，Android 在运行时看到；tools在开发、设计阶段用到；
3. 在xml中指定目标版本；
4. 告知lint你的字符是正确的
5. 在fragment和自定义视图上预览布局；

```
<fragment
	android:name="com.alxxxxx.BooksFragment"
	android:layout_width="match_parent"
	......
	tools:layout="@layout/fragment_boos"
/>
```

链接：

1. NamespaceContext : https://developer.android.com/reference/javax/xml/namespace/NamespaceContext.html
2. 精通 Android 中的 tools 命名空间: https://www.jianshu.com/p/a39dddb46bd8
3. Tools Attributes Reference : https://developer.android.com/studio/write/tool-attributes.html
4. Android中的命名空间 : http://blog.qiji.tech/archives/3744

