当一个函数被调用时，会创建一个`活动记录`(有时候也称为`执行上下文`)。这个记录会包含函数在哪里被调用(调用栈)、函数的调用方法、传入的参数等信息。`this` 就是记录(上下文)的其中一个属性，会在函数执行的过程中用到。

## 一、this的指向

this 总是指向`执行时`的`当前对象`。

JavaScript 的 this 总是指向一个对象，而具体指向哪个对象是在运行时基于**函数的执行环境动态绑定的，而非函数被声明时的环境**。

也就是说 this 的绑定和函数声明的位置没有任何关系，只**取决于函数的调用方式**。

除了使用 with 和 eval 的情况，常用的可分为以下几种：

+   作为对象的方法调用。
+   作为普通函数调用。
+   构造器调用。
+   Function.prototype.call 或 Function.prototype.apply 调用。

### 1\. 作为对象的方法调用

对象的方法被调用时，this 指向该对象。

```js
var obj = {
    a: 1,
    getA() {
        alert ( this === obj ); // 输出:true alert ( this.a ); // 输出: 1
    }
};
obj.getA();
```

### 2\. 作为普通函数调用

当函数不作为对象的属性被调用时，也就是我们常说的普通函数方式，此时的 this 总是指向全局对象。

这里注意对象的方法被**单独拷贝出来后执行**，那么原来的this会`丢失`，变成指向全局对象

```js
//在浏览器的JavaScript 里，这个全局对象是window 对象。
window.name = 'globalName';
var getName = function(){
    return this.name;
};
console.log( getName() ); // 输出：globalName

//或者：
window.name = 'globalName';
var myObject = {
    name: 'sven',
    getName: function(){
        return this.name;
    }
};
var getName = myObject.getName;
console.log( getName() ); // globalName
```

### 3\. 构造器调用

通常情况下，构造器里的 this 就指向返回的这个对象;

+   如果构造器不显式地返回任何数据，或者是返回一个非对象类型的数据，this指向返回的这个对象；
+   如果构造器显式地返回了一个对象，则实例化的结果会返回这个对象，而不是this;

```js
//构造器里的this 就指向返回的这个对象，见如下代码：
var MyClass = function(){
    this.name = 'sven';
};
var obj = new MyClass();
alert ( obj.name ); // 输出：sven


var MyClass = function(){
    this.name = 'sven';
    return { // 显式地返回一个对象
        name: 'anne'
    }
};
var obj = new MyClass();
alert ( obj.name ); // 输出：anne


var MyClass = function(){
    this.name = 'sven'
    return 'anne'; // 返回string 类型
};
var obj = new MyClass();
alert ( obj.name ); // 输出：sven
```

### 4\. call 或 apply 调用

apply和call可以动态地改变传入函数的 this

```js
var obj1 = {
    name: 'sven',
    getName: function(){
        return this.name;
    }
};
var obj2 = {
    name: 'anne'
};
console.log( obj1.getName() ); // 输出: sven
console.log( obj1.getName.call( obj2 ) ); // 输出：anne
```

### 5\. 箭头函数中调用

在箭头函数声明的时候,就绑定对应的this(并且作用域提升), 后续在任何地方调用this都一样.
