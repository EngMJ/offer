## 数据类型

以下是比较重要的几个 js 变量要掌握的点。

#### 1.1 基本的数据类型介绍，及值类型和引用类型的理解

在 JS 中共有 `8`  种基础的数据类型，分别为： `Undefined` 、 `Null` 、 `Boolean` 、 `Number` 、 `String` 、 `Object` 、 `Symbol` 、 `BigInt` 。

其中 `Symbol`  和 `BigInt`  是 ES6 新增的数据类型，可能会被单独问：

+   Symbol 代表独一无二的值，最大的用法是用来定义对象的唯一属性名。
+   BigInt 可以表示任意大小的整数,不可处理小数否则报错。

**值类型的赋值变动过程如下：**

```javascript
let a = 100;
let b = a;
a = 200;
console.log(b); // 100
```

![图片 1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55df6cb63d3346be9ec1f572a1514853~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 值类型是直接存储在\*\*栈（stack）\*\*中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；

**引用类型的赋值变动过程如下：**

```javascript
let a = { age: 20 };
let b = a;
b.age = 30;
console.log(a.age); // 30
```

![图片 2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56c5c43d1c584ed4b8e4cce8855bab52~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 引用类型存储在\*\*堆（heap）\*\*中的对象，占据空间大、大小不固定。如果存储在栈中，将会影响程序运行的性能；

#### 1.2 数据类型的判断

+   **typeof**：能判断所有**值类型，函数**。不可对 **null、对象、数组**进行精确判断，因为都返回 `object` 。

```javascript
console.log(typeof undefined); // undefined
console.log(typeof 2); // number
console.log(typeof true); // boolean
console.log(typeof "str"); // string
console.log(typeof Symbol("foo")); // symbol
console.log(typeof 2172141653n); // bigint
console.log(typeof function () {}); // function
// 不能判别
console.log(typeof []); // object
console.log(typeof {}); // object
console.log(typeof null); // object
```

+   **instanceof**：能判断**对象**类型，不能判断基本数据类型，**其内部运行机制是判断在其原型链中能否找到该类型的原型**。比如考虑以下代码：

```javascript
class People {}
class Student extends People {}

const vortesnail = new Student();

console.log(vortesnail instanceof People); // true
console.log(vortesnail instanceof Student); // true
```

其实现就是顺着**原型链**去找，如果能找到对应的 `Xxxxx.prototype`  即为 `true` 。比如这里的 `vortesnail`  作为实例，顺着原型链能找到 `Student.prototype`  及 `People.prototype` ，所以都为 `true` 。

+   **Object.prototype.toString.call()**：所有原始数据类型都是能判断的，还有 **Error 对象，Date 对象**等。

```javascript
Object.prototype.toString.call(2); // "[object Number]"
Object.prototype.toString.call(""); // "[object String]"
Object.prototype.toString.call(true); // "[object Boolean]"
Object.prototype.toString.call(undefined); // "[object Undefined]"
Object.prototype.toString.call(null); // "[object Null]"
Object.prototype.toString.call(Math); // "[object Math]"
Object.prototype.toString.call({}); // "[object Object]"
Object.prototype.toString.call([]); // "[object Array]"
Object.prototype.toString.call(function () {}); // "[object Function]"
```

在面试中有一个经常被问的问题就是：如何判断变量是否为数组？

```javascript
Array.isArray(arr); // true
arr.__proto__ === Array.prototype; // true
arr instanceof Array; // true
Object.prototype.toString.call(arr); // "[object Array]"
```

#### 1.3 根据 0.1+0.2 ! == 0.3，讲讲 IEEE 754 ，如何让其相等？

建议先阅读这篇文章了解 IEEE 754 ：[硬核基础二进制篇（一）0.1 + 0.2 != 0.3 和 IEEE-754 标准](https://juejin.cn/post/6940405970954616839 "https://juejin.cn/post/6940405970954616839")。 再阅读这篇文章了解如何运算：[0.1 + 0.2 不等于 0.3？为什么 JavaScript 有这种“骚”操作？](https://juejin.cn/post/6844903680362151950 "https://juejin.cn/post/6844903680362151950")。 ​

原因总结：

+   `进制转换` ：js 在做数字计算的时候，0.1 和 0.2 都会被转成二进制后无限循环 ，但是 js 采用的 IEEE 754 二进制浮点运算，最大可以存储 53 位有效数字，于是大于 53 位后面的会全部截掉，将导致精度丢失。
+   `对阶运算` ：由于指数位数不相同，运算时需要对阶运算，阶小的尾数要根据阶差来右移（`0舍1入`），尾数位移时可能会发生数丢失的情况，影响精度。

解决办法：

1.  转为整数（大数）运算。

```javascript
function add(a, b) {
  // 获取小数部分的最大长度
  const maxLen = Math.max(
    a.toString().split(".")[1].length,
    b.toString().split(".")[1].length
  );
  // 计算倍数
  const base = 10 ** maxLen;
  return (base * a + base * b) / base;
}
```

2.  使用 `Number.EPSILON` 误差范围。

```javascript
function isEqual(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true
```

`Number.EPSILON` 的实质是一个可以接受的最小误差范围，一般来说为 `Math.pow(2, -52)` 。 ​

3.  转成字符串，对字符串做加法运算。

```javascript
// 字符串数字相加
var addStrings = function (num1, num2) {
  let i = num1.length - 1;
  let j = num2.length - 1;
  const res = [];
  let carry = 0;
  while (i >= 0 || j >= 0) {
    const n1 = i >= 0 ? Number(num1[i]) : 0;
    const n2 = j >= 0 ? Number(num2[j]) : 0;
    const sum = n1 + n2 + carry;
    res.unshift(sum % 10);
    carry = Math.floor(sum / 10);
    i--;
    j--;
  }
  if (carry) {
    res.unshift(carry);
  }
  return res.join("");
};

function isEqual(a, b, sum) {
  const [intStr1, deciStr1] = a.toString().split(".");
  const [intStr2, deciStr2] = b.toString().split(".");
  const inteSum = addStrings(intStr1, intStr2); // 获取整数相加部分
  const deciSum = addStrings(deciStr1, deciStr2); // 获取小数相加部分
  return inteSum + "." + deciSum === String(sum);
}

console.log(isEqual(0.1, 0.2, 0.3)); // true
```

这是 leetcode 上一道原题：[415\. 字符串相加](https://leetcode-cn.com/problems/add-strings/ "https://leetcode-cn.com/problems/add-strings/")。区别在于原题没有考虑小数，但是也是很简单的，我们分为两个部分计算就行。

## 作用域&作用域链

+   简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。作用域是一套规则，在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。
+   作用域链的作用是保证执行环境里有权访问的变量和函数是有序的，作用域链的变量只能向上访问，变量访问到window对象即被终止，作用域链向下访问变量是不被允许的。

##  原型和原型链

可以说这部分每家面试官都会问了。。首先理解的话，其实一张图即可，一段代码即可。

```javascript
function Foo() {}

let f1 = new Foo();
let f2 = new Foo();
```

千万别畏惧下面这张图，特别有用，一定要搞懂，熟到提笔就能默画出来。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a61ca07672a45d3aecf382100cc9719~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

总结：

+   原型：每一个 JavaScript 对象（null 除外）在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性，其实就是 `prototype` 对象。
+   原型链：由相互关联的原型组成的**链状结构**就是原型链。

先说出总结的话，再举例子说明如何顺着原型链找到某个属性。

参考：[原型链知识](proto.md)

参考：[JavaScript 深入之从原型到原型链](https://github.com/mqyqingfeng/blog/issues/2 "https://github.com/mqyqingfeng/blog/issues/2") 掌握基本概念，再阅读这篇文章[轻松理解 JS 原型原型链](https://juejin.cn/post/6844903989088092174 "https://juejin.cn/post/6844903989088092174")加深上图的印象。

## JavaScript 面向对象&继承

+   参考：[JavaScript面向对象&继承](inherit.md)

## 闭包及练习题

+   参考：[Javascript闭包详解](closure.md)

## 函数节流和防抖

+   参考：[函数节流和防抖](throttle_debounce.md)

## Javascript 中的this

+   参考：[Javascript 中的this（包含apply、call、bind）](this.md)

## 动手实现apply、call、bind

+   参考：[动手实现apply、call、bind](apply.md)

## 字符串/数组/数字/对象 常用API

+   参考：[内置API](_api.md)

## Javascript 小技巧帮你提升代码质量

1.  提炼函数
2.  合并重复的条件片段
3.  把条件分支语句提炼成函数
4.  合理使用循环
5.  提前让函数退出代替嵌套条件分支
6.  传递对象参数代替过长的参数列表
7.  少用三目运算符
8.  合理使用链式调用
9.  分解大型类
10.  活用位操作符
11.  纯函数

## 自己动手实现Promise

```js
const stateObj = {
    pending: 'pending',
    fulfilled: 'fulfilled',
    rejected: 'rejected'
}
class promise{
    constructor(callback) {
        this.state = stateObj.pending
        this.value = ''
        this.resolveFun = []
        this.rejectFun = []
        const resolve = (value) => {
            if (this.state !== stateObj.pending) return
            setTimeout(() => {
                this.state = stateObj.fulfilled
                this.value = value
                for(let fn of this.resolveFun) {
                    fn(value)
                }
            },0)
        }
        const reject = (value) => {
            if(this.state !== stateObj.pending) return
            setTimeout((value) => {
                this.state = stateObj.rejected
                this.value = value
                for(let fn of this.rejectFun) {
                    fn(value)
                }
            },0)
        }
        try{
            callback(resolve,reject)
        }catch (err){
            reject(err)
        }
    }
    then(onResolve, onReject) {
        // resolve 结果处理
        function resolveHandle(pro,value,resolve, reject) {
            if(pro === value) {
                reject('循环引用')
                return
            } else if (value instanceof promise) {
                value.then((val) => {
                    resolveHandle(pro,val,resolve,reject)
                },(res) => {
                    reject(res)
                })
            } else {
                resolve(value)
            }
        }
        // 值穿透
        onResolve = onResolve instanceof Function ? onResolve : value => value
        onReject = onReject instanceof Function ? onReject : res => { throw res }

        if(this.state === stateObj.pending) {
            // 装载函数,并为新promise延续
            let resolveFun = this.resolveFun
            let rejectFun = this.rejectFun
            let currentPromise = new promise((resolve,reject) => {
                resolveFun.push(function (val) {
                    try{
                        let res = onResolve(val)
                        // resolve(res)
                        resolveHandle(currentPromise,res,resolve,reject)
                    } catch(err) {
                        reject(err)
                    }
                })
                rejectFun.push(function(val) {
                    try{
                        let res = onReject(val)
                        // resolve(res)
                        resolveHandle(currentPromise,res,resolve,reject)
                    }catch(err) {
                        reject(err)
                    }
                })
            })
            return currentPromise
        } else {
            // 直接输出
            let value = this.value
            let state = this.state
            let res = null
            let currentPromise = new promise((resolve,reject) => {
                try{
                    if (state === stateObj.fulfilled) {
                        res = onResolve(value)
                    } else {
                        res = onReject(value)
                    }
                    resolveHandle(currentPromise,res,resolve,reject)
                } catch(err) {
                    reject(err)
                }
            })
            return currentPromise
        }
    }
    // 处理错误信息
    catch(rejected) {
        return this.then(null,rejected)
    }
    // 成功失败全部执行
    finally(callback){
        return this.then(callback,callback)
    }
    // all静态方法,等待所有都执行完毕，任何一个失败则失败
    static all(promiseList) {
        const length = promiseList.length
        let num = 0
        let valList = []
        return new promise((resolve,reject) => {
            promiseList.forEach((promise, index) => {
              promise.then((val) => {
                  valList[index] = val
                  num++
                  if(num === length) {
                    resolve(valList)
                  }
              }).catch((err) => {
                  reject(err)
              })
            })
        })
    }
    // allSettled静态方法，无论成功或失败都返回成功
    static allSettled(promiseList) {
        const length = promiseList.length
        let valList = []
        let num = 0
        return new promise((resolve,reject) => {
          promiseList.forEach((promise,index) => {
            promise.then((val) => {
                valList[index] = val
                num++
                if (num === length) {
                    resolve(valList)
                }
            }).catch((err) => {
                valList[index] = err
                num++
                if(num === length) {
                   resolve(valList)
                }
            })
          })
        })
    }
    // race静态方法，返回最先成功的一个，任何一个失败则失败
    static race(promiseList) {
        return new promise((resolve,reject) => {
            promiseList.forEach((promise,index) => {
              promise.then((res) => {
                resolve(res)
              }).catch((err) => {
                reject(err)
              })
            })
        })
    }
    // any静态方法，任何一个成功则返回，否则失败
    static any(promiseList) {
        const length = promiseList.length
        let valList = []
        let num = 0
        return new promise((resolve,reject) => {
          promiseList.forEach((promise,index) => {
            promise.then((val) => {
              resolve(val)
            }).catch((err) => {
                valList[index] = err
                num++
                if(num === length) {
                    reject(valList)
                }
            })
          })
        })
    }
    // resolve静态方法
    static resolve(val) {
        return new promise((resolve, reject) => {
          resolve(val)
        })
    }
    // try静态方法,执行同步或异步函数
    static try(func) {
        return new promise((resolve,reject) => {
            const res = func()
            resolve(res)
        })
    }
    // reject静态方法
    static reject(val) {
        return new promise((resolve,reject) => {
            reject(val)
        })
    }
}
```


## new操作符作用

1.  首先创一个新的空对象。
2.  根据原型链，设置空对象的 `__proto__` 为构造函数的 `prototype` 。
3.  构造函数的 this 指向这个对象，执行构造函数的代码（为这个新对象添加属性）。
4.  判断函数的返回值类型，如果是引用类型，就返回这个引用类型的对象。

```javascript
function myNew(context) {
  const obj = new Object();
  obj.__proto__ = context.prototype;
  const res = context.apply(obj, [...arguments].slice(1));
  return typeof res === "object" ? res : obj;
}
```

## 严格模式的限制

1.  变量必须声明后再使用
2.  函数的参数不能有同名属性，否则报错
3.  不能使用with语句
4.  不能对只读属性赋值，否则报错
5.  不能使用前缀0表示八进制数，否则报错
6.  不能删除不可删除的属性，否则报错
7.  不能删除变量delete prop，会报错，只能删除属性delete global\[prop\]
8.  eval不会在它的外层作用域引入变量
9.  eval和arguments不能被重新赋值
10.  arguments不会自动反映函数参数的变化
11.  不能使用arguments.callee
12.  不能使用arguments.caller
13.  禁止this指向全局对象
14.  不能使用fn.caller和fn.arguments获取函数调用的堆栈
15.  增加了保留字（比如protected、static和interface）

## es6新特性

1.  let const
2.  字符串、数组、对象的方法扩展
3.  symbol、set、map新的数据类型和数据结构
4.  proxy代理拦截
5.  异步解决方案：promise、generate，async、await
6.  class类
7.  module模块

详情可参考：[ES6 入门教程 - 阮一峰](https://es6.ruanyifeng.com/ "https://es6.ruanyifeng.com/")

## 面向对象编程 & 面向过程编程

**概念**：

+   面向过程是将程序拆分为**步骤**，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了；
+   面向对象是将程序看作由一系列对象构成，对象通过通过封装、继承、多态等机制实现各种功能.

**优缺点**：

1.  面向过程：

+   **优点**：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如嵌入式开发、Linux/Unix等一般采用面向过程开发，性能是重要的因素。
+   **缺点**：没有面向对象易维护、易复用、易扩展

2.  面向对象

+   **优点**：可读性高，易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护。
    +   易维护：每个开发人员只需要维护自己所负则的那个类的功能字段和方法的定义和扩展就OK了。
    +   易复用：对象与对象之间相互独立，功能与对象之前的耦合性小。
    +   易扩展：对象的属于与方法扩展性强。
+   **缺点**：类调用时需要实例化，开销比较大，性能比面向过程低

## 箭头函数与普通函数的区别

1.  箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象
2.  箭头函数不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误
3.  箭头函数不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替
4.  箭头函数不可以使用yield命令，因此箭头函数不能用作Generator函数

## 异步编程的实现方式

1.  回调函数  
    优点：简单、容易理解 缺点：不利于维护，代码耦合高

2.  事件监听(采用时间驱动模式，取决于某个事件是否发生)  
    优点：容易理解，可以绑定多个事件，每个事件可以指定多个回调函数 缺点：事件驱动型，流程不够清晰

3.  发布/订阅(观察者模式)  
    类似于事件监听，但是可以通过‘消息中心’，了解现在有多少发布者，多少订阅者 Promise对象 优点：可以利用then方法，进行链式写法；可以书写错误时的回调函数； 缺点：编写和理解，相对比较难

4.  Generator函数 优点：函数体内外的数据交换、错误处理机制 缺点：流程管理不方便

5.  async函数 优点：内置执行器、更好的语义、更广的适用性、返回的是Promise、结构清晰。 缺点：错误处理机制


## 事件循环 & 宏任务、微任务

事件循环：event loop 宏任务：macrotask 也称为 task 微任务：microtask 也称为 jobs

**微任务**

+   process.nextTick (Node)
+   promise(构造函数为同步任务, then等方法为微任务)
+   Object.observe
+   MutationObserver (浏览器)

**宏任务**

+   script
+   setTimeout
+   setInterval
+   setImmediate
+   I/O
+   UI rendering
+   回调函数（事件、ajax等）

在一次事件循环中，会先执行加载js脚本产生宏任务列表,执行第一个宏任务，在这个宏任务中产生微任务microtask（promise），执行完这个宏任务的同步内容后,执行所有微任务，如果微任务中依然产生了微任务,则继续执行所有的微任务，微任务执行完毕后,继续顺序执行列表中下一个宏任务macrotask（setTimeout、setInterval）。

一个事件循环的执行步骤：

+   执行同步代码，这属于宏任务
+   执行所有微任务
+   必要的话渲染 UI
+   然后开始下一轮 Event loop，执行宏任务中的异步代码


```js
setTimeout(() => console.log(1), 0);

new Promise((resolve) => resolve(''), () => {})
    .then(() => {
        console.log(2);
        setTimeout(() => console.log(4), 0);
        new Promise((resolve) => resolve(''), () => {}).then(() => console.log(3))
    });

// 结果：2 3 1 4
```

## Javascript垃圾回收

+   **标记清除**：标记阶段即为所有活动对象做上标记，清除阶段则把没有标记（也就是非活动对象）销毁。
+   **引用计数**：它把**对象是否不再需要**简化定义为**对象有没有其他对象引用到它**。如果没有引用指向该对象（引用计数为 0），对象将被垃圾回收机制回收。

标记清除的缺点：

+   **内存碎片化**，空闲内存块是不连续的，容易出现很多空闲内存块，还可能会出现分配所需内存过大的对象时找不到合适的块。
+   **分配速度慢**，因为即便是使用 First-fit 策略，其操作仍是一个 O(n) 的操作，最坏情况是每次都要遍历到最后，同时因为碎片化，大对象的分配效率会更慢。

解决以上的缺点可以使用 \*\*标记整理（Mark-Compact）算法 \*\*，标记结束后，标记整理算法会将活着的对象（即不需要清理的对象）向内存的一端移动，最后清理掉边界的内存（如下图） ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb543f2fdc634e29add495b8f2ff048f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

引用计数的缺点：

+   需要一个计数器，所占内存空间大，因为我们也不知道被引用数量的上限。
+   解决不了循环引用导致的无法回收问题。

V8 的垃圾回收机制也是基于标记清除算法，不过对其做了一些优化。

+   针对新生区采用并行回收。
+   针对老生区采用增量标记与惰性回收。

## 深浅拷贝

**浅拷贝**：仅拷贝第一层

+   `Object.assign`，
+   展开运算符 `...`

**深拷贝**：内部引用数据类型也进行重新创建复制,不会造成数据引用冲突

简易版本 :`JSON.parse(JSON.stringify(object))` **缺陷**：

+   会忽略 undefined
+   会忽略 symbol
+   不能序列化函数
+   不能解决循环引用的对象

完整版本:
```js
/**
 * 深拷贝
 * @param {Object} obj 要拷贝的对象
 * @param {Map} map 用于存储循环引用对象的地址
 */

function deepClone(obj = {}, map = new Map()) {
  if (typeof obj !== "object") {
    return obj;
  }
  if (map.get(obj)) {
    return map.get(obj);
  }

  let result = {};
  // 初始化返回结果
  if (
    obj instanceof Array ||
    // 加 || 的原因是为了防止 Array 的 prototype 被重写，Array.isArray 也是如此
    Object.prototype.toString(obj) === "[object Array]"
  ) {
    result = [];
  }
  // 防止循环引用
  map.set(obj, result);
  for (const key in obj) {
    // 保证 key 不是原型属性
    if (obj.hasOwnProperty(key)) {
      // 递归调用
      result[key] = deepClone(obj[key], map);
    }
  }

  // 返回结果
  return result;
}
```

## 进程与线程

本质上来说，进程与线程都是 CPU 工作时间片的一个描述。

+   进程描述了 CPU 在运行指令及加载和保存上下文所需的时间，放在应用上来说就代表了一个程序。
+   线程是进程中的更小单位，描述了执行一段指令所需的时间。

**举例**： 把这些概念拿到浏览器中来说，当你打开一个 `Tab` 页时，其实就是创建了一个**进程**，一个进程中可以有多个线程，比如**渲染**线程、**JS 引擎**线程、**HTTP 请求**线程等等。当你发起一个请求时，其实就是创建了一个线程，当请求结束后，该线程可能就会被销毁。

**单线程的好处**： 上文说到了 JS 引擎线程和渲染线程，大家应该都知道，在 JS 运行的时候可能会阻止 UI 渲染，这说明了两个线程是互斥的。这其中的原因是因为 JS 可以修改 DOM，如果在 JS 执行的时候 UI 线程还在工作，就可能导致不能安全的渲染 UI。这其实也是一个单线程的好处，得益于 JS 是单线程运行的，可以达到节省内存，节约上下文切换时间，没有锁的问题的好处

## ajax

```js
function ajaxGet(url, params, success, fail) {
    // 1. 创建连接
    var xhr = null;
    xhr = new XMLHttpRequest()
    // 2. 连接服务器
    xhr.open('get', url + encodeParams(params), true)
    // 3. 发送请求
    xhr.send(null);
    // 4. 接受请求
    xhr.onreadystatechange = function(){
        if(xhr.readyState == 4){
            if((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304){
                success(xhr.responseText);
            } else {
                fail && fail(xhr.status);
            }
        }
    }
    function encodeParams(obj){
	return Object
            .keys(obj)
            .map(function(key) {return encodeURIConponent(key) + '=' +encodeURIConponent(obj[key])})
            .join('&');
    }
}
function ajaxPost(url, params, success, fail) {
    // 1. 创建连接
    var xhr = null;
    xhr = new XMLHttpRequest()
    // 2. 接受请求
    xhr.onreadystatechange = function(){
        if(xhr.readyState == 4){
            if((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304){
                success(xhr.responseText);
            } else {
                fail && fail(xhr.status);
            }
        }
    }
    // 3. 连接服务器
    xhr.open('post', url, true)
    // 4. 发送请求
    xhr.send(JSON.stringify(params));
}
```

## 正则表达式
+ 参考: [正则表达式](https://www.runoob.com/regexp/regexp-syntax.html)

## jQuery如何扩展自定义方法

```js
jQuery.fn.myMethod = function () {
    alert('myMethod');
};
// 或者：
jQuery.fn.extend({
    myMethod: function () {
        alert('myMethod');
    }
});
// 使用：
$("#div").myMethod();
```

## 事件委托

将事件绑定到父节点上，因为事件冒泡的缘故可以获取到实际点击的叶子节点，借此减少dom事件绑定的次数。

```js
document.body.addEventListener('click', e => console.log(e));
/*
{
    currentTarget: null, // 当前绑定事件的节点，那为什么是null呢？看下面解释
    target: h1#title.title // 点击最下层的叶子结点
}
*/
```

为何`currentTarget`是`null`：

+   这是 console 在log一个对象的机制造成的。log 没有包含对象的所有属性，它只包含了对这个对象的引用。当你点击展开时，他才会给你找那个对象的属性。
+   所以，之所以为null的情况是，当调用console.log(e)时，currentTarget属性是有值的，但是过后这个值就被重置为null了。所以当你展开事件对象，看到的就是null。


## 微前端

微前端是将传统前端单体应用拆分成多个小型、独立的前端子应用的架构方式。每个子应用可以由不同团队独立开发、部署和维护，就像微服务在后端架构中的作用一样。

**应用场景:**

1. **大型企业级应用**  
   当应用功能模块众多且复杂时，采用微前端可以将各个模块独立拆分，降低系统耦合度，便于维护和扩展。

2. **多团队协作开发**  
   在多个团队共同开发同一个产品时，微前端允许各团队使用自己熟悉的技术栈独立构建、测试和部署子应用，从而提升开发效率。

3. **渐进式升级与技术演进**  
   对于已有的单体前端应用，通过逐步引入微前端架构，可以实现平滑过渡，逐步替换老旧技术，同时保持系统的持续稳定运行。

4. **跨平台应用开发**  
   当需要针对不同终端（如桌面、移动端、甚至嵌入式设备）进行定制化开发时，微前端架构能使各个子应用专注于特定平台的优化，增强用户体验。

5. **独立部署和按需加载**  
   在需要实现模块化部署、快速迭代以及按需加载的场景下，微前端可以减少整体更新的风险，每个子应用独立部署，更新时不影响其他部分。


**优缺点:**

| 优点                                                         | 缺点                                                         |
|--------------------------------------------------------------|--------------------------------------------------------------|
| **独立开发和部署**：不同团队可以采用各自适合的技术栈和开发周期，无需同步迭代。 | **系统复杂性提升**：各子应用之间的集成、通信和统一路由管理增加了整体系统的复杂性。 |
| **可扩展性好**：单个子应用可以独立扩展，降低了整体应用的维护和更新难度。 | **性能问题**：可能会存在重复加载多个框架、库，导致首屏加载时间增加。 |
| **技术多样性**：允许使用不同技术方案和版本，方便渐进式技术演进。 | **用户体验不统一**：不同团队的风格和交互设计不一致，可能影响整体体验。 |
| **容错性高**：单个子应用出现问题时，不会导致整个应用崩溃。 | **部署和版本管理难度大**：多个独立部署的子应用需要有良好的版本控制和协同管理机制。 |


**常用的微前端框架:**


| 框架                        | 特点                                                         | 优点                                                            | 缺点                                                            |
|-----------------------------|--------------------------------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------------------|
| **Single-SPA**              | 最早支持多框架整合，灵活性较高                                  | 支持 React、Angular、Vue 等多种框架混合使用，社区较成熟           | 配置较复杂，学习曲线较陡峭，需要额外处理子应用间通信和路由管理       |
| **Qiankun**                 | 基于 Single-SPA 封装，提供完善的隔离和通信机制                  | 简化了配置和集成，子应用隔离效果好，适合企业级应用                | 依赖 Single-SPA 架构，生态相对局限，对特定场景适配性要求较高           |
| **Webpack Module Federation** | Webpack 5 内置的模块共享机制，实现运行时动态加载和模块联邦         | 无需额外引入框架，适合现有 Webpack 项目，实现模块级的按需加载       | 需要对 Webpack 有深入了解，调试和跨应用依赖管理可能较为复杂              |
| **Piral**                   | 平台化微前端解决方案，提供丰富的 API 和插件支持                  | 高度可扩展，适合构建插件化的微前端系统，降低应用间耦合性             | 社区生态较小，文档和支持资源相对有限，初期上手可能需要额外投入            |



---

## Typescript

[参考: TS 常用内容](./simple_ts.md)

---

1. TypeScript 的主要优势是什么？

- **静态类型检查**：在编译期发现类型错误，减少运行时错误。
- **更好的编辑器支持**：提供智能提示、重构和自动补全。
- **可读性和可维护性**：通过类型系统使代码意图更明确。
- **丰富的高级类型系统**：支持交叉类型、联合类型、类型守卫、泛型、映射类型等。

---

2. TypeScript 与 JavaScript 的区别？

- **类型系统**：TS 是 JavaScript 的超集，增加了静态类型检查。
- **编译过程**：TypeScript 代码需要编译成 JavaScript 才能运行。
- **ES 新特性支持**：TS 支持部分 ES 的未来特性，并通过编译器转译成目标环境支持的版本。

---


3. TypeScript 的类型系统有哪些？
TypeScript 提供了丰富的类型系统，主要包括以下几类：
- **基本数据类型：**
    - `number`、`string`、`boolean`、`null`、`undefined`、`symbol`
- **特殊类型：**
    - `any`（任意类型，不进行类型检查）、`unknown`（未知类型，必须做类型检查）、`void`（一般用于函数无返回值）、`never`（永远不会返回结果，比如抛出异常或无限循环）
- **高级类型：**
    - **联合类型（Union Types）：** 如 `string | number` 表示可以是字符串或数字
    - **交叉类型（Intersection Types）：** 组合多个类型，满足所有类型的要求
    - **枚举（Enum）：** 用于定义一组命名常量
    - **字面量类型（Literal Types）：** 限定变量只能取固定值
    - **泛型（Generics）：** 编写与类型无关的复用代码
    - **类型推断与类型保护：** 自动根据值推断类型，并使用 `typeof`、`instanceof` 或自定义类型守卫缩小类型范围
    - **映射类型（Mapped Types）：** 根据已有类型生成新类型

---

4. interface 与 type 有什么区别？
- **扩展性：**
    - `interface` 支持声明合并，可以在不同地方对同一个接口进行扩展。
    - `type` 则是类型别名，不支持声明合并。
- **适用范围：**
    - `interface` 主要用于描述对象的结构或类的形状。
    - `type` 不仅可以描述对象，还可以用于联合类型、交叉类型以及原始类型的别名。
- **选择建议：**
    - 当需要描述对象结构且预期未来可能扩展时，推荐使用 `interface`。
    - 当需要定义更复杂的类型（如联合、交叉或原始类型别名）时，使用 `type` 更为合适。

---

5. 如何使用泛型（Generics）？
泛型允许函数、接口、类在保持类型安全的同时处理多种数据类型，提高代码的复用性。
- **泛型函数示例：**
  ```typescript
  function identity<T>(arg: T): T {
      return arg;
  }
  // 使用时会根据传入参数自动推断类型：
  let output = identity<string>("Hello");
  ```
- **泛型接口和类：**  
  定义接口或类时也可以使用泛型来适应不同的数据类型。
- **泛型约束：**  
  使用 `extends` 关键字限定泛型必须满足的条件，例如：
  ```typescript
  interface Lengthwise {
      length: number;
  }
  function logLength<T extends Lengthwise>(arg: T): void {
      console.log(arg.length);
  }
  ```

---

6. TypeScript 中的装饰器（Decorators）
装饰器是一种特殊的声明方式，可以附加到类、方法、属性或参数上，用于修改类的行为或添加元数据。
- **使用场景：**  
  常见于 Angular 等框架中，用于依赖注入、方法增强、日志记录等场景。
- **使用前提：**  
  需要在 `tsconfig.json` 中开启 `experimentalDecorators` 选项。
- **示例：**
  ```typescript
  function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      const originalMethod = descriptor.value;
      descriptor.value = function (...args: any[]) {
          console.log(`Calling ${propertyKey} with`, args);
          return originalMethod.apply(this, args);
      };
  }
  
  class Example {
      @log
      sum(a: number, b: number): number {
          return a + b;
      }
  }
  ```

---


7. 联合类型和交叉类型的区别？

- **联合类型（Union Types）**：表示一个变量可以是几种类型中的一种，使用 `|` 连接。
  ```ts
  let value: string | number
  ```
- **交叉类型（Intersection Types）**：表示一个变量同时满足多个类型的要求，使用 `&` 连接。
  ```ts
  type A = { a: number }
  type B = { b: string }
  type C = A & B  // 必须同时包含 a 和 b
  ```

---

8. TypeScript 中的类型断言（Type Assertion）和非空断言（Non-null Assertion）的用法

- **类型断言**：告诉编译器变量的实际类型，常用于绕过编译器类型检查。
  ```ts
  const someValue: any = 'hello'
  const strLength: number = (someValue as string).length
  ```
- **非空断言**：用于告诉编译器某个表达式不会为 null 或 undefined。
  ```ts
  const el = document.getElementById('app')!
  ```

---

9. Declaration Merging（声明合并）是什么？

**Declaration Merging（声明合并）** 指的是当 TypeScript 遇到 **多个同名的接口、命名空间或模块** 时，它们会自动合并为一个。

**1 接口合并**
如果同名接口被声明多次，TypeScript 会将它们合并：
```ts
interface Person {
  name: string
}

interface Person {
  age: number
}

// 合并后的效果
const person: Person = {
  name: 'Alice',
  age: 25
}
```

**2 扩展已有的 `Window` 对象**
```ts
interface Window {
  myCustomProperty: string
}

window.myCustomProperty = 'Hello'
```

**3 命名空间合并**
```ts
namespace Library {
  export function doSomething() {
    console.log('Doing something')
  }
}

namespace Library {
  export function anotherFunction() {
    console.log('Another function')
  }
}

// 可以像一个合并的对象一样使用
Library.doSomething()
Library.anotherFunction()
```

---



10. 类型推断（Type Inference）是什么？
- **类型推断：**  
  当声明变量时如果没有明确的类型注解，TypeScript 会根据赋值自动推断出变量的类型。例如：
  ```typescript
  let num = 42; // TypeScript 推断 num 的类型为 number
  ```

---

11. 什么是 TypeScript 的声明文件（.d.ts 文件）？
声明文件用于描述 JavaScript 库或模块的类型信息，从而让 TypeScript 项目在引入这些库时能够进行类型检查和代码补全。
- **来源：**  
  大部分流行的 JavaScript 库都有社区提供的声明文件，通常可以通过 npm 包 `@types/xxx` 来获取。
- **用途：**  
  不需要修改 JavaScript 库的源码，只需提供外部声明文件就能获得静态类型支持。

---

12. 如何配置 tsconfig.json？
`tsconfig.json` 文件用于配置 TypeScript 编译器的行为，主要包含以下几个部分：
- **compilerOptions：**  
  定义编译选项，例如：
    - `target`：编译后 JavaScript 的目标版本（如 ES5、ES6/ES2015 等）。
    - `module`：模块系统的选择（如 CommonJS、ES6 modules）。
    - `strict`：开启所有严格类型检查选项。
    - 其他如 `outDir`、`sourceMap`、`experimentalDecorators` 等选项。
- **include 与 exclude：**  
  分别指定要包含和排除的文件路径。

示例配置：
  ```json
  {
      "compilerOptions": {
          "target": "ES6",
          "module": "commonjs",
          "strict": true,
          "experimentalDecorators": true,
          "outDir": "./dist",
          "sourceMap": true
      },
      "include": ["src/**/*"],
      "exclude": ["node_modules", "**/*.spec.ts"]
  }
  ```

---

13. TypeScript 中的类型保护
类型保护用于在运行时缩小变量的类型范围，从而使编译器能够更准确地推断出变量类型。
   - 使用 `typeof`：适用于基本数据类型。
   - 使用 `instanceof`：适用于类实例检查。
   - 自定义类型保护函数：使用类型谓词返回 `value is Type`。
- **示例：**
  ```typescript
  function isString(value: unknown): value is string {
      return typeof value === 'string';
  }
  
  function printValue(value: unknown) {
      if (isString(value)) {
          // 这里 TypeScript 知道 value 是 string 类型
          console.log(value.toUpperCase());
      } else {
          console.log(value);
      }
  }
  ```

---

14. TypeScript 与 ES6/ESNext 特性的关系
- **兼容性：**  
  TypeScript 支持大部分 ES6/ESNext 的新特性，例如箭头函数、类、模块、解构赋值、异步编程等。
- **编译目标：**  
  通过配置 `target` 选项，可以将 TypeScript 编译为兼容旧版 JavaScript 环境的代码。例如，将 ES6 特性转换为 ES5 代码。
- **增强类型安全：**  
  在使用 ES6/ESNext 新特性的同时，TypeScript 提供了静态类型检查，使得开发过程中能够提前发现错误并提高代码的可维护性。

---
