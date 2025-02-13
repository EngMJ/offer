# 字符串 API

## 1. 截取与提取

### **1.1 slice()**
- **用途**：提取字符串的子串，从 `start` 到 `end`（不包含 `end`）。
- **特点**：支持负数索引，负数表示从字符串末尾开始计算。

```js
let str = "Hello, World!";
console.log(str.slice(7, 12));      // "World"
console.log(str.slice(-6, -1));     // "World"
```

### **1.2 substring()**
- **用途**：提取字符串子串，从 `start` 到 `end`（不包含 `end`）。
- **特点**：不支持负数索引；如果参数为负，则会被当作 0。

```js
let str = "Hello, World!";
console.log(str.substring(7, 12));  // "World"
console.log(str.substring(-6, -1)); // 与 substring(0, 0) 等价，返回 ""
```

### **1.3 substr()**
- **用途**：从指定位置开始，提取指定长度的子字符串。
- **特点**：第一个参数支持负数，表示从字符串尾部开始计数；但此方法已被视为不推荐使用（deprecate），建议使用 `slice`。

```js
let str = "Hello, World!";
console.log(str.substr(7, 5));      // "World"
console.log(str.substr(-6, 5));     // "World"
```

---

## 2. 查找与检测

### **2.1 indexOf() 与 lastIndexOf()**
- **indexOf(searchValue, fromIndex)**：返回 `searchValue` 第一次出现的索引，若未找到返回 `-1`。
- **lastIndexOf(searchValue, fromIndex)**：返回 `searchValue` 最后一次出现的索引。

```js
let str = "Hello, World! Hello, JS!";
console.log(str.indexOf("Hello"));     // 0
console.log(str.lastIndexOf("Hello")); // 14
```

### **2.2 includes()**
- **用途**：检查字符串中是否包含指定的子字符串，返回布尔值。
- **特点**：比 `indexOf` 更直观。

```js
let str = "Hello, World!";
console.log(str.includes("World")); // true
console.log(str.includes("world")); // false，区分大小写
```

### **2.3 startsWith() 与 endsWith()**
- **startsWith(searchString, position)**：判断字符串是否以 `searchString` 开头。
- **endsWith(searchString, length)**：判断字符串是否以 `searchString` 结尾。

```js
let str = "Hello, World!";
console.log(str.startsWith("Hello")); // true
console.log(str.endsWith("!"));       // true
```

---

## 3. 替换与匹配

### **3.1 replace()**
- **用途**：替换字符串中第一次匹配的子串（或符合正则表达式的部分）。
- **特点**：如果传入正则表达式并加上 `g` 标志，则可替换所有匹配项。

```js
let str = "Hello, World! Hello, JS!";
console.log(str.replace("Hello", "Hi"));      // "Hi, World! Hello, JS!"
console.log(str.replace(/Hello/g, "Hi"));      // "Hi, World! Hi, JS!"
```

### **3.2 replaceAll()**
- **用途**：替换字符串中所有匹配的子串（ES2021 新增）。
- **特点**：更直观地替换所有匹配项，不需要使用正则表达式的全局标志。

```js
let str = "Hello, World! Hello, JS!";
console.log(str.replaceAll("Hello", "Hi"));   // "Hi, World! Hi, JS!"
```

### **3.3 match() 与 matchAll()**
- **match(regexp)**：根据正则表达式匹配字符串，返回匹配结果数组或 `null`。
- **matchAll(regexp)**：返回所有匹配的迭代器（ES2020 新增），适用于需要获取捕获组的场景。

```js
let str = "Hello, World! Hello, JS!";
console.log(str.match(/Hello/g)); // ["Hello", "Hello"]

// matchAll 例子：
const matches = str.matchAll(/(Hello)/g);
for (const match of matches) {
  console.log(match[0]); // 每个匹配的 "Hello"
}
```

### **3.4 search()**
- **用途**：使用正则表达式查找匹配，返回第一个匹配的索引，未匹配则返回 `-1`。

```js
let str = "Hello, World!";
console.log(str.search(/World/)); // 7
```

---

## 4. 转换与格式化

### **4.1 大小写转换**
- **toUpperCase()**：返回转换为大写后的新字符串。
- **toLowerCase()**：返回转换为小写后的新字符串。

```js
let str = "Hello, World!";
console.log(str.toUpperCase()); // "HELLO, WORLD!"
console.log(str.toLowerCase()); // "hello, world!"
```

### **4.2 去除空白字符**
- **trim()**：去除字符串两端的空白字符。
- **trimStart()/trimLeft()**：去除字符串开头的空白字符。
- **trimEnd()/trimRight()**：去除字符串末尾的空白字符。

