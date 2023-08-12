---
title: JavaScript手写代码
comments: false
tags: 前端面试
categories: 前端
cover: >-
  https://lyk990-my-blog.oss-cn-shenzhen.aliyuncs.com/front-end/JavaScript-write.jpg
abbrlink: 5b3aed5a
date: 2023-08-06 13:31:08
keywords:
description: 函数的call() / apply() / bind()
top_img:
---

###  1. 函数的call() / apply() / bind()

```js
/* 
自定义函数对象的call方法
*/
function call (fn, obj, ...args) {
  // 如果传入的是null/undefined, this指定为window
  if (obj===null || obj===undefined) {
    // obj = window
    return fn(...args)
  }
  // 给obj添加一个方法: 属性名任意, 属性值必须当前调用call的函数对象
  obj.tempFn = fn
  // 通过obj调用这个方法
  const result = obj.tempFn(...args)
  // 删除新添加的方法
  delete obj.tempFn
  // 返回函数调用的结果
  return result
}

Function.prototype.myCall = function () {
    const args = Array.prototype.slice.call(arguments); // 获取所有的入参 [{ x:10 }, 20, 30]
    const t = args.shift();
    if (t === null || t === undefined) {
      t = window;
      return fn(...args);
    }
    const self = this; // 谁调用this就指向谁
    t.fn = self;
    const res = t.fn(...args);
    delete t.fn;
    return res;
  };
  function fn(a, b) {
    console.log("a", a);
    console.log("b", b);
    console.log("this", this);
    return "hello";
  }
  const res = fn.myCall({ x: 10 }, 20, 30);


/* 
自定义函数对象的apply方法
*/
function apply (fn, obj, args) {
  // 如果传入的是null/undefined, this指定为window
  if (obj===null || obj===undefined) {
    obj = window
  }
  // 给obj添加一个方法: 属性名任意, 属性值必须当前调用call的函数对象
  obj.tempFn = fn
  // 通过obj调用这个方法
  const result = obj.tempFn(...args)
  // 删除新添加的方法
  delete obj.tempFn
  // 返回函数调用的结果
  return result
}

/* 
  自定义函数对象的bind方法
*/
function bind (fn, obj, ...args) {
  return function (...args2) {
    return call(fn, obj, ...args, ...args2)
  }
}
```
### 2. 函数的节流(throttle)与防抖(debounce)

```js
/* 
用于产生节流函数的工具函数
*/
function throttle (callback, delay) {
  // 用于保存处理事件的时间, 初始值为0, 保证第一次会执行
  let start = 0
  // 返回事件监听函数 ==> 每次事件发生都会执行
  return function (event) {
    // 发生事件的当前时间
    const current = Date.now()
    // 与上一次处理事件的时差大于delay的时间
    if (current-start>delay) {
      // 执行处理事件的函数
      callback.call(event.target, event)
      // 保证当前时间
      start = current
    }
  }
}

/* 
用于产生防抖函数的工具函数
*/
function debounce (callback, delay) {
  // 返回事件监听函数 ==> 每次事件发生都会执行
  return function (event) {
    // 如果还有未执行的定时器, 清除它
    if (callback.timeoutId) {
      clearTimeout(callback.timeoutId)
    }
    // 启动延时delay的定时器, 并保证定时器id
    callback.timeoutId = setTimeout(() => {
      // 执行处理事件的函数
      callback.call(event.target, event)
      // 删除保存的定时器id
      delete callback.timeoutId
    }, delay);
  }
}
```
### 3. 数组去重(unique)

```js
/*
方法1: 利用forEach()和indexOf()
  双重遍历, 效率差些
*/
function unique1 (array) {
  const arr = []
  array.forEach(item => {
    if (arr.indexOf(item)===-1) { // 内部在遍历判断出来的    includes(item)
      arr.push(item)
    }
  })
  return arr
}

/*
方法2: 利用forEach() + 对象容器
  只需一重遍历, 效率高些
  [1, 3, 5, 3]
*/
function unique2 (array) {    
  const arr = []
  const obj = {}
  array.forEach(item => {
    if (!obj.hasOwnProperty(item)) {// 不用遍历就能判断出是否已经有了
      obj[item] = true
      arr.push(item)
    }
  })
  return arr
}

/*
方法3: Array.form + Set
    编码简洁
*/
function unique3 (array) {
  // return Array.from(new Set(array))
  return [...new Set(array)]
}
```
### 4. 数组扁平化(flatten)

