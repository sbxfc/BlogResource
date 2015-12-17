---
layout: post
title: "工厂设计模式"
date: 2015-01-13 18:38:01 +0800
comments: true
categories: 
---
工厂设计模式
---
---

编程中,当我们创建一个对象时,我们会选择以下方式实例化对象:

	Object* obj = new Object();
	
但当初始化工作并不像赋值操作一样简单,可能需要一段很长的代码去创建对象时,这样写就很难看了。

常常是,我们写了一段很长的初始化代码,当我们再次对其做修改时,可能会牵一发动全身。

还有一种情况，我们可能需要初始化的是一些Object的子类,比如SubObj1,SubObj2,...

按照面向接口编程,我们将Object抽象成一个接口,并实例化它们:

	Object obj1 = new SubObj1();
	Object obj2 = new SubObj2();
	....
	
随着项目的深入,可能会涵生出更多子类,SubObj3,SubObj4.... 那我们要对这些子类一个个实例化,可能更糟糕的是，我们还要对以前的代码进行修改。

工厂模式就是专门负责将大量有共同接口的类实例化,而且不必事先知道每次是要实例化哪一个类的模式。它定义一个用于创建对象的接口，由子类决定实例化哪一个类。

<!--more-->

一,简单工厂模式
---
---

工厂模式属于创建型模式，大致可以分为三类，简单工厂模式、工厂方法模式、抽象工厂模式。

首先是简单工厂模式,简单工厂模式的主要特点是需要在工厂类中做判断，从而创造相应的产品。

缺点:在增加新的类型时，需要修改工厂类。这就违反了开放封闭原则。

	class CarBase
	{
	public:
	    virtual ~CarBase(){};
	    virtual void show() = 0;
	};
	
	class SportsCar:public CarBase
	{
	public:
	    void show() {std::cout<<"制造了一辆跑车!"<<std::endl;}
	};
	
	class SUV:public CarBase
	{
	public:
	    void show() {std::cout<<"制造了一辆越野车!"<<std::endl;}
	};
	
	class Truck:public CarBase
	{
	public:
	    void show() {std::cout<<"制造了一辆卡车!"<<std::endl;}
	};
	
	class CarFactory
	{
	public:
	    CarBase * creatCar(CarType type)
	    {
	        switch(type)
	        {
	            case CarType_SportsCar:
	                return new SportsCar();
	                break;
	            case CarType_SUV:
	                return new SUV();
	                break;
	            case CarType_Truck:
	                return new Truck();
	                break;
	            default:break;
	        }
	    }
	};
	
	int main()
	{
	    CarFactory factory;
	    CarBase* car1 = factory.creatCar(CarType_SportsCar);
	    car1->show();
	    
	    CarBase* car2 = factory.creatCar(CarType_Truck);
	    car2->show();
	    
	    CarBase* car3 = factory.creatCar(CarType_SUV);
	    car3->show();
	}

- 输出:<br>
*制造了一辆跑车!<br>*
*制造了一辆卡车!<br>*
*制造了一辆越野车!<br>*
<br>

二,工厂方法模式
---
---

工厂方法(Factory Method)模式,是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类当中。核心工厂类不再负责产品的创建，这样核心类成为一个抽象工厂角色，仅负责具体工厂子类必须实现的接口，`这样进一步抽象化的好处是使得工厂方法模式可以使系统在不修改具体工厂角色的情况下引进新的产品。`

工厂方法模式是简单工厂模式的衍生，解决了许多简单工厂模式的问题。首先完全实现‘开－闭 原则’，实现了可扩展。其次更复杂的层次结构，可以应用于产品结果复杂的场合。

工厂方法模式对简单工厂模式进行了抽象。有一个抽象的Factory类（可以是抽象类和接口），这个类将不再负责具体的产品生产，而是只制定一些规范，具体的生产工作由其子类去完成。在这个模式中，工厂类和产品类往往可以依次对应。即一个抽象工厂对应一个抽象产品，一个具体工厂对应一个具体产品，这个具体的工厂就负责生产对应的产品。

工厂方法模式(Factory Method pattern)是最典型的模板方法模式(Template Method pattern)应用。

工厂方法模式角色与结构:

1. 抽象工厂(Creator)角色：是工厂方法模式的核心，与应用程序无关。任何在模式中创建的对象的工厂类必须实现这个接口。
 
2. 具体工厂(Concrete Creator)角色：这是实现抽象工厂接口的具体工厂类，包含与应用程序密切相关的逻辑，并且受到应用程序调用以创建产品对象。

3. 抽象产品(Product)角色：工厂方法模式所创建的对象的超类型，也就是产品对象的共同父类或共同拥有的接口。 

4. 具体产品(Concrete Product)角色：这个角色实现了抽象产品角色所定义的接口。某具体产品有专门的具体工厂创建，它们之间往往一一对应。

工厂方法经常用在以下两种情况中:

1. 第一种情况是对于某个产品，调用者清楚地知道应该使用哪个具体工厂服务，实例化该具体工厂，生产出具体的产品来。

