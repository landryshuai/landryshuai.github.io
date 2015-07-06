---
layout: post
title: "java同步"
description: ""
keywords: ""
category: [java]
tags: [java]
---
理解同步的概念，要区分互斥锁和条件变量。虽然，他们往往在一起使用。

#### 问题
建立三个线程，A线程打印10次A，B线程打印10次B,C线程打印10次C，要求线程同时运行，交替打印10次ABC。这个问题用Object的wait()，notify()就可以很方便的解决。
见http://blog.csdn.net/zyplus/article/details/6672775

#### 注意要点
 * 三个线程的启动顺序不确定
 * 条件判断的时候，注意使用while，而不是if，因为wait会有其他的情况被唤醒
 
其实上文提供的答案，是存在我所说的三个问题的。
下面是我写的代码：

#### 我的代码

```
package com.landry;

public class ThreadTest {
    private static Object object = new Object();
    private static Boolean printA = true;
    private static Boolean printB = false;
    private static Boolean printC = false;
    private static int count = 0;
    private static final int MAX = 10;
    public static void main(String... args) {
        new ThreadB().start();
        new ThreadA().start();
        new ThreadC().start();
    }
    public static class ThreadA extends Thread {

        @Override
        public void run() {
            System.out.println("Run A");
            synchronized (object) {
                while (count < 10) {
                    while (!printA) {
                        try {
                            object.notifyAll();
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println('A');
                    printA = false;
                    printB = true;
                    printC = false;
                }
            }
        }
    }
    public static class ThreadB extends Thread {

        @Override
        public void run() {
            System.out.println("Run B");
            synchronized (object) {
                while (count < 10) {
                    while (!printB) {
                        try {
                            object.notifyAll();
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println('B');
                    printA = false;
                    printB = false;
                    printC = true;
                }
            }
        }
    }
    public static class ThreadC extends Thread {

        @Override
        public void run() {
            System.out.println("Run C");
            synchronized (object) {
                while (count < 10) {
                    while (!printC) {
                        try {
                            object.notifyAll();
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println('C');
                    printA = true;
                    printB = false;
                    printC = false;
                    count = count + 1;
                }
            }
        }
    }
}
```
####存在的问题
会多打印一个A出来。

####注意理解的地方
 * notifyAll只是唤醒线程，wait操作，才会释放自己的锁。