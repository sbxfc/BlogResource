---
layout: post
title: "STL - deque容器"
date: 2015-03-13 17:57:18 +0800
comments: true
categories: 
---

deque是一个非常高效地在首尾插入或删除元素的容器。

和vector一样,deque是也一个连续内存容器。但它是由一个或多个内存块组成,每个内存块里的元素是内存连续的。deque的结构里有一个专门管理所有内存块的类似map的容器,map容器里每个元素对应一个内存块指针。

在执行前、后插入元素的操作中,如果当前内存块存储空间已满就会新开辟一块新的内存空间并把指针赋值给map。当删除某个元素时,如果该元素是其所在内存块里的最后一个元素,完成删除后会释放缓存区。整个操作过程中,无需将其他内存块里的元素重新序列话。

除了可以进行高效的首尾操作以外。deque还可以像vector一样使用\[ \]或at访问中间元素,但效率不及vector。

---
容器的选择 

vector是一种可以默认使用的序列类型,当很频繁地对序列中部进行插入和删除时应该用list,当大部分插入和删除发生在序列的头或尾时可以选择deque这种数据结构。

---
成员函数

Interator:

- list.begin() 指向第一个元素的Interator。
- list.end() 指向末尾元素的Interator。
- list.rbegin() 指向第末尾元素的反向Interator。
- list.rend() 指向第一个元素的反向Interator。

Capacity/Size:

- list.empty() 若list内部为空，则回传true值。
- list.size() 回传list内实际的元素个数。
- list.resize() 重新分派list的长度。

Element Access:

- list.front() 存取第一个元素。
- list.back() 存取最末个元素。

Modify Methods:

- list.push_front() 增加一个新的元素在 list 的前端。
- list.pop_front() 删除 list 的第一个元素。
- list.push_back() 增加一个新的元素在 list 的尾端。
- list.pop_back() 删除 list 的最末个元素

---
完整代码:<https://github.com/sbxfc/STLContainers/tree/master/STL-Deque>

---
参考:

Effective.STL

标准模板库:<br><http://zh.wikipedia.org/zh-cn/%E6%A0%87%E5%87%86%E6%A8%A1%E6%9D%BF%E5%BA%93>

STL源码剖析---deque:<br><http://blog.csdn.net/Hackbuteer1/article/details/7729451>

microsoft-deque:<br><https://technet.microsoft.com/zh-cn/sysinternals/bb398188.aspx>
