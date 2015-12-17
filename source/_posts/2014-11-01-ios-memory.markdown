---
layout: post
title: "iOS - 内存优化*"
date: 2014-11-01 20:05:00 +0800
comments: true
categories: 
---

对于一般应用程序,应该留心`imageNamed:`加载图片时不释放内存的问题。同时,要注意使用多线程、Timer时,引用对象不释放的问题,这种情况会造成一定的内存和CPU问题。

配合Instruments查看实时内存的使用情况。



