---
title: "多线程"
date: 2022-06-30
draft: true

tags: ["OC", "dispatch",]
series: ["Objc"]
series_order: 16

summary: "多线程"
---

# 多线程类型，关系

# 线程锁死

# 多线程的安全隐患（线程锁）

## OSSpinLock——自旋锁

## os_unfair_lock——自旋锁

### 死锁：永远拿不到锁

## pthread mutex (互斥锁)

# 自旋锁、互斥锁比较

**什么情况使用自旋锁比较划算？**

- 预计线程等待锁的时间很短
- 加锁的代码（临界区）经常被调用，但竞争情况很少发生
- CPU资源不紧张
- 多核处理器

**什么情况使用互斥锁比较划算？**

- 预计线程等待锁的时间较长
- 单核处理器
- 临界区有IO操作
- 临界区代码复杂或者循环量大
- 临界区竞争非常激烈

# atomic

[Atomic / Nonatomic](%E5%A4%9A%E7%BA%BF%E7%A8%8B%20e767b1ff994f4fa1801ebc6f1f51ec0a/Atomic%20Nonatomic%20ff51fba592364aa0b77447729dd0fef6.md)

# ios中的读写安全（多读单写）

**思考如何实现以下场景**

- 同一时间，只能有1个线程进行写的操作
- 同一时间，允许有多个线程进行读的操作
- 同一时间，不允许既有写的操作，又有读的操作

{{<alert>}}
上面的场景就是典型的“多读单写”，经常用于文件等数据的读写操作，iOS中的实现方案有
{{</alert>}}

## pthread_rwlock：读写锁

## dispatch_barrier_async：异步栅栏调用