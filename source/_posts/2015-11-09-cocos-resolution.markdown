---
layout: post
title: "Cocos - 屏幕适配"
date: 2015-11-09 17:50:41 +0800
comments: true
categories: 
---

现在,几乎人手都有一台移动设备。无论是手机还是iPad,这些设备的尺寸千差万别。因此,想开要发一款能在不同设备上友好显示的游戏,就要综合这些设备的特点,设计合理的适配方案。

从2.0.4开始,Cocos2d-x就提出了自己的多分辨率适配方案,经过这几年的发展已趋于成熟。对于开发者来说,如果能掌握这些技巧,就能够很好地解决适配问题。

#资源到设计尺寸

在Cocos里,官方把适配方案分为了两步,第一步是资源到设计尺寸的适配。

因为设备是多种多样的,当我们在开发游戏时并不会假定界面是基于某台设备来设计界面。通常,我们会选择一个较为通用的尺寸来设计界面(比如1027*768),这个尺寸就是设计尺寸。

从资源到设计尺寸的映射,其实就是图片到设计尺寸的映射。例如,我们为iPhone3G和iPhone 4设计一款游戏。iPhone 3G的分辨率是320*480,而iPhone 4的分辨率是640\*960。尽管这两种设备分辨率不同,但是尺寸一样大。所以,这两种设备可以使用同一个设计尺寸,只需要将iPhone4的图片资源换成高清图即可。

通过Cocos的setContentScaleFactor接口可以设置所有资源的缩放因子:

	director->setContentScaleFactor(2);
	
设置完之后,一张640\*960的背景图在一个sprite显示时,高宽变为为320*480,而分辨率变为了原本的4倍。这样一来,iPhone 3G和iPhone 4的屏幕适配做好了。

当然,因为改变了缩放因子,这两种设备设置加载的资源就不同,为了方便管理,我们为其设置了不同的资源目录:
	
	/Resource
		/iphone
		/iphonehd
		
同时,根据具体设备为其指定Resource下的默认资源路径:
	
	vector<string> searchPath;
	if (iPadHD){	
		searchPath.push_back("iphonehd");
	}else if(iPhone){
		searchPath.push_back("iphone");
	}
	searchPath.push_back("common");
	FileUtils::getInstance()->setSearchPaths(searchPath);
	
	

现在iPhone 3G近乎绝迹,大部分iOS设备都是高清设备。所以,不必考虑为iPhone设备设置普清图。但是普清的iPad2用的还是比较多的(最后会介绍一点iOS原生语言的适配方法)。

#适配屏幕分辨率

一开始时,我们在一个预设的尺寸上进行界面布局,当程序运行在实际设备上时,我们需要为其选择一个合适的适配方案。比如设计尺寸是320*480,而实际分辨率是320\*500,这样一样,如果不做适配上下就会空出一条黑色缝隙。设计尺寸到屏幕分辨率的适配就是设计窗口如何缩放来适应实际屏幕的分辨率的过程。

Cocos提供了五种设计尺寸到屏幕分辨率的适配方案,通过setDesignResolutionSize函数来设置。DW、DH指设计尺寸的宽、高,后面的参数是适配方案的枚举值:
	
	
	setDesignResolutionSize(DW,/*分辨率宽*/
							 DH,/*分辨率高*/
							 resolutionPolicy) 
							 
适配方案resolutionPolicy有五种选择:					 
							 
- ResolutionPolicy::SHOW_ALL 屏幕宽、高分别和设计分辨率宽、高计算缩放因子，取较小者作为宽、高的缩放因子。保证了设计区域全部显示到屏幕上，但可能会有黑边。
- ResolutionPolicy::EXACT_FIT 屏幕宽与设计宽的比例作为X方向的缩放因子，屏幕高与设计高的比例作为Y方向的缩放因子。保证了设计区域完全铺满屏幕，但是可能会出现图像拉伸。
- ResolutionPolicy::NO_BORDER 屏幕宽、高分别和设计分辨率宽、高计算缩放因子，取较(大)者作为宽、高的缩放因子。保证了设计区域总能一个方向上铺满屏幕，而另一个方向一般会超出屏幕区域。
- ResolutionPolicy::FIXED_HEIGHT 保持传入的设计分辨率高度不变,根据屏幕分辨率修正设计分辨率的宽度。(按照适配尺寸的宽度将屏幕撑满,宽度可根据屏幕的分辨率拉伸或者是裁剪)
- ResolutionPolicy::FIXED_WIDTH 保持传入的设计分辨率宽度不变,根据屏幕分辨率修正设计分辨率的高度。

最后两种适配方案,是我们常用的适配方式。以ResolutionPolicy::FIXED_HEIGHT为例,假如我们的设计尺寸是320*480,在640\*1000的设备上运行时,原先一个点为(100,100)的sprite,其实际位置就出现在屏幕上(200,208)这个像素点的位置。实际界面上的图片如果没有设置根据分辨率和设计尺寸的变化而改变,将不会变化。

在Cocos2d-x里的cpp-empty-test项目里有一个AppMacros.h的文件,是官方针对iphone、ipad和ipadhd所做的适配方案。另外,还有一些重要的接口:
	
	Director::getInstance()->getOpenGLView()->setDesignResolutionSize() //设计分辨率大小及模式 
	Director::getInstance()->setContentScaleFactor() //内容缩放因子 
	FileUtils::getInstance()->setSearchPaths() //资源搜索路径 
	Director::getInstance()->getOpenGLView()->getFrameSize() //屏幕分辨率 
	Director::getInstance()->getWinSize() 	    //设计分辨率 
	Director::getInstance()->getVisibleSize() //设计分辨率可视区域大小 
	Director::getInstance()->getVisibleOrigin()//可视区域起点

#iOS原生语言的适配方式

从iPad3、iPad mini2和iPhone4开始,苹果的设备都采用了"Retina"显示技术,即将多个像素点压缩至一块屏幕里。比如,一台iPhone4设备,GPU在工作时渲染出960*640个像素点,其中每四个像素一组,输出到原来屏幕上一个像素的显示区域里。这样一来，用户所看到的图标与文字的大小与原来的480x320分辨率显示屏相同，但精细度是原来的4倍。

在iOS原生开发中,iPad3、iPad mini2和iPhone4等等这些设备在读取图片时会优先选择文件名后缀是@2x的图片。比如,我们要为iPhone4上的应用添加一张背景图,我们会做一张640*960大小的图并命名为bkg@2x.png。如果是iPhone 3G设备,我们做一张320\*480的图片即可。运行时不同设备优先去加载相应分辨率的图片。(iPad3、iPad mini2和iPhone4等这些Retina设备图片资源与屏幕尺寸的比值为2,到了iPhone6 PLUS这个比值达到了2.46左右,加载的图片格式也变为了@3x)


#参见:

- [Cocos2d-x 多分辨率适配完全解析](http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/native/v3/multi-resolution/zh.md)
- [Cocos2d-JS的屏幕适配方案](http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/cocos2d-js/4-essential-concepts/4-4-resolution-policies/zh.md+)


