---
title: Jenkins 开启报错
date: 2018-01-16T14:17:50+08:00
tags: [Jenkins]
---
升级ubuntu17.10之后，打开jenkins之后就报了这个错误。查了一下资料是因为使用了java9的原因，降级到java8就正常了。
```
Stack trace
javax.servlet.ServletException: java.lang.AssertionError: InstanceIdentity is missing its singleton
at org.kohsuke.stapler.Stapler.tryInvoke(Stapler.java:796)
at org.kohsuke.stapler.Stapler.invoke(Stapler.java:876)
at org.kohsuke.stapler.Stapler.invoke(Stapler.java:649)
at org.kohsuke.stapler.Stapler.service(Stapler.java:238)
at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:291)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)
at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:239)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)
at hudson.util.PluginServletFilter$1.doFilter(PluginServletFilter.java:135)
at hudson.plugins.greenballs.GreenBallFilter.doFilter(GreenBallFilter.java:59)
at hudson.util.PluginServletFilter$1.doFilter(PluginServletFilter.java:132)
at hudson.util.PluginServletFilter.doFilter(PluginServletFilter.java:126)
```

参考链接：
https://issues.jenkins-ci.org/browse/JENKINS-38057
https://groups.google.com/forum/#!topic/jenkinsci-users/xFFnIB2uK_8
