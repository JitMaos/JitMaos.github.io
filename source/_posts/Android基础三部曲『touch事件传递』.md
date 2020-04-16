---
title: Android基础三部曲『Touch事件传递』
date: 2020-02-18 18:56:06
tags:
---

touch事件传递，这里有三个关键的方法，分别是`dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent`，其中**View和Activity没有onInterceptTouchEvent，只有ViewGroup三个方法都有**。调用过程是：dispatch(分发)->intercept(是否拦截)->ontouch(处理)。事件由Activity经ViewGroup传递到View，如果一路事件都没有被消费掉，那么事件又会由View经ViewGroup传递到Activity直到被处理掉。**View的dispatchTouchEvent是将事件分发给了自己。**

二、重写onInterceptTouchEvent返回true(默认false)，则后续事件不会往下dispatch，子控件的diaptchTouchEvent无法接受touch事件；如果此时没有重写onTouchEvent，使之返回true，那么ACTION_DOWN将有父控件中onTouchEvent返回true的父控件处理，或者一直传递到Activity，由Activity处理；需要特别注意的是：假设A处理了ACTION_DOWN事件，则后续的ACTION_MOVE、ACTION_UP事件也由A处理！！

三、dispatchTouchEvent的调用逻辑，如果是ViewGroup，则先调用自己的onInterceptTouchEvent,如果是true,则直接返回；如果是false，则调用子View/ViewGroup的dispatchTouchEvent事件，如果子组件没有消费掉事件，则返回到本组件的onTouchEvent，则返回onTouchEvent的返回值！！

四、只重写onTouchEvent，使之返回true，表示消费了该touch事件，需要特别注意的是：假设ViewGroup A处理了ACTION_DOWN事件，不光是后续的ACTION_MOVE、ACTION_UP事件也由A处理，甚至子View的dispatchTouchEvent都不会调用，因为此时本View的dispatchTouchEvent返回true，表示事件已经被handled

五、不管是重写onTouchEvent返回true，还是重写onIterceptTouchEvent返回true，都说明事件已经不需要往子view传递了，此时子view的dispatchTouchEvent将无法收到事件。如果onIterceptTouchEvent返回true,而onTouchEvent没有重写，则事件会继续往父控件传递，直到某个父控件消费了事件，或者传递到Activity，由Activity消费掉。

六、在onTouchEvent中返回true，消费了事件，则事件不会再往上级组件传递；但是依然会调用父控件、Activity的dispatchTouchEvent事件，也就是说父控件还是有机会拦截事件的。

七、在dispatchTouchEvent中返回true(默认返回false)，表示该事件已经被handled，不再继续分发；这样事件不会向传递给父控件处理，不管有没重写本组件的onTouchView消费该事件。



```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    ...
    boolean handled = false;
    if (onFilterTouchEventForSecurity(ev)) {
        final int action = ev.getAction();
        final int actionMasked = action & MotionEvent.ACTION_MASK;
        ...
        // Check for interception.
        final boolean intercepted;
        //ViewGroup去判断这个事件该不该去拦截，首先是这个事件得是ACTION_DOWN事件或者该事件的mFirstTouchTarget（目标子View）是不为空的才会考虑要不要拦截。***这说明mFirstTouchTarget为空的情况下，ACTION_MOVE和ACTION_UP事件是不会经过这个拦截判断的，而是直接intercepted = true表示事件被直接拦截掉。
//      那么mFirstTouchTarget（目标子View）这个变量到底是什么呢？什么时候会为空呢？
        if (actionMasked == MotionEvent.ACTION_DOWN
                || mFirstTouchTarget != null) {
            //这个变量是控制是不是不允许父控件去拦截该事件的，取值是看"mGroupFlags"的取值，这里面涉及到一个方法：requestDisallowInterceptTouchEvent()方法。这个方法确定"mGroupFlags"的取值，控制请求父布局不拦截该事件，而是交给自己去做处理。这个方法在处理滑动冲突等场景时经常用到。
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
        ...
            if (!canceled && !intercepted) {

                // If the event is targeting accessibility focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
                ...
            }
            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                ...

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```

#### 默认情况

![Touch默认流程.png](https://upload-images.jianshu.io/upload_images/7186484-19bd3ac2770676b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### dispatchTouchEvent返回值

`dispatchTouchEvent()中调用了onInterceptTouchEvent()和dispatchTransformedTouchEvent()方法，dispatchTransformedTouchEvent()调用了super.dispatchTouchEvent()方法`，所以如果重写dispatchTouchEvent方法

+ 返回true
![dispatchTouchEvent返回true.png](https://upload-images.jianshu.io/upload_images/7186484-5dd5920e40b17322.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ 返回false
![dispatchTouchEvent返回false.png](https://upload-images.jianshu.io/upload_images/7186484-7b303e08235b2003.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### onInterceptTouchEvent返回值

+ 返回true
![onInterceptTouchEvent返回true.png](https://upload-images.jianshu.io/upload_images/7186484-ef4951c9034fc86a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 返回false
  onInterceptTouchEvent和onTouchEvent如果返回false的话， 效果和默认的一样

  

#### onTouchEvent返回值

+ 返回true
![onTouch返回true.png](https://upload-images.jianshu.io/upload_images/7186484-6ec1d110a654160d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 返回false
![onTouch返回false.png](https://upload-images.jianshu.io/upload_images/7186484-d0c296fdd731ac2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



参考：

[一步步探索学习Android Touch事件分发传递机制（一）](https://www.jianshu.com/p/c98a41a9a5f3)