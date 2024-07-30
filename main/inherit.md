> 主要介绍JavaScript几种经典继承模式原理，以及他们的优缺点。

## 一. 原型链

**原理：** 使用父类的示例重写子类的原型。

**实现：**

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
//继承了 SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
    return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue()); //true
```

**缺点：**

1.  父类包含引用类型值的原型属性会被所有子类共享；
2.  在创建子类的实例时，不能向父类的构造函数中传递参数。

## 二. 借用构造函数

**原理：** 在子类构造函数的内部调用父类的构造函数，将子类的执行环境 this 用 call 方法传到父类的构造函数中，使父类中的属性和方法被重写到子类上。

**实现：**

```js
function SuperType(){
    this.colors = ["red", "blue", "green"];
}
function SubType(){
    // 继承了 SuperType
    SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
var instance2 = new SubType();
alert(instance2.colors); //"red,blue,green"
```

**优点：**

1.  属性和方法不会被子类所共享；

2.  可以向父类的构造函数中传递参数。


**缺点：**

父类中的方法无法被复用，每个方法都要在子类上重新创建。

## 三. 组合式继承

原理：使用原型链实现对原型属性和方法的继承，借用构造函数实现对实例属性的继承。

**实现：**

```js
function SuperType(name){
this.name = name;
this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name, age){
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};
var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29
var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

**优点：**

结合原型链和借用构造函数继承两种方法，取长补短。实现了函数复用，又能够保证每个子类不会共享父类的引用类型属性。

**缺点：**

调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。

```js
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name, age){
    SuperType.call(this, name); // 第二次调用 SuperType()
    this.age = age;
}
SubType.prototype = new SuperType(); // 第一次调用 SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};
```

## 四. 寄生组合式继承

**原理：** 通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。就是使用寄生式继承来继承父类的原型，然后再将结果指定给子类的原型。

**实现：**

```js
//原型式继承，相当于 Object.create()
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}
 
//寄生式继承
function inheritPrototype(subType, superType){
    var prototype = object(superType.prototype); //创建对象
    prototype.constructor = subType; //增强对象
    subType.prototype = prototype; //指定对象
}
 
//寄生组合式继承
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
    alert(this.age);
};
```

**优点：**

改进组合模式中父类构造函数被调用两次这个缺陷。寄生组合式继承是引用类型最理想的继承模式。

## 五、class

在 es6 中，官方给出了 `class` 关键字来实现面向对象风格的写法，但本质上是`寄生组合式继承`的语法糖。

```js
class Person {
    constructor(age) {
        this.age_ = age;
    }
    sayAge() {
        console.log(this.age_);
    }
    // 静态方法
    static create() {
        // 使用随机年龄创建并返回一个 Person 实例
        return new Person(Math.floor(Math.random()*100));
    }
}

// 继承普通构造函数
class Doctor extends Person {}

const doctor = new Doctor(32);
doctor.sayAge(); // 32
```

## 继承中的原型链示意图

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/911b3e448e5248ec96d973b076e012c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 参考

+   [JavaScript高级程序设计（第四版）](https://book.douban.com/subject/35175321/?from=tag "https://book.douban.com/subject/35175321/?from=tag")
