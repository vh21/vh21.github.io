---
layout: post
title: 'Linux concurrency - 5: Spinlock, the busy-waiting lock'
date: 2015-08-18 20:00
comments: true
tags: [linux, kernel, concurrency, lock, spinlock]
categories: linux
---
spinlock 基本作法就是 busy waiting loop，一直等到指定的 lock 被釋放之後，取得
lock 才繼續進行 critical section 的操作。由於深度依賴 atomic instruction 的支援
，primitive 真正的實現都是在每個 architecture 中
（arch/ARCH/include/asm/spinlock.h），原始的實作有如 [20] 中的 pseudo code，

```
   typedef int SpinLock;

   void InitLock(SpinLock *L)
   {   *L = 0; }

   void Lock(SpinLock *L)
   {   while (TestAndSet(L))
           ;
   }

   void UnLock(SpinLock *L)
   {   *L = 0; }
```

相較於 atomic counter， 透過`spin_lock()/spin_unlock()` 我們提供一個可對更複雜
的資料結構操作的 atomic critical section。由於他小巧的實現，spinlock 可說是 SMP
實現的基礎建設，其他的機制如 semaphore 等都會需要它來維持共享資料與狀態的維護。

在 kernel 中情形稍微複雜一點，在 SMP 下資料除了可能在 thread context 間共享外，
還有可能在 ISR 或是 bottom half（softirq 與 tasklet）的 context 間被存取。根據共
享資料在不同 context 間被存取的可能性 [20]，spinlock 提供下列幾類 API，各有不同
的使用時機，

* `spin_lock()/spin_unlock()/spin_trylock()`：資料只在 user context 間 共享時使用
。
* `spin_lock_bh()/spin_unlock_bh()`：資料有可能在 user context 與 softirq/tasklet
間共享，primitive 會一併 disable softirq。
* `spin_lock_irqsave()/spin_lock_irqrestore()`：資料有可能在 user context 或
softirq/tasklet 與 ISR 間共享時使用，primitive 內會 disable IRQ。

原本 spinlock 的 algorithm 就像是演講 Q&A 時點人一樣，先舉手先發問且可重複舉手。
先搶先贏的方式有時會不公平，然而不公平造成的更大的問題的是當 lock 競爭激烈時所造
成的效能減損，在 [22] 中提到

```
On an 8 core (2 socket) Opteron, spinlock unfairness is extremely
noticable, with a userspace test having a difference of up to 2x
runtime per thread, and some threads are starved or "unfairly"
granted the lock up to 1 000 000 (!) times.
```

所以 Nick Piggin 在 2.6.25 中將 x86 實現的 algorithm 改為 ticket spinlock，現今
被許多 architecture 採用。另外為了防止存取亂序最佳化造成 critical section 裡資料
存取的動作被排到取得 lock 前或是 釋放 lock 後，在 lock 的實現上通常會包含
barrier [23]。但實際上不管是使用 lock 或是 barrier 對於效能及 scalability 都會造
成影響。

ticket spinlock 將取得與釋放 lock 的過程分成取得 `next` 及 `owner` 兩個步驟。整
個機制舉例來說就像是你去銀行辦事，假設只有一位行員（針對某個記憶體位址），要辦理
業務前（存取記憶體），要先抽號碼牌（取得屬於自己的`owner` 值，並把下一個 `owner`
加 1 ），之後所有等待的人只要盯著叫號機（`next`）的數字是否是自己的號碼
（`owner`）來決定下個辦理的人是否是自己。當前一個人的事情辦完後，行員把叫號機號
碼加 1（unlock 時會把`next` 加 1 ）也許會有鈴聲提醒號碼更新（ARM 會於此時執行
sev ）。這時候所有人都會抬頭（wakeup）看看新號碼是否跟自己的符合
（ local `owner` == `next`?）。等待的時候沒事的人可能會打個小瞌睡（ ARM 架構會於
此時執行 wfe 以節省 power）。

## Lock debugging
使用 lock 最常遇到的問題就是 deadlock，linux kernel 裡提供 `lockdep` 的機制，可
動態檢查 lock 被使用的時機與 primitive 是否正確，在有可能造成問題的 case 顯示
warning 並且列出相關的 call stack 以供檢查。詳細的資訊可參考 Linux Kernel
Hacking[24] 一書。有需要時需從 menuconfig 中 kernel hacking 裡開啟相關選項。

## Reference

20. [CIS 4307: SpinLocks](http://www.cis.temple.edu/~giorgio/cis307/readings/spinsem.html#4)
21. [Unreliable Guide To Locking](http://kernelbook.sourceforge.net/kernel-locking.pdf)
22. [Ticket spinlocks](https://lwn.net/Articles/267968/)
23. [Volatile considered harmful](https://www.kernel.org/doc/Documentation/volatile-considered-harmful.txt)
24. [Linux Kernel Hacks：改善效能、提昇開發效率及節能的技巧與工具](http://www.books.com.tw/products/0010624519)

## 註：

1. 本文同步刊載與 [Taiwan Linux Kernel Hacker 的 hackpad](https://twlinuxkernelhackers.hackpad.com/Concurrency-in-Linux-Kernel-JX3tiL7l2Uo)
2. [Taiwan Linux Kernel Hacker](https://www.facebook.com/groups/twlinuxkernelhackers/887765881260525/?notif_t=group_activity)
為 Facebook 上研究 Linux Kernel 的社團