```js
/* 
数组扁平化: 取出嵌套数组(多维)中的所有元素放到一个新数组(一维)中
  如: [1, [3, [2, 4]]]  ==>  [1, 3, 2, 4]
*/

/*
方法一: 递归 + reduce() + concat() + some()
*/
function flatten1 (array) {

  return array.reduce((pre, item) => {
    if (Array.isArray(item) && item.some((cItem => Array.isArray(cItem)))) {
      return pre.concat(flatten1(item))
    } else {
      return pre.concat(item)
    }
  }, [])
}

/*
方法二: ... + some() + concat()
*/
function flatten2 (arr) {
  // 只要arr是一个多维数组(有元素是数组)
  while (arr.some(item => Array.isArray(item))) {
    // 对arr进行降维
    arr = [].concat(...arr)
  }
  return arr
}

/*
方法三: ... + some() + concat()
*/ 
var arr = [
        [1, 2, 2],
        [3, 4, 5, 5],
        [5, 6, 7, 8, 9],
        [11, 12, [12, 12, [13]]],
        10,
      ];
      // 数组扁平化
      // 去重
      // 排序
      // 不能使用箭头函数（this会指向window）
      Array.prototype.flat = function () {
        const result = this.map((item) => {
          if (Array.isArray(item)) {
            return item.flat();
          }
          return [item];
        });
        return [].concat(...result);
      };
      // 去重
      Array.prototype.unique = function () {
        return [...new Set(this)];
      };
      // 排序
      const sortFn = (a, b) => {
        return a - b;
      };
      const a = arr.flat().unique().sort(sortFn);
      console.log(a);

```
### 5. 深拷贝

```js
/* 
深度克隆
1). 大众乞丐版
    问题1: 函数属性会丢失
    问题2: 循环引用会出错
2). 面试基础版本
    解决问题1: 函数属性还没丢失
3). 面试加强版本
    解决问题2: 循环引用正常
4). 面试加强版本2(优化遍历性能)
    数组: while | for | forEach() 优于 for-in | keys()&forEach() 
    对象: for-in 与 keys()&forEach() 差不多

    cloneDeep()
*/

const obj = {
    a: {
       m: 1 
    },
    b: [3, 4],
    fn: function (){},
    d: 'abc'
}
obj.a.c = obj.b
obj.b[2] = obj.a

/* 
1). 大众乞丐版
  问题1: 函数属性会丢失
  问题2: 循环引用会出错
*/
export function deepClone1(target) { // 从后台获取的数据都可以用
  return JSON.parse(JSON.stringify(target))
}

/* 
获取数据的类型字符串名
*/
function getType(data) {
  return Object.prototype.toString.call(data).slice(8, -1)  // -1代表最后一位
    // [object Array]  ===> Array  [object Object] ==> Object
}

/*
2). 面试基础版本
  解决问题1: 函数属性还没丢失
*/
function deepClone2(target) {
  const type = getType(target)

  if (type==='Object' || type==='Array') {
    const cloneTarget = type === 'Array' ? [] : {}
    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        cloneTarget[key] = deepClone2(target[key])
      }
    }
    return cloneTarget
  } else {
    return target
  }
}

/* 
3). 面试加强版本
  解决问题2: 循环引用正常
*/
function deepClone3(target, map = new Map()) {
  const type = getType(target)
  if (type==='Object' || type==='Array') {
     // 从map容器取对应的clone对象
    let cloneTarget = map.get(target)
    // 如果有, 直接返回这个clone对象
    if (cloneTarget) {
      return cloneTarget
    }
    cloneTarget = type==='Array' ? [] : {}
    // 将clone产生的对象保存到map容器
    map.set(target, cloneTarget)
    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        cloneTarget[key] = deepClone3(target[key], map)
      }
    }
    return cloneTarget
  } else {
    return target
  }
}

/* 
4). 面试加强版本2(优化遍历性能)
    数组: while | for | forEach() 优于 for-in | keys()&forEach() 
    对象: for-in 与 keys()&forEach() 差不多
*/
function deepClone4(target, map = new Map()) {
  const type = getType(target)
  if (type==='Object' || type==='Array') {
    // 判断是否拷贝完成
    let cloneTarget = map.get(target)
    if (cloneTarget) {
      return cloneTarget
    }

    if (type==='Array') {
      cloneTarget = []
      map.set(target, cloneTarget)
      target.forEach((item, index) => {
        cloneTarget[index] = deepClone4(item, map)
      })
    } else {
      cloneTarget = {}
      map.set(target, cloneTarget)
      Object.keys(target).forEach(key => {
        cloneTarget[key] = deepClone4(target[key], map)
      })
    }

    return cloneTarget
  } else {
    return target
  }
}
```
### 6. 自定义new和instanceof工具函数

