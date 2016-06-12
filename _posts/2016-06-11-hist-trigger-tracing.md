---
layout: post
title: 'Linux 4.7 導入 hist trigger tracing'
date: 2016-06-11 08:00
comments: true
tags: [linux, profile, tracing]
categories: linux
---

Linux-4.7 tracing 導入 hist trigger\[1\]\[2\]，簡單來說就是可針對 trace event 裡提
供的數據，取出任意欄位來統計製出做 histgram。舉例來說 kernel 的 `kmalloc`
event 記錄以下數據，

~~~
format:
    field:unsigned short common_type;   offset:0;   size:2; signed:0;
    field:unsigned char common_flags;   offset:2;   size:1; signed:0;
    field:unsigned char common_preempt_count;   offset:3;   size:1; signed:0;
    field:int common_pid;   offset:4;   size:4; signed:1;

    field:unsigned long call_site;  offset:8;   size:4; signed:0;
    field:const void * ptr; offset:12;  size:4; signed:0;
    field:size_t bytes_req; offset:16;  size:4; signed:0;
    field:size_t bytes_alloc;   offset:20;  size:4; signed:0;
    field:gfp_t gfp_flags;  offset:24;  size:4; signed:0;
~~~

假設想監控每個 process 透過 kmalloc 配置的記憶體總數量，可以透過 hist trigger
指定 key 為 `common_pid`，val 為 `bytes_alloc`。

~~~
# echo 'hist:key=common_pid:val=bytes_req' > /sys/kernel/tracing/events/kmem/kmalloc/trigger
~~~

一段時間後，可將結果倒出來，

~~~
# cat /sys/kernel/tracing/events/kmem/kmalloc/hist
# event histogram
#
# trigger info: hist:keys=common_pid:vals=hitcount,bytes_req:sort=hitcount:size=2048 [active]
#

{ common_pid:        803 } hitcount:          5  bytes_req:        771
{ common_pid:        802 } hitcount:         17  bytes_req:       5931
{ common_pid:        804 } hitcount:         17  bytes_req:       5699
~~~

更方便的是，key 可選擇隨意的參數搭配，當我們想統計 context switch 次數時，就可
將 `prev_pid` 與 `next_pid` 一起當作 key。或者也可將 stacktrace 直接當作 key!
提供了一些想看的數據統計的想像空間。

如同 Brendan Gregg 所言\[2\]，

~~~
Can't enhanced BPF do this too?

Yes, if you program it to. I feel like we've been waiting for advanced tracing 
in Linux for years, and now two busses have arrived at the same time. This 
overlap concern was raised and discussed on lkml, and it was eventually deemed 
that hist triggers was different enough to be included.

BPF can do a lot more than hist triggers, although it also requires a lot more 
effort. Hist triggers is a simple enhancement to trace, for some common 
system-wide observability. Hist triggers is also lightweight to enable in custom
ways: you just need a shell, whereas BPF typically needs one (or more) compilers. 

~~~

由於 hist trigger 不需要另外的 tool support，使用時提供了一些方便性。對於像嵌入
式系統開發者感觸更深。記得之前 perf 剛出來時為了要在 uclibc 的環境測試，光是編
譯 perf 程式本身就必須花費一番工夫！

題外話，之前在 BPF tool 這邊曾試想過該怎麼解決 嵌入式環境的問題，想了幾個方法，

1. 利用現有方案，如 perf, ply 或是其他 solution 在 bpf 的整合。
2. 利用 kernel samples 下的方法，寫 c。
3. 擴充 bcc tool，讓她可以控制 remote target，把編好的 bytecode object 丟到
target daemon.

試著編譯 ARM 平台來測試，但是編譯過程中遇到一個問題，hist trigger 會有
`CONFIG_ARCH_HAVE_NMI_SAFE_CMPXCHG` 的 dependency，頭痛的是 ARM 與 MIPS 都沒有
這個 option。由於在 qemu 測試，姑且先偷懶自行開啟這個 config 測試看看。單純在
qemu 環境做簡單測試是沒什麼問題。看了一下change log\[3\]，有可能是 tracing map
的實現上的需求，改天再來瞭解看看。

~~~
Changes from v2:
 - reimplemented tracing_map, replacing bpf_map with nmi-safe/lock-free map
~~~

總結來說個人覺得之後應該有機會常用到，畢竟有時有個隨手可用的解決方案而不需要另
外編譯軟體進 rootfs 省了很多麻煩。

# TODO

* 瞭解一下 `ARCH_HAVE_NMI_SAFE_CMPXCHG` 是否為必要？在 ARM/MIPS 平台該如何解決。
* 看起來應該也可以結合 kprobe event。這樣的話可紀錄的數據範圍應該更大了。作為一
個 lightweight solution 應該是個不錯的選擇。

# Reference

1. [kernel documentation: event tracing](https://www.kernel.org/doc/Documentation/trace/events.txt)
2. [Brendan Gregg: Hist Triggers in Linux 4.7](http://www.brendangregg.com/blog/2016-06-08/linux-hist-triggers.html)
3. [tracing: 'hist' triggers](https://lwn.net/Articles/678661/)