```js
let str = "  Hello, World!  ";
console.log(str.trim());       // "Hello, World!"
console.log(str.trimStart());  // "Hello, World!  "
console.log(str.trimEnd());    // "  Hello, World!"
```

### **4.3 重复与填充**
- **repeat(count)**：返回一个新字符串，其内容是原字符串重复 `count` 次。
- **padStart(targetLength, padString)**：在字符串开头填充指定字符，直到达到目标长度。
- **padEnd(targetLength, padString)**：在字符串末尾填充指定字符。

```js
let str = "5";
console.log(str.repeat(3));         // "555"
console.log(str.padStart(3, "0"));    // "005"
console.log(str.padEnd(3, "0"));      // "500"
```

---

## 5. 分割与连接

### **5.1 split()**
- **用途**：将字符串按照指定的分隔符拆分为数组。
- **特点**：可以限制拆分出的数组项数量。

```js
let str = "apple,banana,cherry";
console.log(str.split(","));      // ["apple", "banana", "cherry"]
console.log(str.split(",", 2));   // ["apple", "banana"]
```

### **5.2 concat()**
- **用途**：拼接多个字符串，返回新字符串。
- **特点**：通常使用 `+` 运算符或模板字符串更直观。

```js
let str1 = "Hello";
let str2 = "World";
console.log(str1.concat(", ", str2)); // "Hello, World"
```

---

## 6. 其他常用方法

### **6.1 repeat()**
- 重复前面已提及，用于重复字符串。

### **6.2 localeCompare()**
- **用途**：根据当前区域设置比较两个字符串，返回 `-1`、`0` 或 `1`，分别表示小于、等于或大于。
- **用途场景**：常用于排序时的本地化比较。

```js
console.log("a".localeCompare("b")); // -1
```

---

## 总结

- **不可变性**：字符串在 JavaScript 中是不可变的，每个 API 都会返回一个新字符串，而不会直接修改原字符串。
- **截取方法**：`slice` 支持负数索引；`substring` 对负数处理不同；`substr` 语法简单，但已不推荐使用。
- **查找方法**：根据需求选择 `indexOf`、`lastIndexOf`、`includes`、`startsWith`、`endsWith` 等。
- **替换与匹配**：`replace` 与 `replaceAll` 用于替换；`match`、`matchAll` 和 `search` 则适用于基于正则的匹配查找。
- **转换方法**：大小写转换、空白去除、重复、填充等方法便于格式化字符串。
- **分割与连接**：`split` 用于将字符串转换成数组，`concat` 或模板字符串用于拼接字符串。


| API                                  | 用途/描述                                                         | 关键特点与区别                                                    |
|--------------------------------------|------------------------------------------------------------------|------------------------------------------------------------------|
| `slice(start, end)`                  | 从索引 `start` 到 `end`（不含）截取子串；支持负索引                   | 不改变原字符串；负值从末尾开始计数                                  |
| `substring(start, end)`              | 从 `start` 到 `end` 截取子串                                         | 不支持负索引，负值会被当作 0；与 `slice` 除负值外基本一致             |
| `substr(start, length)`              | 从 `start` 开始截取指定长度的子串                                    | 支持负数起始位置，但已不推荐使用，建议使用 `slice`                   |
| `indexOf(sub, pos)`                  | 返回子串首次出现的索引，未找到返回 -1                                 | 区分大小写，不改变原字符串                                         |
| `lastIndexOf(sub, pos)`              | 返回子串最后一次出现的索引，未找到返回 -1                               | 从末尾开始查找                                                   |
| `includes(sub, pos)`                 | 检查字符串是否包含指定子串，返回布尔值                                | 区分大小写，比 `indexOf` 更直观                                    |
| `startsWith(sub, pos)`               | 判断字符串是否以指定子串开头                                          | 返回布尔值                                                       |
| `endsWith(sub, length)`              | 判断字符串是否以指定子串结尾                                          | 返回布尔值                                                       |
| `replace(search, replaceWith)`       | 替换第一次匹配的子串，或借助正则替换匹配项（需添加 `g` 标志替换全部）         | 返回新字符串；原字符串不变                                          |
| `replaceAll(search, replaceWith)`    | 替换所有匹配的子串（ES2021）                                         | 语法更简洁，不需要使用正则全局标志                                   |
| `match(regexp)`                      | 根据正则表达式匹配字符串，返回匹配数组或 `null`                        | 用于单次匹配或全局匹配（需配合 `g` 标志）                            |
| `matchAll(regexp)`                   | 返回所有匹配的迭代器（ES2020），支持捕获组                           | 更适合需要获取所有匹配结果的场景                                    |
| `toUpperCase()`                      | 将字符串转换为大写                                                 | 返回新字符串                                                     |
| `toLowerCase()`                      | 将字符串转换为小写                                                 | 返回新字符串                                                     |
| `trim()`                             | 去除字符串两端空白字符                                               | 返回新字符串                                                     |
| `repeat(count)`                      | 返回重复 `count` 次的新字符串                                        | 返回新字符串                                                     |
| `padStart(targetLength, padString)`  | 在字符串前面填充指定字符至达到目标长度                                | 返回新字符串                                                     |
| `padEnd(targetLength, padString)`    | 在字符串末尾填充指定字符至达到目标长度                                | 返回新字符串                                                     |
| `split(separator, limit)`            | 按指定分隔符将字符串拆分为数组                                        | 返回新数组；原字符串不变                                           |
| `concat(...strings)`                 | 连接多个字符串                                                     | 返回新字符串，效果与 `+` 运算符一致                                 |

