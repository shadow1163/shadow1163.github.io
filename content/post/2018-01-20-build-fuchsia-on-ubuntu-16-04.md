---
title: build fuchsia on ubuntu 16.04
date: 2018-01-20T18:09:34+08:00
tags: [OS, Fuchsia]
---
一直想尝试一下google的新系统Fuchsia，但是由于网络的问题一直没有下载成功，今天用梯子下载完源码之后，就简单试用了一下。由于这个系统还在开发阶段，安装的步骤很有可能发生变化，只能做一个参考记录。

## 依赖
首先安装依赖包

#### ubuntu
```shell
sudo apt-get install texinfo libglib2.0-dev liblz4-tool autoconf libtool libsdl-dev build-essential golang git build-essential curl unzip
```

#### mac
```
xcode-select --install
```

## 下载
由于网络的问题，下载一直是个大问题，再加上系统处于开发阶段，虽然在github上有相关的代码镜像，但是构建的链接都是在fuchsia.googlesource.com上，所以必须要用梯子才能完成下载。
参考之前的一些资料，我是用以下方法下载源码的
```shell
curl -s https://raw.githubusercontent.com/fuchsia-mirror/jiri/master/scripts/bootstrap_jiri | bash -s fuchsia
cd fuchsia
sudo cp .jiri_root/bin/jiri /usr/local/bin
sudo chmod 755 /usr/local/bin/jiri
jiri import fuchsia https://fuchsia.googlesource.com/manifest
jiri update
```
官网上也有另外的命令，我没有试，有兴趣的可以试试。
```shell
curl -s "https://fuchsia.googlesource.com/scripts/+/master/bootstrap?format=TEXT" | base64 --decode | bash -s <layer>
```
关于layer的参考如下
- Zircon
- Garnet
- Peridot
- Topaz

{% blockquote %}
Zircon is the operating system's foundation: it mediates hardware access, implements essential software abstractions over shared resources, and provides a platform for low-level software development.
For example, Zircon contains the kernel, device manager, most core and first-party device drivers, and low-level system libraries, such as libc and launchpad. Zircon also defines the Fuchsia IDL (FIDL), which is the protocol spoken between processes in the system, as well as backends for C and C++. The backends for other languages will be added by other layers.

Garnet provides device-level system services for software installation, administration, communication with remote systems, and product deployment.
For example, Garnet contains the network, media, and graphics services. Garnet also contains the package management and update system.

Peridot provides the services needed to create a cohesive, customizable, multi-device user experience assembled from modules, stories, agents, entities, and other components.
For example, Peridot contains the device, user, and story runners. Peridot also contains the ledger and resolver, as well as the context and suggestion engines.

Topaz augments system functionality by implementing interfaces defined by underlying layers. Topaz contains four major categories of software: modules, agents, shells, and runners.
For example, modules include the calendar, email, and terminal modules, shells include the base shell and the user shell, agents include the email and chat content providers, and runners include the Web, Dart, and Flutter runners.
{% endblockquote %}

## 编译
当下载完成之后，就可以编译了，可以使用一个新命令叫fx，这个命令跟jiri在同一个目录下，如果找不到命令就添加到path或者用绝对路径执行。
```
fx set x86-64/fx set x86-64 --release
fx full-build
```
用这两个命令就可以完成编译了，需要注意的一点是磁盘空间，请确保可用空间40G以上。

## 启动测试
编译需要一些时间，等到编译完成后就可以启动测试一下了。
```
fx run/fx run -g
```
等待一会就可以看到系统已经启动了。

启动加载网卡
```
fx run -N -u $FUCHSIA_SCRIPTS_DIR/start-dhcp-server.sh
```
测试命令如下
```
/system/test/ledger_unittests
```
