---
title: iOS Runloop
categories: 
- iOS开发
tags: 
- runloop
---

# 整理中...

<!-- more -->

# RunLoop与线程
`RunLoop` 和线程是一一对应关系，在源码中，线程和 `Runloop` 互为一对 `Key、Value`，可以关注一下 `loop` 的获取：**是把线程地址作为key来获取RunLoop的**。
```c++
void const *CFDictionaryGetValue(CFDictionaryRef hc, void const *key) {...}
// ...
CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
```
通过这里也可以知道，`RunLoop` 中是没有提供创建的API，只需要通过在线程内部获取当前 `RunLoop` 就会自动创建（主线程除外）。

```c++
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(_CFThreadRef t) {
    if (pthread_equal(t, kNilPthreadT)) {
	t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    if (!__CFRunLoops) {
	CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
	CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
	CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wdeprecated"
	if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
#pragma GCC diagnostic pop
	    CFRelease(dict);
	}
	CFRelease(mainLoop);
    }
    CFRunLoopRef newLoop = NULL;
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    if (!loop) {
	newLoop = __CFRunLoopCreate(t);
        
        cf_trace(KDEBUG_EVENT_CFRL_LIFETIME|DBG_FUNC_START, newLoop, NULL, NULL, NULL);
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
    __CFUnlock(&loopsLock);
    // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
    if (newLoop) { CFRelease(newLoop); }
    
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
#if _POSIX_THREADS
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
#else
            _CFSetTSD(__CFTSDKeyRunLoopCntr, 0, &__CFFinalizeRunLoop);
#endif
        }
    }
    return loop;
}
```

# RunLoop的组成

一个 `RunLoop` 主要由：`CFRunLoopMode / CFRunLoopSourceRef / CFRunLoopObserverRef / CFRunLoopTimerRef` 组成。 `RunLoop` 可以拥有多种Mode，Mode中包含 `Source / Observer / Timer`。
![]()

## CFRunLoopMode（RunLoop的运行模式）共五类

- kCFRunLoopDefaultMode
`App` 的默认 `Mode`，通常主线程是在这个 `Mode` 下运行

- UITrackingRunLoopMode
界面跟踪`Mode`，用于 `ScrollView` 追踪触摸滑动，保证界面滑动时不受其他 `Mode` 影响

- UIInitializationRunLoopMode
在刚启动 `App` 时第进入的第一个 `Mode`，启动完成后就不再使用

- GSEventReceiveRunLoopMode
接受系统事件的内部 `Mode`，通常用不到

- kCFRunLoopCommonModes
这是一个占位用的 `Mode`，不是一种真正的 `Mode`，可以简单理解为 `kCFRunLoopDefaultMode` 和 `UITrackingRunLoopMode` 的结合

## CFRunLoopSource（输入源/事件源）
这里有两个源：_sources0 、 _sources1
**_sources0**
即非基于 `port` 的，也就是用户触发事件，需要手动唤醒线程。将当前线程从{% hint '内核态切换到用户态' 'CPU的两种工作状态，内核态可以调度所有的资源，用户态则只可以调度用户工作界面' %}
**_sources1**
基于 `port` 的，包含一个 `mach_port` 和一个回调，可以监听系统端口和通过内核态和其他线程发送消息，能主动唤醒 `RunLoop`，接收分发系统事件，具备唤醒线程能力。

## CFRunLoopTimer（定时源）
基于时间的触发器，基本上说的就是 `NSTimer`。在预设的时间点唤醒 `RunLoop` 执行回调。因为它是基于 `RunLoop` 的，因此它不是实时的（就是 `NSTimer` 是不准确的。 因为 `RunLoop` 只负责分发源的消息。如果线程当前正在处理繁重的任务，就有可能导致 `Timer` 本次延时，或者少执行一次）

## CFRunLoopObserver（观察者）
可以对 RunLoop 不同状态下进行监听，从而对当前 RunLoop 工作状态进行优化。

- kCFRunLoopEntry
`RunLoop` 准备启动
- kCFRunLoopBeforeTimers
`RunLoop` 将要处理一些 `Timer` 相关事件
- kCFRunLoopBeforeSources
`RunLoop` 将要处理一些 `Source` 事件
- kCFRunLoopBeforeWaiting
`RunLoop` 将要进行休眠状态，即将由用户态切换到内核态
- kCFRunLoopAfterWaiting
`RunLoop` 被唤醒，即从内核态切换到用户态后
- kCFRunLoopExit
`RunLoop` 退出
- kCFRunLoopAllActivities
监听所有状态活动

