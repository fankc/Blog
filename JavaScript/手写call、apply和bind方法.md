# 手写 call、apply 和 bind 方法

## apply 和 call

在 JavaScript 中，`call` 和 `apply` 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 `this` 的指向。

**`call` 的语法：**

```javascript
func.call(thisArg, arg1, arg2, ...)
```

* `thisArg`：在函数运行时使用的 `this` 值。如果省略该参数，则默认为 `undefined`。在非严格模式下，`null` 和 `undefined` 将被替换为全局对象。
* `arg1, arg2, ...`：可选的。指定的参数列表。

**`apply` 的语法：**

```javascript
func.apply(thisArg, [argsArray])
```

* `thisArg`：在函数运行时使用的 `this` 值。如果省略该参数，则默认为 `undefined`。在非严格模式下，`null` 和 `undefined` 将被替换为全局对象。
* `argsArray`：可选的。一个数组或类数组对象，用于指定调用函数时的参数，或者如果不需要向函数提供参数，则为 `null` 或 `undefined`。

**两者的区别：**

`call` 和 `apply` 只有一个区别，`call` 方法接受的是参数列表，而 `apply` 方法接受的是一个参数数组。

### 手写 call 方法

实现的步骤可以分为：

* 将函数设为对象的属性
* 执行该函数
* 删除该函数

```javascript
Function.prototype.myCall = function (context) {
    // 传入的 this 为 null 或 undefined 时要赋值为 window 或 global
    if (!context) {
        context = typeof window === 'undefined' ? global : window;
    }
    // 获取调用 call 的函数，设置为 context 对象的属性
    context.fn = this;
    // 用 slice 方法取第二个到最后一个参数（获取除了 this 指向对象以外的参数）
    const rest = [...arguments].slice(1);
    // 执行函数，隐式绑定，当前函数的 this 指向了 context
    const result = context.fn(...rest);
    // 删除函数
    delete context.fn;
    return result;
}
```

测试代码：

```javascript
var foo = { name: 'Selina' }
var name = 'Chirs';
function bar(job, age) {
    console.log(this.name, job, age);
}
bar.myCall(foo, 'programmer', 20);
// Selina programmer 20
bar.myCall(null, 'teacher', 25);
// 浏览器环境：Chirs teacher 25；node 环境：undefined teacher 25    
```

### 手写 apply 方法

实现步骤与 call 类似，需要注意对数组或类数组的判断。

```javascript
// 《JavaScript权威指南》中给出的类数组判断方法
function isArrayLike(o) {
    if(o &&                                    // o不是null、undefined等
       typeof o === 'object' &&                // o是对象
       isFinite(o.length) &&                   // o.length是有限数值
       o.length >= 0 &&                        // o.length为非负值
       o.length === Math.floor(o.length) &&    // o.length是整数
       o.length < 4294967296)                  // o.length < 2^32
       return true
    else
       return false
}

Function.prototype.myApply = function (context, rest) {
    // 传入的 this 为 null 或 undefined 时要赋值为 window 或 global
    if (!context) {
        context = typeof window === 'undefined' ? global : window;
    }
    // 获取调用 call 的函数，设置为 context 对象的属性
    context.fn = this;
    let result;
    // 判断第二个参数是否为数组或类数组对象
    if (isArrayLike(rest)) {
        rest = Array.from(rest);
        // 执行函数，隐式绑定，当前函数的 this 指向了 context
        result = context.fn(...rest);
    } else {
        throw new TypeError('CreateListFromArrayLike called on non-object');
    }
    // 删除函数
    delete context.fn;
    return result;
}
```

测试代码：

```javascript
var foo = { name: 'Selina' }
var name = 'Chirs';
function bar(job, age) {
    console.log(this.name, job, age);
}
bar.myApply(foo, ['programmer', 20]);
// Selina programmer 20
bar.myApply(null, ['teacher', 25]);
// 浏览器环境：Chirs teacher 25；node 环境：undefined teacher 25
```

## bind

