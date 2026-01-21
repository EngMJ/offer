## 数据类型

### 1.1 基本的数据类型介绍，及值类型和引用类型的理解

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

## 原型和原型链

可以说这部分每家面试官都会问了。首先理解的话，其实一张图即可，一段代码即可。

```javascript
function Foo() {}

let f1 = new Foo();
let f2 = new Foo();
```

千万别畏惧下面这张图，特别有用，一定要搞懂，熟到提笔就能默画出来。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a61ca07672a45d3aecf382100cc9719~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**总结：**

+   **原型**：每一个 JavaScript 对象（null 除外）在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性，其实就是 `prototype` 对象。
+   **原型链**：由相互关联的原型组成的**链状结构**就是原型链。

### 2.1 原型的作用

原型在 Javascript 中是一个非常重要的概念，其作用是：**共享属性**

如果在一个对象上找不到某个属性，就会去它的原型对象上找，以此类推直至找到，或者寻找到原型链的终点都没找到则不存在这个属性。

### 2.2 `__proto__` & `constructor`

+   **`__proto__`**：隐式原型，多数浏览器实现了此属性用来访问对象的原型。
+   **constructor**：构造器，创建一个函数时会为它增加一个 prototype 属性，指向原型对象，原型对象自动获得一个名为 constructor 的属性，指回与之关联的构造函数。

### 2.3 原型链示意图

原型链最终会指向 `Object.prototype`，原型链的终点是 `Object.prototype.__proto__` 也就是 `null`。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/911b3e448e5248ec96d973b076e012c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 2.4 原型属性判断方法

```javascript
// getPrototypeOf - 访问指定对象的 prototype
Object.getPrototypeOf(obj);

// setPrototypeOf - 设置指定对象的 prototype
Object.setPrototypeOf(obj, obj1);

// hasOwnProperty - 是否仅属于对象本身的属性
obj.hasOwnProperty('name');

// in - 对象或原型链上存在就返回 true
'name' in obj;

// Object.keys - 对象上所有可枚举的实例本身属性
Object.keys(obj);
```

---

## JavaScript 继承

### 3.1 原型链继承

**原理**：使用父类的实例重写子类的原型。

```js
function SuperType(){
    this.property = true;
}
SuperType.prototype.getSuperValue = function(){
    return this.property;
};
function SubType(){
    this.subproperty = false;
}
// 继承了 SuperType
SubType.prototype = new SuperType();

var instance = new SubType();
alert(instance.getSuperValue()); // true
```

**缺点**：
1. 父类包含引用类型值的原型属性会被所有子类共享；
2. 在创建子类的实例时，不能向父类的构造函数中传递参数。

### 3.2 借用构造函数

**原理**：在子类构造函数的内部调用父类的构造函数。

```js
function SuperType(){
    this.colors = ["red", "blue", "green"];
}
function SubType(){
    SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors); // "red,blue,green,black"

var instance2 = new SubType();
alert(instance2.colors); // "red,blue,green"
```

**优点**：属性和方法不会被子类所共享，可以向父类传参。  
**缺点**：父类中的方法无法被复用。

### 3.3 组合式继承

**原理**：使用原型链实现对原型属性和方法的继承，借用构造函数实现对实例属性的继承。

```js
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name, age){
    SuperType.call(this, name); // 继承属性
    this.age = age;
}
SubType.prototype = new SuperType(); // 继承方法
SubType.prototype.constructor = SubType;
```

**缺点**：调用两次父类构造函数。

### 3.4 寄生组合式继承（推荐）

```js
function inheritPrototype(subType, superType){
    var prototype = Object.create(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}

function SuperType(name){
    this.name = name;
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}
inheritPrototype(SubType, SuperType);
```

### 3.5 ES6 class 继承

```js
class Person {
    constructor(age) {
        this.age_ = age;
    }
    sayAge() {
        console.log(this.age_);
    }
    static create() {
        return new Person(Math.floor(Math.random()*100));
    }
}

class Doctor extends Person {}

const doctor = new Doctor(32);
doctor.sayAge(); // 32
```

本质上是 **寄生组合式继承** 的语法糖。

---

## 闭包

> 闭包是指有权访问另一个函数作用域中的变量的函数。

通俗理解：**函数 A 里面有个函数 B，函数 B 用到了函数 A 的变量，把函数 B 拿到外面去用，这就形成了闭包**。闭包让函数 A 的变量不会被销毁，一直"活"在内存里。

### 4.1 变量的作用域

