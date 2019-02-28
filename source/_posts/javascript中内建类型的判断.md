---
title: javascript中内建类型的判断
date: 2019-02-28 15:41:01
tags: javascript
---

javascript 中内建类型（built-in types）有七个,包括 `null` `undefined` `boolean` `number` `string` `symbol` `object`。其中除了 `object` 外，另外的 6 个类型都是原始（primitive）类型。javascript 内的函数和数组都是 `object` 类型。

在平时工作中，我们需要判断数据的类型。

## 用 `typeof` 来区分内建类型

```javascript
typeof null; // "object"
typeof undefined; // "undefined"
typeof true; // "boolean"
typeof 10; // "number"
typeof "hello"; // "string"
typeof Symbol(); // "symbol"
typeof { name: "Allen" }; // "object"

typeof [1, 2, 3]; // "object"
typeof function hi() {}; // "function"
typeof NaN; // "number"
```

从上面的结果中可以看出 `typeof` 返回的结果基本是正确的,除了 `typeof null` 和 `typeof function`。

`typeof nulll` 返回 `object` 是早期语言设计上的错误。由于太多代码利用了这个错误特性，所以浏览器基本上没可能去修正这个 bug 了，不然会导致更多的 bug。

`typeof function hi(){}` 返回 `function` 看似很合理，其实 `function` 在 JS 内并不是顶级的内建类型，它是 `object` 的子类型，所以它返回 `object` 会更合理些。
