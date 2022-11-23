---
title: "设计一个可变数组"
# description: ""
date: 2022-11-23T17:14:45+08:00
draft: false
tags: ["Array"]
# series: []
# series_order: 1

summary: "设计一个可变数组"
---

通过设计一个可变数组，来学习数组

## 1. 完成基础的封装

新建类`Array`<br>
声明`data, size`用于内部管理数据，并将其私有化，防止外部修改数据。<br>
实现基本个构造函数`Array(int capacity){}`可以自定义数组大小，实现一个无参构造函数`Array(){}`方便直接创建`capacity=10`的对象。

**根据经验提供一些简便方法**

`getSize()`获取当前数组个数；<br>
`getCapacity()`获取当前数组容量；<br>
`isEmpty()`判断数组是否为空；

```java
public class Array {

    private int[] data;
    private int size;

    // 构造函数
    public Array(int capacity) {
        data = new int[capacity];
        size = 0;
    }

    // 无参构造函数，默认capacity=10
    public Array() {
        this(10);
    }

    // 获取数组个数
    public int getSize() {
        return size;
    }

    // 获取数组容量
    public int getCapacity() {
        return data.length;
    }

    // 返回数组是否为空
    public boolean isEmpty() {
        return size == 0;
    }

}
```

