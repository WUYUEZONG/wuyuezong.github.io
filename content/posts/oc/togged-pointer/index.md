---
title: "Togged Pointer"
# description: ""
date: 2022-11-14T16:17:51+08:00
# draft: false
tags: ["OC"]
series: ["Objc"]
series_order: 18

# summary: ""
---

对 `NSNumber、NSString、NSDate` 优化。

当以上类型对象需要存储的数据没有超出固定分配的字节，该对象可以被看作是简单数据类型；而当对象需要存储的数据超过了固定字节，该对象就会变成真正的对象。