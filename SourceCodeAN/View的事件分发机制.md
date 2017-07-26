本文主要讲解我对Android中事件分发机制的理解，其中包括如下内容：

* 1. View事件分发，这个事件指的是什么？
* 2. 实例：点击屏幕上的一个按钮后，事件分发过程
* 3. Android中跟事件分发有关的方法
* 4. Android中有事件传递能力的类
* 5. 事件分发源码解析
	* 5.1 Activity对事件的分发过程
	* 5.2 ViewGroup对事件的分发过程
		* 5.2.1 当ViewGroup拦截事件的时候
		* 5.2.2 当ViewGroup不拦截事件的时候
	* 5.3 View对事件的分发过程


# 1. View事件分发，这个事件指的是什么？

> 此处事件指 MotionEvent，即点击事件。所谓的点击事件的事件分发，其实就是对MotionEvent事件的分发过程，即当一个MotionEvent产生之后，系统需要把这个事件传递给一个具体的View，而这个传递过程就是分发过程。

MotionEvent是一系列事件的封装，其中包括如下事件：
* ACTION_DOWN——手指刚刚接触屏幕；
* ACTION_MOVE--手指在屏幕上移动；
* ACTION_UP--手指在屏幕上松开的一瞬间。

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，有如下几种情况：

* 点击屏幕后离开松开，事件序列为：DOWN -> UP;
* 点击屏幕滑动一会再松开，事件序列为：DOWN->MOVE->...->MOVE->UP.

通过MotionEvent可以获得点击事件发生的X坐标和Y坐标：

> 系统提供的两组方法：getX/getY 和 getRawX/getRawY。
> 1. getX、getY返回的是相对于**当前View左上角的x 和 y坐标**。
> 2. getRawX、getRawY返回的是相对于**手机屏幕左上角的x和y坐标**。

# 2. 实例：点击屏幕上的一个按钮后，事件分发过程

在点击屏幕上的一个按钮后，事件分发过程如下：

> Activity->Window->顶级View（DocorView or ViewGroup）->子View

事件总是先传递给Activity，Activity传递给Window，此处的Window指PhoneWindow，最后Window再传递给顶级View也就是DocorView，而DocorView本身就是一个ViewGroup。DocorView接收到事件后，就会按照事件分发机制去分发事件。

如果一个View的onTouchEvent返回false，那么它的父容器的onTouchEvent就会被调用，依次类推，如果所有的元素都不处理这个事件，那么这个事件将会最终传递给Activity处理。即：Activity的onTouchEvent方法会被调用。

# 3. Android中跟事件分发有关的方法：

```
public boolean dispatchTouchEvent(MotionEvent event)
```
> 用来进行事件的分发。如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

```
public boolean onInterceptTouchEvent(MotionEvent event)
```
> 在dispatchTouchEvent方法内部，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列当中，此方法不会被再次调用，返回结果表示是否拦截当前事件。

```
public boolean onTouchEvent(MotionEvent event)
```
> 在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件。如果一旦拦截了某个事件，那么同一个事件序列内的所有事件都会直接交给它处理，因为同一个事件序列中的事件不能分别由两个View同时处理。

# 4. Android中有事件传递能力的类：

* Activity
> 其中有方法： dispatchTouchEvent、onTouchEvent

* ViewGroup
> 其中有方法： dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent

* View
> 其中有方法：dispatchTouchEvent、onTouchEvent


# 5. 事件分发源码解析

有了上面的准备知识，现在我们来看看事件分发的源码。总的来说，事件分发包括如下几个过程：

* Activity对事件的分发过程
> 其中又包括：Activity传递事件给Window的过程

* 顶级View对事件的分发过程
* View对事件的处理过程

## 5.1 Activity对事件的分发过程

当一个点击事件发生时，事件最先传递给当前Activity，由Activity的dispatchTouchEvent来进行事件分发，具体工作由Activity内部的Window完成的。Window会将事件传递给DecorView，DecorView就是当前界面的底层容器（通过setContentView所设置的View的父容器），通过Activity.getWindow().getDecorView()可以获得。先看看Activity的`dispatchTouchEvent`方法：

```
public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```

从上面的代码可以看出：

> **Activity将事件传递给Window的过程**：事件先交给了Activity所附属的Window进行分发，如果返回true，整个事件循环就结束了；返回false意味着事件没人处理，所有View的onTouchEvent都返回了false，此时Activity的onTouchEvent方法就会被调用。

