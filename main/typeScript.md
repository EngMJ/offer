## Typescript

[参考: TS 常用内容](./simple_ts.md)

---

## 1. TypeScript 是什么？

`TypeScript`是一种由微软开发的开源编程语言，它是JavaScript的`超集`

- **静态类型检查**：在编译期发现类型错误，减少运行时错误。
- **更好的编辑器支持**：提供智能提示、重构和自动补全。
- **可读性和可维护性**：通过类型系统使代码意图更明确。
- **丰富的高级类型系统**：支持交叉类型、联合类型、类型守卫、泛型、映射类型等。

---

##  2. TypeScript 与 JavaScript 的区别？

- **类型系统**：TS 是 JavaScript 的超集，增加了静态类型检查。
- **编译过程**：TypeScript 代码需要编译成 JavaScript 才能运行。
- **ES 新特性支持**：TS 支持部分 ES 的未来特性，并通过编译器转译成目标环境支持的版本。

---

## 3. TypeScript 与 ES6/ESNext 特性的关系
- **兼容性：**  
  TypeScript 支持大部分 ES6/ESNext 的新特性，例如箭头函数、类、模块、解构赋值、异步编程等。
- **编译目标：**  
  通过配置 `target` 选项，可以将 TypeScript 编译为兼容旧版 JavaScript 环境的代码。例如，将 ES6 特性转换为 ES5 代码。
- **增强类型安全：**  
  在使用 ES6/ESNext 新特性的同时，TypeScript 提供了静态类型检查，使得开发过程中能够提前发现错误并提高代码的可维护性。

---


##  4. TypeScript 的类型系统有哪些？
   TypeScript 提供了丰富的类型系统，主要包括以下几类：

+   **基本数据类型**：

    +   `number`: 表示数字，包括整数和浮点数。
    +   `string`: 表示文本字符串。
    +   `boolean`: 表示布尔值，即`true`或`false`。
    +   `null`、`undefined`: 分别表示null和undefined。
    +   `symbol`: 表示唯一的、不可变的值。
+   **复合类型**：

    +   `array`: 表示数组，可以使用`number[]`或`Array<number>`来声明其中元素的类型。
    +   `tuple`: 表示元组，用于表示固定数量和类型的数组。
    +   `enum`: 表示枚举类型，用于定义具名常量集合。
    +   `literal`: 表示字面量类型，用于限定变量只能取固定值。

+   **对象类型**：

    +   `object`: 表示非原始类型，即除number、string、boolean、symbol、null或undefined之外的类型。
    +   `interface`: 用于描述对象的结构，并且可以重复使用。


+   **函数类型**：
    +   `function`: 表示函数类型。
    +   `void`: 表示函数没有返回值。
    +   `never`: 表示函数永远不会返回结果，比如抛出异常或无限循环。

+   **高级类型**：

    +   `union types`: 联合类型,表示一个值可以是几种类型之一。
    +   `intersection types`: 交叉类型,表示一个值同时拥有多种类型的特性。

+   **特殊类型**：
    - `any`（任意类型，不进行类型检查）、`unknown`（未知类型，必须做类型检查）
---

## 5. interface 与 type 有什么区别？
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

## 6. interface可以给Function/Array/Class/ Indexable Types做声明吗?

在 TypeScript 中，接口（interface）不仅可以描述对象的形状，还可以用来声明函数类型、数组类型、类的实例类型以及支持索引操作的类型。

---

1. 声明函数类型

可以使用接口来描述函数的参数和返回值：

```typescript
interface MyFunction {
  (x: number, y: number): number;
}

const add: MyFunction = (a, b) => a + b;
console.log(add(2, 3)); // 输出：5
```

---

2. 声明数组类型

利用索引签名，可以定义数组或者类似数组的结构：

```typescript
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray = ["Alice", "Bob", "Charlie"];
console.log(myArray[1]); // 输出："Bob"
```

---

3. 描述类的实例类型

接口可以用来定义类的实例需要实现的属性和方法：

```typescript
interface Person {
  name: string;
  sayHello(): void;
}

class Student implements Person {
  constructor(public name: string) {}
  
  sayHello() {
    console.log(`Hello, my name is ${this.name}`);
  }
}

const student = new Student("Alice");
student.sayHello(); // 输出："Hello, my name is Alice"
```

---

4. 支持索引操作的接口（Indexable Types）

