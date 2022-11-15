---
title: "Runtime"
# description: ""
date: 2022-11-14T16:12:53+08:00
draft: true
tags: ["OC", "Runtime"]
series: ["Objc"]
series_order: 14

# summary: ""
---


知识关联

[位运算（& | ^ ~ << >>）](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/%E4%BD%8D%E8%BF%90%E7%AE%97%EF%BC%88&%20%5E%20~%20%EF%BC%89%206fe8ca52bdd64281aba3c83a72fdd93f.md)

[共用体](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/%E5%85%B1%E7%94%A8%E4%BD%93%20cc00ed531158465b8e915cc477f47d54.md)

# **isa指针**

在arm64架构之前，`isa`就是一个普通的指针，存储着`Class`、`Meta-Class`对象的内存地址，从arm64架构开始，对isa进行了优化，变成了一个共用体（`union`）结构，还使用位域来存储更多的信息。

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled.png)

## nonpointer

- 0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
- 1，代表优化过，使用位域存储更多的信息

## has_assoc

- 是否有设置过关联对象，如果没有，释放时会更快

## has_cxx_dtor

- 是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快

## shiftcls

- 存储着Class、Meta-Class对象的内存地址信息

## magic

- 用于在调试时分辨对象是否未完成初始化

## weakly_referenced

- 是否有被弱引用指向过，如果没有，释放时会更快

## deallocating

- 对象是否正在释放

## extra_rc

- 里面存储的值是引用计数器减1

## has_sidetable_rc

- 引用计数器是否过大无法存储在isa中
- 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中

# Class的结构

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%201.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%201.png)

- `class_rw_t`里面的`methods`、`properties`、`protocols`是二维数组，是可读可写的，包含了类的初始内容、分类的内容

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%202.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%202.png)

- `class_ro_t`里面的`baseMethodList`、`baseProtocols`、`ivars`、`baseProperties`是一维数组，是只读的，包含了类的初始内容

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%203.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%203.png)

- `method_t`是对方法\函数的封装

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%204.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%204.png)

- `**IMP**`代表函数的具体实现

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%205.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%205.png)

- **`SEL`**代表方法\函数名，一般叫做选择器，底层结构跟`**char ***`类似
    - 可以通过`@selector()`和`sel_registerName()`获得
    - 可以通过`sel_getName()`和`NSStringFromSelector()`转成字符串
    - 不同类中相同名字的方法，所对应的方法选择器是相同的

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%206.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%206.png)

- `types`包含了函数返回值、参数编码的字符串

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%207.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%207.png)

iOS中提供了一个叫做@encode的指令，可以将具体的类型表示成字符串编码

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%208.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%208.png)

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%209.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%209.png)

[ObjC中的TypeEncodings](https://juejin.cn/post/6844903606403989517)

# 方法缓存

`Class`内部结构中有个方法缓存（`cache_t`），用散列表（哈希表）来缓存曾经调用过的方法，可以提高方法的查找速度。

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2010.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2010.png)

缓存查找

```objectivec
objc-cache.mm
bucket_t * cache_t::find(cache_key_t k, id receiver)
```

## 散列表（哈希表）

数组提前申请固定空间，通过某个key运算得到index，然后然后将值存储到对应index，如果该index有值，index-1操作，直到有空间，满了之后清空扩容。【**牺牲内存（空间）换取时间**】

# OC中调用方法其实都是执行objc_msgSend

<aside>
💡 objc_msgSend**执行流程**

</aside>

## 消息发送

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2011.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2011.png)

- `receiver`通过`isa指针`找到`receiverClass`
- `receiverClass`通过`superclass`指针找到`superClass`
- 如果是从`class_rw_t`中查找方法
    - 已经排序的，二分查找
    - 没有排序的，遍历查找

## 动态方法解析

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2012.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2012.png)

- 开发者可以实现以下方法，来动态添加方法实现
    - `+resolveInstanceMethod:`
    - `+resolveClassMethod:`

<aside>
💡 这两个方法可以去动态的添加方法的实现。

</aside>

**添加实现**

```swift
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == sel_registerName("test")) {
        class_addMethod([MainViewController class], @selector(test), IMP(test2), "v@:");
        return YES;
    }
    return [super resolveInstanceMethod: sel];
}
```

- 动态解析过后，会重新走“消息发送”的流程
    - “从`receiverClass`的`cache`中查找方法”这一步开始执行

## 消息转发

当对象（`receiver`）通过`isa`在类对象中没有找到要调用的方法，并且在父类也没有找到，而且在`动态解析方法`中也没有实现，或者没有返回`nil`，则会进入消息转发。

