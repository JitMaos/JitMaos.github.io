---
title: RxJava2『原理』
date: 2019-04-13 08:23:19
tags:
---

####考虑如何使用RxJava

+ 创建Observable

  要创建一个Observable可以通过create()自定义Observable行为的方式；或者通过操作符(比如：from、just)将现有的数据对象转换成一个Observable。

+ 使用操作符转换Observable

  RxJava允许你通过串联操作符的方式来转换Observable或者合并多个Observable

