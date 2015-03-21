---
layout: post
title:  "Linux kernel uapi header file"
date:   2014-11-21 14:34:25
tags:   linux, kernel, uapi
---

Linux在3.7以後把很多header file移到 include/uapi或是arch/xxxx/include/uapi下，感覺起來要追define變得很麻煩，不太清楚為了什麼做這個修改，找了一下看到LWN有一篇文章介紹[The UAPI header file split](http://lwn.net/Articles/507794/)。看起來是個不賴的修正。重點整理如下，

## 原先待解決的問題
要解決include recursive的問題。如原先要在A.h加inline function時發現裡面用到的某些struct被定義在B.h，而B.h中又有inline function需要用到A.h的struct，就會造成recursive include

## 概念
把userspace API的檔案獨立到 include/uapi跟arch/xxxx/include/uapi下，舉例來說本來header中

```c
/* Header comments (copyright, etc.) */
#ifndef _XXXXXX_H     /* Guard macro preventing double inclusion */
#define _XXXXXX_H

[User-space definitions]
#ifdef __KERNEL__
[Kernel-space definitions]
#endif /* __KERNEL__ */

[User-space definitions]
#endif /* End prevent double inclusion */
```

換成如下兩個檔案，

a. kernel space的東西放在原本path並include uapi下同檔名檔案

```c
/* Header comments (copyright, etc.) */

#ifndef _XXXXXX_H     /* Guard macro preventing double inclusion */
#define _XXXXXX_H

#include <include/uapi/path/to/header.h>

[Kernel-space definitions]

#endif /* End prevent double inclusion */

/* Header comments (copyright, etc.) */
```

b. uapi下的檔案，include guard稍微有點不同

```c
#ifndef _UAPI__XXXXXX_H     /* Guard macro preventing double inclusion */
#define _UAPI__XXXXXX_H

[User-space definitions]

#endif /* End prevent double inclusion */
```

## 好處

* 簡化與減少kernel-only header的size
* 現在kernel header有的是檔案中一部分export給userspace用。這做法簡化header file間複雜的交互相依性。
* 處理userspace ecosystem的人較容易追蹤API的變更，透過git來追蹤uapi下的log很容易在每個kernel的release週期就知道做了哪些修改

## 處理過程遇到的問題
* 檔案太多，必須用scipt自動化
* 如何讓Linus接受？共74 commits，其中65 commits是透過script做的，共修改超過3500檔案，超過30萬行
