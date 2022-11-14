---
title: "+initialize"
date: 2022-06-30


tags: ["OC"]
series: ["OC"]
series_order: 11
---

{{<alert>}}
initialize是类第一次接收到消息时调用的( [类 alloc]的时候)，每个类只会调用initialize一次（父类的initialize方法可能会被多调用），initialize是通过objc_msgSend调用的。
{{</alert>}}



## initialize调用顺序

1. 先出初始化父类
2. 再初始化子类（可能最终调用的是父类的initialize方法，因为是通过isa指针，superclass指针去寻找方法调用的）

## initialize底层实现伪代码

```objectivec
@interface Person
@end
@interface Student: Person
@end
void lookUpImpOrNil() {
    //Student类没有初始化
    if !student {
        //Person类没有初始化
        if !person {
            objc_msgSend([Person class],@seletor(initialize))
        }
        objc_msgSend([Student class],@seletor(initialize))
    }
}
```

## 阅读源码（objc4）

1. objc-msg-arm64.s
    1. objc_msgSend
2. objc-runtime-new.mm
    1. class_getInstanceMethod
    2. lookUpImpOrNil
    3. lookUpImpOrForward
    4. _class_initialize
    5. callInitialize
    6. objc_msgSend(cls, SEL_initialize)