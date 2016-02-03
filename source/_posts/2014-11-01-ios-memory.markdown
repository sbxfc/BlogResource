---
layout: post
title: "iOS - 内存管理"
date: 2014-11-01 20:05:00 +0800
comments: true
categories: 
---

首先应当留心函数 imageNamed: 加载图片不释放内存的问题,imageNamed会将图片加载内存里方便下次能快速加载,但是对于一些不常用的图片这种方式是不必要的。

另外,也要留心使用多线程、Timer时,引用对象不释放的问题,这种情况会造成一定的内存和CPU问题。

配合Instruments查看内存的使用情况。



