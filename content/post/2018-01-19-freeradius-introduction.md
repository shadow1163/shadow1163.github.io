---
title: Freeradius 入门简介
date: 2018-01-19T15:08:50+08:00
tags: [Radius]
---
最近项目上需要用到Radius认证，就找了点资料看看。这里有一本翻译的书可以看一下，很遗憾作者只翻译了4章，可以理解一下相关的概念。
https://freeradius.akagi201.org/ch00-intro/index.html
英文好的同学可以看看原著。
http://yustanto.com/freeradius.pdf

## 安装
我的系统是ubuntu16.04的，使用apt安装的freeradius的版本是2.2.8,在我写这篇文章的时候官网上最新的relase版本是3.0.16。2.X的版本处于维护状态。需要安装最新版本的只能下载源码编译安装。另外需要注意的是openssl的版本，通过apt安装的版本是1.0.2g，最新版需要安装1.0.2h或者更新的版本。openssl同样也需要下载最新源码编译安装。freeradius 3.X不支持最新的openssl 1.1.0,如果要使用openssl 1.1.0，需要在github上下载freeradius 4.X的源码编译安装。
```shell
wget https://www.openssl.org/source/openssl-1.0.2n.tar.gz
tar zxf openssl-1.0.2n.tar.gz
cd openssl-1.0.2n
./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib/openssl-1.0 shared zlib-dynamic
make depend
make && make install
```
安装完成后检查一下版本
```
openssl version
```
接下来是安装freeradius
```
wget ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-3.0.16.tar.gz
tar zxf freeradius-server-3.0.16.tar.gz
cd freeradius-server-3.0.16
./configure --with-openssl-lib-dir=/usr/lib/openssl-1.0
make && make install
```
安装之后测试一下是否正常
```shell
radiusd –X
最后出现: Ready to process requests
就表明安装成功了.
```

## 测试工具
#### linux下的测试工具
```shell
apt-get install freeradius-utils-y
radtest steve testing localhost 1812 testing123
```

#### windows下的测试工具
windows下可以用RADIUS test client。

https://github.com/Xu-Jian/Radius/blob/master/1.Freeradius%20%E6%9C%8D%E5%8A%A1%E7%AB%AF%2B%E6%B5%8B%E8%AF%95%E5%B7%A5%E5%85%B7.md
