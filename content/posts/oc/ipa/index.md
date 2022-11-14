---
title: "安装包瘦身"
date: 2022-06-30


tags: ["OC"]
series: ["OC"]
series_order: 20
---


![](1.png)

## 资源（图片、音频、视频等）

- 采取无损压缩
- 去除没有用到的资源：[https://github.com/tinymind/LSUnusedResources](https://github.com/tinymind/LSUnusedResources)

## 可执行文件瘦身

- 编译器优化
    - Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default设置为YES
    - 去掉异常支持，Enable C++ Exceptions、Enable Objective-C Exceptions设置为NO， Other C Flags添加-fno-exceptions
- 利用AppCode（[https://www.jetbrains.com/objc/）检测未使用的代码：菜单栏](https://www.jetbrains.com/objc/%EF%BC%89%E6%A3%80%E6%B5%8B%E6%9C%AA%E4%BD%BF%E7%94%A8%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%9A%E8%8F%9C%E5%8D%95%E6%A0%8F) -> Code -> Inspect Code
- 编写LLVM插件检测出重复代码、未被调用的代码