---
layout: post
title: "STL - list容器"
date: 2015-03-13 17:56:48 +0800
comments: true
categories: 
---

翻看了很多关于list容器的文章,对list的介绍都是一笔带过。但对我这种初学者来说,对于概念的理解有时候甚至比该怎么用来的重要。平时敲码的过程中,对于不是十分了解的内容,心里总是耿耿于怀。完成了一项任务,心里也不爽。一停下手,想把问题彻底搞清楚，又碍于没有时间,不能沉下心来分析、总结。所以,何不在一开始就把这一切都学到心里去,或是写一份值得翻阅的笔记呢。

关于list的介绍,在WIKI上的[标准模板库](http://zh.wikipedia.org/zh-cn/%E6%A0%87%E5%87%86%E6%A8%A1%E6%9D%BF%E5%BA%93 "WIKI STL")里找到了一小段描述,上面写的比较规范也容易理解:"list 容器是一个有序（Ordered）的数据结构（循序容器），每个元素中存储着上一个元素和下一个元素的地址（指针），因此是一个双向链接的链表。与 vector 相比，其元素的访问速度较慢，而在已知元素位置的情况下，插入和删除速度较快。"

---
List(STL)

list内部以数据结构的双向连接串行实现,内部元素并非放置在一块连续的内存里,而是散落在内存各处。在访问list时,必须从第一个元素逐个往下寻访,不支持随机存取,不能像vector一样使用[]运算符。

list的强项是可以在序列中间进行高效的插入和删除操作。对于list插入和删除,只需要改动元素的前、后元素指针。

若仅需要于集合尾端增删元素，那应该优先考虑vector容器，若仅于头尾二端增删元素，那应该优先考虑deque容器。

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
完整代码:<https://github.com/sbxfc/STLContainers/tree/master/STL-List>

---
参考:

Effective.STL

标准模板库:<br><http://zh.wikipedia.org/zh-cn/%E6%A0%87%E5%87%86%E6%A8%A1%E6%9D%BF%E5%BA%93>

List(STL):<br><http://zh.wikipedia.org/wiki/List_(STL)>

List属性:<br><http://zh.cppreference.com/w/cpp/container/list>