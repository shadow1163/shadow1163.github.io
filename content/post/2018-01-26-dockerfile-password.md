---
title: ubuntu编译dockerfile如何输入密码
date: 2018-01-26T15:57:16+08:00
tags: [Docker]
---
在Docker上使用Dockerfile构建docker image时，有的安装程序会要求输入密码，这个时候又不能交互输入。在类debian的系统中，最常见的安装软件的方式是使用apt-get，在这种方式下可以尝试使用debconf-set-selections来设置你的密码。以最常见的Mysql举例：
```
export DEBIAN_FRONTEND="noninteractive"

sudo debconf-set-selections <<< "mysql-server mysql-server/root_password password $1"
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $1"
```
这条命令有的版本会起作用，有的版本却无效，原因是参数可能会有变化。可以使用debconf-get-selections检查一下。如果命令找不到，就先安装一下。
```
sudo apt-get install -y debconf-utils
```
