---
layout: post
title: 'Linux concurrency - 4: Kernel atomic API'
date: 2015-07-26 14:00
comments: true
tags: [linux, kernel, concurrency, atomic]
categories: linux
---
Linux kernel 為整數資料提供兩種簡單的 atomic 操作，一種是 atomic counter，另一
是 atomic bitmask operation。

## atomic counter

原則上 atomic counter 實現我們只需要一個 `int` 或是 `long` 的變數，透過 atomic
instruction 來包裝成 RMW 的 API 即可。Kernel 中是透過 `atomic_t` 來宣告一個
 atomic counter，它的定義如下，

~~~
typedef struct {
    int counter;
} atomic_t;
~~~

封裝在 data structure 中除了有較佳的擴展性外，可避免使用者沒有使用 atmoic API[18]
直接透過如下列的程式碼存取，因而被轉譯成一般的 load/store 而造成問題。

~~~
atomic_t my_atom;
my_atom++;
~~~

atomic counter API 提供下列幾種操作（`atomic_{read, set}()` 表示 `atomic_read()`
與 `atomic_set()` 兩組 API，其他雷同），

* `ATOMIC_INIT()`：變數初始化。
* `atomic_{read, set}()`：讀或寫通常可由單個 load/store 指令完成，所以通常只是 C
的實現版本。

    ~~~
    #define atomic_read(v)  ACCESS_ONCE((v)->counter)
    #define atomic_set(v,i) (((v)->counter) = (i))
    ~~~

* `atomic_{add, sub, inc, dec}()`：針對 counter 做加減動作， `atomic_{inc, dec}()`
  為加/減 1 的版本。函數實現內部不含 memory barrier。需要確保 counter 與其他變
  數存取順序時可自行在前後加上 `smp_mb__{before/after}_atomic()` 。

* `atomic_{add, sub, inc, dec}_return()`：加減 counter 值並回傳新值。函數實現內
  部在 atomic operation 前後有包含 memory barrier。

* `atomic_{inc, dec, sub}_and_test()/atomic_add_negtive()`：`atomic_xxx_and_test()`
   加減 counter 值並返回布林值檢查 counter 是否為 0。`atomic_add_negtive()` 則是
   檢查新的 counter 是否為負數。函數內含 barrier。

* `atomic_xchg()/atomic_cmpxchg()`：`atomic_xchg()` 將 counter 值與參數交換並返
  回舊值。`atomic_cmpxchg()` 將參數給的舊值與 counter 比對是否相同，相同的話才寫
  入新值，返回原本 counter 中的舊值。函數內含 barrier。

* `atomic_add_unless()/atomic_inc_not_zero()`：條件判斷是否與某個參數的值相同來
  決定是否加 counter 值。函數內含 barrier。

另外有 `atomic_long_t` 版本的型態，在此不另行介紹。

## atomic bitmask operation

bitmask 大致上與 atomic counter 雷同，只是操作的動作從加減變成針對 bit 的邏輯運
算。bitmask 操作的變數是 `unsigned long` 的整數型態。大致分類如下：

* `{set, clear, change}_bit()`：如同 API 的名稱一樣，是用來改變 `unsigned long`
  整數中某個 bit 的函數。所有的操作是 atomic 但是不含 memory barrier。

* `test_and_{set, clear, change}_bit()`：與上幾個函數類似，但是會返回 boolean
  值表示修改前的 bit 值是否為 1。實作內含 memory barrier。

* `test_and_set_bit_lock()/clear_bit_unlock()`：atomic bitops 也可拿來實作
  bit lock。之前使用上都是透過 `test_and_set_bit()` 與
  `smp_mb__before_clear_bit(); clear_bit();` 來處理 lock/unlock 的動作，[19] 的
  commit 將這兩個操作抽出來當作 bit lock 的 API。由 commit 可看到有用到的地方大
  致是 page lock、buffer lock、bit_spin_lock、tasklet locks。

* 最後，上述實作有 non-atomic 的版本（加上 `__` prefix 的版本，如 `__set_bit()` ）
 。這些 API 是在在適當 lock 保護的區域內使用。

## Reference

17. [Semantics and Behavior of Atomic and Bitmask Operations](https://www.kernel.org/doc/Documentation/atomic_ops.txt)
18. [kernelnewbie maillist: Re: atomic_t](http://www.spinics.net/lists/newbies/msg29573.html)
19. [lock bitops](http://lwn.net/Articles/233390/)

## 註：

1. 本文同步刊載與 [Taiwan Linux Kernel Hacker 的 hackpad](https://twlinuxkernelhackers.hackpad.com/Concurrency-in-Linux-Kernel-JX3tiL7l2Uo)
2. [Taiwan Linux Kernel Hacker](https://www.facebook.com/groups/twlinuxkernelhackers/887765881260525/?notif_t=group_activity)
為 Facebook 上研究 Linux Kernel 的社團