`bind` 函数创建一个新的绑定函数。调用绑定函数会执行其所包装的函数，也称为目标函数。

**`bing` 的语法：**

```javascript
func.bind(thisArg, arg1, arg2, ...)
```

* `thisArg`：在函数运行时使用的 `this` 值。如果省略该参数，则默认为 `undefined`。在非严格模式下，`null` 和 `undefined` 将被替换为全局对象。如果使用 `new` 运算符构造绑定函数，则忽略该值。
* `arg1, arg2, ...`：可选的。指定的参数列表。在调用绑定函数时，插入到传入绑定函数的参数前的参数。

`bind` 和 `call`/`apply` 有两个主要区别：

* 一个函数被 `call`/`apply` 时，会被立即执行，而 `bind` 会返回一个新函数，不会立即执行。
* `bind` 的参数列表可以分多次传入（支持柯里化），`call` 和 `apply` 则必须一次性传入所有参数。

> 在绑定函数上调用 `bind` 进一步进行绑定，新绑定的 `thisArg` 值会被忽略：

```javascript
function log(...args) {
  console.log(this, ...args);
}
const boundLog = log.bind("this value", 1, 2);
const boundLog2 = boundLog.bind("new this value", 3, 4);
boundLog2(5, 6); // "this value", 1, 2, 3, 4, 5, 6
```

> 如果目标函数是可构造的，绑定函数也可以使用 `new` 运算符进行构造。前置的参数会像往常一样传递给目标函数，而提供的 `this` 值会被忽略（因为构造函数会准备自己的 `this`）。`new.target`、`instanceof`、`this` 等都如预期工作，就好像构造函数从未被绑定一样。唯一的区别是它不能再用于 `extends`：

```javascript
class Base {
  constructor(...args) {
    console.log(new.target === Base, args);
  }
}
const BoundBase = Base.bind(null, 1, 2);
new BoundBase(3, 4); // true, [1, 2, 3, 4]
```

### 手写 bind 方法

```javascript
Function.prototype.myBind = function (context) {
    // 保存目标函数（即调用 myBind 的函数）的 this
    const self = this;
    // 获取调用 myBind 时传入的除了 context 对象以外的参数
    const args = [...arguments].slice(1);
    const bindFunc = function () {
        // restArgs 是调用绑定函数时传入的参数
        const restArgs = [...arguments];
        // 若通过 new 运算符调用绑定函数，则目标函数的 prototype 属性在 this 的原型链上
        // 此时传入的 context 对象无效，this 指向绑定函数构造的对象实例
        return self.apply(this instanceof self ? this : context, args.concat(restArgs));
    }
    bindFunc.prototype = this.prototype;
    return bindFunc;
}
```

测试代码：

```javascript
var name = 'Jack';
var Yve = { name: 'Yvette' };
function person(age, job, gender) {
    this.work = '福报'; // 实例属性
    console.log(this.name, age, job, gender);
}
person.prototype.clockIn = function () {
    console.log(996);
}
person(22, 'engineer', 'female'); // Jack 22 engineer female
var bindYve = person.myBind(Yve, 22, 'engineer');
bindYve('female'); // Yvette 22 engineer female
var obj = new bindYve('female'); // undefined 22 'engineer' 'female'
console.log(obj.work); // 福报
obj.clockIn(); // 996
```

## 参考

1. [Function.prototype.call (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
2. [Function.prototype.apply (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
3. [JavaScript 基础系列之 call、apply 和 bind 方法的用法、区别和使用场景](https://github.com/yuanyuanbyte/Blog/issues/115)
4. [JavaScript 深入系列之 call 和 apply 方法的模拟实现](https://github.com/yuanyuanbyte/Blog/issues/109)
5. [JS 类数组对象（ArrayLike Object）的判断](https://segmentfault.com/a/1190000007315013)
6. [Function.prototype.bind (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
7. [JavaScript 深入系列之 bind 方法的模拟实现](https://github.com/yuanyuanbyte/Blog/issues/136)