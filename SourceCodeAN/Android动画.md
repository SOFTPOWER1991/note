本文主要讲解Android动画的相关内容，其中包括如下部分：
1. Android动画的分类
2. View动画
3. 自定义View动画
4. 帧动画
5. View动画的使用场景
6. 属性动画的使用及其原理
7. 动画使用中的注意事项

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

# 3. 自定义View动画

自定义View动画：

1. 继承Anima同这个抽象类，重写它的initialize 和 applyTransformation 方法，在initialize方法中做一些初始化工作，在applyTransformation中进行相应的矩阵变换即可。
2. 采用Camera简化矩阵变换。

# 4. 帧动画

帧动画是顺序播放一组预定义好的图片。系统提供了一个AnimationDrawable来使用帧动画。使用时：

1. 在XML中定义一个AnimationDrawable
2. 在代码中进行播放

注意事项：

> 尽量避免使用较大图片，否则容易发生OOM。

# 5. View动画的特殊使用场景

除了View动画的四种形式，View动画还可以有一下特殊使用场景：在ViewGroup中控制子元素的出场效果，在Activity中实现不同Activity之间的切换效果。

## 5.1 LayoutAnimation
LayoutAnimation 作用于ViewGroup，为ViewGroup指定一个动画，这样当它的子元素出场时都会具有这种动画效果。

具体使用步骤：

1. 定义LayoutAnimation

	```
	<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
        android:delay="30%"
        android:animationOrder="reverse"
        android:animation="@anim/slide_right"/>
	
	```
2. 指定具体的入场动画
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
3. 为ViewGroup指定android:layoutAnimation

	```
	<ListView
		......
		android:layoutAnimation = "@anim/anim_layout"
		......
	/>
	```
	
除了在XML中配置之外，还可以借助LayoutAnimationController在代码中实现
	
```
	//通过加载XML动画设置文件来创建一个Animation对象；
   Animation animation=AnimationUtils.loadAnimation(this, R.anim.slide_right);   //得到一个LayoutAnimationController对象；
   LayoutAnimationController controller = new LayoutAnimationController(animation);   //设置控件显示的顺序；
   controller.setOrder(LayoutAnimationController.ORDER_REVERSE);   //设置控件显示间隔时间；
   controller.setDelay(0.3);   //为ListView设置LayoutAnimationController属性；
   listView.setLayoutAnimation(controller);
   listView.startLayoutAnimation();
```

## 5.2 Activity的切换效果

Activity有默认的切换效果，这个效果也可以自定义，这要用到overridePendingTransition(int enterAnim , int exitAnim)这个方法，这个方法必须在startActivity(Intent)或者finish()之后被调用才能生效。

参数含义：

* enterAnim ——Activity被打开时，所需的动画资源id;
* exitAnim —— Activity被暂停时，所需的动画资源id.

Fragment也可以添加切换动画，可以通过FragmentTransaction中的setCustomAnimations()方法来添加切换动画。

# 6. 属性动画的使用及其原理

属性动画是API 11 新加入的特性，动画效果进行了加强，不再像View动画那样只能支持四种简单的变换。属性动画中有ValueAnimator、ObjectAnimator和AnimatorSet等概念，通过他们我们可以实现绚丽的动画。

## 6.1 使用属性动画
属性动画可以对任意对象的属性进行动画而不仅仅是View，动画默认时间间隔300ms，默认帧率是10ms/帧。

常用的几个动画类是：ValueAnimator、ObjectAnimator、AnimatorSet。

使用案例：

1. 改变一个对象的translationY属性
	```
	ObjectAnimator.ofFloat(view,"translationY",values).start();
	```
	
2. 改变一个对象的背景色属性
	```
	ValueAnimator colorAnim = ObjectAnimator.ofInt(view,"backgroundColor",/*red*/0xffff8080,/*blue*/0xff8080ff);
 colorAnim.setDuration(2000);
 colorAnim.setEvaluator(new ArgbEvaluator());
 colorAnim.setRepeatCount(ValueAnimator.INFINITE);
 colorAnim.setRepeatMode(ValueAnimator.REVERSE);
 colorAnim.start();

	```