变量的作用域，就是指变量的有效范围。随着代码执行环境创建的作用域链往外层逐层搜索，一直搜索到全局对象为止。

### 4.2 变量的生存周期

+   全局变量生存周期是永久的，除非主动销毁
+   函数作用域会随着函数调用的结束而被销毁
+   当函数内部调用外部变量就产生了**闭包**

### 4.3 闭包总结

+   **形成**：函数中嵌套函数
+   **作用**：函数内部调用外部变量、构造函数的私有属性、延长变量生命周期
+   **优点**：变量长期存在内存中、模块化代码避免全局污染、私有属性
+   **缺点**：无法回收闭包中引用变量，容易造成内存泄漏

### 4.4 闭包应用

#### 构造函数的私有属性
```js
function Person(param) {
    var name = param.name; // 私有属性
    this.age = 18; // 共有属性
    this.sayName = function () {
        console.log(name);
    }
}
const tom = new Person({name: 'tom'});
tom.name = 'jerry'; // 无法修改私有属性
tom.sayName(); // tom
```

#### 计算缓存
```js
var square = (function () {
    var cache = {};
    return function(n) {
        if (!cache[n]) {
            cache[n] = n * n;
        }
        return cache[n];
    }
})();
```

### 4.5 闭包高频面试题

```js
// 经典 for 循环问题
for (var i = 0; i < 5; i++) {
    setTimeout(function(){ console.log(i); }, 0);
}
// 输出: 5 5 5 5 5

// 解决方案1：IIFE
for (var i = 0; i < 5; i++) {
    (function(j){
        setTimeout(function(){ console.log(j); }, 0);
    })(i);
}
// 输出: 0 1 2 3 4

// 解决方案2：let
for (let i = 0; i < 5; i++) {
    setTimeout(function(){ console.log(i); }, 0);
}
// 输出: 0 1 2 3 4
```

---

## 函数节流和防抖

### 5.1 节流（Throttle）

只在开始执行一次，未执行完成过程中触发的忽略，核心在于**开关锁**。

**例如**：多次点击按钮提交表单，**第一次有效**

```js
function throttle(fn, delay) {
    var timer = null;
    return function () {
        if (timer) { return false; }
        var that = this;
        var args = arguments;
        fn.apply(that, args);
        timer = setTimeout(function () {
            clearTimeout(timer);
            timer = null;
        }, delay || 500);
    };
}
```

### 5.2 防抖（Debounce）

只执行最后一个被触发的，清除之前的异步任务，核心在于**清零**。

**例如**：页面滚动处理事件，搜索框输入联想，**最后一次有效**

```js
function debounce(fn, delay) {
    var timer = null;
    return function () {
        var that = this;
        var args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function () {
            fn.apply(that, args);
        }, delay || 500);
    };
}
```

### 5.3 图解对比

```
原始事件触发:
| | | | | | | | | | | | | | | | | | | | | (连续快速触发)

节流 (每 100ms 执行一次):
| _ _ _ _ | _ _ _ _ | _ _ _ _ | _ _ _ _ | (固定频率执行第一次)

防抖 (等待 100ms):
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ | (仅执行最后一次)
```

**记忆口诀**：节流是单位时间内只触发首次，防抖是只触发最后一次。

---

## this 指向

this 总是指向**执行时**的**当前对象**，取决于函数的调用方式。

### 6.1 作为对象的方法调用

```js
var obj = {
    a: 1,
    getA() {
        alert(this === obj); // true
        alert(this.a); // 1
    }
};
obj.getA();
```

### 6.2 作为普通函数调用

this 指向全局对象（浏览器中是 window）

```js
window.name = 'globalName';
var myObject = {
    name: 'sven',
    getName: function(){ return this.name; }
};
var getName = myObject.getName;
console.log(getName()); // globalName（this 丢失）
```

### 6.3 构造器调用

构造器里的 this 指向返回的实例对象（除非显式返回一个对象）

```js
var MyClass = function(){
    this.name = 'sven';
};
var obj = new MyClass();
alert(obj.name); // sven
```

### 6.4 call / apply 调用

可以动态改变函数的 this 指向

```js
var obj1 = { name: 'sven' };
var obj2 = { name: 'anne' };
function getName(){ return this.name; }
console.log(getName.call(obj2)); // anne
```

### 6.5 箭头函数中的 this

在箭头函数声明时就绑定对应的 this（词法作用域），后续任何地方调用 this 都一样。

---

## apply、call、bind 实现

### 7.1 区别

