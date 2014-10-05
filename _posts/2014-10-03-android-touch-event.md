---
layout: post
title: "Android Touch Event"
description: ""
keywords: ""
category: [work]
tags: [Android, java]
---
# Android Touch事件

在 View 中跟 Touch 相关的传递事件的方法有 dispatchTouchEvent , interceptTouchEvent，onTouchEvent 三个。（PS：View还有一个setTouchListener的方法）。
ViewGroup.java的dispatchTouchEvent逻辑如下：一般我们也不会去覆盖它.   

1. TouchEvent 事件从 activity 传递出来之后，最先到达的就是最顶层 view 的 dispatchTouchEvent ，然后它(即dipatchTouchEvent方法)进行分发. 这个 view 的 onInterceptTouchEvent 方法来决定是否要拦截这个事件：如果拦截，事件不再流转到其子View，而是流转到自己的onTouchEvent上；如果不拦截，则往子View传递。   
2.  在ViewGroup里，如果本身的onIterceptTouchEvent返回为false，也就是不拦截的时候，ViewGroup就会把事件分配给他的子View们的dispatchTouchEvent，去处理，如果一个子View的dispatchTouchEvent返回为True，那其他子View的dispatchTouchEvent则没有机会执行。

总结：  
1. onInterceptTouchEvent 返回的真假，只是表明，是由自己的onTouchEvent来处理，还是由子View来处理。  
2. dispatchTouchEvent返回true，表明不会让他的兄弟有机会去走到dispatchTouchEvent（有可能两兄弟是相互覆盖的）。  
3. 另外，如果一个View自定义了onTouchEvent。那它的onClick事件需要自己去触发，否则无用，因为onClick事件，就是需要onTouchEvent去模拟出来的。  
4. onLongClick返回为true，表明onClick事件不会触发。返回false,onclick事件可以被出发。


最后关于复杂的onInterceptTouchEvent官方描述：  
{% highlight java %}
Implement this method to intercept all touch screen motion events. This allows you to watch events as they are dispatched to your children, and take ownership of the current gesture at any point.
Using this function takes some care, as it has a fairly complicated interaction with View.onTouchEvent(MotionEvent), and using it requires implementing that method as well as this one in the correct way. Events will be received in the following order:

  1. You will receive the down event here.
  2. The down event will be handled either by a child of this view group, or given to your own onTouchEvent() method to handle; this means you should implement onTouchEvent() to return true, so you will continue to see the rest of the gesture (instead of looking for a parent view to handle it). Also, by returning true from onTouchEvent(), you will not receive any following events in onInterceptTouchEvent() and all touch processing must happen in onTouchEvent() like normal.
  3. For as long as you return false from this function, each following event (up to and including the final up) will be delivered first here and then to the target's onTouchEvent().
  4. If you return true from here, you will not receive any following events: the target view will receive the same event but with the action MotionEvent.ACTION_CANCEL, and all further events will be delivered to your onTouchEvent() method and no longer appear here.

Overrides: onInterceptTouchEvent(...) in ViewGroup
Parameters:ev The motion event being dispatched down the hierarchy.Returns:Return true to steal motion events from the children and have them dispatched to this ViewGroup through onTouchEvent(). The current target will receive an ACTION_CANCEL event, and no further messages will be delivered here.
{% endhighlight %}
