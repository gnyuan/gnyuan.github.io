---
layout: post
title: make,gcc以及项目架构小讲
tags: make gcc Linux configure build
categories: Linux
---


# 简介

对于中型以上规模的项目，如果没有合理的架构设计，对项目开发以及维护将是一个灾难。本文主要介绍了合理的架构设计以及通过什么工具落地到源码管理层面。

下面，以“嵌入式软件项目管理”课中李云老师写的ClearRTOS为例，讲讲好的软件架构如何反应在代码层面中。

# ClearRTOS代码结构

```
├── COPYING3
├── build
│   ├── Makefile
│   ├── c++.rule
│   ├── c.rule
│   ├── release
│   │   ├── err2str.exe
│   └── unitest
├── code
│   ├── application
│   │   ├── event
│   │   │   ├── Makefile
│   │   │   └── main.c
│   │   ├── mutex
│   │   │   ├── Makefile
│   │   │   └── main.c
│   │   ├── queue
│   │   │   ├── Makefile
│   │   │   └── main.c
│   │   ├── semaphore
│   │   │   ├── Makefile
│   │   │   └── main.c
│   │   └── task
│   │       ├── Makefile
│   │       └── main.c
│   ├── platform
│   │   ├── common
│   │   │   ├── inc
│   │   │   │   ├── alignment.h
│   │   │   │   ├── ...
│   │   │   └── src
│   │   │       ├── Makefile
│   │   │       ├── alignment.c
│   │   │       ├── ...
│   │   └── kernel
│   │       ├── arch
│   │       │   ├── arm
│   │       │   │   └── share
│   │       │   └── x86
│   │       │       ├── share
│   │       │       │   ├── inc
│   │       │       │   │   └── ioport.h
│   │       │       │   └── src
│   │       │       │       ├── Makefile
│   │       │       │       ├── ioport.c
│   │       │       │       └── robjs
│   │       │       │           ├── ioport.dep
│   │       │       │           └── ioport.o
│   │       │       └── simulator
│   │       │           ├── inc
│   │       │           │   ├── clock.h
│   │       │           │   ├── ...
│   │       │           └── src
│   │       │               ├── Makefile
│   │       │               ├── clock.c
│   │       │               ├── ...
│   │       ├── board
│   │       │   └── simulator
│   │       │       └── src
│   │       │           ├── Makefile
│   │       │           └── inventory.c
│   │       ├── core
│   │       │   ├── inc
│   │       │   │   ├── bitmap.h
│   │       │   │   ├── config.h
│   │       │   └── src
│   │       │       ├── Makefile
│   │       │       ├── bitmap.c
│   │       │       ├── device.c
│   │       │       ├── ...
│   │       └── driver
│   │           └── ctrlc
│   │               ├── README
│   │               ├── inc
│   │               │   ├── ctrlc.h
│   │               │   └── errctrlc.h
│   │               └── src
│   │                   ├── Makefile
│   │                   └── ctrlc.c
│   └── tools
│       └── err2str
│           ├── Makefile
│           └── err2str.cpp
├── tests
│   ├── application
│   ├── framework
│   └── platform
│       ├── common
│       │   ├── Makefile
│       │   ├── unitest_clib.c
│       │   ├── unitest_dll.c
│       │   └── unitest_dllht.c
│       └── kernel
│           └── core
│               ├── Makefile
│               ├── unitest_bitmap.c
│               └── unitest_task.c
└── tools
    └── lint
        ├── au-sm123.lnt
        ├── co-gnu3.lnt
        ├── options.lnt
        └── std.lnt
```
`build`放的是构建出来的可执行文件（包括单元测试、集成测试等测试过程的可执行文件）

`code`放的是项目源码（个人觉得src比较好听^_^）,重点来了，这里分应用层（`application`，写核心业务逻辑）,平台层（`platform`，为项目的基础设施，调用底层的服务，并为其上层提供服务）以及工具层（`tools`，公用工具，如错误信息处理）。

`tests`放单元测试代码，它对应`code`的每一个文件。

每一个装`.c`的目录，都有一个Makefile，这非常方便写小巧精致的Makefile，其好处不言而喻。而`build`中的Makefile可以写成主控类型的，它可以控制其他目录下的Makefile，使得修改完代码后，一键(make unittest)单元测试、一键集成测试、一键回归测试成为可能！

# 怎么写错误代号

这个问题对维护项目或者debug非常重要！有些项目，错误代码竟然是从000001开始往下编，如果项目里只有一个人还好，多个人同时写错误代码往往会出现同一个代码被重复使用场面。那么，正确的错误代码是怎样的呢？

1. 给错误代码分层级。譬如，前4位是模块号，接着4位是功能号，末尾是顺序号。这样，不同的开发人员同时开发各自的功能点，产生的错误代号也不可能重复；重要的是，以后看错误代码报的错，直接可以定位到功能点。非常方便！
2. 错误信息中，加入当前代码文件名及当前行数的信息，如果可以的话，模仿JAVA WEB项目，把报错语句的前后10行打出来，也是非常方便的。



# gcc的常用参数

gcc(GNU C Compiler)即C语言的编译器。这些指令在Makefile中使用得最多了（一般都在makefile中编译，才能保留它的编译依赖什么的）。使用它其实也不复杂，有以下几个重要参数：

`-o`: 编译出的文件的名字，如xxxx.o, 一般写在gcc编译命令的最后面。

`-c`: 对于单个文件功能，只编译，不链接。（A文件调用了外部函数F，那么，只要它include了F的头文件，就可以用-c把A文件编译成二进制文件。即只要A文件知道它要调用的函数需要传入什么，它会返回什么就够了。）

`-g`: 使得可以调试

`-Wall`: 所有warning信息都可以出

`-I`: 编译时，指定INCLUDE文件，找到使用的头文件的目录.（代码中用`#include “xxx.h”`把头文件包含进来，那么这个头文件在哪个目录下呢？用这个参数来指定）

`-L`: 使用-l时，指定它搜索的目录

`-l`: 指定使用具体的LIB名 -lm(数学库libm.o) -lzmq(ZMQ库libzmq.o) libxxx.o libxxx.so 
> 生成静态库	"gcc -c hellos.c -o hellos.o>  
> ar cqs libhellos.a hellos.o"
> > 使用静态库	gcc -static -L (include path) -lhellos
>> 生成动态库	gcc -c -shared -fPIC hellod.c -o libhellod.so或hellod.dll
>> 使用动态库	gcc -L (include path) -lhellod

`-E`: 选项把宏展开输出到标准输出

`-S`: 把文件翻译成汇编并写入.s文件

`-D`: -DMAR表示打开MAR这个宏 -DMAR=VALUE表示设置这个宏的值

因此，典型的gcc编译过程是这样的

```
#用a.c编译a.o
gcc -g -Wall -c a.c -o a.o
#用b.c编译b.o,它还使用了数学库math
gcc -g -Wall -c b.c -lm -o b.o 
#用a.o,b.o链接成c
gcc -g -Wall a.o b.o -o c 
```



# 未完待续

...

