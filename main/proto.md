> **原型定义:** 给其它对象提供共享属性的对象，简称为原型( prototype )。

### 带着问题看原型

+   是什么（定义）？
+   为什么存在（作用）？
+   应用？
+   优缺点（注意事项）？
+   使用方法？

## 一、原型

### 1\. 原型是什么

首先给出**定义**：给其它对象提供共享属性的对象，简称为原型( prototype )。

所有对象，都有一个隐式引用，它被称之为这个对象的 `prototype` 原型。当访问对对象的某个属性时，会先在对象本身的属性上寻找，没找到就会去他的原型对象上寻找。

#### `__proto__` & `constructor` 概念简介

+   ****proto****：隐式原型，js标准里对象的原型无法直接通过属性名访问，但是多数浏览实现了`__proto__`(注意前后都有两个下划线)属性用来访问对象的原型。
+   **constructor**：构造器，创建一个函数时会为它增加一个 prototype 属性，指向原型对象，原型对象自动获得一个名为 constructor 的属性，指回与之关联的构造函数。

### 2\. 原型的作用

原型在Javascript中是一个非常重要的概念，那么它有什么作用呢？

作用在定义中已经提到了：`共享属性`

如果在一个对象上找不到某个属性，就会去它的原型对象上找，以此类推直至找到，或者寻找到原型链的终点都没找到则不存在这个属性；这个特性让任意创建的对象都拥有通用的各种方法，例如：toString、hasOwnProperty、isPrototypeOf等

### 3\. 原型的应用

共享属性的特性可以用来做很多巧妙的事情。

#### 3.1 属性委托

普通对象上的默认方法都是来自于 Object.prototype 上的方法

```js
const obj = {};
obj.toString();
```

#### 3.2 面向对象（类）封装

Javscript本身没有**类**，es6中的类也不过是`寄生组合式`封装的语法糖。

```js
function Person() {}
Person.prototype.name = 'tom';
Person.prototype.sayName = function() {
    console.log(this.name);
};
const person1 = new Person();
person1.sayName(); // "tom"
```

#### 3.3 继承

子类通过原型实现对父类属性和方法的继承，本质上是在子类实例上找不到的属性方法在子类的原型（父类的实例或原型）上找到了。

Javscript中的继承**不是复制**，而是`委托`。委托行为意味着某些对象(子类)在找不到属性或者方法引用时会把这个请求委托给另一个对象(父类)。

```js
function SuperType(){
    this.name = 'tom';
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(){}
// 原型继承了 SuperType
SubType.prototype = new SuperType();

const instance = new SubType();
instance.sayName()); // 'tom'
```

关于Javascript继承的详细介绍请看笔者的另一篇文章：[JavaScript面向对象&继承](http://alanyf.site/blog/#/article/5ff52e743b420b7fc54e7de9 "http://alanyf.site/blog/#/article/5ff52e743b420b7fc54e7de9")

### 4\. 优缺点

#### 优点

+   **节约空间**：Object.prototype上的众多方法（toString、hasOwnProperty、isPrototypeOf等）都不需要重新复制，通过原型链就可以找到并使用。
+   **灵活度高**：自由度大

#### 缺点

+   **难理解**：别的语言没有，原型的原理细究起来也较为复杂。
+   **易被覆盖和修改**：引用类型的原型属性会被所有实例共享，所以修改也会被共享。

### 5\. 注意事项

属性、方法的覆盖和屏蔽会导致意外的问题出现。

#### 覆盖

```js
Object.prototype.hasOwnProperty = function () {
    alert('Ooops, 方法被覆盖了');
    return false;
}

const obj = {name: 'tom'};
console.log(obj.hasOwnProperty('name')); // 原型上的方法被屏蔽
```

#### 屏蔽

由于查找对象属性的顺序是先原对象后原型，所以当对象本身存在和原型上的同名属性时，就会发生屏蔽。

```js
function A() {
    this.name = 'aaa';
}
A.prototype.name = 'a';

const instance = new A();
console.log(instance.name); // 'aaa'原型上的属性被屏蔽
```

```js
function A() {
    this.name = 'a';
    this.sayName = '';
}
A.prototype.sayName = function () {
    console.log(this.name);
};

const instance = new A();
instance.sayName(); // 'aaa'原型上的方法被屏蔽
```

## 二、原型链

### 什么是原型链

**定义**：如果要访问对象中并不存在的一个属性，就会查找对象内部 prototype 关联的对象。在查找属性时会对它进行层层寻找遍历，这个关联关系实际上定义了一条**原型链**。

原型链最终会指向`Object.prototype`，原型链的终点是`Object.prototype.__proto__` 也就是 `null`。

### 原型链示意图

+   **继承中的原型链示意图：**

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/911b3e448e5248ec96d973b076e012c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 三、使用

### 原型属性判断&设置

#### 1\. getPrototypeOf

Object.getPrototypeOf(obj) 间接**访问**指定对象的 prototype 对象。

版本：es6+

#### 2\. setPrototypeOf

Object.setPrototypeOf(obj, obj1) 间接**设置**指定对象的 prototype 对象 版本：es6+

#### 3\. hasOwnProperty

是否仅属于对象本身的属性

### 枚举方法

#### 1\. getOwnPropertyNames

返回所有**实例本身**的属性。 版本：es6+

#### 2\. in

只要通过对象可以访问，in 操作符就返回 true，也就是说在**对象**或者**原型链**上存在，就会返回true

#### 3\. for...in

可以通过对象访问且可以被枚举的属性都会返回，包括`实例属性`和`原型属性`。遮蔽原型中不可枚举(\[\[Enumerable\]\]特性被设置为 false)属性的实例属性也会在 for-in 循环中返回，因为默认情况下开发者定义的属性都是可枚举的。

#### 4\. Object.keys

对象上所有可枚举的实例本身属性。

#### 5\. Object.values

返回对象值的数组 版本：es8+

```js
const obj = {a: 1, b: '2'};
Object.values(obj); // [1, '2']
```

#### 6\. Object.entries

返回键/值对的数组 版本：es8+

```js
const obj = {a: 1, b: '2'};
Object.entries(obj); // [['a', 1], ['b', '2']]
```

#### 7\. 多种枚举方式的区别

1.  **枚举顺序**

+   for-in 循环和 Object.keys() 的枚举顺序是不确定的，取决于 JavaScript 引擎，可能因浏览器而异。
+   Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()和 Object.assign() 的枚举顺序是确定性的。先以**升序枚举数值键**，然后以**插入顺序枚举字符串和符号键**。在对象字面量中定义的键以它们逗号分隔的顺序插入。

2.  **枚举范围**

+   getOwnPropertyNames、Object.keys、Object.values、Object.entries范围仅是**实例本身**的属性/值
+   in、for...in、Object.entries范围仅是**实例本身**的属性/值
