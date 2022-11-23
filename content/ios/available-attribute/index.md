---
title: "Available Attribute"
# description: ""
date: 2022-11-23T12:31:14+08:00
draft: false
tags: ["Available"]
# series: []
# series_order: 1

summary: "@available与调用方进行沟通"
---



## 废弃了一个 API

```swift
@available(*, deprecated, renamed: "newMethod(number:)", message: "不要再使用该方法了")
func oldMethod(value: Int) {
    print(value)
}

func newMethod(number: Int) {
    print("值为：\(number)")
}
```

## @available

@available 有这么几种声明：

**约束平台和版本**
```swift
@available(platform version , platform version ..., *)
```
- `platform` 是指平台，可以是 iOS, macCatalyst, macOS / OSX, tvOS, watchOS 等。[更多可以查看](https://github.com/llvm-mirror/clang/blob/master/include/clang/Basic/Attr.td#L782-L789)
- `version` 是指版本
- `,` 分隔多个版本化平台
- `*` 表示该 API 可用于所有其他平台。

**举例**

```swift
@available(iOS 14, *)
class AppIntro {
}
```

在调用的时候，我们需要通过 `#available`
```swift
if #available(iOS 14, *) {
    let appIntro = AppIntro()
} else {
    // .. 14以下的版本调用
}
```
当然我们也可以指定多个平台：
```swift
@available(iOS 14, macOS 11.0, *)
class AppIntro {
}
```

**约束swift版本**
```swift
@available(swift version)
```

比如需要 Swift 5.5 或更高版本才能使用 `oldMethod：`

```swift
@available(swift 5.5)
func oldMethod(value: Int) {
    print(value)
}
```

**约束指定平台的 introduced, deprecated, obsoleted 和 unavailable**

```swift
// With introduced, deprecated, and/or obsoleted
@available(platform | *
          , introduced: version , deprecated: version , obsoleted: version
          , renamed: "..."
          , message: "...")

// With unavailable
@available(platform | *, unavailable , renamed: "..." , message: "...")
```

- platform：平台名称
- introduced：开始引进的版本号
- deprecated：从指定平台开始过期的版本
- obsoleted：从指定平台开始废弃的版本（注意弃用的区别，deprecated 是还可以继续使用，只不过是不推荐了，obsoleted 是调用就会编译错误）
- message：给出一些附加信息
- renamed：重命名后的新名称
- unavailable：指定平台上是无效的

这些参数可以相互组合使用。下面列举几个常用案例：

```swift
@available(iOS 13, *)
@available(tvOS, unavailable)
@available(macCatalyst, unavailable)
func handleShakeGesture() { … }
```