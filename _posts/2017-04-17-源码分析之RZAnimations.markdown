---
layout: post
title:  "源码拆解之RZTransitions"
subtitle: ""
date:   2017-04-17 18:54:01
categories: [tech]
---

> RZTransitions的源码拆解

##RZTransitions

用作iOS VC切换动画的一个开源库，目前拥有1600多star，github地址为：https://github.com/Raizlabs/RZTransitions

具体拆解一下：
- **RZTransitionsNavigationController**，整个工程的入口NavigationController，没什么额外的代码，个人觉得目录层级放在RZTranstions下面不是很合适，即使RZTranstionsManager作为它的代理。
- **RZTransitionAction**，定义了动画的方式枚举
- **RZUniqueTransition**，完整的定义了一个Transition的描述概念，主要包括，transitionAction，fromViewControllerClass，toViewControllerClass三个基本要素。
- **Interactors**，定义了一组动画，主要用来处理不同手势产生的动画效果。
- **RZTransitionsManager**，核心模块，其实现了UINavigationControllerDelegate,                                      UIViewControllerTransitioningDelegate,                                             UITabBarControllerDelegate三个协议，在其代理方法中定义不同的动画效果。
- **Transitions**， 这个目录中的文件主要实现了UIViewControllerAnimatedTransitioning的方法，用来完成整个动画。


整体来看这个开源库，其封装了独立的Transition描述类，以及单独抽象了不同的手势触发的动画，以及不同的动画实现方式，整体结构可以参考~


**任何技术有其优点必有其局限，保持观望，保持学习**

