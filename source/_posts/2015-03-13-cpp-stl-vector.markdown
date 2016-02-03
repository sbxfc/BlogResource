---
layout: post
title: "STL - vector"
date: 2015-03-13 17:50:34 +0800
comments: true
categories: 
---

vector是一个可以自动扩充容量的容器,其设计的目的是为了取代C语言中的数组。vector的内部实现使用了动态数组的形式来存放数据,其内部元素在内存上的分布是连续的。

vector支持随机存取,并且在集合尾端增删元素是很快的，但是在集合中间增删元素比较费时。vector里的内存是连续的,当其中一个元素被删除时,其他元素就需要向上或者向下移动来填充原来被删除的元素所占的空间。在增加元素时,如果当前容器的容量已满,vector会动态增加容量,一次增加的容量为当前容量的2倍。

频繁的执行内存分配,并把所有元素从容器的旧内存copy到新内存,然后销毁就内存的对象并回收旧内存的代价是昂贵的。我们可以在一开始就利用reserve将容器的容量设置的足够大。

#访问元素
	//访问索引为i的元素的引用
	vec[i] 
	 /*
	 * 访问索引值为i的元素的引用,以 at 访问会做数组边界检查,
	 * 如果访问越界将会抛出一个例外,这是与operator[]的唯一差异
	 */	
	vec.at(i) 
	//回传vector第一个元素的引用
	vec.front() 
	//回传vector最尾元素的引用。
	vec.back() 

#修改数据

	//新增元素至vector的尾端。
	vec.push_back()  
	//删除vector最尾端的元素。
	vec.pop_back()  
	//插入一个或多个元素至vector内的任意位置。
	vec.insert()  
	//删除vector中一个或多个元素。
	vec.erase()
	//清空所有元素。  
	vec.clear() 

#容量和大小
	
	//获取vector目前持有的元素个数。
	vec.size() 
	//如果vector内部为空,则传回true值。尽量用这个值判断vector是否为空。 
	vec.empty()
	//获取vector目前可容纳的最大元素个数 
	vec.capacity() 
	/**
	 * 强制把容量改为至少size大小,提供的size不能小于当前大小。
	 * 如果size小于当前容量,vector忽略它。如果size大于当前容量,重新分配内存。
	 */ 
	vec.reserve(size) 
	/**
	 * 强制把容器改为容纳size个元素。调用resize之后,size将会返回n。
	 * 如果n小于 当前大小,容器尾部的元素会被销毁。
	 * 如果n大于当前大小,新增加的默认元素会添加到容器尾部。
	 * 如果n大于当前容量,在元素加入之前会发生重新分配内存。
	 */ 
	vec.resize(size)  

# 迭代器

	//回传一个Iterator，它指向vector第一个元素。
	vec.begin()  
	//回传一个Iterator，它指向vector最尾端元素的下一个位置（请注意：它不是最末元素）。
	vec.end()  
	//回传一个反向Iterator，它指向vector最尾端元素的。
	vec.rbegin()
	//回传一个Iterator，它指向vector的第一个元素。	
	vec.rend()  

#复制vector到一个普通数组[]

对于一个vector,vector[0]即表示对内部元素中的第一个元素的引用。所以,&vector[0]则表示那个元素的指针。
而在vector中,其内部元素的内存分布是连续的,就相当于一个普通数组。所以,我们可以像普通数据一样去使用vector,而无需将其转化为普通数组。

在使用普通数组传参时,我们一般会传递数组中第一个元素的地址和数组里元素的数量,比如:

	void doSomething(const int* pNumber, size_t size);
	
我们可以直接使用vector这样做

	doSomething(&vec[0], vec.size());

#通过复制内存将vector复制到普通数组

如果需要将一个vector复制到普通数组,我们可以使用内存复制的方式,快速复制vector:

	std::vector<int> vec = {1,2,3};
	    
	//将vector复制到普通数组
	int* a = (int*)malloc(vec.size()*sizeof(int));
	memcpy(a, &vec[0], vec.size()*sizeof(int));
	    
	//输出
	std::cout << a[0] << ","<< a[1] << ","<< a[2] <<std::endl;
    
#代码

- <https://github.com/sbxfc/cppVector>

#参见

- Effective.STL

- Effective C++

- More Effective C++

- [vector(STL)](http://zh.wikipedia.org/wiki/Vector_(STL))

- [标准模板库](http://zh.wikipedia.org/zh-cn/%E6%A0%87%E5%87%86%E6%A8%A1%E6%9D%BF%E5%BA%93)