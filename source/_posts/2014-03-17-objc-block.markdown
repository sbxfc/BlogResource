---
layout: post
title: "在objc中使用Block"
date: 2014-03-17 15:21:33 +0800
comments: true
categories: 
---


一个完整的Block由声明和主体两部分组成,这种形式与定义一个JaveScript函数类似。但由于Block在其语法内部使用<font color="#bd260d">**^**</font>符号,使其整体看起来比较古怪。

	NSInteger x = 123;
    void (^printXAndY)(NSInteger) = ^(NSInteger y) {
        NSLog(@"%d %d\n", x, y);
    };
    printXAndY(456); // prints: 123 456

上面是一个带有简单输出功能的Block,<font color="#bd260d">**=**</font> 号的左侧为Block的声明部分( <font color="#bd260d">**void (^printXAndY)(NSInteger)**</font> )<br>
该声明从左至右依次为<font color="#bd260d">** 返回值类型 、 Block的名称 和 输入参数类型 **</font> , 并用括号进行了分割。在这个Block声明里,我们能看出该Block声明的返回值为空,名称为printXAndY,传入参数为整形。事实上,除了传入参数数量或是返回值类型可能不同外,每个Block声明的写法大致也就是这样子。

<font color="#bd260d">**=**</font> 号右侧是Block的主体部分,在该示例里,主题部分有一个整形的输入参数y(与左侧声明相对应),在其右侧的大括号是主题部分的逻辑实现,类似于函数的大括号部分,因为没有返回值,所以没有写return,只做了一个简单的输出。



#__block

在Block里,外部变量以Copy的形式复制到Block的内部,Block没有对外部变量的修改权(<font color="#bd260d">**若是修改,在编译时会提示错误**</font>)。所以,如果在Block执行前外部变量发生变化,Block内部该值不会随之改变:
	
	NSInteger x = 123;
	void (^printXAndY)(NSInteger) = ^(NSInteger y) {
    	x = x + y; // error
    	NSLog(@"%d %d\n", x, y);
	};
	x = 789;
	printXAndY(456); // prints: 123 456


但是,Block提供了 <font color="#bd260d">**__block**</font> 类型的变量声明,该声明可以使变量与Block绑定在一起,这样当外部变量发生改变时,Block内部变量值也会一起变化:
	
	__block NSInteger x = 123;
	void (^printXAndY)(NSInteger) = ^(NSInteger y) {
    	NSLog(@"%d %d\n", x, y);
	};
	x = 789;
	printXAndY(456); // prints: 789 456
	
#引用声明

除了上述特点之外,Block用到最多的就是其提供的与JS闭包和匿名表达式类似的语法操作。如果熟知JS语法,一定会对其闭包和匿名表达式的灵活使用印象深刻。
	
