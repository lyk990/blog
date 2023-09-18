---
title: 面试官：说一下vue3响应式
comments: false
tags: 前端面试
categories: 前端
cover: >-
  https://lyk990-my-blog.oss-cn-shenzhen.aliyuncs.com/front-end/VO2AGjpS9gOJN1RY1vzq--1--sxjwk.jpg
abbrlink: eb0b5d9f
date: 2023-09-17 21:21:31
keywords:
description:
top_img:
---

## 第一层回答

> 按照源码实现来回答

-   proxy、reflect

创建proxy代理对象，监听用户的get、set操作，再通过reflect反射给代理对象

当get时会track（收集依赖），当set时会trigger（触发依赖）

````js
export function reactive(raw) {
    return new Proxy(raw, {
        get(target, key) {
            const res = Reflect.get(get, key)
            // 执行收集依赖操作
            track(target, key);
            return res
        },
        set(target, key, value) {
            const res = Reflect.set(target, key, value)
            // 执行触发依赖操作
            trigger(target, key);
            retrun res
        }
    })
}
````

-   effect依赖收集、触发依赖

收集响应式对象的fn，创建effect，执行fn，触发get操作，执行track方法，把effect收集起来作为依赖

````js
// 依赖收集
const targetMap = new WeakMap<any, KeyToDepMap>()
export class ReactiveEffect<T = any> {
  active = true
  deps: Dep[] = [] 
  onStop?: () => void

  ...
  
   constructor(
    public fn: () => T,
    public scheduler: EffectScheduler | null = null,
    scope?: EffectScope
  ) {
    recordEffectScope(this, scope)
  }
  
  run() {
    // 在源码中,这里面还有很多操作,这里为了精简
    // 就简单写一下
    ...
    return this.fn()
    
  }

  stop() {
      if (this.onStop) {
        this.onStop()
  }
}


// 依赖收集，传入的fn会触发get操作，同时收集fn
// 对响应式对象进行set操作时，会触发所收集的fn
export function effect(fn, options) {
  const _effect = new ReactiveEffect(fn)
  extend(_effect, options)
   _effect.run()
  const runner = _effect.run.bind(_effect) 
  runner.effect = _effect
  return runner
}
````

-   track 在触发get的时候进行依赖收集

````js
const targetMap = new Map();
export function track(target, type, key: unknown) {
  if (shouldTrack && activeEffect) {
    let depsMap = targetMap.get(target)
    if (!depsMap) {
      targetMap.set(target, (depsMap = new Map()))
    }
    let dep = depsMap.get(key)
    if (!dep) {
      depsMap.set(key, (dep = createDep()))
    }

    const eventInfo = __DEV__
      ? { effect: activeEffect, target, type, key }
      : undefined

    trackEffects(dep, eventInfo)
  }
}
````

-   trigger在触发set的时候进行触发依赖

先取出来之前存起来的depsMap，遍历，然后effect.run()执行依赖

修改响应式对象的值的、触发set操作、执行trigger、重新运行effect函数，执行fn、触发get操作，执行track方法，把effect收集起来作为依赖

````js
// 触发set的时候会触发依赖
export function trigger(target, key) {
    let depsMap = targetMap.get(target)
    let dep = depsMap.get(key)
    // 遍历之前收集到的所有的fn，然后调用它
    for(const effect of dep) {
        effect.run( )
    }
}
````


## 第二层回答

> 结合架构及设计思想回答

如果从这个角度来回答问题,就需要了解一下vue3的设计思想

 **响应式数据驱动**：将数据与 DOM 元素绑定，一旦数据发生变化，相关的 DOM 元素会自动更新，无需手动操作 DOM。
 
 **组件化开发**：将应用程序拆分为多个可复用的组件。每个组件都有自己的状态和逻辑，可以嵌套组合以构建复杂的用户界面。组件化开发使代码更易于维护和理解，同时提高了代码的可复用性。
 
 **单文件组件**：单文件组件 (Single File Components，`SFC`) 这种方式将模板、脚本和样式都封装在一个文件中，使开发更加模块化和可维护。
 
 **虚拟 DOM**：在内存中建立一个虚拟的 DOM 树，然后与实际 DOM 进行比较，只更新实际改变的部分，从而减少了 DOM 操作的开销，提高了渲染效率。
 
  **指令系统**：提供了一套强大的指令系统，允许开发人员直接操作 DOM。除了内置的指令，还可以自定义指令以满足特定需求。
  
  **路由管理**：通常与 Vue Router 配合使用，以实现单页面应用 (`SPA`) 的路由管理。Vue Router 允许你定义路由和视图之间的映射关系，实现无刷新页面切换。
  
  **状态管理**：通常与 Vuex 配合使用，以管理应用程序的状态。Vuex 提供了一个集中的状态存储，用于跨组件共享状态和管理复杂的应用程序状态逻辑。
  
  **插件系统**：支持插件系统，这使得社区可以为 Vue 提供丰富的插件和扩展，以满足不同的需求。
  
  **渐进式框架**：用你想用或者能用的功能特性，你不想用的部分功能可以先不用

Vue3的核心设计思想之一就是响应式, 它采用的是MVVM架构,

在`视图层`(View)和`数据层`(Model)之间建立一个`视图层`(ViewModel),通过观察者模式和采用代理反射的方式,进行数据劫持和数据绑定,实现响应式系统,我们在开发时只需要关心数据就行,vue能够实现自动的数据更新和 DOM 渲染

响应式系统将数据与视图绑定起来。这对于单页面应用非常重要，`单页面应用`（SPA）是一种特殊类型的 Web 应用程序，其特点是在用户与应用程序交互时不会重新加载整个页面。相反，SPA 通过动态地替换页面的一部分来实现页面切换和内容更新

因为它们通常需要在用户与应用程序交互时动态更新内容，而不需要进行完整的页面刷新。响应式系统使得数据变化后视图会自动更新，

Vue 3 的响应式系统为单页面应用提供了一种高效的方式来管理和响应数据的变化，同时，Vue3提供的组件化开发、路由管理、状态管理等工具也使得构建复杂的单页面应用变得更加容易



## 第三层回答

> 为什么要做响应式这样的设计,它的出现解决了什么问题,它存在哪些缺陷?

Vue 3 的核心设计思想之一是数据驱动视图，这意味着可以使用 Vue 3 的响应式系统将数据与视图绑定起来,开发者在开发时只需要关心数据状态和逻辑就行,使用响应式系统的能提供一种简单、高效的方式来管理前端应用程序的数据和界面，这极大的降低了开发门槛

同时,Vue 的响应式系统使用虚拟 DOM 来最小化实际 DOM 操作的次数，从而提高性能。只有在必要时才会进行实际 DOM 更新，减少了开销

但是它还存在一些问题,

- Vue 的响应式系统使用虚拟 DOM 来最小化实际 DOM 操作的次数，但对于大型应用或频繁数据变化的情况，仍然可能导致性能开销较大

- Vue 响应式系统需要为每个响应式数据对象维护一个依赖图，这可能导致一些内存开销，尤其是在具有大量响应式数据的应用中


- 响应式数据和计算属性的管理可能会变得复杂

- Vue 响应式系统在跨组件通信方面的能力有限。

