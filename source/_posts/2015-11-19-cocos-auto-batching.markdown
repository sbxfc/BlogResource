---
layout: post
title: "cocos2d - 批次绘制"
date: 2015-11-19 16:52:57 +0800
comments: true
categories: 
---

#自动批绘制

从3.0版本以后,cocos2d的图形渲染模块实现了自动批绘制(Auto-batching)的优化。引擎的渲染部分对相同材质的连续渲染指令(QuadCommand)进行过滤,如果当前的渲染对象的材质和上一次渲染的材质一样,就不渲染了,保存一下所需的信息,继续遍历下一个,直到发现当前材质和上一个材质不一样，才重新开始渲染。

这样一来,渲染指令里相邻的且材质ID相同的多个draw call就能减少为一个,过程如下图:

![](/images/2015/10/auto-batching.png)

最简单的满足条件就是我们用同一张图片连续创建精灵,这些精灵在绘制时只需要一个draw call:

	for (int i = 0; i < 10000; i++)
	{
        Sprite* sprite = Sprite::create("HelloWorld.png");
        sprite->setPosition(Point(CCRANDOM_0_1() * winSize.width, CCRANDOM_0_1() * winSize.height));
        this->addChild(sprite);
    }
    
需要注意的是,在渲染开始前引擎会对渲染指令根据zOrder进行一次排序。所以,连续添加的相同Sprite需要保证期zOrder是一样的。当然,我们也可以利用zOrder排序的这个机制,对非连续添加的相同纹理精灵设置相同zOrder来满足自动批绘制的条件:

	for(int i = 0; i < 10000; i++)
	{
	    Sprite* sprite1 = Sprite::create("CloseNormal.png");
	    sprite1->setPosition(Point(CCRANDOM_0_1() * winSize.width, 0 + CCRANDOM_0_1() * winSize.height));
	    this->addChild(sprite1);
	
	    Sprite* sprite2 = Sprite::create("CloseSelected.png");
	    sprite2->setPosition(Point(CCRANDOM_0_1() * winSize.width, 0 + CCRANDOM_0_1() * winSize.height));
	    this->addChild(sprite2);
	    sprite2->setZOrder(1);
	}
	
由于,Auto-batching的使用条件是使用QuadCommands渲染命令的连续的相同材质ID的Sprite或ParticleSystem objects。所以,我们也可以使用精灵帧表单来实现Auto-batching。

	SpriteFrameCache::getInstance()->addSpriteFramesWithFile("TankMovePlist.plist");
    for(int i = 0; i < 10000; i++)
    {
        char buf[64];
        sprintf(buf,"image/M26_b_%d.png", i%8 + 4);
        SpriteFrame *frame= SpriteFrameCache::getInstance()->getSpriteFrameByName(buf);
        Sprite *sprite = Sprite::createWithSpriteFrame(frame);
        sprite->setPosition(Point(CCRANDOM_0_1() * winSize.width, 0 + CCRANDOM_0_1() * winSize.height));
        this->addChild(sprite);
    }

进而将Auto-Batching用在动画上:

	SpriteFrameCache::getInstance()->addSpriteFramesWithFile("TankMovePlist.plist");
    for(int i = 0; i < 700; i++)
    {
        auto s_tank = Sprite::createWithSpriteFrameName("image/M26_b_4.png");
        
        auto tankMoveAnimation = Animation::create();
        tankMoveAnimation->setDelayPerUnit(0.1);
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_4.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_5.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_6.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_7.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_8.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_9.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_10.png"));
        tankMoveAnimation->addSpriteFrame(SpriteFrameCache::getInstance()->getSpriteFrameByName("image/M26_b_11.png"));
        s_tank->runAction(RepeatForever::create(Animate::create(tankMoveAnimation)));
        s_tank->setPosition(CCRANDOM_0_1() * winSize.width, 0 + CCRANDOM_0_1() * winSize.height);
        addChild(s_tank);
    }

运行效果:

![](/images/2015/10/tank_move.png)

#使用SpriteBatchNode绘制精灵

和Auto-batching的对绘制优化的策略一样,SpriteBatchNode将多个精灵放到一个纹理上,绘制的时候直接统一绘制该texture，不需要单独绘制子节点。

	SpriteBatchNode* batchNode = SpriteBatchNode::create("HelloWorld.png", 10000);
    addChild( batchNode);
    
    for ( int i = 0; i < 10000; ++i)
    {
        Sprite* sprite = Sprite::createWithTexture( batchNode->getTexture());
        sprite->setPosition(Point(CCRANDOM_0_1() * winSize.width, 0 + CCRANDOM_0_1() * winSize.height));
        batchNode->addChild( sprite);
    }

再添一个官网文档上提供的例子:

	auto batch = SpriteBatchNode::create("Images/grossini_dance_atlas.png", 1);
    addChild(batch, 0, kTagSpriteBatchNode);        

    auto sprite1 = Sprite::createWithTexture(batch->getTexture(), Rect(85*0, 121*1, 85, 121));
    auto sprite2 = Sprite::createWithTexture(batch->getTexture(), Rect(85*1, 121*1, 85, 121));
    
    auto s = Director::getInstance()->getWinSize();
    sprite1->setPosition( Point( (s.width/5)*1, (s.height/3)*1) );
    sprite2->setPosition( Point( (s.width/5)*2, (s.height/3)*1) );

    batch->addChild(sprite1, 0, kTagSprite1);
    batch->addChild(sprite2, 0, kTagSprite2);
    
#参见:

- <https://github.com/chukong/cocos-docs/blob/master/manual/framework/native/v3/auto-batching/zh.md>
- <http://blog.csdn.net/chenqiai0/article/details/46820669>
- [在cocos2d-js上运行动画](http://www.cocos2d-x.org/docs/tutorial/framework/html5/parkour-game-with-javascript-v3.0/chapter5/zh)