---
title: 内存溢出OutOfMemoryError
date: 2017-10-09 21:31:04
tags:
---

#### java.lang.OutOfMemoryError: PermGen space
java虚拟机（JVM）管理着类内存，堆和非堆。堆是给开发人员用的，在JVM启动时才会创建。非堆是JVM留给自己用的，用来放类的信息，非堆在运行生命周期内GC是不会主动释放空间的。发生这个错误的情况是我们开发的代码量很大或者用到的第三方jar包量比较大时，而Tomcat的MaxPermSize设置的不合理时，就会出现这个错。<!--more-->

#### java.lang.OutOfMemoryError: Java heap space
这个错误产生的主要原因是：
1.内存设置参数过小（Xms/Xmx,NewSize/MaxNewSize）
2.程序自身问题，Heap Space的默认空间（-Xms）是物理内存的1/64，最大空间（-Xmx）是物理内存的1/4，不过也可以通过人工进行设置。如果我们系统的内存剩余不到40%，JVM就会增大Xmx设置的值，内存剩余超过70%，JVM就会减少Xms设置的值。因此服务器的Xmx和Xms设置的值一般应该是相同的，这样可以有效避免每次GC（回收）后都要调整虚拟机堆的大小。需要注意的是设置的值不能超过物理内存或操作系统的最大限制，这样会导致起服务器无法启动。

#### java.lang.OutOfMemoryError: c heap space
系统对于C Heap没有任何限制，所以C Heap发生时，java进程所占用的内存会不断增长，直到死机，唯一的解决方法就是杀掉进程或重启计算机。

### 解决方案
1.尽早的释放无用对象的引用。好的方法是用临时变量，让引用变量在使用完成后自动设置为null
2.尽量不要使用String，而使用StringBuffer
3.尽可能少的使用静态变量，因为静态变量是全局性的，GC不会主动回收
4.不要集中创建对象，尤其是大对象
5.尽量使用对象池
6.不要在经常调用的方法中创建对象