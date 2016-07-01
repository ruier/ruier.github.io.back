---
title: audio and display issue
date: 2012-12-20 16:32:01
categories: Android
tags: 
  - Android 
  - 工作
---


	系统:Android 4.0.4
	硬件:sama5d3

测试发现 NFS 和 SD 镜像,音频播放都正常,但是 UBI 镜像老是出现问题:
The Notes from Eric
​
## UBIFS

    UBI/UBIFS have already appeared in mainline

    We should disable DMA when working with UBI

    UBI/UBIFS do not need OOB area

    Working with UBIFS 

    We should disable DMA when working with UBI ,

UBIFS 的工作对 nand 的 DMA 有着严格的要求,开始没有注意这一点,但是所有的功能都没有出现什么问题,只是体现在音频播放上.
所以 disable DMA of nand 就可以解决这个问题.然而这样的问题非常不容易发现.而且容易走错方向.

## 显示的问题:
没有显示加速的硬件,无法得到很好的用户体验,尽管系统做了一些显示优化得到了一些可观的测试数据,但是这并不能提高什么
在测试发现,影响显示性能的绝对因数来自内核,由于内核的一个等待,导致上层所有优化失效.解决这个等待,才能稍微提高测试数据.
但是用户体验仍然不很乐观.

## 总结这个显示优化的结果:
使用 copybit 完全没有起到任何作用,也许在局部绘图没有回拷操作可以节省一点时间,但是效果微乎其微.
使用 hwcomposer 可以得到大概 50% 的数据优化,用户体验也能有一定的改善.这是确实节省了 SurfaceFlinger 的 compose 时间,尤其在
测试的时候,没有全屏幕的测试数据,造成不断的叠加组合,去掉这部分时间,可以大大优化测试数据.