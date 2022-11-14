---
title: "Copy & MutableCopy"
# description: ""
date: 2022-11-14T16:06:23+08:00
draft: true
# tags: []
# series: []
# series_order: 1

# summary: ""
---

- `copy` 是复制出不可变对象
- `mutableCopy` 是复制出可变对象；
- 复制出来的对象互不影响。
- `copy` 复制不可变对象属于浅拷贝（浅拷贝就只是地址拷贝）
- `copy` 复制可变对象属于深拷贝（需要复制出一份不可变对象，以免之前可变对象变更影响复制出来的对象）
- `mutableCopy` 属于深拷贝（重新申请一份内存和指针）

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSArray * arr = @[@1, @2];
    NSArray *arrCopy = [arr copy];
    NSMutableArray * arrMCopy = [arr mutableCopy];
    
    [arrMCopy addObject:@3];
    NSLog(@"%p, %p, %p", arr, arrCopy, arrMCopy);
    
    NSMutableArray * marr = [NSMutableArray arrayWithArray:arr];
    NSArray *marrCopy = [marr copy];
    NSMutableArray * marrMCopy = [marr mutableCopy];
    
    NSLog(@"%p, %p, %p", marr, marrCopy, marrMCopy);
}
```

[copy和mutableCopy](copy%E5%92%8CmutableCopy%20f091420c2401458c861f55a643afc306/copy%E5%92%8CmutableCopy%207e83f671428e4210989a80ff9dbd2243.csv)