---

# 数字 API

## 1. 数字的构造与转换

### **1.1 Number() 构造函数与转换**

- **作为构造函数**：  
  使用 `new Number(value)` 创建的是一个 **Number 对象**（包装对象），而不是一个数字原始值。一般不推荐这么做，除非有特殊需求，因为包装对象与基本类型行为上存在差异。

  ```js
  let numObj = new Number("123");
  console.log(typeof numObj); // "object"
  ```

- **作为函数调用**：  
  直接调用 `Number(value)` 会将传入的值转换为数字原始值。

  ```js
  let num = Number("123");
  console.log(typeof num); // "number"
  ```

### **1.2 parseInt() 与 parseFloat()**

- **用途**：将字符串转换为数字。
- **区别**：
    - `parseInt()` 用于解析整数，并可指定进制（基数）；
    - `parseFloat()` 用于解析浮点数。

- **注意**：  
  ES6 后，Number 对象也提供了 `Number.parseInt()` 和 `Number.parseFloat()`，它们与全局函数完全一致，但归类在 Number 对象下，便于组织。

  ```js
  console.log(parseInt("123px"));       // 123
  console.log(parseFloat("3.14abc"));     // 3.14

  console.log(Number.parseInt("123px"));  // 123
  console.log(Number.parseFloat("3.14abc"));// 3.14
  ```

---

## 2. 数值判断方法

### **2.1 isNaN() vs Number.isNaN()**

- **全局 isNaN()**：
    - 会先尝试将传入值转换为数字，再判断是否为 NaN。
    - 这可能导致非数字类型被错误地判断为 NaN。

- **Number.isNaN()**：
    - 不会进行隐式类型转换，只在参数确实为 NaN 时返回 `true`。

  ```js
  console.log(isNaN("Hello"));        // true，因为 Number("Hello") 得到 NaN
  console.log(Number.isNaN("Hello"));   // false

  console.log(isNaN(NaN));              // true
  console.log(Number.isNaN(NaN));       // true
  ```

### **2.2 isFinite() vs Number.isFinite()**

- **全局 isFinite()**：
    - 会将传入值先转换为数字，然后判断是否为有限数值。

- **Number.isFinite()**：
    - 不进行类型转换，只有当值的类型为“number”且为有限数时返回 `true`。

  ```js
  console.log(isFinite("123"));         // true，因为 "123" 转换为 123
  console.log(Number.isFinite("123"));    // false

  console.log(isFinite(Infinity));        // false
  console.log(Number.isFinite(Infinity)); // false

  console.log(Number.isFinite(123));       // true
  ```

### **2.3 Number.isInteger()**

- **用途**：判断一个数是否为整数（即没有小数部分）。

  ```js
  console.log(Number.isInteger(25));      // true
  console.log(Number.isInteger(25.0));    // true
  console.log(Number.isInteger(25.1));    // false
  console.log(Number.isInteger("25"));    // false，不进行类型转换
  ```

---

## 3. 数字格式化与表示

数字实例方法主要用于将数字转换为字符串表示或进行格式化，常见的方法包括：

### **3.1 toString([radix])**

- **用途**：将数字转换为字符串表示，可以指定基数（进制，范围 2~36）。
- **示例**：

  ```js
  let num = 255;
  console.log(num.toString());      // "255"（默认十进制）
  console.log(num.toString(16));    // "ff"（16 进制）
  console.log(num.toString(2));     // "11111111"（二进制）
  ```

### **3.2 toFixed(digits)**

- **用途**：将数字格式化为固定小数位数的字符串，结果会四舍五入。
- **示例**：

  ```js
  let num = 3.14159;
  console.log(num.toFixed(2));   // "3.14"
  console.log(num.toFixed(4));   // "3.1416"
  ```

### **3.3 toExponential(fractionDigits)**