[参考:索引类型](#24-索引类型index-types是什么好处有什么)

除了数组外，接口还可以用来描述具备索引签名的对象，比如一个可以通过字符串索引获取值的对象：

```typescript
interface StringDictionary {
  [key: string]: string;
}

let dict: StringDictionary = {
  "name": "Alice",
  "city": "Wonderland"
};

console.log(dict["name"]); // 输出："Alice"
```

---

## 7. 什么是命名空间（Namespace）和模块（Module）

**命名空间（Namespace）**

- **概念**  
  命名空间是一种内部模块（Internal Module）的实现方式，用于在一个全局作用域内组织和分组代码。它主要依靠内部封装来防止变量和函数污染全局命名空间。

- **特点**
    - 使用 `namespace` 关键字声明。
    - 通常适用于在单个编译单元中组织大量代码。
    - 不需要使用额外的模块加载器，直接在编译后的文件中以自执行函数的方式组织代码。
    - 如果代码量比较大，多个命名空间还可以通过`/// <reference path="...">`标签来进行跨文件引用。

- **示例代码**:
  ```typescript
  namespace Geometry {
    export function areaOfCircle(radius: number): number {
      return Math.PI * radius * radius;
    }
  }

  // 使用时：
  const area = Geometry.areaOfCircle(5);
  console.log(area);
  ```

---

**模块（Module）**

- **概念**  
  模块是独立的代码单元，具有自己的作用域，可以通过 `import` 和 `export` 来共享代码。ES6 模块（ES2015 modules）在 TypeScript 中得到原生支持，并且推荐用于现代应用程序的构建。

- **特点**
    - 每个模块都有自己的顶级作用域，模块内部声明的变量、函数和类默认不会污染全局作用域。
    - 使用 `export` 导出需要共享的部分，使用 `import` 引入其他模块的内容。
    - 模块通常在编译后生成单独的文件，依赖于模块加载器（如 Node.js、Webpack 或浏览器原生支持）来动态加载。
    - 支持异步加载和更精细的依赖管理，适合构建大型、分布式的项目。

- **示例代码**:

  **mathUtil.ts**
  ```typescript
  export function add(a: number, b: number): number {
    return a + b;
  }
  ```

  **app.ts**
  ```typescript
  import { add } from './mathUtil';

  console.log(add(2, 3));
  ```

---

**主要区别总结:**

- **代码组织方式**
    - **命名空间**：在全局脚本中通过命名空间封装代码，主要用于内部逻辑的组织。
    - **模块**：文件级别的代码分离，每个文件就是一个模块，通过导入和导出实现代码共享和隔离。

- **加载方式**
    - **命名空间**：编译后通常合并为一个单一的文件，运行时直接在全局作用域中访问。
    - **模块**：独立文件，在运行时依赖模块加载器或者打包工具进行加载，具备更灵活的加载机制。

- **作用域隔离**
    - **命名空间**：依然运行在全局作用域内，需要手动管理命名冲突。
    - **模块**：每个模块拥有自己的独立作用域，不会自动污染全局命名空间。

- **开发推荐**
    - 对于新的项目和大型应用，推荐使用模块系统（ES6 模块），它提供更好的依赖管理、按需加载以及工具链支持。
    - 命名空间适合一些简单场景或旧代码的维护，但在现代开发中使用较少。

---

## 8. Declaration Merging（声明合并）是什么？

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


## 9. 如何使用泛型（Generics）？
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

## 10. 如何约束泛型参数的类型范围

可以使用泛型约束（`extends关键字`）来限制泛型参数的类型范围，确保泛型参数符合某种特定的条件。

**代码示例：**

```ts
interface Lengthwise {
  length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
loggingIdentity({length: 10, value: 3});  // 参数满足 Lengthwise 接口要求，可以正常调用
```

---

## 11. 什么是泛型约束中的 keyof 关键字的作用?

+   `keyof`是 TypeScript 中用来获取对象类型所有键（属性名）的操作符。

```ts
interface Person {
  name: string;
  age: number;
}

type PersonKeys = keyof Person; // "name" | "age"

```

+   可以使用`keyof`来定义泛型约束，限制泛型参数为某个对象的键。

**代码示例：**

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
let x = { a: 1, b: 2, c: 3 };
getProperty(x, "a"); // 正确
getProperty(x, "d"); // 错误：Argument of type '"d"' is not assignable to parameter of type '"a" | "b" | "c"'
```


---

## 12. TypeScript中的可选参数/默认参数/剩余参数是什么?

TypeScript 对函数参数和返回值的类型进行检查，可以使用默认参数、可选参数和剩余参数。

```typescript
// 基本函数声明
function add(x: number, y: number): number {
  return x + y;
}

// 可选参数（用 ? 标识）,即可传或不传的参数
function buildName(firstName: string, lastName?: string): string {
  return lastName ? `${firstName} ${lastName}` : firstName;
}

// 默认参数, 即参数有默认值
function multiply(a: number, b: number = 2): number {
  return a * b;
}

// 剩余参数, 用于接收不定数量的参数,以数组形式传入
function sumAll(...numbers: number[]): number {
  return numbers.reduce((prev, curr) => prev + curr, 0);
}
```

---

## 13. 介绍TypeScript中的可选属性、只读属性

+   **可选属性** 使用`?`来标记一个属性可以存在，也可以不存在。
+   **只读属性** 使用`readonly`关键字来标记一个属性是只读的。

**代码示例：**

```ts
// 可选属性
interface Person {
  name: string;
  age?: number; // 可选属性
}

// 只读属性
interface Point {
  readonly x: number;
  readonly y: number;
}

```

---

## 14. TypeScript中的协变、逆变、双变和抗变是什么?


在 TypeScript 以及更广泛的类型系统中，**协变**、**逆变**、**双变**和**抗变**描述了类型之间在子类型关系下的“传递性”或“兼容性”。

---

1. 协变（Covariance）

- **定义**  
  当类型之间的子类型关系在某个“位置”中被保留时，我们称这种位置为协变。例如，如果 `Dog` 是 `Animal` 的子类型，那么一个返回 `Dog` 的函数可以被看作是返回 `Animal` 的函数。

- **在 TypeScript 中的体现**
    - **数组类型**：数组的元素类型通常是协变的，这意味着你可以将 `Dog[]` 赋值给 `Animal[]`，因为每个 `Dog` 都是一个 `Animal`。
      ```typescript
      class Animal { }
      class Dog extends Animal { }
      
      let dogs: Dog[] = [new Dog()];
      let animals: Animal[] = dogs;  // 协变：Dog[] 被认为是 Animal[] 的子类型
      ```
    - **函数返回值**：函数返回值位置通常也是协变的。

---

2. 逆变（Contravariance）

- **定义**  
  当类型之间的子类型关系在“输入”位置中被反转时，称这种位置为逆变。

- **在 TypeScript 中的体现**
    - **函数参数**：理想状态下，函数参数应当是逆变的。例如，函数类型如果要求参数类型为 `Dog`，那么你应该能够传入接受 `Animal` 参数的函数。但由于历史原因和实际使用中的灵活性，TypeScript 默认对函数参数采用**双变**（见下文），在严格模式下可以部分收紧这一规则。

```ts
class Animal {
  name: string = "Animal";
}

class Dog extends Animal {
  breed: string = "Unknown";
}

type Handler<T> = (arg: T) => void;

let handleAnimal: Handler<Animal> = (animal: Animal) => {
  console.log("Handling Animal: " + animal.name);
};

let handleDog: Handler<Dog> = (dog: Dog) => {
  console.log("Handling Dog: " + dog.breed);
};

// 逆变示例：
// 将一个接受 Animal 参数的函数赋值给一个期望接受 Dog 参数的变量是允许的，
// 因为 Dog 是 Animal 的子类型，所以 handleAnimal 能安全地处理传入的 Dog 实例。
handleDog = handleAnimal; // 在 strictFunctionTypes 模式下允许

// 反过来则不安全：
// 如果将一个只能处理 Dog 的函数赋值给期望接受 Animal 参数的变量，则存在风险，
// 因为 Animal 可能不仅仅是 Dog，这时 handleDog 不能处理所有 Animal 实例。
handleAnimal = handleDog; // 在 strictFunctionTypes 模式下会报错
```

---

3. 双变（Bivariance）

- **定义**  
  双变指的是函数参数既可以从协变也可以从逆变方向来检查，即既允许更宽泛也允许更严格的类型。这种行为实际上是不安全的，但出于方便和兼容性的考虑，TypeScript 默认允许函数参数双变。

    - **在 TypeScript 中的体现**
        - 默认情况下（非严格函数类型模式下），`普通对象类型`与`函数参数`检查是双变的。例如：
          ```typescript
          type Handler = (x: number) => void;
          
          let handler1: Handler = (x: number) => { /* ... */ };
          let handler2: Handler = (x: any) => { /* ... */ };
          
          // 默认允许以下赋值：
          handler1 = handler2; // 双变：允许更宽泛或更严格的参数类型互换
          handler2 = handler1; // 双变：允许更宽泛或更严格的参数类型互换
          ```

          ```ts
            interface Animal{
              name: string;
            }
            
            interface Dog extends Animal {
              breed: string;
            }
            
            let animal: Animal = { name: "Animal" };
            let dog: Dog = { name: "Dog", breed: "Labrador" };
            
            animal = dog; // 对象类型是双变的，这是合法的
            dog = animal; // 对象类型是双变的，这也是合法的
          ```

            - 当启用 `strictFunctionTypes` 编译选项后，TypeScript 会让函数参数严格逆变，从而提高类型安全性。

---

4. 抗变（Invariance）

- **定义**  
  抗变（有时称为不变性）既不允许协变也不允许逆变，要求类型必须完全匹配。`基本类型`和`类类型`是抗变的.

- **在 TypeScript 中的体现**
    - **泛型类型参数**：在某些泛型上下文中，除非显式标明，否则 TypeScript 会认为泛型参数是抗变的，也就是说，在不进行特殊处理的情况下，不允许子类型赋值给泛型参数。例如：
      ```typescript
      interface Box<T> {
        value: T;
      }
      
      let box1: Box<Animal>;
      let box2: Box<Dog>;
      // 默认下，Box<Dog> 不能赋值给 Box<Animal>，因为 Box 是抗变的
      // box1 = box2; // 会报错
      ```
    - 这种抗变设计有助于防止因子类型不匹配而引起的潜在问题，但有时也会需要通过设计映射类型或使用类型转换来突破这一限制。

---


## 15. 什么是装饰器?

装饰器（Decorator）是 TypeScript 中的一种实验性语法，它允许你在类、方法、属性或参数上添加注解或元数据，从而对其行为进行修改或扩展。装饰器本质上是一个函数，在编译时被调用，可以用来实现横切关注点（如日志记录、权限检查、依赖注入等）。

**类型**:
1. **类装饰器 `@ClassLogger`**
    - 在类定义后调用，接收构造函数作为参数。
    - 用于在类定义后打印日志或对类进行扩展。

2. **属性装饰器 `@PropertyLogger`**
    - 用于拦截对 `name` 属性的读取和写入操作。
    - 通过 `Object.defineProperty` 重写属性访问，打印相关日志信息。

3. **方法装饰器 `@MethodLogger`**
    - 包装 `greet` 方法，在方法调用前后打印日志。
    - 使用 `descriptor.value` 重写原始方法，通过 `apply` 调用原始实现。

4. **参数装饰器 `@ParameterLogger`**
    - 在方法参数上使用，记录参数的位置。
    - 注意：参数装饰器仅用于收集元数据，不影响参数值。

```typescript
// 类装饰器：接收目标类的构造函数作为参数
function ClassLogger(constructor: Function) {
  console.log(`ClassLogger: 装饰器被应用到类 ${constructor.name} 上`);
  // 可以在此处对类进行扩展或修改，例如增加静态属性、方法等
}

// 方法装饰器：接收目标对象、方法名和属性描述符作为参数
function MethodLogger(
  target: Object,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // 保存原始方法实现
  const originalMethod = descriptor.value;

  // 重写方法，添加日志记录功能
  descriptor.value = function (...args: any[]) {
    console.log(`MethodLogger: 调用方法 ${propertyKey}，参数：`, args);
    // 调用原始方法并获取返回值
    const result = originalMethod.apply(this, args);
    console.log(`MethodLogger: 方法 ${propertyKey} 返回：`, result);
    return result;
  };
}

// 属性装饰器：接收目标对象和属性名作为参数
function PropertyLogger(target: any, propertyKey: string) {
  // 保存属性的初始值
  let value = target[propertyKey];

  // 定义 getter，用于读取属性值时打印日志
  const getter = () => {
    console.log(`PropertyLogger: 读取属性 ${propertyKey}，当前值：`, value);
    return value;
  };

  // 定义 setter，用于修改属性值时打印日志
  const setter = (newVal: any) => {
    console.log(`PropertyLogger: 修改属性 ${propertyKey}，新值：`, newVal);
    value = newVal;
  };

  // 使用 Object.defineProperty 替换属性的访问方式，从而拦截读取和赋值操作
  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true,
  });
}

