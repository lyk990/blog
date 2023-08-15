---
title: AOP装饰函数在前端中的应用
comments: false
tags: 前端架构管理
categories: 前端
abbrlink: 803cab72
date: 2023-08-13 22:25:16
keywords:
description:
top_img:
cover: https://lyk990-my-blog.oss-cn-shenzhen.aliyuncs.com/front-end/AOP-Decoration.jpg
---

> 在传统的面向对象编程中，给对象添加功能常常采用继承的方式，但是继承的方式并不灵活。在实现一些功能时，有可能会创建出大量子类

## 装饰器

在JavaScript中可以很方便的给某个对象添加属性或者方法，但是很难在不改动源代码的基础上给函数添加新的功能。

````js
 // 假如当前我们的代码是这样的
  const a = () => {
    console.log(1);
  };
  
  // 我想添加一个打印2的功能, 这个时候可以直接修改源代码
  const a = () => {
    console.log(1);
    console.log(2);
  };     
````
但是在实际开发中，我们往往接手的是上一个同事留下的代码，鬼知道他在这一段代码里面做了什么。。。。。这个时候可能就要求我们在不改变源代码的基础上给函数增加新的功能。那么我们就可以用到**装饰器**了



## 装饰器模式

在 JavaScript 中，自身并没有原生支持装饰器模式。装饰器模式是一种结构性设计模式，它允许你通过将对象包装在一个或多个装饰器中，以动态地添加或修改对象的行为。**装饰器（Decorator）模式，是一种在运行期动态给某个对象的实例增加功能的方法。**

````js
// 使用高阶函数实现装饰器模式
function withLogging(fn) {
    return function (...args) {
      console.log(`Calling function with arguments: ${args}`);
      const result = fn(...args);
      console.log(`Function result: ${result}`);
      return result;
    };
  }

  function add(a, b) {
    return a + b;
  }

  const decoratedAdd = withLogging(add);
  console.log(decoratedAdd(2, 3));  // Output: 5
  ````

````js
// 使用类继承实现装饰器模式
class Coffee {
    cost() {
      return 5;
    }
  }

  class MilkDecorator extends Coffee {
    constructor(coffee) {
      super();
      this._coffee = coffee;
    }

    cost() {
      return this._coffee.cost() + 2;
    }
  }

  class SugarDecorator extends Coffee {
    constructor(coffee) {
      super();
      this._coffee = coffee;
    }

    cost() {
      return this._coffee.cost() + 1;
    }
  }

  let coffee = new Coffee();
  coffee = new MilkDecorator(coffee);
  coffee = new SugarDecorator(coffee);

  console.log(coffee.cost()); // Output: 8
````
````js
// 使用类成员实现装饰器模式
  function logProperty(target, key, descriptor) {
    const originalValue = descriptor.value;

    descriptor.value = function (...args) {
      console.log(`Calling method ${key} with arguments: ${args}`);
      const result = originalValue.apply(this, args);
      console.log(`Method result: ${result}`);
      return result;
    };

    return descriptor;
  }

  class MathOperations {
    @logProperty
    add(a, b) {
      return a + b;
    }

    @logProperty
    subtract(a, b) {
      return a - b;
    }
  }

  const math = new MathOperations();
  console.log(math.add(5, 3));
  console.log(math.subtract(10, 4));
````



## AOP装饰函数的应用

回到最初的问题，如何在不影响源代码的基础上，给函数添加新的功能

````js 
// 利用AOP装饰函数来解决这个问题
 const a = () => {
    console.log(1);
  };

  function addLogProperty(targetFunction) {
    targetFunction.additionalLog = () => {
      console.log(2);
    };

    return targetFunction;
  }

  const decoratedA = addLogProperty(a);

  decoratedA(); // Output: 1
  decoratedA.additionalLog(); // Output: 2
    
  ````
  

在编写一个函数时，可利用AOP装饰器增加函数的扩展性，可以动态的去为这个函数添加功能，不用去修改源代码。

用AOP装饰函数的技巧在实际开发中非常有用。不管是业务的编写，还是系统框架的设计，都可以把行为按照职责分成粒度更加细粒的函数，随后通过装饰把他们合并到一起。

## 参考
JavaScript设计模式与开发实践 **[书籍]**