- **用途**：以指数计数法返回数字的字符串表示。
- **示例**：

  ```js
  let num = 123456;
  console.log(num.toExponential(2)); // "1.23e+5"
  ```

### **3.4 toPrecision(precision)**

- **用途**：将数字格式化为指定精度的字符串，精度包含整数部分与小数部分的总位数。
- **示例**：

  ```js
  let num = 123.456;
  console.log(num.toPrecision(5)); // "123.46"
  console.log(num.toPrecision(2)); // "1.2e+2"
  ```

---

## 4. 数值的基本获取

### **4.1 valueOf()**

- **用途**：返回数字对象的原始数值（基本类型）。
- **注意**：当使用 Number 对象（例如通过 `new Number()` 创建的对象）时，调用 `valueOf()` 可得到实际的数字值。

  ```js
  let numObj = new Number(100);
  console.log(numObj.valueOf()); // 100
  ```

---

## 总结

- **转换与构造**：
    - 使用 `Number()` 直接转换为数字原始值；
    - `new Number()` 创建的是包装对象，通常不推荐使用。

- **解析字符串**：
    - `parseInt()` 与 `parseFloat()`（以及它们在 Number 对象下的同名方法）用于将字符串转换为数值，但解析规则不同（整数 vs 浮点数）。

- **数值判断**：
    - `Number.isNaN()`、`Number.isFinite()`、`Number.isInteger()` 提供了更严格、不做隐式转换的判断，而全局的 `isNaN()` 和 `isFinite()` 会先做类型转换。

- **格式化方法**：
    - `toString()`、`toFixed()`、`toExponential()`、`toPrecision()` 这些实例方法用于数字与字符串之间的转换和格式化显示。


| API/方法                                 | 用途/描述                                                        | 关键特点与区别                                                         |
|-----------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------------------------|
| `Number(value)`                          | 将传入值转换为数字原始值                                            | 返回基本类型数值；与 `new Number()` 区别在于后者返回包装对象               |
| `new Number(value)`                      | 创建一个 Number 包装对象                                             | 返回对象，通常不推荐使用，易产生隐式类型转换问题                          |
| `parseInt(string, radix)`                | 从字符串中解析整数，可指定进制                                        | 忽略字符串后非数字部分；适用于解析整数                                  |
| `parseFloat(string)`                     | 从字符串中解析浮点数                                               | 解析小数；处理方式与 `parseInt` 不同                                     |
| `isNaN(value)`                           | 全局函数：先转换为数字，再判断是否为 NaN                              | 隐式类型转换可能导致误判                                               |
| `Number.isNaN(value)`                    | 判断值是否为 NaN，不做类型转换                                       | 更严格，仅当参数类型为 number 且为 NaN 时返回 `true`                    |
| `isFinite(value)`                        | 全局函数：先转换为数字，再判断是否为有限数                              | 隐式转换后判断有限性                                                  |
| `Number.isFinite(value)`                 | 判断值是否为有限数，不做类型转换                                     | 更严格，仅对数值类型判断                                               |
| `Number.isInteger(value)`                | 判断值是否为整数（无小数部分）                                        | 不进行类型转换，仅数值类型为整数时返回 `true`                           |
| `num.toString(radix)`                    | 将数字转换为字符串，可指定基数（2~36）                                 | 常用于转换为二进制、十六进制等                                          |
| `num.toFixed(digits)`                    | 将数字格式化为固定小数位数的字符串，四舍五入                            | 返回字符串，适合金额、精度要求场景                                      |
| `num.toExponential(fractionDigits)`      | 返回以指数计数法表示的字符串                                        | 返回字符串；可指定小数部分位数                                          |
| `num.toPrecision(precision)`             | 返回具有指定总有效数字的字符串表示                                    | 当数字较大或较小时可能采用指数计数法                                     |
| `valueOf()`（在 Number 对象上）          | 返回包装对象内部的原始数值                                           | 用于提取基本数值                                                     |

---

# 数组 API

## 1. 增删操作

### **1.1 push() 与 pop()**

- **push(...items)**
    - **用途**：在数组末尾添加一个或多个元素。
    - **返回值**：添加元素后的数组新长度。
    - **副作用**：直接修改原数组。

- **pop()**
    - **用途**：移除数组末尾的元素。
    - **返回值**：被移除的元素。
    - **副作用**：直接修改原数组。

```js
let arr = [1, 2, 3];
arr.push(4, 5); // arr 变为 [1, 2, 3, 4, 5]
let last = arr.pop(); // last 为 5，arr 变为 [1, 2, 3, 4]
```

### **1.2 unshift() 与 shift()**

- **unshift(...items)**
    - **用途**：在数组开头添加一个或多个元素。
    - **返回值**：添加元素后的数组新长度。
    - **副作用**：直接修改原数组。

