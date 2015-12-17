---
layout: post
title: "Xib下的自动布局 - ScrollView(二)"
date: 2015-03-02 11:45:34 +0800
comments: true
categories: 
---

在ScrollView上布局时,有一些区域不能确定大小。我们可以为这些区域设置一个UIView并通过修改**intrinsic size**里的占位符来完成约束。利用这个还有用的技巧,我们可以在这块区域上动态填充期望的内容。比如在上面动态加载一个xib文件。当填充的内容超过UIView的大小时,只要设置好xib和UIView之间的布局,内容区就会自动被扩充。

这些操作一般会在`awakeFromNib`函数里进行设置,`awakeFromNib`在布局生效前被执行。

下图中,红色区域设置了占位符高度:

![](/images/2015/3/autolayout-scrollview-before.png)

<!--more-->

运行时,因为UIView没有添加内容,红色区域不显示,如下图:

![](/images/2015/3/autolayout-scrollview-after.png)

在上一部分,我们提到UIView的默认**intrinsic size**值为0。但对于其他组件,如UILabel,UIButton这些有内容的组件内置大小不为0。将红色区域换成一个UIButton,无需设置**intrinsic size**的值,我们发现ScrollView也不会报错。

运行时一下,我们看到UIButton显示出来了:

![](/images/2015/3/autolayout-button-after.png)

---
动态加载xib
---

在ScrollView里,我们可以使用一个UIView作为占位容器,设置**intrinsic size**值。<br>
在运行时,往UIView上动态添加Xib,并设置布局。

下图中绿色区域动态加载了一个包含UILabel的xib文件:

![](/images/2015/3/autolayout-xib-after.png)

---


完整代码:<https://github.com/sbxIOS/AutoLayoutScrollView2>
 <!--more-->