```js
/* 
自定义new工具函数
  语法: newInstance(Fn, ...args)
  功能: 创建Fn构造函数的实例对象
  实现: 创建空对象obj, 调用Fn指定this为obj, 返回obj
*/
function newInstance(Fn, ...args) {
  // 创建一个新的对象
  const obj = {}
  // 执行构造函数
  const result = Fn.apply(obj, args) // 相当于: obj.Fn()
  // 如果构造函数执行的结果是对象, 返回这个对象
  if (result instanceof Object) {
    return result
  }

  // 给obj指定__proto__为Fn的prototype
  obj.__proto__ = Fn.prototype
  // 如果不是, 返回新创建的对象
  return obj
}

function Fn () {
    this.name = 'tom'
    return []
}
new Fn()

/* 
自定义instanceof工具函数: 
  语法: myInstanceOf(obj, Type)
  功能: 判断obj是否是Type类型的实例
  实现: Type的原型对象是否是obj的原型链上的某个对象, 如果是返回true, 否则返回false
*/
function myInstanceOf(obj, Type) {
  // 得到原型对象
  let protoObj = obj.__proto__

  // 只要原型对象存在
  while(protoObj) {
    // 如果原型对象是Type的原型对象, 返回true
    if (protoObj === Type.prototype) {
      return true
    }
    // 指定原型对象的原型对象
    protoObj = protoObj.__proto__
  }

  return false
}
```
### 7. 字符串处理

```js
/* 
1. 字符串倒序: reverseString(str)  生成一个倒序的字符串
2. 字符串是否是回文: palindrome(str) 如果给定的字符串是回文，则返回 true ；否则返回 false
3. 截取字符串: truncate(str, num) 如果字符串的长度超过了num, 截取前面num长度部分, 并以...结束
*/

/* 
1. 字符串倒序: reverseString(str)  生成一个倒序的字符串
*/
function reverseString(str) {  // abc
  // return str.split('').reverse().join('')
  // return [...str].reverse().join('')
  return Array.from(str).reverse().join('')
}

/* 
2. 字符串是否是回文: palindrome(str) 如果给定的字符串是回文，则返回 true ；否则返回 false
    abcba  abccba
*/
function palindrome(str) {
  return str === reverseString(str)
}

/* 
3. 截取字符串: truncate(str, num) 如果字符串的长度超过了num, 截取前面num长度部分, 并以...结束
abcde...
*/
function truncate(str, num) {
  return str.length > num ? str.slice(0, num) + '...' : str
}

abcdd...
```
### 8. 简单排序: 冒泡 / 选择 / 插入

```js
/* 
冒泡排序的方法
*/
function bubbleSort (array) {
  // 1.获取数组的长度
  var length = array.length;

  // 2.反向循环, 因此次数越来越少
  for (var i = length - 1; i >= 0; i--) {
    // 3.根据i的次数, 比较循环到i位置
    for (var j = 0; j < i; j++) {
      // 4.如果j位置比j+1位置的数据大, 那么就交换
      if (array[j] > array[j + 1]) {
        // 交换
        // const temp = array[j+1]
        // array[j+1] = array[j]
        // array[j] = temp
        [array[j + 1], array[j]] = [array[j], array[j + 1]];
      }
    }
  }

  return arr;
}

/* 
选择排序的方法
*/
function selectSort (array) {
  // 1.获取数组的长度
  var length = array.length

  // 2.外层循环: 从0位置开始取出数据, 直到length-2位置
  for (var i = 0; i < length - 1; i++) {
    // 3.内层循环: 从i+1位置开始, 和后面的内容比较
    var min = i
    for (var j = min + 1; j < length; j++) {
      // 4.如果i位置的数据大于j位置的数据, 记录最小的位置
      if (array[min] > array[j]) {
        min = j
      }
    }
    if (min !== i) {
      // 交换
      [array[min], array[i]] = [array[i], array[min]];
    }
  }

  return arr;
}

/* 
插入排序的方法
*/
function insertSort (array) {
  // 1.获取数组的长度
  var length = array.length

  // 2.外层循环: 外层循环是从1位置开始, 依次遍历到最后
  for (var i = 1; i < length; i++) {
    // 3.记录选出的元素, 放在变量temp中
    var j = i
    var temp = array[i]

    // 4.内层循环: 内层循环不确定循环的次数, 最好使用while循环
    while (j > 0 && array[j - 1] > temp) {
      array[j] = array[j - 1]
      j--
    }

    // 5.将选出的j位置, 放入temp元素
    array[j] = temp
  }

  return array
}


products.sort((item1, item2) => {  
    // 返回正数, item2在右边, 返回负数, item1在右边, 返回0为原本顺序
    return item1.price - item2.price
})
```
### 9、继承

