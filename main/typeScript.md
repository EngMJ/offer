
面试题涉及了 TypeScript 语言的各个方面，包括`基本语法`、`类型系统`、`函数`、`类`、`模块化`、`泛型`、`装饰器`等。在面试中，常见的 TypeScript 面试题主要围绕以下几个方面展开：

> **类型系统**：考察对 TypeScript 类型系统的理解，包括基本类型、联合类型、交叉类型、接口、类型别名、类型推断、类型守卫等。
>
> **函数和类**：涉及函数参数类型、返回值类型、箭头函数、函数重载、类的定义、继承、访问修饰符等概念。
>
> **泛型**：考察在函数、类和接口中如何使用泛型来增加代码的灵活性和复用性。
>
> **模块化**：问题可能涉及 ES6 模块化的语法、导入导出方式以及模块解析等内容。
>
> **装饰器**：了解对装饰器的使用，包括类装饰器、方法装饰器、属性装饰器以及参数装饰器的定义和应用。
>
> **编译配置**：熟悉 tsconfig.json 中的配置选项，包括编译目标、模块系统、严格模式等。
>
> **工程化实践**：了解 TypeScript 在项目中的实际应用，如与 JavaScript 的混用、第三方库的声明文件使用、类型声明等。

下面是上述涵盖内容的具体面试题～👇

#### 1\. 什么是TypeScript

`TypeScript`是一种由微软开发的开源编程语言，它是JavaScript的`超集`。TypeScript通过添加`静态类型`、`类`、`接口`和`模块`等功能，使得在大型应用程序中更容易进行维护和扩展。它可以被编译为纯JavaScript，从而能够在任何支持JavaScript的地方运行。使用TypeScript可以帮助开发人员在编码过程中避免一些常见的错误，并提供更好的代码编辑功能和工具支持。

#### 2\. 类型声明和类型推断的区别，并举例应用

类型声明是显式地为变量或函数指定类型，而类型推断是TypeScript根据赋值语句右侧的值自动推断变量的类型。例如：

```ts
// 类型声明
let x: number;
x = 10;
// 类型推断
let y = 20; // TypeScript会自动推断y的类型为number
```

#### 3\. 什么是接口（interface），它的作用，接口的使用场景。接口和类型别名（Type Alias）的区别

接口是用于描述对象的形状的结构化类型。它定义了对象应该包含哪些属性和方法。在TypeScript中，接口可以用来约束对象的结构，以提高代码的可读性和维护性。例如：

```ts
interface Person {
    name: string;
    age: number;
}
function greet(person: Person) {
    return `Hello, ${person.name}!`;
}
```

`接口`和`类型别名`的区别：

+   `接口`定义了一个契约，描述了对象的形状（属性和方法），以便在多个地方共享。它可以被类、对象和函数实现。
+   `类型别名`给一个类型起了一个新名字，便于在多处使用。它可以用于原始值、联合类型、交叉类型等。与接口不同，类型别名可以用于原始类型、联合类型、交叉类型等，而且还可以为任意类型指定名字。

#### 4\. 什么是泛型（generic），如何创建泛型函数和泛型类，实际用途

`泛型`是一种在定义函数、类或接口时使用类型参数的方式，以增加代码的灵活性和重用性。在TypeScript中，可以使用来创建泛型。例如：

```ts
function identity<T>(arg: T): T {
    return arg;
}
// 调用泛型函数
let output = identity<string>("hello");
```

#### 5\. 枚举（enum）是什么，它的优势，应用案例。枚举和常量枚举的区别

枚举是一种对数字值集合进行命名的方式。它们可以增加代码的可读性，并提供一种便捷的方式来使用一组有意义的常量。例如：

```ts
enum Color {
    Red,
    Green,
    Blue
}

let selectedColor: Color = Color.Red;
```

枚举和常量枚举的区别:

+   `枚举`可以包含计算得出的值，而常量枚举则在编译阶段被删除，并且不能包含计算得出的值，它只能包含常量成员。
+   `常量枚举`在编译后会被删除，而普通枚举会生成真实的对象。

#### 6\. 如何处理可空类型（nullable types）和undefined类型，如何正确处理这些类型以避免潜在错误

