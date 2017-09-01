---
layout: post
title: "Instance Run的提升到底怎么样"
description: "使用Instance Run"
category: Android
tags: [aop]
---

Instance Run是Android stadio 2.0新增的运行方式，目标是减少构建和部署app的时间。

先看一下构建和部署的时间到底能减少多少吧，以我们的项目为例：
不使用Instance Run的效果 三次（不是首次）

Information:Total time: 47.133 secs
Information:Total time: 36.736 secs
Information:Total time: 1 mins 1.499 secs

使用Instance Run的三次（不是首次）

Information:Total time: 22.63 secs
Information:Total time: 17.762 secs
Information:Total time: 18.382 secs

构建和部署的时间有了明显的提升。但是在第一次构建的时候，开启Inatance Run 甚至要慢。

如何开启Instance Run

![](http://ww4.sinaimg.cn/large/801b780agw1f8o6j5339xj21kw113jx7.jpg)

在InstanceRun 中开启 Enable Instance Run。。。。。

Instance Run默认是开启的。Instance Run要求minSdkVersion最低是15.并且要求Gradle plugin 如果版本过低将会提示如下。这时如果要开启Instance Run就需要Update Project

![](http://ww3.sinaimg.cn/large/801b780agw1f8o6qi97a5j217i08kgn8.jpg)

开启了Inastance Run app不在运行时Run的图标如下

![](http://ww3.sinaimg.cn/large/801b780agw1f8oapqlmudj20bq01qmx3.jpg)

当app运行是Run的图标变为

![](http://ww3.sinaimg.cn/large/801b780ajw1f8ob2j64tsj20d201kjrc.jpg)

如果不想通过Instance run可是使用Run中的 Run... 来运行

