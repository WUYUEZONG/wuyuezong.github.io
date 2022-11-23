---
title: "探究UIViewController生命周期"
date: 2022-06-30


tags: ["OC"]
series: ["OC"]
series_order: 30

summary: "探究UIViewController生命周期"
---


进入 `ViewController2`

```swift
+[ViewController2 load] // 加载时就会调用
+[ViewController2 initialize] // 第一次接收通知时调用(alloc)
-[ViewController2 loadView]
-[ViewController2 viewDidLoad]
-[ViewController viewWillDisappear:]
-[ViewController2 viewWillAppear:]
-[ViewController2 viewWillLayoutSubviews]
-[ViewController2 viewDidLayoutSubviews]
-[ViewController viewDidDisappear:]
-[ViewController2 viewDidAppear:]
```

返回 `ViewController`

```swift
-[ViewController2 viewWillDisappear:]
-[ViewController viewWillAppear:]
-[ViewController2 viewDidDisappear:]
-[ViewController viewDidAppear:]
-[ViewController2 dealloc]
```

当添加了一个子视图时(`Storyboard`) - 进入

```swift
+[ViewController2 initialize]
+[TestView2 initialize]
-[TestView2 willMoveToSuperview:]
-[TestView2 didMoveToSuperview]
-[ViewController2 loadView]
-[ViewController2 viewDidLoad]
-[ViewController viewWillDisappear:]
-[ViewController2 viewWillAppear:]
-[TestView2 willMoveToWindow:]
-[TestView2 didMoveToWindow]
-[ViewController2 viewWillLayoutSubviews]
-[ViewController2 viewDidLayoutSubviews]
-[TestView2 layoutSubviews]
-[TestView2 drawRect:]
-[ViewController viewDidDisappear:]
-[ViewController2 viewDidAppear:]
```

当添加了一个子视图时(`Storyboard`) - 返回

```swift
-[ViewController2 viewWillDisappear:]
-[ViewController viewWillAppear:]
-[TestView2 willMoveToWindow:]
-[TestView2 didMoveToWindow]
-[ViewController2 viewDidDisappear:]
-[ViewController viewDidAppear:]
-[ViewController2 dealloc]
-[TestView2 willMoveToSuperview:]
-[TestView2 didMoveToSuperview]
```

## 单独拆分`View2`的生命周期

```swift
+[TestView2 initialize]
-[TestView2 willMoveToSuperview:]
-[TestView2 didMoveToSuperview]
-[TestView2 willMoveToWindow:]
-[TestView2 didMoveToWindow]
-[TestView2 layoutSubviews]
-[TestView2 drawRect:]

-[TestView2 willMoveToWindow:]
-[TestView2 didMoveToWindow]
-[TestView2 willMoveToSuperview:]
-[TestView2 didMoveToSuperview]
```

给 `View2` 动态添加一个子视图时

```swift
+[TestView initialize]
-[TestView willMoveToWindow:]
-[TestView willMoveToSuperview:
-[TestView didMoveToWindow]
-[TestView didMoveToSuperview]
-[TestView2 didAddSubview:]
-[TestView2 addSubview:]
-[TestView2 layoutSubviews]
-[TestView layoutSubviews]
-[TestView drawRect:]
```

修改子视图 `frame` 时

```swift
-[TestView willMoveToWindow:]
-[TestView willMoveToSuperview:]
-[TestView didMoveToWindow]
-[TestView didMoveToSuperview]
-[TestView2 didAddSubview:]
-[TestView2 addSubview:]
-[TestView2 layoutSubviews]
-[TestView layoutSubviews]
-[TestView layoutSubviews]
-[TestView drawRect:]
```

### 简化对View2子视图的操作

被添加时

```swift
+[TestView initialize]
-[TestView2 layoutSubviews]
-[TestView layoutSubviews]
-[TestView drawRect:]
```

被修改时 `frame` 时

```swift
-[TestView2 didAddSubview:]
-[TestView2 addSubview:]
-[TestView2 layoutSubviews]
-[TestView layoutSubviews]
-[TestView layoutSubviews]
-[TestView drawRect:]
```