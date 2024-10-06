# 手写 Promise 静态方法 resolve、reject、all、allSettled、any、race、withResolvers 和 try

## 实现 Promise.resolve

`Promise.resolve()` 将给定的一个值转为 `Promise` 对象。

* 如果这个值是一个 `Promise` ，那么将返回这个 `Promise`；
* 如果这个值是 thenable（即带有 `then` 方法）对象，返回的 `Promise` 会“跟随”这个 thenable 对象，采用它的最终状态；
* 否则返回的 `Promise` 将以此值完成，即以此值执行 `resolve()` 方法（状态为 `fulfilled`）。

```javascript
Promise.myResolve = function (value) {
    if (value instanceof Promise) {
        return value;
    } else if (value instanceof Object && 'then' in value) {
        return new Promise((resolve, reject) => {
            value.then(resolve, reject);
        })
    }
    return new Promise((resolve) => {
        resolve(value);
    })
}
```

## 实现 Promise.reject

`Promise.reject()` 方法返回一个带有拒绝原因的 `Promise` 对象。

```javascript
Promise.myReject = function (reason) {
    return new Promise((resolve, reject) => {
        reject(reason);
    })
}
```

## 实现 Promise.all

`Promise.all()` 方法接收一个 `Promise` 的 `iterable` 类型的输入（注：`Array`、`Map`、`Set` 都属于 ES6 的 `iterable` 类型），并且只返回一个 `Promise` 实例，输入的所有 `Promise` 的 `resolve` 回调的结果是一个数组。

* `Promise.all` 等待所有 `Promise` 都完成（或第一个失败）；
* 如果传入的参数是一个空的可迭代对象，则返回一个已完成状态（`fulfilled`）的 `Promise`；
* 如果参数中包含非 `Promise` 值，这些值将被忽略，但仍然会被放在返回数组中（也就是如果参数里的某值不是 `Promise`，则需要原样返回在数组里）；
* 在任何情况下，`Promise.all` 返回的 `Promise` 的**完成状态**的结果都是一个数组，它包含所有的传入迭代参数对象的值（也包括非 `Promise` 值）。

```javascript
Promise.myAll = function (promises) {
    return new Promise((resolve, reject) => {
        if (Array.isArray(promises)) {
            let result = [];
            let count = 0;
            if (promises.length === 0) resolve(result);
            
            promises.forEach((item, index) => {
                Promise.resolve(item).then(
                    value => {
                        count++;
                        result[index] = value;
                        count === promises.length && resolve(result);
                    },
                    reason => {
                        reject(reason);
                    }
                )
            })
        } else {
            reject(new TypeError('Argument is not iterable'));
        }
    })
}
```

## 实现 Promise.allSettled

`Promise.allSettled()` 方法返回一个在所有给定的 `Promise` 都已经 `fulfilled` 或 `rejected` 后的 `Promise`，并带有一个对象数组，每个对象表示对应的 `Promise` 结果。

* 当你有多个彼此不依赖的异步任务成功完成时，或者你总是想知道每个 `Promise` 的结果时，通常使用它；
* 相比之下，`Promise.all()` 更适合彼此相互依赖或者在其中任何一个 `rejected` 时立即结束。

对于每个结果对象，都有一个 `status` 字符串。如果它的值为 `fulfilled`，则结果对象上存在一个 `value`。如果值为 `rejected`，则存在一个 `reason`。

```javascript
Promise.myAllSettled = function (promises) {
    return new Promise((resolve, reject) => {
        if (Array.isArray(promises)) {
            let result = [];
            let count = 0;
            if (promises.length === 0) resolve(result);

            promises.forEach((item, index) => {
                Promise.resolve(item).then(
                    value => {
                        count++;
                        result[index] = {
                            status: 'fulfilled',
                            value
                        }
                        count === promises.length && resolve(result);
                    },
                    reason => {
                        count++;
                        result[index] = {
                            status: 'rejected',
                            reason
                        }
                        count === promises.length && resolve(result);
                    }
                )
            })
        } else {
            reject(new TypeError('Argument is not iterable'));
        }
    })
}
```

## 实现 Promise.any

本质上，这个方法和 `Promise.all()` 是相反的。

`Promise.any()` 方法接收一个 `Promise` 可迭代对象，只要其中的一个 `Promise` 成功，就返回那个已经成功的 `Promise`。

如果可迭代对象中没有一个 `Promise` 成功，就返回一个失败的 `Promise` 和 `AggregateError` 类型的实例，它是 `Error` 的一个子类，用于把单一的错误集合在一起。

* 如果传入的参数是一个空的可迭代对象，则返回一个已失败（`rejected`）状态的 `Promise`；
* 如果传入的参数不包含任何 `Promise`，则返回一个异步完成的 `Promise`（将非 `Promise` 值，转换为 `Promise` 并当做成功）；
* 只要传入的迭代对象中的任何一个 `Promise` 变成成功（`fulfilled`）状态，或者其中的所有的 `Promise` 都失败，那么返回的 `Promise` 就会异步地变成成功/失败（`fulfilled`/`rejected`）状态。

```javascript
Promise.myAny = function (promises) {
    return new Promise((resolve, reject) => {
        if (Array.isArray(promises)) {
            let errors = [];
            let count = 0;
            if (promises.length === 0) reject(new AggregateError(errors, 'All promises were rejected'));

            promises.forEach(item => {
                Promise.resolve(item).then(
                    value => {
                        resolve(value);
                    },
                    reason => {
                        count++;
                        errors.push(reason);
                        count === promises.length && reject(new AggregateError(errors, 'All promises were rejected'));
                    }
                )
            })
        } else {
            reject(new TypeError('Argument is not iterable'));
        }
    })
}
```

