---
layout: post
title: 'Linux concurrency - 3: architecture atomic instruction'
date: 2015-07-26 10:00
comments: true
tags: [linux, kernel, concurrency, atomic]
categories: linux
---

我們透過變數來維護程式的狀態及流程，SMP 下同時多個 core 共同存取 share data 時
，程式設計上除了重入性以及可能被亂序執行的考量外，另一個就是存取時需確保操作是
atomic：表示說我們對於某個共享記憶體的操作，相對於系統的其他部分來說是完整完成
不可中斷的 [10][11] 。包含 load/store atomic 與 Read-Modify-Write(RMW) atomic。
我們可以從 [11] 的例子對於 64-bit share memory 的處理很明顯的了解這個概念，以及
可能發生的問題（race condition）。

在 single-core 下只要關閉 global interrupt，防止 CPU 被 preempt，就可以確保 atomic
，但是在 multicore 時就需要有特殊的硬體資源（且關閉 global interrupt 會影響
response time）。這當中很大一部分會需要確保 RMW operation 是 atomic，特別是在
lock 與 counter 的實作上面。

通常 CPU 會提供有別於一般 load/store memory 的特殊指令來達成 RMW atomic。介紹其
中兩類，

* compare-and-swap (CAS) / swap (SWP) [12]：CAS 寫入時同時提供舊值（之前讀出來的
值），如果舊值和記憶體裡的相同，就將新值寫入，確保 RMW 的動作完整性，如 x86 的
CMPXCHG 指令。SWP 在寫入時讀回記憶體的值，可用於與之前讀出的值比對。如 ARMv6 之
前的 SWP。

* load-link/store-conditional (LL/SC) [13]：這類的指令是成對使用。在 LL 讀取記
憶體的值，同時標記進入 exclusive 狀態，SC 寫入時清除 exclusive 狀態。在
exclusive 狀態時如有其他 core 較早執行寫入，導致 exclusive 狀態被清除，則忽略這
次的寫入並返回失敗。程式可在下一行發現失敗時重新執行 LL/SC 的動作直到成功為止。
POWERPC、MIPS、ARM （ARMv6 以上）等提供這類的指令。

其他還有如 `test-and-set` [14] 等，不在此作介紹。

ARM 早期 CPU 使用的 `SWP` 指令在 multicore 的系統中有一些效能瓶頸[15]，所以 ARMv6
以上換成 LL/SC 的指令，也就是 `ldrex/strex`。x86 本身支援記憶體存取的指令可加上
`LOCK` prefix [16] 轉成 atomic 的存取。

透過 atomic instruction 我們可以實現一些基本的 counter 操作與簡單的 SMP lock，如
spinlock、rwlock 等，確保程式狀態維護的正確性。然而當競爭相同 lock 的 CPU 變多時
，卻可能會造成 scalability 不佳的問題。

## Reference

10. [wiki: atmoic operation](https://en.wikipedia.org/wiki/Linearizability)
11. [Atomic vs. Non-Atomic Operations](http://preshing.com/20130618/atomic-vs-non-atomic-operations/)
12. [wiki: compare and swap](https://en.wikipedia.org/wiki/Compare-and-swap)
13. [wiki: Load-link/store-conditional](https://en.wikipedia.org/wiki/Load-link/store-conditional)
14. [wiki: test and set](https://en.wikipedia.org/wiki/Test-and-set)
15. [A.1.2 Limitations of SWP and SWPB in ARM®  Synchronization Primitives](http://infocenter.arm.com/help/topic/com.arm.doc.dht0008a/DHT0008A_arm_synchronization_primitives.pdf)
16. [stackoverflow: Illegal instruction in ASM: lock cmpxchg dest, src](http://stackoverflow.com/questions/1746344/illegal-instruction-in-asm-lock-cmpxchg-dest-src/1746603#1746603)

## 註：

1. 本文同步刊載與 [Taiwan Linux Kernel Hacker 的 hackpad](https://twlinuxkernelhackers.hackpad.com/Concurrency-in-Linux-Kernel-JX3tiL7l2Uo)
2. [Taiwan Linux Kernel Hacker](https://www.facebook.com/groups/twlinuxkernelhackers/887765881260525/?notif_t=group_activity)
為 Facebook 上研究 Linux Kernel 的社團