- **shift()**
    - **用途**：移除数组第一个元素。
    - **返回值**：被移除的元素。
    - **副作用**：直接修改原数组。

```js
let arr = [2, 3];
arr.unshift(1); // arr 变为 [1, 2, 3]
let first = arr.shift(); // first 为 1，arr 变为 [2, 3]
```

### **1.3 splice()**

- **用途**：在任意位置进行删除、插入或替换操作。
- **语法**：`array.splice(start, deleteCount, ...items)`
    - **start**：起始索引。
    - **deleteCount**：要删除的元素个数（可选，不传时删除所有后续元素）。
    - **...items**：要插入的新元素（可选）。
- **返回值**：一个数组，包含被删除的元素。
- **副作用**：直接修改原数组。

```js
let arr = [1, 2, 3, 4, 5];
// 删除从索引1开始的2个元素
let removed = arr.splice(1, 2); // removed 为 [2, 3]，arr 变为 [1, 4, 5]
// 在索引1的位置插入元素 6, 7
arr.splice(1, 0, 6, 7); // arr 变为 [1, 6, 7, 4, 5]
```

### **1.4 slice() 与 concat()**

- **slice()**
    - **用途**：提取数组的一部分，返回新数组。
    - **语法**：`array.slice(start, end)`
    - **副作用**：不会修改原数组。

- **concat()**
    - **用途**：合并多个数组或添加新元素，返回新数组。
    - **语法**：`array.concat(value1, value2, ...)`
    - **副作用**：不会修改原数组。

```js
let arr = [1, 2, 3, 4, 5];
let subArr = arr.slice(1, 4); // subArr 为 [2, 3, 4]，arr 不变
let newArr = arr.concat([6, 7]); // newArr 为 [1, 2, 3, 4, 5, 6, 7]，arr 不变
```

---

## 2. 遍历与转换操作

### **2.1 forEach()**

- **用途**：对数组中的每个元素执行指定的函数。
- **返回值**：`undefined`。
- **特点**：只适用于遍历，不适合链式操作。

```js
let arr = [1, 2, 3];
arr.forEach(item => {
  console.log(item * 2); // 输出 2, 4, 6
});
```

### **2.2 map()**

- **用途**：对数组中的每个元素执行指定的函数，并返回由每次函数调用结果组成的新数组。
- **返回值**：新数组。
- **特点**：不修改原数组，适合链式操作。

```js
let arr = [1, 2, 3];
let doubled = arr.map(item => item * 2); // doubled 为 [2, 4, 6]
```

### **2.3 filter()**

- **用途**：根据条件过滤数组中的元素，返回满足条件的新数组。
- **返回值**：新数组。
- **特点**：不修改原数组。

```js
let arr = [1, 2, 3, 4, 5];
let evens = arr.filter(item => item % 2 === 0); // evens 为 [2, 4]
```

### **2.4 reduce()**

- **用途**：将数组中的元素累积为一个单一的值（例如求和、求乘积等）。
- **语法**：`array.reduce(callback, initialValue)`
- **返回值**：最终累加的结果。
- **特点**：不修改原数组。

```js
let arr = [1, 2, 3, 4];
let sum = arr.reduce((accumulator, current) => accumulator + current, 0); // sum 为 10
```

### **2.5 find() 与 findIndex()**

- **find()**
    - **用途**：查找并返回第一个满足条件的元素。
    - **返回值**：符合条件的元素，如果没有找到返回 `undefined`。

- **findIndex()**
    - **用途**：查找第一个满足条件的元素的索引。
    - **返回值**：索引值，如果没有找到返回 `-1`。

```js
let arr = [5, 12, 8, 130, 44];
let found = arr.find(item => item > 10); // found 为 12
let index = arr.findIndex(item => item > 10); // index 为 1
```

### **2.6 some() 与 every()**

- **some()**
    - **用途**：检查数组中是否至少有一个元素满足条件。
    - **返回值**：布尔值。

- **every()**
    - **用途**：检查数组中是否所有元素都满足条件。
    - **返回值**：布尔值。

```js
let arr = [1, 2, 3, 4, 5];
let hasEven = arr.some(item => item % 2 === 0); // true
let allEven = arr.every(item => item % 2 === 0); // false
```

---

## 3. 查找与检测

### **3.1 indexOf() 与 lastIndexOf()**

- **indexOf()**
    - **用途**：返回数组中指定元素第一次出现的索引。
    - **返回值**：索引值，没有找到则返回 `-1`。

- **lastIndexOf()**
    - **用途**：返回数组中指定元素最后一次出现的索引。
    - **返回值**：索引值，没有找到则返回 `-1`。