// 参数装饰器：用于记录方法参数的信息
function ParameterLogger(
  target: Object,
  propertyKey: string | symbol,
  parameterIndex: number
) {
  console.log(
    `ParameterLogger: 装饰器应用于方法 ${propertyKey.toString()} 的第 ${parameterIndex} 个参数`
  );
}

@ClassLogger // 类装饰器应用：在 Example 类定义后执行
class Example {
  @PropertyLogger // 属性装饰器应用：拦截 name 属性的读取和写入操作
  public name: string;

  constructor(name: string) {
    this.name = name;
  }

  // 方法装饰器应用：在 greet 方法定义时包装方法
  greet(@ParameterLogger greeting: string): string {
    // 参数装饰器在此处记录了 greeting 参数的信息
    return `${greeting}, ${this.name}!`;
  }
}

// 创建类实例，执行装饰器的效果
const ex = new Example("TypeScript");
// 读取属性，将触发属性装饰器中的 getter
console.log("调用 greet 方法结果：", ex.greet("Hello"));
// 修改属性，将触发属性装饰器中的 setter
ex.name = "TS";
// 再次调用方法，观察日志输出
console.log("再次调用 greet 方法结果：", ex.greet("Hi"));
```

---

## 16. 装饰器执行顺序是怎样的?

分为**求值顺序**和**应用顺序**, **求值顺序**是指装饰器表达式的执行顺序，**应用顺序**是指装饰器函数的执行顺序。

`求值顺序`总是按照代码的书写顺序 `从上到下`。`应用顺序`总是按照装饰器的调用顺序 `从下到上`（即最后写的最先执行）。

```ts
function ClassDec1() {
    console.log("ClassDec1: 类装饰器求值");
    return function (target: Function) {
        console.log("ClassDec1: 类装饰器应用");
    };
}