3. 动画集合
	```
	AnimatorSet set = new AnimatorSet();
	set.playTogether(animator1,animator2,animator3);
	set.setDuration(3*1000).start();
	```

属性动画除了可以通过代码实现外，还可以通过XML来定义。属性动画需要定义在res/animator下。

```
<set xmlns:android="http://schemas.android.com/apk/res/android"  
    android:ordering="sequentially" >  
  
    <objectAnimator  
        android:duration="2000"  
        android:propertyName="translationX"  
        android:valueFrom="-500"  
        android:valueTo="0"  
        android:valueType="floatType" >  
    </objectAnimator>  
  
    <set android:ordering="together" >  
        <objectAnimator  
            android:duration="3000"  
            android:propertyName="rotation"  
            android:valueFrom="0"  
            android:valueTo="360"  
            android:valueType="floatType" >  
        </objectAnimator>  
  
        <set android:ordering="sequentially" >  
            <objectAnimator  
                android:duration="1500"  
                android:propertyName="alpha"  
                android:valueFrom="1"  
                android:valueTo="0"  
                android:valueType="floatType" >  
            </objectAnimator>  
            <objectAnimator  
                android:duration="1500"  
                android:propertyName="alpha"  
                android:valueFrom="0"  
                android:valueTo="1"  
                android:valueType="floatType" >  
            </objectAnimator>  
        </set>  
    </set>  
  
</set>  
```
在XML中可以定义ValueAnimator、ObjectAnimator以及AnimatorSet，其中\<set>对应AnimatorSet，\<animtor>标签对应ValueAnimator，\<objectAnimator>对应ObjectAnimator。\<set>标签的android:ordering属性有两个可选值:"together"和“sequentially”，其中：“together”表示动画集合中的子动画同时播放，“sequentially”则表示动画集合中的子动画按照前后顺序依次播放。默认值为together。

\<objectAnimator>标签的各个属性的含义：

* android:propertyName——表示属性动画的作用对象的属性的名称。
* android:duration —— 表示动画的时长。
* android:valueFrom—— 表示属性的起始值。
* android:valueTo —— 表示属性的结束值。
* android:startOffset—— 表示动画的延迟时间，当动画开始后，需要延迟多久才会真正播放此动画。
* android:repeatCount —— 表示动画的重复次数。
* android:repeatMode —— 表示动画的重复模式。
* android:valueType —— 表示android:propertyName指定的属性的类型，有intType 和floatType两个可选类型，分别表示属性的类型为整型和浮点型。如果android:propertyName多指定的属性表示的是颜色，那么不需要指定android:valueType，系统会自动对颜色类型的属性做处理。

需要强调的东西：

1. android:repeatCount ,表示动画循环次数。默认值为0 ， -1 表示无线循环。
2. android:repeatMode，表示动画循环的模式，有两个选项：“repeat” 和 “reverse”，分别表示连续重复和逆向重复。逆向重复是指：第一次播放完以后，第二次会倒着播放动画，第三次再重头开始播放动画，第四次再倒着播放动画，如此反复。

使用，通过代码来控制：

```
AnimatorSet set = AnimatorInflater.loadAnimator(myContext, R.anim.property_animator)；
set.setTarget(mButton);
set.start();
```

## 6.2 理解插值器和估值器

TimeInterpolator时间插值器：作用根据时间流逝的百分比来计算出当前属性值改变的百分比。内置的有：LinearInterpolator(线性插值器，匀速动画)、AccelerateDecelerateInterpolator(加速减速插值器：动画两头慢中间快)、DecelerateInterpolator(减速插值器：动画越来越慢)。

TypeEvaluator类型估值算法，估值器：作用根据当前属性改变的百分比来计算改变后的属性值，系统内置的有：IntEvaluator(针对整型属性)、FloatEvaluator(浮点型属性)和ArgbEvaluator(Color属性)。

插值器和估值器，是实现非匀速动画的关键。

## 6.3 属性动画的监听器

属性动画提供了监听器用于监听动画的播放过程，主要有两个接口：
`AnimatorUpdateListener`,`AnimatorListener`.