```js
let arr = [1, 2, 3, 2, 1];
arr.indexOf(2);     // 返回 1
arr.lastIndexOf(2); // 返回 3
```

### **3.2 includes()**

- **用途**：检查数组中是否包含指定的元素。
- **返回值**：布尔值。
- **特点**：比 indexOf 更直观。

```js
let arr = [1, 2, 3];
arr.includes(2); // true
arr.includes(4); // false
```

---

## 4. 排序与反转

### **4.1 sort()**

- **用途**：对数组元素进行排序。
- **语法**：`array.sort([compareFunction])`
    - **默认**：转换为字符串进行排序。
    - **自定义 compareFunction**：可以按照需求排序。
- **副作用**：直接修改原数组。

```js
let arr = [4, 2, 5, 1, 3];
arr.sort(); // 默认按字符串排序，结果可能不符合预期
// 正确的数字排序：
arr.sort((a, b) => a - b); // [1, 2, 3, 4, 5]
```

### **4.2 reverse()**

- **用途**：反转数组元素的顺序。
- **返回值**：反转后的数组。
- **副作用**：直接修改原数组。

```js
let arr = [1, 2, 3];
arr.reverse(); // [3, 2, 1]
```

---

## 5. 其他常用方法

### **5.1 join()**

- **用途**：将数组所有元素转换为一个字符串，各元素之间用指定的分隔符连接。
- **返回值**：字符串。
- **特点**：不会修改原数组。

```js
let arr = ['Hello', 'World'];
let str = arr.join(' '); // "Hello World"
```

### **5.2 flat() 与 flatMap()**

- **flat()**
    - **用途**：将嵌套数组展平成一维数组。
    - **语法**：`array.flat(depth)`，默认深度为 1。

- **flatMap()**
    - **用途**：先对数组每个元素执行映射操作，再将结果展平一级。
    - **返回值**：新数组。

```js
let arr = [1, [2, 3], [4, [5]]];
let flatArr = arr.flat();      // [1, 2, 3, 4, [5]]
let flatArr2 = arr.flat(2);      // [1, 2, 3, 4, 5]

let arr2 = [1, 2, 3];
let mapped = arr2.flatMap(x => [x, x * 2]); // [1, 2, 2, 4, 3, 6]
```

### **5.3 fill()**

- **用途**：用固定值填充数组的所有或部分元素。
- **语法**：`array.fill(value, start, end)`
- **副作用**：直接修改原数组。

```js
let arr = [1, 2, 3, 4];
arr.fill(0, 1, 3); // arr 变为 [1, 0, 0, 4]
```

---

## 总结

- **是否修改原数组**：
    - **会修改原数组的方法**：`push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`, `fill` 等。
    - **不修改原数组的方法**：`slice`, `concat`, `map`, `filter`, `reduce`, `join`, `flat` 等。

- **返回值的区别**：
    - 有的方法返回新数组（如 `map`, `filter`, `concat`），
    - 有的则返回其他类型（如 `reduce` 返回累积值，`forEach` 返回 `undefined`）。

- **适用场景**：
    - 当需要就地修改数组时，可选用 `push/pop/shift/unshift/splice` 等。
    - 当需要保留原数组、进行链式操作时，可选择 `slice`, `map`, `filter` 等。

