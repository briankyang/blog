---
title: Spring+Velocity+Jetty
date: 2017-08-02 09:57:33
tags: web
---

## java.lang.IlleaglStateException: STREAM at org.eclipse.jetty.server.Response.getWriter
## java.lang.IlleaglStateException: WRITER at org.eclipse.jetty.server.Response.getOutputStream

- 两者同时只能使用一个
- annotation-driven开启,默认使用jackson做消息转换,jackson三个包需要导入
