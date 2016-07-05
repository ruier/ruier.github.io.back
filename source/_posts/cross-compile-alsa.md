---
title: 交叉编译 ALSA
date: 2012-12-20 16:35:13
categories: Android
tags: 工作
---

# 交叉编译 ALSA
	问题描述:在需要移植 android 录音问题的时候发现 buildroot 下没有 arecord

解决方法;自己编译,使用交叉编译链

系统已经有 交叉编译工具 : 
ls /usr/local/arm-2007q1/
arm-none-linux-gnueabi/ include/                libexec/
bin/                    lib/                    share/
需要将 bin 加入到 PATH 然后 ./configure 才能找到交叉编译工具
checking for arm-none-linux-gnueabi-strip... arm-none-linux-gnueabi-strip
checking for arm-none-linux-gnueabi-gcc... arm-none-linux-gnueabi-gcc
checking for C compiler default output... a.out
checking whether the C compiler works... yes
checking whether we are cross compiling... yes

否则找不到就使用系统默认的 gcc 工具编译了

---

## 下载
http://www.alsa-project.org/main/index.php/Download
## 交叉编译 alsa-lib
./configure --host=arm-none-linux-gnueabi --prefix=/usr/share/arm-alsa
make & make install
## 交叉编译 alsa-utils
./configure --host=arm-none-linux-gnueabi --with-alsa-inc-prefix=/usr/share/arm-alsa/include --with-alsa-prefix=/usr/share/arm-alsa/lib --without-alsamixer // avoid the ncurses lib 

  使用 
./configure --host=arm-none-linux-gnueabi --with-alsa-inc-prefix=/usr/share/arm-alsa/include --with-alsa-prefix=/usr/share/arm-alsa/lib --with-curses=/usr/lib32/libcurses.so --includedir=/usr/include/ --disable-alsamixer
   ref:http://mailman.alsa-project.org/pipermail/alsa-devel/2007-June/001591.html
make
Copy lib to target board
cp -avr /usr/share/arm-alsa {$rootfs}/usr/share/arm-alsa
## 环境变量
export ALSA_CONFIG_PATH=/usr/share/arm-alsa/share/alsa/alsa.conf
未添加环境变量前出现的错误：
~ # ./aplay 
ALSA lib conf.c:2827:(snd_config_hook_load) cannot access file /usr/share/arm-alsa/share/alsa/cards/aliases.conf
ALSA lib pcm.c:1959:(snd_pcm_open_conf) Invalid type for PCM default definition (id: default, value: cards.pcm.default)
aplay: main:533: audio open error: Invalid argument
~ # ./aplay 
ALSA lib pcm.c:2090:(snd_pcm_open_noupdate) Unknown PCM default
aplay: main:533: audio open error: No such file or directory

## 交叉编译其他程序：
Add included file in program: #include <alsa/asoundlib.h>
~# arm-none-linux-gnueabi-gcc -lasound -L/usr/share/arm-alsa/lib/ -I/usr/share/arm-alsa/include/ -o test my_test.c
