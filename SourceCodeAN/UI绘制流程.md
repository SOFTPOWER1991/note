在自定义View的时候，掌握View的底层工作原理，也就是：View的measure、Layout、draw流程，可以帮助我做出比较有意思的自定义View。

因此，本文主要讲解：
> View的工作原理，其中包括：测量流程、布局流程、绘制流程。

其中包括如下内容：

* 基础知识准备：
	* 认识ViewRoot和DecorView
	* 理解MeasureSpec
	* MeasureSpec和LayoutParams的关系
* View工作流程源码分析：
	* Measure过程源码分析
		* View的Measure过程
		* ViewGroup的Measure过程
	* Layout过程源码分析
	* draw过程源码分析
* 常用的几种自定义控件方式

# 1.基础知识准备

## 1.1 认识ViewRoot和DecorView
### 1.1.1 认识ViewRoot
`ViewRoot`对应于`ViewRootImpl`类，它是连接WindowManager和DecorView的纽带，View的三大流程都是通过ViewRoot完成的。在ActivityThread中，当Activity对象被创建完毕后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联，源码如下：

```
root = new ViewRootImpl(view.getContext(), display);
root.setView(view, wparams, panelParentView);
```
**View的绘制流程是从ViewRoot的`performTraversals`方法开始的，它经过Measure、Layout和draw三个过程才能最终将一个View绘制出来**。

### 1.1.2 认识DecorView
`DecorView`作为顶级View，包含一个竖直方向的LinearLayout布局，这个Linearlayout布局包括两部分：上面的标题栏，下面的内容栏。我们在Activity中通过setContentView所设置的布局文件其实就是被加到了内容栏之中的。DecorView其实是一个FrameLayout，View层的事件都先经过DecorView，然后才传递给我们的View。

## 1.2 理解MeasureSpec

MeasureSpec是什么呢？
> MeasureSpec简单说是：测量规格。往细的说：MeasureSpec很大程度上决定了一个View的尺寸规格，之所以说是很大程度上是因为这个过程还受父容器的影响，因为父容器影响View的MeasureSpec的创建过程。在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换为对应的MeasureSpec，然后再根据这个MeasureSpec来测量出View的宽、高。

MeasureSpec的组成：
> MeasureSpec代表一个32位int值。
> 高2位代表：SpecMode。SpecMode代表`测量模式`。
> 低30位代表：SpecSize。SpecSize代表`在某种测量模式下的规格大小`

下面看看MeasureSpec的源码：

```
public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
        /**
         * Measure specification mode: The parent has not imposed any constraint
         * on the child. It can be whatever size it wants.
         */
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;

        /**
         * Measure specification mode: The parent has determined an exact size
         * for the child. The child is going to be given those bounds regardless
         * of how big it wants to be.
         */
        public static final int EXACTLY     = 1 << MODE_SHIFT;

        /**
         * Measure specification mode: The child can be as large as it wants up
         * to the specified size.
         */
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        /**
         * Creates a measure specification based on the supplied size and mode.
         *
         * The mode must always be one of the following:
         * <ul>
         *  <li>{@link android.view.View.MeasureSpec#UNSPECIFIED}</li>
         *  <li>{@link android.view.View.MeasureSpec#EXACTLY}</li>
         *  <li>{@link android.view.View.MeasureSpec#AT_MOST}</li>
         * </ul>
         *
         * <p><strong>Note:</strong> On API level 17 and lower, makeMeasureSpec's
         * implementation was such that the order of arguments did not matter
         * and overflow in either value could impact the resulting MeasureSpec.
         * {@link android.widget.RelativeLayout} was affected by this bug.
         * Apps targeting API levels greater than 17 will get the fixed, more strict
         * behavior.</p>
         *
         * @param size the size of the measure specification
         * @param mode the mode of the measure specification
         * @return the measure specification based on size and mode
         */
        public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,
                                          @MeasureSpecMode int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        /**
         * Like {@link #makeMeasureSpec(int, int)}, but any spec with a mode of UNSPECIFIED
         * will automatically get a size of 0. Older apps expect this.
         *
         * @hide internal use only for compatibility with system widgets and older apps
         */
        public static int makeSafeMeasureSpec(int size, int mode) {
            if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
                return 0;
            }
            return makeMeasureSpec(size, mode);
        }

        /**
         * Extracts the mode from the supplied measure specification.
         *
         * @param measureSpec the measure specification to extract the mode from
         * @return {@link android.view.View.MeasureSpec#UNSPECIFIED},
         *         {@link android.view.View.MeasureSpec#AT_MOST} or
         *         {@link android.view.View.MeasureSpec#EXACTLY}
         */
        @MeasureSpecMode
        public static int getMode(int measureSpec) {
            //noinspection ResourceType
            return (measureSpec & MODE_MASK);
        }

        /**
         * Extracts the size from the supplied measure specification.
         *
         * @param measureSpec the measure specification to extract the size from
         * @return the size in pixels defined in the supplied measure specification
         */
        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

        ......
}
```
MeasureSpec通过将SpecMode和SpecSize打包成一个int值来避免过多的对象内存分配，为了方便操作，提供了打包和解包方法。

