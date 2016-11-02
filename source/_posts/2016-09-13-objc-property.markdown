---
layout: post
title: "关于属性和成员变量的疑惑"
date: 2016-09-13 10:03:20 +0800
comments: true
categories: 
---

#成员变量和属性

在objc里变量有两种写法，一种是以 <font color='#bd260d'>**@property**</font> 形式声明的属性,另一种是定义在大括号里 (<font color='#bd260d'>**@interface :NSObject{}**</font> )的成员变量( <font color='#bd260d'>**instance variable**</font> )。
	
	@interface MyObject : NSObject {
	    NSInteger memberVar; // 成员变量
	}
	
	@property NSInteger value; // 属性
	
	@end

这两者有什么区别呢?

从设计的角度上来讲, @property 方式声明的属性,主要是用来暴露给外部访问用的。而成员变量( instance variable )是内部变量,主要用作内部使用,外部无法访问。

需要留意的是,成员变量既可以在 <font color='#bd260d'>**@interface :NSObject{}**</font> 里声明,也可以在 <font color='#bd260d'>**@Implementation :NSObject{}**</font> 里声明。在 <font color='#bd260d'>**@Interface**</font> 区域声明的成员变量默认权限是protected,能被被子类访问,而在 <font color='#bd260d'>**@implementation**</font>里声明的成员变量权限是private。在 <font color='#bd260d'>**@implementation**</font> 里定义私有成员变量更符合面向对象的封装原则,因为此类别的信息不会暴露于公开的interface中。

成员变量和属性在访问时也有所不同,属性可以用<font color='#bd260d'>**点表达式(.)**</font> 来访问(<font color='#bd260d'>**self.name**</font>),而成员变量,因为是内部变量可以直接使用变量名(<font color='#bd260d'>**name**</font>)访问或者通过<font color='#bd260d'>**右箭头(->)**</font>来访问(<font color='#bd260d'>**self->name**</font>)。

#属性的声明

经常看到有的人用 <font color='#bd260d'>**@property**</font>声明了属性,又在<font color='#bd260d'>**@interface**</font>里定义了相同的成员变量。

	@interface MyObject : NSObject{
	    NSInteger value;
	}
	
	@property NSInteger value;
	@end

首先应当了解一下 @property 的用法。当我们用 @property （和 @synthsize) 声明一个属性时,在 .h 文件里 @property会让编译器帮我们创建两个存取方法的声明:

	- (NSInteger)propertyName;      
	- (void)setPropertyName:(NSInteger)value;

而在 .m 文件里, <font color='#bd260d'>**@synthesize propertyName**</font> 会告诉编译器,帮我们创建这两个声明的实现函数,并且以 propertyName 命名该属性的成员变量。	
	
	- (void)setPropertyName:(NSInteger)value{	
		propertyName  =  value;
	}
	
	- (NSInteger)propertyName{
		return	propertyName;
	}
	
因此,在使用 @property （和 @synthsize) 声明属性时,编译器帮我们生成读写方法的同时,也会自动创建一个成员变量,不需要我们手动去创建。

假如,我们不想使用现有的属性名作为成员变量名,我们可以将 <font color='#bd260d'>**@synthesize propertyName**</font> 改为 <font color='#bd260d'>**@synthsize propertyName = xxx**</font>  的形式,这样我们的成员变量就变成了 <font color='#bd260d'>**xxx**</font>。Xcode4.5 以后,如果我们没有写 @synthsize ,系统会为我们自动生成一个 @synthsize 并指定一个以下划线 ( <font color='#bd260d'>**_**</font> ) 为前缀,加上属性名的成员变量,即  <font color='#bd260d'>**@synthsize propertyName = _propertyName**</font> 。

#.h和.m文件

在objc里,类的定义（interface）与实现（implementation）被分成了两个部分。我们知道在 @interface 里面定义的属性和函数主要用于外部访问,但是这里有一个的前提条件:

	//MyObject.h
	@interface MyObject : NSObject{
	    NSInteger memberVar1;
	}
	
	@end
	
	//MyObject.m
	@interface MyObject(){
	    NSInteger memberVar2;
	}
	
	@end

在上面的 .h 和 .m文件里的两个 @interface 定义区域里,我们分辨创建了两个成员变量 memberVar1 和 memberVar2。其中在 .h 里面写的成员变量会暴露出来,而在 .m里写的变量外界是看不到的,也不能访问。

当我们在子类里访问该类的成员变量时,只能看到暴露出来的memberVar1。

![](/images/2016/9/tmp799c4889.png)

<font color='#bd260d'>**所有定义在 .m文件里的变量和函数,无论是 @interface 还是 @implementation ,外界是无法访问的。**</font> 
#参见
- <https://zh.wikipedia.org/wiki/Objective-C>
- <http://www.cnblogs.com/letmefly/archive/2012/07/20/2601338.html>
- <http://www.devtalking.com/articles/you-should-to-know-property/>