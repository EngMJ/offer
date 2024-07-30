> 闭包是指有权访问另一个函数作用域中的变量的函数。

闭包简单理解就是`内嵌函数`，内部函数使用外部函数的变量, 通过输入或返回函数实现。

*由于在JS中，变量的作用域属于函数作用域，在函数执行后作用域就会被清理、内存也随之回收，但是由于闭包是建立在一个函数内部的子函数，由于其可访问上级作用域的原因，即使上级函数执行完，作用域也不会随之销毁，这时的子函数——也就是闭包，便拥有了访问上级作用域中的变量的权限，即使上级函数执行完后作用域内的值也不会被销毁。*

闭包的`形成`、变量的`作用域`以及变量的`生存周期`密切相关。

## 1\. 变量的作用域

变量的作用域，就是指变量的有效范围。

随着代码执行环境创建的作用域链往外层逐层搜索，一直搜索到全局对象为止。

## 2\. 变量的生存周期

+   对于全局变量来说生存周期是永久的，除非主动销毁。
+   一般对于函数作用域或者局部作用域（let const），会随着函数调用的结束而被销毁。
+   当函数内部调用外部变量就产生了`闭包`，这时候变量的回收策略可以参考`引用计数`等`垃圾回收策略`。

## 3\. 内存泄漏

由于IE的js对象和DOM对象使用不同的垃圾收集方法，因此闭包在IE中会导致内存泄露问题，也就是无法销毁在内存中的dom元素。

因此我们在不需要使用dom时，应将dom解除引用来避免内存泄露

```js
function closure(){
    // div用完之后一直存在内存中，无法被回收
    var div = document.getElementById('div');
    div.onclick = function () {
       console.log(div.innerHTML);// 这里用oDiv导致内存泄露
    };
}

// 解除引用避免内存泄露
function closure(){    
    var div = document.getElementById('div');    
    var test = div.innerHTML;
    div.onclick = function () {
        console.log(test);
    };
    div = null;
}
```

## 4\. 闭包总结

+   **形成：** 函数中嵌套函数
+   **作用：** 函数内部调用外部变量、构造函数的私有属性、延长变量生命周期
+   **优点：** 希望一个变量长期存在内存中、模块化代码避免全局变量的污染、私有属性
+   **缺点：** 无法回收闭包中引用变量，容易造成内存泄漏

## 5\. 闭包的应用

闭包可以使代码组织方式的自由度大大提升，在日常使用中有非常广泛的用途。

**简单的有：**

+   ajax请求的成功回调
+   事件绑定的回调方法
+   setTimeout的延时回调
+   函数内部返回另一个匿名函数

我们详细讲下以下几个应用场景：

+   构造函数的私有属性
+   计算缓存
+   函数节流、防抖

### 5.1 构造函数的私有属性

+   由于javascript中天然没有类的实现，某些不希望被外部修改的`私有属性`可以通过闭包的方式实现。

```js
function Person(param) {
    var name = param.name; // 私有属性
    this.age = 18; // 共有属性

    this.sayName = function () {
        console.log(name);
    }
}

const tom = new Person({name: 'tom'});
tom.age += 1; // 共有属性，外部可以更改
tom.sayName(); // tom
tom.name = 'jerry';// 共有属性，外部不可更改
tom.sayName(); // tom
```

### 5.2 计算缓存

```js
// 平方计算
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

### 5.3 函数节流、防抖

```js
// 节流
function throttle(fn, delay) {
    var timer = null, firstTime = true;
    return function () {
        if (timer) { return false;}
        var that = this;
        var args = arguments;
        fn.apply(that, args);
        timer = setTimeout(function () {
            clearTimeout(timer);
            timer = null;
        }, delay || 500);
    };
}
// 防抖
function debounce(fn, delay) {
    var timer = null;
    return function () {
        var that = this;
        var args = arguments;
        clearTimeout(timer);// 清除重新计时
        timer = setTimeout(function () {
            fn.apply(that, args);
        }, delay || 500);
    };
}
```

## 6\. 闭包高频面试题（代码理解）

```js
// 1
for ( var i = 0 ; i < 5; i++ ) {
    setTimeout(function(){
        console.log(i);
    }, 0);
}
// 5 5 5 5 5


// 2
for ( var i = 0 ; i < 5; i++ ) {
    (function(j){
        setTimeout(function(){
            console.log(j);
        }, 0);
    })(i);
    // 这样更简洁
    // setTimeout(function(j) {
    //     console.log(j);
    // }, 0, i);
}
// 0 1 2 3 4

setTimeout(function(j) {
    console.log(j);
}, 0, i);

// 3
for ( let i = 0 ; i < 5; i++ ) {
    setTimeout(function(){
        console.log(i);
    },0);
}
// 0 1 2 3 4


// 4
var scope = 'global scope';
function checkscope(){
    var scope = 'local scope';
    console.log(scope);
}
// local scope


// 5
var scope = 'global scope';
function checkscope(){
    var scope = 'local scope';
    return function f(){
        console.log(scope);
    };
}
var fn = checkscope(); 
console.log(fn()); // local scope


// 6
var scope = 'global scope';
function checkscope(){
    var scope = 'local scope';
    return function f(){
        console.log(scope);
    };
}
checkscope()(); // local scope


var obj = {
    name: 'tom',
    sayName() {
        console.log(this.name);
    }
}
obj.sayName(); // tom


var obj = {
    name: 'tom',
    sayName() {
        var name = 'alan';
        console.log(this.name);
    }
}
obj.sayName();// 'tom'

var name = 'jerry';   
var obj = {  
    name : 'tom',  
    sayName(){  
        return function(){  
            console.log(this.name);  
        };
    }   
};  
obj.sayName()(); // jerry

// var name = 'jerry';
var obj = {
    name: 'tom',
    sayName() {
        var name = 'alan';
        console.log(this.name);
    }
};
var sayName = obj.sayName;
sayName(); // '' // jerry




function fun(a,b) {
    console.log(b)
    return {
        fun: function(c) {
            return fun(c,a);
        }
    };
}
var d = fun(0); // undefined
d.fun(1); // 2 0
d.fun(2); // 2 0
d.fun(3); // 2 0

var d1 = fun(0).fun(1).fun(2).fun(3);
// 2 undefined 2 0
// 2 1
// 2 2

var d2 = fun(0).fun(1); d2.fun(2);
// 2 undefined
// 2 0
// 2 1
// 2 1

d2.fun(3); // {fun: ƒ}

```
