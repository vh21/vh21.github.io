---
layout: post
title: 'RISC-V Linux (1)'
date: 2017-04-23 08:00
comments: true
tags: [linux, riscv]
categories: linux
---

RISC-V 最近打算做 Linux Kernel upstreaming。順便找了一下資源並測試一下。主要資
訊在[官網](https://riscv.org)都可以找到：

1. [CPU SPEC](https://riscv.org/specifications/)
2. 先編譯 [riscv-tools](https://riscv.org/software-tools/#quickstart)。其中編譯
   出來的 toolchain 是 elf 版本，linux 版需要另外編譯。
3. 另行編譯 [linux-gcc](https://riscv.org/software-tools/#building-linux)。並且
   抓 [riscv-linux](https://github.com/riscv/riscv-linux) 下來編譯。

目前編譯 kernel 時遇到一個 compile error

~~~
  AS      arch/riscv/kernel/head.o
arch/riscv/kernel/head.S: Assembler messages:
arch/riscv/kernel/head.S:75: Error: unrecognized opcode `sfence.vma'
scripts/Makefile.build:395: recipe for target 'arch/riscv/kernel/head.o' failed
make[1]: *** [arch/riscv/kernel/head.o] Error 1

~~~

看起來是新加入的指令。可能看一下 assembler 這邊的 branch 是否正確。

其他資源：

1. [coldnew: 使用 Busybox 建立 RISC-V 的迷你系統](http://coldnew.github.io/blog/2015/busybox_for_riscv_on_qemu/)
