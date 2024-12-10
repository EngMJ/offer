# 设计模式

设计模式一般根据其用途和结构可以分为三大类：**创建型模式**、**结构型模式**和**行为型模式**。

### 1. **创建型模式 (Creational Patterns)**
创建型模式旨在帮助创建对象时提高灵活性和可重用性，减少代码的耦合度。它们关注如何实例化对象，确保在创建对象时遵循特定的规则。

**常见的创建型模式**：
- **单例模式 (Singleton Pattern)**：确保类只有一个实例，并提供一个全局访问点。
- **工厂模式 (Factory Pattern)**：定义一个接口，用于创建对象，子类决定实例化哪一个类。
- **抽象工厂模式 (Abstract Factory Pattern)**：创建一组相关或依赖的对象，而不需要指定具体类。
- **建造者模式 (Builder Pattern)**：使用多个步骤构建复杂对象，并提供一个流畅的接口来定制创建过程。
- **原型模式 (Prototype Pattern)**：通过复制现有对象来创建新的对象，而不需要通过构造函数。

### 2. **结构型模式 (Structural Patterns)**
结构型模式关注如何将类或对象组合成更大的结构，以实现更高效的功能和复用。它们通过合理的组合来扩展系统功能而不影响现有代码。

**常见的结构型模式**：
- **适配器模式 (Adapter Pattern)**：将一个类的接口转换为客户端所期望的接口，使不兼容的接口能够协同工作。
- **装饰器模式 (Decorator Pattern)**：动态地给对象添加额外的职责，而不改变对象的结构。
- **桥接模式 (Bridge Pattern)**：将抽象部分与实现部分分离，使它们可以独立地变化。
- **组合模式 (Composite Pattern)**：将对象组合成树形结构以表示“部分-整体”的层次结构，使客户端可以统一对待单个对象和组合对象。
- **外观模式 (Facade Pattern)**：为一组接口提供一个统一的接口，使子系统更加易于使用。
- **享元模式 (Flyweight Pattern)**：通过共享对象来减少内存使用，并提高程序的性能。
- **代理模式 (Proxy Pattern)**：为另一个对象提供一个代理以控制对该对象的访问，常用于懒加载、权限控制、缓存等场景。

### 3. **行为型模式 (Behavioral Patterns)**
行为型模式关注对象之间如何协作和交互，帮助定义对象之间的责任和通信方式，从而实现灵活的行为。

**常见的行为型模式**：
- **观察者模式 (Observer Pattern)**：定义对象间的一对多依赖关系，使得一个对象的状态变化时，所有依赖它的对象都能自动收到通知。
- **策略模式 (Strategy Pattern)**：定义一系列算法，将每个算法封装起来，并使它们可以互换，客户端可以根据需要选择合适的算法。
- **命令模式 (Command Pattern)**：将请求封装为一个对象，从而允许用户参数化请求的对象、队列请求或日志请求。
- **状态模式 (State Pattern)**：允许对象在内部状态改变时改变其行为，使得对象看起来就像改变了其类一样。
- **访问者模式 (Visitor Pattern)**：将操作封装在一个访问者类中，可以在不改变元素的类的情况下，增加新的操作。
- **模板方法模式 (Template Method Pattern)**：定义一个操作的骨架，而将一些步骤延迟到子类中，使得子类可以在不改变算法结构的情况下，重新定义算法的某些特定步骤。
- **中介者模式 (Mediator Pattern)**：通过引入一个中介者对象来减少多个对象间的直接通信和依赖。
- **备忘录模式 (Memento Pattern)**：保存对象的状态，以便在需要时恢复到原来的状态。
- **迭代器模式 (Iterator Pattern)**：提供一种方法顺序访问集合中的元素，而无需暴露集合的内部表示。
- **解释器模式 (Interpreter Pattern)**：为语言提供一个解释器，使得能够解释和执行语言的句子。


# 前端常用模式:

### 1. **单例模式 (Singleton Pattern)**
**示例**：实现一个单例的日志服务。
```javascript
class Logger {
  constructor() {
    if (!Logger.instance) {
      this.logs = [];
      Logger.instance = this;
    }
    return Logger.instance;
  }

  log(message) {
    this.logs.push(message);
    console.log(message);
  }
}

const logger = new Logger();
Object.freeze(logger); // 防止再次实例化

export default logger;
```

