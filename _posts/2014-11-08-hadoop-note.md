---
layout: post
title: "Hadoop note"
description: ""
keywords: ""
category: [hadoop]
tags: [hadoop]
---

hadoop笔记
==========
###mapper reducer的由来。
计算文本单词出现的个数，分开计算，分开合成。

###组件
 * nameNode。管理数据相关部分。
 * dataNode。管理slave里存放的数据。
 * secondaryNameNode。nameNode的备份。
 * jobNode。管理工作。与taskNode通讯。
 * taskNode。根据储存在同一机子上的数据，进行计算！并与jobNode通讯。心跳机制！

###基础
主从分布模式！
数据以key-value为基础，存在hdfs的文件系统上。保持冗余，数据分散存储！

###map reduce
 
 mapper。reducer。两步！
 
  * 将hdfs的key，value转换成分析字段k1, v1. 以及分析结果的键值对k2，v2；经过mapper后，就形成了list<k2,list(v2)>. 
  * reducer再根据mapper的结果，产生需要的结果！

###具体实现：
####文件系统
hdfs文件系统！和普通文件系统使用起来很像！
####hadoop数据类型。
value是writable，key是writableComparable。都必须支持序列化！

####主要的类
 * mapper。实现map方法！
 * reducer。实现reduce方法！
 * partitioner。将key映射！
 * combiner。优化合并。
 * inputformat。inputsplit与recordreader：提供文件分片（inputformat），将文件分片映射成键值对（recordreader）。
 * outputformat。

####对数据流的理解
输入、中间结果、输出的键值对对k1,v1;k2,v2;k3,v3.

   * inputformat分inputsplit
   * recordreader将inpitsplit分解成k1，v1
   * mapper将k1，v1转换成k2，v2
   * reducer将k2，v2转换成k3，v3。

####其它
 * streaming 可以用脚本语言而不是java库，编写简单，短小的mapreduce程序！
 * mapper reducer可以多层链接！