消息转发首先会调用

```objectivec
// (id) forwardingTargetForSelector:(SEL)aSelector

-(id) forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == sel_registerName("tryAtRuntimeImplementationThisFunc:")) {
        // 转发给谁？
        return [[Person alloc] init];
        //return nil;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

如果此时`返回转发对象`，则在该对象的类对象内存中找到同名的方法进行调用。

如果返回`nil`，则继续后面的流程；

<aside>
💡 1. 设定需要转发的方法类型

</aside>

```objectivec
**// Person中的方法**

//[NSMethodSignature signatureWithObjCTypes:"i17@0:8c16"]
- (int)tryAtRuntimeImplementationThisFunc:(char)c;
// i 代表返回类型；
// 17代表所占内存总数；
// @ 代表对象参数：self
// 0 代表从0位开始 // self 站8个字节
// ：代表方法名参数：SEL
// 8 从第8未开始 // SEL 占8个字节
// c 代表char类型
// 16 从16位开始 // char字节

// 实现
- (int)tryAtRuntimeImplementationThisFunc:(char)c {
    NSLog(@"消息转发成功，原值为：A，修改后：%c", c);
    return 10;
}
```

```objectivec
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    if (aSelector == @selector(tryAtRuntimeImplementationThisFunc:)) {
        // 设定需要转发的方法类型
        //[NSMethodSignature signatureWithObjCTypes:"i17@0:8c16"];
        return [NSMethodSignature signatureWithObjCTypes:"i@:c"];
    }
    return [super methodSignatureForSelector:aSelector];
}
```

<aside>
💡 2. **修改**参数、**获取**参数、**确定转发对象**

</aside>

```objectivec
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if (anInvocation.selector == @selector(tryAtRuntimeImplementationThisFunc:)) {
        
        // 修改参数，方法参数有前两个默认参数self，_cmd,第2位才是我们自己定义的参数
        char ar = 'B';
//        [anInvocation getArgument:&ar atIndex:2];
//        ar = ;
        // 修改参数的值
        [anInvocation setArgument:&ar atIndex:2];
        // 将消息转发给谁，并调用。确定target，就会去该instance对象的class类对象内存中找到同名的方法
        // 只需确保该对象的方法名是相通的，参数类型、返回值均可修改。
        [anInvocation invokeWithTarget:[[Person alloc] init]];
        int i;
        // 确定好哪个target，就可以找对对应的方法的返回值。
        [anInvocation getReturnValue:&i];
        NSLog(@"打印转发后的方法的返回值：%d", i);
        
    }
}

// 打印输出
2021-06-01 12:21:25.686874+0800 Unit[3278:175185] 消息转发成功，原值为：A，修改后：B
2021-06-01 12:21:25.686976+0800 Unit[3278:175185] 打印转发后的方法的返回值：10
```

# super关键字

```objectivec
{
		[super class];
		// 翻译为源码
		((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("Student"))}, sel_registerName("class"));
}
```

可知: super的本质是`struct objc_super`

```objectivec
struct objc_super {
		id receiver;
		Class cls;    // the class to search
}

// {(id)self, (id)class_getSuperclass(objc_getClass("Student"))}

所以发送消息的**receiver**本质还是**self**
```

结论：`super`本质还是当前类一个包含了父类对象信息的特殊实例对象，只是当`super`调用方法时，直接是从父类开始查找。

```objectivec
// 所以：

[super class];
// 打印结果：仍未当前类，同 **[self class];**
```

# 奇怪的面试题

以下代码能不能执行成功？如果能执行结果是什么？

```objectivec
@interface Person : NSObject

- (void)print;
@property (copy, nonatomic) NSString* name;

@end

@implementation Person

- (void)print {
    NSLog(@"name is %@", self.name);
}

@end

- (void)viewDidLoad {
    [super viewDidLoad];

    id cls = [Person class];
    void *obj = &cls;
    [(__bridge id)obj print];
    
}

// 打印结果
2021-06-01 17:22:43.388674+0800 Unit[4571:357882] name is <ViewController: 0x7f8923407af0>
```

**理解：是否能运行成功？能。**

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];

// cls 是Person类对象，obj则是Person类对象的地址
// 也就是Person实例对象isa & ISA_MASK的结果
// 即：[[Person alloc] init]->isa & ISA_MASK

    id cls = [Person class];
    void *obj = &cls;

// 如同Perosn实例对象通过isa调用print，所以可以运行成功
    [(__bridge id)obj print];
    
}
```

**理解：打印的结果是什么？**

