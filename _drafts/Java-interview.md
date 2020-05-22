---
layout: post
title: 面试
categories: interview
description: 面试总结
keywords: keyword1, keyword2
---

面试题目总结

# Java中++i 是线程安全的吗?为什么不安全,如何使其线程安全?

答:

如果是方法里定义的,一定是线程安全的,因为每个方法栈是线程私有的.

JVM的栈是线程私有的,所以每个栈帧上定义的局部变量也是线程私有的,意味着是线程安全的。

如果是类的成员变量,++i则不是线程安全的,因为++i会被编译成几句字节码语句执行(读值，+1，写值)，不是原子操作,因此不是线程安全的.

volatile不能解决这个线程安全问题.因为volatile只能保证可见性,不能保证原子性.

一.  可以使用synchronized或者ReentrantLock来解决这个问题.我认为一般使用synchronized更好,因为JVM团队一直以来都在优先改进这个机制,可以尽早获得更好的性能,并且synchronized对大多数开发人员来说更加熟悉,方便代码的阅读.

二.  还可以使用java.util.concurrent.AtomicInteger这个类,它提供了线程安全且高效的原子操作，是线程安全的

AtomicInteger类的底层实现原理是利用处理器的CAS操作（Compare And Swap，比较与交换，一种有名的无锁算法）来检测栈中的值是否被其他线程改变，如果被改变则CAS操作失败。这种实现方法在CPU指令级别实现了原子操作，因此，其比使用synchronized来实现同步效率更高。