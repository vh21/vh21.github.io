---
layout: post
title: '自建 buildroot package'
date: 2016-05-07 20:00
comments: true
tags: [linux, buildroot]
categories: linux
---
最近想拿 buildroot 來測 Linux tracing 的相關機制。大概做一下使用記錄。首先是要
加一個自己的 local package，與原生 package 不同的地方，不需要去 official site
抓 tar ball。

1. 首先就是建立程式的資料夾，程式與 Makefile，路徑不限。
2. 接著，在 buildroot 內建立對應的環境，描述如下，
   *  建立package directory: 舉例來說 `package/foo`
   *  加入 config file: `package/foo/Config.in`

      ~~~
      config BR2_PACKAGE_FOO
           bool "foo"
           help
             FOO example program
      ~~~

      並在 `package/Config.in` 加入這個檔案

      ~~~
      "package/foo/Config.in"
      ~~~

   * 加入 mk 檔: `package/foo/foo.mk`
     基本上如 manual 所述，使用 local package 的話變數設定為

     ~~~
         FOO_SITE = /path/to/foo
         FOO_SOURCE = "hello.c Makefile"
         FOO_SITE_METHOD = local
     ~~~

     另外需定義 `FOO_EXTRACT_CMDS`，當作原始碼的安裝步驟

     ~~~
         define FOO_EXTRACT_CMDS
                 cp $(TOPDIR)/$(FOO_SOURCE) $(@D)
         endef
     ~~~

# Reference
這裡有兩份文件個人覺得不錯，一個是官方的 manual，另一個是 Free Electron 的投影
片，

* [Adding package](http://buildroot.uclibc.org/downloads/manual/manual.html#adding-packages)
* [ELC2011: Using Buildroot for real projects](http://elinux.org/images/2/2a/Using-buildroot-real-project.pdf)

最後，當然是感謝 google 與 stackoverflow 了。
