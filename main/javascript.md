
## JavaScript 面向对象&继承

+   参考：[JavaScript面向对象&继承](https://juejin.cn/post/6915190212016472077 "https://juejin.cn/post/6915190212016472077")

## 原型和原型链

+   参考：[通俗易懂的理解Javacsript原型和原型链](https://juejin.cn/post/6914827188025950216 "https://juejin.cn/post/6914827188025950216")

## 闭包及练习题

+   参考：[Javascript闭包详解](https://juejin.cn/post/6911473286627098638 "https://juejin.cn/post/6911473286627098638")

## 函数节流和防抖

+   参考：[函数节流和防抖](https://juejin.cn/post/6911118171600584718 "https://juejin.cn/post/6911118171600584718")

## Javascript 中的this

+   参考：[Javascript 中的this（包含apply、call、bind）](https://juejin.cn/post/6910015962058063885 "https://juejin.cn/post/6910015962058063885")

## 动手实现apply、call、bind

+   参考：[动手实现apply、call、bind](https://juejin.cn/post/6910015962058063885#heading-5 "https://juejin.cn/post/6910015962058063885#heading-5")

## Javascript 小技巧帮你提升代码质量

+   参考：[12个 Javascript 小技巧帮你提升代码质量](https://juejin.cn/post/6909638377247604750 "https://juejin.cn/post/6909638377247604750")

## 自己动手实现Promise

+   参考：[自己动手实现Promise](https://juejin.cn/post/6885295302933217294 "https://juejin.cn/post/6885295302933217294")

## 作用域&作用域链

+   简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。作用域是一套规则，在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。
+   作用域链的作用是保证执行环境里有权访问的变量和函数是有序的，作用域链的变量只能向上访问，变量访问到window对象即被终止，作用域链向下访问变量是不被允许的。

## new操作符具体干了什么

+   创建一个空对象
+   继承了该函数的原型
+   属性和方法被加入到 this 引用的对象中
+   新创建的对象由 this 所引用，并且最后隐式的返回 this

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
+   promise
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

在一次事件循环中，会先执行js线程的主任务，然后会去查找是否有微任务microtask（promise），如果有那就优先执行微任务，如果没有，在去查找宏任务macrotask（setTimeout、setInterval）进行执行。

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

1.  标记清除（mark and sweep）

+   这是JavaScript最常见的垃圾回收方式，当变量进入执行环境的时候，比如函数中声明一个变量，垃圾回收器将其标记为“进入环境”，当变量离开环境的时候（函数执行结束）将其标记为“离开环境”
+   垃圾回收器会在运行的时候给存储在内存中的所有变量加上标记，然后去掉环境中的变量以及被环境中变量所引用的变量（闭包），在这些完成之后仍存在标记的就是要删除的变量了

2.  引用计数(reference counting)

+   在低版本IE中经常会出现内存泄露，很多时候就是因为其采用引用计数方式进行垃圾回收。
+   引用计数的策略是跟踪记录每个值被使用的次数。
+   当声明了一个 变量并将一个引用类型赋值给该变量的时候这个值的引用次数就加1
+   如果该变量的值变成了另外一个，则这个值得引用次数减1，
+   当这个值的引用次数变为0的时 候，说明没有变量在使用，这个值没法被访问了，因此可以将其占用的空间回收，这样垃圾回收器会在运行的时候清理掉引用次数为0的值占用的空间

## 深浅拷贝

**浅拷贝**：仅拷贝第一层

+   `Object.assign`，
+   展开运算符 `...`

**深拷贝**： `JSON.parse(JSON.stringify(object))` **缺陷**：

+   会忽略 undefined
+   会忽略 symbol
+   不能序列化函数
+   不能解决循环引用的对象

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

### 与vue2和vue3搭配使用的姿势

**§ vue2 + ts**

+   参考：[Vue2 + TypeScript 最佳入门实践](https://juejin.cn/post/6844903865255477261 "https://juejin.cn/post/6844903865255477261")

src目录下多了两个文件的作用：

+   `shims-tsx.d.ts`，允许你以.tsx结尾的文件，在Vue项目中编写jsx代码
+   `shims-vue.d.ts` 主要用于 TypeScript 识别.vue 文件，Ts默认并不支持导入 vue 文件，这个文件告诉ts 导入.vue 文件都按VueConstructor处理。

**§ vue2 + typescript组件的写法：**

```html
<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component({
    name: 'HelloWorld',
    components:{ componentA, componentB},
})
export default class HelloWorld extends Vue {
    // 组件属性props
    @Prop({ default: 'default value' }) private msg!: string;

    // 组件数据data
    private test: { value: string }

    // 计算属性
    private get reversedMessage (): string[] {
  	return this.message.split('').reverse().join('')
    }
    // 监听数据变化（onMsgChanged方法名并无实际意义有即可）
    @Watch('child')
    onMsgChanged(val: string, oldVal: string) {}

    // Vuex 数据
    @State((state: IRootState) => state . booking. currentStep) step!: number
    @Getter( 'person/name') name!: name
  
    // 生命周期
    private created ()：void { },

    // method
    public getName(): string {
        let storeName = name
        return storeName
    }
}
</script>


```

**§ vue3 + typescript组件的写法：**

+   参考：[Vue3 + TypeScript 完整项目上手教程](https://jishuin.proginn.com/p/763bfbd34d60 "https://jishuin.proginn.com/p/763bfbd34d60")

```ts
import { defineComponent, PropType } from 'vue'

interface Student {
    name: string
    class: string
    age: number
}
// option 风格
const Component = defineComponent({
    props: {
        success: { type: String },
        callback: {
            type: Function as PropType<() => void>
        },
        student: {
            type: Object as PropType,
            required: true
        }
    },
    data() {
        return {
            message: 'Vue3 code style'
        }
    },
    computed: {
        reversedMessage(): string {
            return this.message.split(' ').reverse().join('')
        }
    }
})
// Composition API风格
const Component = defineComponent(() => {
    const year = ref(2020)
    const month = ref('9')

    month.value = 9 // OK
    const result = year.value.split('') // => Property 'split' does not exist on type 'number'
    // reactive
    const student = reactive({ name: '阿勇', age: 16 })
})
```