```javascript
 //封装一个父类
        function Person(name, age, sex) {
            this.name = name;
            this.age = age;
            this.sex = sex;
        }
        Person.prototype.eat = function () {
            console.log("螺蛳粉");
        }


        //封装一个子类
        function Student(classRoom, score, name, age, sex) {
            this.classRoom = classRoom;
            this.score = score;
            //构造函数继承调用父类，并把父类的this指向自己的this
            Person.call(this, name, age, sex)
        }
        //以下方法缺点：1.eat共享了  2.如果继承多个属性或方法 需要一个个添加
        // Student.prototype.eat = Person.prototype.eat;

        //以下方法的缺点：1.赋值是地址的赋值，子类和父类的原型对象共享了  2.构造器属性也是不对的
        // Student.prototype = Person.prototype;

        //原型对象继承:把父类的实例化对象 当做自己的原型对象
        Student.prototype = new Person;
        //修正构造器属性(添加constructor属性)
        Student.prototype.constructor = Student;
        Student.prototype.study = function () {
            console.log("面向对象");
        }


        var s1 = new Student(405, 100, "张三", 18, "男");
        console.log(s1);
        console.log(s1.eat);
        console.log(s1.study);
        console.log(s1.constructor);
```
### 10、js调度器（并发编程）

````javascript
  class Scheduler {
    constructor(max) {
      // 最大可并发任务数
      this.max = max;
      // 当前并发任务数
      this.count = 0;
      // 阻塞的任务队列
      this.queue = [];
    }

    async add(fn) {
      if (this.count >= this.max) {
        // 若当前正在执行的任务，达到最大容量max
        // 阻塞在此处，等待前面的任务执行完毕后将resolve弹出并执行
        await new Promise((resolve) => this.queue.push(resolve));
      }
      // 当前并发任务数++
      this.count++;
      // 使用await执行此函数
      const res = await fn();
      let a = 0
      a++
      console.log(res, a)
      // 执行完毕，当前并发任务数--
      this.count--;
      // 若队列中有值，将其resolve弹出，并执行
      // 以便阻塞的任务，可以正常执行
      this.queue.length && this.queue.shift()();
      // 返回函数执行的结果
      return res;
    }
    }

    const sleep = (time) =>
    new Promise((resolve) => {
      setTimeout(resolve, time);
    });

    const scheduler = new Scheduler(2);

    const addTask = (time, val) => {
    scheduler.add(() => {
      return sleep(time).then(() => {
        console.log(val);
      });
    });
    };

    addTask(1000, "1");
    addTask(200, "5");
    addTask(500, "2");
    addTask(300, "3");
    addTask(400, "4");
````
### 11、手写async和await

````javascript
// generator函数
// async、await的特征是能暂停执行并能恢复执行
// generator的特征也是如此  
function* gen() {
    yield 1 // yield用来暂停执行
    yield 2 // next（）方法来恢复执行
    yield 3
    return 4
}
const g = gen()
console.log(g.next())	// {value: 1, done: false} value是返回的值，done是generator函数是否已经走完
console.log(g.next())	// {value: 2, done: false}
console.log(g.next())	// {value: 3, done: false}
console.log(g.next())	// {value: 4, done: false}  // 最终的返回值

