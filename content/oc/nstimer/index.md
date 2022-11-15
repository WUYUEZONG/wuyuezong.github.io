---
title: "定时器"
date: 2022-06-30


tags: ["OC", "NSTimer"]
series: ["Objc"]
series_order: 17
---


## NSTimer、CADisplayLink定时器

**CADisplayLink使用**

```objectivec
@interface TMViewController ()
@property (strong, nonatomic) CADisplayLink * link;
@end

- (void)viewDidLoad {
    [super viewDidLoad];
    _link = [CADisplayLink displayLinkWithTarget:(id)target selector:@selector(linkTest)];
    [_link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];    
}

- (void)dealloc {
    [_link invalidate];
}
```

**NSTimer使用**

```objectivec
@interface TMViewController ()
@property (strong, nonatomic) NSTimer * timer;
@end

- (void)viewDidLoad {
    [super viewDidLoad];
    _timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:(id)target selector:@selector(linkTest) userInfo:nil repeats:YES];
}

- (void)dealloc {
    [_timer invalidate];
}
```

以上方式如果将`target`传为`self`，则会造成`self`对`timer`的强引用，`timer`又对`self`强引用，双方释放不了，造成`循环引用`。

### 解决方法

1. 创建一个不依赖于`self`的`timer`

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _timer = [NSTimer timerWithTimeInterval:1 repeats:YES block:^(NSTimer * _Nonnull timer) {
        NSLog(@"执行任务");
    }];
    [[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSDefaultRunLoopMode];
    
}

- (void)dealloc {
    [_timer invalidate];
}
```

2. 添加中间对象（NSProxy），弱引用`self`然后将消息转发给`self`。

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
     
    _timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:[TMProxy initWithTarget:self] selector:@selector(linkTest) userInfo:nil repeats:YES];

}

- (void)dealloc {
    [_timer invalidate];
}

// TMProxy实现

@interface TMProxy ()

@property (weak, nonatomic) id target;

@end

@implementation TMProxy

+ (TMProxy *)initWithTarget:(id)target {
    TMProxy *proxy =  [TMProxy alloc];
    proxy.target = target;
    return proxy;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [_target methodSignatureForSelector:sel];
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    [invocation invokeWithTarget:_target];
}

@end
```

## GCD定时器

由于`NSTimer`是依赖于`RunLoop`进行工作的，而`RunLoop`内部除了需要执行`NSTimer`事务，同时也可能处理其他事务，所以`RunLoop`进行计时会又不准确的情况。而`GCD`不依赖于`RunLoop`，`GCD定时器`属于系统层面。所以`GCD定时器`能够相对准确。

### 创建GCD定时器

```objectivec
@interface ViewController ()
@property (strong, nonatomic) dispatch_source_t timer;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(0, 0));
    dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC, 0);
    dispatch_source_set_event_handler(timer, ^{
        NSLog(@"gcd timer counting.");
    });
    dispatch_resume(timer);
		// 防止timer被释放。
    _timer = timer;
}
@end
```