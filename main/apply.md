### 简介及区别

call 和 apply作用一模一样，区别仅在于传入参数形式的不同。

#### apply

`apply` 接受两个参数，第一个参数指定了函数体内 this 对象的指向，第二个参数为一个带下标的集合，这个集合可以为数组，也可以为`类数组`。

类数组：

1.  对象本身要可以存取属性;
2.  对象的 length 属性可读写。

#### call

`call` 传入的参数数量不固定，第一个参数也是代表函数体内的 this 指向，从第二个参数开始往后，每个参数被依次传入函数

#### bind

`bind` 参数类型和 call 相同，不过它不会执行函数，而是修改 this 后返回一个新的函数

```js
var func = function( a, b, c ){
    alert ( [ a, b, c ] );
};
func.apply( null, [ 1, 2, 3 ] );
func.call( null, 1, 2, 3 );
```

动手实现`apply`函数

```js
Function.prototype.apply = function (context) {
    var context = context || window;
    var args = arguments[1] || [];
    context.fn = this;
    var result = context.fn(...args);
    delete context.fn;
    return result;
}
```

`call` 和 `bind` 是包装在 `apply` 上面的`语法糖`：

```js
Function.prototype.call = function( context ){ 
    var argus = [];
    for (var i = 1; i < arguments.length; i++) {
        argus.push(arguments[i]);
    }
    this.apply(context, argus);
};

Function.prototype.bind = function( context ){ 
    var self = this; // 保存原函数
    var arg1 = []
    for (var i = 1; i < arguments.length; i++) {
        arg1.push(arguments[i]);
    }
    // 返回一个新的函数
    return function innerFun(){ 
        var arg2 = []
        for (var n = 1; n < arguments.length; n++) {
            arg2.push(arguments[n]);
        }
        // bind函数是能new操作的,所以查询下原型,继承自身则使用this
        return self.apply( this instanceof innerFun ? this : context, arg1.concat(arg2) );
    }
};
```

### 用途

1.  **改变 this 指向**
2.  **借用其他对象的方法：** 在`操作类数组`（比如 arguments） 的时候，可以找 Array.prototype 对象借用方法`Array.prototype.push.call(a, 'first' )`