**使用**：
```javascript
import logger from './Logger';
logger.log('This is a singleton pattern example.');
```

### 2. **观察者模式 (Observer Pattern)**
**示例**：实现简单的观察者模式。
```javascript
class Subject {
  constructor() {
    this.observers = [];
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  notifyObservers(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  update(data) {
    console.log('Observer received data:', data);
  }
}

const subject = new Subject();
const observer1 = new Observer();
subject.addObserver(observer1);

subject.notifyObservers('Hello, observers!');
```

### 3. **发布/订阅模式 (Publisher/Subscriber Pattern)**
**示例**：实现简单的发布/订阅模式。
```javascript
class EventBus {
  constructor() {
    this.events = {};
  }

  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  publish(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

const eventBus = new EventBus();
eventBus.subscribe('message', (data) => console.log('Received message:', data));

eventBus.publish('message', 'Hello, subscribers!');
```

### 4. **模块化模式 (Module Pattern)**
**示例**：使用 ES6 模块化。
```javascript
// utils.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// main.js
import { add, subtract } from './utils';
console.log(add(2, 3)); // 输出: 5
console.log(subtract(5, 2)); // 输出: 3
```

### 5. **工厂模式 (Factory Pattern)**
**示例**：实现一个简单的工厂模式来创建不同的形状。
```javascript
class Circle {
  draw() {
    console.log('Drawing a Circle');
  }
}

class Square {
  draw() {
    console.log('Drawing a Square');
  }
}

class ShapeFactory {
  static createShape(shapeType) {
    switch (shapeType) {
      case 'circle':
        return new Circle();
      case 'square':
        return new Square();
      default:
        throw new Error('Unknown shape type');
    }
  }
}

const circle = ShapeFactory.createShape('circle');
circle.draw(); // 输出: Drawing a Circle

const square = ShapeFactory.createShape('square');
square.draw(); // 输出: Drawing a Square
```

### 6. **策略模式 (Strategy Pattern)**
**示例**：实现一个简单的支付策略。
```javascript
class PayStrategy {
  pay(amount) {
    throw new Error('PayStrategy is abstract');
  }
}

class CreditCardPayment extends PayStrategy {
  pay(amount) {
    console.log(`Paid ${amount} using Credit Card`);
  }
}

class PayPalPayment extends PayStrategy {
  pay(amount) {
    console.log(`Paid ${amount} using PayPal`);
  }
}

class PaymentContext {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  pay(amount) {
    this.strategy.pay(amount);
  }
}

const context = new PaymentContext(new CreditCardPayment());
context.pay(100); // 输出: Paid 100 using Credit Card

context.setStrategy(new PayPalPayment());
context.pay(200); // 输出: Paid 200 using PayPal
```

### 7. **命令模式 (Command Pattern)**
**示例**：实现一个简单的命令模式。
```javascript
class Command {
  execute() {
    throw new Error('Command execute method is abstract');
  }
}

class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }

  execute() {
    this.light.turnOn();
  }
}

class Light {
  turnOn() {
    console.log('The light is on');
  }
}

const light = new Light();
const lightOnCommand = new LightOnCommand(light);

lightOnCommand.execute(); // 输出: The light is on
```

### 8. **代理模式 (Proxy Pattern)**
**示例**：实现一个简单的代理模式来延迟加载。
```javascript
class RealImage {
  constructor(filename) {
    this.filename = filename;
    this.loadImageFromDisk();
  }

  loadImageFromDisk() {
    console.log(`Loading ${this.filename}`);
  }

  display() {
    console.log(`Displaying ${this.filename}`);
  }
}

class ProxyImage {
  constructor(filename) {
    this.realImage = null;
    this.filename = filename;
  }

  display() {
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

const image = new ProxyImage('example.jpg');
image.display(); // 输出: Loading example.jpg, Displaying example.jpg
image.display(); // 输出: Displaying example.jpg
```

### 9. **装饰器模式 (Decorator Pattern)**
**示例**：实现一个简单的装饰器模式来扩展功能。
```javascript
class Coffee {
  cost() {
    return 5;
  }
}

class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost() + 2;
  }
}

const coffee = new Coffee();
console.log(`Cost of coffee: $${coffee.cost()}`);

const milkCoffee = new MilkDecorator(coffee);
console.log(`Cost of milk coffee: $${milkCoffee.cost()}`); // 输出: Cost of milk coffee: $7
```