如何设置监听


# RunLoop运行机制

1️⃣ 调用 `Observer` 监听方法状态为 `kCFRunLoopEntry`
2️⃣ 调用 `Observer` 监听方法状态为 `kCFRunLoopBeforeTimers`
3️⃣ 调用 `Observer` 监听方法状态为 `kCFRunLoopBeforeSources`
4️⃣ 处理 `Blocks` ：`__CFRunLoopDoBlocks`
5️⃣ 处理 `sources0`，如果被处理过则再次处理 `Blocks`
⁉️ 判断是否存在 `sources1`，如果存在 7️⃣，如果不存在 6️⃣
6️⃣ 调用 `Observer` 监听方法状态为 `kCFRunLoopBeforeWaiting`，并随时等待被 `sources1` / `dipatch` / `timer` / `source0` / `手动唤醒` 唤醒，如有，则会：调用 `Observer` 监听方法状态为 `kCFRunLoopAfterWaiting` 继续 7️⃣
7️⃣ 处理 `timers`
8️⃣ 处理 `GCD Main`
9️⃣ 处理 `sources1`
🔟 处理 `Blocks`
⁉️ 是否还有需要处理的任务？是跳转到 2️⃣ 重新开始， 否则：调用 `Observer` 监听方法状态为 `kCFRunLoopExit`





# 主线程

自己能够保活，因为底层判断是根据 `modes` 是否为空来决定线程是否休眠（销毁），如果是主线程则是个例外，直接返回不为空。非主线程则是根据 `_sources0` `_sources1` `_timers` 中是否有数据来判断。
[源码如是](https://github.com/apple/swift-corelibs-foundation)
```c++
// expects rl and rlm locked
static Boolean __CFRunLoopModeIsEmpty(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFRunLoopModeRef previousMode) {
    CHECK_FOR_FORK();
    if (NULL == rlm) return true;
#if TARGET_OS_WIN32
    if (0 != rlm->_msgQMask) return false;
#endif
#if __HAS_DISPATCH__
    Boolean libdispatchQSafe = pthread_main_np() == 1 && ((HANDLE_DISPATCH_ON_BASE_INVOCATION_ONLY && NULL == previousMode) || (!HANDLE_DISPATCH_ON_BASE_INVOCATION_ONLY && 0 == _CFGetTSD(__CFTSDKeyIsInGCDMainQ)));
    if (libdispatchQSafe && (CFRunLoopGetMain() == rl) && CFSetContainsValue(rl->_commonModes, rlm->_name)) return false; // represents the libdispatch main queue
#endif
    if (NULL != rlm->_sources0 && 0 < CFSetGetCount(rlm->_sources0)) return false;
    if (NULL != rlm->_sources1 && 0 < CFSetGetCount(rlm->_sources1)) return false;
    if (NULL != rlm->_timers && 0 < CFArrayGetCount(rlm->_timers)) return false;
    struct _block_item *item = rl->_blocks_head;
    while (item) {
        struct _block_item *curr = item;
        item = item->_next;
        Boolean doit = false;
        if (_kCFRuntimeIDCFString == CFGetTypeID(curr->_mode)) {
            doit = CFEqual(curr->_mode, rlm->_name) || (CFEqual(curr->_mode, kCFRunLoopCommonModes) && CFSetContainsValue(rl->_commonModes, rlm->_name));
        } else {
            doit = CFSetContainsValue((CFSetRef)curr->_mode, rlm->_name) || (CFSetContainsValue((CFSetRef)curr->_mode, kCFRunLoopCommonModes) && CFSetContainsValue(rl->_commonModes, rlm->_name));
        }
        if (doit) return false;
    }
    return true;
}
```

# 所以如何进行线程保活?

只需要在 `_sources0` `_sources1` `_timers` 中添加事件，线程就能处于活跃状态。


**线程是保活了，但如果我想停止怎么办？**

# 如何控制RunLoop


# RunLoop还解决哪些问题？

**卡顿**


**NSRunLoop VS CFRunLoop**
CFRunLoop存在于Foundation框架中，使用的是纯C函数实现，相对于NSRunloop，这些C函数API都是线程安全🔐的。


# 参考

[1](https://www.jianshu.com/p/d97729a4e8a7)
[2](https://zhuanlan.zhihu.com/p/277380342)
[3](https://blog.ibireme.com/2015/05/18/runloop/)