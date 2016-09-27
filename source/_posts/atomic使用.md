---
title: atomic使用
date: 2016-07-18 15:41:32
tags:
---

平时我们使用atomic的时候，是希望对一个porperty的操作进行一个原子操作，但是这里的原子操作应该是仅仅对应于Setter和Getter方法，如果property是一个array，比如


- (void)setProp:(NSString *)newValue {
    [_prop lock];
    _prop = newValue;
    [_prop unlock];
}
按我理解：
1.此处的线程安全是就getter,setter而言的。比如对于@property(nonatomic,copy)NSString *str; 当调用self.str = @"HELLO,GUY";如果是多线程，在一个线程执行setter方法的时候，会涉及到字符串拷贝，另一个线程去读取，很可能读到一半的数据，也就是garbage数据。
2.另外的话，它也仅限于getter,setter时的线程安全。比如@property(atomic,strong)NSMutableArray *arr;如果一个线程循环读数据，一个线程循环写数据，肯定会产生内存问题。因为它和setter,getter没有关系。 