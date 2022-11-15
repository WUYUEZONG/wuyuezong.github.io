---
title: "Objective-C的本质"
# description: ""
date: 2022-11-14T16:09:26+08:00
draft: true
tags: ["OC", "Objc"]
series: ["Objc"]
series_order: 4

# summary: ""
---

<aside>
💡 Objective-C底层实现其实都是C、C++代码，Objective-C的面向对象都是基于C、C++的数据结构实现的，Objective-C的对象、类主要是基于C、C++的结构体实现的。

</aside>

**Objective-C被翻译的过程**

![Objective-C%E7%9A%84%E6%9C%AC%E8%B4%A8%20f8b9fed4e31a43eab9cdc53d079edaa9.jpeg](Objective-C%E7%9A%84%E6%9C%AC%E8%B4%A8%20f8b9fed4e31a43eab9cdc53d079edaa9.jpeg)

**Objective-C代码转换为C、C++代码**

```objectivec
// 终端命令
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.c -o main.cpp
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc “oc源文件” -o “输出cpp文件”
```

# 一个OC对象在内存中是如何布局的？

```objectivec
// alloc 申请一块内存空间，init 初始化一个类的实例对象
NSObject *obj = [[NSObject alloc] init];
```

**OC中对象至少会被分配16个字节的空间（64位）；32位则是8个字节**

[OC基本数据类型所占字节数对比](Objective-C%E7%9A%84%E6%9C%AC%E8%B4%A8%20f8b9fed4e31a43eab9cdc53d079edaa9/OC%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E6%89%80%E5%8D%A0%E5%AD%97%E8%8A%82%E6%95%B0%E5%AF%B9%E6%AF%94%205e436cb211f1494a8c424b4663a5d9c2.csv)

## 参考

[1、OC基本数据类型](https://www.daimajiaoliu.com/daima/4ed3aba5710041c)

[关于NSObject对象的内存布局，看我就够了！](https://zhuanlan.zhihu.com/p/98432137)

# 实时查看内存数据

**DeBug > DeBug Workflow >  View Memory，Shift + Command + M**

## 创建一个实例对象，至少需要多少内存？

一个对象至少需要8个字节

```objectivec
#import <objc/runtime.h>
class_getInstanceSize([NSObject class]);
```

## 创建一个实例对象，实际分配了多少内存？

64位系统至少会分配16个字节，32位则是8个字节。

```objectivec
#import <malloc/malloc.h>
NSObject *obj = [[NSObject alloc] init];
malloc_size((__bridge const void *)obj);
```

## 常用LLDB指令

> print、p： 打印
po：打印对象
> 
- 读取内存
    - Memory read/数量格式字节数 内存地址
    - x/数量格式字节数 内存地址

<aside>
💡 例如：x/3xw 0x10010

</aside>

- **格式**
    - x是16进制，f是浮点数，d是10进制
- **字节大小**
    - b：byte 1字节
    - h：half wrod 2字节
    - w： word 4字节
    - g：giant word 8字节
- **修改内存中的值**
    - memory write 内存地址 数值：memory write 0x0000010 10