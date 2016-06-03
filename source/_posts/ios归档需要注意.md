---
title: iOS归档之NSKeyedArchiver
date: 2016-04-29 14:01:52
tags: iOS 
---

如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，可以直接用NSKeyedArchiver进行归档和恢复。

不是所有的对象都可以直接用这种方法进行归档，只有遵守了NSCoding协议的对象才可以。

NSCoding协议有2个方法：

	encodeWithCoder:
	
每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用encodeObject:forKey:方法归档实例变量

	initWithCoder:
	
每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用decodeObject:forKey方法解码实例变量