+   `call` 和 `apply` 会立即执行，区别仅在于传入参数形式的不同
+   `bind` 会返回一个修改原函数 this 与参数的函数

### 7.2 手写实现

```js
// apply 实现
Function.prototype.myApply = function (context) {
    var context = context || window;
    var args = arguments[1] || [];
    context.fn = this;
    var result = context.fn(...args);
    delete context.fn;
    return result;
}

// call 实现
Function.prototype.myCall = function(context){ 
    var args = [];
    for (var i = 1; i < arguments.length; i++) {
        args.push(arguments[i]);
    }
    return this.myApply(context, args);
};

// bind 实现
Function.prototype.myBind = function(context){ 
    var self = this;
    var arg1 = Array.prototype.slice.call(arguments, 1);
    return function innerFun(){ 
        var arg2 = Array.prototype.slice.call(arguments);
        return self.apply(
            this instanceof innerFun ? this : context, 
            arg1.concat(arg2)
        );
    }
};
```

### 7.3 用途

1. **改变 this 指向**
2. **借用其他对象的方法**：`Array.prototype.push.call(arrayLike, 'item')`

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
            .map(function(key) {return encodeURIComponent(key) + '=' + encodeURIComponent(obj[key])})
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


## let/const/var 的区别

| 特性 | var | let | const |
|------|-----|-----|-------|
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 是（值为 undefined） | 否（暂时性死区 TDZ） | 否（暂时性死区 TDZ） |
| 重复声明 | 允许 | 不允许 | 不允许 |
| 重新赋值 | 允许 | 允许 | 不允许（对象属性可修改） |
| 全局声明挂载 window | 是 | 否 | 否 |

```javascript
// 暂时性死区（TDZ）
console.log(a); // undefined（变量提升）
var a = 1;

console.log(b); // ReferenceError（暂时性死区）
let b = 2;

// const 引用类型
const obj = { name: 'test' };
obj.name = 'changed'; // 允许
obj = {}; // TypeError
```

---

## == 和 === 的区别

**`===` 严格相等**：类型和值都相等才返回 true

**`==` 宽松相等**：会进行类型转换后比较

**类型转换规则**：
1. `null == undefined` 为 true
2. 数字和字符串比较，字符串转数字
3. 布尔值转数字（true → 1, false → 0）
4. 对象与原始值比较，对象调用 valueOf/toString

```javascript
// 经典面试题
[] == false    // true  [] → '' → 0, false → 0
[] == ![]      // true  ![] → false → 0, [] → 0
{} == !{}      // false {} → NaN, !{} → false → 0
null == undefined  // true
null === undefined // false
NaN == NaN     // false
```

**建议**：始终使用 `===` 进行比较

---

## null 和 undefined 的区别

| 特性 | null | undefined |
|------|------|-----------|
| 含义 | 空值，表示"无"对象 | 未定义，表示缺少值 |
| typeof | "object"（历史遗留 bug） | "undefined" |
| 转数字 | 0 | NaN |
| 场景 | 主动赋值表示空 | 变量声明未赋值 |

```javascript
// undefined 产生场景
let a;              // 声明未赋值
function fn(b) {}   // 参数未传递
fn();               // b 为 undefined
const obj = {};
obj.name;           // 访问不存在的属性
function f() {}     // 函数无返回值
f();                // 返回 undefined
```

---

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

## 数组常用方法

### 创建与转换
```javascript
// Array.from：类数组/可迭代对象转数组
Array.from('abc');           // ['a', 'b', 'c']
Array.from({ length: 3 });   // [undefined, undefined, undefined]
Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]

// Array.of：创建数组（解决 new Array 的歧义）
Array.of(3);        // [3]
Array.of(1, 2, 3);  // [1, 2, 3]
new Array(3);       // [empty × 3]

// Array.isArray：判断是否为数组
Array.isArray([]);  // true
Array.isArray({});  // false

// 展开运算符
const copy = [...arr];
const merged = [...arr1, ...arr2];
```

