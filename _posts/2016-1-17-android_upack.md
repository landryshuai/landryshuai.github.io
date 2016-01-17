---
layout: post
title: "Zjdroid使用心得"
description: ""
keywords: ""
category: [android,security]
tags: [android,security]
---
###ZjDroid使用心得

####ZjDroid基础

ZjDroid是由百度安全实验室，出的一款android脱壳工具，号称脱壳神器。他是基于基于Xposed Framewrok的动态逆向分析模块。更多的基础介绍请参照[作者文章][Zjdroid]. 由于某些不可知的原因，最初放在github的开源代码已经被删除。好在网上一些热心好友又在github上放出了当初备份的源码。我自己也整理了一个比较完整的一份(为了应对一些情况，自己做了一些修改)。[地址在这里][Zjdroid-shuai]

####ZjDroid操作步骤

1. ZjDroid作为Xposed的模块设置好（注意生成apk的时候依赖的xposed库不要编译进apk)
2. 运行dump_dexinfo命令。观察日志查看输出结果。
3. 利用上一步得到的dexpath,运行backsmali命令。最后得到脱壳后的dex文件

####ZjDroid原理

ZjDroid是基于Xposed的模块的。ZjDroid通过在应用加载的时候，注册一个BroadcastReceiver，注意这个BroadcastReceiver，是运行在目标dex运行的进程中。然后通过am命令来向这个receiver发送命令的。

每一个apk运行的代码都是在dex里的。而代码运行的时候，这些dex都是需要加载进入内存的。而ZjDroid就是利用hook住dex加载的函数。记住这些dex在内存中的信息的。然后dump_dexinfo也就可以拿到这些dex信息。然后backsmali通过dex在内容中的信息，读取在内容中的dex（通过JNI，也就是提供的so库），然后分析dex文件，最后解析出所有的smali文件的。

####ZjDroid使用的时候遇到的坑

1. 执行backsmali命令一定要adb shell进入手机上，再执行。否则receiver会接受不到命令。
2. 脱壳某数字公司加壳的时候，根据dump_dexinfo步骤得到的dexpath后，运行的到的backsmali时一直报cookie无效。有可能是加壳做了针对性的修改。考虑到backsmali命令是通过dexPath去找dex在java面的句柄cookie。为何不直接用观察到的cookie直接backsmali。所以我就改动了下代码增加了backsmali_cookie命令。最后利用新命令尝试了所有观察到的cookie后，终于成功。
3. 关于原作者说到的脱壳自家产品的，需要将apk改为jar的方案。我也按照他们的方案尝试了。最后还是目标程序无法运行。有可能百度对这种方案又做了修正。不过，我自己尝试出了一个新的方案。照常ZjDroid apk作为模块配置好后重启，这个时候，目标程序监测到后无法运行。我们把ZjDroid卸掉后，再重装，注意不要重启。这个时候目标程序就可以正常hook并脱壳了。猜测是百度加壳会监测xposed的模块。发现模块卸载后就可以启动了。其实Xposed模块在手机重启后，模块已经加载到内存中，并没有因为卸载而清除。这个时候其实ZjDroid就可以用了。重装上ZjDroid其实是为了提供运行backsmali需要的so库。


####ZjDroid存在的问题

1. backsmali貌似并不能把所有的代码都dump出来。原因我还没研究清楚。有可能是使用的了动态解密关键代码的功能。如果想进一步完整破解。可以参照现有的一份方案[Android应用程序通用自动脱壳方法研究][unpack] 和 [从Android运行时出发，打造我们的脱壳神器][unpack1]












[Zjdroid]:http://blog.csdn.net/zhangmiaoping23/article/details/46235941
[Zjdroid-shuai]:https://github.com/landryshuai/ZjDroid
[unpack]:http://drops.wooyun.org/tips/9214
[unpack1]:http://drops.wooyun.org/tips/9335