| API                                      | 用途/描述                                                       | 关键特点与区别                                                     |
|------------------------------------------|----------------------------------------------------------------|-------------------------------------------------------------------|
| `push(...items)`                         | 在数组末尾添加元素，返回新长度                                      | 直接修改原数组                                                    |
| `pop()`                                  | 移除数组最后一个元素，返回该元素                                    | 直接修改原数组                                                    |
| `unshift(...items)`                      | 在数组开头添加元素，返回新长度                                      | 修改原数组，所有元素索引会发生变化                                   |
| `shift()`                                | 移除数组第一个元素，返回该元素                                      | 直接修改原数组                                                    |
| `splice(start, deleteCount, ...items)`   | 在任意位置删除、插入或替换元素，返回被删除元素的数组                    | 功能强大，但会直接修改原数组                                        |
| `slice(start, end)`                      | 提取数组的一个片段，返回新数组                                      | 不修改原数组，返回浅拷贝                                             |
| `concat(...items)`                       | 合并多个数组或值，返回新数组                                       | 不修改原数组，可用于合并数组或添加单个元素                             |
| `forEach(callback)`                      | 对每个元素执行回调函数，返回 `undefined`                            | 仅用于遍历，不支持链式调用                                          |
| `map(callback)`                          | 对数组每个元素执行回调并返回新数组                                   | 不改变原数组，适合链式调用                                          |
| `filter(callback)`                       | 根据条件过滤元素，返回符合条件的新数组                                | 不修改原数组                                                      |
| `reduce(callback, initialValue)`         | 将数组元素累计为一个单一值                                          | 返回单一值；可用于求和、求积等累计计算                                |
| `find(callback)`                         | 返回第一个满足条件的元素                                           | 未找到时返回 `undefined`                                           |
| `findIndex(callback)`                    | 返回第一个满足条件的元素索引，未找到返回 -1                           | 不修改原数组                                                      |
| `some(callback)`                         | 判断数组中是否至少有一个元素满足条件，返回布尔值                        | 不修改原数组                                                      |
| `every(callback)`                        | 判断数组中是否所有元素满足条件，返回布尔值                             | 不修改原数组                                                      |
| `indexOf(item, fromIndex)`               | 返回指定元素首次出现的索引，未找到返回 -1                              | 使用严格相等比较                                                  |
| `lastIndexOf(item, fromIndex)`           | 返回指定元素最后出现的索引，未找到返回 -1                              | 从数组末尾开始搜索                                                 |
| `sort(compareFunction)`                  | 对数组元素进行排序，返回排序后的数组                                  | 会修改原数组；默认按字符串顺序排序，需提供比较函数实现数值排序           |
| `reverse()`                              | 反转数组元素顺序，返回反转后的数组                                    | 直接修改原数组                                                    |
| `join(separator)`                        | 将数组所有元素连接成一个字符串，以指定分隔符分隔                         | 返回新字符串；不修改原数组                                           |
| `flat(depth)`                            | 将嵌套数组展平为一维数组，默认深度为 1                                  | 返回新数组，不改变原数组；可指定展开深度                               |
| `flatMap(callback)`                      | 先对每个元素映射，再将结果展平一层                                   | 组合了 `map` 和 `flat` 的功能                                       |
| `fill(value, start, end)`                | 用指定值填充数组中从 `start` 到 `end` 的位置                           | 直接修改原数组                                                    |

---

# 对象 API

## 1. 属性复制与对象创建

### 1.1 Object.assign(target, ...sources)
- **用途**：将一个或多个源对象的**可枚举**属性复制到目标对象，并返回目标对象。
- **特点**：
    - **浅拷贝**：仅复制第一层属性（嵌套对象不会深拷贝）。
    - 会**修改**目标对象。
- **示例**：

  ```js
  const target = { a: 1 };
  const source = { b: 2, c: 3 };
  const result = Object.assign(target, source);
  console.log(result); // { a: 1, b: 2, c: 3 }
  ```

### 1.2 Object.create(proto, [propertiesObject])
- **用途**：根据指定的原型对象 `proto` 创建一个新对象，同时可定义新对象的自有属性。
- **特点**：
    - 不复制现有属性，而是设置对象的**原型链**。
    - 适合实现继承和定制对象原型。
- **示例**：

  ```js
  const person = {
    greet() {
      console.log("Hello!");
    }
  };
  const student = Object.create(person);
  student.name = "Alice";
  student.greet(); // "Hello!"
  ```

---

## 2. 属性提取与遍历

### 2.1 Object.keys(obj)
- **用途**：返回对象自身所有**可枚举**属性的键名数组。
- **特点**：不包含原型链上的属性。

  ```js
  const obj = { a: 1, b: 2 };
  console.log(Object.keys(obj)); // ["a", "b"]
  ```

### 2.2 Object.values(obj)
- **用途**：返回对象自身所有**可枚举**属性的值数组。
- **特点**：与 `Object.keys` 顺序对应。

  ```js
  console.log(Object.values(obj)); // [1, 2]
  ```

### 2.3 Object.entries(obj)
- **用途**：返回对象自身所有**可枚举**属性的 `[key, value]` 键值对数组。
- **特点**：方便遍历和解构赋值。

  ```js
  console.log(Object.entries(obj)); // [["a", 1], ["b", 2]]
  ```

### 2.4 obj.hasOwnProperty(key)
- **用途**：检查对象自身是否拥有指定属性，而不检查原型链。
- **特点**：常用于防止继承属性的干扰。

  ```js
  console.log(obj.hasOwnProperty("a")); // true
  ```

---

## 3. 属性描述与定义

### 3.1 Object.getOwnPropertyDescriptor(obj, prop)
- **用途**：获取对象指定属性的描述符，包括 `value`、`writable`、`enumerable` 和 `configurable` 等信息。
- **特点**：用于了解属性的详细配置。

  ```js
  const descriptor = Object.getOwnPropertyDescriptor(obj, "a");
  console.log(descriptor);
  // 可能输出: { value: 1, writable: true, enumerable: true, configurable: true }
  ```