// 
async function sixuetang() {
        const data1 = await getData(1);
        const data2 = await getData(data1);
        return `success: ${data2}`;
      }

      function* sixuetang() {
        const data1 = yield getData(1);
        const data2 = yield getData(data1);
        return `success: ${data2}`;
      }

      const g = sixuetang();
      const next1 = g.next(); // {value: Promise{<fulfilled>}, done: false}

      next1.value.then((data1) => {
        console.log("data1: ", data1);
        const next2 = g.next(data1);
        next2.value.then((data2) => {
          console.log("data2: ", data2);
          console.log(g.next(data2));
        });
      });
	// 封装一个适用性更强
      function generatorToAsync(generatorFun) {
        return function () {
          return new Promise((resolve, reject) => {
            const g = generatorFun();
            const next1 = g.next();
            next1.value.then((data1) => {
              const next2 = g.next(data1);
              next2.value.then((data2) => {
                resolve(g.next(data2).value);
              });
            });
          });
        };
      }

      function generatorToAsync(generatorFun) {
        return function () {
          const gen = generatorFun.apply(this.arguments);
          return new Promise((resolve, reject) => {
            function step(key, arg) {
              let res;
              try {
                res = gen[key](arg);
              } catch (error) {
                return reject(error);
              }
              const { value, done } = res;
              if (done) {
                return resolve(value);
              } else {
                return Promise.resolve(value).then(
                  (val) => step("next", val),
                  (err) => step("throw", err)
                );
              }
            }
            step("next");
          });
        };
      }

      const asyncFn = generatorToAsync(sixuetang);
      asyncFn().then((res) => console.log(res));
````
### 12、函数柯里化

#### 精简版柯里化

```javascript
const curring = (protocol) => {
  return function (hostname, pathname) {
    return `${protocol}${hostname}${pathname}`;
  };
};
const https = curring("https://");
const uri = https('www.baidu.com', '/picture')
console.log(uri) // https://www.baidu.com/picture

// 2、如何通过数组解耦获取属性值
const list = [
  {mid:'123', per:'123'},
  {mid: 'qqq', per: '2222'}
]
const curring = key => item => item[key]
const mid = curring('mid')
const per = curring('per')
console.log(list.map(mid))    // [ '123', 'qqq' ]
console.log(list.map(per))    // [ '123', '2222' ]
```

#### 柯里化 (隐式调用)

```javascript
function num(...args) {
  console.log(...args) // 1 2 3·
  return args
}

num(1,2,3)

// 此方法无法无限输入参数链式调用
function add(a) {
  // arguments是一个类数组对象，
  // 可通过Array.prototype.slice.call 转换成数组
  // Array.prototype.slice.call(arguments)能将具有length属性的对象转成数组
  let args = Array.prototype.slice.call(arguments);
  let inner = function () {
    args.push(...arguments)
    let sum = args.reduce(function (perv, cur) {
      return prev + cur
    })
    return sum
  }
  return inner
}

console.log(add(1)(2))

// 通过修改toString 隐式调用来达到柯里化的目的，但是输出的是一个函数类型
function add(a) {
  // arguments是一个类数组对象，
  // 可通过Array.prototype.slice.call 转换成数组
  // Array.prototype.slice.call(arguments)能将具有length属性的对象转成数组
  let args = Array.prototype.slice.call(arguments);
  let inner = function () {
    args.push(...arguments)
    return inner
  }
  inner.toString = function () {
    return args.reduce(function (prev, cur) {
      return prev + cur
    })
  }
  return inner
}
```

#### 柯里化

```javascript
const curring = (func) => {
    const args = [];
    return function result(...rest) {
        if(rest.length === 0) {
            return func(...args)
        } else {
            args.push(...rest);
            return result;
        }
    }
}    
```

#### 定制柯里化

```JavaScript
// 求和函数
const sumFn = (...args) => {
  return args.reduce((a, b) => {
    return a + b;
  });
};
// 对参数进行排序
const sortFn = (...args) => {
  return args.sort((a, b) => {
    a - b;
  });
};
const curring = (func) => {
  const args = [];
  return function result(...rest) {
    if (rest.length === 0) {
      return func(...args);
    } else {
      args.push(...rest);
      return result;
    }
  };
};

console.log(curring(sumFn)(1, 2, 3, 4, 5)(6)() )  // 21
console.log(curring(sortFn)(1)(3)(2)(5, 4)() )    //  [1,2,3,4,5]
```
## 参考
- [JS每日一题：如何设计一个任务队列？控制请求最大并发数](https://www.bilibili.com/video/BV1XZ4y1y7BX/?spm_id_from=333.337.search-card.all.click&vd_source=ad69edc9a457e7180dde2d7baf02ad26)
- [JS每日一题：一道编程题来了解函数柯里化的妙用](https://www.bilibili.com/video/BV1yS4y1C786/?spm_id_from=333.337.search-card.all.click&vd_source=ad69edc9a457e7180dde2d7baf02ad26)
- [手写async](https://www.bilibili.com/video/BV13a41187yp/?spm_id_from=333.337.search-card.all.click&vd_source=ad69edc9a457e7180dde2d7baf02ad26)
- [初、中级前端应该要掌握的手写代码实现](https://juejin.cn/post/6844904052237713422)
