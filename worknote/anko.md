Anko

Anko 是Kotlin库，它可以让开发Android应用程序变的更简单和快速。它让你的代码干净并且容易阅读。让你忘掉Java和Android  SDK 的繁琐之处。

他包含以下几部分：

* Anko Commons: 一个轻量级的工具库，包括：
    * Intents
    * Dialogs 和 toasts
    * Logging
    * Resources 和 dimensions；

* Anko Layouts： 快速和类型安全的方式来动态编写Android的布局文件；
* Anko SQLites: Android SQLite的查询DSL和解析器集合。
* Anko Coroutines: 基于kotlinx.coroutines库的实用程序。


====

https://github.com/Kotlin/anko/wiki/Anko-Layouts#why-anko-layouts

为什么会有 Anko Layouts？

为何选择DSL？

默认情况下，Android中的UI 是使用XML 进行布局。它有以下几种不便的地方：

* 它不是类型安全的；
* 它不是空安全的；
* 它会强制你为每个布局编写几乎同样的代码；
* 在设备上解析XML 会浪费CPU 时间和电池；

最重要的是，它不允许你代码重用。

虽然您可以以编程方式创建UI，但它很难完成，因为它有点难看且难以维护。这是一个简单的Kotlin版本（Java中的版本更长）：

```
val act = this
val layout = LinearLayout(act)
layout.orientation = LinearLayout.VERTICAL
val name = EditText(act)
val button = Button(act)
button.text = "Say Hello"
button.setOnClickListener {
    Toast.makeText(act, "Hello, ${name.text}!", Toast.LENGTH_SHORT).show()
}
layout.addView(name)
layout.addView(button)
```


DSL使相同的逻辑易于读取，易于编写且没有运行时开销。这又是：

```
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

请注意，onClick（）支持协程（接受挂起lambda），因此您可以在没有显式异步（UI）调用的情况下编写异步代码。

支持现在已存的代码：

您不必使用Anko重写所有UI。您可以使用Java编写旧类。此外，如果您仍然希望（或有）编写Kotlin活动类并因某种原因而膨胀XML布局，则可以使用View属性，让事情变得简单：

```
// Same as findViewById() but simpler to use
val name = find<TextView>(R.id.name)
name.hint = "Enter your name"
name.onClick { /*do something*/ }
```

它是怎么运行的？

There is no :tophat:. Anko consists of some Kotlin extension functions and properties arranged into type-safe builders, as described under Type Safe Builders.

Since it's somewhat tedious to write all these extensions by hand, they're generated automatically using android.jar files from Android SDK as sources.

它是否可以扩展？


在你的项目中使用Anko布局： 


理解Anko

基础

Anko 支持插件：

Anko 插件已经在Intellij IDEA 和 Android Studio上可以下载到了。它允许你在IDE的工具窗口直接预览使用Anko写的AnkoComponent 类。

安装Anko 支持插件：

使用这个插件：

假设你有一个这样的类使用Anko写的：

```
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MyActivityUI().setContentView(this)
    }
}

class MyActivityUI : AnkoComponent<MyActivity> {
    override fun createView(ui: AnkoContext<MyActivity>) = ui.apply {
        verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { ctx.toast("Hello, ${name.text}!") }
            }
        }
    }.view
}
```

将光标定位在MyActivityUI 的声明出，打开Anko 布局预览窗口 （View--> Tool Windows -> Anko Layout Preview） 然后按下Refresh.

这需要rebuilding以下工程，因此在真正的预览出来之前需要一些时间。

XML 到 DSL 的转换：

这个插件也支持将XML布局文件转换为Anko布局代码。打开XML文件选择 Code-> Covert to Anko Layouts DSL 。你可以同时转好好多布局文件。



