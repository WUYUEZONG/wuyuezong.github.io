---
# title: 高性能的AutoLayout
title: "High Performance Autolayout"
# description: ""
date: 2022-11-14T17:44:45+08:00

tags: ["AutoLayout", "WWDC", "iOS"]
# series: ["IOS"]
# series_order: 5
---

以下是看完 [WWDC18: High Performance Auto Layout](https://developer.apple.com/videos/play/wwdc2018/220/) 的简单总结。

## 避免操作updateConstants()
避免操作updateConstants() ，它的调用频次是 120次/s ，如果需要在updateConstants()中操作布局，尽量只布局未设置约束的视图。
```swift
func updateConstants() {
	if constants == nil {
		// set constants.
	}
	super.updateConstants()
}
```

## 尽量少而简单的视图依赖关系
设置约束时，尽量只依赖父视图，多个依赖关系可能会造成更多的计算量，而只依赖一个父试图，则只是需要的计算。

## 无需视图时使用hideen属性，而不是删除约束

## 通过激活或停用改变视图，而不是修改约束。
- UILabe、UIImageView、自动测量尺寸并不会消耗多少性能。
- 如果需要频繁计算视图尺寸或已经知道该视图尺寸可以重写属性，提高部分性能。

```swift
override var intrinsicContentSize: CGSize {
	return CGSize(width: UIView.noIntrinsicMetric, height: UIView.noIntrinsicMetric)
}
```

----
想了解的更多可以看 [WWDC18: High Performance Auto Layout](https://developer.apple.com/videos/play/wwdc2018/220/) 相关视频、资料。