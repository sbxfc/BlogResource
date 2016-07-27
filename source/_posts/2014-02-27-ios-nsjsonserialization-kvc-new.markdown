---
layout: post
title: "iOS - JSON解析 & KVC"
date: 2014-02-27 10:24:06 +0800
comments: true
categories: 
---

#NSJSONSerialization

解析JSON字符串:

		NSString* jsonString = @"[{\"name\":\"Paul McCartney\",\"age\":73,\"sex\":\"male\"},{\"name\":\"Mark\",\"age\":53,\"sex\":\"male\"}]";
		NSData* jsonData = [jsonString dataUsingEncoding:NSUTF8StringEncoding];
	
		//解析
		NSError* error;
		NSArray * jsonArray = [NSJSONSerialization JSONObjectWithData: jsonData options: NSJSONReadingMutableContainers error: &error];	
- NSJSONReadingMutableContainer：指定创建的可变对象为NSArray或NSDictionary。
- NSJSONReadingMutableLeaves：指定JSON树上的子数据为NSMutableString类型的对象
- NSJSONReadingAllowFragments：指定JSON顶层的对象可以不是NSArray或NSDictionary类型的对象

转化成JSON:

		if([NSJSONSerialization isValidJSONObject:jsonArray])
		{
			jsonData = [NSJSONSerialization dataWithJSONObject:jsonArray options:NSJSONWritingPrettyPrinted error:&error];
			jsonString =[[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];	
			
			NSLog(@"json data:%@",jsonString);
		}


对象转化为JSON必须符合以下条件:

- 1,顶层对象必须是NSArray或NSDictionary
- 2,所有对象必须是NSString, NSNumber, NSArray, NSDictionary, 或 NSNull.
- 3，所有dictionary的key必须是NSString。
- 4, Numbers不能是NaN或infinity。
	
转化JSON时用参数NSJSONWritingPrettyPrinted的作用是在输出时增加空格，增加输出内容的可读性。
#KVC

KVC 即 Key-value coding,允许我们通过属性的字符串名称来访问属性。

假如,我们声明一个属性 name:

		@property(nonatomic,copy) NSString *name;

我们可以通过一下函数访问属性:
	
		//取值
		NSString *name = [obj valueForKey:@"name"]
		//赋值
		[obj setValue:@"goodman" forKey:@"name"]
	

还可以通过 KVC 将 NSDictionary 的键值映射到类中相同名称的属性上。 使用 setValuesForKeysWithDictionary 函数进行映射:

Person.h

	@interface Person : NSObject
	@property (nonatomic, copy) NSString *name;
	@property (nonatomic) NSUInteger age;
	@end

Player.m
	
	-(id)initWith:(NSDictionary*)data		
	{
		if(self=[super init]){
			[self setValuesForKeysWithDictionary:data];
		}
		return self;
	}

如果 NSDictionary 里的键值没有对应到类中的属性上,运行时会出现下面这样的错误提示：

'**NSUnknownKeyException**'……this class is not key value coding-compliant for the key sex.'

此时,需要重写setUndefinedKey方法来忽略错误:

	-(void)setValue:(id)valueforUndefinedKey:(NSString*)key
	{
		//NSLog(@"undefinedkey:%@",key);
	}

