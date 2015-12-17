---
layout: post
title: "Xib下的自动布局 - ScrollView(一)"
date: 2015-03-01 19:09:49 +0800
comments: true
categories: 
---

为ScrollView布局实际上是为ScrollView里的content设置布局。<br>区别于一般组件,ScrollView有一个contentSize的概念。contentSize是ScrollView填充的内容大小,<br>ScrollView会根据这个范围来设置滚动区域。

假如我们对ScrollView里的内容按照一般组件那样设置布局。很容易出现组件内容不能拖动<br>或者直接在布局界面就出错的情况:
(无法确定拖动的区域)

*has ambiguous scrollable content height*<br>
*has ambiguous scrollable content width* <br>

<!--more-->

---
#为ScrollView设置布局

一,新建一个xib取名为MyScrollView.xib

二,在viewDidLoad里加载该Xib,并设置xib距离父视图的四个边的边距为0。

	- (void)viewDidLoad {
	    [super viewDidLoad];

	    //加载xib文件
	    [self loadScrollView];
	}
	/**
	 * 加载xib,并设置布局全屏显示。
	*/
	-(void)loadScrollView
	{
	    MyScrollView *myScrollView = [[[NSBundle mainBundle] loadNibNamed:@"MyScrollView" owner:self options:nil] firstObject];
	    [self.view addSubview:myScrollView];
	    //...
	    //设置xib距离父视图的四个边的距离为0
	    //...
	}

三,打开新建的MyScrollView.xib,拖入一个ScrollView组件,设置距离四个边的距离为0。<br>
再拖入一个View到ScrollView上,同样设置四个方向边距为0。<br>
此时会报错:

*has ambiguous scrollable content height*<br>*has ambiguous scrollable content width*<br>

**为View设置了四个方向的边距,但是ScrollView并不知道contentSize的大小。**<br>
**设置四个边距实际是告诉ScrollView内容会“撑到”多大。**<br>
**ScrollView必须知道四条边多大来确定contentSize的大小。**

四,设置View和ScrollView的宽度相等(**Equal Widths**)。此时错误只剩下一个:

*has ambiguous scrollable content height*
 
我们为其设置了两边的宽度,但高度还未确定。接下来,我们通过向View添加组件来填充View的高度。

五,在View上再拖入一个子View并设置四个方向上的边距为10,高度为2000,设置背景色为粉色,此时错误消失。<br>运行会看到一个可以上下拖动的粉色面板。

<br>

---
#组件内置大小

假如有多个子View,他们纵向依次间隔排列。但中间一个View并不确定高度(至少在xib布局时无法给出一个明确高度)。如果是普通的View,这种布局没有问题,但对于ScrollView就会出错:

*has ambiguous scrollable content height*


此时,我们可以为其设置**intrinsic size**(内置大小),属性面板里的 **intrinsic size** 选项。一个原始的UIView对象，它的内置大小为0。我们通过设置占位符来改变内置宽、高，也可以根据需要只设置其中一项。

然后在xib绑定类的awakeFromNib函数里设置该View的高度约束,awakeFromNib函数在约束生效前执行。若没有设置高度约束也不会出错,但该View不显示。因为组件的内置大小是由组件本身的内容所确定的。UIView不像UIButton或UILabel,它的内容为空,运行时UIView的内置大小为0。
	
	- (void)awakeFromNib {
	    [super awakeFromNib];
	    
	    //重新设置高度
	    [self.topSubView addConstraint:[NSLayoutConstraint
	                                 constraintWithItem:self.topSubView
	                                 attribute:NSLayoutAttributeHeight
	                                 relatedBy:NSLayoutRelationEqual
	                                 toItem:nil
	                                 attribute:NSLayoutAttributeNotAnAttribute
	                                 multiplier:1
	                                 constant:250]];
	}
	
	
---

完整代码:<https://github.com/sbxIOS/AutoLayoutUIScrollView>