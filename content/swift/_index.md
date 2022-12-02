---
title: "Swift"
# description: ""
date: 2022-11-29T15:09:00+08:00
# draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---

## 基本类型

Int

Double  
`Swift`能够安全的存储15～16位数的`Double`类型，例如：  
```swift
// _ 数字中 _ 会被自动忽略
let a: Double = 123_456_789_012_345
```

Decimal  
精度要求较高时使用，比如涉及金钱。  
`Swift`能够安全的存储38～39位数的`Decimal`类型，初始化时注意，通过`Int\Double`转换过来可能会有误差，通过`String`则更为精准。

String

Bool

Tuple  
```swift
let allen = (name: "allen", height: 170)
let (_, height) = allen
print("allen height", height)
```
