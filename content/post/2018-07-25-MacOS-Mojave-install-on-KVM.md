---
title: MacOS Mojave install on KVM
date: 2018-07-25T16:10:35+08:00
tags: [KVM]
---
最近项目上需要在最新的MacOS 10.14上测试, 以前也在VMware ESXi上部署过MacOS 10.12，由于vmware unlocker宣布不在支持ESXi，找到另外一个KVM项目支持MacOS 10.14，那么就在KVM上部署试试吧。

## Install
Clone source
```
git clone https://github.com/kholia/OSX-KVM
```
Install QEMU
```
sudo apt-get install qemu uml-utilities libguestfs-tools
# First edit /etc/apt/sources.list to add/uncomment deb-src lines
sudo apt-get update
sudo apt-get build-dep qemu
git clone https://github.com/kholia/qemu.git
cd qemu
git checkout macOS
git submodule init
git submodule update --recursive
./configure --prefix=/home/$(whoami)/QEMU --target-list=x86_64-softmmu --audio-drv-list=pa
$ make clean; make -j8; make install
```
在编译的时候有可能报错，"error: static declaration of memfd_create follows non-static declaration", 可参考这个补丁。
```
https://git.qemu.org/?p=qemu.git;a=commitdiff;h=75e5b70e6b5dcc4f2219992
--- a/configure
+++ b/configure
@@ -3923,7 +3923,7 @@ fi
 # check if memfd is supported
 memfd=no
 cat > $TMPC << EOF
-#include <sys/memfd.h>
+#include <sys/mman.h>

 int main(void)
 --- a/util/memfd.c
 +++ b/util/memfd.c
 @@ -31,9 +31,7 @@

  #include "qemu/memfd.h"

 -#ifdef CONFIG_MEMFD
 -#include <sys/memfd.h>
 -#elif defined CONFIG_LINUX
 +#if defined CONFIG_LINUX && !defined CONFIG_MEMFD
  #include <sys/syscall.h>
  #include <asm/unistd.h>
```
Create HD disk file
```
qemu-img create -f qcow2 mac_hdd.img 128G
```

## Boot & Install macOS
在项目根目录下有一个启动脚本文件，"boot-macOS-Mojave.sh"，其中有几个参数可能需要修改：
- mac_hdd.img文件需要填写正确的目录
- ISO文件需要修改位对应的目录
- qemu-system-x86_64替换为安装到Home下的目录

修改完成之后支持"boot-macOS-Mojave.sh"即可安装.
