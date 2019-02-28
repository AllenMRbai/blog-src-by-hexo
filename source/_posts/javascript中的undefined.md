---
title: javascript中的undefined
date: 2019-02-28 15:38:50
tags: javascript
---

javascript 中的`undefined`有很多的迷惑性，非常让人难以理解。

## `undefined` 等同于 "未声明" 吗？

`undefined`中文意思为“未定义”，我们可能就会认为`undefined`就是变量“未声明”。

但其实并不是这样，看下面的例子：

```javascript
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

我们只声明了变量`a`，但获取`a`时，返回的结果是`undefined`,获取`b`时抛出了一个异常。

这说明`undefined`不是“未声明”的意思，在引用“未声明”的变量时会抛出`ReferenceError`的异常。

### 安全守卫（safety guard）

上面的例子说明了“未声明”的变量在引用时会抛出异常。但是我们的代码有时需要依赖全局的某个变量,而且我们不知道该变量是否已被声明。我们可能会像下面那样写一个`if`语句来判断该变量是否存在。

```javascript
if (DEBUG) {
  console.log("Debugging is starting");
}
```

在上面例子中，如果`DEBUG`并未被声明的话，会抛出`ReferenceError`并打断了代码的运行。我们需要的是当`DEBUG`为 true 时，执行内部的逻辑，`DEBUG`为 falsy 或“未声明”时，跳过里面的代码，而不是抛出异常。

以下有多种方式实现一个安全守卫，防止 javascript 抛出异常。

##### 方法 1：`typeof`

```javascript
var a;

typeof a; // "undefined"
typeof b; // "undefined"
```

从上面的例子中可以发现，无论是已声明的还是未声明的变量，`typeof` 返回的结果都是`undefined`，而且`typeof b`也不会抛出异常。

可以利用这个特性，作为一个 safety guard。代码如下：

```javascript
if (typeof DEBUG !== "undefined") {
  console.log("Debugging is starting");
}
```

##### 方法 2：对象属性

```javascript
var person = {
  name: "Allen"
};

person.gender; // undefined
```

上面的例子中可以得出，访问一个未定义的对象属性不会抛出异常。

我们知道，javascript 中全局的变量，其实就是`window`对象上的属性。结合上面得出的结论，我们可以利用该特性，实现 safety guard。

```javascript
if (window.DEBUG) {
  console.log("Debugging is starting");
}
```

但是，这个方法，不利与跨平台，应为在 nodejs 中没有`window`对象。