AnimatorListener代码如下：

```
  /**
     * <p>An animation listener receives notifications from an animation.
     * Notifications indicate animation related events, such as the end or the
     * repetition of the animation.</p>
     */
    public static interface AnimatorListener {
        /**
         * <p>Notifies the start of the animation.</p>
         *
         * @param animation The started animation.
         */
        void onAnimationStart(Animator animation);

        /**
         * <p>Notifies the end of the animation. This callback is not invoked
         * for animations with repeat count set to INFINITE.</p>
         *
         * @param animation The animation which reached its end.
         */
        void onAnimationEnd(Animator animation);

        /**
         * <p>Notifies the cancellation of the animation. This callback is not invoked
         * for animations with repeat count set to INFINITE.</p>
         *
         * @param animation The animation which was canceled.
         */
        void onAnimationCancel(Animator animation);

        /**
         * <p>Notifies the repetition of the animation.</p>
         *
         * @param animation The animation which was repeated.
         */
        void onAnimationRepeat(Animator animation);
    }
```
它可以监听动画的开始、结束、取消以及重复播放。

系统还提供了AnimatorListenerAdapter这个类，代码如下：
```
/**
 * Implementors of this interface can add themselves as update listeners
 * to an <code>ValueAnimator</code> instance to receive callbacks on every animation
 * frame, after the current frame's values have been calculated for that
 * <code>ValueAnimator</code>.
 */
public static interface AnimatorUpdateListener {
    /**
     * <p>Notifies the occurrence of another frame of the animation.</p>
     *
     * @param animation The animation which was repeated.
     */
    void onAnimationUpdate(ValueAnimator animation);

}
```

AnimatorUpdateListener会监听整个动画过程，动画是由许多帧组成的，每播放一帧，onAnimationUpdate就会被调用一次。

## 6.4 对任意属性做动画

以对Object的属性abc做动画为例，如果想让动画生效，要满足两个条件：
1. Object必须要提供setAbc方法，如果动画的时候没有传递初始值，那么还要提供getAbc方法，因为系统要去取abc属性的初始值（如果这个不满足，直接Crash）
2. Object的setAbc对属性abc所做的改变必须能够通过某种方法反映出来，比如会带来UI的改变（如果条件不满足，动画无效果但不会Crash）

将Button的宽度做扩大到500像素的动画时，发现不起作用，解决方案，官方文档给出了3中解决方案：

* 给你的对象加上get和set方法，如果你有权限的话；
* 用一个类来包装原始对象，间接为其提供get和set方法；
* 采用ValueAnimator，监听动画过程，自己实现属性的改变。 

## 6.5 属性动画的工作原理

属性动画的原理：
> 属性动画要求**动画作用的对象提供该属性的get和set方法**，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用set方法，每次传递给set的值都不一样，确切来说是随着时间的推移，所传递的值越来越接近最终值。

以如下代码为入口，分析源码：

```
ObjectAnimator objectAnimator = ObjectAnimator.ofInt(this , "width" , 500).setDuration(5000); objectAnimator.start();
```
从start方法开始:
```
@Override
public void start() {
    AnimationHandler.getInstance().autoCancelBasedOn(this);
    if (DBG) {
        Log.d(LOG_TAG, "Anim target, duration: " + getTarget() + ", " + getDuration());
        for (int i = 0; i < mValues.length; ++i) {
            PropertyValuesHolder pvh = mValues[i];
            Log.d(LOG_TAG, "   Values[" + i + "]: " +
                pvh.getPropertyName() + ", " + pvh.mKeyframes.getValue(0) + ", " +
                pvh.mKeyframes.getValue(1));
        }
    }
    super.start();
}
```