### 增删方法（改变原数组）
```javascript
const arr = [1, 2, 3];

// push：末尾添加，返回新长度
arr.push(4);        // 4, arr: [1, 2, 3, 4]

// pop：末尾删除，返回删除元素
arr.pop();          // 4, arr: [1, 2, 3]

// unshift：开头添加，返回新长度
arr.unshift(0);     // 4, arr: [0, 1, 2, 3]

// shift：开头删除，返回删除元素
arr.shift();        // 0, arr: [1, 2, 3]

// splice(start, deleteCount, ...items)：删除/替换/插入
arr.splice(1, 1);         // [2], arr: [1, 3]（删除）
arr.splice(1, 0, 2);      // [], arr: [1, 2, 3]（插入）
arr.splice(1, 1, 'a');    // [2], arr: [1, 'a', 3]（替换）

// fill：填充数组
[1, 2, 3].fill(0);        // [0, 0, 0]
[1, 2, 3].fill(0, 1, 2);  // [1, 0, 3]

// copyWithin：内部复制
[1, 2, 3, 4].copyWithin(0, 2); // [3, 4, 3, 4]

// reverse：反转数组
[1, 2, 3].reverse();      // [3, 2, 1]

// sort：排序（默认按字符串排序）
[3, 1, 2].sort();                    // [1, 2, 3]
[3, 1, 2].sort((a, b) => b - a);     // [3, 2, 1]（降序）
```

### 截取方法（不改变原数组）
```javascript
const arr = [1, 2, 3, 4, 5];

// slice(start, end)：截取，支持负数
arr.slice(1, 3);   // [2, 3]
arr.slice(-2);     // [4, 5]
arr.slice();       // [1, 2, 3, 4, 5]（浅拷贝）

// concat：合并数组
arr.concat([6, 7]); // [1, 2, 3, 4, 5, 6, 7]
```

### 查找方法
```javascript
const arr = [1, 2, 3, 2, 1];

// indexOf/lastIndexOf：返回索引，找不到返回 -1
arr.indexOf(2);      // 1
arr.lastIndexOf(2);  // 3

// includes：是否包含（ES7）
arr.includes(2);     // true
arr.includes(2, 2);  // true（从索引 2 开始查找）

// find：返回第一个满足条件的元素
arr.find(x => x > 1);      // 2

// findIndex：返回第一个满足条件的索引
arr.findIndex(x => x > 1); // 1

// findLast/findLastIndex：从后往前查找（ES2023）
arr.findLast(x => x > 1);      // 2
arr.findLastIndex(x => x > 1); // 3

// at：按索引取值，支持负数（ES2022）
arr.at(0);   // 1
arr.at(-1);  // 1
```

### 遍历方法
```javascript
const arr = [1, 2, 3];

// forEach：遍历，无返回值，不能 break
arr.forEach((item, index, array) => {
  console.log(item, index);
});

// map：遍历，返回新数组
const doubled = arr.map(x => x * 2); // [2, 4, 6]

// filter：过滤，返回符合条件的新数组
const evens = arr.filter(x => x % 2 === 0); // [2]

// find：返回第一个符合条件的元素
const first = arr.find(x => x > 1); // 2

// some：有一个满足就返回 true
arr.some(x => x > 2);  // true

// every：全部满足才返回 true
arr.every(x => x > 0); // true

// reduce：累加器，从左到右
arr.reduce((acc, cur) => acc + cur, 0); // 6

// reduceRight：累加器，从右到左
[[1], [2], [3]].reduceRight((acc, cur) => acc.concat(cur), []); // [3, 2, 1]
```

### 扁平化方法
```javascript
const arr = [1, [2, [3, [4]]]];

// flat：扁平化，参数为深度（默认 1）
arr.flat();      // [1, 2, [3, [4]]]
arr.flat(2);     // [1, 2, 3, [4]]
arr.flat(Infinity); // [1, 2, 3, 4]（完全扁平化）

// flatMap：map + flat(1)
[1, 2].flatMap(x => [x, x * 2]); // [1, 2, 2, 4]
```

### 其他方法
```javascript
// join：数组转字符串
[1, 2, 3].join('-'); // '1-2-3'

// toString：数组转字符串
[1, 2, 3].toString(); // '1,2,3'

// keys/values/entries：返回迭代器
[...['a', 'b'].keys()];    // [0, 1]
[...['a', 'b'].values()];  // ['a', 'b']
[...['a', 'b'].entries()]; // [[0, 'a'], [1, 'b']]

// toReversed/toSorted/toSpliced：不改变原数组的版本（ES2023）
const arr = [3, 1, 2];
arr.toReversed();         // [2, 1, 3]，arr 不变
arr.toSorted();           // [1, 2, 3]，arr 不变
arr.toSpliced(1, 1, 'a'); // [3, 'a', 2]，arr 不变
arr.with(1, 'a');         // [3, 'a', 2]，arr 不变
```

