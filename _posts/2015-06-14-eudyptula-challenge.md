---
layout: post
title: 'Eudyptula Challenge 心得'
date: 2015-06-14 10:00
comments: true
tags: [linux, kernel, eudyptula challenge]
categories: linux
---

~~~
The Eudyptula Challenge is a series of programming exercises for the Linux
kernel, that start from a very basic "Hello world" kernel module, moving on up
in complexity to getting patches accepted into the main Linux kernel source
tree.
~~~

最近看到有人重貼[Eudyptula Challenge](http://eudyptula-challenge.org/)，碰
Linux Kernel 也一陣子了。想說來試看看好了。興沖沖連到首頁後，

~~~
Email little at eudyptula-challenge.org and say that you want to join in. ....
~~~

看到這裡二話不說立刻就寄信了。結果完全忘了信件的格式必須是 plain text。果然被
打槍了，lol

~~~
Really?  You send me HTML email after being told explicitly not to?

What type of script do you think I am?

Please fix your email client and try again.
~~~

更正後就取得第一個作業，Hellod World module。這次謹慎多了，雖然是一個簡單的
module 但還是好好看看文件再開始。 kernel module 相關的編譯環境可參考 kernel
source 裡的 `Documentation/` 裡的 `kbuild/modules.txt` 與
`kbuild/makefiles.txt`。

寫 moudle 還算好，但是之後寄出就麻煩了，除了不接受 HTML format 外，也不接受
base64 的附件檔。之前送 patch 時用過 `git send-email`。但是作業的內容也不是
patch，又不推薦使用 Gmail client，所以重新設定了 postfix 來用，找了老半天終於搞
定了 mail client 的問題。事實上 kernel Documentation 裡也有相關的文件，可參考
`SubmittingPatches` 與 `email-clients.txt` 兩份文件。如果你跟我一樣有興趣試試
postfix 與 mailx，可參考 [Configure Postfix to Use Gmail SMTP on Ubuntu](https://rtcamp.com/tutorials/linux/ubuntu-postfix-gmail-smtp/)
設定你的 postfix， attachment 目前透過mailx 的 tilder `~r` 來inline text（
還不知道行不行）。

Eudyptula Challenge 基本上不會給太多提示，你可以試著去找需要的資訊，但是不歡
迎公開或複製解法，也不歡迎公開求助。整個過程中收穫最多的就是把 kernel
documentation 真的好好的看一遍（怕再被打槍）。不像之前趕時間時總是希望透過
google 快速找解法。

最後，我很喜歡 Little Penguin 的 mail 裡的話，

~~~
Fourthly, this challenge is meant to be done over a period of time.
Don't rush things, there's no hurry, I'm not going anywhere, and neither
are you.  So sit back, take your time, and enjoy the process.  Life is
about the journey, not the end result, as that's the same for everyone.
~~~
