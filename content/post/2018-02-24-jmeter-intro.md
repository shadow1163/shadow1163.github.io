---
title: Apache Jmeter 入门
date: 2018-02-24T16:26:21+08:00
tags: [jmeter]
---
很早之前就听到过Jmeter，最近项目上需要用到Jmeter，就看了看相关的资料。

## 简介
Apache JMeter 是100%纯JAVA桌面应用程序，被设计为用于测试CS结构的软件。 同时也可以用来测试静态和动态资源的性能，例如：静态文件，Java Servlets，CGI Scripts，Java Object，数据库和FTP服务器等等。JMeter可用于模拟大量负载来测试一台服务器，网络或者对象的健壮性或者分析不同负载下的整体性能.

同时，JMeter可以帮助你对你的应用程序进行回归测试。通过你创建的测试脚本和assertions来验证你的程序返回了所期待的值。为了更高的适应性，JMeter允许你使用常规表达式来创建这些assertions.
jmeter的未来： 随着开发人员借助它的可嵌入的结构，希望看到JMeter的能力也随之快速的被扩大。更进一步的主要发展目标是把JMeter在没有减弱它的负载测试的能力的同时尽可能的做成最有效的回归测试工具.

## 安装
 - 从 http://jmeter.apache.org/ 下载最新版本的JMeter，解压文件到你的工作目录。
 - 安装JAVA，配置好JAVA_HOME
 - JMeter可运行在Windows系统，类Unix的系统。

注意JMeter的目录不能空格。

## 运行
进入JMeter的安装目录下的bin目录，运行jmeter.bat(windows)或jmeter.sh(类Unix)。
打开JMeter的主界面如下：

![JMeter4.0主界面](/images/jmeter-ui.png)

在JMeter4.0里取消了工作台，相关的功能合并到了测试计划里了。
右键单击"Test Plan"弹出菜单选择添加线程组：

![](/images/jmeter-add.png)

然后设置线程组的相关属性。

![](/images/jmeter-thread-group.png)

右键单击线程组，添加http request

![](/images/jmeter-request.png)

## 录制
除了手动添加测试组件，还可以使用脚本录制的方法生成测试脚本。
- 添加线程组
- 添加http request defaults设置默认的URL
- 添加录制控制器
- 添加http代理服务器
- 打开http代理服务器
- 把浏览器的代理服务器设置为之前配置的代理

## 分布式
在使用Jmeter进行性能测试时，如果并发数比较大(比如最近项目需要支持1000并发)，单台电脑的配置(CPU和内存)可能无法支持，这时可以使用Jmeter提供的分布式测试的功能。
#### Slave
在主机上安装同版本的jmeter，然后执行jmeter-server.bat(windows)或jmeter-server.sh(类unix)。启动成功后如下：
{% asset_img slave.png %}

#### Master
找到Jmeter的bin目录下jmeter.properties文件，修改如下配置，IP和Port是slave机的IP以及自定义的端口
```
remote_hosts=127.0.0.1,10.2.2.3，10.1.1.4
```
添加之后就可以选择远程执行了
{% asset_img remote-host.png %}