### 常用技巧
```javascript
// 数组去重
const unique = [...new Set(arr)];
const unique = arr.filter((item, index) => arr.indexOf(item) === index);

// 数组求和
const sum = arr.reduce((acc, cur) => acc + cur, 0);

// 数组最大/最小值
const max = Math.max(...arr);
const min = Math.min(...arr);

// 数组乱序
arr.sort(() => Math.random() - 0.5);

// 数组分组（ES2023 Object.groupBy）
const grouped = Object.groupBy(users, user => user.age > 18 ? 'adult' : 'minor');

// 生成 0-n 数组
const arr = Array.from({ length: n }, (_, i) => i);
const arr = [...Array(n).keys()];

// 清空数组
arr.length = 0;

// 删除假值
arr.filter(Boolean); // 删除 false, 0, '', null, undefined, NaN
```

---

## 字符串常用方法

### 查找类方法
```javascript
const str = 'Hello World';

// indexOf/lastIndexOf：返回索引，找不到返回 -1
str.indexOf('o');      // 4
str.lastIndexOf('o');  // 7

// includes：是否包含（ES6）
str.includes('World'); // true

// startsWith/endsWith：是否以指定字符串开头/结尾（ES6）
str.startsWith('Hello'); // true
str.endsWith('World');   // true

// search：支持正则，返回索引
str.search(/world/i);  // 6

// match：匹配正则，返回数组
str.match(/o/g);       // ['o', 'o']

// at：按索引取字符，支持负数（ES2022）
str.at(0);   // 'H'
str.at(-1);  // 'd'
```

### 截取类方法
```javascript
const str = 'Hello World';

// slice(start, end)：支持负数，不改变原字符串
str.slice(0, 5);   // 'Hello'
str.slice(-5);     // 'World'

// substring(start, end)：不支持负数，会自动交换参数
str.substring(0, 5); // 'Hello'
str.substring(5, 0); // 'Hello'（自动交换）

// substr(start, length)：已废弃，不推荐使用
str.substr(0, 5);  // 'Hello'
```

### 转换类方法
```javascript
// split：字符串转数组
'a,b,c'.split(',');  // ['a', 'b', 'c']
'hello'.split('');   // ['h', 'e', 'l', 'l', 'o']

// toUpperCase/toLowerCase
'hello'.toUpperCase(); // 'HELLO'
'HELLO'.toLowerCase(); // 'hello'

// trim/trimStart/trimEnd：去除空格
'  hello  '.trim();      // 'hello'
'  hello  '.trimStart(); // 'hello  '
'  hello  '.trimEnd();   // '  hello'

// replace/replaceAll
'hello'.replace('l', 'L');    // 'heLlo'（只替换第一个）
'hello'.replaceAll('l', 'L'); // 'heLLo'（替换全部，ES2021）
'hello'.replace(/l/g, 'L');   // 'heLLo'（正则全局替换）

// concat：拼接字符串（推荐用 + 或模板字符串）
'hello'.concat(' ', 'world'); // 'hello world'
```

### 填充与重复
```javascript
// padStart/padEnd：填充到指定长度（ES2017）
'5'.padStart(2, '0');  // '05'
'5'.padEnd(3, '0');    // '500'
'123'.padStart(5);     // '  123'（默认用空格填充）

// repeat：重复
'ab'.repeat(3);        // 'ababab'
```

### 常用技巧
```javascript
// 字符串反转
'hello'.split('').reverse().join(''); // 'olleh'

// 首字母大写
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);

// 统计字符出现次数
const count = (str, char) => str.split(char).length - 1;

// 驼峰转换
'hello-world'.replace(/-([a-z])/g, (_, c) => c.toUpperCase()); // 'helloWorld'

// 模板字符串
const name = 'World';
`Hello ${name}!`; // 'Hello World!'
```

---

## 对象常用方法

### 创建与复制
```javascript
// Object.create：以指定原型创建对象
const obj = Object.create(proto);
const pureObj = Object.create(null); // 无原型的纯净对象

// Object.assign：浅拷贝合并对象（后面覆盖前面）
const merged = Object.assign({}, obj1, obj2);

// 展开运算符：浅拷贝（ES2018）
const copy = { ...obj };
const merged = { ...obj1, ...obj2 };
```