接着看看Window是如何将事件传递给DecorView（也就是ViewGroup）的，因为Window的直接实现类是PhoneWindow,所以直接去看PhoneWindow的源码：

```
@Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }
```
到这里就清晰了：
> PhoneWindow中的superDispatchTouchEvent方法中调用mDecor的superDispatchTouchEvent方法，完成了事件向顶级View的传递过程。

小结：

1. 在Activity的dispatchTouchEvent方法中，将事件传递给了PhoneWindow。
2. 在PhoneWindow的superDispatchTouchEvent方法中，将事件传递给了顶级View——DecorView。

因为DecorView继承自FrameLayout，其本质也就是一个ViewGroup。接下来看顶级View对事件的分发过程。

## 5.2 ViewGroup对事件分发过程

ViewGroup对事件的分发过程大致是这样的：

* 如果顶级ViewGroup拦截事件即onInterceptTouchEvent返回true，则事件由ViewGroup处理（//TODO: ViewGroup对事件的处理，之后再说）
* 如果顶级ViewGroup不拦截事件，则事件会传递给它的子View，这是子View的dispatchTouchEvent会被调用。

接下来看ViewGroup对点击事件的分发过程，主要实现在ViewGroup的dispatchTouchEvent方法中。

下面的代码描述了：当前View是否拦截点击事件这个逻辑。

```
// Check for interception.
  final boolean intercepted;
  if (actionMasked == MotionEvent.ACTION_DOWN
          || mFirstTouchTarget != null) {
      final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
      if (!disallowIntercept) {
          intercepted = onInterceptTouchEvent(ev);
          ev.setAction(action); // restore action in case it was changed
      } else {
          intercepted = false;
      }
  } else {
      // There are no touch targets and this action is not an initial down
      // so this view group continues to intercept touches.
      intercepted = true;
  }
```
从上面我们可以看出：ViewGroup会在两种情况下判断是否要拦截当前事件：事件类型为ACTION_DOWN和mFirstTouchTarget != null.

ACTION_DOWN好理解，那mFirstTouchTarget != null是什么情况？从后面的代码逻辑可以看出，当事件由ViewGroup的子元素成功处理时，mFirstTouchTarget会被赋值并指向子元素，也就是说，当ViewGroup不拦截事件并将事件交给子元素处理时mFirstTouchTouchTarget ！=null。反过来，一旦事件交给ViewGroup拦截，mFirstTouchTarget！=null就不成立。那么当ACTION_MOVE和ACTION_UP事件到来时，由于（actionMasked == MotionEvent.ACTION_DOWN
          || mFirstTouchTarget != null）为false，将导致ViewGroup的onInterceptTouchEvent不会再被调用，并且同一序列中的其他事件都会默认交给它处理。
          
### 5.2.1 当ViewGroup拦截事件的时候
当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件，当ViewGroup决定拦截事件后，那么后续的点击事件就会默认交给它处理并且不再调用它的onInterceptTouchEvent方法。

### 5.2.2 当ViewGroup不拦截事件的时候      

当ViewGroup不拦截事件时，事件会向下分发交给它的子View进行处理。

```
final View[] children = mChildren;
for (int i = childrenCount - 1; i >= 0; i--) {
final int childIndex = getAndVerifyPreorderedIndex(
        childrenCount, i, customOrder);
final View child = getAndVerifyPreorderedView(
        preorderedList, children, childIndex);

// If there is a view that has accessibility focus we want it
// to get the event first and if not handled we will perform a
// normal dispatch. We may do a double iteration but this is
// safer given the timeframe.
if (childWithAccessibilityFocus != null) {
    if (childWithAccessibilityFocus != child) {
        continue;
    }
    childWithAccessibilityFocus = null;
    i = childrenCount - 1;
}

if (!canViewReceivePointerEvents(child)
        || !isTransformedTouchPointInView(x, y, child, null)) {
    ev.setTargetAccessibilityFocus(false);
    continue;
}

newTouchTarget = getTouchTarget(child);
if (newTouchTarget != null) {
    // Child is already receiving touch within its bounds.
    // Give it the new pointer in addition to the ones it is handling.
    newTouchTarget.pointerIdBits |= idBitsToAssign;
    break;
}

resetCancelNextUpFlag(child);
if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
    // Child wants to receive touch within its bounds.
    mLastTouchDownTime = ev.getDownTime();
    if (preorderedList != null) {
        // childIndex points into presorted list, find original index
        for (int j = 0; j < childrenCount; j++) {
            if (children[childIndex] == mChildren[j]) {
                mLastTouchDownIndex = j;
                break;
            }
        }
    } else {
        mLastTouchDownIndex = childIndex;
    }
    mLastTouchDownX = ev.getX();
    mLastTouchDownY = ev.getY();
    newTouchTarget = addTouchTarget(child, idBitsToAssign);
    alreadyDispatchedToNewTouchTarget = true;
    break;
}

// The accessibility focus didn't handle the event, so clear
// the flag and do a normal dispatch to all children.
ev.setTargetAccessibilityFocus(false);
}
```

