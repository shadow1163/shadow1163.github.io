---
title: ngrok intro
date: 2018-07-20T16:07:16+08:00
tags: [tools, ngrok]
---
最近做Jenkins和Github的集成，计划是收到一个代码的Push就执行一次测试。但是同时遇到一个问题，由于Jenkins没有设置公有IP，导致了Github上设置的webhook无法跟Jenkins通信。在无法设置公有IP的情况下，通过一个小工具来解决这个问题。这个工具叫ngrok。官网：https://ngrok.com/ 。
## 下载登录
进入官网后，注册或者通过社交账号登录都可以，下载ngrok，解压后执行：
```
./ngrok authtoken <your Auth code>
```
## 开启
在连接之后，就可以开始使用了。
```
./ngrok help
./ngrok http 80
```
执行第二条命令会看到如下格式的记录
```
Forwarding                    http://xxxxxxxx.ngrok.io -> 127.0.0.1:80
```
这个网址就可以放在github的webhook中，通过这个网址就可以和本地80端口通信了。

## Automation
```
ngrok -log=stdout 80 > /dev/null &
curl localhost:4040/api/tunnels
```
这两条命令是后台启动ngrok, 使用API获取转换域名。
