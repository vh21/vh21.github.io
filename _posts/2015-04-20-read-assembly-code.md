---
layout: post
title: '閱讀assembly code'
date: 2015-04-20 09:00
comments: true
tags: [arm, mips, as, gcc]
categories: assembly
---

有朋友問看 assembly 時遇到一些奇怪的符號如`%`或是`=`時，哪裏有文件可以參考這些語法?
後來覺得東西太雜了，乾脆寫一篇心得文好了！

基本上 assembly source 由幾個部分組成，

* instruction set
* assembler syntax
* directive

# Instruction set

基本上對於相同的 CPU 來說是固定的，比較不會隨著 assembler 不同。如

~~~
    MVNNE   r11, #0xF000000B
~~~

每個 CPU 可能會有不同 version 的 [instruction set architecture (ISA)](http://en.wikipedia.org/wiki/List_of_instruction_sets)
如，

* ARMv6
* [ARMv7-A](http://infocenter.arm.com/help/topic/com.arm.doc.qrc0001mc/QRC0001_UAL.pdf)
* [ARMv7-M](https://web.eecs.umich.edu/~prabal/teaching/eecs373-f10/readings/ARMv7-M_ARM.pdf)
* [MIPS32r2](https://ti.tuwien.ac.at/cps/teaching/courses/cavo/files/MIPS32-IS.pdf)

等等，及每一個 version 可能有不同的 extension 如，

* [Thumb2](http://infocenter.arm.com/help/topic/com.arm.doc.qrc0001mc/QRC0001_UAL.pdf)
* [Neon](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0489e/CJAJIIGG.html)
* [MIPS SIMD](http://www.imgtec.com/mips/architectures/simd.asp)

等。請參閱各自的 instrction set manual 或是找 Quick Referece Card 當索引，通常在 CPU 的網頁上都有。
或是詢問 google 大神看看。

# Assembler syntax

每個不同的assembler會有一些自訂的語法，如 symbol 的標記方式、 include header 、如何宣告資料、
註解格式等，請參閱各自的 toolchain 的說明文件。

### gcc and as

如果你的 assembly file 使用 gcc 編譯，gcc 對於一些預設的副檔名有不同的處理流程，如`.S`會先
經過 preprocessing 處理，所以你可以使用如 C 語言的

~~~
#include <foo.h>
~~~

與 macro

~~~
#define ADD3(a,b,c)   add a, b, a; add a, c, a;
~~~

`.s` 預設不會，但可在編譯時加上參數 `-X assembler-with-cpp` 讓它也可以處理這些部分。

GNU 體系語法請參考 [binutils as syntax](https://sourceware.org/binutils/docs/as/Syntax.html#Syntax)

### ARM toolchain

ARM 有自家開發的 toolchain ，從早期的 ADS, RVCT 到現在的 DS-5，可參考各自版本的
[armasm user guide](http://infocenter.arm.com/help/topic/com.arm.doc.dui0473k/DUI0473K_armasm_user_guide.pdf)

ARM 的 [information center](http://infocenter.arm.com/help/index.jsp) 有齊全的tool 與 architecture 的 manual。

# Directives

給 assembler 看的一些特殊語法，如

~~~
 .data
~~~

標記接下來的東西都放到 `.data` section。這部分同時會跟 assembler 與 architecture 有關。

GNU as 通用的部分請參閱 [Assembler Directives](https://sourceware.org/binutils/docs/as/Pseudo-Ops.html#Pseudo-Ops)
章節。ARM 專有的部分請參考 [ARM Machine Directives](https://sourceware.org/binutils/docs/as/ARM-Directives.html)。
MIPS 的部分請參考 [MIPS Dependent Features](https://sourceware.org/binutils/docs/as/MIPS_002dDependent.html#MIPS_002dDependent)。
之前有看到一個文件[MIPS Assembly Language Programmer’s Guide](http://www.cs.unibo.it/~solmi/teaching/arch_2002-2003/AssemblyLanguageProgDoc.pdf)，
雖然很舊但是有很詳細的列表。

# Inline assembly

好吧，如果你要在 C code 裡嵌入 assembly code 或是要查閱一些相關語法可參考
[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)

# Other references

這邊找到一份 GNU ARM 的 [quick start guide](http://www.ic.unicamp.br/~celio/mc404-2014/docs/gnu-arm-directives.pdf)
但是看起來年代可能有點久了， ABI 的部分建議改看
[AAPCS, Procedure Call Standard for the ARM Architecture](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0042e/IHI0042E_aapcs.pdf)。
其他就看 assembler 的 manual吧。

