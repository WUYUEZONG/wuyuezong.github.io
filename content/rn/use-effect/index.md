---
title: "Use Effect"
# description: ""
date: 2022-11-23T11:46:43+08:00
draft: false
tags: ["RN", "Use Effect"]
series: ["RN"]
series_order: 2

summary: "UseEffect简单介绍"
---

## useEffect

{{<alert>}}
使用[Effect Hook](https://zh-hans.reactjs.org/docs/hooks-effect.html)
{{</alert>}}

- useEffect 类似一个 class 的生命周期；
- UI每次需要更新会调用 useEffect；
- 所以需要防止 useEffect 中重复刷新，添加参数让 useEffect 对比，使 useEffect 自动对比需不需要执行；
- 一个方法内可以有多个 useEffect。

```ts
  // navigation 控制是否需要执行这个 useEffect
  useEffect(() => {
    navigation.setOptions({headerShown: false});   
  }, [navigation]);
  
  // page 控制是否需要执行这个 useEffect
  useEffect(() => {
    fetchPhotoListWith(page)    
  }, [page]);
  
  // 或者多个参数 控制是否需要执行这个 useEffect
  // 如果数组中有多个元素，即使只有一个元素发生变化，React 也会执行 effect。
  useEffect(() => {
    navigation.setOptions({headerShown: false});   
    fetchPhotoListWith(page)    
  }, [navigation, page]);
```