**首先一个`Person`实例对象的存储结构。**

```objectivec
struct Person_0 {
		Class isa;
		NSString * _name;
}
```

`**self.name`的本质是什么？**

本质是`isa`高位地址后8个字节。所以有了`isa`的地址，在找到它前面的8个字节，系统就认为它是`name`。

**此时内存中的结构**

![Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2013.png](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/Untitled%2013.png)

`cls`前面是

```objectivec
[super viewDidLoad];

// super 的本质结构
struct objc_super {
		id receiver;
		Class cls;    // the class to search
}

// 所以取得前8个字节, 是 receiver , 而 receiver 本质就是 self

// 打印结果
2021-06-01 17:22:43.388674+0800 Unit[4571:357882]
name is <ViewController: 0x7f8923407af0>
```

此时如果cls前面数据不是正好8个字节，就会访问错误，`报错EXC_BAD_ACCESS`。

[LLVM中间代码](Runtime%200d1bb5ba08864fd2b2c5a665eb028a4c/LLVM%E4%B8%AD%E9%97%B4%E4%BB%A3%E7%A0%81%204e8ad9173adc4c7d860e321cbf7756d3.md)

# Runtime API

## 通过类名获取类对象

```objectivec
/**
 * 通过类名获取类对象
 */
- (void)objc_getClassMethods_API_0 {
    /**
     Class objc_getClass(const char *aClassName)
     {
         if (!aClassName) return Nil;

         // NO unconnected, YES class handler
         return look_up_class(aClassName, NO, YES);
     }
     */
    Class cls1 = objc_getClass("ViewController");
    
    /**
     Class objc_lookUpClass(const char *aClassName)
     {
         if (!aClassName) return Nil;

         // NO unconnected, NO class handler
         return look_up_class(aClassName, NO, NO);
     }
     */
    Class cls2 = objc_lookUpClass("ViewController");
    Class cls3 = objc_getRequiredClass("ViewController");
    
    Class cls4 = objc_getClass("ViewController1");
    Class cls5 = objc_lookUpClass("ViewController1");
    
    /**
     使用 objc_getRequiredClass(const char * _Nonnull name)
     必须传入有效的类名，否则会崩溃
     底层实现也是objc_getClass(),如果找不到类，就报错。
     Class objc_getRequiredClass(const char *aClassName)
     {
         Class cls = objc_getClass(aClassName);
         if (!cls) _objc_fatal("link error: class '%s' not found.", aClassName);
         return cls;
     }
     */
    
//    Class cls6 = objc_getRequiredClass("ViewController1");
    
    NSLog(@"cls1 is %@", cls1);
    NSLog(@"cls2 is %@", cls2);
    NSLog(@"cls3 is %@", cls3);
    NSLog(@"cls4 is %@", cls4);
    NSLog(@"cls5 is %@", cls5);
//    NSLog(@"cls6 is %@", cls6);
}

// 打印输出
2021-06-02 13:21:48.507819+0800 RuntimeAPI[1839:100002] cls1 is ViewController
2021-06-02 13:21:48.507909+0800 RuntimeAPI[1839:100002] cls2 is ViewController
2021-06-02 13:21:48.507994+0800 RuntimeAPI[1839:100002] cls3 is ViewController
2021-06-02 13:21:48.508072+0800 RuntimeAPI[1839:100002] cls4 is (null)
2021-06-02 13:21:48.508148+0800 RuntimeAPI[1839:100002] cls5 is (null)
```

`objc_getClass()`、`objc_lookUpClass()`，找不类会返回`nil`，`objc_getRequiredClass()`找不到类则会直接报错。

## 通过实例对象获取类对象，通过类对象获取元类对象

```objectivec
{
		/// 通过实例对象获取类对象
    ViewController *obj = [[ViewController alloc] init];
    // 获取类对象
    Class cls7 = object_getClass(obj);
    NSLog(@"cls7 is obj`s class %@, %p, is metaclass %d", cls7, cls7, class_isMetaClass(cls7));
    
    /// 通过类名获取元类对象
    Class cls8 = objc_getMetaClass("ViewController");
    NSLog(@"cls8 is ViewController`s metaclass %@, %p, is metaclass %d", cls8, cls8, class_isMetaClass(cls8));
    
    /// 通过类对象获取元类对象
    Class cls9 = object_getClass(cls7);
    NSLog(@"cls9 is obj`s metaclass %@, %p, is metaclass %d", cls9, cls9, class_isMetaClass(cls9));
}

