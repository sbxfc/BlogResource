---
layout: post
title: "iOS - 属性修饰符"
date: 2014-02-25 13:36:18 +0800
comments: true
categories: 
---

#assign，retain和copy

没有ARC之前,使用assign，retain，copy来修饰属性。

1）assign,主要用于数值类变量,即标量,直接赋值即可,不涉及引用计数的变化（标量值,也没有引用技术可以供管理）；

2）copy 内容拷贝,对于NSString来说,一个内容为"STR"指针是**0x111111**的字符串str1,赋值给copy修饰的字符串str2之后,str1不变,str2的内容变为"STR",指针变为**0x222222**;

3）retain,指针拷贝。obj1赋值给retain修饰的对象obj2之后,obj1和obj2指向相同的内存对象。对象的引用计数 +1;

#strong和weak

有了ARC后使用weak或strong来说明属性是弱引用还是强引用;