在TypeScript中，可空类型是指一个变量可以存储特定类型的值，也可以存储`null`或`undefined`。（通过使用可空类型，开发者可以明确表达一个变量可能包含特定类型的值，也可能不包含值（即为`null`或`undefined`）。这有助于提高代码的可读性，并使得变量的可能取值范围更加清晰明了）。

为了声明一个可空类型，可以使用联合类型（Union Types），例如 `number | null` 或 `string | undefined`。 例如：

```ts
let numberOrNull: number | null = 10; 
numberOrNull = null; // 可以赋值为null 
    
let stringOrUndefined: string | undefined = "Hello"; 
stringOrUndefined = undefined; // 可以赋值为undefined
```

#### 7\. 什么是联合类型和交叉类型

`联合类型`表示一个值可以是多种类型中的一种，而`交叉类型`表示一个新类型，它包含了多个类型的特性。

+   联合类型示例：

```ts
// typescript
let myVar: string | number;
myVar = "Hello"; // 合法
myVar = 123; // 合法
```

+   交叉类型示例：

```ts
interface A {
  a(): void;
}
interface B {
  b(): void;
}
type C = A & B; // 表示同时具备 A 和 B 的特性
```

#### 8\. 什么是TypeScript中的声明文件（Declaration Files）

声明文件（通常以 `.d.ts` 扩展名结尾）用于描述已有 JavaScript 代码库的类型信息。它们提供了类型定义和元数据，以便在 TypeScript 项目中使用这些库时获得智能感知和类型安全。

#### 9\. 什么是命名空间（Namespace）和模块（Module） 

`模块`

+   在一个大型项目中，可以将相关的代码组织到单独的文件，并使用模块来导入和导出这些文件中的功能。
+   在一个 Node.js 项目中，可以使用 import 和 export 关键字来创建模块，从而更好地组织代码并管理依赖关系。

`命名空间`

+   在面向对象的编程中，命名空间可以用于将具有相似功能或属性的类、接口等进行分组，以避免全局命名冲突。
+   这在大型的 JavaScript 或 TypeScript 应用程序中特别有用，可以确保代码结构清晰，并且不会意外地重复定义相同的名称。

`模块`提供了一种组织代码的方式，使得我们可以轻松地在多个文件中共享代码，

`命名空间`则提供了一种在全局范围内组织代码的方式，防止命名冲突。

模块示例:

```ts
// greeter.ts
export function sayHello(name: string) {
  return `Hello, ${name}!`;
}
// app.ts
import { sayHello } from './greeter';
console.log(sayHello('John'));
```

命名空间示例:

```ts
// greeter.ts
namespace Greetings {
  export function sayHello(name: string) {
    return `Hello, ${name}!`;
  }
}
// app.ts
<reference path="greeter.ts" />
console.log(Greetings.sayHello('John'));
```

在上面的示例中：

+   使用模块时，我们可以使用 `export` 和 `import` 关键字来定义和引入模块中的函数或变量。

+   而在命名空间中，我们使用 namespace 来创建命名空间，并且需要在使用之前使用  `<reference path="file.ts" />` 来引入命名空间。


#### 10\. 什么是类型断言（Type Assertion）

类型断言允许程序员手动指定一个值的类型。这在需要明确告诉编译器某个值的类型时非常有用。

