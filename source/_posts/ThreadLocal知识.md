---
title: ThreadLocal知识
date: 2020-03-23 23:27:16
tags:
---

通过当前thread作为键获取一个ThreadLocalMap，

这个ThreadLocalMap属于当前线程，当前线程对他做的所有操作其他线程都不可见，相当于线程执行栈帧中的局部变量，但他却没有自动的生命周期，使用起来非常需要注意内存泄漏问题
