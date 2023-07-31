---
title: AOP在前端编程中的应用
comments: false
abbrlink: 4e6f4a5d
date: 2023-07-30 22:28:20
tags: 前端架构管理
categories: 前端
keywords:
description:
top_img:
cover: https://lyk990-my-blog.oss-cn-shenzhen.aliyuncs.com/front-end/AOPInFrontEnd.jpg
---

最近在学代码优化的时候，看到了AOP(面向切面编程)，听着好像挺厉害的，就把他记录了下来

## 什么是AOP

面向对象的特点是继承、多态和封装。而封装就要求将功能分散到不同的对象中去，这在软件设计中往往称为职责分配。实际上也就是说，让不同的类设计不同的方法。这样代码就分散到一个个的类中去了。这样做的好处是降低了代码的复杂程度，使类可重用。  

但是人们也发现，在分散代码的同时，也增加了代码的重复性。什么意思呢？比如说，我们在两个类中，可能都需要在每个方法中做日志。按面向对象的设计方法，我们就必须在两个类的方法中都加入日志的内容。也许他们是完全相同的，但就是因为面向对象的设计让类与类之间无法联系，而不能将这些重复的代码统一起来。  

也许有人会说，那好办啊，我们可以将这段代码写在一个独立的类独立的方法里，然后再在这两个类中调用。但是，这样一来，这两个类跟我们上面提到的独立的类就有耦合了，它的改变会影响这两个类。那么，有没有什么办法，能让我们在需要的时候，随意地加入代码呢？**这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是AOP(面向切面的编程)。**  

一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。

这样看来，AOP其实只是OOP的补充而已。OOP从横向上区分出一个个的类来，而AOP则从纵向上向对象中加入特定的代码。有了AOP，OOP变得立体了。如果加上时间维度，AOP使OOP由原来的二维变为三维了，由平面变成立体了。从技术上来说，AOP基本上是通过代理机制实现的。

[上面的这一段话附上链接](https://www.iteye.com/blog/hejiajunsh-1776569)

## 为什么会叫做切面
AOP的主要思想是将应用程序中存在耦合的功能模块进行封装，这些模块通常与核心业务逻辑无关，但是在应用程序的多个地方都需要进行处理。
例如：一个网站，存在以下几个功能
- 登录
- 注册
- 日志记录

登录、注册就是核心业务逻辑，而日志记录则不属于核心逻辑，但是多个地方都需要复用。在AOP中，就可以将日志记录的逻辑定义为一个切面，然后通过配置或代码来指定在哪个地方来应用这个切面。

在面向对象编程中，很多业务逻辑都在混杂在一起，导致代码的可读性和可维护性下降。AOP就是将这些非核心业务逻辑抽离出来，形成独立的模块，然后通过一种**切面**的机制，AOP框架会在编译时或者运行时，将其动态的织入应用程序的核心逻辑中。

## 应用场景

````js
// 出自JavaScript设计模式与开发实践3.2.3（高阶函数实现AOP）
  Function.prototype.before = function (beforeFn) {
        const _self = this;
        return function () {
          beforeFn.apply(this, arguments);
          return _self.apply(this, arguments);
        };
      };

      Function.prototype.after = function (afterFn) {
        const _self = this;
        return function () {
          const ret = _self.apply(this, arguments);
          afterFn.apply(this, arguments);
          return ret;
        };
      };

      var func = function () {
        console.log(2);
      };

      func = func
        .before(function () {
          console.log(1);
        })
        .after(function () {
          console.log(3);
        });

      func();
      
    输出:
      1
      2
      3
 
````
看着好像非常疑惑，这样写的意义何在，这些代码主要是为了帮助理解AOP。其实简单来讲就是，通过函数调用，找到对应的切入点（PointCut），再从切入点中对应的织入（Weaving）动作，这样来实现逻辑解耦。



 # 参考
 - 书籍： JavaScript设计模式与开发实践
 - [简谈前端开发中的AOP(一) -- 前端AOP的实现思路](https://zhuanlan.zhihu.com/p/269504590)
 - [简谈前端开发中的AOP(二) -- 前端AOP的发展与完善](https://zhuanlan.zhihu.com/p/468837058)