首先在start方法中通过AnimationHandler把相同的动画取消掉。然后调用super.start()方法，ObjectAnimator的父类是ValueAnimator，所以看看ValueAnimator的方法:
```
private void start(boolean playBackwards) {
        if (Looper.myLooper() == null) {
            throw new AndroidRuntimeException("Animators may only be run on Looper threads");
        }
        mReversing = playBackwards;
        // Special case: reversing from seek-to-0 should act as if not seeked at all.
        if (playBackwards && mSeekFraction != -1 && mSeekFraction != 0) {
            if (mRepeatCount == INFINITE) {
                // Calculate the fraction of the current iteration.
                float fraction = (float) (mSeekFraction - Math.floor(mSeekFraction));
                mSeekFraction = 1 - fraction;
            } else {
                mSeekFraction = 1 + mRepeatCount - mSeekFraction;
            }
        }
        mStarted = true;
        mPaused = false;
        mRunning = false;
        mAnimationEndRequested = false;
        // Resets mLastFrameTime when start() is called, so that if the animation was running,
        // calling start() would put the animation in the
        // started-but-not-yet-reached-the-first-frame phase.
        mLastFrameTime = 0;
        AnimationHandler animationHandler = AnimationHandler.getInstance();
        animationHandler.addAnimationFrameCallback(this, (long) (mStartDelay * sDurationScale));

        if (mStartDelay == 0 || mSeekFraction >= 0) {
            // If there's no start delay, init the animation and notify start listeners right away
            // to be consistent with the previous behavior. Otherwise, postpone this until the first
            // frame after the start delay.
            startAnimation();
            if (mSeekFraction == -1) {
                // No seek, start at play time 0. Note that the reason we are not using fraction 0
                // is because for animations with 0 duration, we want to be consistent with pre-N
                // behavior: skip to the final value immediately.
                setCurrentPlayTime(0);
            } else {
                setCurrentFraction(mSeekFraction);
            }
        }
    }
```
可以看出属性动画需要运行在有Looper的线程中，最后会调用到startAnimation（）来开启属性动画。
```
 /**
     * Called internally to start an animation by adding it to the active animations list. Must be
     * called on the UI thread.
     */
    private void startAnimation() {
        if (Trace.isTagEnabled(Trace.TRACE_TAG_VIEW)) {
            Trace.asyncTraceBegin(Trace.TRACE_TAG_VIEW, getNameForTrace(),
                    System.identityHashCode(this));
        }

        mAnimationEndRequested = false;
        initAnimation();
        mRunning = true;
        if (mSeekFraction >= 0) {
            mOverallFraction = mSeekFraction;
        } else {
            mOverallFraction = 0f;
        }
        if (mListeners != null) {
            notifyStartListeners();
        }
    }
```
这个方法将需要执行的动画放入动画列表里。必须在UI线程调用。最终的开启动画操作，走到了：
```
private void notifyStartListeners() {
        if (mListeners != null && !mStartListenersCalled) {
            ArrayList<AnimatorListener> tmpListeners =
                    (ArrayList<AnimatorListener>) mListeners.clone();
            int numListeners = tmpListeners.size();
            for (int i = 0; i < numListeners; ++i) {
                tmpListeners.get(i).onAnimationStart(this);
            }
        }
        mStartListenersCalled = true;
    }
```
在notifyStartListeners中，取出每一个动画的监听器监听每一个动画的执行。

# 7. 动画使用中的注意事项

* OOM问题
> 帧动画中，图片数量过多、图片较大时极容易出现OOM。

* 内存泄露
> 属性动画中的无限循环动画，需要在Activity退出时及时停止，否则将导致Activity无法释放从而造成内存泄露。
* 兼容性问题
> 动画在3.0以下的系统上有兼容性问题，在某些特殊场景可能无法正常工作，要做兼容性适配。
* View动画的问题
> View动画是对View的影像做动画，并不是真正地改变View的状态，因此有时候会出现动画完成后View无法隐藏的现象，即setVisibility(View.GONE)失效了，这时候需要调用view.clearAnimation()清楚View动画即可。
* 不要使用px
> 动画进行过程中，尽量使用dp,使用px会导致在不同设备上有不同的效果。
* 动画元素的交互
> 将View移动后，在Android3.0以前的系统上，不管是View动画还是属性动画，新位置都不发触发点击事件，同时，老位置仍可以触发点击事件。 3.0以后，属性动画的单击事件触发位置为移动后的位置，但是View动画仍然在原位置。
* 硬件加速
> 使用动画过程中，建议开启硬件加速，这会提高动画流畅性。