首先遍历ViewGroup的所有子元素，然后判断子元素是否能够接收点击事件。是否能够接收点击事件主要由两点来衡量：
1. 子元素是否在播放动画；
2. 点击事件的坐标是否落在子元素的区域内。
如果子元素满足这两个条件，那么事件就会传递给它来处理。可以看到，dispatchTransformedTouchEvent实际上调用的就是子元素的dispatchTouchEvent方法：

```
if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
    event.setAction(MotionEvent.ACTION_CANCEL);
    if (child == null) {
        handled = super.dispatchTouchEvent(event);
    } else {
        handled = child.dispatchTouchEvent(event);
    }
    event.setAction(oldAction);
    return handled;
}
```
如果child传递的不是Null，那么就会直接调用子元素的dispatchTouchEvent方法，这样事件就交由子元素处理了，从而完成了一轮事件分发。

如果子元素的dispatchTouchEvent返回true，那么mFirstTouchTarget就会被赋值同时跳出for循环：

```
newTouchTarget = addTouchTarget(child, idBitsToAssign);
alreadyDispatchedToNewTouchTarget = true;
break;
```
上面几行代码完成了mFirstTouchTarget的赋值并终止对子元素的遍历。如果子元素的dispatchTouchEvent返回false，ViewGroup就会把事件分发给下一个元素（如果还有下一个元素的话）。

对mFirstTouchTarget的赋值过程在方法addTouchTarget中，上代码：

```
/**
 * Adds a touch target for specified child to the beginning of the list.
 * Assumes the target child is not already present.
 */
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;
    return target;
}
```
从addTouchTarget方法的内部结构可以看出，mFirstTouchTarget其实是一种单链表结构。mFirstTouchTarget是否被赋值，将直接影响到ViewGroup对事件的拦截策略。如果mFirstTouchTarget = null，那么ViewGroup就默认拦截接下来同一序列中所有的点击事件。

如果遍历所有的子元素后事件都没有被合适的处理，这包含两种情况：
1. ViewGroup没有子元素；
2. 子元素处理了点击事件，但是在dispatchTouchEvent返回了false，这一般是因为子元素在onTouchEvent中返回了false。
这两种情况下ViewGroup会自己处理点击事件。代码如下：

```
// Dispatch to touch targets.
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
            TouchTarget.ALL_POINTER_IDS);
} 
```
上面代码第三个参数null，它会调用super.dispatchTouchEvent(event)，这就转到了View的dispatchTouchEvent方法，即点击事件传递到了View中。

## 5.3 View对点击事件的处理过程

先看View的dispatchTouchEvent代码：

```
	/**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.
     *
     * @param event The motion event to be dispatched.
     * @return True if the event was handled by the view, false otherwise.
     */
    public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
```

因为View没有子元素所以无法向下传递事件，只能自己处理事件。

从源码可以看出View对点击事件的处理过程，首先会判断有没有设置OnTouchListener，如果OnTouchListener中的onTouch方法返回true，那么onTouchEvent就不会被调用。可见OnTouchListener的优先级高于onTouchEvent。

接着再分析onTouchEvent的实现。先看当View处于不可用状态下点击事件的处理过程。不可用状态下View照样会消耗点击事件，代码如下：

```

if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        // A disabled view that is clickable still consumes the touch
        // events, it just doesn't respond to them.
        return (((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
    }LICKABLE) == CONTEXT_CLICKABLE);
}
```
接下来看看onTouchEvent中对点击事件的具体处理过程：