// 打印结果
cls7 is obj`s class ViewController, 0x10519c560, is metaclass 0
cls8 is ViewController`s metaclass ViewController, 0x10519c588, is metaclass 1
cls9 is obj`s metaclass ViewController, 0x10519c588, is metaclass 1
```

## 通过类对象获取父类类对象

```objectivec
/**
 * Runtime api: 获取类对象的父类对象;
 */
- (void)get_superclass_runtime_api {
    Class cls = class_getSuperclass([ViewController class]);
    NSLog(@"ViewController`s superclass is %@", cls);
}
```

## 改变实例对象的类型

```objectivec
/**
 * runtime时改变实例对象的类型；
 */
- (void)changeInstance_obj_classType {
    ViewController *obj = [[ViewController alloc] init];
    Class oldClass = object_setClass(obj, [UIViewController class]);
    NSLog(@"oldClass is %@, obj is %@", oldClass, obj);
}

// 打印结果
oldClass is ViewController, obj is <UIViewController: 0x7fc201005ce0>
```

## 动态创建一个类

```objectivec
void forNewClass(id self, SEL _cmd) {
    NSLog(@"this is a runtime added method");
}
/**
 * runtime时创建一个类；
 */
- (void) createClassWithRuntime {
    // 创建继承UIViewController的子类YZ_ViewController
    Class NewClass = objc_allocateClassPair([UIViewController class], "YZ_ViewController", 0);
    // 给YZ_ViewController类添加成员变量：someName
    class_addIvar(NewClass, "someName", 8, 1, @encode(NSString *));
    // 给YZ_ViewController类添加成员变量：age
    class_addIvar(NewClass, "age", 4, 1, @encode(int));
    // 注册该类，否则无法使用
    objc_registerClassPair(NewClass);
    // 给该类添加方法
    class_addMethod(NewClass, @selector(runtimeLog), (IMP)forNewClass, "v@:");
    id newObj = [NewClass new];
    
    [newObj setValue:@"newObj`s someName" forKey:@"someName"];
    [newObj setValue:@20 forKey:@"age"];
    [newObj runtimeLog];
    NSLog(@"newClass is %@, and newObj someName is %@", NewClass, [newObj valueForKey:@"someName"]);
    NSLog(@"newObj age is %d", [[newObj valueForKey:@"age"] intValue]);
    
    //objc_disposeClassPair(NewClass);
}
```

## 获取一个实例变量信息

```objectivec
- (void)getInstanceVarsInfos {
    Ivar varInfo = class_getInstanceVariable([ViewController class], "_title");
    NSLog(@"name is %s, type of encoding is %s", ivar_getName(varInfo), ivar_getTypeEncoding(varInfo));
}

// 打印结果
name is _title, type of encoding is @"NSString"
```

## 拷贝实例变量列表（最后需要调用free释放）

```objectivec
- (void)copyVarsList {
    // 拷贝实例变量列表（最后需要调用free释放）
    unsigned int counts;
    Ivar *vars = class_copyIvarList([UILabel class], &counts);
    for (int i = 0; i < counts; i++) {
        Ivar var = vars[i];
        NSLog(@"%s ---- %s", ivar_getName(var), ivar_getTypeEncoding(var));
    }
    free(vars);
}

...
2021-06-05 10:04:05.380205+0800 RuntimeAPI[1397:35429] _adjustsFontForContentSizeCategory ---- B
2021-06-05 10:04:05.380399+0800 RuntimeAPI[1397:35429] _preferredMaxLayoutWidth ---- d
2021-06-05 10:04:05.380523+0800 RuntimeAPI[1397:35429] _multilineContextWidth ---- d
2021-06-05 10:04:05.380701+0800 RuntimeAPI[1397:35429] _fontForShortcutBaselineCalculation ---- @"UIFont"
2021-06-05 10:04:05.380841+0800 RuntimeAPI[1397:35429] __visualStyle ---- @"_UILabelVisualStyle"
...
```

## 将方法实现进行交换

```objectivec
- (void)runtimeExchangeMethods {
    Method mExchange01 = class_getInstanceMethod([ViewController class], @selector(methodForExchange01));
    Method mExchange02 = class_getInstanceMethod([ViewController class], @selector(methodForExchange02));
    method_exchangeImplementations(mExchange01 , mExchange02);
    [self methodForExchange01];
    [self methodForExchange02];
}

// 打印结果
2021-06-05 10:14:56.327340+0800 RuntimeAPI[1508:44243] -[ViewController methodForExchange02]
2021-06-05 10:14:56.327434+0800 RuntimeAPI[1508:44243] -[ViewController methodForExchange01]
```