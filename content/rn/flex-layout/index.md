---
title: React Native Flexbox
date: 2022-06-30

tags: ["Flexbox"]
series: ["RN"]
series_order: 1
---

这里只是个人学习对官方的Flexbox布局的简单总结以及备忘，内容并不全面，深入学习建议阅读官方文档。


> 推荐阅读官网[“使用Flexbox布局”](https://www.react-native.cn/docs/flexbox)


<!-- more -->

# Flex
flex属性决定元素在主轴上如何填满可用区域。整个区域会根据每个元素设置的 flex属性值被分割成多个部分。

# Flex Direction
在组件的style中指定flexDirection可以决定布局的主轴。子元素是应该沿着水平轴(row)方向排列，还是沿着竖直轴(column)方向排列呢？默认值是竖直轴(column)方向。
- column | 从上到下
- row | 从左到右
- column-reverse | 从下到上
- row-reverse | 从右到左

# Layout Direction
- ltr | 从左到右
- rtl | 从右到左

# justifyContent
在组件的 style 中指定justifyContent可以决定其子元素沿着主轴的排列方式。子元素是应该靠近主轴的起始端还是末尾段分布呢？亦或应该均匀分布？可用的选项有：

- flex-start
- flex-end
- space-between
- space-around
- space-evenly

# Align Items
在组件的 style 中指定alignItems可以决定其子元素沿着次轴（与主轴垂直的轴，比如若主轴方向为row，则次轴方向为column）的排列方式。子元素是应该靠近次轴的起始端还是末尾段分布呢？亦或应该均匀分布？可用的选项有

- stretch | 填满次轴 
- flex-start
- flex-end
- center
- baseline

# Align Self
作用同 Align Items，但是只作用于被定义的对象，不影响容器子对象。

# Align Content
适用于被折行的子视图，即，设置属性flexWrap时（Wrap）。效果同justifyContent，是次轴上的justifyContent。

- flex-start
- flex-end
- space-between
- space-around
- space-evenly

# Flex Wrap
- wrap
- no-wrap

# Flex Basis, Grow, and Shrink
flexBasis: 视图在主轴上的宽度（auto，int）
flexGrow: 视图在主轴上的生长能力（int），grow较大则会填充容器。
flexShrink: 视图在主轴上的缩小能力（int），shrink较大视图则会根据basis或width的值显示。