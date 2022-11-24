---
title: "Bit Operation"
# description: ""
date: 2022-11-24T21:06:20+08:00
draft: false
# tags: []
# series: []
# series_order: 1

summary: "位运算"
---

**位运算（& | ^ ~ << >>）**

**&** 与运算 两个位都是 1 时，结果才为 1，否则为 0，如

```c
  1 0 0 1 1
& 1 1 0 0 1
----------------
  1 0 0 0 1
```

**|** 或运算 两个位都是 0 时，结果才为 0，否则为 1，如

```c
  1 0 0 1 1
| 1 1 0 0 1
----------------
  1 1 0 1 1
```

**^** 异或运算，两个位相同则为 0，不同则为 1，如

```c
  1 0 0 1 1
^ 1 1 0 0 1
----------------
  0 1 0 1 0
```

**~** 取反运算，0 则变为 1，1 则变为 0，如

```c
~ 1 0 0 1 1
----------------
  0 1 1 0 0
```

**<<** 左移运算，向左进行移位操作，高位丢弃，低位补 0，如

```c
int a = 8;
a << 3;
移位前：0000 0000 0000 0000 0000 0000 0000 1000
移位后：0000 0000 0000 0000 0000 0000 0100 0000
```

**>>** 右移运算，向右进行移位操作，对无符号数，高位补 0，对于有符号数，高位补符号位，如

```c
unsigned int a = 8;
a >> 3;
移位前：0000 0000 0000 0000 0000 0000 0000 1000
移位后：0000 0000 0000 0000 0000 0000 0000 0001

int a = -8;
a >> 3;
移位前：1111 1111 1111 1111 1111 1111 1111 1000
移位前：1111 1111 1111 1111 1111 1111 1111 1111
```

## 参考阅读

[位运算有什么奇技淫巧？](https://www.zhihu.com/question/38206659)

## 应用

多选枚举

```objectivec
// 定义MYEnum枚举
typedef enum {
    EnumTypeNone = 0,
    EnumType01 = 1<<0,
    EnumType02 = 1<<1,
    EnumType03 = 1<<2
} MYEnum;

// 声明MYEnum类型的枚举
MYEnum type = EnumType01 | EnumType02;

// 使用MYEnum枚举
- (NSArray *)getEnumCounts:(MYEnum) enumType {
    NSMutableArray * arr = [[NSMutableArray alloc] init];
    if (enumType & EnumType01) {
        [arr addObject:@(EnumType01)];
    }
    if (enumType & EnumType02) {
        [arr addObject:@(EnumType02)];
    }
    if (enumType & EnumType03) {
        [arr addObject:@(EnumType03)];
    }
    return arr;
}
```