---
layout: post
title: "扁平化布局利器ConstraintLayout"
description: "ConstraintLayout约束性布局入门"
category: Android
tags: [ConstraintLayout]
---

今年的google IO，Android团队给我们带来了一个全新的布局ConstraintLayout，Android stadio也升级到了2.2，有了这个布局Android stadio的布局编辑器终于不那么鸡肋了，ConstraintLayout一种新的布局方式，可以看做是RelativeLayout的增强，向下兼容至API level 9（Android 2.3），它的目标是减少布局的层级，同时改善布局性能，还减少了使用RelativeLayout的复杂性。

![](http://ww3.sinaimg.cn/large/801b780ajw1f8a8cdejg3j206801iq2r.jpg)

在使用ConstraintLayout的时候会在布局编辑器上的左上角看到上面四个按键，我根据个人的理解解释下线面四个按键的作用。

第一个 眼睛形状，选择的话可以看到所有控件的相对位置关系。

第二个 磁铁形状，新增的控件是否自动生成该控件的相对位置关系。

第三个 错号形状，取消布局内所有控件的相对位置关系。

第四个 灯泡形状，根据位置自动生成所有控件的相对位置关系，使用的时候我们只需要把控件拖到，需要的的位置就可以自动确定相对位置关系。

使用ConstraintLayout可以极大程度的扁平化布局，Android stadio也支持将现有布局转换成ConstraintLayout，

![](http://ww2.sinaimg.cn/large/801b780ajw1f8m8tsyqdnj20sa0fwn04.jpg)

在编辑器的Component Tree选择布局右键，选择convert to ConstraintLayout就可以转化为ConstraintLayout。但可能会出一些问题，就需要我们自己去修改。在处理一些宽度高度不确定等复杂情况，还需要组合其他布局来使用。




