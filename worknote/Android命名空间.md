
##  命名空间使用碰到的问题

我碰到的问题：

> 最近在做投资标的改版的时候有一个项目图片的地方，设计稿上的图片是圆形的，使用的FaceBook 的图片加载框架 fresco，搞成圆形只需要配置一个属性：
```
fresco:roundAsCircle="true"
```

> 然而，怎么都不管用。后来对比了下官方文档上的案例——发现原来在导入命名空间时导错了。

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

问题既然能来了，就要搞明白。如果说，只是解决当前这个问题，那么so easy ，只需要把
```
xmlns:fresco="http://schemas.android.com/apk/tools"
```

改成：
```
xmlns:fresco="http://schemas.android.com/apk/res-auto"
```

But

> 反思一下，**这可以归结为一类问题：Android命名空间的使用。** 对于这类问题真的能彻底搞明白了? 貌似只是经常在用，但从来没总结过这块儿的内容，借这个机会，总结下Android中命名空间相关的内容：

从以下几个问题入手：

0. 命名空间是什么？
1. Android中的命名空间有几种？
2. 都是怎么用的？

以下逐个讨论：

## 什么是命名空间？命名空间用来解决什么问题？

[W3School](http://www.w3school.com.cn/xml/xml_namespaces.asp)  如是说：

> XML 命名空间提供避免元素命名冲突的方法。

命名冲突如何来理解呢？

> 举个生活中的栗子： 
> 
> 我司技术部有两个小明，一个搞PHP的，一个搞Java的。
> 
> 如果你这样吼——“小明过来一下。” 两个小明可能都会一脸懵逼的样子。
> 
> 如果你这样吼——“搞PHP的小明过来一下。” 一下就清晰了好多。

回头来看这个问题：
> 当我们召唤小明的时候，如果不加以区分。两个小明就会引起冲突，而加上**搞PHP的**这几个词儿，问题立马就解决了。

而其中的 **搞PHP的** 这个修饰词，就相当于我们这里的命名空间。


## Android中命名空间


Android中的命名空间大致分为以下三类：

1. Android自带的：xmlns:android=”http://schemas.android.com/apk/res/android”
2. Android自带的：xmlns:tools=”http://schemas.android.com/tools”
3. 自定义控件的，比如Fresco的：xmlns:fresco="http://schemas.android.com/apk/res-auto"

他们的格式遵循如下规范：

xmlns:namespace-prefix="namespaceURI"

以 `xmlns:android=”http://schemas.android.com/apk/res/android” `为例：

* xmlns: xml namespace, 声明我们要开始定义一个命名空间。
* namespace-prefix: 代表赋予命名空间一个名字，如上文所说 ”搞PHP的“ 就相当于一个命名空间。
* namespaceURI : 是一个不可访问的地址，这是一个URI（统一资源标识符），是一个固定的常量。

在上面的布局代码中 只要以 `android：` 开头的属性便是引用了Android FrameWork为我们准备好的命名空间中的属性。它便是系统为我们准备的一个命名空间。

问题来了：

刚开始我导入错误的 "http://schemas.android.com/tools" 这又是什么鬼?这就引出了Android中的另一个命名空间tools。

> tools的命名空间，是帮助开发人员的工具，它的作用只在于开发阶段发挥作用，当APP被打包后，所有关于tools属性将会被摒弃掉。

以下内容来自官方文档——[Tools Attributes Reference
](https://developer.android.com/studio/write/tool-attributes.html)

tools 属性可以用来做以下三类事：

### 属性错误处理

以下属性可以帮助压制lint的警告信息。

`tools:ignore`

可以用这个属性告诉每个节点要忽略的信息。例如，你可以告诉tools忽略翻译错误。

```
<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>
```

`tools:targetApi`

这个属性和Java代码中的注解@TargetApi的作用是一样的。他告诉tools，你认为这个属性被用在指定的API或者更高版本的API上。

```
<GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:targetApi="14" >
```

`tools:locale`

告诉tools在<resources>元素中默认使用的语言或者国别信息。

```
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:locale="es">
```

### 设计view时用到的属性

如果你想在设计时开启这些属性，在运行时关闭。你可以使用`tools: `代替 `android:`.

对于Android FrameWork提供的任何view相关的属性，你可以使用tools替代Android 来在设计时进行预览。

`tools:text`

你可以使用tools:text 替代 android:text 在设计时加入简单的数据进行预览，这些信息在打包时将会被移除掉。

![](http://7xkl0t.com1.z0.glb.clouddn.com/18-3-21/74085301.jpg)

`tools:context`

在xml的跟布局中加入这个属性有两个作用:


1. 如果你的Activity配置了相应的主题，在用Layout Editor预览的时候就可以看到相应的主题；
2. 方便设置onClickListener.如果你在xml的根布局中引入了tools:context后，如下插图，借助快捷fix键，可以很方便的知道这个监听方法将会在哪个Activity中生成。

![](http://7xkl0t.com1.z0.glb.clouddn.com/18-3-21/26101573.jpg)

`tools:itemCount`

这个属性在RecycleView 中非常有用，如下代码：

```
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:itemCount="3"/>

```
使用tools:itemCount 指定item数，将会在Layout Editor预览器中显示出预览的item数。

`tools:layout`

这个属性将会告诉编辑器，在Fragment中绘制哪个布局来进行预览。

如下代码所示:

```
<fragment android:name="com.example.master.ItemListFragment"
    tools:layout="@layout/list_content" />
```

补充：
> 一般将Fragment以这种形式引入之后是无法进行预览的，将需要引入的Fragment的布局通过tools:layout引入到这里，将会很方便的进行预览。

`tools:listitem / tools:listheader / tools:listfooter`

如下代码所示：

```
<ListView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@android:id/list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:listitem="@layout/sample_list_item"
    tools:listheader="@layout/sample_list_header"
    tools:listfooter="@layout/sample_list_footer" />

```

将listview的item布局、header布局、footer布局以这种形式填写在布局文件中，将会很方便的进行预览，而不是显示的item1,item2......之类的信息。

`tools：showIn`

当你使用<include/>属性在另一个布局文件中引用当前属性时，可以使用showIn来预览当前布局在另一个布局中的样式。

需要被include的布局：

```
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:text="@string/hello_world"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:showIn="@layout/activity_main" />

```

主布局
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include layout="@layout/item_tv_test" />

</RelativeLayout>
```

这个时候在需要被引入的布局中使用tools:showIn就可以看到它被引入后在另一个布局中的样子。

`tools:menu`

这个属性声明在APP bar进行预览的时候展示哪些菜单。它的值可以是一个或多个菜单ID，使用逗号进行分隔。

如下代码所示：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:menu="menu1,menu2" />

```

`tools:minValue / tools:maxValue`

这个属性给NumberPicker 设置最大值和最小值：

```
<NumberPicker xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/numberPicker"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:minValue="0"
    tools:maxValue="10" />
```

`tools:openDrawer`

这个属性允许你在布局编辑器中打开一个DrawerLayout进行预览。你可以修改这个属性的值来控制以何种方式来打开这个布局：

![](http://7xkl0t.com1.z0.glb.clouddn.com/18-3-21/85471519.jpg)

如下代码所示：

```
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:openDrawer="start" />

```

### 资源压缩属性

官方文档链接：https://developer.android.com/studio/build/shrink-code.html

开启资源压缩属性的，需要在build.gradle中进行如下修改：
```
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}

```
主要是加入以下两行代码：
```
shrinkResources true
minifyEnabled true
```

`tools:shrinkMode`

这个属性有两种值：`safe`和`strict`

1. safe mode: 保留被显式引用的，或者可能通过Resources.getIdentifier()被动态引用的资源。
2. strict mode:保留 resources 或者 代码中 被显示引用的资源

开启 strict 模式之后, 可以使用 tools:keep 保留那些你不想被移除的资源, 或者使用tools:discard 直接移除资源。

`tools:keep`

使用资源压缩去移除未被使用的资源时，该属性将允许你指明哪些资源可以被保留（比如一些通过Resources.getIdentifier() 间接引用的资源）

如下代码所示：

```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/used_1,@layout/used_2,@layout/*_3" />

```
`tools:discard`

当使用资源压缩工具去除一些无用资源时，使用该属性可以指明一些需要手动删除的资源 (比如：被引用了但是未能生效的资源，或者 Gradle 插件误引用了某些资源被引用).

用法如下：
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:discard="@layout/unused_1" />

```

自定义的命名空间：

xmlns:fresco="http://schemas.android.com/apk/res-auto"

其中 fresco 是可以自己定义的任意值；
后面的url使用http://schemas.android.com/apk/res-auto


以上便是对这次碰到的问题——Android命名空间，使用一个总结。

链接：

1. NamespaceContext : https://developer.android.com/reference/javax/xml/namespace/NamespaceContext.html
2. Tools Attributes Reference : https://developer.android.com/studio/write/tool-attributes.html
3. shrink-code: https://developer.android.com/studio/build/shrink-code.html