### 遍历方法
```javascript
const obj = { a: 1, b: 2, c: 3 };

// Object.keys：返回可枚举属性名数组
Object.keys(obj);    // ['a', 'b', 'c']

// Object.values：返回属性值数组（ES2017）
Object.values(obj);  // [1, 2, 3]

// Object.entries：返回键值对数组（ES2017）
Object.entries(obj); // [['a', 1], ['b', 2], ['c', 3]]

// Object.fromEntries：键值对数组转对象（ES2019）
Object.fromEntries([['a', 1], ['b', 2]]); // { a: 1, b: 2 }

// for...in：遍历可枚举属性（包括原型链）
for (let key in obj) {
  if (obj.hasOwnProperty(key)) {
    console.log(key, obj[key]);
  }
}
```

### 属性描述符
```javascript
// Object.defineProperty：定义单个属性
Object.defineProperty(obj, 'name', {
  value: 'test',
  writable: false,      // 是否可修改
  enumerable: true,     // 是否可枚举
  configurable: false   // 是否可删除/重新配置
});

// Object.defineProperties：定义多个属性
Object.defineProperties(obj, {
  name: { value: 'test', writable: false },
  age: { value: 18, enumerable: true }
});

// Object.getOwnPropertyDescriptor：获取属性描述符
Object.getOwnPropertyDescriptor(obj, 'name');

// Object.getOwnPropertyNames：获取所有自有属性名（包括不可枚举）
Object.getOwnPropertyNames(obj);
```

### 对象限制
```javascript
// Object.freeze：冻结对象（不能增删改）- 浅冻结
const frozen = Object.freeze({ a: 1 });
frozen.a = 2;     // 静默失败，严格模式报错
frozen.b = 3;     // 静默失败

// Object.seal：密封对象（不能增删，可改）
const sealed = Object.seal({ a: 1 });
sealed.a = 2;     // 可以修改
sealed.b = 3;     // 静默失败

// Object.preventExtensions：禁止扩展（不能增，可删改）
const obj = Object.preventExtensions({ a: 1 });

// 检测方法
Object.isFrozen(frozen);           // true
Object.isSealed(sealed);           // true
Object.isExtensible(obj);          // false
```

### 原型相关
```javascript
// Object.getPrototypeOf：获取原型
Object.getPrototypeOf(obj);

// Object.setPrototypeOf：设置原型（性能差，不推荐）
Object.setPrototypeOf(obj, proto);

// hasOwnProperty：是否自有属性
obj.hasOwnProperty('name');

// Object.hasOwn：ES2022 新增，推荐使用
Object.hasOwn(obj, 'name');

// isPrototypeOf：检查原型链
Array.prototype.isPrototypeOf([]); // true
```

### 常用技巧
```javascript
// 判断空对象
Object.keys(obj).length === 0;

// 对象解构与默认值
const { name = 'default', age } = obj;

// 动态属性名
const key = 'name';
const obj = { [key]: 'value' };

// 可选链（ES2020）
const value = obj?.nested?.property;

// 空值合并（ES2020）
const value = obj.name ?? 'default';

// 对象转 Map
const map = new Map(Object.entries(obj));

// Map 转对象
const obj = Object.fromEntries(map);

// 过滤对象属性
const filtered = Object.fromEntries(
  Object.entries(obj).filter(([key, val]) => val > 0)
);
```

---

## Promise 静态方法区别

| 方法 | 说明 | 返回时机 |
|------|------|----------|
| `Promise.all` | 全部成功才成功 | 任一失败立即失败 |
| `Promise.allSettled` | 等待全部完成 | 无论成功失败都返回结果 |
| `Promise.race` | 返回最先完成的 | 第一个完成（无论成功失败） |
| `Promise.any` | 返回最先成功的 | 第一个成功，全失败才失败 |

```javascript
// all：全部成功
Promise.all([p1, p2, p3]).then(results => {});

// allSettled：获取所有结果
Promise.allSettled([p1, p2]).then(results => {
  // [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]
});

// race：竞速
Promise.race([p1, p2]).then(first => {});

// any：任一成功（ES2021）
Promise.any([p1, p2]).then(first => {}).catch(errors => {});
```

---

## async/await 原理

**本质**：Generator + 自动执行器的语法糖

```javascript
// async 函数返回 Promise
async function fn() {
  return 1;
}
fn(); // Promise {fulfilled: 1}

// await 等待 Promise 完成
async function getData() {
  try {
    const res = await fetch('/api');
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}

// 并行执行
async function parallel() {
  const [res1, res2] = await Promise.all([fetch(url1), fetch(url2)]);
}
```

**注意事项**：
- await 只能在 async 函数中使用（ES2022 支持顶层 await）
- await 后面的代码相当于放在 .then() 中执行
- 多个无依赖的 await 应该用 Promise.all 并行

---

