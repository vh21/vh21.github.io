---
layout: post
title: '關於smp_load_acquire()/smp_store_release()'
date: 2015-04-25 12:30
comments: true
tags: [barrier, linux, kernel]
categories: linux
---

最近 trace linux kernel 時看到一對 function `smp_load_acquire()` 與 `smp_store_release()`，
一直很納悶這跟 read/write memory barrier 有什麼不同。

近年來由於 CPU 與 compiler 的發展，為了提供更高效能，只要資料間沒有 dependency 。 compiler 與 CPU
有可能會對程式執行的過程做 reordering，執行的 sequence 有可能不照你從 source 看到的樣子。尤其在
smp 的情況下，我們通常會加上一些 memory barrier 來防止這樣的 reordering 造成程式計算的錯誤。但這
其實會影響 執行的效能。但為何我們需要另外多兩個函式來做這件事呢？

最近看到一篇 LWN 的 [文章](https://lwn.net/Articles/576486/)，剛好可以解答疑惑，摘錄如下，

* smp_wmb (write memory barrier)：any write operation executed after the barrier
                                   will not become visible until all writes executed
                                   prior to the barrier are visible.
* smp_rmb (read memory barrier)： any reads executed before the barrier are forced
                                 to complete before any reads after the barrier can happen.
* smp_mb：a barrier for both read and write operations

簡單來說 `smp_wmb()` 主要就是防止 write operation 的 reordering，`smp_rmb()` 防止 read operation 的 reordering ，
而 `smp_mb()` 則是防止所有 memory access 的 reordering。

x86 提供另一種 CPU 的 support，可以做到

```
reads are ordered before reads, writes before writes, and reads before writes, but not writes before reads.
```

基本上除了原本的 read/write barrier 外，針對 read 跟 write 同時發生時的微調（rule: reads before writes）。文中
提到可能有一部份原本的 memory barrier 會改用這個 API 來實作。感覺起來 kernel 這幾年不斷針對 lock 與 barrier
方面進行更細部的控制阿。

