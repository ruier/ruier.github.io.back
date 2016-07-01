---
title: two ramdisk in linux
date: 2016-07-01 10:42:05
categories: 
  - linux
tags:
  - linux
  - ramdisk
---

# 使用 ramdisk 启动 Linux 内核

## Buildroot 的 ramdisk 镜像支持
> linux 支持从 ramdisk 启动，比如在 buildroot 中可以配置生成 ramdisk 供内核加载。

```
 BR2_TARGET_ROOTFS_CPIO_UIMAGE:                                                                                              ³  
  ³                                                                                                                             ³  
  ³ Add a U-Boot header to the cpio root filesystem. This allows                                                                ³  
  ³ the initramfs to be loaded with the bootm command in U-Boot.                                                                ³  
  ³                                                                                                                             ³  
  ³ The U-Boot image will be called rootfs.cpio.uboot                                                                           ³  
  ³                                                                                                                             ³  
  ³ Symbol: BR2_TARGET_ROOTFS_CPIO_UIMAGE [=y]                                                                                  ³  
  ³ Type  : boolean                                                                                                             ³  
  ³ Prompt: Create U-Boot image of the root filesystem                                                                          ³  
  ³   Location:                                                                                                                 ³  
  ³     -> Filesystem images                                                                                                    ³  
  ³       -> cpio the root filesystem (for use as an initial RAM filesystem) (BR2_TARGET_ROOTFS_CPIO [=y])                      ³  
  ³   Defined at fs/cpio/Config.in:51                                                                                           ³  
  ³   Depends on: BR2_TARGET_ROOTFS_CPIO [=y]                                                                                   ³  
  ³   Selects: BR2_PACKAGE_HOST_UBOOT_TOOLS [=y]                                                                                ³  
  ³                                                

```
## u-boot 启动 ramdisk 的设置
然后在 u-boot 中读取 ramdisk 加载并运行：

```
=> printenv bootcmd 
bootcmd=fatload mmc 0 0x80800000 zImage;fatload mmc 0 0x83000000 imx7d-warp7-mipi-dsi.dtb;fatload mmc 0 0x88000000 rootfs.cpio.uboot;run initaudio;;bootz 0x80800000 0x88000000 0x83000000
=> 
```

## Linux 内核本身对 ramdisk 的支持

Linux 内核自己也提供了对 ramdisk 的支持，而且无需手动加载，在编译时就已经包含在内核镜像里面。ramdisk 会随着内核加载而加载进入系统：

```
 CONFIG_INITRAMFS_SOURCE:                                                                                                    ³  
  ³                                                                                                                             ³  
  ³ This can be either a single cpio archive with a .cpio suffix or a                                                           ³  
  ³ space-separated list of directories and files for building the                                                              ³  
  ³ initramfs image.  A cpio archive should contain a filesystem archive                                                        ³  
  ³ to be used as an initramfs image.  Directories should contain a                                                             ³  
  ³ filesystem layout to be included in the initramfs image.  Files                                                             ³  
  ³ should contain entries according to the format described by the                                                             ³  
  ³ "usr/gen_init_cpio" program in the kernel tree.                                                                             ³  
  ³                                                                                                                             ³  
  ³ When multiple directories and files are specified then the                                                                  ³  
  ³ initramfs image will be the aggregate of all of them.                                                                       ³  
  ³                                                                                                                             ³  
  ³ See <file:Documentation/early-userspace/README> for more details.                                                           ³  
  ³                                                                                                                             ³  
  ³ If you are not sure, leave it blank.                                                                                        ³  
  ³                                                                                                                             ³  
  ³ Symbol: INITRAMFS_SOURCE [=root]                                                                                            ³  
  ³ Type  : string                                                                                                              ³  
  ³ Prompt: Initramfs source file(s)  
```

# 两个 ramdisk 同时加载

通过上面的描述，我们知道有两种方式加载 ramdisk，而且两者可以同时被加载进入内核文件系统。那么实际情况是两种ramdisk加载后会发生什么？

## Buildroot 的ramdisk
按照以上描述，编译 Buildroot 并生成 ramdisk, buildroot 中的内容：
```
ls ~/workspace/github/buildroot/output/target/
THIS_IS_NOT_YOUR_ROOT_FILESYSTEM  dev  filelist  lib    linuxrc  mnt  proc  run   sys  usr
bin                               etc  init      lib32  media    opt  root  sbin  tmp  var
```

## 内核的 ramdisk
同时在内核中编译ramdisk， 内核文件系统的内容：
```
ls android/root/
charger       file_contexts    init.freescale.rc      init.usb.rc        res              sepolicy          ueventd.freescale.rc
data          fstab.freescale  init.freescale.usb.rc  init.zygote32.rc   sbin             service_contexts  ueventd.rc
default.prop  init             init.rc                proc               seapp_contexts   sys
dev           init.environ.rc  init.trace.rc          property_contexts  selinux_version  system
```

# 结果
系统启动之后：
两个ramdisk会合并在一起，我们能够在根文件系统里面同时访问这两个ramdisk的内容：

```
# cat /proc/version 
Linux version 4.1.15-01642-g46abb4f-dirty (jiangxd@Embest-tech) (gcc version 4.8.4 (Ubuntu/Linaro 4.8.4-2ubuntu1~14.04.1) ) #32 SMP PREEMPT Thu Jun 30 11:21:25 CST 2016
# ls /
bin                    init.trace.rc          sbin
charger                init.usb.rc            seapp_contexts
data                   init.zygote32.rc       selinux_version
default.prop           lib                    sepolicy
dev                    lib32                  service_contexts
etc                    linuxrc                sys
file_contexts          media                  system
filelist               mnt                    tmp
fstab.freescale        opt                    ueventd.freescale.rc
init                   proc                   ueventd.rc
init.environ.rc        property_contexts      usr
init.freescale.rc      res                    var
init.freescale.usb.rc  root
init.rc                run
```

至于为何合并，以及如何合并，暂时没有研究。
