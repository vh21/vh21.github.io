---
layout: post
title: 'Linux concurrency - 1: 簡介'
date: 2015-06-21 20:00
comments: true
tags: [linux, kernel, concurrency]
categories: linux
---

kernel 本身就是個 multithread concurrent 的系統。在 沒有適當保護下，存取共享資
源很容易發生 race condition。共享資源包含周邊IO及共用的變數資料結構等。可透過適
當的 synchronization 機制提供 critical section 來保護資源的存取。

個人認為 kernel 裡面與 concurrency 相關的機制大致分為以下幾類，

1. architecture support
 * barrier
 * atomic operation
2. busy-waiting lock：CPU loop 等待直到取得lock。critical section 中不允許睡眠
。通常用來在 SMP 下保護資料數據結構。有以下幾種，
 * atomic operation
 * spinlock
 * reader/writer spinlock
 * seqlock
3. lock-free synchronization： 透過區分 reader 與 writer 的角色以及 deferred
destruction 來減少 lock 的使用。
 * rcu
 * kfifo
 * percpu data
4. blocking synchronization：如果沒有拿到 lock 會進入睡眠。通常用來保護較大的
critical section 與存取 IO 相關的機制。可分為以下幾種，
 * semaphore
 * reader/writer semaphore
 * mutex
 * completion

註：

1. 本文同步刊載與 [Taiwan Linux Kernel Hacker 的 hackpad](https://twlinuxkernelhackers.hackpad.com/Concurrency-in-Linux-Kernel-JX3tiL7l2Uo)
2. [Taiwan Linux Kernel Hacker](https://www.facebook.com/groups/twlinuxkernelhackers/887765881260525/?notif_t=group_activity)
為 Facebook 上研究 Linux Kernel 的社團
