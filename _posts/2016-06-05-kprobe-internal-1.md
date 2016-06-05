---
layout: post
title: 'kprobe 實現機制'
date: 2016-06-05 08:00
comments: true
tags: [linux, kernel, profile, kprobe]
categories: linux
---
kprobe 允許使用者透過編寫 kernel module，動態地對想要 probe 的位址註冊處理函式
。kernel 執行到這個指令時，會去呼叫所註冊的處理函數。

處理函數分成幾個部分，

1. `pre_handler`：被 probe 的位址被執行前觸發。
2. `post_handler`：probe 的指令執行後觸發。
3. 其他情形不在此介紹。

kprobe 有幾種變種，`jprobe` / `kretprobe` / `uprobe`。大概瞭解一下 kprobe 的實
現機制。

當註冊 kprobe 時會將被 probe 的位址改成 software breakpoint，x86 是 透過 `INT 3`
，arm 則是自訂一個未定義的指令。透過觸發 undefined instruction exception 來處理
。

於是程式執行到這個位址時變成執行 break 而跳進 exception handler。kprobe 此時就
在 exception handler 先執行 `pre_handler()`，接著分析原指令並模擬原指令的操作，
最後接著執行 `post_handler()`。
