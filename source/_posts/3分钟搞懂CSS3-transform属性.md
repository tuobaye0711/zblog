---
title: 3分钟搞懂CSS3 transform属性
date: 2018-01-17 14:59:34
tags: CSS
picture: transform.png
description: transform是CSS3中非常有用的一个属性，今天我们就来挑战一下3分钟能否搞懂transform的基本用途吧~
---

欢迎光临我的博客[拓跋的前端客栈](http://tuobaye.com)，这个是[原文地址](http://tuobaye.com/2018/01/17/3%E5%88%86%E9%92%9F%E6%90%9E%E6%87%82CSS3-transform%E5%B1%9E%E6%80%A7/)。如果您发现我文章中存在错误，请尽情向我吐槽，大家一起学习一起进步φ(>ω<*)

***

> CSS transform 属性允许你修改CSS视觉格式模型的坐标空间。使用它，元素可以被转换（translate）、旋转（rotate）、缩放（scale）、倾斜（skew）。

首先有个直观的印象，接下来放大招了~

![transform转换](transform转换.png)

怎么样，是不是很直观？如果图片不清晰，在图片上点击右键-“在新标签页中打开图片”，然后在图片上点放大镜就可以查看原图了。

接下来有更直观的，我手写了一个简单的transform体验器，自己尝试一下吧：

[transform体验器](http://tuobaye.com/demo/transform)

不知道你搞懂了吗？

PS:

- transform是一个理解上很简单，使用起来却很有难度的属性。短时间内能理解80%就不错了，想完全掌握还需要多多练习。
- 本例中为了尽可能缩短篇幅，没有涉及到transform中matrix部分和rotate3d的部分，想要继续深入的话可以自行查找资料学习。
- translateZ()是一个很特别的属性，需要结合perspective(透视)来理解。perspective可以简单理解为视点与屏幕之间的距离。当未设置perspective属性时，translateZ()的变化不会有任何影响。而设置perspective以后，translateZ()就符合“近大远小”的规律，Z轴上离你越近，图形显示越大。直到大过perspective值的时候，你就看不见了，这时候你就可以理解为元素到你“脑后”去了，人理所当然的看不见脑后的东西。
- skew比较难搞懂，直接在图里用文字很难描述清楚。这里引用[知乎-css3中-webkit-transform 的 skew 如何使用？](https://www.zhihu.com/question/21725826)中Kori Lee的回复比较容易理解

![skew](skew.png)