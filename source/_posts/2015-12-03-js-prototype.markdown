---
layout: post
title: "Javascript - 基于Prototype的对象结构"
date: 2015-12-03 18:11:18 +0800
comments: true
categories: 
---

Javascript是基于Prototype的编程语言,而非我们熟悉的基于Class的编程语言。

在Javascript里,当我们new一个对象实例时,该对象的Prototype属性会被实例继承,但是Prototype和实例之间是引用关系。
	
当我们为实例对象的一个属性赋值时实际上并不是对Prototype里的属性进行修改,而是在实例上设置一个新属性。当程序访问该实例上的属性时,编译器会首先搜索实例自身的属性(即新添加的属性),如果没找到才会去搜索其继承的Prototype上的值:

	function Person{}
	Person.prototype.name = "sbxfc";
	
	var p = new Person();
	//p身上没有name属性,转而去Person身上找
	console.log(p.name);
	
	//该name属性非Person上的name属性
	p.name = "ss";
	console.log(p.name);

#参见:

- <http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html>
- <http://www.cnblogs.com/mindsbook/archive/2009/09/19/javascriptYouMustKnowPrototype.html>
