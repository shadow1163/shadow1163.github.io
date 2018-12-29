---
title: go mobile 简单示例
date: 2018-01-18T15:18:35+08:00
tags: [Golang, Gomobile]
comments: true
---
Go语言官方提供了一个编写手机APP的框架mobile, 看了看例子觉得还是比较简单的。

## 安装
首先要配置好golang的环境，Android需要golang 1.5以上的版本， iOS的需要1.7.4以上的版本。然后配置好Android NDK，可以用Android Studio配置。iOS同样需要配置好Xcode。
```shell
go get golang.org/x/mobile/cmd/gomobile
gomobile init # it might take a few minutes
```
由于网络的原因，有可能无法直接从官网上获取gomobile，也可以从github上获取。
```shell
go get github.com/golang/mobile
ln -s $GOAPTH/src/github.com/golang/mobile $GOAPTH/srcgolang.org/x/mobile
go get golang.org/x/mobile/cmd/gomobile
gomobile init # it might take a few minutes
```
如果NDK没有安装好，init可能不会报错，但是在build的时候也有提示。也可以指定NDK的目录。
```shell
gomobile init -ndk <your ndk path>
```
## 编译
安装完成后就可以编译APK文件了，很简单的命令。这里编译官方提供的一个示例。
```shell
gomobile build -target=android golang.org/x/mobile/example/basic
```
看下你的目录下有没有basic.apk文件。

以上只是对gomobile很简单的介绍，接下来就要写一些具体的页面来看看这个框架怎么样。

示例参考：https://github.com/golang/go/wiki/Mobile
