https://antonioleiva.com/kotlin-android-extensions/

ktx 是kotlin的另一个插件包含在kotlin的常规插件中，他允许你从Activity、Fragments 和 Views中以一种非常惊奇的方式查找View。

它会生成一些额外的代码来让你从布局xml中查找View，使用你在布局中定义的View ID。

它会生成一个本地的View cache。当你第一次使用的时候，它还是像往常一样使用findviewbyid。但是下一次，将会从View cache中来查找这个View，所以这样获取非常快。


如何使用：

第一步，在android  module 中引入以下代码：



apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'


第二步，在布局文件中写好View 的ID

第三步，在Activity中使用

导入： 	
import i kotlinx.android.synthetic.main.activity_main.*


KAE 背后的原理：

使用tools -> Kotlin -> show kotlin bytecode _> 反编译。

1. 先是看到kotlin的bytecode ，
2. 将kotlin的bytecode  Decomplie to Java 代码。

将会看到kotlin代码被反编译后的源码。

将会把kotlin源码Decomplie 为Java源码：


来用Java帮助你理解这个插件在做什么事儿。

看这部分代码：

```
    private HashMap _$_findViewCache;
    ...
    public View _$_findCachedViewById(int var1) {
       if(this._$_findViewCache == null) {
          this._$_findViewCache = new HashMap();
       }
     
       View var2 = (View)this._$_findViewCache.get(Integer.valueOf(var1));
       if(var2 == null) {
          var2 = this.findViewById(var1);
          this._$_findViewCache.put(Integer.valueOf(var1), var2);
       }
     
       return var2;
    }
     
    public void _$_clearFindViewByIdCache() {
       if(this._$_findViewCache != null) {
          this._$_findViewCache.clear();
       }
     
    }
```

=====
kotlin的代码在做什么？

welcomeMessage.text = "Hello Kotlin!"



((TextView)this._$_findCachedViewById(id.welcomeMessage)).setText((CharSequence)"Hello Kotlin!");


id.text 这样的属性是不真实的，插件不会为每个视图生成属性。它只会在编译期间替换代码以访问视图缓存，并将其转为正确的类型并调用该方法。



Kotlin Android Extensions on fragments

这个插件也可用在Fragments中. 在Fragments中的问题是View可以被重新创建，但是Fragment的实例将会保持存活。 然后怎么办？这意味着在cache中的View缓存将不再可用。

让我们看看把
```
welcomeMessage.text = "Hello Kotlin!"
```
移动到Fragment后它生成的代码。我创建了一个简单的Fragment，并且用上面的同一个xml:

```
class Fragment : Fragment() {
 
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment, container, false)
    }
 
    override fun onViewCreated(view: View?, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        welcomeMessage.text = "Hello Kotlin!"
    }
}
```

在onViewCreated, 我改变了TextView中的内容。那么生成的二级制文件是什么样的呢？

看看反编译后的Java代码：

```
// $FF: synthetic method
public void onDestroyView() {
   super.onDestroyView();
   this._$_clearFindViewByIdCache();
}
```

当View被销毁的时候，这个方法将会被调用 clearFindViewByIdCache，所以我们是安全的。

Kotlin Android extensions on a Custom View

它在自定义View中是一样工作的，让我们看看像这样的View：

```

<merge xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    
    <ImageView
        android:id="@+id/itemImage"
        android:layout_width="match_parent"
        android:layout_height="200dp"/>
    
    <TextView
        android:id="@+id/itemTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
 
</merge>
```

我创建了一个非常简单的自定义View并且使用@JvmOverloads 注解来生成构造方法。
```
class CustomView @JvmOverloads constructor(
        context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : LinearLayout(context, attrs, defStyleAttr) {
 
    init {
        LayoutInflater.from(context).inflate(R.layout.view_custom, this, true)
        itemTitle.text = "Hello Kotlin!"
    }
}
```

在上面的例子中，我改变了ID的名字为 itemTitle。生成的代码应该尝试着从cache中去查找View。在复制同样的代码没有意思，但是我们可以看看下面这行代码的改变：
```
((TextView)this._$_findCachedViewById(id.itemTitle)).setText((CharSequence)"Hello Kotlin!");
```

在自定义View中也是第一次会调用findViewById.

从另一个视图充恢复视图:

Kotlin Android Extensions提供的最后一个替代方案是直接从另一个视图使用这些属性。

如果是在一个Adapter中使用呢，会是什么样子？

你也可以直接使用这个插件来获取子View：

```
val itemView = ...
itemView.itemImage.setImageResource(R.mipmap.ic_launcher)
itemView.itemTitle.text = "My Text"
```
尽管插件也可以帮你进行完全导入，这儿有点小不一样：

```
import i kotlinx.android.synthetic.main.view_item.view.*
```

下面这些事儿是你需要知道的:

* 在编译时，你可以从任何布局中引用任何View。也就是说，你可以引用B布局中的TextView，尽管本应该引用的是A布局中的TextView。这在执行时，将会失败，因为它尝试恢复一个不存在的View。
* 在这种情况下，View没有被缓存下来。

为什么是这样？与之前的情况相反，此处插件没有地方为缓存生成所需的代码。如果您再次查看从视图调用属性时插件生成的代码，您将看到：

	
(((TextView)itemView.findViewById(id.itemTitle)).setText((CharSequence)"My Text");

如您所见，没有调用缓存。如果您的视图很复杂并且您在适配器中使用它，请小心。它可能会影响性能。

1.1.4 中的kotlin Android extensions:

自从这个新版本的Kotlin以来，Android Extensions已经集成了一些新的有趣功能：任何类中的缓存（有趣的包括  ViewHolder），以及一个新的注释  @Parcelize。还有一种方法可以自定义生成的缓存。



