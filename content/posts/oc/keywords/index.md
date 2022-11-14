---
title: "Keywords"
# description: ""
date: 2022-11-14T16:16:52+08:00
draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---
# strong / weak / copy(ARC环境)

> Automatic Reference Counting (ARC) is a compiler feature that provides automatic memory management of Objective-C objects. Rather than having to think about retain and release operations, ARC allows you to concentrate on the interesting code, the object graphs, and the relationships between objects in your application.
> 

**Automatic Reference Counting(ARC)技术是用于OC对象的内存管理。即在适当的时候对OC对象retain和release操作。**

[ARC修饰符](strong%20weak%20copy(ARC%E7%8E%AF%E5%A2%83)%20c51150e241594ed99473f2facaaa1bbd/ARC%E4%BF%AE%E9%A5%B0%E7%AC%A6%20f1a22bfbb8c8452791086ec30cc4772c.csv)

[常用修饰符作用](strong%20weak%20copy(ARC%E7%8E%AF%E5%A2%83)%20c51150e241594ed99473f2facaaa1bbd/%E5%B8%B8%E7%94%A8%E4%BF%AE%E9%A5%B0%E7%AC%A6%E4%BD%9C%E7%94%A8%20224e4a6e903d4b67b74983524d7f3a58.csv)

# strong是强引用，会持有对象，引用计数器会+1

`strong`是为了告诉编译器（xcode），被`strong`修饰的对象是强引用，需要`retain`操作，引用计数器会`+1`，默认情况下声明变量都是隐式`strong`申明。

- **默认情况下`strArr`引用`testArr`打印输出**

```objectivec

NSArray *testArr = @[@"a", @"b"];
NSArray *strArr = testArr;

testArr = nil;
NSLog(@"print str is %@", strArr);
    
// 打印结果
2021-05-30 15:33:14.931372+0800 Strong&Weak[3480:197342] print str is (
    a,
    b
)
```

- `**__strong`修饰情况下`strArr`引用`testArr`打印输出**

```objectivec
NSArray *testArr = @[@"a", @"b"];
__strong NSArray * strongStrArr = testArr;

testArr = nil;
NSLog(@"print strongStr is %@", strongStrArr);

// 打印结果
2021-05-30 15:37:30.308564+0800 Strong&Weak[3551:201548] print strongStr is (
    a,
    b
)
```

- **`__weak`修饰情况下`strArr`引用`testArr`打印输出**

```objectivec
NSArray *testArr = @[@"a", @"b"];
__weak NSArray * weakStrArr = testArr;

testArr = nil;
NSLog(@"print weakStr is %@", weakStrArr);

// 打印结果
2021-05-30 15:39:38.772768+0800 Strong&Weak[3579:203646] print weakStr is (null)
```

**小结**

对比以上结果，可以看出不使用`__strong`修饰和使用`__strong`结果一样，说明oc默认情况下声明变量都是`__strong`，因为`weak`修饰引用计数器`不会+1`，所修饰的对象可能随时被释放。

# weak是弱引用，不会持有对象，引用计数器不会+1

常用于修饰UI控件、delegate

声明为`weak`的指针，`weak`指针指向的对象一旦被释放，`weak`的指针都将被赋值为`nil`，防止野指针。

# copy分为浅拷贝、深拷贝

修饰NSString、block

`stackblock`如果不`copy`的话，`stackblock`是存放在栈里面的，他的生命周期会随着函数的结束而出栈的，`copy`之后会转变为`mallocblock`放在堆里面。

# assign简单赋值，不改变引用计数。

- 基础数据类型`（NSInteger、CGFloat）`
- C数据类型`（int、float、double、char等）`
- `枚举、结构体`等非OC对象