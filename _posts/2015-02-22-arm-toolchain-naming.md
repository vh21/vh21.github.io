---
layout: post
title: 'arm toolchain naming'
date: 2015-02-22 22:57
comments: true
tags: [GCC, ARM, floating point]
category: toolchain
---

之前雖然有看過但常常忘記，還是整理一下好了，主要是整理[1]的文章。

# ABI
目前使用中的主要是EABI[3]，E表示Embedded。與OABI差異主要是支援floating point與對於syscall呼叫的最佳化。

# gcc options and naming
根據某簡報術說要先講結論，

## floating point options

* -mfloat-abi
  選擇使用的ABI，有以下三個選擇：
  * soft: Full software floating point
  * softfp: 使用FPU，但與soft float相容。
  * hard: Full hardware floating point
* -mfpu
  選擇VFP的版本[2, 4]，所以當-mfloat-abi=soft時無用。

## Name of Port

| Name  | endian  | ABI    | Supported CPUs    | VFP | Status    |
| ----- | -------: | ------: | :-----------------: | :---- | :-------- |
| arm   | little  | OABI   |                   |  | Used in version before Debian Lenny |
| armeb | big     | OABI   |                   | | unofficial, inactive and dead |
| armel | little  | EABI   | armv4t, armv5, armv6, armv7 | no | actively maintained |
| armhf | little  | EABI   | armv7+vfp             | yes | flavored VFP+NEON |

## Cross Compile Naming

| Name  | Descriptions  |
| ----- | :------- |
| arm-linux-gnueabi   | little  |
| arm-linux-gnueabihf | support both the HF and SF calling conventions |

# VFP版本
擇日再整理。

# Referece
1. [Debian ARM Hard Float Port](https://wiki.debian.org/ArmHardFloatPort)
2. [Debian ARM Hart Float Port VFP Comparision](https://wiki.debian.org/ArmHardFloatPort/VfpComparison)
3. [Debian ARM EABI Port](https://wiki.debian.org/ArmEabiPort)
4. [ARM Architecture - floating point](http://en.wikipedia.org/wiki/ARM_architecture#Floating-point_.28VFP.29)