function ClassDec2() {
    console.log("ClassDec2: 类装饰器求值");
    return function (target: Function) {
        console.log("ClassDec2: 类装饰器应用");
    };
}

function MethodDec1() {
    console.log("MethodDec1: 方法装饰器求值");
    return function (target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("MethodDec1: 方法装饰器应用");
    };
}

function MethodDec2() {
    console.log("MethodDec2: 方法装饰器求值");
    return function (target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("MethodDec2: 方法装饰器应用");
    };
}

function PropertyDec1() {
    console.log("PropertyDec1: 属性装饰器求值");
    return function (target: Object, propertyKey: string) {
        console.log("PropertyDec1: 属性装饰器应用");
    };
}

function PropertyDec2() {
    console.log("PropertyDec2: 属性装饰器求值");
    return function (target: Object, propertyKey: string) {
        console.log("PropertyDec2: 属性装饰器应用");
    };
}

function ParamDec1() {
    console.log("ParamDec1: 参数装饰器求值");
    return function (target: Object, propertyKey: string, parameterIndex: number) {
        console.log("ParamDec1: 参数装饰器应用");
    };
}

function ParamDec2() {
    console.log("ParamDec2: 参数装饰器求值");
    return function (target: Object, propertyKey: string, parameterIndex: number) {
        console.log("ParamDec2: 参数装饰器应用");
    };
}

