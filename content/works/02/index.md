---
title: "时长云存逻辑梳理优化"
date: 2022-11-13

tags: ['WorkNote']
---

当时有这样一个需求，用户可以设置当天的某个时间段，进行录像任务，当用户设置的时间段时，已设置过的时间段需要置灰不可选，这里时间段涉及两个点，一是开始时间，二是时长，而时长的设置是有限制的，它不能超过下一个计划的开始时间，不能大于剩余可设置时长，不大于剩余可设置时长，这个判断很简单，难点在于，根据给出的开始时间找到下一个录像的开始时间，最后计算出可设置时长。

一开始的想法是，把数据分为了，天、时对象。天包含所有不可用的小时及对应的小时对象，小时包含当前小时被占用的分钟区间。
这样在封装数据的时候，小时默认有个初始60分钟，根据被占用分钟区间相减，如果初始值被减到0则表示该小时不可用🚫。天的对象不可用就比较简单，如果小时对象数量小于24表示可用，如果等于24则找到可用的小时，没找到则当天不可用。

至此，解决了页面对应天，小时，分钟的颜色置灰显示。还剩下一个问题，计算可设置的时长。

所以根据数据封装的格式，很自然的把用户选定的开始时间分为，小时和分钟，先从小时开始找，如果当天不可用的小时不包含选定的小时，那处理比较简单，将数据排好序，找到第一个大于当前小时的数据，做减法，得出的时间就是可设置时才，不过这里就要注意⚠，没找到要从跨天数据数据中找。

另一种情况，如果当天不可用的小时包含选定的小时，这里就要判断选定的分钟在当前小时哪个位置，如果在最后，这又要处理跨小时数据。

想想就很复杂，后来再看这里的代码终于想通，其实，就是给定一个开始时间，找到刚好比开始时间大的数据，然后做个减法的问题。时间对比除了可以逐级对比小时、分钟，还可以直接将小时换算成分钟，然后对比分钟啊。所以把不可用的时间段铺平再一天的时间线上，这样对比就简单得多了。就没有跨小时的说法了。而跨天，只需查一下第二天是否有计划，拿到第一个计划时间段，加入当天就行了。

豁然开朗！😊

用原数据对比简直简单…😓！！！
