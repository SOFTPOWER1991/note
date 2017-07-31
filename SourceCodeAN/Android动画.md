本文主要讲解Android动画的相关内容，其中包括如下部分：
1. Android动画的分类
2. View动画
3. 自定义View动画
4. 帧动画
5. View动画的使用场景
6. 属性动画的使用及其原理

# 1. Android动画的分类

Android的动画分为三类：View动画、帧动画、属性动画。

**View动画**：通过对场景里的对象不断做图像变换（平移、缩放、旋转、透明度）从而产生动画效果，是一种渐进式的动画，并且View动画支持自定义。

**帧动画**：通过顺序播放一系列图像从而产生动画效果，可以理解为图片切换动画，如果图片过多过大就会导致OOM。

**属性动画**：通过动态地改变对象的属性从而达到动画效果，为API11的新特性，低版本无法使用属性动画，需要使用兼容库。

# 2. View动画

View动画的作用对象是View，支持4中动画效果，分别是平移动画TranslateAnimation、缩放动画ScaleAnimation、旋转动画RotateAnimation、透明度动画AlphaAnimation。他们都可以通过代码来动态创建。

View动画的四种变换表格：

|名称|标签|子类|效果|
|:---:|:---:|:---:|:---:|
|平移动画| \<translate\> |TranslateAnimation|移动View|
|缩放动画|\<scale>|ScaleAnimation|放大or缩小View|
|旋转动画|\<rotate>|RotateAnimation|旋转View|
|透明度动画|\<alpha>|AlphaAnimation|改变View的透明度|

在Android工程中，动画文件存放于：res/anim/animation_test.xml。

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator" 
    android:shareInterpolator="true"
    android:fillAfter="true">

    <translate
        android:duration="100"
        android:fromXDelta="0.0f"
        android:fromYDelta="0.0f"
        android:toXDelta="100.0f"
        android:toYDelta="100.0f" />

    <scale
        android:fromXScale="0.0f"
        android:fromYScale="0.0f"
        android:pivotX="0.0f"
        android:pivotY="0.0f"
        android:toXScale="1.0f"
        android:toYScale="1.0f" />


    <rotate
        android:fromDegrees="0.0f"
        android:pivotX="0.0f"
        android:pivotY="0.0f"
        android:toDegrees="30.0f" />

    <alpha
        android:fromAlpha="0.0f"
        android:toAlpha="1.0f" />

</set>

```
View动画，可以是单个动画，也可以由一系列动画组成。

\<set>标签表示动画集合，对应AnimationSet类，它可以包含若干个动画，并且它的内部也可以嵌套其他动画集合，它的两个属性含义如下：

* **android:interpolator**

> 表示动画集合所采用的插值器，插值器影响了动画的速度。
 
* **android:shareInterpolator**

> 表示集合中的动画是否和集合共享同一个插值器。如果集合不指定插值器，那么子动画就需要单独指定所需的插值器或者使用默认值。

下面对各个View动画的标签进行讲解：

**\<translate>:平移动画**，对应TranslateAnimation类，它可以使一个View在水平和竖直方向完成平移的动画效果，它的属性含义如下：

* android：fromXDelta ——表示x的起始值，比如：0；
* android：toXDelta ——表示x的结束值，比如：100；
* android: fromYDelta ——表示Y的起始值；
* android: toYDelta —— 表示Y的结束值。

**\<scale>:缩放动画**，对应ScaleAnimation，它可以使View具有放大或者缩小的动画效果，它的含义如下：

* android:fromXScale—— 水平方向缩放的起始值，比如0.5；
* android:toXScale —— 水平方向的结束值，比如1.2；
* android:fromYScale —— 竖直方向缩放的起始值；
* android:toYScale —— 竖直方向缩放的起始值；

**\<rotate>:旋转动画**，对应RotateAnimation，它可以使View具有旋转的动画效果，它的属性含义如下：

* android:fromDegrees——旋转开始的角度，比如0；
* android:toDegrees —— 旋转结束的角度，比如180；
* android:pivotX —— 旋转的轴点的x坐标；
* android:pivotY —— 旋转的轴点的Y坐标；

**\<alpha>:透明度动画**，对应AlphaAnimation，可以改变View的透明度，含义如下：

* android:fromAlpha——表示透明度的起始值，比如0.1；
* android:toAlpha——表示透明度的结束值，比如1；

除此之外，还有常用属性：

* android：duration —— 动画的持续时间；
* android:fillAfter —— 动画结束以后View是否停留在结束为止，true表示停留在结束为止，false表示不停留。

动画的具体使用：

```
Button button = (Button)findViewById(R.id.button);
Animation animation = AnimationUtils.loadAnimation(this, R.animation_test);
button.startAnimation(animation);
```

除此之外，可以通过Animation的setAnimationListener方法可以给View动画添加过程监听，接口如下，接口内容如下：

```
public static interface AnimationListener{

	void onAnimationStart(Animation animation);
	void onAnimationEnd(Animation animation);
	void onAnimationRepeat(Animation animation);

}
```


