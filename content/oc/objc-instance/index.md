---
title: "Objc Instance"
# description: ""
date: 2022-11-14T16:10:35+08:00
# draft: true
tags: ["OC", "Objc Instance"]
series: ["Objc"]
series_order: 5

summary: "实例对象、类对象、元类对象（Instance \ Class \ Meta-Class）"
---


---

|oc对象|在内存中存储的东西|
|-|-|
|instance|isa, _property(value)|
|class|isa, property, function, protocol, _property, superclass|
|meta-class|isa, superclass, class function|

# instance对象（实例对象）

```objectivec
// obj1，obj2 是NSObject的instance对象（实例对象）
// 他们不同的两个对象，分别占据着两个不同的内存。
NSObject *obj1 = [[NSObject alloc] init];
NSObject *obj2 = [[NSObject alloc] init];
```

## instance对象在内存中存储的信息包括

|instance|
|---|
|---  isa指针|
|---  其他成员变量（这里指的是变量的值，**比如变量age = 4，存储这个4**）|


# class对象（类对象）

```objectivec
// obj1，obj2 是NSObject的instance对象（实例对象）
// 他们不同的两个对象，分别占据着两个不同的内存。
NSObject *obj1 = [[NSObject alloc] init];
NSObject *obj2 = [[NSObject alloc] init];
// objClass1 ~ objClass5 都是NSObject的class对象（类对象）
// 他们是同一个对象，每个类在内存中有且只有一个class对象
Class objClass1 = [obj1 class];
Class objClass2 = [obj2 class];
Class objClass3 = [NSObject class];
Class objClass4 = object_getClass(obj1);
Class objClass5 = object_getClass(obj2);
```

## class对象在内存中存储的信息包括

|class|
|---|
|--- isa指针|
|--- superclass指针|
|--- 类的属性信息（@property）、类的对象方法信息（instance method）|
|--- 类的协议信息（protocol）、类的成员变量信息（ivar，这里主要是变量名不会发生改变的信息，变量类型）|

## meta-class对象（元类对象）

```objectivec
// objMetaClass 是NSObject的meta-class对象（元类对象）
// 每个类在内存中有且只有一个meta-class对象
Class objMetaClass = object_getClass([NSObject class]); // runtime api
```

## meta-class对象和class对象的内存结构是一样的，只是用途不一样


|meta-class|
|---|
|--- isa指针|
|--- superclass指针|
|--- 类的类方法信息（class method）|
|--- ...|

## 注意

```objectivec
//objectClass是class对象，并不是meta-class对象
Class objClass = [[NSObject class] class];
// 查看Class是否是meta-class
#import <objc/runtime.h>
BOOL result = class_isMetaClass([NSObject class]);
```

isa指针