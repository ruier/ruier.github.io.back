---
title: compile binder for linux
date: 2012-02-08 18:03:34
categories: Android
tags:
  - Linux
  - Android
---

# binder
binder 是 Android 通信的基础驱动，可以直接集成在 Linux 内核调试。以下是 makefile
```
  1 ifneq ($(KERNELRELEASE),)
  2     obj-m := binder.o
  3 else
  4 KDIR := /lib/modules/$(shell uname -r)/build
  5 PWD := $(shell pwd)
  6 default:
  7     $(MAKE) -C $(KDIR) M=$(PWD) modules
  8 endif
```