```
if (((viewFlags & CLICKABLE) == CLICKABLE ||
        (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
        (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
    switch (action) {
        case MotionEvent.ACTION_UP:
            boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
            if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                // take focus if we don't have it already and we should in
                // touch mode.
                boolean focusTaken = false;
                if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                    focusTaken = requestFocus();
                }

                if (prepressed) {
                    // The button is being released before we actually
                    // showed it as pressed.  Make it show the pressed
                    // state now (before scheduling the click) to ensure
                    // the user sees it.
                    setPressed(true, x, y);
               }

                if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                    // This is a tap, so remove the longpress check
                    removeLongPressCallback();

                    // Only perform take click actions if we were in the pressed state
                    if (!focusTaken) {
                        // Use a Runnable and post this rather than calling
                        // performClick directly. This lets other visual state
                        // of the view update before click actions start.
                        if (mPerformClick == null) {
                            mPerformClick = new PerformClick();
                        }
                        if (!post(mPerformClick)) {
                            performClick();
                        }
                    }
                }

                if (mUnsetPressedState == null) {
                    mUnsetPressedState = new UnsetPressedState();
                }

                if (prepressed) {
                    postDelayed(mUnsetPressedState,
                            ViewConfiguration.getPressedStateDuration());
                } else if (!post(mUnsetPressedState)) {
                    // If the post failed, unpress right now
                    mUnsetPressedState.run();
                }

                removeTapCallback();
            }
            mIgnoreNextUpEvent = false;
            break;

        case MotionEvent.ACTION_DOWN:
            mHasPerformedLongPress = false;

            if (performButtonActionOnTouchDown(event)) {
                break;
            }

            // Walk up the hierarchy to determine if we're inside a scrolling container.
            boolean isInScrollingContainer = isInScrollingContainer();

            // For views inside a scrolling container, delay the pressed feedback for
            // a short period in case this is a scroll.
            if (isInScrollingContainer) {
                mPrivateFlags |= PFLAG_PREPRESSED;
                if (mPendingCheckForTap == null) {
                    mPendingCheckForTap = new CheckForTap();
                }
                mPendingCheckForTap.x = event.getX();
                mPendingCheckForTap.y = event.getY();
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
            } else {
                // Not inside a scrolling container, so show the feedback right away
                setPressed(true, x, y);
                checkForLongClick(0, x, y);
            }
            break;

        case MotionEvent.ACTION_CANCEL:
            setPressed(false);
            removeTapCallback();
            removeLongPressCallback();
            mInContextButtonPress = false;
            mHasPerformedLongPress = false;
            mIgnoreNextUpEvent = false;
            break;

        case MotionEvent.ACTION_MOVE:
            drawableHotspotChanged(x, y);

            // Be lenient about moving outside of buttons
            if (!pointInView(x, y, mTouchSlop)) {
                // Outside button
                removeTapCallback();
                if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                    // Remove any future long press/tap checks
                    removeLongPressCallback();

                    setPressed(false);
                }
            }
            break;
    }

    return true;
}
```

从上面的代码可以看到，只要View的CLICKABLE和LONG_CLICKABLE有一个为true，那么它就会消耗这个事件，即onTouchEvent方法返回true，不管它是不是DISABLE状态。然后就是当ACTION_UP事件发生时，会触发performClick方法，如果View设置了OnClicklistener，那么performClick方法内部会调用它的onClick方法，如下所示：

```
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }

    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}·		
```

View的LONG_CLICKABLE属性默认为false，而CLICKABLE属性是否为false和具体的View有关，确切来说是：
1. 可点击的View的CLICKABLE为true；
2. 不可点击的View的CLICKABLE为false。
通过setClickable和setLongClickable可以分别改变View的CLICKABLE和LONG_CLICKABLE属性。另外，setOnClickListener会自动将View的CLICKABLE设为true，setOnLongClickListener会自动将View的LONG_CLICKABLE设为true，这一点从源码可以看出：

```
 public void setOnClickListener(@Nullable OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        getListenerInfo().mOnClickListener = l;
 }
    
public void setOnLongClickListener(@Nullable OnLongClickListener l) {
        if (!isLongClickable()) {
            setLongClickable(true);
        }
        getListenerInfo().mOnLongClickListener = l;
}
```

至此，View的事件分发机制源码分析完毕！

