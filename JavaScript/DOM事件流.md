# DOM 事件流

## 什么是事件流？

事件流是事件在目标元素和顶层元素间的触发顺序。在早期，微软和网景实现了相反的事件流，网景主张捕获方式，微软主张冒泡方式:

1. **事件捕获**：事件由最顶层逐级向下传播，直至到达目标元素。
2. **事件冒泡**：事件由第一个被触发的元素接收，然后逐级向上传播。

后来 `w3c` 规定 `DOM` 事件流分为三个阶段：

1. **捕获阶段**：事件从最顶层元素 `window` 一直传递到目标元素的父元素。
2. **目标阶段**：事件到达目标元素。如果事件指定不冒泡。那就会在这里中止。
3. **冒泡阶段**：事件从目标元素父元素向上逐级传递直到最顶层元素 `window`。

`EventTarget.addEventListener()` 方法将指定的监听器注册到目标元素上, 当该对象触发指定的事件时, 指定的回调函数就会被执行。`addEventListener`有三个参数：

```javascript
element.addEventListener(event, function, useCapture)
```

* `event`：必须。字符串，指定事件名。
* `function`：必须。指定要事件触发时执行的函数。
* `useCapture`：可选。布尔值，指定事件是否在捕获或冒泡阶段执行。`true` — 事件句柄在捕获阶段执行；`false` — 事件句柄在冒泡阶段执行。默认 `false`

## 阻止事件冒泡（捕获）

事件冒泡（捕获）有时会产生问题，但有一种方法可以防止这些问题。`Event` 对象有一个可用的函数，叫做 `stopPropagation()`，当在一个事件处理器中调用时，可以防止事件向任何其他元素传递。

例如：

```javascript
const video = document.querySelector("video");
video.addEventListener("click", (event) => {
  event.stopPropagation();
  video.play();
});
```

## 事件委托（事件代理）

当我们想在大量的子元素上绑定事件时，我们可以在它们的父元素上设置事件监听器，让发生在子元素上的事件冒泡到它们的父元素上，而不必在每个子元素上单独设置事件监听器。

事件委托具有如下优点：

* `document` 对象随时可用，任何时候都可以给它添加事件处理程序（不用等待`DOMContentLoaded` 或 `load` 事件）。这意味着只要页面渲染出可点击的元素，就可以无延迟地起作用。
* 节省花在设置页面事件处理程序上的时间。只指定一个事件处理程序既可以节省 `DOM` 引用，也可以节省时间。
* 减少整个页面所需的内存，提升整体性能。

例子：

```html
<ul id="myLinks">
    <li id="goSomewhere">Go somewhere</li>
    <li id="doSomething">Do something</li>
    <li id="sayHi">Say hi</li>
</ul>
```

这里的 `HTML` 包含 3 个列表项，在被点击时应该执行某个操作。对此，通常的做法是像这样指定 3 个事件处理程序：

```javascript
let item1 = document.getElementById("goSomewhere");
let item2 = document.getElementById("doSomething");
let item3 = document.getElementById("sayHi");
item1.addEventListener("click", (event) => {
    location.href = "http://www.wrox.com ";
});
item2.addEventListener("click ", (event) => {
    document.title = "I changed the document 's title";
});
item3.addEventListener("click", (event) => {
    console.log("hi");
});
```

使用事件委托，只要给所有元素共同的祖先节点添加一个事件处理程序，就可以解决问题：

```javascript
let list = document.getElementById("myLinks");
list.addEventListener("click", (event) => {
    let target = event.target;
    switch (target.id) {
        case "doSomething":
            document.title = "I changed the document's title";
            break;
        case "goSomewhere":
            location.href = "http:// www.wrox.com";
            break;
        case "sayHi":
            console.log("hi");
            break;
    }
});
```

> 在这个例子中，我们使用 `event.target` 来获取事件的目标元素（也就是最里面的元素）。如果我们想要获取处理这个事件的元素（在这个例子中是 `ul`），我们可以使用 `event.currentTarget`。

## 参考

1. [事件冒泡](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Building_blocks/Event_bubbling)
2. [JavaScript 深入系列之 DOM 事件响应机制](https://github.com/yuanyuanbyte/Blog/issues/93)