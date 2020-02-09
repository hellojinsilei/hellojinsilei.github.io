---
layout:     post                    # 使用的布局（不需要改）
title:      synchronized和ReentrantLock的区别            # 标题 
subtitle:   synchronized和ReentrantLock的区别            #副标题
date:       2020-01-19              # 时间
author:     ListJ                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 锁
    - java
    - 面试
---

## 常规区别

 - `synchronized` 是 Java 中的关键字，而`ReentrantLock`是一个类，继承自父类Lock
 - `ReentrantLock`更加灵活，提供了超时获取锁，可中断锁。提供了公平锁和非公平锁，而`synchronized`仅仅是非公平锁。
   
 - 用法上，`ReentrantLock`必须手动释放锁，并且只能修饰代码块。而`synchronized`不用手动释放锁，除此之外可以修饰方法。

