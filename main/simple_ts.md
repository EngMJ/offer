# TS 常用内容
---

## 1. 基本类型

TypeScript 内置了多种基本数据类型，包括字符串、数字、布尔值、null、undefined 等。

```typescript
// 字符串类型
let username: string = "TypeScript";

// 数字类型
let score: number = 100;

// 布尔类型
let isActive: boolean = true;

// null 和 undefined
let nothing: null = null;
let notAssigned: undefined = undefined;
```

---

## 2. 数组与元组

### 数组

可以指定数组中元素的类型：
```typescript
// 数组类型写法一：使用类型后跟 []
let numbers: number[] = [1, 2, 3];

// 数组类型写法二：使用 Array 泛型
let names: Array<string> = ["Alice", "Bob", "Charlie"];
```

### 元组

元组允许你定义一个固定长度且每个位置具有特定类型的数组：
```typescript
let userInfo: [string, number] = ["Alice", 30];
```

---

## 3. 接口（Interfaces）

接口用于描述对象的结构或函数类型，可用于类型检查和代码补全。

```typescript
// 描述对象结构
interface Person {
  name: string;
  age: number;
}

let person: Person = {
  name: "Bob",
  age: 25,
};

// 描述函数类型
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const search: SearchFunc = (source, subString) => {
  return source.indexOf(subString) > -1;
};
```

---

## 4. 类（Classes）

TypeScript 对 ES6 类进行了扩展，支持访问修饰符、抽象类、接口实现等。

```typescript
class Animal {
  // 访问修饰符：public（默认）、private、protected
  private name: string;

  constructor(name: string) {
    this.name = name;
  }

  public move(distance: number): void {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

let cat = new Animal("Kitty");
cat.move(10);
```

### 抽象类与接口实现

```typescript
abstract class Shape {
  abstract area(): number;

  display(): void {
    console.log(`Area: ${this.area()}`);
  }
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }
  
  area(): number {
    return this.width * this.height;
  }
}

const rect = new Rectangle(10, 5);
rect.display(); // 输出 Area: 50
```

---

## 5. 泛型（Generics）

泛型使代码更加灵活，同时保证类型安全，适用于函数、接口、类等。

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("Hello");
let output2 = identity<number>(123);

// 泛型接口
interface Pair<T, U> {
  first: T;
  second: U;
}

let pair: Pair<string, number> = { first: "one", second: 1 };

// 泛型类
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(zeroValue: T, addFunc: (x: T, y: T) => T) {
    this.zeroValue = zeroValue;
    this.add = addFunc;
  }
}

const myGenericNumber = new GenericNumber<number>(0, (a, b) => a + b);
console.log(myGenericNumber.add(5, 10)); // 输出 15
```

---

## 6. 高级类型

### 联合类型（Union Types）

允许变量取多种类型中的一种：
```typescript
let id: number | string;
id = 101;
id = "202";
```

### 交叉类型（Intersection Types）

将多个类型合并为一个类型，要求同时满足所有类型的约束：
```typescript
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface Data {
  data: any;
}

type Response = ErrorHandling & Data;

let response: Response = {
  success: true,
  data: { info: "Details" },
};
```

### 字面量类型（Literal Types）

限制变量只能取固定的值：
```typescript
type Direction = "left" | "right" | "up" | "down";

let move: Direction;
move = "left";
// move = "forward"; // 错误，不在允许的范围内
```

---

## 7. 函数

TypeScript 对函数参数和返回值的类型进行检查，可以使用默认参数、可选参数和剩余参数。

```typescript
// 基本函数声明
function add(x: number, y: number): number {
  return x + y;
}

// 可选参数（用 ? 标识）
function buildName(firstName: string, lastName?: string): string {
  return lastName ? `${firstName} ${lastName}` : firstName;
}

// 默认参数
function multiply(a: number, b: number = 2): number {
  return a * b;
}

// 剩余参数
function sumAll(...numbers: number[]): number {
  return numbers.reduce((prev, curr) => prev + curr, 0);
}
```

---

## 8. 模块与命名空间

### 模块

模块化可以帮助你分割代码并实现代码复用。使用 `export` 导出变量或函数，使用 `import` 导入。

```typescript
// math.ts 模块
export function add(x: number, y: number): number {
  return x + y;
}

// main.ts 文件中导入并使用
import { add } from "./math";
console.log(add(5, 3));
```

### 命名空间

命名空间用于在一个全局作用域中组织代码，避免名称冲突。

```typescript
namespace Geometry {
  export function areaOfCircle(radius: number): number {
    return Math.PI * radius * radius;
  }
}

console.log(Geometry.areaOfCircle(10));
```

---

## 9. 装饰器（Decorators）

装饰器是一种特殊的声明，可以附加到类、方法、属性或参数上，用于修改或增强行为。需要在 `tsconfig.json` 中启用 `experimentalDecorators` 选项。

```typescript
// 方法装饰器示例
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return originalMethod.apply(this, args);
  };
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3); // 调用时会打印日志
```

---

## 10. 类型断言（Type Assertions）

类型断言用于告诉编译器变量的实际类型，语法上类似于类型转换，但不会进行实际的数据转换。

```typescript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
// 或者使用尖括号语法（在 JSX 中不建议使用）
// let strLength: number = (<string>someValue).length;
```

