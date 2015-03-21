---
layout: post
title: 'Linux kernel CURRENT implementation'
date: 2015-03-21 17:00
comments: true
tags: linux, kernel, process
categories:

---

Linux kernel程式裡利用`current`來指向當前的process的data structure，在
Linux Device Drivers chapter 2面提到，

```
The current pointer refers to the user process currently executing.
During the execution of a system call, such as open or read, the current
process is the one that invoked the call. Kernel code can use process-specific
information by using current, if it needs to do so. [...]
```

實際上它在整個kernel裡面並非唯一的global variable，而是每個thread有自己的storage。
他是怎麼實現的呢？在`include/asm-generic/current.h`裡，

```c
#define get_current() (current_thread_info()->task)
#define current get_current()
```

所以最終是透過每個thread的`current_thread_info()`來找到task pointer。記得以前看過mips是
透過特殊的register `gp` 來實現，所以在kernel中gp也是有特殊含義的。

```c
static inline struct thread_info *current_thread_info(void)
{
        register struct thread_info *__current_thread_info __asm__("$28");

        return __current_thread_info;
}

```

來看一下arm的case，

```c
static inline struct thread_info *current_thread_info(void)
{
        register unsigned long sp asm ("sp");
        return (struct thread_info *)(sp & ~(THREAD_SIZE - 1));
}
```

看起來是直接透過原本的計算取得task information pointer。另外arm64與arm的實現相同。
關於task structure的描述可見[Process Descriptor and the Task Structure](http://www.makelinux.net/books/lkd2/ch03lev1sec1)


