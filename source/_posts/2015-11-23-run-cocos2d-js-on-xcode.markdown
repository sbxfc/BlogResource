---
layout: post
title: "在Xcode上运行Cocos2dx-JS项目"
date: 2015-11-23 17:21:14 +0800
comments: true
categories: 
---

在Mac下通过命令生成Cocos2d-JS项目时,编译命令也会生成相应的Xcode集成环境,用Xcode打开下面目录下的工程就可以直接在真机或模拟器下运行调试:
	
	project
		/frameworks
			/runtime-src
				/proj.ios_mac
					/project.xcodeproj
					



使用Xcode 7.0以上版本会出现如下错误:

	error:-fembed-bitcode is not supported on versions of iOS prior to 6.0
	
修改方法,选择General->Deployment Target 6.0以上<br>
或者设置Build Settings->Build Options->Enabled Bitcode为NO。
