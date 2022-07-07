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

**CFRunLoopMode（RunLoop的运行模式）共五类**
kCFRunLoopDefaultMode
App的默认Mode，通常主线程是在这个Mode下运行

UITrackingRunLoopMode
界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响

UIInitializationRunLoopMode
在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用

GSEventReceiveRunLoopMode
接受系统事件的内部 Mode，通常用不到

kCFRunLoopCommonModes
这是一个占位用的Mode，不是一种真正的Mode，可以简单理解为 `kCFRunLoopDefaultMode` 和 `UITrackingRunLoopMode` 的结合



# RunLoop运行机制



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

