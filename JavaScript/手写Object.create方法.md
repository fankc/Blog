# 手写 Object.create 方法

## Object.create 原理

`Object.create` 静态方法以一个现有对象作为原型，创建一个新对象。

语法：

```javascript
Object.create(proto, propertiesObject)
```

* `proto`：新创建对象的原型对象。如果 `proto` 既不是 `null`，也不是 `Object`，则抛出 `TypeError`。
* `propertiesObject`：可选的。如果该参数被指定且不为 `undefined`，则该传入对象的自有可枚举属性将为新创建的对象添加具有对应属性名称的属性描述符。这些属性对应于 `Object.defineProperties()` 的第二个参数。

`Object.create` 可以对对象创建过程进行精细的控制。实际上，字面量初始化对象语法是 `Object.create` 的一种语法糖。

```javascript
o = Object.create(Object.prototype); // 等价于：o = {};
o = Object.create(null); // 等价于：o = { __proto__: null };
```

使用 `Object.create` 可以创建具有指定原型和某些属性的对象。第二个参数将键映射到属性描述符，这意味着你可以控制每个属性的可枚举性、可配置性等，而这在字面量初始化对象语法中是做不到的。

```javascript
o = Object.create(Object.prototype, {
  // foo 是一个常规数据属性
  foo: {
    value: "hello",
    writable: true,
    enumerable: true,
    configurable: true
  },
  // bar 是一个访问器属性
  bar: {
    configurable: false,
    get() {
      return 10;
    },
    set(value) {
      console.log("Setting `o.bar` to", value);
    },
  },
});
```

可以使用 `Object.create` 来模拟 `new` 运算符的行为。

```javascript
function Constructor() {}
o = new Constructor();
// 等价于：o = Object.create(Constructor.prototype);
```

但是，如果 `Constructor` 函数中有实际的初始化代码，此时仅靠 `Object.create` 方法无法模拟 `new`，具体模拟方法可查看《[手写 new 运算符](./手写new运算符.md)》。

## 用 Object.create 实现类式继承

下面的例子演示了如何使用 `Object.create` 来实现类式继承。这是一个所有版本 JavaScript 都支持的单继承。

```javascript
// Shape——父类
function Shape() {
  this.x = 0;
  this.y = 0;
}
// 父类方法
Shape.prototype.move = function (x, y) {
  this.x += x;
  this.y += y;
  console.info("Shape moved.");
};
// Rectangle——子类
function Rectangle() {
  Shape.call(this); // 调用父类构造函数。
}
// 子类继承父类
Rectangle.prototype = Object.create(Shape.prototype, {
  // 如果不将 Rectangle.prototype.constructor 设置为 Rectangle，
  // 它将采用 Shape（父类）的 prototype.constructor。
  // 为避免这种情况，我们将 prototype.constructor 设置为 Rectangle（子类）。
  constructor: {
    value: Rectangle,
    enumerable: false,
    writable: true,
    configurable: true,
  },
});

const rect = new Rectangle();
console.log("rect 是 Rectangle 类的实例吗？", rect instanceof Rectangle); // true
console.log("rect 是 Shape 类的实例吗？", rect instanceof Shape); // true
rect.move(1, 1); // 打印 'Shape moved.'
```

## 手写 Object.create

```javascript
function create(proto, propertiesObject) {
    // 创建一个空的构造函数
    function F() {};
    // 将传入的对象作为这个构造函数的 `prototype` 属性
    F.prototype = proto;
    // 利用构造函数创建实例对象
    let obj = new F();
    if (propertiesObject) {
        // 设置实例对象的属性
        Object.defineProperties(obj, propertiesObject);
    }
    return obj;
}
```

测试代码：

```javascript
const person = {
  isHuman: false,
  printIntroduction: function () {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
  },
};
const me = create(person);
me.name = 'Matthew'; // 在 me 对象上设置 name 属性
me.isHuman = true; // 覆盖继承的属性
me.printIntroduction(); // My name is Matthew. Am I human? true
person.printIntroduction(); // My name is undefined. Am I human? false
```

## 参考

1. [Object.create (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
2. [Object.defineProperties (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)
3. [JavaScript 深入系列之 Object.create 的模拟实现](https://github.com/yuanyuanbyte/Blog/issues/114)