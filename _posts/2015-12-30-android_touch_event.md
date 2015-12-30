---
layout: post
title: "android touch event理解"
description: ""
keywords: ""
category: [java]
tags: [java]
---
###理解TouchEvent事件

####默认情况下：路径如下：此时LinearLayout:onInterceptTouchEvent返回为false， 先走TextView的onTouchEvent再走ViewGroup的

#####当TextView的onTouchEvent为false的时候，如下：

```
I LinearLayout::dispatchTouchEvent::action=ACTION_DOWN
I LinearLayout::onInterceptTouchEvent::action=ACTION_DOWN
I EventTextView::dispatchTouchEvent::action=ACTION_DOWN
I EventTextView::onTouchEvent::action=ACTION_DOWN
I LinearLayout::onTouchEvent::action=ACTION_DOWN
I Activity:onTouchEvent::action=ACTION_DOWN
I Activity:onTouchEvent::action=ACTION_MOVE
I Activity:onTouchEvent::action=ACTION_MOVE
I Activity:onTouchEvent::action=ACTION_MOVE
I Activity:onTouchEvent::action=ACTION_MOVE
I Activity:onTouchEvent::action=ACTION_MOVE
System.out I Activity:onTouchEvent::action=ACTION_MOVE
I Activity:onTouchEvent::action=ACTION_UP
```
#####当TextView的onTouchEvent为true的时候，如下：
```
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_DOWN
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_DOWN
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_DOWN
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_DOWN
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_MOVE
I/System.out(16085): LinearLayout::dispatchTouchEvent::action=ACTION_UP
I/System.out(16085): LinearLayout::onInterceptTouchEvent::action=ACTION_UP
I/System.out(16085): EventTextView::dispatchTouchEvent::action=ACTION_UP
I/System.out(16085): EventTextView::onTouchEvent::action=ACTION_UP
```
####当此时LinearLayout:onInterceptTouchEvent返回为true, 就没子view什么事情了。直接走ViewGroup的onTouchEvent

#####ViewGroup的onTouchEvent为false的时候，如下
```
I/System.out(13814): LinearLayout::dispatchTouchEvent::action=ACTION_DOWN
I/System.out(13814): LinearLayout::onInterceptTouchEvent::action=ACTION_DOWN
I/System.out(13814): LinearLayout::onTouchEvent::action=ACTION_DOWN
I/System.out(13814): Activity:onTouchEvent::action=ACTION_DOWN
I/System.out(13814): Activity:onTouchEvent::action=Unknow:id=261
I/System.out(13814): Activity:onTouchEvent::action=Unknow:id=517
I/System.out(13814): Activity:onTouchEvent::action=Unknow:id=6
I/System.out(13814): Activity:onTouchEvent::action=ACTION_MOVE
I/System.out(13814): Activity:onTouchEvent::action=Unknow:id=262
I/System.out(13814): Activity:onTouchEvent::action=ACTION_MOVE
I/System.out(13814): Activity:onTouchEvent::action=ACTION_MOVE
I/System.out(13814): Activity:onTouchEvent::action=ACTION_MOVE
I/System.out(13814): Activity:onTouchEvent::action=ACTION_MOVE
I/System.out(13814): Activity:onTouchEvent::action=ACTION_UP
```
#####ViewGroup的onTouchEvent为true的时候，如下直接走ViewGroup的onTouchEvent,甚至连ViewGroup本身的onInterceptTouchEvent也不走了。

```
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_DOWN
I/System.out(21195): LinearLayout::onInterceptTouchEvent::action=ACTION_DOWN
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_DOWN
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=Unknow:id=261
I/System.out(21195): LinearLayout::onTouchEvent::action=Unknow:id=261
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=Unknow:id=517
I/System.out(21195): LinearLayout::onTouchEvent::action=Unknow:id=517
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=Unknow:id=6
I/System.out(21195): LinearLayout::onTouchEvent::action=Unknow:id=6
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=Unknow:id=6
I/System.out(21195): LinearLayout::onTouchEvent::action=Unknow:id=6
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_MOVE
I/System.out(21195): LinearLayout::dispatchTouchEvent::action=ACTION_UP
I/System.out(21195): LinearLayout::onTouchEvent::action=ACTION_UP
```
####总结

其实就我个人理解而言。这套系统的目标其实是这样的。一个touch事件该由谁来消费的问题。touch事件而且只能由onTouchEvent来表示有没有消费。

默认dispatchTouchEvent的情况下：

 * 首先，touch事件，一层一层，从最外层，往里层传递。
 * 到了每一层，每一层的父view都有机会通过onInterceptTouchEvent 返回true，来表示他不希望他的子view接触到事件；通过 onTouchEvent 返回true来表示他消费了事件（注意父view，可以截获事件，可以不消费事件，也就是防止子view接触到事件）
 * 只有有一个view表示他想在ACTION_DOWN的时候，通过onInterceptTouchEvent 来返回true来表示他感兴趣的话。以后到了这一层，直接走其onTouchEvent，连onInterceptTouchEvent 也不会走。
 * 并且，只要一个view，即使onInterceptTouchEvent为false，只要onTouchEvent返回为true。其实也是表示感兴趣。以后直接走其onTouchEvent，连onInterceptTouchEvent 也不会走。
 * 如果一个子view的消费Touch事件（即onTouchEvent返回true），那他的父view的onTouchEvent就无法获取该事件了。

从而达到整个view树中，只有一个view消费了onTouchEvent事件（除了两个view同一层，互相覆盖）。

这么一个效果，都是通过dispatchTouchEvent这个方法来做到的。那么这个效果，要怎么代码实现呢？具体其实就是看ViewGroup的dispatchTouchEvent的方法。

另外还有一个问题。就是其实viewGroup的dispatchTouchEvent起到的作用，我们已经知道了。那么其实view本身就有个dispatchTouchEvent。他是起到什么作用呢？点击啊，双击啊，都是通过onTouchEvent来处理的啊。

```java
ListenerInfo li = mListenerInfo;
if (li != null && li.mOnTouchListener != null
&& (mViewFlags & ENABLED_MASK) == ENABLED
&& li.mOnTouchListener.onTouch(this, event)) {
result = true;
}

if (!result && onTouchEvent(event)) {
result = true;
}
```

已经清楚的说明到，他会先走setOnTouchEventListener，最后才走onTouchEvent事件的。也很显然的猜测到viewgroup onInterceptTouchEvent为true的时候，后面肯定是要调用到viewgroup的touchEvent的，通过什么方法呢？必然是view的dispatchTouchEvent。

可以自己好好考虑下，要是你，你会怎么实现ViewGroup的dispatchTouchEvent的方法。