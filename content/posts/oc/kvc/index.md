---
title: "KVC（Key-Value Coding）键值编码"
# description: ""
date: 2022-11-14T16:08:03+08:00
draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---


# KVC主要方法

```objectivec
- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;
- (void)setValue:(id)value forKey:(NSString *)key;
- (id)valueForKeyPath:(NSString *)keyPath;
- (id)valueForKey:(NSString *)key;
```

# setValue: forKey: 的原理

![KVC%EF%BC%88Key-Value%20Coding%EF%BC%89%E9%94%AE%E5%80%BC%E7%BC%96%E7%A0%81%203a5763245da042dca1a4a4df8d612a30/Untitled.png](KVC%EF%BC%88Key-Value%20Coding%EF%BC%89%E9%94%AE%E5%80%BC%E7%BC%96%E7%A0%81%203a5763245da042dca1a4a4df8d612a30/Untitled.png)

# **valueForKey: 的原理**

![KVC%EF%BC%88Key-Value%20Coding%EF%BC%89%E9%94%AE%E5%80%BC%E7%BC%96%E7%A0%81%203a5763245da042dca1a4a4df8d612a30/Untitled%201.png](KVC%EF%BC%88Key-Value%20Coding%EF%BC%89%E9%94%AE%E5%80%BC%E7%BC%96%E7%A0%81%203a5763245da042dca1a4a4df8d612a30/Untitled%201.png)

# 使用KVC修改是否会触发KVO？（设置了监听）

使用KVC，不管是修改成员变量还是属性的值都会触发KVO。