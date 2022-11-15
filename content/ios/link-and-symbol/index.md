---
# title: 链接与符号 Link&Symbol
title: "Link and Symbol"
# description: ""
date: 2022-11-14T17:45:43+08:00

tags: ["iOS", "Mach-O"]
# series: ["IOS"]
# series_order: 8
---

# Mach-O

Mach-O格式全称为Mach Object文件格式的缩写，是MacOS或者iOS上可执行的程序格式，类似于Windows上的PE格式 (Portable Executable)，linux上的ELF格式 (Executable and Linking Format)。

<!-- more -->

**Mach-O文件的分类**

- `Executable` 应用可执行文件
- `Dylib Library` 动态链接库（又称DSO或DLL）
- `Static Library` 静态链接库
- `Bundle` 不能被链接的Dylib，只能在运行时使用dlopen( )加载，可当做macOS的插件
- `Relocatable Object File` 可重定向文件类型

## Mach-O文件的组成

Mach-O文件主要包括三部分内容： Header(头部)、Load Commands(加载命令)、Data(数据区)

### Header

指明了 CPU 架构、大小端序、文件类型、Load Commands 个数等一些基本信息，Headers 能帮助校验 Mach-O 合法性和定位文件的运行环境，64位架构为例，[Header结构](https://opensource.apple.com/source/xnu/xnu-792/EXTERNAL_HEADERS/mach-o/loader.h)定义如下

```objectivec
struct mach_header_64 {
    uint32_t    magic;        /* mach magic number identifier 魔数，用于快速确认该文件用于64位还是32位 */
    cpu_type_t    cputype;    /* cpu specifier，CPU**类型，比如 arm */
    cpu_subtype_t    cpusubtype;    /* machine specifier，对应的具体类型，比如arm64、armv7 */
    uint32_t    filetype;    /* type of file，文件类型，比如可执行文件、库文件、Dsym文件，demo中是2 `MH_EXECUTE`，代表可执行文件*/
    uint32_t    ncmds;        /* number of load commands 加载命令条数 */
    uint32_t    sizeofcmds;    /* the size of all the load commands  所有加载命令的大小 */
    uint32_t    flags;        /* flags 标志位 */
    uint32_t    reserved;    /* reserved  保留字段 */
};
```

**filetype**

- OBJECT，指的是 .o 文件或者 .a 文件；
- EXECUTE，指的是 IPA 拆包后的文件；
- DYLIB，指的是 .dylib 或 .framework 文件；
- DYLINKER，指的是动态链接器；
- DSYM，指的是保存有符号信息用于分析闪退信息的文件。

**待更新... 感觉还没学会😭**

# LLVM-NM


**参考**
[了解Mach-O文件](https://juejin.cn/post/7066791636205830181)
[2](https://juejin.cn/post/7045928743310721037)