```ts
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

#### 11\. TypeScript中的可选参数和默认参数是什么

+   可选参数允许函数中的某些参数不传值，在参数后面加上问号`?`表示可选。
+   默认参数允许在声明函数时为参数指定`默认值`，这样如果调用时未提供参数值，则会使用默认值。

可选参数示例：

```ts
function greet(name: string, greeting?: string) {
  if (greeting) {
    return `${greeting}, ${name}!`;
  } else {
    return `Hello, ${name}!`;
  }
}
```

默认参数示例：

```ts
function greet(name: string, greeting: string = "Hello") {
  return `${greeting}, ${name}!`;
}
```

#### 12\. 类型守卫（Type Guards）是什么

类型守卫是一种用于在运行时检查类型的技术，它允许开发人员在特定的作用域内缩小变量的范围，以确保正确推断类型。

```ts
function isString(test: any): test is string {
  return typeof test === "string";
}
if (isString(input)) {
  // input 在此代码块中被收窄为 string 类型
}
```

#### 13\. 索引类型（Index Types）是什么，好处有什么

索引类型允许我们在 TypeScript 中创建具有动态属性名称的对象，并且能够根据已知的键来获取相应的属性类型。 好处：

**1.动态属性访问**

> 在处理动态属性名的对象时，可以使用索引类型来实现类型安全的属性访问。例如，当从服务器返回的 JSON 数据中提取属性时，可以利用索引类型来确保属性名存在并获取其对应的类型。

**2.代码重用**

> 当需要创建通用函数来操作对象属性时，索引类型可以帮助我们实现更加通用和灵活的代码。例如，一个通用的函数可能需要根据传入的属性名称获取属性值，并进行特定的处理。

```ts
interface ServerData {
  id: number;
  name: string;
  age: number;
  // 可能还有其他动态属性
}
function getPropertyValue(obj: ServerData, key: keyof ServerData): void {
  console.log(obj\[key]); // 确保 obj\[key] 的类型是正确的 // 这里可以直接使用索引类型来获取属性值
}
```

**3.动态扩展对象**

> 当需要处理来自外部来源（比如 API 响应或数据库查询）的动态数据时，索引类型可以让我们轻松地处理这种情况，而不必为每个可能的属性手动定义类型。

```ts
interface DynamicObject {
  [key: string]: number | string; // 允许任意属性名，但属性值必须为 number 或 string 类型

}
function processDynamicData(data: DynamicObject): void {
  for (let key in data) {
    console.log(key + ": " + data\[key]); // 对任意属性进行处理
  }
}
```

**4.类型安全性**

> 索引类型可以增强代码的类型安全性，因为它们可以捕获可能的属性名拼写错误或键不存在的情况。

**5.映射类型**

> TypeScript 还提供了映射类型（Mapped Types）的概念，它们利用索引类型可以根据现有类型自动生成新类型。这在创建新类型时非常有用，特别是当需要在现有类型的基础上添加或修改属性时。

#### 14\. const和readonly的区别

当在TypeScript中使用`const`和`readonly`时，它们的行为有一些显著的区别：

+   **const：**

    +   `const`用于声明常量值。一旦被赋值后，其值将不能被重新赋值或修改。
    +   常量必须在声明时就被赋值，并且该值不可改变。
    +   常量通常用于存储不会发生变化的值，例如数学常数或固定的配置值。

```ts
const PI = 3.14;
PI = 3.14159; // Error: 无法重新分配常量
```

+   **readonly：**

    +   `readonly`关键字用于标记类的属性，表明该属性只能在类的构造函数或声明时被赋值，并且不能再次被修改。
    +   `readonly`属性可以在声明时或构造函数中被赋值，但之后不能再被修改。
    +   `readonly`属性通常用于表示对象的某些属性是只读的，防止外部代码修改这些属性的值。

```ts
class Person {
    readonly name: string;

    constructor(name: string) {
        this.name = name; // 可以在构造函数中赋值
    }
}