## 实现 Promise.race

`Promise.race()` 方法返回一个 `Promise`，一旦迭代器中的某个 `Promise` 解决或拒绝，返回的 `Promise` 就会解决或拒绝。

如果传的迭代是空的，则返回的 `Promise` 将永远等待。

```javascript
Promise.myRace = function (promises) {
    return new Promise((resolve, reject) => {
        if (Array.isArray(promises)) {
            promises.forEach(item => {
                Promise.resolve(item).then(resolve, reject);
            })
        } else {
            reject(new TypeError('Argument is not iterable'));
        }
    })
}
```

## 实现 Promise.withResolvers

`Promise.withResolvers()` 方法返回一个对象，其包含一个新的 `Promise` 对象和两个函数，用于解决或拒绝它，对应于传入给 `Promise()` 构造函数执行器的两个参数。

```javascript
Promise.myWithResolvers = function () {
  let resolve, reject;
  const promise = new Promise((res, rej) => {
    resolve = res;
    reject = rej;
  })
  return { promise, resolve, reject }
}
```

在没有 `Promise.withResolvers` 之前，你可能实现了这样的代码：

```javascript
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
})

let val = Math.random();
val > 0.5 ? resolve(val) : reject(val);
promise.then(res => {
    console.log(`${res}大于0.5`);
}).catch(err => {
    console.log(`${err}小于等于0.5`);
})
```

利用 `Promise.withResolvers` 可以这样实现：

```javascript
const { promise, resolve, reject } = Promise.withResolvers();

let val = Math.random();
val > 0.5 ? resolve(val) : reject(val);
promise.then(res => {
    console.log(`${res}大于0.5`);
}).catch(err => {
    console.log(`${err}小于等于0.5`);
})
```

## 实现 Promise.try

`Promise.try()` 方法接受任何类型的回调（同步或异步，返回结果或抛出错误），并将其结果封装在 `Promise` 中。该方法仍是**实验性**技术。

```javascript
Promise.myTry = function (fn) {
    return new Promise((resolve, reject) => {
        try {
            resolve(fn());
        } catch (error) {
            reject(error);
        }
    })
}
```

实际开发中，经常遇到一种情况：不知道或者不想区分，函数 `fn` 是同步函数还是异步操作，但是想用 `Promise` 来处理它。因为这样就可以不管 `fn` 是否包含异步操作，都用 `then` 方法指定下一步流程，用 `catch` 方法处理 `fn` 抛出的错误。一般就会采用下面的写法：

```javascript
Promise.resolve().then(fn)
```

上面的写法有一个缺点，就是如果 `fn` 是同步函数，那么它会在本轮事件循环的末尾执行。

```javascript
const fn = () => console.log('now');
Promise.resolve().then(fn);
console.log('next');
// next
// now
```

那么有没有一种方法，让同步函数同步执行，异步函数异步执行，并且让它们具有统一的 API 呢？这是可以的，并且还有两种写法。第一种写法是用 `async` 函数来写：

```javascript
const fn = () => console.log('now');
(async () => fn())();
console.log('next');
// now
// next
```

上面代码中，第二行是一个立即执行的匿名函数，会立即执行里面的 `async` 函数，因此如果 `fn` 是同步的，就会得到同步的结果；如果 `fn` 是异步的，就可以用 `then` 指定下一步。需要注意的是，`async () => fn()` 会吃掉 `fn()` 抛出的错误。所以，如果想捕获错误，要使用 `promise.catch` 方法：

```javascript
(async () => fn())()
.then(...)
.catch(...)
```

第二种写法是使用 `new Promise()`：

```javascript
const fn = () => console.log('now');
(
  () => new Promise(
    resolve => resolve(fn())
  )
)();
console.log('next');
// now
// next
```

上面代码也是使用立即执行的匿名函数，执行 `new Promise()`。这种情况下，同步函数也是同步执行的。

鉴于这是一个很常见的需求，所以现在有一个提案，提供 `Promise.try` 方法替代上面的写法：

```javascript
const fn = () => console.log('now');
Promise.try(fn);
console.log('next');
// now
// next
```

使用 `Promise.try` 有许多好处，其中一点就是可以更好地管理异常。例如，一个 `database.users.get()` 方法如果抛出异步错误，可以用 `catch` 方法捕获。但它还可能抛出同步错误，这时不得不用 `try...catch` 去捕获：

```javascript
try {
  database.users.get({id: userId})
  .then(...)
  .catch(...)
} catch (e) {
  // ...
}
```

上面这样的写法就很笨拙了，这时就可以统一用 `promise.catch()` 捕获所有同步和异步的错误：

```javascript
Promise.try(() => database.users.get({id: userId}))
  .then(...)
  .catch(...)
```

## 参考

1. [Promise.resolve (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)
2. [Promise.reject (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)
3. [Promise.all (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
4. [Promise.allSettled (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)
5. [Promise.any (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)
6. [Promise.race (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
7. [Promise.withResolvers (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers)
8. [Promise.try (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/try)
9.  [JavaScript 深入系列之 Promise 全部实例方法和静态方法的模拟实现，包括处于 TC39 第四阶段草案的 Promise.any()](https://github.com/yuanyuanbyte/Blog/issues/126)
10. [手写Promise.withResolvers()](https://lsnx.top/article/promise-withresolvers)
11. [全新API——Promise.withResolvers](https://juejin.cn/post/7332387986536464399)
12. [阮一峰 ECMAScript 6 (ES6) 标准入门教程 第三版](https://www.bookstack.cn/read/es6-3rd/spilt.13.docs-promise.md)
13. [手写Promise.try方法](https://wenku.csdn.net/answer/4ymve7gh23)