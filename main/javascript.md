## 数据类型

以下是比较重要的几个 js 变量要掌握的点。

#### 1.1 基本的数据类型介绍，及值类型和引用类型的理解

在 JS 中共有 `8`  种基础的数据类型，分别为： `Undefined` 、 `Null` 、 `Boolean` 、 `Number` 、 `String` 、 `Object` 、 `Symbol` 、 `BigInt` 。

其中 `Symbol`  和 `BigInt`  是 ES6 新增的数据类型，可能会被单独问：

+   Symbol 代表独一无二的值，最大的用法是用来定义对象的唯一属性名。
+   BigInt 可以表示任意大小的整数。

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
  const maxLen = Math.max(
    a.toString().split(".")[1].length,
    b.toString().split(".")[1].length
  );
  const base = 10 ** maxLen;
  const bigA = BigInt(base * a);
  const bigB = BigInt(base * b);
  const bigRes = (bigA + bigB) / BigInt(base); // 如果是 (1n + 2n) / 10n 是等于 0n的。。。
  return Number(bigRes);
}
```

这里代码是有问题的，因为最后计算 `bigRes` 的大数相除（即 `/`）是会把小数部分截掉的，所以我很疑惑为什么网络上很多文章都说可以通过**先转为整数运算再除回去，为了防止转为的整数超出 js 表示范围，还可以运用到 ES6 新增的大数类型，我真的很疑惑，希望有好心人能解答下。**

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

参考：[JavaScript 深入之从原型到原型链](https://github.com/mqyqingfeng/blog/issues/2 "https://github.com/mqyqingfeng/blog/issues/2") 掌握基本概念，再阅读这篇文章[轻松理解 JS 原型原型链](https://juejin.cn/post/6844903989088092174 "https://juejin.cn/post/6844903989088092174")加深上图的印象。
参考：[原型链](proto.md)


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
                try {
                  promise.then((val) => {
                      valList[index] = val
                      num++
                      if(num === length) {
                        resolve(valList)
                      }
                  })
                }catch (err){
                    reject(err)
                }
            }).catch((err) => {
              reject(err)
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

## 作用域&作用域链

+   简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。作用域是一套规则，在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。
+   作用域链的作用是保证执行环境里有权访问的变量和函数是有序的，作用域链的变量只能向上访问，变量访问到window对象即被终止，作用域链向下访问变量是不被允许的。

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

+   面向过程就是分析出解决问题所需要的**步骤**，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了；
+   面向对象是把构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为。面向对象是以**功能**来划分问题，而不是**步骤**。面向对象使用对象，类，继承，封装等基本概念来进行程序设计。

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

1.  函数体内的this对象，就是定义时所在的对象，而不是使��时所在的对象
2.  不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误
3.  不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替
4.  不可以使用yield命令，因此箭头函数不能用作Generator函数

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

在一个事件循环中，微任务会比宏任务先执行，因此即使`new promise()`里立即执行的代码比`setTimeout(() => {}, 0)`的代码后面，也会提前执行。

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
            .map(function(key) {return encodeURIConponent(key) + '=' +encodeURIConponent(obj[key]})
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

## § typescript

TypeScript是一种由微软开发和维护的免费开源编程语言。它是一个强类型的JavaScript超集，目前不能直接在浏览器环境运行，可编译为纯JavaScript后执行。

TypeScript可以被认为是Javascript的优化版本，弃其糟粕，取其精华，在保持其动态语言的相对自由度前提下，修饰了没有编译类型检查，不适合超大型项目的缺陷。

**使用ts的场景**：TypeScript拥有更好的类型检查，但在某种程度上也让js的灵活性收到了限制。因此我们应该在日常开发相对独立的业务需求使用js，在开发多人协作、逻辑复杂的大型项目时使用ts。

+   参考：[TypeScript 入门教程](https://ts.xcatliu.com/ "https://ts.xcatliu.com/")

### ts 和 js 详细对比

|  | JavaScript | TypeScript |
| --- | --- | --- |
| 1 | 它是由网景公司在1995年开发的。 | 它是2012年由安德斯·海尔斯伯格(Anders Hejlsberg)开发的。 |
| 2 | JavaScript源文件在”。js”扩展。 | TypeScript源文件是”.ts”扩展名。 |
| 3 | JavaScript不支持ES6。 | TypeScript 支持ES6。 |
| 4 | 它不支持强类型或静态类型。 | 它支持强类型或静态类型特性。 |
| 5 | 它只是一种脚本语言。 | 它支持面向对象的编程概念，如类、接口、继承、泛型等。 |
| 6 | JavaScript没有可选的参数特性。 | TypeScript有可选的参数特性。 |
| 7 | 它是解释语言，这就是为什么它在运行时突出显示错误。 | 它编译代码并在开发期间突出显示错误。 |
| 8 | JavaScript不支持模块。 | TypeScript支持模块。 |
| 9 | 在这里，number和string是对象。 | 在这里，number和string是接口。 |
| 10 | JavaScript不支持泛型。 | TypeScript支持泛型。 |

**面向对象**： 三大特性："封装、"多态"、"继承"， 五大原则："单一职责原则"、"开放封闭原则"、"里氏替换原则"、"依赖倒置原则"、"接口分离原则"、"迪米特原则（高内聚低耦合）"

+   模块
+   类
+   接口
+   继承
+   封装
+   多态
+   数据类型

### ts安装和使用

```sh
# 全局安装typescript
npm install -g typescript  
# 安装ts后会自动安装一个命令行工具“tsc”，它将用于编译ts代码生成js文件。
tsc helloworld.ts  
# 实时热更新编译
tsc --watch helloworld.ts  
# ts源码调试
tsc -sourcemap helloworld.ts  
```

### interface 和 type 的区别

**1\. interface接口** **定义**：对值所具有的结构进行类型检查。 **作用**：为类型命名和第三方代码定义数据类型规范。

**2\. type类型别名**

**作用**：给类型起个新名字。

**3\. type 可以而 interface 不行**

+   type 可以声明基本类型别名，联合类型，元组等类型，类和interface不能(只能描述对象和函数)。
+   type 语句中还可以使用 typeof 获取实例的类型进行赋值

**4\. interface 可以而 type 不行**

+   interface 能够声明合并（当接口名字相同，取并集）

**5\. interface 和 type 都可以**

+   两者都可以用来描述对象或函数的类型，但是语法不同。

```ts
// 接口
interface Point {
  x: number;
  y: number;
}
interface SetPoint {
  (x: number, y: number): void;
}
// 类型别名
type Point = {
  x: number;
  y: number;
};
type SetPoint = (x: number, y: number) => void;
```

+   两者都可以扩展，但是语法不同

类型别名不能**继承**其他的类型别名或者接口，也不能**被继承**。

```ts
// 从 interface 扩展到 interface
interface PartialPointX { x: number; }
interface Point extends PartialPointX { y: number; }

