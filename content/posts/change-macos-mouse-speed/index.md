---
title: "通过终端更改Macos鼠标的速度"
# description: ""
date: 2022-11-23T14:46:24+08:00
draft: false
tags: ["Macos", "鼠标"]
# series: []
# series_order: 1

summary: "通过终端更改Macos鼠标的速度"
---

查看当前鼠标速度

```shell
defaults read -g com.apple.mouse.scaling
```

更改鼠标速度， 10 表示要设置的速度，可以根据自己手感设置，重启生效。

```shell
defaults write -g com.apple.mouse.scaling 10
```


{{<alert>}}
如果设置完去系统设置中修改鼠标速度，则命令设置的速度会失效
{{</alert>}}