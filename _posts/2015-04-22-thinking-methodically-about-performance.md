---
layout: post
title: 'Thinking Methodically about Performance 整理'
date: 2015-04-22 21:00
comments: true
tags: [arm mips as]
categories: methodology
---

原作來源：https://queue.acm.org/detail.cfm?id=2413037
作者：Brendan Gregg, Joyent

現代的系統由於牽涉到多個 component 間的交互作用，使得系統效能分析越來越難鑑識出 bottleneck 的來源。
以往的 methodolody 有的缺乏科學化分析，有的則是曠日費時。所以作者提出一個可以快速分析的方法，
`USE Method`，希望可以在效能分析的早期，快速的定位出系統性 bottleneck 的方向。之後再搭配其他的
methodology 找出真的問題。

USE 表示 utilization、saturation 及 error。在效能分析的早期，跑完 problem-statement method 之後，
首先，我們可以大略描繪出整個系統的 function block diagram ，列出資源列表。並且針對每個 resource
檢查它們的 utilization、saturation 與 error 的情形是否有異常。

舉例來說，我們可大概列出如下的 resource list (擷取自原文）

* CPUs—Sockets, cores, hardware threads (virtual CPUs).
* Main memory—DRAM.
* Network interfaces—Ethernet ports.
* Storage devices—Disks.
* Controllers—Storage, network.
* Interconnects—CPU, memory, I/O.

針對每個 resource 尋找跟 U/S/E 相關的量化指標，如 CPU 使用率，記憶體使用量可視為 saturation 指標、
network error number 等。有些較難分析的部分也許必須自行客製化相關的 profiling tool 與指標。

最後就是針對這些量測出來的數據分析了。作者對於指標的解讀提出了一些建議，

* utilization： 100% 的使用率通常是瓶頸的徵兆。高使用率（例：超過 60％）有可能開始造成其他問題的發生。
  * 量測通常是取一對時間平均，所以測量到高使用率，比如說總計 60% 使用率，可能隱藏短時間達到 100% 使用率。
  * 再來，有些系統資源操作中通常無法被中斷，如硬碟，一旦使用率在60%以上，在 queue 等待的延遲可能會越來越頻繁越顯著。
* saturation： 任何程度的 saturation 都可能是問題（非零）。可以以一個等待 queue 的長度或是花在等待 queue 的時間來測量。
* error：非零的 error count 是值得調查的，特別是如果他們在效能很差時還持續增加。
  * error count 先檢查，因為他們通常比起使用率與飽和度更容易且更快解譯。

USE method 辨認出可能是系統瓶頸的問題。不幸的系統可能出現超過一個 performance issue，所以第一個你找到的可能是
其中一個但不是真正的問題。你可以使用更進一步的 methodology 來調查每一個發現的結果。

