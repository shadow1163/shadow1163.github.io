---
title: "access libvirt vms via telnet"
date: 2020-04-30
description: "access libvirt vms via telnet"
tags: [kvm]
featured_image: ""
categories: kvm
comment : true
---
2020年4月23日，ubuntu 20.04 LTS正式发布，我的桌面机也是第一时间做了升级，升级过程中一切正常，日常使用也没有什么变化。但是另外一台ubuntu server 18.04升级之后就没那么好运了，升级之后由于无法找到LVM卷不能启动，只得用ubuntu server 20.04重装了，带来的问题是部署在这台server的KVM虚拟机没了。在配置重装的KVM主机时，遇到了一个问题是virt-manager最新版2.2.1配置serial串口时，没有了network选项。查询了virt-manager的升级日志，也没有发现这个功能被取消了。但是这个选项在ubuntu 18.04上的virt-manager 1.5.1上的确是可以正常配置的。在网络上搜索发现了可以通过XML来设置。代码片段如下：
```
<devices>
  <serial type="tcp">
    <source mode="connect" host="0.0.0.0" service="4555"/>
    <protocol type="telnet"/>
    <target port="0"/>
  </serial>
</devices>
```

在virt-manager界面，进入"edit"->"Prefenences"， 在general选项卡上点击"Enable XML editing"。然后在新创建的serial端口上进入XML配置，复制粘贴上面的配置片段，修改相应的端口号，重启KVM虚拟机后应该可以用telnet访问KVM的串口了。 

Link: https://lukas.zapletalovi.com/2018/02/accessing-libvirt-vms-via-telnet.html