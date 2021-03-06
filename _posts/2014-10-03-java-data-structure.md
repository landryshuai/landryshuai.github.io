---
layout: post
title: "java data structure"
description: ""
keywords: ""
category: [work]
tags: [java]
---
程序都需要存储数据，谈谈我对java中容器的理解：

一提到容器，我们很容易想到数组和列表。java中的数组和C语言里面的数组，是差不多的。分配的就是连续的一段内存地址。数组很有特点：一旦分配，数组大小就不能改变了，换来了好的优点是，访问速度快。

很显然，你在程序中，有时候很难预先知道你想存在的数据有多少，最大能有多大。这个时候，有两种解决方案：

 1. **用链表。链表的每一个元素节点都有指向下一个元素的指针。**这样一来，就不需要在程序一开始就确定数据的多少了。在C语言里面，就是这么做的。而在JAVA中，LinkedList就是采用了这种方式。
 2. **我们可以在新建一个数组的时候，给他一个默认的大小。**一旦发现数据量超过数组大小的时候，我们新建一个新的容量更大的数组，把原来的数据全拷贝到新的数组里。 其实，在JAVA中，ArrayList其实就是这么实现的。

在C语言中，列表其实就是链表的意思。而在JAVA中，你可以看到：它指的是继承了List接口的类。上面说到的LinkedList和ArrayList其实都是List。但无论是在C还是在JAVA里，一个很明显的共同点是：他们的大小都是不固定的。可以存任意大小的数据，除非程序的内存不够。

但是LinkedList和ArrayList也有区别：LinkedList由双向链表实现，而ArrayList由数组实现。这就决定了：*随机访问的时候，ArrayList更快；而中间插入或删除LinkedList效率更高。*

现在我们来想想：对于List里的元素来说，我们并没有什么其它具体要求，基本上只要是同类型的元素就OK。那如果我们要求元素唯一，或者元素插入的顺序和取出的顺序一样的？当然，我们可以自己通过List，实现出这类类型的List. 只是在插入的时候，注意检查插入元素是否已经存在相同的了，或者注意存元素和取元素的顺序就行。

JAVA是强大的（基本可以达成共识，我们所说的语言：一般指的是语法以及库文件）。在java中，已经定义了一个比较普遍的接口：Collection。List，Set（不能有重复的元素），Queue（按照排队规则来）都是Collection。

撇开Queue不谈，Set这个不能有重复元素的特性，确实在程序中，经常会碰到。在JAVA中，主要提供了

 1. HashSet（便于快速定位元素，比较两元素是否相等－－－具体的采用方式，请参照我上一篇关于hashCode的文章），元素在内部是无序的。
 2. TreeSet（是一种SortedSet，便于比较大小，采用的是红黑树？），元素按照一定的顺序保存的。
 3. LinkedHashSet：它按照添加的顺序保存对象。

就上面的数据结构，譬如说数组和列表，我们想要访问里面的某一个元素，我们必须事先知道这个数据的索引。也就是说，这些数据都是根据数字进行索引的。那么有没有一个更普遍的索引方法。我们根据一个对象去访问另外一个对象？也就是说，这两个数据是关联在一起的。在JAVA中，提供了Map这个数据结构，我们也可以称之为关联数组或者字典。用去索引的对象我们称之为Key，获得的数据我们称之为Value。

显然，这个时候，我们不希望一个Key对应多个value。
在JAVA中，Map主要有以下实现：

 1. HashMap：基于散列表的实现。提供最快的查找顺序。
 2. TreeMap：基于红黑树的实现。它是按照比较结果的升序保存键。
 3. LinkedHashMap：按照插入顺序保存键，同时保留了HashMap的查询速度。
 4. ConcurrentHashMap：提供线程安全的HashMap（上面三个都不是线程安全的），但不涉及同步加锁。速度快？
 
由于Map保证Key唯一，也就是说，所有Key的集合应该是一个Set。

*其实HashSet内部维护的就是一个HashMap的，每一个Key是要存的对象。Value是一个定值Object。*

被存入HashMap的元素要重写，hashcode和equals方法，而被存入TreeMap的元素要要实现Comparable。