---
layout: post
title: "iOS - Property的修饰词"
date: 2016-09-14 14:11:45 +0800
comments: true
categories: 
---

#原子性

<font color='#bd260d'>**nonatomic/atomic**</font>  是objc里有关属性原子操作的两种修饰词,其中 nonatomic 是非线程安全的,即不对 setter/getter 操作进行加锁,而 atomic 会对生成的 setter/getter 添加<font color='#bd260d'>**类似于**</font>下面这样的互斥锁操作:
	
	- (void) setProperty:(property_type *) property {
	    @synchronized(self){
			_property = property;
	    }
	}

尽管,Apple 可能会使用一些比 @synchronized 更加高级的互斥操作,但是由于 atomic 本身并不能保证已加锁的类和对象是 <font color='#bd260d'>**thread safe**</font> 的,并且 atomic 在加锁时会消耗一些系统资源,我们一般会用 nonatomic 来作为修饰词。

需要注意的是,如果你没有在属性上键入 nonatomic 修饰词,编译器会把 atomic 当做默认修饰词( [<font color='#bd260d'>**吐槽看这里**</font>](http://stackoverflow.com/questions/5168331/why-is-atomic-a-default-property-qualifier-in-objective-c-when-i-find-myself) )。

#读写权限

<font color='#bd260d'>**readwrite/readonly**</font> 定义了属性的访问权限,其中 readwrite 是可读写,当你声明这个修饰词时,编译器会生成该属性的 setter/getter方法。readonly 为只读,当声明该属性时只会生成 getter 方法。

假如你只声明了 readonly 权限,但自己手动实现了 setter 方法,该属性也会变成可读可写的(囧)。所有属性,如果未声明读写权限,编译器会以 readwrite 作为默认方式处理。

#内存管理

<font color='#bd260d'>**1.  strong/retain**</font> 在 MRC (Mannul Reference Counting) 时代,当我们想得到一个对象的控制权时,我们会对该对象进行 retain 操作。这样一来,该对象的引用次数就会增加1,即便在其他地方,该对象的其他指针被置为 nil 或者是被 release过一次。但由于其总引用次数没有减小至0,系统依旧不会释放该对象,不会出现 "EXC_BAD_ACCESS" 错误。

	/**
	 * 使用retain标识属性的内存处理方式时,会在 setter 方法里,
	 * 对传入的对象进行引用计数加1的操作。
	 */
	-(void)setProperty:(property_type*)property{  
	     if ( _property != property){  
	          [_property release];  
	          _property = [property retain];  
	     }  
	}

进入 ARC (Auto Reference Counting) 时代,retain 被 strong 取代了。实际上,在ARC环境里,无论是使用 strong 还是 retain ,其作用是完全一致的, strong = retain 。但 ARC 环境里,因为编译器会帮我们自动处理 setter 和 getter 里的 retain 和 release 操作,这时用 strong 修饰词就显得更 ARC 的语意。

在objc里,object 类型的属性默认内存处理方式是 strong,也就是说,即便你没有指明 object 属性内存修饰词时,系统会默认按照 strong 的方式处理该属性。而基本类型(int、float、double、NSInteger，CGFloat等) 的属性,其默认是以 <font color='#bd260d'>**assign**</font> 的方式处理的。

<font color='#bd260d'>**2.  assign/weak/unsafe_unretained**</font> 使用 assign 修饰的属性,在赋值时 setter 方法里进行的是简单的指针拷贝,并没有像 retain 或 strong 那样进行引用计数的增加。对于基本类型,由于其本身是由系统自动分配和管理的,不需要考虑内存问题,所以可以直接使用指针来赋值。但如果用 assign 修饰 object 属性,一旦属性指向的对象中的其中一个引用指针被为nil,系统就会回收该对象,由此会导致,当我们访问该 object 属性时出现 "EXC_BAD_ACCESS" 错误。

对于 object 类型的属性,如果我们不想持有该属性指向的对象。我们可以使用 weak 修饰词,weak 修饰词和 assign 类似 ,唯一的区别是,当 weak 修饰的属性如果其指向对象的指针在别处被置为nil,导致该对象被释放,那么 weak 属性里的成员变量指针也会被置为nil,从而不会出现野指针的情况。(delegate 和 Outlet 一般用weak来声明。)。

在ARC之前,我们通常用 assign 来修饰基本类型,而 unsafe_unretained 则通常用于“对象类型”,unsafe_unretained 与 assign 是等价的,在实际使用时 unsafe_unretained 修饰“基础类型”并不会报错 。进入ARC之后,我们用weak代替unsafe_unretained修饰“对象类型”。

<font color='#bd260d'>**3.  copy**</font> 与strong类似，但区别在于属性的成员变量是对传入对象的副本拥有所有权，而非对象本身。

#参见
- <http://stackoverflow.com/questions/5168331/why-is-atomic-a-default-property-qualifier-in-objective-c-when-i-find-myself>