// 从 type 扩展到 type
type PartialPointX = { x: number; };
type Point = PartialPointX & { y: number; };

// 从 type 扩展到 interface
type PartialPointX = { x: number; };
interface Point extends PartialPointX { y: number; }

// 从 interface 扩展到 type
interface PartialPointX { x: number; }
type Point = PartialPointX & { y: number; };
```

**6\. 最后** 如果不清楚什么时候用interface/type，能用 interface 实现，就用 interface , 如果不能就用 type。

### 元祖

> ts数组合并了相同类型的对象，而元组（Tuple）合并了不同类型的对象。

js里面的**数组**本身包含了**元祖**特性，支持多类型数组。

```ts
let tom: [string, number] = ['Tom', 25];
```

### 枚举

> 枚举（Enum）类型用于取值被限定在一定范围内的场景，比如一周只能有七天，颜色限定为红绿蓝等。

+   枚举成员会被赋值为从 0 开始递增的数字，同时也会对枚举值到枚举名进行反向映射
+   可以给枚举项手动赋值

```ts
enum Days {Sun, Mon = 1, Tue, Wed, Thu, Fri, Sat};
console.log(Days["Sun"] === 0); // true
console.log(Days[0] === "Sun"); // true
// 上面的代码实际会被编译为：Days[Days["Sun"] = 0] = "Sun";

console.log(Days["Sun"] === 7); // true 手动赋值
```

### 范型

> 泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。

+   参考：[TypeScript 入门教程 - 泛型](https://ts.xcatliu.com/advanced/generics.html "https://ts.xcatliu.com/advanced/generics.html")

```ts
// 可以指定默认类型：T = string
function createArray<T = string>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray<string>(3, 'x'); // ['x', 'x', 'x']
createArray(3, 'x'); // 也可以自动推断类型
```

**§ 多个类型参数** 定义泛型的时候，可以借助元祖一次定义多个类型参数：

```ts
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}

swap([7, 'seven']); // ['seven', 7] 
```

**§ 泛型约束** 在函数内部使用泛型变量的时候，由于事先不知道它是哪种类型，所以不能随意的操作它的属性或方法

这种情况可以对泛型进行约束，只允许这个函数传入那些包含 length 属性的变量。这就是泛型约束：

```ts
interface Lengthwise {
    length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```

### 枚举

```ts
enum statusCode{
    ok=0,
    error=1,
    pedding=2
}
statusCode[0] === 'ok'
statusCode['ok'] === 0

```