let person = new Person("Alice");
person.name = "Bob"; // Error: 无法分配到"name"，因为它是只读属性
```

总结来说，`const`主要用于声明常量值，而`readonly`则用于标记类的属性使其只读。

#### 15\. TypeScript 中 any 类型的作用是什么，滥用会有什么后果

在TypeScript中，`any`类型的作用是允许我们在编写代码时不指定具体的类型，从而可以接受任何类型的值。使用`any`类型相当于放弃了对该值的静态类型检查，使得代码在编译阶段不会对这些值进行类型检查。

主要情况下，`any`类型的使用包括以下几点：

+   当我们不确定一个变量或表达式的具体类型时，可以使用any类型来暂时绕过类型检查。

+   在需要与动态类型的JavaScript代码交互时，可以使用any类型来处理这些动态类型的值。

+   有时候某些操作难以明确地定义其类型，或者需要较复杂的类型推导时，也可以使用any类型。


**滥用的后果：**

尽管any类型提供了灵活性，但由于它会放弃TypeScript的静态类型检查，因此滥用any类型可能会降低代码的健壮性和可维护性。当滥用`any`类型时，可能会导致以下后果：

**1.代码可读性下降：**

```ts
let data: any;
// 代码中的使用方式
data.someUnknownMethod(); // 在编译阶段不会报错，但实际上可能是一个错误
```

**2.潜在的运行时错误：**

```ts
let myVariable: any = 123;
myVariable.toUpperCase(); // 在编译阶段不会报错，但在运行时会引发错误
```

**3.类型安全受损：**

```ts
function add(x: any, y: any): any {
    return x + y; // 编译器无法推断返回值的具体类型
}
```

滥用`any`类型会导致代码失去了TypeScript强大的类型检查功能，带来了如下问题：

+   可能引入未知的运行时行为和错误。
+   降低了代码的可维护性和可读性，因为难以理解某些变量或参数的具体类型。

因此，在实际开发中，应尽量避免过度使用`any`类型。可以通过合适的类型声明、接口定义和联合类型等方式，提高代码的健壮性和可维护性。

#### 16\. TypeScript中的this有什么需要注意的

在TypeScript中，与JavaScript相比，`this`的行为基本上是一致的。然而，TypeScript提供了类型注解和类型检查，可以帮助开发者更容易地理解和处理`this`关键字的使用。

> 在noImplicitThis为true 的情况下，必须声明 this 的类型，才能在函数或者对象中使用this。
>
> Typescript中箭头函数的 this 和 ES6 中箭头函数中的 this 是一致的。

在TypeScript中，当将`noImplicitThis`设置为`true`时，意味着在函数或对象中使用this时，必须显式声明`this`的类型。这一设置可以帮助开发者更明确地指定this的类型，以避免因为隐式的`this`引用而导致的潜在问题。

具体来说，如果将`noImplicitThis`设置为`true`，则在下列情况下必须显式声明this的类型：

+   在函数内部使用this时，需要使用箭头函数或显示绑定this。
+   在某些类方法或对象方法中，需要明确定义this的类型。

示例代码如下所示：

```ts
class MyClass {
  private value: number = 42;

  public myMethod(this: MyClass) {
    console.log(this.value);
  }

  public myMethod2 = () => {
    console.log(this.value);
  }
}

let obj = new MyClass();
obj.myMethod(); // 此处必须传入合适的 this 类型
```

通过将`noImplicitThis`设置为`true`，TypeScript要求我们在使用`this`时明确指定其类型，从而在编译阶段进行更严格的类型检查，帮助避免一些可能出现的错误和不确定性。

注：`noImplicitThis`是TypeScript编译器的一个配置选项，用于控制在函数或对象方法中使用`this`时的严格性。当将`noImplicitThis`设置为`true`时，意味着必须显式声明`this`的类型，否则会触发编译错误。

#### 17\. TypeScript数据类型

在TypeScript中，常见的数据类型包括以下几种：

+   **基本类型**：

    +   `number`: 表示数字，包括整数和浮点数。
    +   `string`: 表示文本字符串。
    +   `boolean`: 表示布尔值，即`true`或`false`。
    +   `null`、`undefined`: 分别表示null和undefined。
    +   `symbol`: 表示唯一的、不可变的值。
+   **复合类型**：

    +   `array`: 表示数组，可以使用`number[]`或`Array<number>`来声明其中元素的类型。
    +   `tuple`: 表示元组，用于表示固定数量和类型的数组。
    +   `enum`: 表示枚举类型，用于定义具名常量集合。
+   **对象类型**：

    +   `object`: 表示非原始类型，即除number、string、boolean、symbol、null或undefined之外的类型。
    +   `interface`: 用于描述对象的结构，并且可以重复使用。
+   **函数类型**：

    +   `function`: 表示函数类型。
    +   `void`: 表示函数没有返回值。
    +   `any`: 表示任意类型。
+   **高级类型**：

    +   `union types`: 表示一个值可以是几种类型之一。
    +   `intersection types`: 表示一个值同时拥有多种类型的特性。

#### 18\. interface可以给Function/Array/Class（Indexable)做声明吗

在TypeScript中，`interface`可以用来声明函数、数组和类（具有索引签名的类）。下面是一些示例代码：

**1\. Interface 声明函数**

```ts
interface MyFunc {
  (x: number, y: number): number;
}

let myAdd: MyFunc = function(x, y) {
  return x + y;
};
```

在上述示例中，`MyFunc`接口描述了一个函数类型，该函数接受两个参数并返回一个数字。

**2\. Interface 声明数组**

```ts
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Alice"];
```

上面的示例中，`StringArray`接口描述了一个具有数字索引签名的字符串数组。意味着我们可以通过数字索引来访问数组元素。

**3\. Interface 声明类（Indexable）**

```ts
interface StringDictionary {
  [index: string]: string;
}