### 3.2 Object.defineProperty(obj, prop, descriptor)
- **用途**：在对象上定义或修改属性，并指定属性的行为（如只读、不可枚举）。
- **特点**：提供精细控制属性特性，不会自动覆盖原有属性配置。

  ```js
  const person = {};
  Object.defineProperty(person, "name", {
    value: "Bob",
    writable: false,
    enumerable: true,
    configurable: false
  });
  console.log(person.name); // "Bob"
  // 尝试修改 person.name 将不起作用
  person.name = "Alice";
  console.log(person.name); // 仍为 "Bob"
  ```

---

## 4. 对象状态控制

### 4.1 Object.freeze(obj)
- **用途**：冻结对象，使其**不能添加、删除或修改**属性。
- **特点**：
    - **浅冻结**：仅冻结对象第一层属性，嵌套对象仍可修改。
    - 一旦冻结，对象状态不可变。

  ```js
  const config = { port: 3000 };
  Object.freeze(config);
  // config.port = 5000; // 无效，修改失败
  ```

### 4.2 Object.seal(obj)
- **用途**：密封对象，禁止添加或删除属性，但允许修改已有属性的值。
- **特点**：
    - 同样是**浅密封**，适用于希望防止结构变化但允许数值更新的场景。

  ```js
  const settings = { theme: "dark" };
  Object.seal(settings);
  settings.theme = "light"; // 可修改
  // delete settings.theme; // 无效，删除失败
  ```

### 4.3 Object.isFrozen(obj) 与 Object.isSealed(obj)
- **用途**：分别判断对象是否被冻结或密封。
- **特点**：返回布尔值，只针对对象第一层状态。

  ```js
  console.log(Object.isFrozen(config)); // true
  console.log(Object.isSealed(settings)); // true
  ```

---

## 总结

- **复制与创建**：
    - 使用 `Object.assign` 来合并对象（浅拷贝），
    - 使用 `Object.create` 设定对象的原型，实现继承。

- **属性提取**：
    - `Object.keys`、`Object.values`、`Object.entries` 用于提取对象的自有可枚举属性，方便遍历。

- **属性描述与定义**：
    - `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty` 能够提供和设定属性的详细配置，从而实现只读、不可枚举等特性。

- **对象状态控制**：
    - `Object.freeze` 用于冻结对象，使其完全不可修改，
    - `Object.seal` 则允许修改现有属性值，但禁止添加或删除属性。

| API/方法                                        | 用途/描述                                                       | 关键特点与区别                                                      |
|------------------------------------------------|----------------------------------------------------------------|----------------------------------------------------------------------|
| `Object.assign(target, ...sources)`            | 将源对象的可枚举属性复制到目标对象，返回目标对象                        | 会修改目标对象；浅拷贝，不会复制嵌套对象                               |
| `Object.create(proto, [propertiesObject])`     | 创建一个以指定原型对象为原型的新对象，可同时定义属性                      | 用于实现继承；不会复制属性，只设定原型链                               |
| `Object.keys(obj)`                             | 返回对象自身的所有可枚举属性名数组                                   | 不包含继承属性；顺序与对象内部属性添加顺序一致                           |
| `Object.values(obj)`                           | 返回对象自身的所有可枚举属性值数组                                   | 与 `Object.keys` 对应，顺序相同                                        |
| `Object.entries(obj)`                          | 返回对象自身的 `[key, value]` 键值对数组                              | 方便遍历对象；顺序与 `Object.keys` 一致                                 |
| `obj.hasOwnProperty(key)`                      | 检查对象自身是否拥有指定属性（不检查原型链）                             | 实例方法，易受同名属性覆盖，必要时可用 `Object.prototype.hasOwnProperty.call(obj, key)` |
| `Object.getOwnPropertyDescriptor(obj, prop)`   | 获取指定属性的描述符，包含可写性、可枚举性、配置性及值等信息                | 用于精细控制对象属性                                               |
| `Object.defineProperty(obj, prop, descriptor)` | 定义或修改对象属性，并指定属性描述符                                  | 允许设定属性的行为，如只读、不可枚举等                                 |
| `Object.freeze(obj)`                           | 冻结对象，使其不能添加、删除或修改属性（浅冻结）                         | 返回冻结后的对象；仅冻结第一层，嵌套对象仍可修改                          |
| `Object.seal(obj)`                             | 密封对象，禁止添加或删除属性，但允许修改已有属性                         | 返回密封后的对象；同样是浅密封                                          |
| `Object.isFrozen(obj)`                         | 判断对象是否被冻结                                               | 返回布尔值                                                          |
| `Object.isSealed(obj)`                         | 判断对象是否被密封                                               | 返回布尔值                                                          |

---
