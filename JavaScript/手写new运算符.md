# 手写 new 运算符

## new 的原理

`new` 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

当使用 `new` 关键字调用函数时，该函数将被用作构造函数。`new` 将执行以下操作：

1. 创建一个空的简单 JavaScript 对象。为方便起见，我们称之为 `newInstance`（即 `{}`）；
2. 如果构造函数的 `prototype` 属性是一个对象，则将 `newInstance` 的 `__proto__` 指向构造函数的 `prototype` 属性，否则 `newInstance` 将保持为一个普通对象，其 `__proto__` 为 `Object.prototype`。
3. 使用给定参数执行构造函数，并将 `newInstance` 绑定为 `this` 的上下文（在构造函数中的所有 `this` 引用都指向 `newInstance`）。
4. 如果构造函数返回**非原始值**，则该返回值成为整个 `new` 表达式的结果。否则返回 `newInstance`。

## 手写 new

```javascript
function myNew(constructor, ...args) {
    // 对应上面的步骤 1 和 2
    const obj = Object.create(constructor.prototype);
    // 对应上面的步骤 3
    const res = constructor.apply(obj, args);
    // 对应上面的步骤 4
    return res instanceof Object ? res : obj;
}
```

测试代码：

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
       console.log(this.name)
    }
}

// var person = new Person('Nicholas', 29, 'Front-end developer'); 
var person = myNew(Person, 'Nicholas', 29, 'Front-end developer');

console.log(person.name) // Nicholas
person.sayName();        // Nicholas
console.log(person.__proto__ === Person.prototype);   // true
```

## 参考

1. [new (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
2. [JavaScript 深入系列之 new 操作符的模拟实现](https://github.com/yuanyuanbyte/Blog/issues/110)