@ClassDec1()
@ClassDec2()
class Example {
    @PropertyDec1()
    @PropertyDec2()
    prop!: string;

    @MethodDec1()
    @MethodDec2()
    method(@ParamDec1() @ParamDec2() param: string) {}
}

// 输出内容:
// PropertyDec1: 属性装饰器求值
// PropertyDec2: 属性装饰器求值
// MethodDec1: 方法装饰器求值
// MethodDec2: 方法装饰器求值
// ParamDec1: 参数装饰器求值
// ParamDec2: 参数装饰器求值
// ClassDec1: 类装饰器求值
// ClassDec2: 类装饰器求值
//
// ParamDec2: 参数装饰器应用
// ParamDec1: 参数装饰器应用
// PropertyDec2: 属性装饰器应用
// PropertyDec1: 属性装饰器应用
// MethodDec2: 方法装饰器应用
// MethodDec1: 方法装饰器应用
// ClassDec2: 类装饰器应用
// ClassDec1: 类装饰器应用
```

---

## 17. 装饰器工厂是什么?
**装饰器工厂**是一种返回装饰器函数的函数，它允许在使用装饰器时传递参数，从而使装饰器具有更高的灵活性。  
**示例**：
```typescript
function Log(prefix: string) {
  // 装饰器工厂返回实际的装饰器函数
  return function(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = function(...args: any[]) {
      console.log(`${prefix} 调用方法: ${propertyKey}，参数: ${JSON.stringify(args)}`);
      return originalMethod.apply(this, args);
    };
  };
}

