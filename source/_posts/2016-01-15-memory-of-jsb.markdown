---
layout: post
title: "Cocos2d-JS JSB内存管理"
date: 2016-01-15 14:42:31 +0800
comments: true
categories: 
---

如果你写的Cocos2d-JS程序在iOS/Android原生环境里运行时出现 <font color='#bd260d'>Error : Invalid Native Object</font> ,那么你有可能遇到了JSB的内存问题。

我们知道,Cocos2d-JS在原生环境里运行时,实际上运行的是Cocos2dx引擎而非Cocos2d-HTML5。在程序执行时Javascript通过JSB调用Cocos2dx引擎的API,因为 Javascript 本身有一套垃圾回收机制,Cocos2dx也有一套,当这两者一起运行时,就会由于沟通问题出现一些差错。

Invalid Native Object 的错误原因是,JS端访问了一个已经释放了相应C++绘制对象的JS对象。官网列举了一个这样的示例。首先,在程序里创建了一个全局变量globalNode:
	
	var globalNode = new cc.Node();
	...
	onTouched:function(sender){
		sender.addChild(globalNode);
	}

当程序运行时,globalNode没有通过 addChild 函数添加场景上,而是在按钮事件onTouched触发时才被添加到场景上。当按钮点击时出现下列错误:
	
><font color='#bd260d'>*jsb: ERROR: File /Users/sbxfc/Documents/XXX/frameworks/cocos2d-x/cocos/scripting/js-bindings/
auto/jsb_cocos2dx_auto.cpp: Line: 1973, Function: js_cocos2dx_Node_addChild<br>
Invalid Native Object*</font> 

按照我们一贯的思路,可能察觉不出这个程序的问题。我们通过CCNode.js里一段注释作为引子来解析这段程序错误的原因:

> If you created an engine object and haven't added it into the scene graph during the same frame.JSB's native autorelease pool will consider this object a useless one and release it directly

当程序当前帧绘制完成之后,那些没有被添加到场景上的绘制对象将被删除并进行垃圾回收,所以globalNode在C++端的对象会被移除,按正常逻辑globalNode也将被回收,但是由于globalNode是一个全局对象不能被回收,所以就出现了上述错误。

#cc.Node的生命周期

为了更清除地了解整个过程,我们看一下Node对象被创建和释放的整个流程。首先,我们使用 cc.Node.create() 来创建对象时会执行以下操作:
	