* `makeMeasureSpec`可以根据SpecMode和SpecSize打包出对应的MeasureSpec。
* `getMode`和`getSize`可以从MeasureSpec解包出对应的SpecMode和SpecSize。

SpecMode的有三类：

* UNSPECIFIED
> 父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态。

* EXACTLY
> 父容器已经检测出View所需要的精确大小。这个时候View的最终大小就是SpecSize指定的值。它对应LayoutParams中的match_parent和具体的数值这两种模式。
* AT_MOST:
> 父容器指定了一个可用大小即：SpecSize，View的大小不能大于这个值，具体是什么值要看不同的View具体实现。它对应于LayoutParams中的wrap_content。

## 1.3 MeasureSpec和LayoutParams的关系

* 对于顶级View：DecorView来说：
> 其MeasureSpec由窗口的尺寸和其自身的LayoutParams来共同确定。

* 对于普通View来说：
> 其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定，MeasureSpec一旦确定后，onMeasure中就可以确定View的测量宽高。

下面看一个源码分析：

**对于DecorView来说**，在ViewRootImpl的measureHierarchy方法中有一段代码，展示了DecorView的MeasureSpec创建过程：

```
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
```
其中的desiredWindowWidth，desiredWindowHeight是屏幕的尺寸。

接着在看下getRootMeasureSpec方法：

```
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {

    case ViewGroup.LayoutParams.MATCH_PARENT:
        // Window can't resize. Force root view to be windowSize.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        // Window can resize. Set max size for root view.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        // Window wants to be an exact size. Force root view to be that size.
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
```
从上面的代码可以看出：

DecorView的MeasureSpec产生过程遵从如下规则：

* LayoutParams.MATCH_PARENT:精确模式，大小就是窗口大小。
* LayoutParams.WRAP_CONENT:最大模式，大小不变，但是不能超过窗口的大小。
* 固定大小：精确模式，大小为LayoutParams中指定的大小。

**对于普通View来说**：View的Measure过程由ViewGroup传递而来，先看一下ViewGroup的measureChildWithMargins方法：

```
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
从上面的代码可以看出，对子元素进行Measure，在调用子元素的Measure方法之前会先通过getChildMeasureSpec方法来得到子元素的MeasureSpec。由此可以看出，**子元素的MeasureSpec的创建与父容器的MeasureSpec和子元素本身的LayoutParams有关，此外还和View的margin及padding有关。** 

那getChildMeasureSpec又是如何来获得子元素的MeasureSpec的呢？上代码：

```
	public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);

    int size = Math.max(0, specSize - padding);

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

从上面的代码可以看出，它的主要作用是根据父容器的MeasureSpec同时结合View本身的LayoutParams来确定子元素的MeasureSpec，参数中的padding是指父容器中已占用的空间，因此子元素可用的大小为父容器的尺寸减去padding。代码如下：

```
int specSize = MeasureSpec.getSize(spec);
int size = Math.max(0, specSize - padding);
```

总的来说：
> 只要提供父容器的MeasureSpec和子元素的LayoutParams，就可以快速地确定子元素的MeasureSpec了，有了MeasureSpec就可以进一步确定出子元素测量后的大小了。

# 2 View工作流程源码分析

整体工作流程简介：
> 

## 2.1 Measure过程源码分析
### 2.1.1 View的Measure过程分析
### 2.1.2 ViewGroup的Measure过程分析

## 2.2 Layout过程源码分析
## 2.3 draw过程源码分析

# 3. 常用的自定义控件分析