class Calculator {
  @Log("DEBUG")
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
// 在这个例子中，`Log` 就是一个装饰器工厂，它接收一个字符串参数 `prefix`，并返回一个方法装饰器。调用 `@Log("DEBUG")` 时，实际上是先调用 `Log("DEBUG")` 得到装饰器函数，再应用到 `add` 方法上。
calc.add(2, 3);
```

---

## 18. 枚举（enum）是什么?枚举和常量枚举的区别?

枚举是一种对数字值集合进行命名的方式,从而使代码更加直观、可读和易于维护。例如：

```ts
enum Color {
    Red,
    Green,
    Blue,
}

console.log(Color.Red); // 输出：0
console.log(Color[0]);  // 输出："Red"
```

枚举和常量枚举的区别:

+   `枚举`编译后生成对象,所以运行时可遍历、查找等。
+   `常量枚举`不能包含计算成员,且在编译后替换成具体的数值，编译后不会生成额外的对象,不能使用枚举成员且无法进行反向映射,无法进行运行时各种操作。

---

## 19. 联合类型和交叉类型的区别？

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

## 20. 如何处理可空类型（nullable types）和undefined类型?

在TypeScript中，可空类型是指一个变量可以存储特定类型的值，也可以存储`null`或`undefined`。（通过使用可空类型，开发者可以明确表达一个变量可能包含特定类型的值，也可能不包含值（即为`null`或`undefined`）。这有助于提高代码的可读性，并使得变量的可能取值范围更加清晰明了）。

为了声明一个可空类型，可以使用联合类型（Union Types），例如 `number | null` 或 `string | undefined`。 例如：

```ts
let numberOrNull: number | null = 10; 
numberOrNull = null; // 可以赋值为null 
    
let stringOrUndefined: string | undefined = "Hello"; 
stringOrUndefined = undefined; // 可以赋值为undefined
```

---

## 21. TypeScript 中的类型断言（Type Assertion）和非空断言（Non-null Assertion）的用法

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



## 22. 类型推断（Type Inference）是什么？
- **类型推断：**  
  当声明变量时如果没有明确的类型注解，TypeScript 会根据赋值自动推断出变量的类型。例如：
  ```typescript
  let num = 42; // TypeScript 推断 num 的类型为 number
  ```

---


## 23. 什么是条件类型（conditional types）?

条件类型（Conditional Types）是 TypeScript 中的一种高级类型机制，允许你根据类型之间的关系来选择不同的类型。它类似于 JavaScript 中的三元表达式语法，但应用在类型层面上。

**代码示例：**

```ts
// 基本使用
type IsString<T> = T extends string ? "yes" : "no";

type Test1 = IsString<string>;  // "yes"
type Test2 = IsString<number>;  // "no"

```

```ts
// 泛型使用
function example<T>(value: T): T extends number ? number : string {
  if (typeof value === "number") {
    return (value * 2) as any;
  } else {
    return "不是数字" as any;
  }
}

const result1 = example(10);    // 推断为 number
const result2 = example("hello"); // 推断为 string
```

---

## 24. 索引类型（Index Types）是什么?

索引类型允许我们在 TypeScript 中创建具有动态属性名称的对象，并且能够根据已知的键来获取相应的属性类型。 好处：

**属性名的安全管理**：通过 `keyof` 限制和查询对象的属性名称, 避免拼写错误。

```ts
interface ServerData {
 id: number;
 name: string;
 age: number;
 // 可能还有其他动态属性
}
function getPropertyValue(obj: ServerData, key: keyof ServerData): void {
 console.log(obj[key]); // 确保 key 值是 ServerData 中对应的属性值, 避免拼写错误
}
```

**动态扩展对象**: 批量定义数据类型，不必为每个可能的属性手动定义类型。

```ts
interface DynamicObject {
 [key: string]: number | string; // 允许任意属性名，但属性值必须为 number 或 string 类型

}
function processDynamicData(data: DynamicObject): void {
 for (let key in data) {
     console.log(key + ": " + data[key]); // 对任意属性进行处理
 }
}
```

**属性类型的访问**: 使用索引访问类型（`T[K]`）获得具体属性的类型。

```ts
interface Person {
 name: string;
 age: number;
}

type NameType = Person["name"]; // string
type AgeType = Person["age"]; // number
```

**增强泛型编程能力**:在泛型函数中确保参数类型与对象属性保持一致。

```typescript
// 在这个例子中，通过 `K extends keyof T` 限制了传入的属性名称必须是对象 `T` 的属性，从而在编译期捕捉错误
function pluck<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map(key => obj[key]);
}

const person: Person = { name: "Alice", age: 30 };
const values = pluck(person, ["name"]); // 类型推断为 string[]
```


**创建映射类型**:利用索引访问类型，可以创建映射类型，对现有类型进行遍历、转换，生成新类型（如只读、可选、部分类型等）。

```typescript
// 可以利用 `in` 操作符创建一个只读版本的类型：
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

