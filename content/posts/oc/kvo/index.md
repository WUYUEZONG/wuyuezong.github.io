---
title: "KVO（Key-Value Observing）键值监听"
# description: ""
date: 2022-11-14T16:08:48+08:00
draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---

**KVO是利用runtime的特性动态生成观察对象类的子类，然后重写被观察对象的属性的set方法。**

> 可以用于监听某个对象属性值的改
> 

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

# KVO的本质（内部实现）

## 打印对象isa地址、name类名

在设置监听前后打印person1，2的isa地址。发现person1在设置完监听后地址发生了改变；在设置监听前后打印person1，2的类名。发现person1在设置完监听后改变为：NSKVONotifying_Person；

![KVO%EF%BC%88Key-Value%20Observing%EF%BC%89%E9%94%AE%E5%80%BC%E7%9B%91%E5%90%AC%20e5bb7ae262a146a190def633869f8860/Untitled.png](KVO%EF%BC%88Key-Value%20Observing%EF%BC%89%E9%94%AE%E5%80%BC%E7%9B%91%E5%90%AC%20e5bb7ae262a146a190def633869f8860/Untitled.png)

## 小结

可以看出，程序在`runtime`的时候动态生成了 `NSKVONotifying_Person` ，`NSKVONotifying_Person` 继承于 `Person`，当改变 age 的值时，即调用 `setAge:` 方法。`NSKVONotifying_Person` 重写了 `setAge：`并做了其他事情。具体做了那些事可以猜测：

![KVO%EF%BC%88Key-Value%20Observing%EF%BC%89%E9%94%AE%E5%80%BC%E7%9B%91%E5%90%AC%20e5bb7ae262a146a190def633869f8860/Untitled%201.png](KVO%EF%BC%88Key-Value%20Observing%EF%BC%89%E9%94%AE%E5%80%BC%E7%9B%91%E5%90%AC%20e5bb7ae262a146a190def633869f8860/Untitled%201.png)

`didChangeValueForKey:`内部会调用`observer`的`observeValueForKeyPath:ofObject:change:context:`方法

<aside>
💡 **即：KVO是利用runtime的特性动态生成观察对象类的子类，然后重写被观察对象的属性的set方法。**

</aside>

## 如何手动触发KVO？

手动调用`willChangeValueForKey:` 和`didChangeValueForKey:` 。只调用`didChangeValueForKey:` 是无法触发的。

# 可能的使用场景（这几个都是什么意思？）

- 实现上下拉刷新控件 content offset
- webview 混合排版 content size
- 监听模型属性实时更新UI
- **NSOperation**
- **NSoperationQueue**
- **RAC**