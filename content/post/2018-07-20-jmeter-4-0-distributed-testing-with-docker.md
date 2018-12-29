---
title: jmeter 4.0 distributed testing with docker
date: 2018-07-20T15:13:04+08:00
tags: [jmeter]
---
Jmeter支持分布式测试，所以打算用docker来部署Master和Slave测试一下。在Master/Client下映射一个控制端口，在Slave/Server下映射两个端口。
- client.rmi.localport=60000
- Server_port=1099
- server.rmi.localport=50000

## Jmeter base dockerfile
首先构建一个Jmeter的基础镜像。Master和Slave基于基础镜像再构建。基础镜像的Dockerfile如下
```
FROM ubuntu:18.04

MAINTAINER shadow1163 (674602286@qq.com)

RUN apt-get update && apt-get install -y curl openjdk-8-jdk openjfx

ARG JMETER_VERSION=4.0

RUN curl http://mirror.bit.edu.cn/apache//jmeter/binaries/apache-jmeter-$JMETER_VERSION.tgz -o /opt/jmeter.tgz && cd /opt && tar zxf jmeter.tgz && rm -f /opt/jmeter.tgz

ENV JMETER_HOME /opt/apache-jmeter-$JMETER_VERSION/

ENV PATH $JMETER_HOME/bin:$PATH
```
## Jmeter Master dockerfile
基于基础镜像构造Master
```
FROM jmeter-wgbase
MAINTAINER shadow1163 (674602286@qq.com)
# Ports to be exposed from the container for JMeter Master
EXPOSE 60000
```
## Jmeter Slave dockerfile
基于基础镜像构造Slave
```
FROM jmeter-wgbase
MAINTAINER shadow1163 (674602286@qq.com)
# Ports to be exposed from the container for JMeter Master
EXPOSE 1099 50000
ENTRYPOINT $JMETER_HOME/bin/jmeter-server -Dserver.rmi.localport=50000 -Dserver_port=1099 -Jserver.rmi.ssl.disable=true
```
## Testing
在Jmeter 4.0里，client和server的连接默认开启了RMI keystore. 有两个解决办法
- 关闭RMI keystore
- 生成一个RMI keystore 文件，并应用于Jmeter

### Disable RMI Keystore
在启动的命令行加上"-Jserver.rmi.ssl.disable=true"即可。可以看到Slave的dockerfile里，就是采用的这种方式, Master上也需要加上同样的参数。

### Secure our channel
到Jmeter的执行目录下，执行create-rmi-keystore命令，生成rmi_keystore.jks文件。在启动Container时，把这个文件添加进去。
- Slave/server端添加参数-Jserver.rmi.ssl.keystore.file=${jmeter_path}/rmi_keystore.jks
- Master/client端添加参数-Jserver.rmi.ssl.keystore.file=${jmeter_path}/rmi_keystore.jks