const readonlyPerson: Readonly<Person> = { name: "Bob", age: 25 };
// readonlyPerson.age = 26; // 报错：无法分配到 "age" ，因为它是只读属性
```

---


## 25. 类型守卫/类型保护（Type Guards）是什么?

TypeScript 中的类型守卫（Type Guards）是一种在运行时检查变量类型并缩小联合类型范围的机制。

---

1. **`typeof` 操作符**  
   用于判断基本类型，如 `string`、`number`、`boolean` 等。
   ```typescript
   function printValue(val: string | number) {
     if (typeof val === "string") {
       console.log(val.toUpperCase()); // 此处 val 被缩小为 string
     } else {
       console.log(val.toFixed(2)); // 此处 val 被缩小为 number
     }
   }
   ```

2. **`instanceof` 操作符**  
   用于判断对象是否是某个类的实例。
   ```typescript
   class Animal {
     name: string = "animal";
   }
   
   class Dog extends Animal {
     bark() {
       console.log("Woof!");
     }
   }
   
   function speak(animal: Animal) {
     if (animal instanceof Dog) {
       animal.bark(); // animal 被缩小为 Dog 类型
     } else {
       console.log("Not a dog.");
     }
   }
   ```

3. **`in` 操作符**  
   检查对象中是否存在某个属性，可以用来缩小类型。
   ```typescript
   interface Bird {
     fly(): void;
   }
   
   interface Fish {
     swim(): void;
   }
   
   function move(animal: Bird | Fish) {
     if ("fly" in animal) {
       animal.fly(); // animal 被缩小为 Bird 类型
     } else {
       animal.swim(); // animal 被缩小为 Fish 类型
     }
   }
   ```

4. **自定义类型守卫函数**  
   通过返回 `arg is SpecificType` 的布尔值，告诉编译器在特定条件下变量属于某种类型。
   ```typescript
   interface Cat {
     meow(): void;
   }
   
   interface Dog {
     bark(): void;
   }
   
   function isCat(animal: Cat | Dog): animal is Cat {
     return (animal as Cat).meow !== undefined;
   }
   
   function makeSound(animal: Cat | Dog) {
     if (isCat(animal)) {
       animal.meow(); // animal 被缩小为 Cat 类型
     } else {
       animal.bark(); // animal 被缩小为 Dog 类型
     }
   }
   ```

---

## 26. const和readonly的区别

**const**

- **适用范围：变量声明**  
  `const` 用于声明变量，并且在声明时必须初始化。声明后，该变量不能再被赋予新的值。

- **特点**
    - **值不可重新赋值**：一旦声明就不能修改绑定，但如果绑定的是对象，则对象内部的属性仍然可以修改（除非对象被深冻结）。
    - **作用于基本类型与引用类型**：对于基本类型，值完全不可变；对于引用类型，引用地址不可变，但对象内容可能仍然可变。

- **示例代码**:
  ```typescript
  const num = 42;
  // num = 50; // 报错：不能给常量赋值

  const person = { name: "Alice", age: 25 };
  person.age = 26; // 允许修改对象内部属性
  // person = { name: "Bob", age: 30 }; // 报错：不能重新赋值对象引用
  ```

---

**readonly**

- **适用范围：类和接口的属性**  
  `readonly` 关键字用于修饰类、接口或类型中的属性，表示该属性在对象构造完成后不能被修改。

- **特点**
    - **只能在初始化或构造函数中赋值**：`readonly` 属性可以在声明时或者在构造函数中赋值，之后不能更改。
    - **适用于对象内部属性的只读设计**：确保对象的某些属性一旦设定后不会被误改动，从而提高代码的安全性和可维护性。

- **示例代码**:
  ```typescript
  class Person {
    readonly name: string;
    readonly birthYear: number;

    constructor(name: string, birthYear: number) {
      this.name = name;
      this.birthYear = birthYear;
    }
  }

  const person = new Person("Alice", 1990);
  // person.name = "Bob"; // 报错：无法分配到 "name" ，因为它是只读属性
  ```

- **在接口中使用**:
  ```typescript
  interface Point {
    readonly x: number;
    readonly y: number;
  }

  const point: Point = { x: 10, y: 20 };
  // point.x = 15; // 报错：x 是只读属性
  ```

---

**总结:**

- **`const`** 用于声明变量，其值在声明后不能重新赋值，但对于引用类型，只保证引用地址不变，内部数据仍可能修改。
- **`readonly`** 用于修饰对象的属性，保证属性在初始化之后不可更改，更适合用于设计不可变的数据结构或对象的部分属性。

---

## 27. TypeScript 中 any 类型的作用是什么?
在 TypeScript 中，`any` 类型用于表示任意类型，它允许你跳过编译器的静态类型检查。

---

**any 类型的作用:**

- `any` 提供了灵活性，允许开发者在类型不明确或者暂时不想指定具体类型时使用。
  ```typescript
      let data: any;
      data = "Hello";
      data = 42;
      data = { key: "value" };
  ```
- 连接 JavaScript 时，`any` 类型可以作为一个临时解决方案，帮助逐步添加类型。

- 处理来自第三方库等数据结构不确定类型，可先用 `any` 类型接收数据，再逐步进行类型定义和校验。

---

**滥用 any 的后果:**

- **失去类型安全**  
  过度使用 `any` 会使 TypeScript 的类型检查失效，导致无法在编译阶段捕获潜在的类型错误，从而使代码变得更容易出现运行时错误。
  ```typescript
  let value: any = "Hello";
  // 假设开发者误认为 value 是数字
  let result = value * 2; // 运行时可能得到 NaN 或其他不可预期结果
  ```

- **降低代码可读性和可维护性**

- **失去开发工具TS支持**

- **失去类型检查导致隐藏潜在错误**

---


## 28. TypeScript中的this的使用?

TypeScript 的 `this` 机制和 JavaScript 保持一致，均取决于函数调用时的上下文。增加 `this` 类型声明与类型检查。

- **静态类型检查**  
  TypeScript 提供了一种方式来显式指定函数中的 `this` 类型，这样在编译时就能捕获由于错误的 `this` 指向而引起的类型问题。例如：
  ```typescript
  interface User {
    name: string;
    greet(this: User): void;
  }

  const user: User = {
    name: "Alice",
    greet() {
      console.log(`Hello, ${this.name}`);
    }
  };

  user.greet(); // 正确调用
  // const greet = user.greet;
  // greet(); // 编译错误，因为 this 未被指定为 User 类型
  ```

- **`this` 参数**  
  在 TypeScript 中，可以为函数添加一个显式的 `this` 参数，从而确保该函数只能在特定的上下文中调用。这种参数在实际运行时不会被传递，但它在编译阶段起到了类型检查的作用。
  ```typescript
    interface User {
        name: string;
        greet(this: User): void;    
    }
  
    function greet(this: User) {
      console.log(`Hello, ${this.name}`);
    }
    
    greet.call({ name: "Alice" }); // 正确调用
    // greet(); // 编译错误，因为 this 未被指定为 User 类型
  ```

---

## 29. TypeScript中的静态类型和JavaScript 动态类型有什么区别?

1. 静态类型（Static Typing）

- **编译期检查**  
  TypeScript 在编译阶段就对代码进行类型检查，能够在代码运行前发现类型错误。例如，类型不匹配、函数参数错误等问题都能提前暴露，减少运行时错误。

- **明确的类型声明**  
  开发者可以为变量、函数参数、返回值等显式声明类型，提升代码的可读性和维护性。例如：
  ```typescript
  function add(a: number, b: number): number {
    return a + b;
  }
  
  let result: number = add(2, 3);
  ```

- **工具支持与自动补全**  
  静态类型信息使得 IDE 和编辑器可以提供更精准的自动补全、重构和错误提示功能，改善开发体验。

- **优势**
    - 提前捕捉错误
    - 更好的代码文档和可维护性
    - 增强代码的健壮性

---

2. 动态类型（Dynamic Typing）

- **运行时确定类型**  
  在 JavaScript 中，变量的类型是在运行时确定的，可以随时改变。例如：
  ```javascript
  let value = 42;   // value 为数字
  value = "hello";  // 现在 value 为字符串
  ```

- **灵活性高**  
  动态类型允许更灵活的编程风格，但同时也可能导致类型不一致的问题，从而增加了运行时错误的风险。

- **无编译时类型检查**  
  由于类型在运行时确定，错误可能直到代码执行时才暴露，这对调试和维护大型应用来说可能是个隐患。

---


## 30. 什么是 TypeScript 的声明文件（.d.ts 文件）？
    声明文件用于描述 JavaScript 库或模块的类型信息，从而让 TypeScript 项目在引入这些库时能够进行类型检查和代码补全。
- **来源：**  
  大部分流行的 JavaScript 库都有社区提供的声明文件，通常可以通过 npm 包 `@types/xxx` 来获取。
- **用途：**  
  不需要修改 JavaScript 库的源码，只需提供外部声明文件就能获得静态类型支持。

---

## 31. 如何配置 tsconfig.json？
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

