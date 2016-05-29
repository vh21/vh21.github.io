---
layout: post
title: '編譯 kernel ebpf 的範例'
date: 2016-05-06 20:00
comments: true
tags: [linux, kernel, ebpf]
categories: linux
---

kernel 本身有自帶 ebpf 的範例程式，該程式中將所有 ebpf 會用到的相關物件放在
`xxx_kern.o`，經由 LLVM 編譯成 ebpf 的 bytecode object，再透過 kernel syscall
載入執行。

# 編譯 ebpf samples
ebpf 的範例程式在 `samples/bpf` 下，但是選擇 `CONFIG_SAMPLES` 時卻不會被編譯
進去，需要自行編譯。如 [1] 所示，

~~~
  make samples/bpf/
~~~

但是 clean 時沒找到對應指令，只好用類似 kernel module 的方式清除了。

~~~
make M=samples/bpf clean
~~~

幾點注意，

1. 須先 install kernel header。
2. 目前建議使用 LLVM 3.7

# Reference
1. [LKML: make samples/bpf/ using gcc instead of clang/llvm](https://lkml.org/lkml/2016/1/27/48)
