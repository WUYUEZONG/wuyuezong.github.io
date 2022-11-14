---
title: "Runloop"
# description: ""
date: 2022-11-14T16:12:19+08:00
draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---


# 参考阅读

[理解 OC 中 RunLoop](https://zhuanlan.zhihu.com/p/277380342)

# RunLoop的简介、作用

`RunLoop`可以简单理解为，让程序保持运行的一个`while`循环，这个循环内监听各种事件（如触摸事件、`performSelector`、定时器`NSTimer`等），没有事件的时候睡眠，从而有效的利用CPU（只有在有事件的时候才用CPU，没事件的时候睡眠）

不管RunLoop有多复杂，其本质就是：一个循环，有事件的时候处理事件，无事件的时候休眠（这里的睡眠是指用户态切换到内核态，这样的休眠线程是被挂起的，不会再占用cpu资源）。

## **RunLoop与线程有如下关系：**

- 一个线程只有一个RunLoop对象（一一对应关系）
- 主线程的RunLoop默认已经创建好了，而子线程的需要手动创建。
- RunLoop在第一次获取时创建，在线程结束时销毁。

## RunLoop的应用

- 定时器（Timer）、PerformSelector
- GCD Async Main Queue
- 事件响应、手势识别、界面刷新
- 网络请求
- AutoreleasePool

## RunLoop的基本作用

- 保持程序的持续运行
- 处理App中的各种事件（比如触摸事件、定时器事件等）
- 节省CPU资源，提高程序性能：该做事时做事，该休息时休息

# RunLoop的基本结构

# RunLoop的创建

# RunLoop的基本使用

## NSTimer的失效

`NSTimer`默认创建事件表，对应的`RunLoop`，默认是`NSDefaultRunLoopMode`，所以在滑动事件上该模式会被暂停。计时器就不会工作。

```objectivec
__block int count = 0;
[NSTimer scheduledTimerWithTimeInterval:1 repeats:YES block:^(NSTimer *timer) {
		NSLog(@"counting is %d", ++count);
}];
```

只需把`NSTimer`放在`RunLoop`的`NSRunLoopCommonModes`模式下，滑动事件就不会影响该`NSTimer`的工作。

```objectivec
__block int count = 0;
NSTimer *timer = [NSTimer timerWithTimeInterval:1 repeats:YES block:^(NSTimer * _Nonnull timer) {
   NSLog(@"counting is %d", ++count);
}];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

## RunLoop线程保活

某个操作如果需要频繁在子线程中进行操作，可以延长该线程的生命周期，而不需要频繁创建新的线程来工作。

创建一个线程，并在该线程的`runloop`中添加`source`使得线程停留。

```objectivec
_thread = [[YZThread alloc] initWithBlock:^{
		NSLog(@"thread is begin");
		CFRunLoopSourceContext context = {0};
		CFRunLoopSourceRef source = CFRunLoopSourceCreate(kCFAllocatorDefault, 0, &context);
    CFRunLoopAddSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);
    CFRunLoopRunInMode(kCFRunLoopDefaultMode, 1.0e10, false);
    NSLog(@" -------- thread is end -------- ");
}];
[_thread start];
```

在子线程中做相应操作

```objectivec
- (void)excutedInBlock:(LongtimeThreadBlock)block {
    if (!block || !_thread) return;
    [self performSelector:@selector(__excutedDosome:) onThread:_thread withObject:block waitUntilDone:NO];
}

- (void)__excutedDosome:(LongtimeThreadBlock)block {
    block();
}
```

结束线程

```objectivec
- (void)stop {
    if (!_thread) return;
		// waitUnitDone, 这里要设置YES，防止self被提前释放。
    [self performSelector:@selector(__stop) onThread:_thread withObject:nil waitUntilDone:YES];
}

- (void)__stop {
    CFRunLoopStop(CFRunLoopGetCurrent());
    self.thread = nil;
}
```

运用

```objectivec
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _thread = [[LongtimeThread alloc] init];
    
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [_thread excutedInBlock:^{
        NSLog(@"do something in thread.");
    }];
}

@end
```