在Block里,声明和主体部分可以单独拆分使用,拆分后的Block声明相当于一个带有输入参数和返回值的模板类型,比如我们常用的动画API就使用了Block: 
	
	//动画函数声明
    + (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations //...
    
从代码可以看出,函数参数是 <font color="#bd260d">**void (^)(void)**</font>,这是一个匿名的Block声明。当动画函数调用时,我们传递一个与之匹配的Block主体:
	
	//动画函数调用
	[UIView animateWithDuration:5.0f animations:^{
        label.alpha = 0.5f;
    }];
    
为了使代码结构更加清晰,我们可以直接声明一个Block类型:

	typedef void (^TableViewCellConfigureBlock)(id cell, id item);
    
然后在使用时进行Block参数传递:
	
	//传递参数
	void (^configureCell)(PhotoCell*, Photo*) = ^(PhotoCell* cell, Photo* photo) {
   		cell.label.text = photo.name;
	};
	photosArrayDataSource = [[ArrayDataSource alloc] initWithItems:photos
                                                cellIdentifier:PhotoCellIdentifier
                                            configureCellBlock:configureCell];
	self.tableView.dataSource = photosArrayDataSource;
	
	//执行Block
	self.configureCellBlock(cell, item);
	
从中我们可以看出Block函数能像JS里的闭包和匿名函数一样灵活传递和使用,填补了objc语法的缺陷。
	

#参见
- <https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Blocks/Articles/bxVariables.html#//apple_ref/doc/uid/TP40007502-CH6-SW6>




<!--Block是一个C语言语法以及运行时的一个特性,和标准C中的函数（函数指针）类似。但是其运行需要编译器和运行时支持,2010年发布iOS4时Apple开始支持Block。

	int multiplier = 7;
	int (^myBlock)(int) = ^(int num) {
    	return num * multiplier;
	};

这是一段简单的Block程序。符号<font color="#bd260d">**^**</font>声明了一个名为myBlock的Block,它的左侧定义了该Block的返回值类型为int,<font color="#bd260d">**^**</font>右侧表示传入参数也是int类型。<font color="#bd260d">**=**</font>的右侧是myBlock主体的部分。<font color="#bd260d">**^(int num)**</font>设置一个名为num的传入参数。<font color="#bd260d">**{}**</font>包含的部分是block主体,该Block返回了一个乘积值。最后,整个Block以<font color="#bd260d">**;**</font>结束。

myBlock的结构:

![Blocks](/images/2014/3/blocks.jpg)

<font color="#bd260d">**Block 可以访问同一域下的局部变量 multiplier 。但需要注意的是,虽然可以访问 multiplier 的引用但却不能对其做处理。**</font>


#把Block当变量使用


	int multiplier = 7;
	int (^myBlock)(int) = ^(int num) {
    	return num * multiplier;
	};
 
	printf("%d", myBlock(3));
	// prints "21"



# 直接使用Block

你也可以不直接声明Block而直接使用定义部分。

	char *myCharacters[3] = { "TomJohn", "George", "Charles Condomine" };
 
	qsort_b(myCharacters, 3, sizeof(char *), ^(const void *l, const void *r) 	{
    	char *left = *(char **)l;
    	char *right = *(char **)r;
    	return strncmp(left, right, 1);
	});
 
	// myCharacters is now { "Charles Condomine", "George", "TomJohn" }

示例代码中,以`^`开始,剩下了Block的定义部分。  

qsort_b与标准的qsort_r方法类似,区别在于qsort_b以Block为参数。  

#在Cocoa中使用Block

Cocoa Frameworks里很多用到了Block。  

下面示例是NSArray在排序中对Block的使用:
Block接收两个参数并返回一个NSComparator类型的比较结果。

	NSArray *stringsArray = @[ @"string 1",
                           @"String 21",
                           @"string 12",
                           @"String 11",
                           @"String 02" ];

	static NSStringCompareOptions comparisonOptions = NSCaseInsensitiveSearch | NSNumericSearch |
        NSWidthInsensitiveSearch | NSForcedOrderingSearch;
	NSLocale *currentLocale = [NSLocale currentLocale];
 	
 	//比较数组中的两个String对象，这里用到了Block。
	NSComparator finderSortBlock = ^(id string1, id string2) {
 
    	NSRange string1Range = NSMakeRange(0, [string1 length]);
    	return [string1 compare:string2 options:comparisonOptions range:string1Range locale:currentLocale];
	};
 
	NSArray *finderSortArray = [stringsArray sortedArrayUsingComparator:finderSortBlock];
	NSLog(@"finderSortArray: %@", finderSortArray);
 
	/*
	Output:
	finderSortArray: (
    	"string 1",
    	"String 02",
   	 	"String 11",
    	"string 12",
    	"String 21"
	)
	*/
	
**NOTE:**  
	
string1是数组里的元素，所以不会存在nil的情况。  </br>若一个值为nil的string1,根据Objective-C的消息调用规则，对nil发送的任何消息，都会得到nil的空指针,而它代表的值就是0,与NSOrderedSame的值相等。所以下列判断就会出问题。
	
	if ([string1 compare:@"some text"] == NSOrderedSame) {
        // Do something
    }
    else {
        // Do something else
    }
    
正确写法:
	
	if (string1 != nill && [string1 compare:@"some text"] == NSOrderedSame) {
        // Do something
    }
    else {
        // Do something else
    }
    
#局部变量声明 __block

在上面的事例的基础上，我们增加一个整形的计数变量orderedSameCount，变量以__block声明。当该声明的局部变量在Block内发生变化时，外部的变量值也发生变化。

	NSArray *stringsArray = @[ @"string 1",
                          @"String 21", // <-
                          @"string 12",
                          @"String 11",
                          @"Strîng 21", // <-
                          @"Striñg 21", // <-
                          @"String 02" ];
 
	NSLocale *currentLocale = [NSLocale currentLocale];
	__block NSUInteger orderedSameCount = 0;
 
	NSArray *diacriticInsensitiveSortArray = [stringsArray sortedArrayUsingComparator:^(id string1, id string2) {
 
    	NSRange string1Range = NSMakeRange(0, [string1 length]);
    	NSComparisonResult comparisonResult = [string1 compare:string2 options:NSDiacriticInsensitiveSearch range:string1Range locale:currentLocale];
 
    	if (comparisonResult == NSOrderedSame) {
        	orderedSameCount++;
    	}
    	return comparisonResult;
	}];
 
	NSLog(@"diacriticInsensitiveSortArray: %@", diacriticInsensitiveSortArray);
	NSLog(@"orderedSameCount: %d", orderedSameCount);
 
	/*
	Output:
 
	diacriticInsensitiveSortArray: (
    	"String 02",
    	"string 1",
    	"String 11",
    	"string 12",
    	"String 21",
    	"Str\U00eeng 21",
    	"Stri\U00f1g 21"
	)
	orderedSameCount: 2
	*/

-->