---
layout: post
title: "Cocos - Spine骨骼动画"
date: 2015-11-25 14:14:58 +0800
comments: true
categories: 
---

Spine是一款出众的2D的骨骼动画编辑器。

#使用步骤

1.首先,在SETUP模型下选中右侧的Images标签页,然后点击Browse将包含图片资源的文件夹添加进来。

2.将Images下的图片拖入场景,并按照角色的样子将各个部位拼接起来。可通过右侧Draw Order调整图片的绘制顺序;

3.创建骨骼,并绑定图片到骨骼上,注意各骨骼的父子关系。

4.切换到ANIMATE模式,选中要“动”的骨骼,对其进行旋转、移动、缩放等操作,每次改动后要记得打关键帧。 

5.选择菜单中的Texture Packer选项,导出Cocos2d-x工程需要的json文件和atlas以及png合图。

#在Cocos2d-x里使用

首先,包含Spine动画需要的相关头文件:

	#include <spine/spine-cocos2dx.h>
	#include "spine/spine.h"
	using namespace spine;

然后,使用SkeletonAnimation创建Spine动画

	auto skeletonNode = new SkeletonAnimation("player.json", "player.atlas");
    skeletonNode->setAnimation(0, "walk", true);
    skeletonNode->debugBones = false;
    skeletonNode->setPosition(CCRANDOM_0_1() * windowSize.width, 0 + CCRANDOM_0_1() * windowSize.height);
    addChild(skeletonNode);

#参见:

- <http://zh.esotericsoftware.com/spine-in-depth>
- <http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/native/v3/spine/zh.md>



