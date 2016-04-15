---
layout: post
title: "为ScrollView设置自动布局"
date: 2015-03-01 19:09:49 +0800
comments: true
categories: 
---

在为ScrollView设置布局约束时,由于ScrollView容器内的内容可能是后续动态添加的,在设置布局时,我们不能获知内容的确切尺寸,这时我们可以借助于Intrinsic Size属性来为子视图设置一个预估的宽度值或高度值,来完成后续的布局操作。

![](/images/2016/4/intrinsic_size.png)


#加载xib文件

	CustomScrollView *scrollView = [[[NSBundle mainBundle] loadNibNamed:@"CustomScrollView" owner:self options:nil] firstObject];
    [self.view addSubview:scrollView];

#设置约束

设置约束让scrollView与主视图的四边重合:

	scrollView.translatesAutoresizingMaskIntoConstraints = NO;
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:scrollView
                                                          attribute:NSLayoutAttributeLeading
                                                          relatedBy:NSLayoutRelationEqual
                                                             toItem:self.view
                                                          attribute:NSLayoutAttributeLeading
                                                         multiplier:1.0
                                                           constant:0.0f]];
    
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:scrollView
                                                          attribute:NSLayoutAttributeTrailing
                                                          relatedBy:NSLayoutRelationEqual
                                                             toItem:self.view
                                                          attribute:NSLayoutAttributeTrailing
                                                         multiplier:1.0
                                                           constant:0.0f]];
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:scrollView
                                                          attribute:NSLayoutAttributeTop
                                                          relatedBy:NSLayoutRelationEqual
                                                             toItem:self.view
                                                          attribute:NSLayoutAttributeTop
                                                         multiplier:1.0
                                                           constant:0.0f]];
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:scrollView
                                                          attribute:NSLayoutAttributeBottom
                                                          relatedBy:NSLayoutRelationEqual
                                                             toItem:self.view
                                                          attribute:NSLayoutAttributeBottom
                                                         multiplier:1.0
                                                           constant:0.0f]];


#示例

示例中,CustomScrollView视图与主视图的四边重合,这个布局约束是ScrollView加载后用代码设置的。

ScrollView视图,首先拖入了一个ScrollView组件,设置四边与重合。然后拖入一个UIView取名为Container作为ScrollView的content。同样设置布局让四边与ScrollView重合。

分为拖入两个UIView取名为TopView和BottomView,TopView设置与上、左、右三边与Container重合的约束,底边设置与BottomView的纵向间隙为0。BottomView设置左、右边距并设置了一个高度。**最后,别忘了设置TopView的Intrinsic Size**

- <https://github.com/sbxfc/objc/tree/master/autolayout/>


