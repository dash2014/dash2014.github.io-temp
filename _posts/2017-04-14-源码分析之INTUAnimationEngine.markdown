---
layout: post
title:  "源码拆解之INTUAnimationEngine"
subtitle: ""
date:   2017-04-14 17:34:01
categories: [tech]
---

> INTUAnimationEngine的源码拆解

INTUAnimationEngine

比较老的库，目前拥有1001个Star，github地址：https://github.com/intuit/AnimationEngine

具体拆解一下:
- 提供了一组方法，用于传递生成animation所需要的不同参数，比如duration，delay， easing，options等，具体可参照INTUAnimationEngine这个类

- 同时提供了两个block， animations，以及completion，其中animations的参数为progress，使用者可以通过这个progress来实时更改所需要的动画， completion用于完成动画的回调。

- 在animations这个Block中，当修改某些组件的动画样式的时候，其在INTUInterpolationFunctions中提供了一些c方法来用于计算某些动画形式下，各组件应该显示的值。

- 在INTUEasingFunction中，其也提供了一些c方法，用于封装不同的动画样式曲线。

- INTUSpringAnimation继承自INTUAnimation，其主要与运动速度等有关，因此作者封装了一个Context，专门用于计算不同位置下的速度，以及位置信息等信息，并不断随着动画而改变其值，同时设定了一些指定的阈值，当速度阈值与加速度同时小于指定范围，则将动画移除。

- 采用CADisplayLink来驱动动画，与系统帧保持一致。

整体来看这个库封装了比较多的动画计算公式，同时采用Block的形式使得编码比较简洁，同时提供了方法组使用户更便捷的使用，另外，其针对每一个动画生成一个animationID，同时将其返回，这样用户可以在任意时刻手动取消某个animation，而不影响其他动画进行。

**任何技术有其优点必有其局限，保持观望，保持学习**

