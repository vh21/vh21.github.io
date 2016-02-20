---
layout: post
title: 'mbed-OS 開發記錄'
date: 2015-10-21 20:00
comments: true
tags: [mbed, arm]
categories: mbed-os
---

最近 ARM 釋出了 mbed-OS source code 在 github 上。大概做了一些評估。由於還在
beta 版，其實開發途中遇到很多問題。把問題大致整理如下，

* 透過 `yotta install uvisor-helloworld` 時遇到 error，

    ~~~
    error: minar-platform-posix does not exist in the modules registry.
    Check that the name is correct, and that it has been published.
    ~~~

    看了一下並沒有這個 package，我想這可能是因為把 PC 當成 default target 的原因
    吧！透過下列指令將 default target 設定成 mbed-OS 支援的開發版後解決。

    ~~~
     $ yotta target -g frdm-k64f-gcc
    ~~~

* 找不到某個檔案（有點忘了，好像叫 `pins.cmake`）。透過 upgrade yotta
（0.7.0 => 0.8.5）就解決了。

* stm32 跑 uvisor-helloworld example 跑不起來，後來東改西改終於把 testcase 跑
起來了，滿腹狐疑最後過了幾天看到 ARM 的聲明，

    ~~~
    Currently this board and MCU is not as well supported as the K64F,
    especially if you want to use uVisor, networking or mbed TLS
    (which are not yet supported).
    ~~~

    好吧，它都已經說現在還在 beta 版了。
