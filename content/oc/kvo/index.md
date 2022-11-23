---
title: "KVO (Key-Value Observing)"
# description: ""
date: 2022-11-14T16:08:48+08:00
# draft: false
tags: ["KVO", "OC"]
series: ["Objc"]
series_order: 12

summary: "KVO是利用runtime的特性动态生成观察对象类的子类，然后重写被观察对象的属性的set方法。"
---

{{<alert>}}
KVO是利用runtime的特性动态生成观察对象类的子类，然后重写被观察对象的属性的set方法。
{{</alert>}}

> 可以用于监听某个对象属性值的改


```objectivec
// 如何设置属性的KVO
@property (nonatomic, strong) Person *person1;
- (void)viewDidLoad {
  [super viewDidLoad];
  _person1 = [[Person alloc] init];
  _person1.age = 32;
  // 需要返回的数据，新的值 ｜ 旧的值
  NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
  // 对person1属性age设置监听，附带信息：@"附加信息"
  [self.person1 addObserver:self forKeyPath:@"age" options:options context:@"附加信息"];
}
// person1.ag改变时进行调用这里
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
  NSLog(@"keyPath: %@, object: %@;\nchange: %@, context: %@", keyPath, object, change, context);
}
// 监听者销毁时移除相应的监听
-(void)dealloc {
  [_person1 removeObserver:self forKeyPath:@"age"];
}
```

## KVO的本质（内部实现）

### 打印对象isa地址、name类名

在设置监听前后打印person1，2的isa地址。发现person1在设置完监听后地址发生了改变；在设置监听前后打印person1，2的类名。发现person1在设置完监听后改变为：NSKVONotifying_Person；

```objc
// person1->isa: 0x10bcb0710
NSLog(@"person1->isa: %p", object_getClass(_person1));
// person2->isa: 0x10bcb0710
NSLog(@"person2->isa: %p", object_getClass(_person2));
// person1 class name: Person
NSLog(@"person1 class name: %p", object_getClass(object_getClass(_person1)));
// person2 class name: Person
NSLog(@"person2 class name: %p", object_getClass(object_getClass(_person2)));

NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
[self.person1 addObserver:self forKeyPath:@"age" options:options context:@"附加信息"];

// person1->isa: 0x6000022a03f0
NSLog(@"person1->isa: %p", object_getClass(_person1));
// person2->isa: 0x10bcb0710
NSLog(@"person2->isa: %p", object_getClass(_person2));
// person1 class name: NSKVONotifying_Person
NSLog(@"person1 class name: %p", object_getClass(object_getClass(_person1)));
// person2 class name: Person
NSLog(@"person2 class name: %p", object_getClass(object_getClass(_person2)));

// person1 class`s class name: Person
NSLog(@"person1 class`s class name: %@", object_getClass(object_getClass(person1)).superclass);
```

### 小结

可以看出，程序在`runtime`的时候动态生成了 `NSKVONotifying_Person` ，`NSKVONotifying_Person` 继承于 `Person`，当改变 age 的值时，即调用 `setAge:` 方法。`NSKVONotifying_Person` 重写了 `setAge：`并做了其他事情。具体做了那些事可以猜测：

```objc
- (void)setAge:(int)age {
  [_person1 willChangeValueForKey:@"age"];
  [super setAge:age];
  [_person1 didChangeValueForKey:@"age"];
}
```

`didChangeValueForKey:`内部会调用`observer`的`observeValueForKeyPath:ofObject:change:context:`方法

{{<alert>}}
即：KVO是利用runtime的特性动态生成观察对象类的子类，然后重写被观察对象的属性的set方法。
{{</alert>}}

### 如何手动触发KVO？

手动调用`willChangeValueForKey:` 和`didChangeValueForKey:` 。只调用`didChangeValueForKey:` 是无法触发的。

## 使用场景

- 实现上下拉刷新控件 content offset
- webview 混合排版 content size
- 监听模型属性实时更新UI
- **NSOperation**
- **NSoperationQueue**
- **RAC**