let myDict: StringDictionary = {
  "name": "John",
  "age": "30"
};
```

在这个例子中，`StringDictionary`接口用于描述具有字符串索引签名的类或对象。这使得我们可以像操作字典一样使用对象的属性。

综上：TypeScript中的`interface`可以被用来声明函数、数组和具有索引签名的类，从而帮助我们定义和限定这些数据结构的形式和行为。

#### 19\. TypeScript中的协变、逆变、双变和抗变是什么

在TypeScript中，`协变（Covariance）`、`逆变（Contravariance）`、`双变（Bivariance）`和`抗变（Invariance` 是与类型相关的概念，涉及到参数类型的子类型关系。下面对这些概念进行解释，并提供示例代码。

**协变（Covariance）**

+   **区别**：协变意味着子类型可以赋值给父类型。
+   **应用场景**：数组类型是协变的，因此可以将子类型的数组赋值给父类型的数组。

`协变`表示类型T的子类型可以赋值给类型U，当且仅当T是U的子类型。在TypeScript中，`数组`是协变的，这意味着可以将子类型的数组赋值给父类型的数组。

```ts
let subtypes: string[] = ["hello", "world"];
let supertype: Object[] = subtypes; // 数组是协变的，这是合法的
```

**逆变（Contravariance）**

+   **区别**：逆变意味着超类型可以赋值给子类型。
+   **应用场景**：函数参数类型是逆变的，因此可以将超类型的函数赋值给子类型的函数。

`逆变`表示类型T的超类型可以赋值给类型U，当且仅当T是U的子类型。在TypeScript中，`函数参数`是逆变的，这意味着可以将超类型的函数赋值给子类型的函数。

```ts
type Logger<T> = (arg: T) => void;
let logNumber: Logger<number> = (x: number) => console.log(x);
let logAny: Logger<any> = logNumber; // 函数参数是逆变的，这是合法的
```

**双变（Bivariance）**

+   **区别**：双变允许参数类型既是协变又是逆变的。
+   **应用场景**：对象类型是双变的，这意味着可以将子类型的对象赋值给父类型的对象，同时也可以将超类型的对象赋值给子类型的对象。

`双变`允许参数类型既是`协变`又是`逆变`的。在TypeScript中，`普通对象类型`是双变的，这意味着可以将子类型的对象赋值给父类型的对象，并且可以将超类型的对象赋值给子类型的对象。

```ts
interface Animal {
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

**抗变（Invariance）**

+   **区别**：抗变表示不允许类型之间的任何赋值关系。
+   **应用场景**：通常情况下，基本类型和类类型是抗变的。

`抗变`表示不允许类型T和U之间的任何赋值关系，即T既不是U的子类型，也不是U的超类型。在TypeScript中，一般情况下，`基本类型`和`类类型`是抗变的。

```ts
let x: string = "hello";
let y: string = x; // 这是合法的

let a: Animal = { name: "Animal" };
let b: Animal = a; // 这也是合法的
```

#### 20\. TypeScript中的静态类型和动态类型有什么区别

+   `静态类型`是在 **编译期间** 进行类型检查，可以在编辑器或 IDE 中发现大部分类型错误。
+   `动态类型`是在 **运行时** 才确定变量的类型，通常与动态语言相关联。

**静态类型（Static Typing）**

+   **定义**：静态类型是指在编译阶段进行类型检查的类型系统，通过类型注解或推断来确定变量、参数和返回值的类型。
+   **特点**：静态类型能够在编码阶段就发现大部分类型错误，提供了更好的代码健壮性和可维护性。
+   **优势**：可以在编辑器或 IDE 中实现代码提示、自动补全和类型检查，帮助开发者减少错误并提高代码质量。

**动态类型（Dynamic Typing）**

+   **定义**：动态类型是指在运行时才确定变量的类型，通常与动态语言相关联，允许同一个变量在不同时间引用不同类型的值。
+   **特点**：动态类型使得变量的类型灵活多变，在运行时可以根据上下文或条件动态地改变变量的类型。
+   **优势**：动态类型可以带来更大的灵活性，适用于一些需要频繁变化类型的场景。

**区别总结**

+   **时机差异**：静态类型在编译期间进行类型检查，而动态类型是在运行时才确定变量的类型。
+   **代码稳定性**：静态类型有助于在编码阶段发现大部分类型错误，提高代码稳定性；动态类型对类型的要求较为灵活，但可能增加了代码的不确定性。
+   **使用场景**：静态类型适合于大型项目和团队，能够提供更强的类型安全性；动态类型适用于快速原型开发和灵活多变的场景，能够更快地迭代和测试代码。

#### 21\. 介绍TypeScript中的可选属性、只读属性和类型断言

+   **可选属性** 使用 `?` 来标记一个属性可以存在，也可以不存在。
+   **只读属性** 使用 `readonly` 关键字来标记一个属性是只读的。
+   **类型断言** 允许将一个实体强制指定为特定的类型，使用 `<Type>` 或 `value as Type`。

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
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // Error: 只读属性无法重新赋值

// 类型断言
let someValue: any = "hello";
let strLength: number = (someValue as string).length;
```

#### 22\. TypeScript 中的模块化是如何工作的，举例说明

**答案：**

+   TypeScript 中使用 ES6 模块系统，可以使用 `import` 和 `export` 关键字来导入和导出模块。
+   可以通过 `export default` 导出默认模块，在导入时可以使用 `import moduleName from 'modulePath'`。

**代码示例：**

```ts
// math.ts
export function sum(a: number, b: number): number {
  return a + b;
}
export function subtract(a: number, b: number): number {
  return a - b;
}

// app.ts
import { sum, subtract } from './math';
console.log(sum(3, 5)); // 输出 8
```

#### 23\. 如何约束泛型参数的类型范围

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

#### 24\. 什么是泛型约束中的 keyof 关键字，举例说明其用法。

+   `keyof` 是 TypeScript 中用来获取对象类型所有键（属性名）的操作符。
+   可以使用 `keyof` 来定义泛型约束，限制泛型参数为某个对象的键。

**代码示例：**

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
let x = { a: 1, b: 2, c: 3 };
getProperty(x, "a"); // 正确
getProperty(x, "d"); // 错误：Argument of type '"d"' is not assignable to parameter of type '"a" | "b" | "c"'
```

#### 25\. 什么是条件类型（conditional types），能够举例说明其使用场景吗

+   条件类型是 TypeScript 中的高级类型操作符，可以根据一个类型关系判断结果类型。
+   例如，可以使用条件类型实现一个类型过滤器，根据输入类型输出不同的结果类型。

**代码示例：**

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
type T0 = NonNullable<string | null>;  // string
type T1 = NonNullable<string | null | undefined>;  // string
type T2 = NonNullable<string | number | null>;  // string | number
```

#### 25\. 什么是装饰器，有什么作用，如何在TypeScript中使用类装饰器

+   `装饰器`是一种特殊类型的声明，可以附加到类、方法、访问符、属性或参数上，以修改其行为。
+   在 TypeScript 中，装饰器提供了一种在声明时定义如何处理类的方法、属性或参数的机制。

**如何在 TypeScript 中使用类装饰器：**

```ts
function classDecorator<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    newProperty = "new property";
    hello = "override";
  };
}

@classDecorator
class Greeter {
  property = "property";
  hello: string;
  constructor(m: string) {
    this.hello = m;
  }
}
console.log(new Greeter("world")); // 输出 { property: 'property', hello: 'override', newProperty: 'new property' }
```

#### 26\. 类装饰器和方法装饰器的执行顺序是怎样的

+   当有`多个装饰器应用于同一个声明`时（比如一个类中的方法），它们将按照`自下而上`的顺序应用。
+   对于`方法装饰器`，从顶层方法开始`依次向下` `递归调用`方法装饰器函数。

#### 27\. 装饰器工厂是什么，请给出一个装饰器工厂的使用示例

+   `装饰器工厂`是一个返回装饰器的函数。它可以接受参数，并根据参数动态生成装饰器。

**以下是一个简单的装饰器工厂示例：**

```ts
function color(value: string) {
  return function (target: any, propertyKey: string) {
    // ... 在此处使用 value 和其他参数来操作装饰目标
  };
}

class Car {
  @color('red')
  brand: string;
}
```

* * *
