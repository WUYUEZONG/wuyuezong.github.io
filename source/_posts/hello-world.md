---
title: Hello World
---
### 1. 导入头文件
```objectivec
#import <pthread.h>
```
### 2. 创建 `pthread_t` 对象以及使用
```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 申明变量
    pthread_t thread;
    // 开启现场，执行任务
    pthread_create(&thread, NULL, run, NULL);
    // 设置子线程状态为 detach, 该线程运行结束后会自动释放所有资源
    pthread_detach(thread);
}

void *run(void *param) {
    NSLog(@"%@", [NSThread currentThread]);
    return NULL;
}
```
**`pthread_create` 函数参数说明**
```h
int pthread_create(pthread_t _Nullable * _Nonnull __restrict,
		const pthread_attr_t * _Nullable __restrict,
		void * _Nullable (* _Nonnull)(void * _Nullable),
		void * _Nullable __restrict);
```
1. 表示线程对象，指向线程标识符的指针 `&thread`
2. 表示线程属性，可赋值 `NULL`
3. 表示函数指针，在 `thread` 线程中要执行的任务
4. 表示函数参数

### `pthread` 其他相关方法

- `pthread_create()` 创建一个线程
- `pthread_exit()` 终止当前线程
- `pthread_cancel()` 中断另外一个线程的运行
- `pthread_join()` 阻塞当前的线程，直到另外一个线程运行结束
- `pthread_attr_init()` 初始化线程的属性
- `pthread_attr_setdetachstate()` 设置脱离状态的属性（决定这个线程在终止时是否可以被结合）
- `pthread_attr_getdetachstate()` 获取脱离状态的属性
- `pthread_attr_destroy()` 删除线程的属性
- `pthread_kill()` 向线程发送一个信号