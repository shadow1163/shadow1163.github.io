---
title: openwrt on vmware workstation & ESXi
date: 2018-07-12T15:22:40+08:00
tags:
---
最近需要一个搭一个DHCP server，所有的主机都放在了一台很老的DELL服务器上搭建的ESXi主机上，资源相对很少，想到之前用过的openwrt比较适合这种场景。我的ESXi的版本是6.5，应该是向前兼容的。

首先在官网上下载img文件，然后解压缩，转换为VMDK格式的文件。
```
gunzip openwrt-x86-generic-combined-ext4.img.gz
qemu-img convert -f raw -O vmdk openwrt-x86-generic-combined-ext4.img openwrt-x86-generic-combined-ext4.vmdk
```
如果没有qemu-img，记得先安装。

接下来就是把vmdk文件上传到ESXi服务器的磁盘里，最好是创建一个单独的文件夹来存放。
然后创建一个32bit的主机，设置好处理器个数和内存，然后删除自动创建的磁盘，添加一个已存在的磁盘，文件设置为你上传的文件。磁盘的类型最好设置为IDE，网卡的类型为E1000。设置好之后，启动主机就可以看到openwrt可以正常运行了。

Refer to: https://wiki.openwrt.org/doc/howto/vmware
