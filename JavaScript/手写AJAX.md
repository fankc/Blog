# 手写 AJAX

## 什么是 AJAX？

AJAX 不是 JavaScript 的规范，它只是一个哥们“发明”的缩写：Asynchronous JavaScript and XML，意思就是用 JavaScript 执行异步网络请求。

AJAX 是一种用于创建快速动态网页的技术。

通过在后台与服务器进行少量数据交换，AJAX 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

传统的网页（不使用 AJAX）如果需要更新内容，必需重载整个网页。

## 手写 AJAX 步骤详解

### 1. 创建 XMLHttpRequest 对象

所有现代浏览器均内建 `XMLHttpRequest` 对象。

```javascript
var xhr = new XMLHttpRequest();
```

### 2. 设置状态监听函数

下面是 `XMLHttpRequest` 对象的三个重要的属性：

* `readyState`：存有 `XMLHttpRequest` 的状态。从 `0` 到 `4` 发生变化。
  * `0`：请求未初始化
  * `1`：服务器连接已建立
  * `2`：请求已接收
  * `3`：请求处理中
  * `4`：请求已完成，且响应已就绪

* `onreadystatechange`：存储函数（或函数名），每当 `readyState` 属性改变时，就会调用该函数。

* `status`：HTTP 响应码
  * `200`："OK"
  * `404`："Not Found"
  * 等

在 `onreadystatechange` 事件中，我们规定当服务器响应已做好被处理的准备时所执行的任务。

如需获得来自服务器的响应，请使用 `XMLHttpRequest` 对象的 `responseText` 或 `responseXML` 属性。

* `responseText`：获得字符串形式的响应数据。
* `responseXML`：获得 XML 形式的响应数据。

如需进行错误监控处理，请使用 `XMLHttpRequest` 对象的 `statusText` 属性，该属性包含状态信息，比如 "OK" 和 "Not Found"。如果服务器没有返回状态提示，该属性的值默认为 "OK"。

```javascript
xhr.onreadystatechange = function () {
    if (xhr.readyState !== 4) return;
    if (xhr.status === 200) {
        // success: do something
        console.log(xhr.responseText);
    } else {
        // fail: do something else
        new Error(xhr.statusText);
    }
};
```

### 3. 向服务器发送请求

如需将请求发送到服务器，我们使用 `XMLHttpRequest` 对象的 `open()` 和 `send()` 方法：

```javascript
xhr.open("GET", url, true);
xhr.send();
```

`open(method, url, async)` 方法规定请求的类型、URL 以及是否异步处理请求：

* `method`：请求的类型，`GET` 或 `POST`。
* `url`：文件在服务器上的位置。
* `async`：`true`（异步）或 `false`（同步）。

`send(string)` 方法将请求发送到服务器：

* `string`：仅用于 `POST` 请求。

如果需要像 HTML 表单那样 `POST` 数据，请使用 `setRequestHeader()` 来添加 HTTP 头。然后在 `send()` 方法中规定您希望发送的数据：

```javascript
xhr.open("POST", url, true); 
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded"); 
xhr.send("fname=Henry&lname=Ford");
```

## 完整示例

```javascript
// 1.创建 XMLHttpRequest 对象
var xhr = new XMLHttpRequest();
// 2.设置状态监听函数
xhr.onreadystatechange = function () {
    if (xhr.readyState !== 4) return;
    if (xhr.status === 200) {
        // success: do something
        console.log(xhr.responseText);
    } else {
        // fail: do something else
        new Error(xhr.statusText);
    }
}
// 3.向服务器发送请求
xhr.open('GET', url, true);
xhr.send();
```

### 使用 Promise 封装 AJAX

```javascript
function ajax(method, url, data) {
    var xhr = new XMLHttpRequest();
    return new Promise(function (resolve, reject) {
        xhr.onreadystatechange = function () {
            if (xhr.readyState !== 4) return;
            if (xhr.status === 200) {
                resolve(xhr.responseText);
            } else {
                reject(xhr.statusText);
            }
        };
        xhr.open(method, url, true);
        xhr.send(data);
    });
}
```

使用上面封装好的 ajax 发起一个请求：

```javascript
ajax('GET', url).then(function (data) {
    console.log(data);
}).catch(function (err) {
    console.log(err)
});
```

## 参考

1. [AJAX (菜鸟教程)](https://www.runoob.com/ajax/ajax-tutorial.html)
2. [AJAX (廖雪峰)](https://liaoxuefeng.com/books/javascript/browser/ajax/index.html)
3. [JavaScript 深入系列之实现 AJAX，以及使用 Promise 封装 AJAX 请求](https://github.com/yuanyuanbyte/Blog/issues/108)