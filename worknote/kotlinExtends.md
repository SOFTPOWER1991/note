### 一、扩展是什么
> Kotlin能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式，通过叫做 **扩展** 的特殊声明来完成。

### 二、Kotlin中的扩展：
> Kotlin支持 **扩展函数** 和 **扩展属性**

####  2.1 扩展函数的声明: 
> 声明一个扩展函数，需要用一个 接收者类型也就是被扩展的类型来作为他的前缀。其中的this关键字在扩展函数内部对应到接收者对象（传过来的在点符号之前的对象）。

常见的需求：
> 当一个字符串为空时，将这个TextView隐藏;
> 如果字符串不为空时再将这个TextView展示出来，并将字符串展示出来

我们可以为TextView定义一个扩展函数出来：
```
/**
 * 适用场景：
 *
 * 当一个字符串为空时，将这个TextView隐藏;
 * 如果字符串不为空时再将这个TextView展示出来，并将字符串展示出来
 *
 * 在Java中，要这样来写:
 *
 *      if(!TextUtils.isEmpty(inputString){
 *          textView.setText(inputString)
 *          textView.setVisibility(View.VISIBLE)
 *      }else{
 *          textView.setVisibility(View.GONE)
 *      }
 *
 * 在Kotlin中 使用扩展, 只需要这样写即可:
 *
 *      textView.setEnhancedVisibilityWithText(inputString)
 */
fun TextView.setEnhancedVisibilityWithText(inputString: String) {
    if (!TextUtils.isEmpty(inputString)) {
        this.text = inputString
        this.visibility = View.VISIBLE
    } else {
        this.visibility = View.GONE
    }
}
```

这样，我们在使用的时候就可以这样使用了：
```
textView.setEnhancedVisibilityWithText(inputString)
```
#### 2.2 扩展属性的声明：

扩展属性提供了一种方法用能通过属性语法进行访问的API来扩展。但它们不能拥有任何状态。扩展属性并不等于这个类上真实的属性，它并没有实际的将这个属性插入到这个类当中。

因此，对扩展属性来说，幕后字段field是不存在的，所以扩展属性不能有初始化器。虽然扩展属性没有幕后字段，但是它们的行为我们依然可以通过显示提供的getters/setters 来定义。

```
val String.lastChar: Char
    get() = get(length - 1)

var StringBuilder.lastChar: Char
    get() =  get(length - 1)
    set(value) {
        this.setCharAt(length -1, value)
    }
```
使用的时候就可以这样用了：
```
"chanls".lastChar
```
### 三、扩展的原理

扩展不能真正的修改他们所扩展的类。通过定义一个扩展，并没有在一个类中插入新成员，仅仅是通过该类型的变量用点表达式去调用这个函数。

上面的TextView的扩展方法，借助Kotlin tools提供的反编译功能，查看反编译后的Java文件，代码如下:
```
 public static final void setEnhancedVisibilityWithText(@NotNull TextView $receiver, @NotNull String inputString) {
      Intrinsics.checkParameterIsNotNull($receiver, "$receiver");
      Intrinsics.checkParameterIsNotNull(inputString, "inputString");
      if (!TextUtils.isEmpty((CharSequence)inputString)) {
         $receiver.setText((CharSequence)inputString);
         $receiver.setVisibility(0);
      } else {
         $receiver.setVisibility(8);
      }

   }
```

至此，Kotlin扩展的原理就了然了。
### 四、 扩展出现要解决的问题：

在此之前如果要解决这个问题，我们需要搞出一堆的XXXUtils.java的工具类，然后通过这样的方式：
```
XXXUtils.setEnhancedVisibilityWithText(view, string)
```
有了扩展，就可以这样写了：
```
view.setEnhancedVisibilityWithText(string)
```

### 五、如何在Java中调用Kotlin的扩展

以易项优选中的kotlin扩展集合类KotlinExtendManager.kt为例，在Java中调用的时候就变成:

```
KotlinExtendManagerKt.setEnhancedVisibilityWithText(@NotNull TextView $receiver, @NotNull String inputString)
```

### 六、易项优选中的Kotlin的扩展

```
package com.ethercap.base.android.extension

import android.content.Context
import android.text.TextUtils
import android.view.View
import android.widget.TextView

/**
 * @author zg
 * @date 2018/12/18 2:03 PM
 * @Description: 存放kotlin中的扩展 ， 将Kotlin中的扩展进行集中管理
 */


/**
 * 适用场景:
 *
 * 给String扩展出一个方法，当String不为空时返回本身，如果为null or  "" , 返回 ""
 *
 * 在Java中, 对字符串的判断 :
 *
 * String showText = "";
 *
 * if(!TextUtils.isEmpty(inputString)){
 *    showText = inputString;
 * }else{
 *    showText = "";
 * }
 *
 * 适用方式： textView.setText(showText)
 *
 * 有了Kotlin 扩展，可以这样适用：
 *
 * textView.text = inputString.getEnhancedString()
 */
fun String.getEnhancedString(): String = if (!TextUtils.isEmpty(this)) {
    this
} else {
    ""
}

/**
 * 适用场景：
 *
 * 当一个字符串为空时，将这个View隐藏; 如果字符串不为空时再将这个view展示出来
 *
 * 在 Java 中， 要这样来写:
 *
 *      if(!TextUtils.isEmpty(inputString){
 *          imgView.visibility = View.VISIBLE;
 *      }else{
 *          imgView.visibility = View.GONE;
 *      }
 *
 * 在Kotlin中 使用扩展, 只需要这样写即可:
 *
 *      imgView.setEnhancedVisibility(inputString)
 *
 */
fun View.setEnhancedVisibility(inputString: String) {
    this.visibility = if (!TextUtils.isEmpty(inputString)) {
        View.VISIBLE
    } else {
        View.GONE
    }
}

/**
 * 适用场景：
 *
 * 当给定一个条件：
 *
 *  为true 时，将View 展示出来；
 *  为false时，将View 隐藏；
 */
fun View.setEnhancedVisibility(boolean: Boolean) {
    this.visibility = if (boolean) View.VISIBLE else View.GONE
}

/**
 * 适用场景：
 *
 * 当一个字符串为空时，将这个TextView隐藏;
 * 如果字符串不为空时再将这个TextView展示出来，并将字符串展示出来
 *
 * 在 Java 中， 要这样来写:
 *
 *      if(!TextUtils.isEmpty(inputString){
 *          textView.setText(inputString)
 *          textView.setVisibility(View.VISIBLE)
 *      }else{
 *          textView.setVisibility(View.GONE)
 *      }
 *
 * 在Kotlin中 使用扩展, 只需要这样写即可:
 *
 *      textView.setEnhancedVisibilityWithText(inputString)
 *
 */
fun TextView.setEnhancedVisibilityWithText(inputString: String) {
    if (!TextUtils.isEmpty(inputString)) {
        this.text = inputString
        this.visibility = View.VISIBLE
    } else {
        this.visibility = View.GONE
    }
}

```