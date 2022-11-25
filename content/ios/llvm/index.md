---
title: "Llvm"
# description: ""
date: 2022-11-24T21:39:26+08:00
# draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---

## LLVM中间代码

[llvm](https://llvm.org/docs/LangRef.html)

**Objective-C在变为机器代码之前，会被LLVM编译器转换为中间代码（Intermediate Representation**

可以使用以下命令行指令生成中间代码

```shell
clang -emit-llvm -S main.m
```

|符号|说明|
|---|---|
|@|全局变量|
|%|局部变量|
|alloca|在当前执行的函数的堆栈中分配内存，当该函数返回到其调用者时，将自动释放内存|
|i32|32位4字节的整数|
|align|对齐|
|load|读出|
|store|写入|
|icmp|两个整数值比较，返回布尔值|
|br|选择分支，根据条件来转向label，不根据条件跳转的话类似 goto|
|label|代码标签|
|call|调用函数|