2. 第二种情况，只是需要一种产品，而不想知道也不需要知道究竟是哪个工厂为生产的，即最终选用哪个具体工厂的决定权在生产者一方，它们根据当前系统的情况来实例化一个具体的工厂返回给使用者，而这个决策过程这对于使用者来说是透明的。

缺点:每增加一种产品,就需要增加一个对象的工厂。如果增加许多新类型,就要创建相应的工厂。相比于简单工厂模式,工厂方法模式需要更多的类定义。而且随着产品类型的增多,增加了结构复杂度。`相比于简单工厂模式只需要维护一个类,工厂方法模式增加了维护的复杂度。`
	
	//抽象产品类
	class CarBase
	{
	public:
	    virtual ~CarBase(){};
	    virtual void show() = 0;
	};
	
	class SportsCar:public CarBase
	{
	public:
	    void show() {std::cout<<"建造了一辆跑车!"<<std::endl;}
	};
	
	class SUV:public CarBase
	{
	public:
	    void show() {std::cout<<"建造了一辆越野车!"<<std::endl;}
	};
	
	class Truck:public CarBase
	{
	public:
	    void show() {std::cout<<"建造了一辆卡车!"<<std::endl;}
	};
	
	//抽象工厂类
	class FactoryBase
	{
	public:
	    virtual CarBase * creatCar() = 0;
	};
	
	class SportsCarFactory:public FactoryBase
	{
	public:
	    CarBase * creatCar()
	    {
	        return new SportsCar();
	    }
	};
	
	class TruckFactory:public FactoryBase
	{
	public:
	    CarBase * creatCar()
	    {
	        return new Truck();
	    }
	};
	
	class SUVFactory:public FactoryBase
	{
	public:
	    CarBase * creatCar()
	    {
	        return new SUV();
	    }
	};
	
	
	
	int main()
	{
	    SportsCarFactory factory1;
	    CarBase* car1 = factory1.creatCar();
	    car1->show();
	    
	    SUVFactory factory2;
	    CarBase* car2 = factory2.creatCar();
	    car2->show();
	    
	    TruckFactory factory3;
	    CarBase* car3 = factory3.creatCar();
	    car3->show();
	}
	
- 输出:<br>
*建造了一辆跑车!<br>*
*建造了一辆越野车!<br>*
*建造了一辆卡车!<br>*
<br>


抽象工厂模式
---
---

所谓的抽象工厂是指一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象。

设想一下,当我们要在为不同操作系统创建显示控件时。Win 系统要创建WinButton、WinTex、...而Unix操作系统里会创建UnixButton、UnixText、...等一系列按钮。此时,Button和Text属于不同的抽象产品,而MacButton 和 MacText则属于同一个产品族。我们使用抽象工厂Factory来创建同一族的产品,WinUIFactory创建Win操作系统下的按钮。而MacUIFactory创建Mac操作系统下的按钮。

缺点:抽象工厂模式是工厂方法模式的升级版，继承了工厂方法模式结构复杂的缺点。同时,当有增加新的产品族的时候,就需要想改相应的核心工厂机构。

![icon](/images/2015/1/factorydesign1.png)

	#include <iostream>
	
	class ButtonBase
	{
	public:
	    virtual ~ButtonBase(){};
	    virtual void show() = 0;
	};
	
	class WinButton:public ButtonBase
	{
	public:
	    void show() {std::cout<<"Win button."<<std::endl;}
	};
	
	class MacButton:public ButtonBase
	{
	public:
	    void show() {std::cout<<"Mac button."<<std::endl;}
	};
	
	
	class TextBase
	{
	public:
	    virtual ~TextBase(){};
	    virtual void show() = 0;
	};
	
	class WinText:public TextBase
	{
	public:
	    void show() {std::cout<<"Win Text."<<std::endl;}
	};
	
	class MacText:public TextBase
	{
	public:
	    void show() {std::cout<<"Max Text."<<std::endl;}
	};
	
	
	class FactoryBase
	{
	public:
	    virtual ButtonBase * creatButton() = 0;
	    virtual TextBase * creatText() = 0;
	};
	
	class WinUIFactory:public FactoryBase
	{
	public:
	    ButtonBase * creatButton()
	    {
	        return new WinButton();
	    }
	    
	    TextBase * creatText()
	    {
	        return new WinText();
	    }
	};
	
	class MacUIFactory:public FactoryBase
	{
	public:
	    ButtonBase * creatButton()
	    {
	        return new MacButton();
	    }
	    
	    TextBase * creatText()
	    {
	        return new MacText();
	    }
	};
	
	
	int main()
	{
	    WinUIFactory factory1;
	    ButtonBase* btn1 = factory1.creatButton();
	    btn1->show();
	    TextBase* text1 = factory1.creatText();
	    text1->show();
	    
	    MacUIFactory factory2;
	    ButtonBase* btn2 = factory2.creatButton();
	    btn2->show();
	    TextBase* text2 = factory2.creatText();
	    text2->show();
	    
	    delete btn1;
	    delete text1;
	    delete btn2;
	    delete text2;
	    return 0;
	}

- 输出:<br>*Win button.<br>
Win Text.<br><br>
Mac button.<br>
Max Text.*