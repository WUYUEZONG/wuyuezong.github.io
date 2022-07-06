---
title: iOS Runloop
categories: 
- iOS开发
tags: 
- runloop
---

# 整理中...

<!-- more -->

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
CFRunLoop存在于Foundation框架中，使用C语言实现，相对于NSRunloop是线程安全🔐的