首先,在游戏启动的时候(applicationDidFinishLaunching),游戏引擎会通过addRegisterCallback将JS代码往C++的映射:
	
	static JSFunctionSpec st_funcs[] = {
    	JS_FN("create", js_cocos2dx_CCNode_create, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    	JS_FS_END
	};

	jsb_CCNode_prototype = JS_InitClass(
    	cx, global,
    	NULL, // parent proto
    	jsb_CCNode_class,
    	js_cocos2dx_CCNode_constructor, 0, // constructor
    	properties,
    	funcs,
    	NULL, // no static properties
    	st_funcs);
    	
cc.Node.create() 会被隐射到 C函数 js_cocos2dx_CCNode_create()上,该函数的结构如下:

	JSBool js_cocos2dx_CCNode_create(JSContext *cx, uint32_t argc, jsval *vp)
	{
	    if (argc == 0) {
	        cocos2d::CCNode* ret = cocos2d::CCNode::create();
	        jsval jsret;
	        do {
	        if (ret) {
	            js_proxy_t *proxy = js_get_or_create_proxy(cx, ret);
	            jsret = OBJECT_TO_JSVAL(proxy->obj);
	        } else {
	            jsret = JSVAL_NULL;
	        }
	    } while (0);
	        JS_SET_RVAL(cx, vp, jsret);
	        return JS_TRUE;
	    }
	    JS_ReportError(cx, "wrong number of arguments");
	    return JS_FALSE;
	}

在该函数里,通过调用 js_cocos2dx_CCNode_create 来创建了一个js_proxy_t类型的对象 proxy,js_cocos2dx_CCNode_create 是通过下面的SpiderMonkey API 完成调用的:
	
	JS_AddObjectRoot(cx, &proxy->obj);
	
JS_AddObjectRoot 将 JSObject 添加到垃圾回收中,proxy->obj将JSObject隐射到Javascript中,也就是说,最后proxy得到了JS端对象(cc.Node)的访问权限。

cc.Node.create()创建的对象最终在Cocos2dx端被保存到内存里,并且通过JS_RemoveObjectRoot来移除JS端对象的引用。

当cocos2d::CCNode对象被移除时,会在下一帧自动释放掉,接着CCObject的析构函数会被调用,代码如下:

	// if the object is referenced by Lua engine, remove it
	if (m_nLuaID)
	{
	    CCScriptEngineManager::sharedManager()->getScriptEngine()->removeScriptObjectByCCObject(this);
	}
	else
	{
	    CCScriptEngineProtocol* pEngine = CCScriptEngineManager::sharedManager()->getScriptEngine();
	    if (pEngine != NULL && pEngine->getScriptType() == kScriptTypeJavascript)
	    {
	        pEngine->removeScriptObjectByCCObject(this);
	    }
	}
	
	
析构函数触发了pEngine->removeScriptObjectByCCObject,而该函数做了下面这些事情:
	
	void ScriptingCore::removeScriptObjectByCCObject(CCObject* pObj)
	{
	    js_proxy_t* nproxy;
	    js_proxy_t* jsproxy;
	    void *ptr = (void*)pObj;
	    nproxy = jsb_get_native_proxy(ptr);
	    if (nproxy) {
	        JSContext *cx = ScriptingCore::getInstance()->getGlobalContext();
	        jsproxy = jsb_get_js_proxy(nproxy->obj);
	        JS_RemoveObjectRoot(cx, &jsproxy->obj);
	        jsb_remove_proxy(nproxy, jsproxy);
	    }
	}
	
其中,函数JS_RemoveObjectRoot被调用,JS端对象引用被移除,jsb_remove_proxy将会移除掉哈希表中的映射关系。至此,Cocos2d-x完成了整个对象的垃圾回收管理。

#retain()和release()

在上面的问题里,cc.Node的引用对象是全局变量所以没有被释放掉,而其绘制对象cocos2d::CCNode却被垃圾回收了。 

为了避免出现 Invalid Native Object 的情形,我们可以使用retain()函数来使Cocos2dx端的绘制对象增加引用计数,我们知道C++端是通过引用计算来控制内存管理的。当我们需要释放时,只需要手动调用release()即可。但是需要注意的是,retain和release是一一对应关系,retain之后必须要release一下,不然会造成内存泄露。

#其他

除了全局变量以外,还有一些情形也会导致JS端的对象无法被释放掉。下面是另一个错误示例,首先,创建了一个node对象,node在当前帧没有被添加到场景里,而是在一秒钟(1000ms)后被添加到场景里:

    var self = this;
	this.node = new cc.Node();
    setTimeout(function(){
        self.node.setLocalZOrder(1);
    },1000);

当改程序运行时,会出现 Invalid Native Object 错误:
	
<font color='#bd260d'>*File /Users/sbxfc/Documents/cocos/projects/demo/frameworks/cocos2d-x/cocos/scripting/js-bindings/auto/jsb_cocos2dx_auto.cpp: Line: 4186, Function: js_cocos2dx_Node_setLocalZOrder js_cocos2dx_Node_setLocalZOrder : Invalid Native Object*</font>
	

理解该示例错误的关键在于JS里的参数传递和闭包。setTimeout形成的闭包会在程序运行时异步执行,其中self.node通过匿名参数的形式传递到函数内部,而函数参数传递本身是一种值传递,其传递的是对象引用的副本。所以Cocos2dx无法释放setTimeout函数里的self.node,最终也会导致Invalid Native Object错误。

当然,上面的问题也可以使用retain、release对解决。CCNode.js里关于函数retain、release有详细的注释,开发者可以借此了解到有关释放问题和JSB机制不健全的说明,并以此理解引擎开发者的困扰。

#参见
- [JS是按值传递还是按引用传递?](http://bosn.me/js/js-call-by-sharing/)
- [Javascript Binding](http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/cocos2d-js/1-about-cocos2d-js/1-1-a-brief-history/zh.md)
- [JSB内存管理](http://www.cocos.com/doc/article/index?type=wiki&url=/doc/cocos-docs-master/manual/framework/native/wiki/memory-management-of-jsb/zh.md)
