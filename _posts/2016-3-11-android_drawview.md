---
layout: post
title: "关于滑动View"
description: ""
keywords: ""
category: [android,view]
tags: [android,view]
---
###Android 滑动View的原理

####滚动是View本身的属性

View的窗口和内容的区别。View窗口显示的是view的内容的mScrollX、mScrollY。

####View是如何更新视图的

* draw方法会根据mScrollX、mScrollY去绘制。
* 在绘制之前，会调用computeScroll方法。

####关于scrollTo、scrollBy

* scrollTo、scrollBy的方法的实质，就是改变mScrollX和mScrollY.

####关于computeScroll的作用

* scrollTo、scrollBy是直接滑动目标位置的，这对用户体验不好。
* computeScroll就提供了滑动的过程中，再次滑动的机会。

####关于Scroller类

* 作为辅助类，提供更好的交互优化。其实它更像一个Util方法。
* startScroll方法和fling方法，只是记载开始滑动的时间。
* 然后在computeScroll里调用scroller的computeScrollOffset方法来计算应该滑动的距离。然后方便开发者调用scrollTo、scrollBy方法, 后刷新view。这样就会再次调用computeScroll实现循环渐进滑动
