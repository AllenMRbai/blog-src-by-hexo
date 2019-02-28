---
title: event loop in javascript
date: 2019-02-28 15:34:29
tags: javascript
---

浏览器中的 event loop 和 Node.js 中的不一样，分开讨论。

## 浏览器中的 event loop

一张图看懂浏览器中的 event loop:

![浏览器中的event loop](https://note.youdao.com/yws/res/6030/WEBRESOURCE67b4b1f80c069e9d458173d2f4b5fdb0)

这里有一个调用栈（Call Stack）、一个任务队列（Task Queue）和一个 微任务队列（Microtask Queue）。

异步接口根据类型可以分为宏任务（macrotask 也称 task）和微任务（microtask）。

- 微任务：`Promise` `mutation observer`
- 宏任务：`script` `setTimeout` `setInterval` `xhr` `UI事件`

### 执行流程

JavaScript 是单线程的，遇到异步接口时它并不会被阻塞，而是将异步接口的回调函数挂起，然后继续执行同步代码。

异步接口会在其它的线程上运行，当异步接口处理完毕，被挂起的回调函数才会被添加到对应的队列中。微任务会被添加到`Microtask Queue`,宏任务会被添加到`Task Queue`。

当 JavaScript 脚本（JavaScript 脚本是宏任务）执行完毕，并且调用栈为空时，就会先去取`Microtask Queue`内的回调函数放到调用栈内执行，执行完后再去取，直到清空`Microtask Queue`为止。

清空`Microtask Queue`后，会按情况更新 UI 界面,然后从`Task Queue`内取出一个任务放到调用栈内执行，执行完毕后按刚才的步骤去清空`Microtask Queue`,之后就是按前面的规则循环。

总结：浏览器会依次执行每个宏任务，每个宏任务之后会按序清空`Microtask Queue`并根据需要更新 UI。

### 练习

以下练习来自知乎内的一篇文章，参考链接在底部。

##### 练习一

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

Promise.resolve()
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(4);
  });

console.log(5);
// Google Chrome v72 内运行结果
// 1 5 3 4 2
```

##### 练习二

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
  new Promise(resolve => {
    console.log(4);
    resolve();
  }).then(() => {
    console.log(5);
  });
});

new Promise(resolve => {
  console.log(7);
  resolve();
}).then(() => {
  console.log(8);
});

setTimeout(() => {
  console.log(9);
  new Promise(resolve => {
    console.log(11);
    resolve();
  }).then(() => {
    console.log(12);
  });
});
// Google Chrome v72 内运行结果
// 1 7 8 2 4 5 9 11 12
```

## Node 中的 event loop

Node.js 中的 event loop 实现和浏览器中的不同。下图是 Node 中 event loop 的实现：

![eventloop_in_node.jpg](https://note.youdao.com/yws/res/6232/WEBRESOURCE62cc8eef1d6861fc872a9c8d0fd9ae17)

Node 中的 event loop 有多个阶段，每个阶段都有对应的任务队列（Task Queue）。如图所示有：

- `expired timers/intervals queue`：即到期的 `setTimeout`和`setInterval`
- `IO events queue`：包括文件读取、网络等 IO 操作
- `immediates queue`：通过`setImmediate`注册的函数
- `close handlers queue`： 各种资源关闭的回调函数，比如 TCP 连接断开

Node 中的微任务队列（Microtask Queue）也不只一个，每个 event loop 阶段之后会清空所有的微任务队列,如图所示有：

- `next tick queue`： 通过`process.nextTick`注册的函数，优先被清空
- `other micro tasks queue`： 其它微任务，常见的如`Promise`

Node 中异步接口

- 微任务：`Promise` `process.nextTick`
- 宏任务：`script` `setTimeout` `setInterval` `setImmediate` 各类 IO 相关的异步接口 和 各类资源关闭的异步接口

### 流程

按顺序执行每个阶段，每个阶段会清空对应的任务队列（Task Queue），每个阶段之后会清空微任务队列(先`next tick queue`再`other micro tasks queue`)

与浏览器的 event loop 不同的地方：

- 清空每个阶段对应的任务队列后再清空微任务队列
- 有多个任务队列和微任务队列

### 练习

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
  new Promise(resolve => {
    console.log(4);
    resolve();
  }).then(() => {
    console.log(5);
  });
  process.nextTick(() => {
    console.log(3);
  });
});

new Promise(resolve => {
  console.log(7);
  resolve();
}).then(() => {
  console.log(8);
});

process.nextTick(() => {
  console.log(6);
});

setTimeout(() => {
  console.log(9);
  process.nextTick(() => {
    console.log(10);
  });
  new Promise(resolve => {
    console.log(11);
    resolve();
  }).then(() => {
    console.log(12);
  });
});

// node v10.15.1
// 1 7 6 8 2 4 9 11 3 10 5 12
```

## 参考

- 知乎文章[《Event Loop 的规范和实现》](https://zhuanlan.zhihu.com/p/33087629)，大量参考了这篇文章。
- 掘金小册[《前端面试之道》](https://juejin.im/book/5bdc715fe51d454e755f75ef)的第七章节《Event Loop》
- [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
