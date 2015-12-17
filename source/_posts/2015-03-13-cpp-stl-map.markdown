---
layout: post
title: "STL - map容器"
date: 2015-03-13 17:57:00 +0800
comments: true
categories: 
---

map是一种关联容器。容器中每个元素含有key/value两项。map将其中一项key值映射到另一个项value中,key值在map容器中是唯一的。

map被修改时会对所有key值进行排序。

---
成员函数

Interator:

- map.begin() 指向第一个元素的Interator。
- map.end() 指向末尾元素的Interator。
- map.rbegin() 指向第末尾元素的反向Interator。
- map.rend() 指向第一个元素的反向Interator。

Size:

- map.empty() 如果 map 内部为空，则传回 true 值。
- map.size() 获取 map 目前持有的元素个数。

Element Access:

- map.at(i) 访问key值为 i 的元素引用。
- map[i] 访问key值为 i 的元素引用。

Modify Methods:

- map.insert 按照key/value值插入元素
- map.erase() 删除 map 中一个或多个元素。
- map.clear() 清空所有元素。

---
代码示例:

	int main() {
	    typedef std::map<char, int> MapType;
	    MapType myMap;
	    myMap.insert(std::pair<char, int>('f', 6));
	    myMap.insert(std::map<char, int>::value_type('d', 4));
	    myMap.insert(MapType::value_type('c', 3)); 
	    myMap.insert(std::make_pair('b', 2));      
	    myMap.insert({'a', 1});                    // using C++11 initializer list
	    
	     /*
	     * map被修改时会对所有key值自动由低往高排列
	     * 所以,map.begin()会指向key值最小的元素
	     */
	    MapType::iterator iter = myMap.begin();
	    
	    //清除第一个元素
	    myMap.erase(iter);
	    
	    //map中元素数量
	    std::cout << "Size of myMap: " << myMap.size() << '\n';
	    
	    //查找key值为'c'的元素
	    char c = 'c';
	    iter = myMap.find(c);
	    if (iter != myMap.end() )
	        std::cout << "Value is: " << iter->second << '\n';
	    else
	        std::cout << "Key is not in myMap" << '\n';
	    
	    //删除key值为'd'的元素,成功会返回1，否则返回0
	    unsigned long n = myMap.erase('d');
	    if(n == 1){
	        std::cout << "delete key 'd'" << '\n';
	    }
	    
	    //清空map
	    myMap.clear();

	    return 0;
	}

---
完整代码:<https://github.com/sbxfc/STLContainers/tree/master/STL-Map>

---
参考:

Effective.STL

标准模板库:<br><http://zh.wikipedia.org/zh-cn/%E6%A0%87%E5%87%86%E6%A8%A1%E6%9D%BF%E5%BA%93>
